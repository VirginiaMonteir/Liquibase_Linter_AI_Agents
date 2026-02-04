# Mecanismo de Notificação de Exceções Críticas aos Administradores

## 1. Visão Geral

Este documento detalha o mecanismo de notificação para alertar os administradores de dados (AD-GROUP) sobre a detecção de exceções `linter-ignore-rule` de alta e crítica severidade. O sistema garante que as partes interessadas sejam informadas imediatamente quando exceções potencialmente perigosas forem identificadas nos changesets do Liquibase.

## 2. Tipos de Exceções que Requerem Notificação

### 2.1 Exceções Críticas (Critical)
- `linter-ignore-all` - Ignora todas as regras de validação
- Regras de segurança crítica (ex: `no-drop-table`, `no-delete-all`)
- Regras de governança restrita (ex: `forbid-sale-schema`, `forbid-production-access`)

### 2.2 Exceções de Alta Severidade (High)
- Regras de segurança importantes (ex: `no-grant-on-public`, `no-truncate`)
- Regras de desempenho crítico (ex: `no-unbounded-update`, `no-unbounded-delete`)
- Regras de governança sensível (ex: `has-author`, `has-id`) em ambiente de produção

## 3. Canais de Notificação

### 3.1 Email
Notificação primária para garantir cobertura mesmo quando outros canais não estão disponíveis.

#### 3.1.1 Template de Email
```html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Alerta de Exceção Crítica - Liquibase Governance</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 0; padding: 20px; background-color: #f5f5f5; }
        .container { max-width: 800px; margin: 0 auto; background-color: white; border-radius: 8px; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .header { background-color: #dc3545; color: white; padding: 20px; border-radius: 8px 8px 0 0; }
        .content { padding: 20px; }
        .exception-card { border: 1px solid #ddd; border-radius: 5px; margin: 15px 0; padding: 15px; background-color: #fff; }
        .severity-critical { border-left: 5px solid #dc3545; }
        .severity-high { border-left: 5px solid #fd7e14; }
        .button { 
            background-color: #007bff; 
            color: white; 
            padding: 12px 24px; 
            text-decoration: none; 
            border-radius: 5px;
            display: inline-block;
            margin: 10px 5px;
            font-weight: bold;
        }
        .button-secondary { background-color: #6c757d; }
        .footer { padding: 20px; background-color: #f8f9fa; border-radius: 0 0 8px 8px; font-size: 12px; color: #6c757d; }
        .highlight { background-color: #fff3cd; padding: 2px 4px; border-radius: 3px; }
    </style>
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>🚨 Alerta de Exceção Crítica Detectada</h1>
            <p>Uma ou mais exceções de alta severidade foram encontradas e requerem sua atenção imediata</p>
        </div>
        
        <div class="content">
            <h2>Resumo do Alerta</h2>
            <p><strong>Total de Exceções:</strong> {{ exception_count }}</p>
            <p><strong>Build Number:</strong> {{ build_number }}</p>
            <p><strong>Ambiente:</strong> {{ environment }}</p>
            <p><strong>Data/Hora:</strong> {{ timestamp }}</p>
            
            <h2>Exceções Detectadas</h2>
            {% for exception in exceptions %}
            <div class="exception-card severity-{{ exception.severity }}">
                <h3>{{ exception.changeset_author }}:{{ exception.changeset_id }}</h3>
                <p><strong>Regra Ignorada:</strong> <span class="highlight">{{ exception.rule_name }}</span></p>
                <p><strong>Severidade:</strong> {{ exception.severity_label }}</p>
                <p><strong>Arquivo:</strong> {{ exception.file_name }}</p>
                <p><strong>Linha:</strong> {{ exception.line_number }}</p>
                <p><strong>Justificativa do Desenvolvedor:</strong> {{ exception.justification or 'Nenhuma justificativa fornecida' }}</p>
            </div>
            {% endfor %}
            
            <h2>Ações Necessárias</h2>
            <p>Por favor, revise estas exceções e tome as decisões apropriadas:</p>
            <a href="{{ approval_url }}" class="button">Acessar Painel de Aprovações</a>
            <a href="{{ build_url }}" class="button button-secondary">Ver Detalhes do Build</a>
            
            <h2>Próximos Passos</h2>
            <ol>
                <li>Analise cuidadosamente cada exceção listada</li>
                <li>Verifique o contexto do código e a justificativa fornecida</li>
                <li>Aprove ou rejeite cada exceção conforme apropriado</li>
                <li>O pipeline permanecerá pausado até que todas as decisões sejam tomadas</li>
            </ol>
        </div>
        
        <div class="footer">
            <p>Esta é uma notificação automática do sistema de governança do Liquibase.</p>
            <p>Se você não é o destinatário correto, por favor encaminhe esta mensagem para o grupo AD-GROUP.</p>
            <p>Timeout para aprovação: {{ timeout_hours }} horas</p>
        </div>
    </div>
</body>
</html>
```

