# Políticas de Timeout para Aprovações Pendentes

## 1. Visão Geral

Este documento estabelece as políticas de timeout para aprovações pendentes de exceções `linter-ignore-rule` no pipeline Jenkins. As políticas são projetadas para equilibrar a necessidade de governança rigorosa com a eficiência do processo de CI/CD.

## 2. Políticas de Timeout por Severidade

### 2.1 Timeout Base por Severidade
| Severidade | Timeout Base | Descrição |
|------------|--------------|-----------|
| Crítica | 48 horas | Exceções que representam riscos máximos |
| Alta | 24 horas | Exceções de alto impacto |
| Média | 6 horas | Exceções de impacto moderado |
| Baixa | 1 hora | Exceções de baixo impacto |

### 2.2 Ajustes por Ambiente
Os timeouts são ajustados com base no ambiente de implantação:

#### 2.2.1 Ambiente de Produção (+50%)
- Crítica: 72 horas
- Alta: 36 horas
- Média: 9 horas
- Baixa: 1.5 horas

#### 2.2.2 Ambiente de Homologação (Base)
- Crítica: 48 horas
- Alta: 24 horas
- Média: 6 horas
- Baixa: 1 hora

#### 2.2.3 Ambiente de Desenvolvimento (-50%)
- Crítica: 24 horas
- Alta: 12 horas
- Média: 3 horas
- Baixa: 30 minutos

### 2.3 Ajustes por Horário de Trabalho
Timeouts são pausados fora do horário comercial:

#### 2.3.1 Horário Comercial
- Segunda a Sexta: 09:00 - 18:00 (timezone local)
- Feriados são excluídos do cálculo

#### 2.3.2 Horário Não Comercial
- Finais de semana
- Fora do horário 09:00 - 18:00
- Feriados

Durante períodos não comerciais, o contador de timeout é pausado e retomado no próximo horário comercial.

## 3. Implementação Técnica

