# Guia de Configuração por Ambiente para Integração com Microsoft Teams

## 1. Estratégia de Configuração Multi-Ambiente

### Princípios Fundamentais
- **Isolamento de ambientes**: Cada ambiente deve ter seus próprios canais e webhooks
- **Segurança por design**: URLs de produção nunca devem vazar para ambientes não-produtivos
- **Testabilidade**: Ambientes de desenvolvimento devem permitir testes sem impacto real
- **Consistência**: Estrutura de configuração uniforme entre ambientes

### Estrutura de Configuração Recomendada
```
environments/
├── development/
│   ├── teams-webhooks.yaml
│   └── teams-settings.yaml
├── staging/
│   ├── teams-webhooks.yaml
│   └── teams-settings.yaml
└── production/
    ├── teams-webhooks.yaml
    └── teams-settings.yaml
```

## 2. Configurações por Tipo de Ambiente

### Ambiente de Desenvolvimento (DEV)
Foco: Testabilidade e desenvolvimento iterativo

#### Características
- Notificações direcionadas para canais de teste
- Validação estrutural sem envio real
- Logs detalhados para debugging
- Configurações flexíveis e permissivas

#### Configuração Exemplo
```yaml
# config/environments/development/teams-webhooks.yaml
notifications:
  teams:
    enabled: true
    dry_run: true  # Não envia notificações reais
    webhook_urls:
      alerts: "https://outlook.office.com/webhook/DEV-ALERTS-TOKEN"
      approvals: "https://outlook.office.com/webhook/DEV-APPROVALS-TOKEN"
      general: "https://outlook.office.com/webhook/DEV-GENERAL-TOKEN"
      audit: "https://outlook.office.com/webhook/DEV-AUDIT-TOKEN"
    
    channels:
      alerts: "alerts-dev"
      approvals: "approvals-dev"
      general: "notifications-dev"
      audit: "audit-dev"
    
    # Configurações otimizadas para desenvolvimento
    rate_limit:
      requests_per_second: 10  # Mais permissivo em DEV
      burst_size: 20
    
    # Recursos adicionais para debugging
    debug:
      log_payloads: true
      log_responses: true
      save_to_file: true  # Salvar payloads em arquivos para inspeção
```

```yaml
# config/environments/development/teams-settings.yaml
teams_integration:
  # Simular latência para testar comportamento sob condições reais
  simulate_network_conditions: false
  
  # Detalhes para logging
  logging_level: DEBUG
  log_file_path: "/var/log/teams-notifications-dev.log"
  
  # Testes de erro
  simulate_errors:
    enabled: false
    error_rate: 0.1  # 10% de falhas simuladas
    error_types: ["429", "503"]
  
  # Validação estrutural
  validation:
    strict_mode: true  # Validar payloads mesmo sem enviar
    schema_validation: true
```

### Ambiente de Homologação (STAGING/QA)
Foco: Validação próxima da produção

#### Características
- Webhooks apontando para canais de homologação
- Configurações similares à produção
- Monitoramento habilitado
- Sem dados reais de produção

#### Configuração Exemplo
```yaml
# config/environments/staging/teams-webhooks.yaml
notifications:
  teams:
    enabled: true
    dry_run: false
    webhook_urls:
      alerts: "https://outlook.office.com/webhook/STAGING-ALERTS-TOKEN"
      approvals: "https://outlook.office.com/webhook/STAGING-APPROVALS-TOKEN"
      general: "https://outlook.office.com/webhook/STAGING-GENERAL-TOKEN"
      audit: "https://outlook.office.com/webhook/STAGING-AUDIT-TOKEN"
    
    channels:
      alerts: "alerts-staging"
      approvals: "approvals-staging"
      general: "notifications-staging"
      audit: "audit-staging"
    
    rate_limit:
      requests_per_second: 2  # Mais conservador que DEV
      burst_size: 10
    
    security:
      validate_payloads: true
      sanitize_content: true
```

