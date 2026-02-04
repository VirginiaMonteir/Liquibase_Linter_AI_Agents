# Logs e Auditoria para o Processo de Aprovação

## 1. Visão Geral

Este documento detalha a estratégia de logging e auditoria para o processo de aprovação de exceções `linter-ignore-rule`. O sistema fornece rastreabilidade completa, segurança e compliance para todas as decisões tomadas durante o pipeline de CI/CD.

## 2. Estrutura de Logs

### 2.1 Tipos de Logs
1. **Audit Logs** - Registros de decisões e ações de aprovação
2. **Execution Logs** - Logs detalhados da execução do processo
3. **Security Logs** - Eventos relacionados à segurança
4. **Error Logs** - Erros e exceções ocorridos
5. **Performance Logs** - Métricas de desempenho

### 2.2 Níveis de Log
- **DEBUG** - Informações detalhadas para diagnóstico
- **INFO** - Eventos importantes do processo
- **WARNING** - Situações que requerem atenção
- **ERROR** - Erros que impedem funcionalidades
- **CRITICAL** - Erros graves que afetam o sistema

## 3. Formatos de Log

### 3.1 Estrutura Padrão JSON
```json
{
  "timestamp": "2026-02-02T15:30:45.123Z",
  "level": "INFO",
  "service": "approval-service",
  "module": "exception-handler",
  "action": "exception_detected",
  "correlation_id": "uuid-v4-correlation",
  "session_id": "uuid-v4-session",
  "user": "system|username",
  "source": {
    "type": "liquibase-changeset",
    "file": "db/changelog/001-init.xml",
    "line": 15
  },
  "payload": {
    "exception_id": "exc-123",
    "rule": "no-drop-table",
    "severity": "high",
    "changeset": "dev.junior:init-changeset"
  },
  "metadata": {
    "environment": "production",
    "build_number": "12345",
    "pipeline": "liquibase-validation",
    "version": "1.2.3"
  }
}
```

### 3.2 Logs de Auditoria Específicos

#### 3.2.1 Detecção de Exceção
```json
{
  "timestamp": "2026-02-02T15:30:45.123Z",
  "level": "INFO",
  "service": "detection-service",
  "action": "exception_detected",
  "user": "system",
  "correlation_id": "det-uuid-12345",
  "payload": {
    "exception": {
      "id": "exc-001",
      "type": "linter-ignore-rule",
      "rule": "no-drop-table",
      "changeset": {
        "author": "dev.junior",
        "id": "drop-temp-tables"
      },
      "location": {
        "file": "db/migrations/001-drop-temp.xml",
        "line": 15
      },
      "justification": "Remoção necessária de tabelas temporárias"
    }
  },
  "metadata": {
    "detection_method": "pattern_matching",
    "confidence_score": 0.98
  }
}
```

#### 3.2.2 Classificação de Exceção
```json
{
  "timestamp": "2026-02-02T15:30:46.456Z",
  "level": "INFO",
  "service": "classification-service",
  "action": "exception_classified",
  "user": "system",
  "correlation_id": "det-uuid-12345",
  "payload": {
    "exception_id": "exc-001",
    "original_severity": "high",
    "adjusted_severity": "high",
    "category": "security",
    "requires_approval": true,
    "reasoning": "Regra no-drop-table sempre requer aprovação"
  },
  "environment": "production"
}
```

#### 3.2.3 Registro de Aprovação
```json
{
  "timestamp": "2026-02-02T16:45:22.789Z",
  "level": "INFO",
  "service": "approval-service",
  "action": "approval_recorded",
  "user": "admin.ad",
  "correlation_id": "det-uuid-12345",
  "payload": {
    "exception_id": "exc-001",
    "decision": "approve",
    "justification": "Exceção aprovada após validação de necessidade de negócio",
    "approval_time": "2026-02-02T16:45:22.789Z",
    "response_time_seconds": 4477
  },
  "metadata": {
    "approver_role": "AD-GROUP",
    "build_number": "12345"
  }
}
```