### 3.1 Serviço de Cálculo de Timeout
```python
# /approval_services/timeout_calculator.py
from datetime import datetime, timedelta
from typing import Dict, List
import holidays

class TimeoutCalculator:
    def __init__(self, config: Dict):
        self.config = config
        self.holidays = self._load_holidays()
    
    def calculate_timeout_deadline(self, severity: str, environment: str, 
                                 submission_time: datetime = None) -> datetime:
        """
        Calcula o deadline para aprovação baseado em severidade e ambiente
        
        Args:
            severity: Nível de severidade da exceção
            environment: Ambiente de implantação
            submission_time: Hora de submissão (default: agora)
            
        Returns:
            Datetime com o deadline de aprovação
        """
        if submission_time is None:
            submission_time = datetime.now()
        
        # Obter timeout base
        base_timeout_hours = self._get_base_timeout(severity)
        
        # Ajustar por ambiente
        adjusted_timeout_hours = self._adjust_timeout_for_environment(
            base_timeout_hours, environment
        )
        
        # Calcular deadline considerando horário comercial
        deadline = self._calculate_business_deadline(
            submission_time, adjusted_timeout_hours
        )
        
        return deadline
    
    def _get_base_timeout(self, severity: str) -> int:
        """Obtém timeout base por severidade"""
        base_timeouts = self.config.get('base_timeouts', {
            'critical': 48,
            'high': 24,
            'medium': 6,
            'low': 1
        })
        return base_timeouts.get(severity, 24)  # Default 24 horas
    
    def _adjust_timeout_for_environment(self, base_timeout: int, environment: str) -> int:
        """Ajusta timeout baseado no ambiente"""
        environment_multipliers = self.config.get('environment_multipliers', {
            'production': 1.5,
            'staging': 1.0,
            'development': 0.5
        })
        
        multiplier = environment_multipliers.get(environment, 1.0)
        return int(base_timeout * multiplier)
    
    def _calculate_business_deadline(self, start_time: datetime, 
                                   timeout_hours: int) -> datetime:
        """
        Calcula deadline considerando apenas horário comercial
        
        Args:
            start_time: Hora de início
            timeout_hours: Horas úteis para timeout
            
        Returns:
            Datetime com o deadline ajustado
        """
        business_start_hour = self.config.get('business_hours', {}).get('start', 9)
        business_end_hour = self.config.get('business_hours', {}).get('end', 18)
        
        current_time = start_time
        remaining_hours = timeout_hours
        
        while remaining_hours > 0:
            # Pular fins de semana e feriados
            while self._is_non_business_day(current_time):
                current_time = self._next_business_day(current_time)
                current_time = current_time.replace(hour=business_start_hour, minute=0, second=0)
            
            # Calcular horas disponíveis no dia atual
            day_end = current_time.replace(hour=business_end_hour, minute=0, second=0)
            if current_time.hour < business_start_hour:
                current_time = current_time.replace(hour=business_start_hour, minute=0, second=0)
                day_end = current_time.replace(hour=business_end_hour, minute=0, second=0)
            
            available_today = max(0, (day_end - current_time).total_seconds() / 3600)
            
            if remaining_hours <= available_today:
                # Deadline cai hoje
                deadline = current_time + timedelta(hours=remaining_hours)
                return deadline
            else:
                # Consumir todas as horas disponíveis hoje e passar para o próximo dia
                remaining_hours -= available_today
                current_time = self._next_business_day(current_time)
                current_time = current_time.replace(hour=business_start_hour, minute=0, second=0)
        
        return current_time
    
    def _is_non_business_day(self, date_time: datetime) -> bool:
        """Verifica se é dia não comercial (fim de semana ou feriado)"""
        # Fins de semana
        if date_time.weekday() >= 5:  # 5 = Saturday, 6 = Sunday
            return True
        
        # Feriados
        if date_time.date() in self.holidays:
            return True
        
        return False
    
    def _next_business_day(self, date_time: datetime) -> datetime:
        """Obtém o próximo dia útil"""
        next_day = date_time + timedelta(days=1)
        while self._is_non_business_day(next_day):
            next_day += timedelta(days=1)
        return next_day
    
    def _load_holidays(self) -> List:
        """Carrega lista de feriados"""
        country = self.config.get('country', 'BR')
        year = datetime.now().year
        
        try:
            country_holidays = holidays.country_holidays(country, years=year)
            return list(country_holidays.keys())
        except:
            # Fallback para lista vazia se não conseguir carregar feriados
            return []

# Configuração de exemplo
TIMEOUT_CONFIG = {
    'base_timeouts': {
        'critical': 48,
        'high': 24,
        'medium': 6,
        'low': 1
    },
    'environment_multipliers': {
        'production': 1.5,
        'staging': 1.0,
        'development': 0.5
    },
    'business_hours': {
        'start': 9,
        'end': 18
    },
    'country': 'BR'
}
```

