# Especificação do Mecanismo de Pausa Condicional

## 1. Visão Geral

Este documento detalha o mecanismo de pausa condicional que será implementado no sistema de governança de changesets para interagir com o pipeline Jenkins. O mecanismo tomará decisões de pausa com base na severidade das exceções `linter-ignore-rule` detectadas.

## 2. Critérios de Pausa Baseados na Severidade

### 2.1 Níveis de Severidade
O sistema utiliza quatro níveis de severidade conforme definido no documento `exception_classification_criteria.md`:

1. **Crítica (Critical)**: Máxima severidade
2. **Alta (High)**: Severidade muito alta
3. **Média (Medium)**: Severidade moderada
4. **Baixa (Low)**: Severidade mínima

### 2.2 Regras de Pausa por Severidade

| Severidade | Pausa Pipeline | Grupo de Aprovação | Timeout | Aprovação Automática |
|------------|----------------|-------------------|---------|---------------------|
| Crítica    | Sim (obrigatória) | AD-GROUP apenas | 24 horas | Não |
| Alta       | Sim (obrigatória) | AD-GROUP apenas | 12 horas | Não |
| Média      | Sim (opcional/configurável) | DEVELOPERS + TEAM-LEADS | 6 horas | Configurável |
| Baixa      | Não | N/A | N/A | Sim |

### 2.3 Fatores Adicionais que Influenciam a Pausa

#### 2.3.1 Ambiente de Implantação
- **Produção**: Aumenta uma categoria de severidade
- **Homologação**: Mantém severidade base
- **Desenvolvimento**: Reduz uma categoria de severidade (mínimo Baixa)

#### 2.3.2 Frequência de Uso por Autor
- **Primeira vez**: Mais rigoroso na avaliação
- **Uso repetido**: Pode permitir padrões aprovados
- **Histórico problemático**: Aumento de severidade

#### 2.3.3 Justificativa Fornecida
- **Clara e detalhada**: Pode reduzir severidade
- **Genérica ou ausente**: Pode aumentar severidade

## 3. Implementação Técnica do Mecanismo de Pausa

### 3.1 Classe de Decisão de Pausa
```python
# /src/core/pause_decision_engine.py
from enum import Enum
from typing import List, Dict, Optional
from exceptions.models.exception_record import ExceptionRecord
from src.models.environment import Environment

class PauseDecision(Enum):
    NO_PAUSE = "no_pause"
    PAUSE_REQUIRED = "pause_required"
    PAUSE_OPTIONAL = "pause_optional"

class PauseMechanism:
    def __init__(self, config):
        self.config = config
        self.environment_rules = config.get('environment_adjustments', {})
        self.author_history = {}  # Seria injetado em produção
    
    def evaluate_pause_requirement(self, exceptions: List[ExceptionRecord], 
                                 environment: Environment) -> Dict:
        """
        Avalia se o pipeline deve ser pausado com base nas exceções detectadas
        
        Returns:
            Dict com decisão de pausa e detalhes para aprovação
        """
        if not exceptions:
            return {
                'decision': PauseDecision.NO_PAUSE,
                'required_approvers': [],
                'timeout_hours': 0,
                'reason': 'Nenhuma exceção detectada'
            }
        
        # Calcular severidade máxima ajustada
        max_severity = self._calculate_adjusted_max_severity(exceptions, environment)
        
        # Tomar decisão com base na severidade
        decision_info = self._make_pause_decision(max_severity)
        
        # Adicionar detalhes específicos do ambiente
        decision_info['environment'] = environment.value
        
        return decision_info
    
    def _calculate_adjusted_max_severity(self, exceptions: List[ExceptionRecord], 
                                       environment: Environment) -> str:
        """Calcula a severidade máxima ajustada por ambiente e outros fatores"""
        severities = []
        
        for exception in exceptions:
            base_severity = exception.calculated_severity
            adjusted_severity = self._adjust_severity_for_environment(
                base_severity, environment
            )
            severities.append(adjusted_severity)
        
        # Retornar a severidade mais alta
        severity_hierarchy = ['low', 'medium', 'high', 'critical']
        max_severity_level = max(severity_hierarchy.index(s) for s in severities)
        return severity_hierarchy[max_severity_level]
    
    def _adjust_severity_for_environment(self, severity: str, 
                                       environment: Environment) -> str:
        """Ajusta a severidade com base no ambiente"""
        adjustments = self.environment_rules.get(environment.value, {})
        adjustment_value = adjustments.get(severity, 0)
        
        severity_hierarchy = ['low', 'medium', 'high', 'critical']
        current_index = severity_hierarchy.index(severity)
        new_index = max(0, min(len(severity_hierarchy) - 1, current_index + adjustment_value))
        
        return severity_hierarchy[new_index]
    
    def _make_pause_decision(self, max_severity: str) -> Dict:
        """Toma a decisão de pausa com base na severidade máxima"""
        pause_config = self.config.get('pause_rules', {})
        severity_config = pause_config.get(max_severity, {})
        
        return {
            'decision': PauseDecision(severity_config.get('pause_type', 'no_pause')),
            'required_approvers': severity_config.get('approvers', []),
            'timeout_hours': severity_config.get('timeout_hours', 0),
            'reason': f'Exceções de severidade {max_severity} detectadas'
        }
```