### 3.2 Slack/Teams
Notificação em tempo real para plataformas de colaboração.

#### 3.2.1 Mensagem Rich para Slack
```json
{
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "🚨 Alerta de Exceção Crítica"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*{{ exception_count }} exceção(ões) crítica(s) detectada(s)* no build *{{ build_number }}* para o ambiente *{{ environment }}*"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Exceções Encontradas:*"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Changeset:*\n{{ exception.changeset_author }}:{{ exception.changeset_id }}"
        },
        {
          "type": "mrkdwn",
          "text": "*Regra:*\n{{ exception.rule_name }}"
        },
        {
          "type": "mrkdwn",
          "text": "*Severidade:*\n{{ exception.severity_label }}"
        },
        {
          "type": "mrkdwn",
          "text": "*Arquivo:*\n{{ exception.file_name }}"
        }
      ]
    },
    {
      "type": "actions",
      "elements": [
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Aprovar/Rejeitar"
          },
          "url": "{{ approval_url }}",
          "style": "primary"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Ver Build"
          },
          "url": "{{ build_url }}",
          "style": "danger"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":hourglass_flowing_sand: *Timeout:* {{ timeout_hours }} horas | :clock1: {{ timestamp }}"
        }
      ]
    }
  ]
}
```

### 3.3 SMS/Push Notifications
Notificação de backup para situações críticas.

#### 3.3.1 Template de SMS
```
ALERTA CRÍTICO: {{ exception_count }} exceção(ões) detectada(s) no build {{ build_number }}. 
Acesse {{ short_approval_url }} para aprovar/rejeitar. Timeout: {{ timeout_hours }}h.
```

## 4. Implementação Técnica