### 3.2 Monitoramento de Timeout no Jenkins
```groovy
// /ci-scripts/jenkins/steps/timeout_monitoring.groovy
def setupTimeoutMonitoring(exceptionData) {
    script {
        // Calcular deadline usando o serviço de governança
        def deadlineJson = sh(
            script: """
                liquibase-governance calculate-timeout \
                    --severity ${exceptionData.severity} \
                    --environment ${ENVIRONMENT} \
                    --submission-time "$(date -u +%Y-%m-%dT%H:%M:%SZ)"
            """,
            returnStdout: true
        ).trim()
        
        def deadlineData = readJSON text: deadlineJson
        def deadlineIso = deadlineData.deadline
        
        // Converter para timestamp para cálculos
        def deadlineTimestamp = sh(
            script: "date -d '${deadlineIso}' +%s",
            returnStdout: true
        ).trim().toLong()
        
        // Armazenar deadline como variável de ambiente
        env.APPROVAL_DEADLINE = deadlineIso
        env.APPROVAL_DEADLINE_TIMESTAMP = deadlineTimestamp.toString()
        
        // Calcular duração do timeout em horas
        def nowTimestamp = sh(script: "date +%s", returnStdout: true).trim().toLong()
        def timeoutDurationHours = (deadlineTimestamp - nowTimestamp) / 3600
        
        echo "Deadline para aprovação: ${deadlineIso} (${timeoutDurationHours.round(1)} horas)"
        
        // Configurar timeout do Jenkins com margem de segurança
        timeout(time: Math.max(1, timeoutDurationHours.toInteger() + 1), unit: 'HOURS') {
            // Registrar deadline no sistema de governança
            sh """
                liquibase-governance register-approval-deadline \
                    --exception-id ${exceptionData.id} \
                    --deadline '${deadlineIso}' \
                    --build-number ${BUILD_NUMBER}
            """
            
            // Continuar com o estágio de aprovação
            waitForApprovalWithDeadline(exceptionData, deadlineIso)
        }
    }
}

def waitForApprovalWithDeadline(exceptionData, deadlineIso) {
    script {
        // Enviar notificação inicial com deadline
        sendApprovalNotificationWithDeadline(exceptionData, deadlineIso)
        
        // Configurar lembretes periódicos
        setupApprovalReminders(exceptionData, deadlineIso)
        
        // Aguardar input de aprovação
        def approvalInput = input(
            message: createApprovalMessageWithDeadline(exceptionData, deadlineIso),
            submitter: getRequiredApprovers(exceptionData.severity),
            parameters: [
                choice(
                    choices: ['Aprovar', 'Rejeitar'],
                    description: 'Decisão sobre a exceção encontrada',
                    name: 'DECISION'
                ),
                text(
                    defaultValue: '',
                    description: 'Justificativa detalhada para a decisão',
                    name: 'JUSTIFICATION'
                )
            ]
        )
        
        // Processar decisão
        processApprovalDecision(approvalInput, exceptionData)
    }
}

def createApprovalMessageWithDeadline(exceptionData, deadlineIso) {
    def deadlineFormatted = sh(
        script: "date -d '${deadlineIso}' '+%d/%m/%Y %H:%M'", 
        returnStdout: true
    ).trim()
    
    return """
        🚨 Exceção ${exceptionData.severity} encontrada requer aprovação
        
        Changeset: ${exceptionData.changeset_author}:${exceptionData.changeset_id}
        Regra: ${exceptionData.rule_name}
        Arquivo: ${exceptionData.file_name}
        
        ⏰ Deadline para aprovação: ${deadlineFormatted}
        
        Por favor, revise cuidadosamente e tome uma decisão.
    """
}
```

## 4. Sistema de Lembretes

### 4.1 Política de Lembretes Progressivos
Lembretes são enviados em intervalos específicos antes do deadline:

#### 4.1.1 Para Exceções Críticas
- 24 horas antes
- 6 horas antes
- 1 hora antes
- 15 minutos antes
- A cada 2 horas após o deadline ser ultrapassado

#### 4.1.2 Para Exceções de Alta Severidade
- 12 horas antes
- 3 horas antes
- 30 minutos antes
- A cada 4 horas após o deadline ser ultrapassado

#### 4.1.3 Para Exceções de Média Severidade
- 3 horas antes
- 1 hora antes
- 15 minutos antes
- A cada 8 horas após o deadline ser ultrapassado

#### 4.1.4 Para Exceções de Baixa Severidade
- 30 minutos antes
- 15 minutos antes
- A cada 12 horas após o deadline ser ultrapassado