### 3.2 Configuração do Mecanismo de Pausa
```yaml
# /config/pause_mechanism.yaml
pause_mechanism:
  # Regras básicas de pausa por severidade
  pause_rules:
    critical:
      pause_type: pause_required
      approvers: ["AD-GROUP"]
      timeout_hours: 24
      notification_priority: highest
    
    high:
      pause_type: pause_required
      approvers: ["AD-GROUP"]
      timeout_hours: 12
      notification_priority: high
    
    medium:
      pause_type: pause_required
      approvers: ["DEVELOPERS", "TEAM-LEADS"]
      timeout_hours: 6
      notification_priority: medium
    
    low:
      pause_type: no_pause
      approvers: []
      timeout_hours: 0
      notification_priority: low
  
  # Ajustes por ambiente
  environment_adjustments:
    production:
      critical: 0    # Já é o máximo
      high: 1        # High -> Critical
      medium: 1      # Medium -> High
      low: 1         # Low -> Medium
    
    staging:
      critical: 0
      high: 0
      medium: 0
      low: 0         # Sem ajuste
    
    development:
      critical: -1   # Critical -> High
      high: -1       # High -> Medium
      medium: -1     # Medium -> Low
      low: 0         # Já é o mínimo
  
  # Configurações de aprovação automática
  auto_approval:
    enabled: true
    max_severity_for_auto: low
    excluded_environments: [production]
    
    # Configurações para padrões conhecidos
    trusted_patterns:
      - author: "dba.*"
        rules: ["no-drop-table", "no-delete-all"]
        max_severity: medium
        auto_approve_if_justified: true
      
      - author: ".*intern.*"
        max_severity: low
        require_additional_review: true
```

## 4. Integraação com o Sistema de Exceções

### 4.1 Serviço de Coordenação
```python
# /exceptions/services/pause_coordination_service.py
from src.core.pause_decision_engine import PauseMechanism
from exceptions.repositories.exception_repository import ExceptionRepository
from ci_scripts.jenkins.approval_gate import ApprovalGate

class PauseCoordinationService:
    def __init__(self, pause_mechanism: PauseMechanism, 
                 exception_repo: ExceptionRepository):
        self.pause_mechanism = pause_mechanism
        self.exception_repo = exception_repo
    
    def process_exceptions_for_pause_decision(self, changeset_files: List[str], 
                                           environment: str) -> Dict:
        """
        Processa exceções detectadas e determina necessidade de pausa
        
        Args:
            changeset_files: Lista de arquivos de changeset para análise
            environment: Ambiente de implantação
            
        Returns:
            Dict com decisão de pausa e informações para integração com Jenkins
        """
        # Detectar todas as exceções nos changesets
        all_exceptions = []
        for file_path in changeset_files:
            exceptions = self.exception_repo.find_exceptions_in_file(file_path)
            all_exceptions.extend(exceptions)
        
        # Converter string de ambiente para enum
        env_enum = self._convert_environment_string(environment)
        
        # Avaliar necessidade de pausa
        pause_decision = self.pause_mechanism.evaluate_pause_requirement(
            all_exceptions, env_enum
        )
        
        # Preparar resposta formatada para Jenkins
        jenkins_response = self._format_for_jenkins_integration(
            pause_decision, all_exceptions
        )
        
        return jenkins_response
    
    def _convert_environment_string(self, env_string: str):
        """Converte string de ambiente para enum"""
        from src.models.environment import Environment
        mapping = {
            'prod': Environment.PRODUCTION,
            'production': Environment.PRODUCTION,
            'staging': Environment.STAGING,
            'homolog': Environment.STAGING,
            'dev': Environment.DEVELOPMENT,
            'development': Environment.DEVELOPMENT
        }
        return mapping.get(env_string.lower(), Environment.DEVELOPMENT)
    
    def _format_for_jenkins_integration(self, pause_decision: Dict, 
                                      exceptions: List) -> Dict:
        """Formata a decisão para integração com Jenkins"""
        return {
            'shouldPause': pause_decision['decision'] != 'no_pause',
            'pauseType': pause_decision['decision'].value,
            'requiredApprovers': pause_decision['required_approvers'],
            'timeoutHours': pause_decision['timeout_hours'],
            'exceptionSummary': self._summarize_exceptions(exceptions),
            'approvalUrl': '/api/approval/pending' if pause_decision['decision'] != 'no_pause' else None
        }
    
    def _summarize_exceptions(self, exceptions: List) -> Dict:
        """Cria um resumo das exceções para exibição"""
        severity_count = {}
        for exc in exceptions:
            severity = exc.calculated_severity
            severity_count[severity] = severity_count.get(severity, 0) + 1
        
        return {
            'totalCount': len(exceptions),
            'bySeverity': severity_count,
            'requiresADApproval': any(exc.calculated_severity in ['high', 'critical'] 
                                    for exc in exceptions)
        }
```