### 4.1 Serviço de Notificação Centralizado
```python
# /notifications/notification_service.py
import smtplib
import requests
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
from typing import List, Dict, Any
import logging

class NotificationService:
    def __init__(self, config: Dict[str, Any]):
        self.config = config
        self.logger = logging.getLogger(__name__)
        self.template_service = TemplateService()
    
    def notify_critical_exceptions(self, exceptions: List[Dict], build_info: Dict):
        """
        Notifica sobre exceções críticas através de múltiplos canais
        
        Args:
            exceptions: Lista de exceções críticas detectadas
            build_info: Informações do build atual
        """
        # Preparar dados para templates
        notification_data = self._prepare_notification_data(exceptions, build_info)
        
        # Determinar canais de notificação com base na severidade
        channels = self._determine_notification_channels(exceptions)
        
        # Enviar notificações
        for channel in channels:
            try:
                if channel == 'email':
                    self._send_email_notification(notification_data)
                elif channel == 'slack':
                    self._send_slack_notification(notification_data)
                elif channel == 'sms':
                    self._send_sms_notification(notification_data)
                elif channel == 'push':
                    self._send_push_notification(notification_data)
                
                self.logger.info(f"Notificação enviada via {channel}")
                
            except Exception as e:
                self.logger.error(f"Falha ao enviar notificação via {channel}: {str(e)}")
    
    def _prepare_notification_data(self, exceptions: List[Dict], build_info: Dict) -> Dict:
        """Prepara dados para os templates de notificação"""
        critical_exceptions = [exc for exc in exceptions if exc['calculated_severity'] in ['high', 'critical']]
        
        # Enriquecer dados das exceções
        enriched_exceptions = []
        for exc in critical_exceptions:
            enriched_exc = exc.copy()
            enriched_exc['severity_label'] = self._get_severity_label(exc['calculated_severity'])
            enriched_exceptions.append(enriched_exc)
        
        return {
            'exception_count': len(critical_exceptions),
            'exceptions': enriched_exceptions,
            'build_number': build_info.get('build_number'),
            'environment': build_info.get('environment', 'unknown'),
            'timestamp': build_info.get('timestamp'),
            'timeout_hours': self._calculate_timeout(critical_exceptions),
            'approval_url': f"{self.config.get('dashboard_url')}/approval/{build_info.get('build_number')}",
            'build_url': build_info.get('build_url'),
            'short_approval_url': f"{self.config.get('short_url_service')}/approval/{build_info.get('build_number')}"
        }
    
    def _determine_notification_channels(self, exceptions: List[Dict]) -> List[str]:
        """Determina quais canais de notificação usar"""
        max_severity = self._get_max_severity(exceptions)
        channels = []
        
        # Sempre enviar email para exceções críticas
        if max_severity == 'critical':
            channels.extend(['email', 'slack'])
        
        # Para alta severidade, enviar email e slack
        elif max_severity == 'high':
            channels.append('email')
            if self.config.get('notifications', {}).get('slack_for_high_severity', False):
                channels.append('slack')
        
        # SMS/Push somente para situações muito críticas
        if self._has_very_critical_exceptions(exceptions):
            channels.extend(['sms', 'push'])
        
        return channels
    
    def _send_email_notification(self, data: Dict):
        """Envia notificação por email"""
        recipients = self.config.get('notifications', {}).get('email_recipients', {}).get('critical_exceptions', [])
        if not recipients:
            self.logger.warning("Nenhum destinatário configurado para emails de exceções críticas")
            return
        
        subject = f"🚨 Alerta de Exceção Crítica - Build {data['build_number']}"
        body_html = self.template_service.render('critical_exception_alert.html', data)
        
        msg = MIMEMultipart('alternative')
        msg['Subject'] = subject
        msg['From'] = self.config.get('notifications', {}).get('email_sender', 'no-reply@governance.com')
        msg['To'] = ', '.join(recipients)
        
        msg.attach(MIMEText(body_html, 'html'))
        
        # Enviar email
        smtp_config = self.config.get('notifications', {}).get('smtp', {})
        with smtplib.SMTP(smtp_config.get('server'), smtp_config.get('port', 587)) as server:
            if smtp_config.get('use_tls', True):
                server.starttls()
            
            if smtp_config.get('username'):
                server.login(smtp_config['username'], smtp_config['password'])
            
            server.send_message(msg)
    
    def _send_slack_notification(self, data: Dict):
        """Envia notificação para Slack"""
        webhook_url = self.config.get('notifications', {}).get('slack_webhook_url')
        if not webhook_url:
            self.logger.warning("Webhook URL do Slack não configurado")
            return
        
        # Preparar payload para Slack
        payload = self._prepare_slack_payload(data)
        
        response = requests.post(webhook_url, json=payload)
        if response.status_code not in [200, 201, 204]:
            raise Exception(f"Falha ao enviar notificação para Slack: {response.text}")
    
    def _send_sms_notification(self, data: Dict):
        """Envia notificação por SMS"""
        sms_config = self.config.get('notifications', {}).get('sms', {})
        if not sms_config.get('enabled', False):
            return
        
        # Preparar mensagem SMS
        message = self.template_service.render_string(
            "ALERTA CRÍTICO: {{ exception_count }} exceção(ões) detectada(s) no build {{ build_number }}. "
            "Acesse {{ short_approval_url }} para aprovar/rejeitar. Timeout: {{ timeout_hours }}h.",
            data
        )
        
        # Enviar via serviço de SMS (Twilio, AWS SNS, etc.)
        # Implementação específica dependerá do provedor escolhido
        self._send_via_sms_provider(sms_config, message)
    
    def _send_push_notification(self, data: Dict):
        """Envia notificação push"""
        push_config = self.config.get('notifications', {}).get('push', {})
        if not push_config.get('enabled', False):
            return
        
        # Preparar notificação push
        push_data = {
            'title': 'Alerta de Exceção Crítica',
            'body': f"{data['exception_count']} exceção(ões) crítica(s) detectada(s)",
            'data': {
                'approval_url': data['approval_url'],
                'build_number': data['build_number']
            }
        }
        
        # Enviar via serviço de push (Firebase, APNs, etc.)
        self._send_via_push_provider(push_config, push_data)
    
    def _prepare_slack_payload(self, data: Dict) -> Dict:
        """Prepara payload para envio via webhook do Slack"""
        # Criar blocos para mensagem rich no Slack
        blocks = [
            {
                "type": "header",
                "text": {
                    "type": "plain_text",
                    "text": "🚨 Alerta de Exceção Crítica"
                }
            },
            {
                "type": "section",
                "text": {
                    "type": "mrkdwn",
                    "text": f"*{data['exception_count']} exceção(ões) crítica(s) detectada(s)* no build *{data['build_number']}* para o ambiente *{data['environment']}*"
                }
            }
        ]
        
        # Adicionar informações das exceções
        blocks.append({"type": "divider"})
        blocks.append({
            "type": "section",
            "text": {
                "type": "mrkdwn",
                "text": "*Exceções Encontradas:*"
            }
        })
        
        for exc in data['exceptions'][:5]:  # Limitar a 5 exceções para não sobrecarregar
            blocks.append({
                "type": "section",
                "fields": [
                    {
                        "type": "mrkdwn",
                        "text": f"*Changeset:*\n{exc['changeset']['author']}:{exc['changeset']['id']}"
                    },
                    {
                        "type": "mrkdwn",
                        "text": f"*Regra:*\n{exc['ignoredRule']['name']}"
                    },
                    {
                        "type": "mrkdwn",
                        "text": f"*Severidade:*\n{exc['severity_label']}"
                    },
                    {
                        "type": "mrkdwn",
                        "text": f"*Arquivo:*\n{exc['fileName']}"
                    }
                ]
            })
        
        # Adicionar botões de ação
        blocks.append({
            "type": "actions",
            "elements": [
                {
                    "type": "button",
                    "text": {
                        "type": "plain_text",
                        "text": "Aprovar/Rejeitar"
                    },
                    "url": data['approval_url'],
                    "style": "primary"
                },
                {
                    "type": "button",
                    "text": {
                        "type": "plain_text",
                        "text": "Ver Build"
                    },
                    "url": data['build_url'],
                    "style": "danger"
                }
            ]
        })
        
        # Adicionar contexto
        blocks.append({
            "type": "context",
            "elements": [
                {
                    "type": "mrkdwn",
                    "text": f":hourglass_flowing_sand: *Timeout:* {data['timeout_hours']} horas | :clock1: {data['timestamp']}"
                }
            ]
        })
        
        return {"blocks": blocks}
    
    def _get_severity_label(self, severity: str) -> str:
        """Converte severidade para label legível"""
        labels = {
            'low': 'Baixa',
            'medium': 'Média',
            'high': 'Alta',
            'critical': 'Crítica'
        }
        return labels.get(severity, severity)
    
    def _get_max_severity(self, exceptions: List[Dict]) -> str:
        """Obtém a severidade máxima entre as exceções"""
        severities = [exc['calculated_severity'] for exc in exceptions]
        severity_order = ['low', 'medium', 'high', 'critical']
        return max(severities, key=lambda s: severity_order.index(s)) if severities else 'low'
    
    def _calculate_timeout(self, exceptions: List[Dict]) -> int:
        """Calcula timeout baseado na severidade máxima"""
        max_severity = self._get_max_severity(exceptions)
        timeout_config = self.config.get('approval_timeouts', {})
        return timeout_config.get(max_severity, 24)  # Default 24 horas
    
    def _has_very_critical_exceptions(self, exceptions: List[Dict]) -> bool:
        """Verifica se há exceções muito críticas"""
        very_critical_rules = ['linter-ignore-all', 'no-drop-table', 'no-delete-all', 'forbid-sale-schema']
        return any(exc['ignoredRule']['name'] in very_critical_rules for exc in exceptions)
```

