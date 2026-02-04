# Integração com API do Microsoft Teams para Notificações do Sistema de Governança do Liquibase

## 1. Métodos de Integração

### 1.1 Webhooks de Entrada (Incoming Webhooks)
- Método principal para envio de notificações
- Simples e eficiente para mensagens unidirecionais
- Configuração através da interface do Microsoft Teams
- Similar ao método usado no Slack, mas com algumas diferenças estruturais

### 1.2 Conectores do Office 365 (Office 365 Connectors)
- Abordagem mais robusta para integrações com Teams
- Permite configurações mais avançadas e personalizadas
- Suporte a múltiplos formatos de mensagem
- Maior controle sobre autenticação e segurança

### 1.3 Microsoft Graph API
- Para casos que requerem funcionalidades avançadas
- Permite interação bidirecional com o Teams
- Requer permissões mais elevadas e autenticação complexa
- Adequado para aplicações empresariais complexas

## 2. Estrutura de Payload para Webhooks

### 2.1 Estrutura Básica do Teams
```json
{
  "@type": "MessageCard",
  "@context": "http://schema.org/extensions",
  "themeColor": "0076D7",
  "summary": "Resumo da notificação",
  "sections": [],
  "potentialAction": []
}
```

### 2.2 Elementos do Payload do Teams

#### @type
- Define o tipo de cartão a ser exibido
- Para notificações de governança, usar "MessageCard"

#### @context
- Especifica o contexto do schema JSON-LD
- Valor fixo: "http://schema.org/extensions"

#### themeColor
- Cor da borda esquerda do cartão
- Deve refletir a gravidade da notificação
- Padrões: azul (informação), amarelo (aviso), vermelho (erro)

#### summary
- Texto de resumo exibido nas notificações compactadas
- Deve conter as informações essenciais da notificação
- Limitado a 1024 caracteres

#### sections
- Componentes principais do conteúdo da mensagem
- Array de objetos que definem a aparência e organização do conteúdo

#### potentialAction
- Ações interativas disponíveis para o usuário
- Botões e links para ações relacionadas à notificação

## 3. Componentes de Cartões do Teams

### 3.1 Section Básica
```json
{
  "activityTitle": "Título da atividade",
  "activitySubtitle": "Subtítulo",
  "activityImage": "https://url-do-avatar.com/imagem.png",
  "facts": [],
  "markdown": true
}
```

### 3.2 Facts (Campos de Informação)
```json
{
  "name": "Campo:",
  "value": "Valor do campo"
}
```

### 3.3 Ações Potenciais
```json
{
  "@type": "OpenUri",
  "name": "Texto do Botão",
  "targets": [
    { "os": "default", "uri": "https://exemplo.com" }
  ]
}
```

## 4. Formatação de Texto e Markdown

### 4.1 Formatação Básica
- **Negrito**: `**texto**`
- *Itálico*: `*texto*`
- `Código`: \`texto\`
- ~~Tachado~~: `~~texto~~`

### 4.2 Links
- URL simples: `[Texto do Link](http://exemplo.com)`
- Links automáticos são convertidos em hyperlinks

### 4.3 Listas
- Bullet points: `- item` ou `* item`
- Numeração: `1. item`

### 4.4 Menções
- Usuário: `<at>Nome do Usuário</at>`
- Canal: Não suportado diretamente em webhooks (requer Graph API)

## 5. Ícones e Cores Temáticas

### 5.1 Cores Recomendadas por Contexto
- Alertas: FF0000 (vermelho)
- Sucesso: 00FF00 (verde)
- Informação: 0076D7 (azul)
- Aviso: FFC000 (amarelo)
- Tempo: 800080 (roxo)

### 5.2 Diretrizes de Uso
- Usar cores consistentes com a severidade da notificação
- Aplicar tema adequado ao contexto da mensagem
- Evitar uso excessivo de cores que possam distrair

## 6. Configuração de Canais e Destinatários

### 6.1 Canais por Tipo de Notificação
- `General`: Exceções críticas e alertas de alta severidade
- `Approvals`: Solicitações de aprovação de exceções
- `Developers`: Notificações gerais para desenvolvedores
- `Governance`: Métricas e relatórios do sistema

### 6.2 Menções Contextuais
- AD-GROUP: `<at>Administradores de Dados</at>`
- Usuário específico: `<at>Nome do Desenvolvedor</at>`
- Canal inteiro: Não recomendado (limitações técnicas)

## 7. Tratamento de Erros e Resiliência

### 7.1 Códigos de Retorno Comuns
- 200: Sucesso
- 400: Payload inválido
- 401: Autenticação inválida
- 403: Acesso negado
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

### 8.1 Armazenamento de URLs de Webhooks
- Nunca hardcode URLs em código-fonte
- Usar variáveis de ambiente ou sistemas de secrets management
- Rotacionar URLs periodicamente

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
- Tamanho máximo de payload: 28 KB
- 50 fatos (facts) por seção
- 4 ações potenciais por cartão

### 9.2 Otimizações
- Reutilizar conexões HTTP
- Minimizar tamanho dos payloads
- Agendar envios fora de horários de pico se possível

## 10. Testabilidade

### 10.1 Ambientes de Teste
- Criar canais específicos para testes
- Utilizar equipes de desenvolvimento/integração
- Ter mecanismos para desabilitar notificações em ambientes locais

### 10.2 Mocks para Testes
- Implementar interfaces mockáveis para serviços de notificação
- Criar fixtures de payloads para testes unitários
- Simular cenários de erro para testar tratamento

## 11. Configuração por Ambiente

### 11.1 Estrutura de Configuração
```yaml
notifications:
  teams:
    enabled: true
    webhook_urls:
      alerts: "${TEAMS_ALERTS_WEBHOOK}"
      approvals: "${TEAMS_APPROVALS_WEBHOOK}"
      general: "${TEAMS_GENERAL_WEBHOOK}"
    colors:
      critical: "FF0000"
      warning: "FFC000"
      info: "0076D7"
    rate_limit:
      requests_per_second: 1
      burst_size: 5
```

### 11.2 Sobrescrita por Ambiente
- DEV: Desabilitar envio real, apenas log
- HOMOLOG: Enviar para canais de teste
- PROD: Enviar para canais oficiais