## 5. API REST para Integração

### 5.1 Endpoint de Avaliação de Pausa
```python
# /src/api/pause_controller.py
from flask import Blueprint, request, jsonify
from exceptions.services.pause_coordination_service import PauseCoordinationService

pause_bp = Blueprint('pause', __name__)

@pause_bp.route('/api/pause/evaluate', methods=['POST'])
def evaluate_pause_requirement():
    """
    Avalia se o pipeline deve ser pausado com base nas exceções detectadas
    
    Request Body:
    {
        "changesetFiles": ["file1.sql", "file2.xml"],
        "environment": "production",
        "buildInfo": {
            "jobName": "liquibase-validation",
            "buildNumber": 12345
        }
    }
    
    Response:
    {
        "shouldPause": true,
        "pauseType": "pause_required",
        "requiredApprovers": ["AD-GROUP"],
        "timeoutHours": 24,
        "exceptionSummary": {
            "totalCount": 2,
            "bySeverity": {
                "critical": 1,
                "high": 1
            },
            "requiresADApproval": true
        },
        "approvalUrl": "/api/approval/pending/12345",
        "jenkinsIntegration": {
            "stageName": "Aprovação para Exceções Críticas",
            "timeoutConfig": "timeout(time: 24, unit: 'HOURS')",
            "inputMessage": "Changeset contém exceções críticas que requerem aprovação do AD-GROUP"
        }
    }
    """
    try:
        data = request.get_json()
        changeset_files = data.get('changesetFiles', [])
        environment = data.get('environment', 'development')
        
        # Processar decisão de pausa
        coordination_service = PauseCoordinationService()  # Injeção de dependência
        pause_decision = coordination_service.process_exceptions_for_pause_decision(
            changeset_files, environment
        )
        
        return jsonify(pause_decision)
    
    except Exception as e:
        return jsonify({
            'error': 'Failed to evaluate pause requirement',
            'message': str(e)
        }), 500
```

## 6. Mecanismo de Override e Configuração

### 6.1 Configuração por Projeto
```yaml
# /projects/project-name/pause-overrides.yaml
project_pause_overrides:
  # Override para regras específicas
  rule_specific:
    "no-drop-table":
      override_severity_to: high  # Sempre tratar como high
      require_ad_approval: true
      custom_timeout_hours: 48
    
    "has-description":
      disable_pause: true  # Nunca pausar para esta regra
      auto_approve: true
  
  # Override para autores específicos
  author_specific:
    "dba.admin":
      max_auto_approve_severity: medium
      bypass_environment_restrictions: true
    
    "intern.*":
      always_pause: true
      minimum_timeout_hours: 24
      required_approvers_addition: ["MENTOR"]
  
  # Configurações específicas do ambiente
  environment_overrides:
    production:
      enforce_all_pauses: true
      minimum_timeout_hours: 12
      additional_approvers: ["SECURITY-OFFICER"]
    
    development:
      allow_auto_approval_patterns: true
      reduced_timeout_factor: 0.5
```