### 4.2 Implementação de Lembretes
```python
# /notifications/reminder_scheduler.py
from datetime import datetime, timedelta
from typing import List, Dict, Any
import logging

class ReminderScheduler:
    def __init__(self, notification_service, timeout_calculator):
        self.notification_service = notification_service
        self.timeout_calculator = timeout_calculator
        self.logger = logging.getLogger(__name__)
    
    def schedule_reminders(self, exception_data: Dict[str, Any], deadline: datetime):
        """
        Agenda lembretes para uma exceção com base em sua severidade
        
        Args:
            exception_data: Dados da exceção
            deadline: Deadline para aprovação
        """
        severity = exception_data.get('calculated_severity', 'medium')
        reminder_times = self._calculate_reminder_times(severity, deadline)
        
        for reminder_time in reminder_times:
            self._schedule_single_reminder(exception_data, reminder_time, deadline)
    
    def _calculate_reminder_times(self, severity: str, deadline: datetime) -> List[datetime]:
        """
        Calcula horários para envio de lembretes baseado na severidade
        
        Args:
            severity: Severidade da exceção
            deadline: Deadline para aprovação
            
        Returns:
            Lista de horários para lembretes
        """
        now = datetime.now()
        reminder_intervals = self._get_reminder_intervals(severity)
        reminder_times = []
        
        for interval in reminder_intervals:
            if isinstance(interval, dict) and 'before_deadline' in interval:
                # Lembrete antes do deadline
                reminder_time = deadline - timedelta(**interval['before_deadline'])
                if reminder_time > now:
                    reminder_times.append(reminder_time)
            elif isinstance(interval, dict) and 'after_start' in interval:
                # Lembrete recorrente após início
                # Esta lógica seria implementada em um serviço de scheduling real
                pass
        
        return sorted(reminder_times)
    
    def _get_reminder_intervals(self, severity: str) -> List[Dict]:
        """
        Obtém intervalos de lembrete baseados na severidade
        
        Args:
            severity: Severidade da exceção
            
        Returns:
            Lista de configurações de intervalos
        """
        intervals = {
            'critical': [
                {'before_deadline': {'hours': 24}},
                {'before_deadline': {'hours': 6}},
                {'before_deadline': {'hours': 1}},
                {'before_deadline': {'minutes': 15}}
            ],
            'high': [
                {'before_deadline': {'hours': 12}},
                {'before_deadline': {'hours': 3}},
                {'before_deadline': {'minutes': 30}}
            ],
            'medium': [
                {'before_deadline': {'hours': 3}},
                {'before_deadline': {'hours': 1}},
                {'before_deadline': {'minutes': 15}}
            ],
            'low': [
                {'before_deadline': {'minutes': 30}},
                {'before_deadline': {'minutes': 15}}
            ]
        }
        
        return intervals.get(severity, intervals['medium'])
    
    def _schedule_single_reminder(self, exception_data: Dict[str, Any], 
                                reminder_time: datetime, deadline: datetime):
        """
        Agenda um único lembrete (em implementação real, isso usaria um job scheduler)
        
        Args:
            exception_data: Dados da exceção
            reminder_time: Hora para enviar lembrete
            deadline: Deadline para aprovação
        """
        # Em uma implementação real, isto criaria um job agendado
        # Para esta implementação, registramos o lembrete planejado
        self.logger.info(f"Lembrete agendado para {reminder_time} para exceção {exception_data.get('id')}")
        
        # Em produção, isso poderia usar:
        # - Celery Beat para scheduling distribuído
        # - APScheduler para scheduling em processo
        # - Cloud Scheduler services (AWS EventBridge, Google Cloud Scheduler, etc.)

# Exemplo de uso
def setup_reminders_for_exception(exception_data: Dict, deadline: datetime):
    """Configura lembretes para uma exceção"""
    scheduler = ReminderScheduler(notification_service, timeout_calculator)
    scheduler.schedule_reminders(exception_data, deadline)
```

## 5. Tratamento de Timeout Expirado

### 5.1 Ações Automáticas Quando Timeout Expira
Quando o prazo para aprovação expira, o sistema executa ações automáticas:

#### 5.1.1 Para Exceções Críticas e Altas
1. Pipeline é automaticamente **interrompido**
2. Notificação é enviada aos stakeholders
3. Ticket é criado no sistema de incidentes
4. Equipe de emergência é acionada

#### 5.1.2 Para Exceções Médias
1. Pipeline entra em estado de **quarentena**
2. Aprovação é escalada para nível superior
3. Notificação prioritária é enviada
4. Processo manual de aprovação é iniciado

#### 5.1.3 Para Exceções Baixas
1. Exceção é **automaticamente rejeitada**
2. Pipeline continua com a regra aplicada
3. Notificação informativa é enviada
4. Mudança é revertida automaticamente