```yaml
# config/environments/staging/teams-settings.yaml
teams_integration:
  simulate_network_conditions: false
  
  logging_level: INFO
  log_file_path: "/var/log/teams-notifications-staging.log"
  
  monitoring:
    enabled: true
    metrics_endpoint: "http://monitoring-service:9091/metrics"
  
  validation:
    strict_mode: true
    schema_validation: true
  
  # Configurações de resiliência próximas da produção
  resilience:
    retry_attempts: 3
    circuit_breaker_enabled: true
```

### Ambiente de Produção (PROD)
Foco: Confiabilidade, segurança e performance

#### Características
- Canais oficiais com stakeholders reais
- Políticas rigorosas de segurança
- Monitoramento completo
- Resiliência configurada para produção

#### Configuração Exemplo
```yaml
# config/environments/production/teams-webhooks.yaml
notifications:
  teams:
    enabled: true
    dry_run: false
    webhook_urls_from_vault: true  # URLs vem de secrets management
    
    # Mapeamento lógico de canais (urls reais em secrets)
    channels:
      alerts: "alerts-production"
      approvals: "approvals-production"
      general: "notifications-production"
      audit: "audit-production"
      governance: "governance-reports"
    
    rate_limit:
      requests_per_second: 1  # Conforme limites do Teams
      burst_size: 5
    
    security:
      validate_payloads: true
      sanitize_content: true
      encryption_required: true
      audit_trail: true
    
    compliance:
      data_retention_days: 90
      pii_protection_enabled: true
```

```yaml
# config/environments/production/teams-settings.yaml
teams_integration:
  logging_level: WARN
  log_file_path: "/var/log/teams-notifications-prod.log"
  
  monitoring:
    enabled: true
    metrics_endpoint: "https://monitoring.company.com/metrics"
    alerting_enabled: true
    
    alert_thresholds:
      failure_rate: 0.05  # 5%
      latency_threshold_ms: 5000
      queue_size_threshold: 100
  
  resilience:
    retry_attempts: 3
    circuit_breaker_enabled: true
    circuit_breaker:
      failure_threshold: 5
      timeout_seconds: 60
    
    queue:
      max_size: 1000
      persistence_enabled: true
      
  backup_channels:
    enabled: true
    fallback_to_email: true
    sms_alerts_for_critical: true
```

## 3. Secrets Management por Ambiente

### Estratégia de Armazenamento
| Ambiente     | Sistema de Secrets      | Rotação            | Acesso Restrito |
|--------------|-------------------------|--------------------|-----------------|
| Development  | Variáveis de ambiente   | Mensal             | Equipe técnica  |
| Staging      | Azure Key Vault/Test    | Trimestral         | QA + Devs       |
| Production   | Azure Key Vault/Prod    | Trimestral         | Apenas ops/sec  |

### Exemplo de Configuração de Secrets
```bash
# Development - .env file
TEAMS_ALERTS_WEBHOOK=https://outlook.office.com/webhook/DEV-ALERTS-TOKEN
TEAMS_APPROVALS_WEBHOOK=https://outlook.office.com/webhook/DEV-APPROVALS-TOKEN
TEAMS_GENERAL_WEBHOOK=https://outlook.office.com/webhook/DEV-GENERAL-TOKEN

# Staging - Azure Key Vault (exemplo)
/staging/teams/alerts-webhook: "https://outlook.office.com/webhook/STAGING-ALERTS-TOKEN"
/staging/teams/approvals-webhook: "https://outlook.office.com/webhook/STAGING-APPROVALS-TOKEN"
/staging/teams/general-webhook: "https://outlook.office.com/webhook/STAGING-GENERAL-TOKEN"

# Production - Azure Key Vault (exemplo)
/production/teams/alerts-webhook: "https://outlook.office.com/webhook/PROD-ALERTS-TOKEN"
/production/teams/approvals-webhook: "https://outlook.office.com/webhook/PROD-APPROVALS-TOKEN"
/production/teams/general-webhook: "https://outlook.office.com/webhook/PROD-GENERAL-TOKEN"
```

## 4. Processo de Implantação Entre Ambientes