### 6.2 Classe de Configuração Hierárquica
```python
# /src/config/pause_config_manager.py
import yaml
from typing import Dict, Any

class PauseConfigManager:
    def __init__(self, base_config_path: str, project_config_path: str = None):
        self.base_config = self._load_config(base_config_path)
        self.project_config = self._load_config(project_config_path) if project_config_path else {}
    
    def _load_config(self, path: str) -> Dict[Any, Any]:
        """Carrega configuração de arquivo YAML"""
        if not path:
            return {}
        
        try:
            with open(path, 'r') as f:
                return yaml.safe_load(f)
        except FileNotFoundError:
            return {}
    
    def get_pause_rules_for_severity(self, severity: str) -> Dict:
        """Obtém regras de pausa para uma severidade específica, com overrides"""
        base_rules = self.base_config.get('pause_rules', {}).get(severity, {})
        
        # Aplicar overrides do projeto se existirem
        if self.project_config:
            project_rules = self.project_config.get('pause_rules', {}).get(severity, {})
            base_rules.update(project_rules)
        
        return base_rules
    
    def get_environment_adjustment(self, environment: str, severity: str) -> int:
        """Obtém ajuste de severidade para um ambiente específico"""
        base_adjustment = self.base_config.get('environment_adjustments', {}) \
                                         .get(environment, {}) \
                                         .get(severity, 0)
        
        # Verificar override do projeto
        if self.project_config:
            project_adjustment = self.project_config.get('environment_adjustments', {}) \
                                              .get(environment, {}) \
                                              .get(severity, base_adjustment)
            return project_adjustment
        
        return base_adjustment
    
    def should_override_rule_pause(self, rule_name: str) -> Dict:
        """Verifica se uma regra específica tem configuração de override"""
        if not self.project_config:
            return {}
        
        rule_overrides = self.project_config.get('rule_specific', {})
        return rule_overrides.get(rule_name, {})
    
    def get_author_specific_config(self, author: str) -> Dict:
        """Obtém configuração específica para um autor"""
        if not self.project_config:
            return {}
        
        author_configs = self.project_config.get('author_specific', {})
        
        # Verificar padrões regex
        import re
        for pattern, config in author_configs.items():
            if re.match(pattern, author):
                return config
        
        # Verificar correspondência exata
        return author_configs.get(author, {})
```

## 7. Considerações de Performance

### 7.1 Caching de Decisões
```python
# /src/core/pause_cache.py
import hashlib
import time
from typing import Dict, Any

class PauseDecisionCache:
    def __init__(self, ttl_seconds: int = 300):  # 5 minutos por padrão
        self.ttl = ttl_seconds
        self.cache = {}
    
    def _generate_key(self, exceptions_hash: str, environment: str) -> str:
        """Gera chave única para cache"""
        key_string = f"{exceptions_hash}:{environment}"
        return hashlib.md5(key_string.encode()).hexdigest()
    
    def store_decision(self, exceptions: List, environment: str, decision: Dict):
        """Armazena decisão no cache"""
        exceptions_hash = self._hash_exceptions(exceptions)
        key = self._generate_key(exceptions_hash, environment)
        
        self.cache[key] = {
            'decision': decision,
            'timestamp': time.time(),
            'expires_at': time.time() + self.ttl
        }
    
    def get_cached_decision(self, exceptions: List, environment: str) -> Optional[Dict]:
        """Recupera decisão do cache se válida"""
        exceptions_hash = self._hash_exceptions(exceptions)
        key = self._generate_key(exceptions_hash, environment)
        
        cached_item = self.cache.get(key)
        if not cached_item:
            return None
        
        # Verificar se expirou
        if time.time() > cached_item['expires_at']:
            del self.cache[key]
            return None
        
        return cached_item['decision']
    
    def _hash_exceptions(self, exceptions: List) -> str:
        """Gera hash das exceções para uso como chave"""
        # Simplificação - em produção seria um hashing mais robusto
        exception_str = ''.join(str(e.__dict__) for e in exceptions)
        return hashlib.md5(exception_str.encode()).hexdigest()
```

## 8. Testabilidade do Mecanismo

