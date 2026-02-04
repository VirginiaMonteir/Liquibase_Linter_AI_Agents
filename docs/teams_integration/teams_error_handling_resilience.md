# Tratamento de Erros e Resiliência na Integração com Microsoft Teams

## 1. Códigos de Retorno HTTP e Seus Significados

### Códigos de Sucesso
| Código | Descrição                | Ação Necessária                              |
|--------|--------------------------|----------------------------------------------|
| 200    | OK                       | Nenhuma. Mensagem entregue com sucesso       |
| 201    | Created                  | Nenhuma. Mensagem entregue com sucesso       |
| 204    | No Content               | Nenhuma. Mensagem entregue com sucesso       |

### Códigos de Erro do Cliente
| Código | Descrição                | Causa Provável                                 | Ação Recomendada                            |
|--------|--------------------------|------------------------------------------------|---------------------------------------------|
| 400    | Bad Request              | Payload mal formatado ou inválido              | Validar estrutura e conteúdo do payload     |
| 401    | Unauthorized             | Falha de autenticação (URL webhook inválida)   | Verificar e renovar URL do webhook          |
| 403    | Forbidden                | Acesso negado ao recurso                       | Confirmar permissões e URL do webhook       |
| 404    | Not Found                | Endpoint webhook não encontrado                | Verificar URL e reconfigurar webhook        |
| 413    | Payload Too Large        | Tamanho do payload excede limite (28KB)        | Reduzir tamanho do payload                  |
| 429    | Too Many Requests        | Limite de taxa excedido                        | Implementar backoff e retry                 |

### Códigos de Erro do Servidor
| Código | Descrição                | Causa Provável                                 | Ação Recomendada                            |
|--------|--------------------------|------------------------------------------------|---------------------------------------------|
| 500    | Internal Server Error    | Erro interno no serviço Teams                  | Tentar novamente mais tarde                 |
| 502    | Bad Gateway              | Gateway inválido entre cliente e servidor      | Tentar novamente mais tarde                 |
| 503    | Service Unavailable      | Serviço temporariamente indisponível           | Implementar retry com backoff exponencial   |
| 504    | Gateway Timeout          | Gateway não recebeu resposta a tempo           | Tentar novamente mais tarde                 |

## 2. Estratégias de Retry e Backoff

### Política de Retry Recomendada
Implementar uma estratégia de retry com backoff exponencial:

```python
import time
import random
import requests
from typing import Dict, Any

class TeamsNotificationRetryStrategy:
    def __init__(self, max_retries: int = 3, base_delay: float = 1.0):
        self.max_retries = max_retries
        self.base_delay = base_delay
    
    def send_with_retry(self, webhook_url: str, payload: Dict[str, Any]) -> bool:
        """Envia notificação com retry e backoff exponencial"""
        
        for attempt in range(self.max_retries + 1):
            try:
                response = requests.post(webhook_url, json=payload, timeout=30)
                
                if response.status_code in [200, 201, 204]:
                    return True
                
                # Se for um erro que não deve ser retentado
                if response.status_code in [400, 401, 403, 404, 413]:
                    print(f"Erro não recuperável: {response.status_code} - {response.text}")
                    return False
                
                # Para erros recuperáveis, fazer retry
                if attempt < self.max_retries:
                    delay = self._calculate_backoff(attempt)
                    print(f"Tentativa {attempt + 1} falhou. Retry em {delay}s...")
                    time.sleep(delay)
                else:
                    print(f"Todas as {self.max_retries + 1} tentativas falharam")
                    return False
                    
            except requests.RequestException as e:
                if attempt < self.max_retries:
                    delay = self._calculate_backoff(attempt)
                    print(f"Erro de rede na tentativa {attempt + 1}: {str(e)}. Retry em {delay}s...")
                    time.sleep(delay)
                else:
                    print(f"Erro persistente após {self.max_retries + 1} tentativas: {str(e)}")
                    return False
        
        return False
    
    def _calculate_backoff(self, attempt: int) -> float:
        """Calcula delay com jitter para backoff exponencial"""
        base_delay = self.base_delay * (2 ** attempt)  # Exponencial
        jitter = random.uniform(0, base_delay * 0.1)   # 10% jitter
        return base_delay + jitter
```

### Parâmetros Recomendados
- **Máximo de tentativas**: 3
- **Delay base**: 1 segundo
- **Backoff**: Exponencial (1s, 2s, 4s, 8s)
- **Jitter**: ±10% para evitar thundering herd
- **Timeout de requisição**: 30 segundos

## 3. Monitoramento e Logging

### Métricas Essenciais
Registrar as seguintes métricas para monitoramento:

#### Taxas de Entrega
- Total de notificações enviadas
- Taxa de sucesso de envio
- Taxa de falhas por tipo de erro
- Latência média de entrega

#### Performance
- Tempo médio de resposta do serviço
- Utilização de banda
- Tamanho médio dos payloads