### 4.2 Integração com o Pipeline Jenkins
```groovy
// /ci-scripts/jenkins/steps/notification_steps.groovy
def sendCriticalExceptionsNotification(exceptionsReport) {
    script {
        // Verificar se há exceções críticas
        def criticalExceptions = exceptionsReport.exceptions.findAll { 
            it.calculatedSeverity in ['high', 'critical'] 
        }
        
        if (criticalExceptions.size() > 0) {
            echo "Enviando notificação de ${criticalExceptions.size()} exceções críticas"
            
            // Preparar dados do build
            def buildInfo = [
                build_number: BUILD_NUMBER,
                environment: ENVIRONMENT ?: 'unknown',
                timestamp: new Date().format("yyyy-MM-dd HH:mm:ss"),
                build_url: BUILD_URL,
                job_name: JOB_NAME
            ]
            
            // Enviar notificação através do sistema de governança
            sh """
                liquibase-governance send-notification \\
                    --type critical-exception-alert \\
                    --build-info '${groovy.json.JsonBuilder(buildInfo).toString()}' \\
                    --exception-count ${criticalExceptions.size()}
            """
            
            // Também enviar notificação direta via Slack se configurado
            if (env.SLACK_WEBHOOK_URL) {
                def slackMessage = """
                    🚨 *Alerta de Exceção Crítica*
                    ${criticalExceptions.size()} exceção(ões) crítica(s) detectada(s) no build *${BUILD_NUMBER}*.
                    
                    Acesse o painel de aprovações para revisar: ${BUILD_URL}
                """
                
                slackSend(
                    channel: '#ad-alerts',
                    message: slackMessage
                )
            }
        } else {
            echo "Nenhuma exceção crítica encontrada para notificação"
        }
    }
}
```

## 5. Agendamento e Frequência de Notificações

### 5.1 Política de Debounce
Para evitar spam de notificações, implementar política de debounce:

