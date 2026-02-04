# Templates de Mensagens do Slack para Sistema de Governança do Liquibase

## 1. Notificação de Exceção Crítica Detectada

### 1.1 Estrutura de Blocos
```json
{
  "text": "🚨 Alerta de Exceção Crítica - Build #12345",
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
        "text": "*2 exceção(ões) crítica(s) detectada(s)* no build *12345* para o ambiente *produção*"
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
          "text": "*Changeset:*\ndev_user:create_table_orders"
        },
        {
          "type": "mrkdwn",
          "text": "*Regra:*\nno-drop-table"
        },
        {
          "type": "mrkdwn",
          "text": "*Severidade:*\nAlta"
        },
        {
          "type": "mrkdwn",
          "text": "*Arquivo:*\ndb/changelog/001-orders.xml"
        }
      ]
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Changeset:*\ndba_user:delete_old_data"
        },
        {
          "type": "mrkdwn",
          "text": "*Regra:*\nlinter-ignore-all"
        },
        {
          "type": "mrkdwn",
          "text": "*Severidade:*\nCrítica"
        },
        {
          "type": "mrkdwn",
          "text": "*Arquivo:*\ndb/changelog/002-cleanup.sql"
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
          "url": "https://governance.company.com/approval/12345",
          "style": "primary"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Ver Build"
          },
          "url": "https://jenkins.company.com/job/build/12345",
          "style": "danger"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":hourglass_flowing_sand: *Timeout:* 24 horas | :clock1: 2026-02-03 14:30:00"
        }
      ]
    }
  ],
  "icon_emoji": ":robot_face:",
  "username": "Liquibase Governance",
  "channel": "#ad-alerts"
}
```

## 2. Notificação de Aprovação Pendente

### 2.1 Estrutura de Blocos
```json
{
  "text": "📋 Solicitação de Aprovação - 3 exceções pendentes",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "📋 Solicitação de Aprovação de Exceções"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*3 exceção(ões) aguardam aprovação* no build *67890* para o ambiente *homologação*"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Exceções Prioritárias:*"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Changeset:*\njohn.doe:update_schema_config"
        },
        {
          "type": "mrkdwn",
          "text": "*Regra:*\nhas-author"
        },
        {
          "type": "mrkdwn",
          "text": "*Severidade:*\nMédia"
        },
        {
          "type": "mrkdwn",
          "text": "*Prazo:*\n4 horas restantes"
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
            "text": "Revisar Aprovações"
          },
          "url": "https://governance.company.com/dashboard/approvals",
          "style": "primary"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Extender Prazo"
          },
          "url": "https://governance.company.com/approval/extend/67890",
          "style": "default"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":busts_in_silhouette: *Responsáveis:* <!subteam^S012AB3CD> | :clock1: 2026-02-03 14:45:00"
        }
      ]
    }
  ],
  "icon_emoji": ":clipboard:",
  "username": "Liquibase Governance",
  "channel": "#ad-approvals"
}
```

## 3. Notificação de Aprovação Realizada

### 3.1 Estrutura de Blocos
```json
{
  "text": "✅ Aprovação Concluída - 2 exceções processadas",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "✅ Aprovação de Exceções Concluída"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Aprovador:* Maria Silva\n*Ações realizadas:* 2 aprovações, 0 rejeições"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Exceções Tratadas:*"
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Changeset:*\ndeveloper_x:add_index"
        },
        {
          "type": "mrkdwn",
          "text": "*Status:*\n:white_check_mark: Aprovada"
        },
        {
          "type": "mrkdwn",
          "text": "*Justificativa:*\nNecessário para performance"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":memo: *Comentário do Aprovador:* \"Exceção aceita devido ao impacto positivo na performance. Build liberado.\""
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
            "text": "Ver Histórico Completo"
          },
          "url": "https://governance.company.com/approval/history/XYZ123",
          "style": "default"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":clock1: 2026-02-03 15:00:00 | :package: Build #67890"
        }
      ]
    }
  ],
  "icon_emoji": ":white_check_mark:",
  "username": "Liquibase Governance",
  "channel": "#ad-approvals"
}
```

## 4. Notificação de Timeout de Aprovação