### 5.2 Implementação de Tratamento de Timeout
```groovy
// /ci-scripts/jenkins/steps/timeout_handling.groovy
def handleExpiredTimeout(exceptionData, severity) {
    script {
        def currentTime = sh(script: "date -u +%Y-%m-%dT%H:%M:%SZ", returnStdout: true).trim()
        
        // Registrar timeout expirado
        sh """
            liquibase-governance record-timeout-expiration \\
                --exception-id ${exceptionData.id} \\
                --severity ${severity} \\
                --expiration-time '${currentTime}' \\
                --build-number ${BUILD_NUMBER}
        """
        
        // Tomar ação baseada na severidade
        switch(severity) {
            case 'critical':
            case 'high':
                handleCriticalTimeout(exceptionData)
                break
            case 'medium':
                handleMediumTimeout(exceptionData)
                break
            case 'low':
                handleLowTimeout(exceptionData)
                break
            default:
                handleMediumTimeout(exceptionData)
        }
    }
}

def handleCriticalTimeout(exceptionData) {
    script {
        echo "⏱️ Timeout expirado para exceção crítica!"
        
        // Interromper pipeline
        def interruptionReason = "Timeout expirado para aprovação de exceção crítica: ${exceptionData.rule_name}"
        
        // Enviar notificações de emergência
        sendEmergencyNotifications(exceptionData, interruptionReason)
        
        // Criar ticket de incidente
        createIncidentTicket(exceptionData, interruptionReason)
        
        // Interromper pipeline
        error(interruptionReason)
    }
}

def handleMediumTimeout(exceptionData) {
    script {
        echo "⏱️ Timeout expirado para exceção média. Escalando aprovação..."
        
        // Escalar para nível superior
        def escalationMessage = """
            Aprovação escalada devido a timeout expirado para exceção média:
            Changeset: ${exceptionData.changeset_author}:${exceptionData.changeset_id}
            Regra: ${exceptionData.rule_name}
        """
        
        // Enviar notificação para grupo de escalation
        slackSend(
            channel: '#ad-management',
            message: "🚨 ${escalationMessage}"
        )
        
        // Registrar escalonamento
        sh """
            liquibase-governance record-escalation \\
                --exception-id ${exceptionData.id} \\
                --reason "Timeout expirado" \\
                --escalated-to "MANAGEMENT-GROUP" \\
                --build-number ${BUILD_NUMBER}
        """
        
        // Solicitar aprovação de nível superior
        def escalatedApproval = input(
            message: "Aprovação escalada - ${escalationMessage}",
            submitter: 'MANAGEMENT-GROUP',
            parameters: [
                choice(
                    choices: ['Aprovar', 'Rejeitar'],
                    description: 'Decisão de aprovação escalada',
                    name: 'DECISION'
                ),
                text(
                    defaultValue: '',
                    description: 'Justificativa para decisão escalada',
                    name: 'JUSTIFICATION'
                )
            ]
        )
        
        // Processar decisão escalada
        processEscalatedDecision(escalatedApproval, exceptionData)
    }
}

def handleLowTimeout(exceptionData) {
    script {
        echo "⏱️ Timeout expirado para exceção baixa. Rejeitando automaticamente..."
        
        // Registrar rejeição automática
        sh """
            liquibase-governance record-automatic-rejection \\
                --exception-id ${exceptionData.id} \\
                --reason "Timeout expirado" \\
                --build-number ${BUILD_NUMBER}
        """
        
        // Enviar notificação informativa
        sendTimeoutNotification(exceptionData, 'low')
        
        // Interromper pipeline com mensagem específica
        error("Exceção de baixa severidade rejeitada automaticamente devido a timeout expirado")
    }
}
```

## 6. Configuração de Políticas