#### 3.2.4 Timeout de Aprovação
```json
{
  "timestamp": "2026-02-03T15:30:45.123Z",
  "level": "WARN",
  "service": "approval-service",
  "action": "approval_timeout",
  "user": "system",
  "correlation_id": "det-uuid-12345",
  "payload": {
    "exception_id": "exc-001",
    "deadline_missed": "2026-02-03T15:30:00Z",
    "action_taken": "pipeline_terminated"
  },
  "metadata": {
    "timeout_duration_hours": 24,
    "final_status": "timeout_expired"
  }
}
```

## 4. Implementação Técnica

### 4.1 Serviço de Logging Centralizado
```python
# /logging/audit_logger.py
import json
import logging
from datetime import datetime
from typing import Dict, Any, Optional
import uuid

class AuditLogger:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger('audit')
        self.formatter = AuditLogFormatter()
        
        # Configurar handlers baseado na configuração
        self._setup_handlers()
    
    def log_exception_detected(self, exception_data: Dict, correlation_id: str = None):
        """Registra detecção de exceção"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'INFO',
            'service': 'detection-service',
            'action': 'exception_detected',
            'user': 'system',
            'correlation_id': correlation_id or str(uuid.uuid4()),
            'payload': {
                'exception': exception_data
            }
        }
        self._write_log(log_entry)
    
    def log_exception_classified(self, exception_id: str, classification_data: Dict, 
                               correlation_id: str, environment: str):
        """Registra classificação de exceção"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'INFO',
            'service': 'classification-service',
            'action': 'exception_classified',
            'user': 'system',
            'correlation_id': correlation_id,
            'payload': {
                'exception_id': exception_id,
                **classification_data
            },
            'environment': environment
        }
        self._write_log(log_entry)
    
    def log_approval_required(self, exception_id: str, approval_data: Dict, 
                            correlation_id: str):
        """Registra necessidade de aprovação"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'INFO',
            'service': 'approval-service',
            'action': 'approval_required',
            'user': 'system',
            'correlation_id': correlation_id,
            'payload': {
                'exception_id': exception_id,
                **approval_data
            }
        }
        self._write_log(log_entry)
    
    def log_approval_recorded(self, exception_id: str, decision_data: Dict,
                            approver: str, correlation_id: str):
        """Registra decisão de aprovação"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'INFO',
            'service': 'approval-service',
            'action': 'approval_recorded',
            'user': approver,
            'correlation_id': correlation_id,
            'payload': {
                'exception_id': exception_id,
                **decision_data
            }
        }
        self._write_log(log_entry)
    
    def log_approval_timeout(self, exception_id: str, timeout_data: Dict,
                           correlation_id: str):
        """Registra timeout de aprovação"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'WARN',
            'service': 'approval-service',
            'action': 'approval_timeout',
            'user': 'system',
            'correlation_id': correlation_id,
            'payload': {
                'exception_id': exception_id,
                **timeout_data
            }
        }
        self._write_log(log_entry)
    
    def log_security_event(self, event_type: str, event_data: Dict, 
                          user: str = 'system'):
        """Registra eventos de segurança"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'SECURITY',
            'service': 'security-service',
            'action': event_type,
            'user': user,
            'correlation_id': str(uuid.uuid4()),
            'payload': event_data
        }
        self._write_log(log_entry)
    
    def log_error(self, error_type: str, error_data: Dict, 
                 correlation_id: str = None, user: str = 'system'):
        """Registra erro no sistema"""
        log_entry = {
            'timestamp': datetime.utcnow().isoformat() + 'Z',
            'level': 'ERROR',
            'service': 'error-handler',
            'action': error_type,
            'user': user,
            'correlation_id': correlation_id or str(uuid.uuid4()),
            'payload': error_data
        }
        self._write_log(log_entry)
    
    def _write_log(self, log_entry: Dict):
        """Escreve entrada de log"""
        formatted_log = self.formatter.format(log_entry)
        
        # Escrever em diferentes destinos conforme configuração
        if self.config.get('console_output', True):
            print(formatted_log)
        
        if self.config.get('file_output', True):
            self.logger.info(formatted_log)
        
        # Enviar para sistema externo de logging se configurado
        if self.config.get('external_logging', {}).get('enabled', False):
            self._send_to_external_system(log_entry)
    
    def _setup_handlers(self):
        """Configura handlers de log baseado na configuração"""
        # Configurar logger base
        handler = logging.StreamHandler()
        handler.setFormatter(logging.Formatter('%(message)s'))
        self.logger.addHandler(handler)
        self.logger.setLevel(logging.INFO)
    
    def _send_to_external_system(self, log_entry: Dict):
        """Envia log para sistema externo"""
        # Implementação para enviar logs para sistemas como:
        # - ELK Stack (Elasticsearch, Logstash, Kibana)
        # - Splunk
        # - Cloud logging services (CloudWatch, Stackdriver, etc.)
        pass

class AuditLogFormatter:
    def format(self, log_entry: Dict) -> str:
        """Formata entrada de log como JSON"""
        return json.dumps(log_entry, separators=(',', ':'))

# Configuração de exemplo
LOGGING_CONFIG = {
    'console_output': True,
    'file_output': True,
    'file_path': '/var/log/liquibase-governance/',
    'rotation': {
        'max_size_mb': 100,
        'backup_count': 5
    },
    'external_logging': {
        'enabled': False,
        'system': 'elk',  # ou 'splunk', 'cloudwatch', etc.
        'endpoint': 'https://logging.company.com'
    },
    'pii_filtering': {
        'enabled': True,
        'fields_to_mask': ['user.email', 'user.phone']
    }
}
```