### 4.1 Estrutura de Blocos
```json
{
  "text": "⏰ Timeout de Aprovação - Exceções bloqueando build",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "⏰ Timeout de Aprovação de Exceções"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Tempo limite esgotado* para aprovação das exceções no build *54321*"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": ":rotating_light: *Pipeline bloqueado!* O processo de deploy foi pausado até a resolução das exceções."
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Exceções Pendentes:* 1"
        },
        {
          "type": "mrkdwn",
          "text": "*Severidade:* Alta"
        },
        {
          "type": "mrkdwn",
          "text": "*Responsáveis:* <!subteam^S012AB3CD>"
        },
        {
          "type": "mrkdwn",
          "text": "*Canal de Ação:* #ad-emergency"
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
            "text": "Resolver Imediatamente"
          },
          "url": "https://governance.company.com/emergency/54321",
          "style": "danger"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Contatar Equipe"
          },
          "url": "https://slack.com/app_redirect?channel=C012AB3CD",
          "style": "default"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":warning: *Ação Imediata Requerida* | :clock1: 2026-02-03 15:15:00"
        }
      ]
    }
  ],
  "icon_emoji": ":hourglass:",
  "username": "Liquibase Governance",
  "channel": "#ad-alerts"
}
```

## 5. Notificação de Build Impactado

### 5.1 Estrutura de Blocos
```json
{
  "text": "🔧 Build Impactado por Exceções Não Resolvidas",
  "blocks": [
    {
      "type": "header",
      "text": {
        "type": "plain_text",
        "text": "🔧 Build Impactado por Exceções"
      }
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*O build #98765 foi impactado* por 2 exceções não resolvidas\n*Status:* :large_yellow_circle: Degradado"
      }
    },
    {
      "type": "divider"
    },
    {
      "type": "section",
      "text": {
        "type": "mrkdwn",
        "text": "*Impacto no Deploy:* O pipeline continuará, mas com monitoramento intensificado."
      }
    },
    {
      "type": "section",
      "fields": [
        {
          "type": "mrkdwn",
          "text": "*Exceções Ativas:* 2"
        },
        {
          "type": "mrkdwn",
          "text": "*Ambiente:* Homologação"
        },
        {
          "type": "mrkdwn",
          "text": "*Monitoramento:* Ativado"
        },
        {
          "type": "mrkdwn",
          "text": "*Deadline:* 2026-02-05"
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
            "text": "Ver Dashboard"
          },
          "url": "https://governance.company.com/dashboard/builds/98765",
          "style": "default"
        },
        {
          "type": "button",
          "text": {
            "type": "plain_text",
            "text": "Documentação"
          },
          "url": "https://wiki.company.com/liquibase/governance",
          "style": "default"
        }
      ]
    },
    {
      "type": "context",
      "elements": [
        {
          "type": "mrkdwn",
          "text": ":information_source: A equipe de DBA será notificada sobre qualquer problema | :clock1: 2026-02-03 15:30:00"
        }
      ]
    }
  ],
  "icon_emoji": ":wrench:",
  "username": "Liquibase Governance",
  "channel": "#dev-notifications"
}
```

## 6. Diretrizes de Formatação Consistentes

### 6.1 Cores e Estilos
- **Header**: Sempre usar header block para título principal
- **Severidade Alta/Crítica**: Usar style: "danger" nos botões
- **Severidade Média**: Usar style: "primary" nos botões  
- **Severidade Baixa/Info**: Usar style: "default" nos botões

### 6.2 Ícones e Emojis
- Alertas: 🚨 ⚠️ ❗ 
- Sucesso: ✅ ✔️ 
- Ações: 📋 🔧 ⚙️ 
- Tempo: ⏰ ⌛ 🕒 
- Informação: ℹ️ 🔍 📝 

### 6.3 Estrutura Consistente
1. Cabeçalho (header)
2. Sumário (section)
3. Divisor (divider)
4. Detalhes (sections com fields)
5. Ações (actions)
6. Contexto (context)

### 6.4 Limites Importantes
- Máximo 50 blocos por mensagem
- Limitar a 5 exceções por notificação (usar paginação se necessário)
- Texto do botão: máximo 75 caracteres
- Título do header: máximo 150 caracteres