### 6.1 Arquivo de Configuração de Timeout
```yaml
# /config/timeout_policies.yaml
timeout_policies:
  # Timeout base por severidade (em horas)
  base_timeouts:
    critical: 48
    high: 24
    medium: 6
    low: 1
  
  # Multiplicadores por ambiente
  environment_multipliers:
    production: 1.5
    staging: 1.0
    development: 0.5
  
  # Horário comercial (24h format)
  business_hours:
    start: 9
    end: 18
  
  # País para cálculo de feriados
  country: "BR"
  
  # Políticas de lembrete
  reminder_policies:
    critical:
      before_deadline:
        - hours: 24
        - hours: 6
        - hours: 1
        - minutes: 15
      after_expiration:
        interval_hours: 2
        max_reminders: 12
    
    high:
      before_deadline:
        - hours: 12
        - hours: 3
        - minutes: 30
      after_expiration:
        interval_hours: 4
        max_reminders: 6
    
    medium:
      before_deadline:
        - hours: 3
        - hours: 1
        - minutes: 15
      after_expiration:
        interval_hours: 8
        max_reminders: 3
    
    low:
      before_deadline:
        - minutes: 30
        - minutes: 15
      after_expiration:
        interval_hours: 12
        max_reminders: 2
  
  # Ações para timeout expirado
  timeout_actions:
    critical:
      action: "terminate_pipeline"
      notification_level: "emergency"
      create_incident_ticket: true
    
    high:
      action: "terminate_pipeline"
      notification_level: "high_priority"
      create_incident_ticket: true
    
    medium:
      action: "escalate_approval"
      notification_level: "normal"
      escalate_to: "MANAGEMENT-GROUP"
    
    low:
      action: "auto_reject"
      notification_level: "informational"
      auto_revert_changes: true
  
  # Configurações de tolerância
  tolerance:
    # Margem de segurança para cálculos (minutos)
    safety_margin_minutes: 30
    
    # Verificação de deadline (minutos)
    deadline_check_interval: 5
    
    # Tolerância para pequenos delays
    grace_period_minutes: 15

# Holidays configuration would typically be handled separately
# or loaded from a dedicated service
```

## 7. Monitoramento e Métricas

### 7.1 Métricas de Timeout
```python
# /monitoring/timeout_metrics.py
from prometheus_client import Counter, Histogram, Gauge

class TimeoutMetrics:
    def __init__(self):
        self.timeouts_triggered = Counter(
            'approval_timeouts_triggered_total',
            'Total de timeouts de aprovação disparados',
            ['severity', 'environment', 'action_taken']
        )
        
        self.time_to_approval = Histogram(
            'time_to_approval_seconds',
            'Tempo até a aprovação/rejeição',
            ['severity', 'environment'],
            buckets=[60, 300, 600, 1800, 3600, 7200, 14400, 28800, 86400, float('inf')]
        )
        
        self.approvals_before_deadline = Counter(
            'approvals_before_deadline_total',
            'Total de aprovações feitas antes do deadline',
            ['severity']
        )
        
        self.approvals_after_deadline = Counter(
            'approvals_after_deadline_total',
            'Total de aprovações feitas após o deadline',
            ['severity']
        )
        
        self.average_approval_time = Gauge(
            'average_approval_time_hours',
            'Tempo médio de aprovação em horas',
            ['severity']
        )
    
    def record_timeout_triggered(self, severity: str, environment: str, action: str):
        """Registra timeout disparado"""
        self.timeouts_triggered.labels(
            severity=severity,
            environment=environment,
            action_taken=action
        ).inc()
    
    def record_time_to_approval(self, duration_seconds: float, severity: str, environment: str):
        """Registra tempo até aprovação"""
        self.time_to_approval.labels(
            severity=severity,
            environment=environment
        ).observe(duration_seconds)
    
    def record_approval_timing(self, approved_before_deadline: bool, severity: str):
        """Registra se aprovação foi antes ou depois do deadline"""
        if approved_before_deadline:
            self.approvals_before_deadline.labels(severity=severity).inc()
        else:
            self.approvals_after_deadline.labels(severity=severity).inc()
    
    def update_average_approval_time(self, avg_hours: float, severity: str):
        """Atualiza média de tempo de aprovação"""
        self.average_approval_time.labels(severity=severity).set(avg_hours)
```