### 4.2 Integração com Jenkins Pipeline
```groovy
// /ci-scripts/jenkins/steps/logging_steps.groovy
def logApprovalEvent(eventType, eventData) {
    script {
        def correlationId = env.CORRELATION_ID ?: UUID.randomUUID().toString()
        
        // Criar payload para log
        def logPayload = [
            eventType: eventType,
            eventData: eventData,
            buildNumber: BUILD_NUMBER,
            jobName: JOB_NAME,
            environment: ENVIRONMENT ?: 'unknown',
            timestamp: new Date().format("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", TimeZone.getTimeZone('UTC'))
        ]
        
        // Registrar log através do sistema de governança
        sh """
            liquibase-governance log-event \\
                --type ${eventType} \\
                --payload '${groovy.json.JsonBuilder(logPayload).toString()}' \\
                --correlation-id ${correlationId}
        """
        
        // Também registrar em variável de ambiente para rastreabilidade
        env.CORRELATION_ID = correlationId
    }
}

def logExceptionDetection(exceptionData) {
    script {
        logApprovalEvent('exception_detected', [
            exception: exceptionData
        ])
    }
}

def logApprovalDecision(exceptionId, decisionData, approver) {
    script {
        logApprovalEvent('approval_recorded', [
            exceptionId: exceptionId,
            decision: decisionData.DECISION,
            justification: decisionData.JUSTIFICATION,
            approver: approver,
            approvalTime: new Date().format("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'", TimeZone.getTimeZone('UTC'))
        ])
    }
}

def logApprovalTimeout(exceptionId, timeoutData) {
    script {
        logApprovalEvent('approval_timeout', [
            exceptionId: exceptionId,
            deadlineMissed: timeoutData.deadline,
            actionTaken: timeoutData.action,
            buildNumber: BUILD_NUMBER
        ])
    }
}
```

## 5. Estratégia de Retenção de Logs

### 5.1 Períodos de Retenção por Tipo
| Tipo de Log | Retenção | Motivo |
|-------------|----------|---------|
| Audit Logs | 7 anos | Requisitos de compliance e auditoria |
| Execution Logs | 2 anos | Depuração e análise de performance |
| Security Logs | 5 anos | Segurança e investigações |
| Error Logs | 1 ano | Resolução de problemas |
| Performance Logs | 6 meses | Otimização contínua |

### 5.2 Política de Compactação
- Logs são compactados após 30 dias
- Formato: gzip ou similar
- Taxa de compactação alvo: 80%