```python
# /notifications/debounce_manager.py
import time
import hashlib
from typing import Dict, Any

class NotificationDebounceManager:
    def __init__(self, debounce_window_seconds: int = 300):  # 5 minutos padrão
        self.debounce_window = debounce_window_seconds
        self.sent_notifications = {}  # cache em memória, em produção usar Redis/DB
    
    def should_send_notification(self, notification_data: Dict[str, Any]) -> bool:
        """
        Determina se uma notificação deve ser enviada baseado na política de debounce
        
        Args:
            notification_data: Dados da notificação
            
        Returns:
            True se deve enviar, False caso contrário
        """
        # Criar chave única para esta notificação
        notification_key = self._generate_notification_key(notification_data)
        
        current_time = time.time()
        last_sent_time = self.sent_notifications.get(notification_key, 0)
        
        # Verificar se está dentro da janela de debounce
        if current_time - last_sent_time < self.debounce_window:
            return False
        
        # Atualizar timestamp da última notificação
        self.sent_notifications[notification_key] = current_time
        return True
    
    def _generate_notification_key(self, notification_data: Dict[str, Any]) -> str:
        """Gera chave única para identificar uma notificação"""
        # Criar string identificadora baseada nos dados relevantes
        key_parts = [
            str(notification_data.get('build_number', '')),
            str(notification_data.get('environment', '')),
            str(len(notification_data.get('exceptions', [])))
        ]
        
        # Adicionar IDs das exceções para maior granularidade
        exception_ids = [str(exc.get('id', '')) for exc in notification_data.get('exceptions', [])]
        key_parts.extend(sorted(exception_ids))
        
        # Gerar hash para a chave
        key_string = '|'.join(key_parts)
        return hashlib.md5(key_string.encode()).hexdigest()
    
    def reset_debounce(self, notification_key: str):
        """Remove um registro de debounce (para forçar reenvio)"""
        if notification_key in self.sent_notifications:
            del self.sent_notifications[notification_key]
```

### 5.2 Notificações de Lembrete
Enviar lembretes periódicos para exceções pendentes:

```python
# /notifications/reminder_service.py
import time
from datetime import datetime, timedelta
from typing import List, Dict
import threading

class ReminderService:
    def __init__(self, notification_service, config):
        self.notification_service = notification_service
        self.config = config
        self.pending_approvals = {}  # cache em memória
        self.reminder_thread = None
        self.running = False
    
    def start_reminder_service(self):
        """Inicia o serviço de lembretes em thread separada"""
        if self.running:
            return
        
        self.running = True
        self.reminder_thread = threading.Thread(target=self._reminder_loop, daemon=True)
        self.reminder_thread.start()
        print("Serviço de lembretes iniciado")
    
    def stop_reminder_service(self):
        """Para o serviço de lembretes"""
        self.running = False
        if self.reminder_thread:
            self.reminder_thread.join()
        print("Serviço de lembretes parado")
    
    def register_pending_approval(self, exception_id: str, due_time: datetime, 
                                approvers: List[str], exception_data: Dict):
        """Registra uma exceção pendente de aprovação para lembretes"""
        self.pending_approvals[exception_id] = {
            'due_time': due_time,
            'approvers': approvers,
            'exception_data': exception_data,
            'last_reminded': None,
            'remind_count': 0
        }
    
    def unregister_pending_approval(self, exception_id: str):
        """Remove uma exceção da fila de lembretes"""
        if exception_id in self.pending_approvals:
            del self.pending_approvals[exception_id]
    
    def _reminder_loop(self):
        """Loop principal para verificar e enviar lembretes"""
        while self.running:
            try:
                self._check_and_send_reminders()
                time.sleep(300)  # Verificar a cada 5 minutos
            except Exception as e:
                print(f"Erro no serviço de lembretes: {e}")
                time.sleep(60)  # Esperar 1 minuto antes de tentar novamente
    
    def _check_and_send_reminders(self):
        """Verifica exceções pendentes e envia lembretes se necessário"""
        now = datetime.now()
        
        for exception_id, approval_info in self.pending_approvals.items():
            due_time = approval_info['due_time']
            last_reminded = approval_info['last_reminded']
            remind_count = approval_info['remind_count']
            
            # Calcular tempos de lembrete baseado na configuração
            reminder_schedule = self._calculate_reminder_schedule(due_time, now)
            
            # Verificar se é hora de enviar lembrete
            if self._should_send_reminder(reminder_schedule, last_reminded, remind_count):
                self._send_reminder(exception_id, approval_info)
                approval_info['last_reminded'] = now
                approval_info['remind_count'] += 1
    
    def _calculate_reminder_schedule(self, due_time: datetime, now: datetime) -> List[datetime]:
        """Calcula horários programados para lembretes"""
        time_until_due = due_time - now
        schedule = []
        
        # Lembretes configuráveis
        reminder_intervals = self.config.get('reminder_intervals', [
            {'when': '1 hour before', 'enabled': True},
            {'when': '30 minutes before', 'enabled': True},
            {'when': '15 minutes before', 'enabled': True},
            {'when': 'every 2 hours after start', 'enabled': True}
        ])
        
        for interval_config in reminder_intervals:
            if not interval_config.get('enabled', True):
                continue
                
            when = interval_config['when']
            reminder_time = self._calculate_reminder_time(when, due_time, now)
            if reminder_time and reminder_time > now:
                schedule.append(reminder_time)
        
        return sorted(schedule)
    
    def _calculate_reminder_time(self, when: str, due_time: datetime, now: datetime) -> datetime:
        """Calcula horário específico para lembrete"""
        if when == '1 hour before':
            return due_time - timedelta(hours=1)
        elif when == '30 minutes before':
            return due_time - timedelta(minutes=30)
        elif when == '15 minutes before':
            return due_time - timedelta(minutes=15)
        elif when.startswith('every'):
            # Lógica para lembretes recorrentes
            interval_hours = int(when.split()[1])  # Extrair número de horas
            if (now - due_time).total_seconds() > 0:  # Já passou do due time
                elapsed_hours = (now - due_time).total_seconds() / 3600
                next_reminder_hour = int(elapsed_hours // interval_hours + 1) * interval_hours
                return due_time + timedelta(hours=next_reminder_hour)
        
        return None
    
    def _should_send_reminder(self, schedule: List[datetime], last_reminded: datetime, 
                            remind_count: int) -> bool:
        """Determina se deve enviar lembrete agora"""
        if not schedule:
            return False
        
        now = datetime.now()
        next_reminder = schedule[0]
        
        # Enviar se próximo lembrete é agora ou já passou
        if now >= next_reminder:
            # Mas verificar debounce também
            if last_reminded is None or (now - last_reminded).total_seconds() > 900:  # 15 minutos
                return True
        
        return False
    
    def _send_reminder(self, exception_id: str, approval_info: Dict):
        """Envia lembrete para os aprovadores"""
        exception_data = approval_info['exception_data']
        approvers = approval_info['approvers']
        
        reminder_data = {
            'exception_id': exception_id,
            'exception_data': exception_data,
            'due_time': approval_info['due_time'].isoformat(),
            'remaining_time': str(approval_info['due_time'] - datetime.now()),
            'remind_count': approval_info['remind_count'] + 1
        }
        
        # Enviar lembrete por todos os canais configurados
        self.notification_service.send_reminder_notification(
            approvers, 
            reminder_data
        )
```