### Pipeline de Deploy
```
[DESENVOLVIMENTO] → [TESTES AUTOMATIZADOS] → [HOMOLOGAÇÃO] → [PRODUÇÃO]

1. Desenvolvimento:
   - Configuração via variáveis de ambiente
   - Testes locais e unitários

2. CI/CD Build:
   - Validação de schemas
   - Testes de integração mockados

3. Deploy Staging:
   - Validação com webhooks de teste
   - Testes E2E em ambiente isolado

4. Deploy Production:
   - Rollout gradual
   - Monitoramento intensivo
   - Plano de rollback definido
```

### Script de Validação de Configuração
```bash
#!/bin/bash
# validate-teams-config.sh

validate_teams_config() {
    local environment=$1
    local config_file="config/environments/$environment/teams-webhooks.yaml"
    
    echo "Validando configuração Teams para ambiente: $environment"
    
    # Verificar se arquivo existe
    if [[ ! -f "$config_file" ]]; then
        echo "ERRO: Arquivo de configuração não encontrado: $config_file"
        exit 1
    fi
    
    # Verificar se webhooks estão configurados
    if grep -q "webhook_urls:" "$config_file"; then
        echo "✓ Webhooks configurados"
    else
        echo "⚠ Webhooks não encontrados na configuração"
    fi
    
    # Validar canais esperados
    local required_channels=("alerts" "approvals" "general")
    for channel in "${required_channels[@]}"; do
        if grep -q "$channel:" "$config_file"; then
            echo "✓ Canal $channel configurado"
        else
            echo "✗ Canal $channel NÃO encontrado"
        fi
    done
    
    # Verificar ambiente específico
    case $environment in
        "production")
            if grep -q "dry_run: true" "$config_file"; then
                echo "✗ Dry-run ativado em produção!"
                exit 1
            fi
            echo "✓ Configuração de produção validada"
            ;;
        "development")
            echo "✓ Configuração de desenvolvimento"
            ;;
        "staging")
            echo "✓ Configuração de staging"
            ;;
    esac
    
    echo "Validação concluída para ambiente $environment"
}

# Executar validação
validate_teams_config "$1"
```

## 5. Monitoramento e Métricas por Ambiente

### Painéis de Monitoramento Recomendados

#### Visão Geral
- Taxa de sucesso de envio por ambiente
- Volume de notificações por tipo e canal
- Latência média de entrega
- Estados do circuit breaker

#### Alertas Críticos
```yaml
# alerts/teams-notifications.yaml
alert_rules:
  - name: "HighFailureRate-TeamsNotifications"
    condition: "teams_notification_failure_rate > 0.05"
    duration: "5m"
    severity: "warning"
    environments: ["staging", "production"]
  
  - name: "CriticalFailureRate-TeamsNotifications"
    condition: "teams_notification_failure_rate > 0.20"
    duration: "2m"
    severity: "critical"
    environments: ["production"]
    actions:
      - notify_oncall
      - create_incident_ticket
      - trigger_automatic_rollback
  
  - name: "HighLatency-TeamsNotifications"
    condition: "teams_notification_avg_latency > 5s"
    duration: "10m"
    severity: "warning"
    environments: ["staging", "production"]
  
  - name: "LargeQueueSize-TeamsNotifications"
    condition: "teams_notification_queue_size > 100"
    duration: "5m"
    severity: "warning"
    environments: ["production"]
```

## 6. Rotação de Secrets e URLs de Webhook

### Cronograma Recomendado
| Ambiente     | Frequência de Rotação | Processo                          |
|--------------|-----------------------|-----------------------------------|
| Development  | Mensal                | Manual simples                    |
| Staging      | Trimestral            | Automatizado com CI/CD            |
| Production   | Trimestral            | Automatizado com janela de manutenção |