### 5.3 Arquivamento de Longo Prazo
- Logs antigos são movidos para storage de baixo custo
- Indexação para buscas futuras
- Compliance com normas GDPR, SOX, etc.

## 6. Segurança e Compliance

### 6.1 Proteção de Dados Sensíveis
```python
# /logging/pii_filter.py
import re
from typing import Dict, Any

class PII Filter:
    def __init__(self):
        self.pii_patterns = {
            'email': r'\b[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Z|a-z]{2,}\b',
            'phone': r'\b\d{3}-\d{3}-\d{4}\b|\b\d{10}\b|\(\d{3}\)\s*\d{3}-\d{4}',
            'ssn': r'\b\d{3}-\d{2}-\d{4}\b',
            'credit_card': r'\b\d{4}[\s-]?\d{4}[\s-]?\d{4}[\s-]?\d{4}\b'
        }
    
    def filter_sensitive_data(self, log_data: Dict[str, Any]) -> Dict[str, Any]:
        """Filtra dados sensíveis dos logs"""
        filtered_data = log_data.copy()
        return self._recursive_filter(filtered_data)
    
    def _recursive_filter(self, data: Any) -> Any:
        """Filtra recursivamente dados sensíveis"""
        if isinstance(data, dict):
            filtered_dict = {}
            for key, value in data.items():
                # Verificar se a chave indica dado sensível
                if self._is_sensitive_field(key):
                    filtered_dict[key] = self._mask_sensitive_value(value)
                else:
                    filtered_dict[key] = self._recursive_filter(value)
            return filtered_dict
        elif isinstance(data, list):
            return [self._recursive_filter(item) for item in data]
        elif isinstance(data, str):
            return self._mask_pii_in_string(data)
        else:
            return data
    
    def _is_sensitive_field(self, field_name: str) -> bool:
        """Verifica se campo contém dados sensíveis"""
        sensitive_indicators = ['email', 'phone', 'ssn', 'credit', 'password', 'token']
        return any(indicator in field_name.lower() for indicator in sensitive_indicators)
    
    def _mask_sensitive_value(self, value: Any) -> str:
        """Mascara valor sensível"""
        if value is None:
            return 'null'
        
        str_value = str(value)
        if len(str_value) <= 4:
            return '****'
        else:
            # Mostrar apenas os últimos 4 caracteres
            return '*' * (len(str_value) - 4) + str_value[-4:]
    
    def _mask_pii_in_string(self, text: str) -> str:
        """Mascara PII encontrado em strings"""
        masked_text = text
        for pii_type, pattern in self.pii_patterns.items():
            masked_text = re.sub(pattern, f'[MASKED_{pii_type.upper()}]', masked_text)
        return masked_text
```

### 6.2 Criptografia de Logs
- Logs em trânsito: TLS 1.3
- Logs em repouso: AES-256
- Chaves de criptografia rotacionadas a cada 90 dias