## 6. Template Service para Renderização de Mensagens

```python
# /notifications/template_service.py
import os
from jinja2 import Environment, FileSystemLoader, select_autoescape
from typing import Dict, Any, Optional

class TemplateService:
    def __init__(self, templates_dir: str = 'templates'):
        self.templates_dir = templates_dir
        self.env = Environment(
            loader=FileSystemLoader(templates_dir),
            autoescape=select_autoescape(['html', 'xml', 'txt'])
        )
    
    def render(self, template_name: str, data: Dict[str, Any]) -> str:
        """
        Renderiza um template com os dados fornecidos
        
        Args:
            template_name: Nome do arquivo de template
            data: Dados para renderizar no template
            
        Returns:
            String com o template renderizado
        """
        try:
            template = self.env.get_template(template_name)
            return template.render(**data)
        except Exception as e:
            raise Exception(f"Erro ao renderizar template {template_name}: {str(e)}")
    
    def render_string(self, template_string: str, data: Dict[str, Any]) -> str:
        """
        Renderiza uma string template com os dados fornecidos
        
        Args:
            template_string: String de template
            data: Dados para renderizar
            
        Returns:
            String renderizada
        """
        try:
            template = self.env.from_string(template_string)
            return template.render(**data)
        except Exception as e:
            raise Exception(f"Erro ao renderizar string template: {str(e)}")
    
    def get_available_templates(self) -> list:
        """Lista todos os templates disponíveis"""
        if not os.path.exists(self.templates_dir):
            return []
        
        templates = []
        for file in os.listdir(self.templates_dir):
            if file.endswith(('.html', '.txt', '.md')):
                templates.append(file)
        
        return templates
```

## 7. Configuração do Sistema de Notificações