#### Health Checks
- Disponibilidade do serviço Teams
- Tempo médio entre falhas
- Taxa de retry

#### Exemplo de Implementação de Métricas
```python
import time
from prometheus_client import Counter, Histogram, Gauge

class TeamsNotificationMetrics:
    def __init__(self):
        self.notifications_sent = Counter(
            'teams_notifications_sent_total',
            'Total de notificações enviadas',
            ['status', 'error_type']
        )
        
        self.notification_latency = Histogram(
            'teams_notification_latency_seconds',
            'Latência no envio de notificações',
            buckets=[0.1, 0.5, 1.0, 2.0, 5.0, 10.0, 30.0]
        )
        
        self.retry_attempts = Counter(
            'teams_retry_attempts_total',
            'Total de tentativas de retry',
            ['reason']
        )
        
        self.queue_size = Gauge(
            'teams_notification_queue_size',
            'Número de notificações na fila'
        )
    
    def record_success(self, latency: float):
        self.notifications_sent.labels(status='success', error_type='none').inc()
        self.notification_latency.observe(latency)
    
    def record_failure(self, error_type: str, retry_attempted: bool = False):
        self.notifications_sent.labels(status='failure', error_type=error_type).inc()
        if retry_attempted:
            self.retry_attempts.labels(reason=error_type).inc()
```

### Logging Recomendado
Registrar informações detalhadas para troubleshooting:

```python
import logging
import json

logger = logging.getLogger(__name__)

def log_notification_attempt(webhook_url: str, payload: dict, response_status: int, 
                           response_text: str, attempt_number: int, latency: float):
    """Loga tentativa de envio de notificação"""
    
    # Extrair apenas informações seguras para log
    safe_payload = {
        'summary': payload.get('summary', '')[:100],
        'title': payload.get('title', '')[:100],
        'sections_count': len(payload.get('sections', [])),
        'actions_count': len(payload.get('potentialAction', []))
    }
    
    logger.info(
        "Teams notification attempt",
        extra={
            'webhook_domain': webhook_url.split('/')[2] if '/' in webhook_url else 'unknown',
            'status_code': response_status,
            'response_preview': response_text[:200] if response_text else '',
            'attempt': attempt_number,
            'latency_ms': round(latency * 1000, 2),
            'payload_info': safe_payload
        }
    )

def log_persistent_failure(webhook_url: str, payload: dict, error_details: str):
    """Loga falha persistente após todas as tentativas"""
    
    logger.error(
        "Teams notification failed permanently",
        extra={
            'webhook_domain': webhook_url.split('/')[2] if '/' in webhook_url else 'unknown',
            'error_details': error_details,
            'payload_summary': payload.get('summary', '')[:200] if payload.get('summary') else ''
        }
    )
```

## 4. Circuit Breaker Pattern

Implementar circuit breaker para evitar sobrecarga contínua:

```python
import time
from enum import Enum
from typing import Optional

class CircuitState(Enum):
    CLOSED = "closed"      # Normal operation
    OPEN = "open"          # Fail fast mode
    HALF_OPEN = "half_open"  # Testing recovery

class TeamsCircuitBreaker:
    def __init__(self, failure_threshold: int = 5, timeout: int = 60):
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.failure_count = 0
        self.last_failure_time: Optional[float] = None
        self.state = CircuitState.CLOSED
    
    def can_execute(self) -> bool:
        """Verifica se operação pode ser executada"""
        
        if self.state == CircuitState.CLOSED:
            return True
        
        if self.state == CircuitState.OPEN:
            # Verificar se timeout expirou
            if (self.last_failure_time and 
                time.time() - self.last_failure_time >= self.timeout):
                self.state = CircuitState.HALF_OPEN
                return True
            return False
        
        if self.state == CircuitState.HALF_OPEN:
            return True
        
        return False
    
    def on_success(self):
        """Registrar sucesso"""
        self.failure_count = 0
        self.state = CircuitState.CLOSED
    
    def on_failure(self):
        """Registrar falha"""
        self.failure_count += 1
        self.last_failure_time = time.time()
        
        if self.failure_count >= self.failure_threshold:
            self.state = CircuitState.OPEN
    
    def get_state(self) -> CircuitState:
        return self.state
```

## 5. Fallback e Degraded Operations

### Estratégias de Fallback
Quando o Teams não está disponível, implementar fallbacks:

#### Fallback para Email
```python
class NotificationFallbackHandler:
    def __init__(self, teams_service, email_service, metrics):
        self.teams_service = teams_service
        self.email_service = email_service
        self.metrics = metrics
    
    def send_notification_with_fallback(self, notification_data: dict):
        """Envia notificação com mecanismo de fallback"""
        
        # Tentar enviar via Teams primeiro
        teams_success = self.teams_service.send_notification(notification_data)
        
        if teams_success:
            self.metrics.record_notification_sent('teams', 'primary')
            return True
        
        # Fallback para email
        email_success = self.email_service.send_notification(notification_data)
        if email_success:
            self.metrics.record_notification_sent('email', 'fallback')
            return True
            
        self.metrics.record_notification_failure('all_channels')
        return False
```

