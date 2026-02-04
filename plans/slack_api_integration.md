# Integração com API do Slack para Notificações do Sistema de Governança do Liquibase

## 1. Métodos de Integração

### 1.1 Webhooks de Entrada (Incoming Webhooks)
- Método principal para envio de notificações
- Simples e eficiente para mensagens unidirecionais
- Configuração através da interface do Slack

### 1.2 API Web (conversations.postMessage)
- Para casos que requerem mais controle
- Permite edição posterior das mensagens
- Requer token OAuth com escopos apropriados

### 1.3 Apps do Slack
- Abordagem recomendada para integrações robustas
- Permite funcionalidades avançadas (menus, modais, etc.)
- Maior segurança e controle de acesso

## 2. Estrutura de Payload para Webhooks

### 2.1 Estrutura Básica
```json
{
  "text": "Mensagem de fallback",
  "attachments": [],
  "blocks": [],
  "icon_emoji": ":robot_face:",
  "username": "Liquibase Governance"
}
```

### 2.2 Elementos do Payload

#### Text
- Mensagem de texto simples para clientes que não suportam blocos
- Deve conter as informações essenciais da notificação

#### Blocks
- Estrutura principal para mensagens ricas
- Array de objetos que definem a aparência da mensagem
- Limite de 50 blocos por mensagem

#### Icon Emoji/Icon URL
- Identificador visual do remetente
- Deve ser consistente com a marca da ferramenta

#### Username
- Nome de exibição do remetente
- Deve identificar claramente a origem da notificação

## 3. Componentes de Blocos do Slack

### 3.1 Header
```json
{
  "type": "header",
  "text": {
    "type": "plain_text",
    "text": "Título da mensagem"
  }
}
```

### 3.2 Section
```json
{
  "type": "section",
  "text": {
    "type": "mrkdwn",
    "text": "*Texto em Markdown* com formatação"
  },
  "fields": [
    {
      "type": "mrkdwn",
      "text": "*Campo 1:*\nValor"
    }
  ]
}
```

### 3.3 Divider
```json
{
  "type": "divider"
}
```

### 3.4 Actions
```json
{
  "type": "actions",
  "elements": [
    {
      "type": "button",
      "text": {
        "type": "plain_text",
        "text": "Texto do Botão"
      },
      "url": "https://exemplo.com",
      "style": "primary"
    }
  ]
}
```

### 3.5 Context
```json
{
  "type": "context",
  "elements": [
    {
      "type": "mrkdwn",
      "text": ":information_source: Informação adicional"
    }
  ]
}
```

## 4. Formatação de Texto e Markdown

### 4.1 Formatação Básica
- **Negrito**: `*texto*`
- *Itálico*: `_texto_`
- `Código`: `` `texto` ``
- ~~Tachado~~: `~texto~`

### 4.2 Links
- URL simples: `<http://exemplo.com>`
- Link com label: `<http://exemplo.com|Texto do Link>`

### 4.3 Listas
- Bullet points: `• item`
- Numeração: `1. item`

### 4.5 Menções
- Usuário: `<@ID_DO_USUARIO>`
- Canal: `<#ID_DO_CANAL>`
- Grupo: `<!subteam^ID_DO_GRUPO>`

## 5. Emojis e Ícones

### 5.1 Emojis Recomendados por Contexto
- Alertas: 🚨 ❗ ⚠️ 
- Sucesso: ✅ ✔️ 
- Informação: ℹ️ 🔍 
- Tempo: ⏰ 🕒 ⌛ 
- Ação: 🔧 ⚙️ 

### 5.2 Diretrizes de Uso
- Usar máximo 2-3 emojis por mensagem
- Colocar emojis no início das frases quando usados para categorização
- Evitar emojis que possam ser confundidos com outros elementos

## 6. Configuração de Canais e Destinatários

### 6.1 Canais por Tipo de Notificação
- `#ad-alerts`: Exceções críticas e alertas de alta severidade
- `#ad-approvals`: Solicitações de aprovação de exceções
- `#dev-notifications`: Notificações gerais para desenvolvedores
- `#liquibase-governance`: Métricas e relatórios do sistema

### 6.2 Menções Contextuais
- AD-GROUP: `<!subteam^ID_SUBTEAM_AQUI>`
- Usuário específico: `<@ID_USUARIO_AQUI>`
- Canal inteiro: `<!channel>` (usar com moderação)

## 7. Tratamento de Erros e Resiliência

### 7.1 Códigos de Retorno Comuns
- 200: Sucesso
- 400: Payload inválido
- 403: Token/Autorização inválida
- 429: Rate limiting

### 7.2 Estratégia de Retry
- Backoff exponencial (1s, 2s, 4s, 8s)
- Máximo de 3 tentativas
- Logar falhas persistentes para investigação

### 7.3 Monitoramento
- Registrar envios bem-sucedidos
- Contabilizar falhas por tipo
- Alertar sobre taxas altas de falha

## 8. Segurança e Práticas Recomendadas

### 8.1 Armazenamento de Tokens/Webhooks
- Nunca hardcode tokens em código-fonte
- Usar variáveis de ambiente ou sistemas de secrets management
- Rotacionar tokens periodicamente

### 8.2 Validação de Conteúdo
- Sanitizar inputs antes de incluir em mensagens
- Limitar tamanho de campos dinâmicos
- Validar integridade dos dados antes do envio

### 8.3 Privacy e Compliance
- Evitar exposição de dados sensíveis em mensagens
- Respeitar políticas de retenção de dados
- Garantir conformidade com regulamentações aplicáveis

## 9. Performance e Limites

### 9.1 Limites da API
- 1 requisição por segundo por webhook
- 50 blocos por mensagem
- Tamanho máximo de payload: 3MB

### 9.2 Otimizações
- Reutilizar conexões HTTP
- Comprimir payloads grandes quando apropriado
- Agendar envios fora de horários de pico se possível

## 10. Testabilidade

### 10.1 Ambientes de Teste
- Criar canais específicos para testes (`#liquibase-test`)
- Utilizar workspaces de desenvolvimento/integração
- Ter mecanismos para desabilitar notificações em ambientes locais

### 10.2 Mocks para Testes
- Implementar interfaces mockáveis para serviços de notificação
- Criar fixtures de payloads para testes unitários
- Simular cenários de erro para testar tratamento

## 11. Configuração por Ambiente

### 11.1 Estrutura de Configuração
```yaml
notifications:
  slack:
    enabled: true
    webhook_urls:
      alerts: "${SLACK_ALERTS_WEBHOOK}"
      approvals: "${SLACK_APPROVALS_WEBHOOK}"
      general: "${SLACK_GENERAL_WEBHOOK}"
    channels:
      alerts: "#ad-alerts"
      approvals: "#ad-approvals"
      general: "#dev-notifications"
    rate_limit:
      requests_per_second: 1
      burst_size: 5
```

### 11.2 Sobrescrita por Ambiente
- DEV: Desabilitar envio real, apenas log
- HOMOLOG: Enviar para canais de teste
- PROD: Enviar para canais oficiais