### 6.3 Controle de Acesso
```python
# /logging/access_control.py
from typing import List
import logging

class LogAccessControl:
    def __init__(self, config):
        self.config = config
        self.logger = logging.getLogger(__name__)
    
    def can_access_logs(self, user: str, log_type: str, 
                       access_level: str = 'read') -> bool:
        """
        Verifica se usuário pode acessar tipo específico de log
        
        Args:
            user: Nome do usuário
            log_type: Tipo de log (audit, execution, security, etc.)
            access_level: Nível de acesso (read, write, delete)
            
        Returns:
            Boolean indicando se acesso é permitido
        """
        user_roles = self._get_user_roles(user)
        required_permissions = self._get_required_permissions(log_type, access_level)
        
        # Verificar se usuário tem alguma das permissões necessárias
        has_permission = any(role in required_permissions for role in user_roles)
        
        if not has_permission:
            self.logger.warning(f"Acesso negado: usuário {user} tentou acessar logs {log_type} com nível {access_level}")
        
        return has_permission
    
    def _get_user_roles(self, user: str) -> List[str]:
        """Obtém papéis do usuário"""
        # Em implementação real, isto viria de um sistema de autenticação/autorização
        role_mapping = {
            'admin.ad': ['ADMIN', 'AUDITOR', 'SECURITY_OFFICER'],
            'dev.junior': ['DEVELOPER'],
            'team.lead': ['TEAM_LEAD', 'DEVELOPER'],
            'auditor.external': ['AUDITOR']
        }
        return role_mapping.get(user, ['GUEST'])
    
    def _get_required_permissions(self, log_type: str, access_level: str) -> List[str]:
        """Obtém permissões necessárias para acesso ao tipo de log"""
        permission_matrix = {
            'audit': {
                'read': ['ADMIN', 'AUDITOR', 'SECURITY_OFFICER', 'TEAM_LEAD'],
                'write': ['ADMIN'],
                'delete': ['ADMIN']
            },
            'execution': {
                'read': ['ADMIN', 'DEVELOPER', 'TEAM_LEAD'],
                'write': ['ADMIN'],
                'delete': ['ADMIN']
            },
            'security': {
                'read': ['ADMIN', 'SECURITY_OFFICER'],
                'write': ['ADMIN', 'SECURITY_OFFICER'],
                'delete': ['ADMIN']
            },
            'error': {
                'read': ['ADMIN', 'DEVELOPER', 'TEAM_LEAD'],
                'write': ['ADMIN'],
                'delete': ['ADMIN']
            }
        }
        
        return permission_matrix.get(log_type, {}).get(access_level, ['ADMIN'])

# Exemplo de uso
access_control = LogAccessControl(config)
if access_control.can_access_logs('admin.ad', 'audit', 'read'):
    # Permitir acesso aos logs de auditoria
    pass
```

## 7. Monitoramento e Alertas

### 7.1 Métricas de Logging
```python
# /monitoring/logging_metrics.py
from prometheus_client import Counter, Histogram, Gauge

class LoggingMetrics:
    def __init__(self):
        self.logs_written = Counter(
            'logs_written_total',
            'Total de logs escritos',
            ['log_type', 'level', 'service']
        )
        
        self.log_write_latency = Histogram(
            'log_write_latency_seconds',
            'Latência na escrita de logs',
            ['destination']
        )
        
        self.filtered_pii_logs = Counter(
            'filtered_pii_logs_total',
            'Total de logs com PII filtrado',
            ['pii_type']
        )
        
        self.log_errors = Counter(
            'log_errors_total',
            'Total de erros no sistema de logging',
            ['error_type']
        )
        
        self.log_volume = Gauge(
            'log_volume_bytes',
            'Volume total de logs gerados',
            ['log_type']
        )
    
    def record_log_written(self, log_type: str, level: str, service: str):
        """Registra escrita de log"""
        self.logs_written.labels(
            log_type=log_type,
            level=level,
            service=service
        ).inc()
    
    def record_log_latency(self, latency_seconds: float, destination: str):
        """Registra latência de escrita de log"""
        self.log_write_latency.labels(destination=destination).observe(latency_seconds)
    
    def record_filtered_pii(self, pii_type: str):
        """Registra log com PII filtrado"""
        self.filtered_pii_logs.labels(pii_type=pii_type).inc()
    
    def record_log_error(self, error_type: str):
        """Registra erro no sistema de logging"""
        self.log_errors.labels(error_type=error_type).inc()
```