### Processo de Rotação em Produção
```
1. [D-7] Gerar novas URLs de webhook nos canais do Teams
2. [D-3] Adicionar novas URLs ao secrets management (dual-write)
3. [D-1] Testar novas URLs em ambiente de staging
4. [D] Durante janela de manutenção:
   - Atualizar configuração da aplicação
   - Validar envios com novas URLs
   - Monitorar por 30 minutos
5. [D+1] Remover URLs antigas do secrets management
6. [D+7] Remover conectores antigos do Teams (se não usados)
```

## 7. Backup e Recuperação

### Estratégia de Backup
- Versionar arquivos de configuração em repositório Git
- Exportar configurações de conectores periodicamente
- Documentar processos de setup de canais

### Plano de Recuperação
```markdown
## Recovery Plan - Teams Integration

### Perda de Webhook URL
1. Acessar canal no Teams
2. Reconfigurar conector Incoming Webhook
3. Atualizar URL em secrets management
4. Reiniciar serviço de notificações

### Perda de Canal/Time
1. Recriar estrutura de times e canais
2. Reconfigurar conectores
3. Reatribuir permissões
4. Validar envios

### Falha Massiva no Serviço Teams
1. Acionar modo fallback (email/notificações alternativas)
2. Monitorar status do serviço Microsoft
3. Comunicar stakeholders
4. Manter fila de notificações para reprocessamento
```

## 8. Testes de Configuração

### Suite de Testes por Ambiente
```python
# tests/environment_config_tests.py

import pytest
import yaml
from pathlib import Path

class TestEnvironmentConfiguration:
    
    @pytest.mark.parametrize("environment", ["development", "staging", "production"])
    def test_required_channels_present(self, environment):
        """Verifica se todos os canais obrigatórios estão configurados"""
        config_path = Path(f"config/environments/{environment}/teams-webhooks.yaml")
        with open(config_path) as f:
            config = yaml.safe_load(f)
        
        required_channels = ["alerts", "approvals", "general"]
        configured_channels = config["notifications"]["teams"]["channels"].keys()
        
        for channel in required_channels:
            assert channel in configured_channels, f"Canal {channel} não encontrado em {environment}"
    
    def test_prod_no_dry_run(self):
        """Verifica que produção não está em modo dry-run"""
        config_path = Path("config/environments/production/teams-webhooks.yaml")
        with open(config_path) as f:
            config = yaml.safe_load(f)
        
        assert config["notifications"]["teams"]["dry_run"] == False, \
            "Ambiente de produção não deve estar em modo dry-run"
    
    @pytest.mark.parametrize("environment", ["staging", "production"])
    def test_secure_config_in_non_dev(self, environment):
        """Verifica configurações de segurança fora do ambiente de desenvolvimento"""
        config_path = Path(f"config/environments/{environment}/teams-settings.yaml")
        with open(config_path) as f:
            config = yaml.safe_load(f)
        
        assert config["teams_integration"]["logging_level"] in ["WARN", "ERROR"], \
            "Nível de log deve ser restritivo em ambientes não-dev"
```

## 9. Documentação de Setup Inicial

### Passo a Passo para Novo Ambiente

#### 1. Criar Estrutura de Equipes e Canais
```
1. Criar nova equipe: "Liquibase-Governance-{ENVIRONMENT}"
2. Criar canais:
   - alerts-{environment}
   - approvals-{environment}
   - notifications-{environment}
   - audit-{environment}
3. Configurar permissões apropriadas para cada canal
```

#### 2. Configurar Webhooks
```
1. Para cada canal:
   a. Acessar "Conectores" nas opções do canal
   b. Adicionar "Incoming Webhook"
   c. Definir nome: "Liquibase Governance Notifier"
   d. Adicionar imagem (opcional)
   e. Copiar URL gerada
2. Armazenar URLs em sistema de secrets apropriado
```

#### 3. Validar Configuração
```
1. Testar envio para cada canal
2. Verificar formatação de mensagens
3. Confirmar permissões e visibilidade
4. Documentar setup no runbook do ambiente
```

Este guia completo cobre todas as considerações essenciais para configurar e gerenciar a integração com Microsoft Teams em diferentes ambientes, garantindo segurança, confiabilidade e facilidade de manutenção.