#### Cache de Notificações
Para evitar perda de notificações durante indisponibilidades:

```python
class NotificationQueue:
    def __init__(self, max_size: int = 1000):
        self.queue = []
        self.max_size = max_size
    
    def enqueue(self, notification: dict):
        """Enfileira notificação para envio posterior"""
        if len(self.queue) >= self.max_size:
            # Remover notificação mais antiga
            self.queue.pop(0)
        
        notification['timestamp'] = time.time()
        notification['retry_count'] = 0
        self.queue.append(notification)
    
    def dequeue(self) -> Optional[dict]:
        """Retira próxima notificação da fila"""
        return self.queue.pop(0) if self.queue else None
    
    def size(self) -> int:
        return len(self.queue)
```

## 6. Health Checks e Auto-diagnóstico

### Health Check para Serviço de Notificação
```python
class TeamsHealthChecker:
    def __init__(self, webhook_urls: dict, timeout: int = 10):
        self.webhook_urls = webhook_urls
        self.timeout = timeout
    
    def check_connectivity(self) -> dict:
        """Verifica conectividade com endpoints Teams"""
        results = {}
        
        for channel_name, webhook_url in self.webhook_urls.items():
            try:
                # Enviar health check payload mínimo
                health_payload = {
                    "@type": "MessageCard",
                    "@context": "http://schema.org/extensions",
                    "summary": "Health check",
                    "text": "Conectividade OK"
                }
                
                response = requests.post(
                    webhook_url, 
                    json=health_payload, 
                    timeout=self.timeout
                )
                
                results[channel_name] = {
                    'status': 'healthy' if response.status_code in [200, 201, 204] else 'unhealthy',
                    'status_code': response.status_code,
                    'response_time_ms': response.elapsed.total_seconds() * 1000
                }
                
            except Exception as e:
                results[channel_name] = {
                    'status': 'unreachable',
                    'error': str(e)
                }
        
        return results
```

## 7. Alertas de Monitoramento

### Thresholds Críticos
Configurar alertas para os seguintes cenários:

#### Falhas de Envio
- Taxa de falha > 5% por 5 minutos consecutivos
- Qualquer erro 401 ou 403 (possível comprometimento de URL)
- Erros 429 repetidos (necessidade de ajuste de rate limiting)

#### Performance Degradada
- Latência média > 5 segundos por 10 minutos
- Mais de 10 tentativas de retry por minuto
- Fila de notificações > 100 itens por 5 minutos

#### Disponibilidade do Serviço
- Mais de 3 endpoints retornando estado unhealthy simultaneamente
- Circuit breaker em estado OPEN por mais de 15 minutos
- Falhas persistentes (>90% de falhas) por 30 minutos

## 8. Troubleshooting Guiado

### Erros Comuns e Soluções

#### "401 Unauthorized"
1. Verificar se URL do webhook ainda é válida
2. Reconfigurar webhook no canal do Teams
3. Atualizar URL nas variáveis de ambiente/secrets
4. Reiniciar serviço que utiliza o webhook

#### "400 Bad Request"
1. Validar estrutura JSON do payload
2. Verificar limites de tamanho (28KB total)
3. Confirmar valores dos campos obrigatórios
4. Testar com payload mínimo de exemplo

#### "429 Too Many Requests"
1. Implementar backoff exponencial
2. Reduzir frequência de envio
3. Distribuir carga entre diferentes canais
4. Considerar upgrade de plano (se aplicável)

#### Timeout de Conexão
1. Verificar conectividade de rede
2. Confirmar acesso ao domínio do webhook
3. Verificar firewalls/proxies
4. Aumentar timeout (máximo recomendado: 60 segundos)

## 9. Configurações Recomendadas para Resiliência

### Parâmetros de Configuração
```yaml
notifications:
  teams:
    retry:
      max_attempts: 3
      base_delay_seconds: 1
      max_delay_seconds: 60
      jitter_percentage: 0.1
    
    circuit_breaker:
      failure_threshold: 5
      timeout_seconds: 60
      half_open_attempts: 3
    
    http:
      timeout_seconds: 30
      connection_timeout_seconds: 10
    
    queue:
      max_size: 1000
      persistence_enabled: true
      flush_interval_seconds: 30
    
    rate_limiting:
      requests_per_second: 1
      burst_size: 5
```

### Boas Práticas de Implementação
- Isolar chamadas externas em threads separadas
- Usar timeouts rigorosos para evitar bloqueios
- Implementar graceful degradation
- Fornecer feedback claro sobre status de envio
- Manter histórico para análise pós-incidente