### 7.2 Alertas de Monitoramento
```python
# /monitoring/log_alerts.py
class LogAlerts:
    def __init__(self, metrics, notification_service):
        self.metrics = metrics
        self.notification_service = notification_service
    
    def check_unusual_activity(self, recent_logs: List[Dict]):
        """Verifica atividade incomum nos logs"""
        # Verificar padrões suspeitos
        suspicious_patterns = self._detect_suspicious_patterns(recent_logs)
        
        if suspicious_patterns:
            self._send_security_alert(suspicious_patterns)
    
    def check_log_volume_anomalies(self, current_volume: int, baseline_volume: int):
        """Verifica anomalias no volume de logs"""
        # Alertar se volume estiver significativamente diferente da baseline
        threshold = baseline_volume * 0.5  # 50% da baseline
        
        if abs(current_volume - baseline_volume) > threshold:
            deviation_percent = ((current_volume - baseline_volume) / baseline_volume) * 100
            self._send_volume_alert(deviation_percent)
    
    def _detect_suspicious_patterns(self, logs: List[Dict]) -> List[Dict]:
        """Detecta padrões suspeitos nos logs"""
        suspicious = []
        
        # Contar tentativas de acesso negadas
        denied_access_count = sum(
            1 for log in logs 
            if log.get('action') == 'access_denied' and log.get('level') == 'WARN'
        )
        
        if denied_access_count > 10:  # Threshold configurável
            suspicious.append({
                'type': 'excessive_denied_access',
                'count': denied_access_count,
                'severity': 'HIGH'
            })
        
        # Contar erros críticos
        critical_errors = sum(
            1 for log in logs 
            if log.get('level') == 'CRITICAL'
        )
        
        if critical_errors > 5:  # Threshold configurável
            suspicious.append({
                'type': 'critical_errors_spike',
                'count': critical_errors,
                'severity': 'CRITICAL'
            })
        
        return suspicious
    
    def _send_security_alert(self, patterns: List[Dict]):
        """Envia alerta de segurança"""
        alert_message = "Atividade suspeita detectada nos logs:\n"
        for pattern in patterns:
            alert_message += f"- {pattern['type']}: {pattern['count']} ocorrências\n"
        
        self.notification_service.send_security_alert(alert_message)
    
    def _send_volume_alert(self, deviation_percent: float):
        """Envia alerta de volume anômalo"""
        severity = 'HIGH' if abs(deviation_percent) > 100 else 'MEDIUM'
        alert_message = f"Volume de logs anômalo: {deviation_percent:+.1f}% da baseline"
        
        self.notification_service.send_operational_alert(alert_message, severity)
```

## 8. Dashboard de Logs e Auditoria

### 8.1 Estrutura do Dashboard
```
DASHBOARD: Logs e Auditoria de Aprovações

┌─────────────────────────────────────────────────────────────┐
| RESUMO EM TEMPO REAL                                       |
├─────────────────────────────────────────────────────────────┤
| LOGS HOJE          │ ERROS HOJE        │ VOLUME (MB)       |
| 12,450             │ 23                │ 127.5             |
├─────────────────────────────────────────────────────────────┤
| TIPOS DE LOG                                               |
| ┌─────────────────────────────────────────────────────────┐ |
| | Audit: 45%  │ Execution: 30%  │ Security: 15%  │ Error: 10% | |
| └─────────────────────────────────────────────────────────┘ |
├─────────────────────────────────────────────────────────────┤
| EVENTOS DE APROVAÇÃO                                       |
| ┌─────────────────────────────────────────────────────────┐ |
| | Detectados: 42  │ Aprovados: 38  │ Rejeitados: 2  │ Timeout: 2 | |
| └─────────────────────────────────────────────────────────┘ |
├─────────────────────────────────────────────────────────────┤
| TEMPO MÉDIO DE APROVAÇÃO                                   |
| ┌─────────────────────────────────────────────────────────┐ |
| | Crítica: 2.1h │ Alta: 1.8h │ Média: 0.9h │ Baixa: 0.3h   | |
| └─────────────────────────────────────────────────────────┘ |
├─────────────────────────────────────────────────────────────┤
| ALERTAS ATIVOS                                             |
| ┌─────────────────────────────────────────────────────────┐ |
| | 🔴 2 Erros Críticos nos últimos 5 minutos               | |
| | ⚠️  5 Acessos Negados suspeitos                         | |
| | ℹ️  Volume de logs 150% acima da média                  | |
| └─────────────────────────────────────────────────────────┘ |
└─────────────────────────────────────────────────────────────┘
```