### 7.1 Arquivo de Configuração
```yaml
# /config/notifications.yaml
notifications:
  # Configurações de email
  email:
    enabled: true
    smtp:
      server: smtp.company.com
      port: 587
      use_tls: true
      username: governance-notifications@company.com
      password: ${EMAIL_PASSWORD}
    sender: governance-notifications@company.com
    recipients:
      critical_exceptions: 
        - ad-team@company.com
        - release-managers@company.com
      pipeline_failures:
        - dev-team@company.com
      approval_reminders:
        - ad-team@company.com
  
  # Configurações do Slack
  slack:
    enabled: true
    webhook_url: ${SLACK_WEBHOOK_URL}
    channels:
      critical_alerts: "#ad-alerts"
      approval_notifications: "#ad-approvals"
      general_updates: "#dev-notifications"
    username: "Liquibase Governance"
    icon_emoji: ":robot_face:"
    for_high_severity: true  # Enviar notificações Slack para severidade alta também
  
  # Configurações de SMS
  sms:
    enabled: false  # Desabilitado por padrão
    provider: twilio  # ou sns, etc.
    account_sid: ${TWILIO_ACCOUNT_SID}
    auth_token: ${TWILIO_AUTH_TOKEN}
    from_number: "+1234567890"
    to_numbers:
      - "+1987654321"  # Número do líder de AD
  
  # Configurações de Push
  push:
    enabled: false  # Desabilitado por padrão
    provider: firebase  # ou apns, etc.
    server_key: ${FIREBASE_SERVER_KEY}
    topic: "critical-exceptions"
  
  # URLs para links nas notificações
  urls:
    dashboard: "https://governance.company.com"
    short_url_service: "https://go.company.com"
  
  # Políticas de debounce
  debounce:
    window_seconds: 300  # 5 minutos
    per_exception: true   # Por exceção individual
  
  # Configurações de lembretes
  reminders:
    enabled: true
    intervals:
      - when: "1 hour before"
        enabled: true
      - when: "30 minutes before"
        enabled: true
      - when: "15 minutes before"
        enabled: true
      - when: "every 2 hours after start"
        enabled: true
    
    # Grupos que recebem lembretes
    reminder_recipients:
      - AD-GROUP
      - RELEASE-MANAGERS
  
  # Timeout para diferentes severidades
  timeouts:
    critical: 48   # horas
    high: 24       # horas
    medium: 6      # horas
    low: 1         # hora
```

## 8. Métricas e Monitoramento

### 8.1 Métricas de Notificação
```python
# /monitoring/notification_metrics.py
from prometheus_client import Counter, Histogram, Gauge

class NotificationMetrics:
    def __init__(self):
        self.notifications_sent = Counter(
            'notifications_sent_total',
            'Total de notificações enviadas',
            ['channel', 'type', 'severity']
        )
        
        self.notification_failures = Counter(
            'notification_failures_total',
            'Total de falhas no envio de notificações',
            ['channel', 'error_type']
        )
        
        self.notification_latency = Histogram(
            'notification_delivery_latency_seconds',
            'Latência no envio de notificações',
            ['channel']
        )
        
        self.pending_notifications = Gauge(
            'pending_notifications_count',
            'Número de notificações pendentes de envio',
            ['type']
        )
    
    def record_notification_sent(self, channel: str, notification_type: str, severity: str):
        """Registra envio bem-sucedido de notificação"""
        self.notifications_sent.labels(
            channel=channel,
            type=notification_type,
            severity=severity
        ).inc()
    
    def record_notification_failure(self, channel: str, error_type: str):
        """Registra falha no envio de notificação"""
        self.notification_failures.labels(
            channel=channel,
            error_type=error_type
        ).inc()
    
    def record_notification_latency(self, latency_seconds: float, channel: str):
        """Registra latência de envio de notificação"""
        self.notification_latency.labels(channel=channel).observe(latency_seconds)
    
    def set_pending_notifications(self, count: int, notification_type: str):
        """Define número de notificações pendentes"""
        self.pending_notifications.labels(type=notification_type).set(count)
```

## 9. Tratamento de Erros e Resiliência