### 8.1 Testes Unitários
```python
# /tests/unit/test_pause_mechanism.py
import unittest
from unittest.mock import Mock
from src.core.pause_decision_engine import PauseMechanism, PauseDecision
from exceptions.models.exception_record import ExceptionRecord

class TestPauseMechanism(unittest.TestCase):
    def setUp(self):
        self.config = {
            'pause_rules': {
                'critical': {'pause_type': 'pause_required', 'approvers': ['AD-GROUP'], 'timeout_hours': 24},
                'high': {'pause_type': 'pause_required', 'approvers': ['AD-GROUP'], 'timeout_hours': 12},
                'medium': {'pause_type': 'pause_required', 'approvers': ['DEVELOPERS'], 'timeout_hours': 6},
                'low': {'pause_type': 'no_pause', 'approvers': [], 'timeout_hours': 0}
            },
            'environment_adjustments': {
                'production': {'high': 1, 'medium': 1, 'low': 1}
            }
        }
        self.pause_mechanism = PauseMechanism(self.config)
    
    def test_no_exceptions_no_pause(self):
        result = self.pause_mechanism.evaluate_pause_requirement([], 'development')
        self.assertEqual(result['decision'], PauseDecision.NO_PAUSE)
    
    def test_critical_exception_requires_pause(self):
        exception = Mock(spec=ExceptionRecord)
        exception.calculated_severity = 'critical'
        
        result = self.pause_mechanism.evaluate_pause_requirement([exception], 'development')
        self.assertEqual(result['decision'], PauseDecision.PAUSE_REQUIRED)
        self.assertIn('AD-GROUP', result['required_approvers'])
        self.assertEqual(result['timeout_hours'], 24)
    
    def test_environment_adjustment(self):
        exception = Mock(spec=ExceptionRecord)
        exception.calculated_severity = 'medium'
        
        # Em desenvolvimento, deve pausar para DEVELOPERS
        result_dev = self.pause_mechanism.evaluate_pause_requirement([exception], 'development')
        self.assertIn('DEVELOPERS', result_dev['required_approvers'])
        
        # Em produção, medium se torna high, então precisa de AD-GROUP
        result_prod = self.pause_mechanism.evaluate_pause_requirement([exception], 'production')
        self.assertIn('AD-GROUP', result_prod['required_approvers'])
```

## 9. Monitoramento e Métricas

### 9.1 Métricas de Pausa
```python
# /src/monitoring/pause_metrics.py
from prometheus_client import Counter, Histogram, Gauge

class PauseMetrics:
    def __init__(self):
        self.pause_decisions = Counter(
            'pause_decisions_total',
            'Total de decisões de pausa tomadas',
            ['decision_type', 'severity', 'environment']
        )
        
        self.pause_duration = Histogram(
            'pause_duration_seconds',
            'Duração das pausas em segundos',
            ['severity']
        )
        
        self.approval_time = Histogram(
            'approval_time_seconds',
            'Tempo até a aprovação/rejeição',
            ['approver_group']
        )
        
        self.current_paused_pipelines = Gauge(
            'current_paused_pipelines',
            'Número de pipelines atualmente pausados',
            ['severity']
        )
    
    def record_pause_decision(self, decision_type: str, severity: str, environment: str):
        """Registra uma decisão de pausa"""
        self.pause_decisions.labels(
            decision_type=decision_type,
            severity=severity,
            environment=environment
        ).inc()
    
    def record_pause_duration(self, duration_seconds: float, severity: str):
        """Registra a duração de uma pausa"""
        self.pause_duration.labels(severity=severity).observe(duration_seconds)
    
    def increment_paused_pipelines(self, severity: str):
        """Incrementa contador de pipelines pausados"""
        self.current_paused_pipelines.labels(severity=severity).inc()
    
    def decrement_paused_pipelines(self, severity: str):
        """Decrementa contador de pipelines pausados"""
        self.current_paused_pipelines.labels(severity=severity).dec()
```

## 10. Documentação de Uso

Esta especificação fornece uma implementação completa do mecanismo de pausa condicional baseado na severidade das exceções. A implementação permite:

1. **Avaliação dinâmica** da necessidade de pausa com base em múltiplos fatores
2. **Configuração flexível** para diferentes ambientes e projetos
3. **Integração direta** com o pipeline Jenkins
4. **Monitoramento robusto** das decisões tomadas
5. **Extensibilidade** para futuras melhorias

O mecanismo pode ser facilmente integrado ao fluxo existente de detecção de exceções e coordenação com o sistema de validação.