### 8.2 Pesquisa e Análise de Logs
```python
# /analytics/log_analyzer.py
from typing import List, Dict, Any
from datetime import datetime, timedelta
import json

class LogAnalyzer:
    def __init__(self, log_storage):
        self.log_storage = log_storage
    
    def search_logs(self, query: Dict[str, Any], 
                   time_range: tuple = None) -> List[Dict]:
        """
        Pesquisa logs com base em critérios
        
        Args:
            query: Critérios de pesquisa
            time_range: Tuple com (start_time, end_time)
            
        Returns:
            Lista de logs correspondentes
        """
        # Construir filtro para pesquisa
        filters = self._build_filters(query, time_range)
        
        # Executar pesquisa no storage
        results = self.log_storage.search(filters)
        
        return results
    
    def get_approval_statistics(self, time_range: tuple = None) -> Dict:
        """
        Obtém estatísticas de aprovações
        
        Args:
            time_range: Tuple com (start_time, end_time)
            
        Returns:
            Dicionário com estatísticas
        """
        # Pesquisar logs de aprovação no período
        approval_logs = self.search_logs(
            {'service': 'approval-service'}, 
            time_range
        )
        
        # Agrupar por tipo de decisão
        statistics = {
            'total_approvals': 0,
            'approved': 0,
            'rejected': 0,
            'timeout': 0,
            'by_severity': {},
            'average_response_time': 0
        }
        
        response_times = []
        
        for log in approval_logs:
            action = log.get('action')
            if action == 'approval_recorded':
                statistics['total_approvals'] += 1
                decision = log['payload'].get('decision', '').lower()
                if decision == 'approve':
                    statistics['approved'] += 1
                elif decision == 'reject':
                    statistics['rejected'] += 1
                
                # Coletar tempo de resposta
                response_time = log['payload'].get('response_time_seconds')
                if response_time:
                    response_times.append(response_time)
            
            elif action == 'approval_timeout':
                statistics['timeout'] += 1
        
        # Calcular tempo médio de resposta
        if response_times:
            statistics['average_response_time'] = sum(response_times) / len(response_times)
        
        return statistics
    
    def get_user_activity_report(self, user: str, 
                               time_range: tuple = None) -> Dict:
        """
        Gera relatório de atividade do usuário
        
        Args:
            user: Nome do usuário
            time_range: Tuple com (start_time, end_time)
            
        Returns:
            Relatório de atividade
        """
        user_logs = self.search_logs(
            {'user': user},
            time_range
        )
        
        report = {
            'user': user,
            'total_actions': len(user_logs),
            'actions_by_type': {},
            'actions_by_service': {},
            'first_activity': None,
            'last_activity': None
        }
        
        for log in user_logs:
            # Contar tipos de ação
            action = log.get('action', 'unknown')
            report['actions_by_type'][action] = report['actions_by_type'].get(action, 0) + 1
            
            # Contar serviços
            service = log.get('service', 'unknown')
            report['actions_by_service'][service] = report['actions_by_service'].get(service, 0) + 1
            
            # Rastrear primeira e última atividade
            log_time = datetime.fromisoformat(log['timestamp'].replace('Z', '+00:00'))
            if report['first_activity'] is None or log_time < report['first_activity']:
                report['first_activity'] = log_time
            if report['last_activity'] is None or log_time > report['last_activity']:
                report['last_activity'] = log_time
        
        return report
    
    def _build_filters(self, query: Dict[str, Any], 
                      time_range: tuple = None) -> Dict:
        """Constrói filtros para pesquisa"""
        filters = query.copy()
        
        if time_range:
            filters['timestamp_range'] = {
                'gte': time_range[0].isoformat() + 'Z',
                'lte': time_range[1].isoformat() + 'Z'
            }
        
        return filters

# Exemplo de uso
def generate_daily_report():
    """Gera relatório diário de auditoria"""
    analyzer = LogAnalyzer(log_storage)
    today = datetime.now()
    yesterday = today - timedelta(days=1)
    
    # Obter estatísticas de aprovação
    approval_stats = analyzer.get_approval_statistics((yesterday, today))
    
    # Gerar relatório
    report = {
        'date': today.strftime('%Y-%m-%d'),
        'approval_statistics': approval_stats,
        'top_users': [],  # Implementar obtenção de usuários mais ativos
        'security_events': [],  # Implementar obtenção de eventos de segurança
        'errors_summary': []  # Implementar resumo de erros
    }
    
    return report
```