### 9.1 Sistema de Fallback para Notificações
```python
# /notifications/fallback_handler.py
import logging
from typing import List, Dict, Any

class NotificationFallbackHandler:
    def __init__(self, notification_service, metrics):
        self.notification_service = notification_service
        self.metrics = metrics
        self.logger = logging.getLogger(__name__)
    
    def send_with_fallback(self, primary_channel: str, fallback_channels: List[str], 
                          notification_func, *args, **kwargs):
        """
        Envia notificação com mecanismo de fallback
        
        Args:
            primary_channel: Canal primário para envio
            fallback_channels: Canais alternativos em caso de falha
            notification_func: Função de envio de notificação
            *args, **kwargs: Argumentos para a função de notificação
        """
        # Tentar canal primário primeiro
        try:
            notification_func(*args, **kwargs)
            self.metrics.record_notification_sent(primary_channel, 'primary', 'critical')
            return True
        except Exception as primary_error:
            self.logger.warning(f"Falha no envio via canal primário {primary_channel}: {primary_error}")
            self.metrics.record_notification_failure(primary_channel, str(primary_error))
        
        # Tentar canais de fallback
        for fallback_channel in fallback_channels:
            try:
                # Adaptar função de notificação para o canal de fallback
                fallback_func = self._adapt_for_fallback(fallback_channel, notification_func)
                fallback_func(*args, **kwargs)
                self.metrics.record_notification_sent(fallback_channel, 'fallback', 'critical')
                self.logger.info(f"Notificação enviada com sucesso via fallback {fallback_channel}")
                return True
            except Exception as fallback_error:
                self.logger.warning(f"Falha no envio via fallback {fallback_channel}: {fallback_error}")
                self.metrics.record_notification_failure(fallback_channel, str(fallback_error))
        
        # Todos os canais falharam
        self.logger.error("Todos os canais de notificação falharam")
        return False
    
    def _adapt_for_fallback(self, channel: str, original_func):
        """Adapta função de notificação para canal de fallback"""
        # Esta função pode adaptar o formato da mensagem para diferentes canais
        # Por exemplo, converter HTML para texto simples para SMS
        if channel == 'sms':
            return self._wrap_for_sms(original_func)
        elif channel == 'push':
            return self._wrap_for_push(original_func)
        else:
            return original_func
    
    def _wrap_for_sms(self, func):
        """Adapta função para envio via SMS"""
        def sms_wrapper(*args, **kwargs):
            # Extrair mensagem principal e converter para texto simples
            # Esta é uma implementação simplificada
            result = func(*args, **kwargs)
            # Lógica adicional para SMS aqui
            return result
        return sms_wrapper
    
    def _wrap_for_push(self, func):
        """Adapta função para envio via Push Notification"""
        def push_wrapper(*args, **kwargs):
            # Adaptar formato para notificação push
            result = func(*args, **kwargs)
            # Lógica adicional para Push aqui
            return result
        return push_wrapper
```

## 10. Testabilidade do Sistema de Notificações

### 10.1 Testes Unitários
```python
# /tests/unit/test_notification_service.py
import unittest
from unittest.mock import Mock, patch
from notifications.notification_service import NotificationService
from notifications.template_service import TemplateService

class TestNotificationService(unittest.TestCase):
    def setUp(self):
        self.config = {
            'notifications': {
                'email': {
                    'enabled': True,
                    'smtp': {
                        'server': 'smtp.test.com',
                        'port': 587,
                        'use_tls': True
                    },
                    'sender': 'test@test.com',
                    'recipients': {
                        'critical_exceptions': ['admin@test.com']
                    }
                },
                'slack': {
                    'enabled': True,
                    'webhook_url': 'https://hooks.slack.com/services/test'
                }
            }
        }
        self.notification_service = NotificationService(self.config)
    
    @patch('notifications.notification_service.smtplib.SMTP')
    def test_send_email_notification(self, mock_smtp):
        # Preparar dados de teste
        exceptions = [{
            'id': 'exc-001',
            'changeset': {'author': 'test', 'id': 'test-id'},
            'ignoredRule': {'name': 'no-drop-table'},
            'calculated_severity': 'critical',
            'fileName': 'test.sql',
            'lineNumber': 10
        }]
        
        build_info = {
            'build_number': '123',
            'environment': 'test',
            'timestamp': '2023-01-01 12:00:00'
        }
        
        # Executar método de teste
        self.notification_service._send_email_notification(
            self.notification_service._prepare_notification_data(exceptions, build_info)
        )
        
        # Verificar se SMTP foi chamado corretamente
        mock_smtp.assert_called_once_with('smtp.test.com', 587)
    
    @patch('notifications.notification_service.requests.post')
    def test_send_slack_notification(self, mock_post):
        mock_post.return_value.status_code = 200
        
        data = {
            'exception_count': 1,
            'exceptions': [{'rule_name': 'no-drop-table', 'severity_label': 'Crítica'}],
            'build_number': '123',
            'environment': 'test'
        }
        
        self.notification_service._send_slack_notification(data)
        
        mock_post.assert_called_once()
    
    def test_prepare_notification_data(self):
        exceptions = [{
            'id': 'exc-001',
            'changeset': {'author': 'test', 'id': 'test-id'},
            'ignoredRule': {'name': 'no-drop-table'},
            'calculated_severity': 'critical',
            'fileName': 'test.sql',
            'lineNumber': 10,
            'justification': 'Test justification'
        }]
        
        build_info = {
            'build_number': '123',
            'environment': 'test',
            'timestamp': '2023-01-01 12:00:00',
            'build_url': 'http://jenkins/job/test/123'
        }
        
        data = self.notification_service._prepare_notification_data(exceptions, build_info)
        
        self.assertEqual(data['exception_count'], 1)
        self.assertEqual(data['build_number'], '123')
        self.assertEqual(len(data['exceptions']), 1)

if __name__ == '__main__':
    unittest.main()
```

Este mecanismo de notificação abrangente garante que os administradores de dados sejam alertados imediatamente sobre exceções críticas, com múltiplos canais de comunicação, políticas de debounce para evitar spam, e um sistema resiliente com fallbacks em caso de falhas.