### 7.2 Dashboard de Timeout
```
DASHBOARD: Métricas de Timeout de Aprovação

┌─────────────────────────────────────────────────────────────┐
| STATUS EM TEMPO REAL                                       |
├─────────────────────────────────────────────────────────────┤
| APPROVALS PENDENTES    │ APPROVALS EXPIRADOS  │ TIMEOUT HOJE |
| 12                     │ 3                    │ 2            |
├─────────────────────────────────────────────────────────────┤
| TEMPO MÉDIO DE APROVAÇÃO (horas)                           |
| ┌─────────────────────────────────────────────────────────┐ |
| | Crítica: 2.1  │ Alta: 1.8  │ Média: 0.9  │ Baixa: 0.3    | |
| └─────────────────────────────────────────────────────────┘ |
├─────────────────────────────────────────────────────────────┤
| APROVAÇÕES POR STATUS                                      |
| ┌─────────────────────────────────────────────────────────┐ |
| | 🟢 Antes Deadline: 85%                                  | |
| | 🔴 Após Deadline: 10%                                   | |
| | ⏱️ Timeout: 5%                                          | |
| └─────────────────────────────────────────────────────────┘ |
├─────────────────────────────────────────────────────────────┤
| ALERTAS ATIVOS                                             |
| ┌─────────────────────────────────────────────────────────┐ |
| | 🚨 2 Exceções Críticas Expirando em < 6h                | |
| | ⚠️  5 Exceções Altas Expirando em < 12h                 | |
| | ℹ️  12 Exceções Médias Expirando em < 24h               | |
| └─────────────────────────────────────────────────────────┘ |
└─────────────────────────────────────────────────────────────┘
```

## 8. Testes e Validação

### 8.1 Testes de Unidade para Cálculo de Timeout
```python
# /tests/unit/test_timeout_calculator.py
import unittest
from datetime import datetime, timedelta
from approval_services.timeout_calculator import TimeoutCalculator

class TestTimeoutCalculator(unittest.TestCase):
    def setUp(self):
        self.config = {
            'base_timeouts': {
                'critical': 48,
                'high': 24,
                'medium': 6,
                'low': 1
            },
            'environment_multipliers': {
                'production': 1.5,
                'staging': 1.0,
                'development': 0.5
            },
            'business_hours': {
                'start': 9,
                'end': 18
            },
            'country': 'BR'
        }
        self.calculator = TimeoutCalculator(self.config)
    
    def test_basic_timeout_calculation(self):
        """Teste de cálculo básico de timeout"""
        submission_time = datetime(2023, 1, 2, 10, 0)  # Segunda-feira 10:00
        
        # Testar timeout para severidade crítica em staging
        deadline = self.calculator.calculate_timeout_deadline(
            'critical', 'staging', submission_time
        )
        
        # Deve ser 48 horas após, mas considerando horário comercial
        expected_deadline = datetime(2023, 1, 4, 10, 0)  # Quarta-feira 10:00
        self.assertEqual(deadline.date(), expected_deadline.date())
    
    def test_environment_multiplier(self):
        """Teste de multiplicador por ambiente"""
        submission_time = datetime(2023, 1, 2, 10, 0)
        
        # Production (1.5x)
        prod_deadline = self.calculator.calculate_timeout_deadline(
            'medium', 'production', submission_time
        )
        
        # Development (0.5x)
        dev_deadline = self.calculator.calculate_timeout_deadline(
            'medium', 'development', submission_time
        )
        
        # Timeout em produção deve ser maior que em desenvolvimento
        self.assertGreater(prod_deadline, dev_deadline)
    
    def test_business_hours_consideration(self):
        """Teste de consideração de horário comercial"""
        # Submissão sexta-feira às 17:00 (próximo ao fim do expediente)
        submission_time = datetime(2023, 1, 6, 17, 0)
        
        deadline = self.calculator.calculate_timeout_deadline(
            'medium', 'staging', submission_time
        )
        
        # Deadline deve ser na segunda-feira, não no sábado
        self.assertGreaterEqual(deadline.weekday(), 0)  # Segunda-feira ou depois
        self.assertLess(deadline.weekday(), 5)  # Antes do sábado

if __name__ == '__main__':
    unittest.main()
```

Estas políticas de timeout fornecem um framework completo para gerenciar o tempo de aprovação de exceções no pipeline de CI/CD, garantindo tanto a governança rigorosa quanto a eficiência operacional.