## 9. Configuração e Deployment

### 9.1 Arquivo de Configuração de Logging
```yaml
# /config/logging.yaml
logging:
  # Nível de log global
  level: INFO
  
  # Configurações de formatação
  formatting:
    json_output: true
    include_correlation_id: true
    include_timestamp: true
    timezone: UTC
  
  # Destinos de log
  outputs:
    console:
      enabled: true
      level: INFO
    
    file:
      enabled: true
      path: /var/log/liquibase-governance/
      rotation:
        max_size_mb: 100
        max_files: 10
        compress_old_files: true
      
    external:
      enabled: true
      system: elk
      endpoint: https://elk.company.com
      api_key: ${ELK_API_KEY}
  
  # Tipos específicos de log
  log_types:
    audit:
      level: INFO
      retention_days: 2555  # 7 anos
      pii_filtering: true
    
    execution:
      level: DEBUG
      retention_days: 730  # 2 anos
      pii_filtering: true
    
    security:
      level: WARN
      retention_days: 1825  # 5 anos
      pii_filtering: true
      alert_on_critical: true
    
    error:
      level: ERROR
      retention_days: 365  # 1 ano
      alert_threshold: 10  # Alertar após 10 erros consecutivos
    
    performance:
      level: INFO
      retention_days: 180  # 6 meses
      metrics_collection: true
  
  # Filtragem de PII
  pii_filtering:
    enabled: true
    fields_to_mask:
      - user.email
      - user.phone
      - user.ssn
      - payment.credit_card
    mask_character: '*'
  
  # Criptografia
  encryption:
    at_rest: true
    algorithm: aes-256-gcm
    key_rotation_days: 90
  
  # Monitoramento
  monitoring:
    enable_metrics: true
    alert_thresholds:
      log_volume_spike_percent: 50
      error_rate_threshold: 0.05  # 5%
      suspicious_activity_count: 5
    
    dashboard_integration:
      enabled: true
      endpoint: https://monitoring.company.com
```

## 10. Testes e Validação

### 10.1 Testes de Logging
```python
# /tests/unit/test_audit_logger.py
import unittest
import tempfile
import os
import json
from logging.audit_logger import AuditLogger

class TestAuditLogger(unittest.TestCase):
    def setUp(self):
        self.config = {
            'console_output': False,
            'file_output': True,
            'file_path': tempfile.mkdtemp(),
            'pii_filtering': {
                'enabled': True,
                'fields_to_mask': ['user.email']
            }
        }
        self.logger = AuditLogger(self.config)
    
    def tearDown(self):
        # Limpar arquivos temporários
        import shutil
        shutil.rmtree(self.config['file_path'])
    
    def test_exception_detection_logging(self):
        """Testa logging de detecção de exceção"""
        exception_data = {
            'id': 'exc-001',
            'type': 'linter-ignore-rule',
            'rule': 'no-drop-table',
            'changeset': {
                'author': 'test.user',
                'id': 'test-changeset'
            }
        }
        
        self.logger.log_exception_detected(exception_data)
        
        # Verificar se log foi escrito
        log_files = os.listdir(self.config['file_path'])
        self.assertGreater(len(log_files), 0)
    
    def test_pii_filtering(self):
        """Testa filtragem de PII nos logs"""
        log_data = {
            'user': {
                'email': 'test@example.com',
                'name': 'Test User'
            }
        }
        
        # Este teste verificaria a implementação do filtro PII
        # Na prática, seria necessário verificar o conteúdo real do log escrito

if __name__ == '__main__':
    unittest.main()
```

Este sistema completo de logs e auditoria fornece rastreabilidade total para todas as decisões de aprovação de exceções `linter-ignore-rule`, garantindo compliance, segurança e capacidade de investigação em caso de problemas.