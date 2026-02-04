# Configuração de Canais e Destinatários no Microsoft Teams

## 1. Estrutura de Equipes e Canais

### Equipes (Teams)
- Unidades organizacionais de nível superior
- Agrupam pessoas, canais e recursos relacionados
- Podem ser públicas ou privadas
- Associadas a grupos do Microsoft 365

### Canais (Channels)
- Espaços dentro de uma equipe para discussões específicas
- Equivalentes aos canais do Slack
- Podem ter conectores configurados individualmente
- Permitem segmentação de notificações

## 2. Configuração de Webhooks em Canais

### Processo de Configuração
1. Acessar o canal desejado no Microsoft Teams
2. Clicar em "..." (Mais opções) ao lado do nome do canal
3. Selecionar "Conectores" (Connectors)
4. Procurar por "Webhook de entrada" (Incoming Webhook)
5. Clicar em "Configurar" (Configure)
6. Definir nome e imagem opcional para o webhook
7. Copiar a URL gerada para uso na integração

### Limitações de Webhooks
- Um webhook por conector por canal
- URLs são específicas para cada canal
- Remoção do conector invalida a URL
- Sem autenticação embutida (segurança por obscuridade)

## 3. Mapeamento de Canais por Tipo de Notificação

### Canais Recomendados
| Tipo de Notificação           | Canal Sugerido    | Descrição                                           |
|------------------------------|-------------------|-----------------------------------------------------|
| Exceções Críticas            | General/Alerts    | Para problemas urgentes exigindo ação imediata     |
| Solicitações de Aprovação    | Approvals         | Para fluxos de aprovação de exceções               |
| Notificações Gerais          | Developers        | Informações gerais para a equipe de desenvolvimento|
| Relatórios e Métricas        | Governance/Metrics| Métricas agregadas e relatórios do sistema         |
| Auditoria e Compliance       | Compliance/Audit  | Registros e eventos de auditoria                   |

### Criação de Canais
Recomenda-se criar canais dedicados para cada tipo de notificação:

```
Equipe: Liquibase Governance
├── alerts         - Exceções críticas e alertas urgentes
├── approvals      - Solicitações de aprovação de exceções
├── developers     - Notificações gerais para desenvolvedores
├── governance     - Relatórios, métricas e análises do sistema
└── audit-trail    - Eventos de auditoria e compliance
```

## 4. Destinatários e Menções

### Menções no Teams
Ao contrário do Slack, o Teams tem limitações com menções em webhooks:

#### Menções Suportadas
- `<at>Nome do Usuário</at>` - Menciona usuário específico
- `<at>Todos</at>` - Menciona todos no canal (uso moderado)

#### Limitações
- Não é possível mencionar grupos ou times via webhooks
- Requer conhecimento prévio do nome exato do usuário
- Menções não geram notificações push confiáveis em webhooks

### Formato de Menções
```json
{
  "text": "<at>Administrador de Dados</at>, atenção necessária para exceção crítica."
}
```

### Boas Práticas de Menções
- Usar menções somente para situações realmente urgentes
- Preferir canais específicos em vez de mencionar todos
- Treinar usuários sobre significado das menções
- Documentar convenções de nomenclatura de usuários

## 5. Configuração Multi-ambiente

### Estratégia de Canais por Ambiente
- **Desenvolvimento**: Canais separados para testes
- **Homologação**: Canais espelhando produção
- **Produção**: Canais oficiais com stakeholders reais

### Exemplo de Estrutura Multi-ambiente

#### Ambiente DEV
```
Equipe: Liquibase-Governance-DEV
├── alerts-test    - Testes de alertas críticos
├── approvals-test - Testes de aprovações
└── notifications-dev - Notificações gerais de desenvolvimento
```

#### Ambiente PROD
```
Equipe: Liquibase-Governance
├── alerts         - Exceções críticas em produção
├── approvals      - Aprovações oficiais
├── developers     - Notificações para equipe de desenvolvimento
├── governance     - Relatórios oficiais do sistema
└── audit-trail    - Eventos de auditoria
```

## 6. Gerenciamento de Permissões

### Controle de Acesso aos Canais
- Canais privados: acesso restrito a membros selecionados
- Canais públicos: acesso para todos da equipe
- Conectores herdam permissões do canal

### Configuração Recomendada
- Canal `alerts`: Acesso restrito a administradores
- Canal `approvals`: Acesso a grupo de aprovadores
- Canal `developers`: Acesso aberto à equipe técnica
- Canal `governance`: Acesso a stakeholders relevantes

## 7. Monitoramento de Destinatários

### Métricas Importantes
- Taxa de leitura das notificações
- Tempo de resposta às solicitações
- Engajamento com as ações das notificações

### Configurações de Monitoramento
1. Configurar notificações de ausência de leitura
2. Estabelecer SLAs para resposta às aprovações
3. Criar dashboards de engajamento dos canais

## 8. Gestão de Configurações

### Práticas Recomendadas
1. Documentar mapeamentos de canais em arquivos de configuração
2. Versionar URLs de webhooks em sistemas de secrets management
3. Estabelecer processo de revisão periódica dos canais
4. Criar procedimentos para rotação de webhooks

### Exemplo de Mapeamento em YAML
```yaml
teams:
  channels:
    production:
      alerts: "https://outlook.office.com/webhook/..."
      approvals: "https://outlook.office.com/webhook/..."
      developers: "https://outlook.office.com/webhook/..."
      governance: "https://outlook.office.com/webhook/..."
    development:
      alerts_test: "https://outlook.office.com/webhook/..."
      approvals_test: "https://outlook.office.com/webhook/..."
      notifications_dev: "https://outlook.office.com/webhook/..."
```

## 9. Troubleshooting Comum

### Problemas Frequentes
1. **Webhook não funciona**
   - Verificar URL copiada corretamente
   - Confirmar que canal ainda existe
   - Validar formato do payload

2. **Menções não funcionam**
   - Confirmar nome exato do usuário
   - Verificar se usuário está no canal
   - Testar com menções simples primeiro

3. **Permissões insuficientes**
   - Verificar acesso ao canal
   - Confirmar permissões de criação de conectores
   - Validar políticas de grupo da organização

## 10. Boas Práticas de Organização

### Nomenclatura de Canais
- Usar nomes descritivos e consistentes
- Prefira hífens em vez de espaços
- Mantenha nomes curtos mas significativos

### Estruturação de Conteúdo
- Uma notificação por tipo de evento
- Agrupar eventos relacionados no mesmo canal
- Usar threads para discussões específicas

### Governança
- Estabelecer owners para cada canal
- Criar guidelines de uso dos canais
- Implementar arquivamento de canais inativos