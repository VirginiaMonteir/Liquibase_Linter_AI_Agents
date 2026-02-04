# Estrutura de Payload para Notificações no Microsoft Teams

## 1. Estrutura Geral do MessageCard

A estrutura básica de uma notificação Teams segue o padrão MessageCard do Adaptive Cards:

```json
{
  "@type": "MessageCard",
  "@context": "http://schema.org/extensions",
  "themeColor": "0076D7",
  "summary": "Resumo da notificação do sistema de governança Liquibase",
  "title": "Título da Notificação",
  "text": "Texto descritivo da notificação",
  "sections": [],
  "potentialAction": []
}
```

## 2. Detalhes dos Campos Principais

### @type
- Valor fixo: "MessageCard"
- Define o tipo de cartão a ser renderizado no Teams

### @context
- Valor fixo: "http://schema.org/extensions"
- Fornece contexto para o schema JSON-LD

### themeColor
- Código hexadecimal da cor da borda esquerda do cartão
- Recomendações por tipo de notificação:
  - Informação geral: 0076D7 (azul)
  - Aviso: FFC000 (amarelo)
  - Erro/Crítico: FF0000 (vermelho)
  - Sucesso: 00FF00 (verde)

### summary
- Texto curto mostrado em notificações compactadas
- Limite: 1024 caracteres
- Deve conter informação essencial mesmo em forma resumida

### title
- Título principal da notificação
- Visualmente destacado no cartão
- Deve ser conciso mas informativo

### text
- Descrição detalhada da notificação
- Suporta Markdown básico
- Texto complementar ao título

## 3. Estrutura de Sections

As sections são os containers principais para conteúdo estruturado:

```json
{
  "activityTitle": "**Liquibase Governance System**",
  "activitySubtitle": "Detecção de exceções críticas",
  "activityImage": "https://exemplo.com/icon-liquibase.png",
  "activityText": "Foram detectadas exceções que requerem atenção imediata",
  "facts": [],
  "markdown": true
}
```

### activityTitle
- Título da seção
- Pode conter formatação Markdown básica

### activitySubtitle
- Subtítulo ou descrição secundária
- Menos proeminente que o título principal

### activityImage
- URL de uma imagem/avatar associada à notificação
- Dimensões recomendadas: 48x48 pixels
- Formatos suportados: PNG, JPEG, GIF

### activityText
- Texto descritivo da seção
- Mais detalhado que o subtitle

### facts
- Lista de campos de informação estruturada

### markdown
- Habilita/desabilita suporte a Markdown
- Recomendado manter como true

## 4. Estrutura de Facts

Os facts são pares nome-valor para apresentar informações estruturadas:

```json
{
  "facts": [
    {
      "name": "Changeset:",
      "value": "database/migrations/V1_0_1__create_user_table.sql"
    },
    {
      "name": "Autor:",
      "value": "João Silva"
    },
    {
      "name": "Severidade:",
      "value": "Crítica"
    },
    {
      "name": "Data/Hora:",
      "value": "2026-02-04 14:00:00"
    }
  ]
}
```

### Limitações
- Máximo de 50 facts por seção
- Cada name/value limitado a 1024 caracteres

## 5. Estrutura de PotentialActions

Ações interativas disponíveis para os usuários:

```json
{
  "potentialAction": [
    {
      "@type": "OpenUri",
      "name": "Visualizar Detalhes",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance-system.example.com/exceptions/EXC-2026-001"
        }
      ]
    },
    {
      "@type": "OpenUri",
      "name": "Aprovar Exceção",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance-system.example.com/approvals/EXC-2026-001/approve"
        }
      ]
    }
  ]
}
```

### Tipos de Ações Suportadas

#### OpenUri
- Abre uma URL em um navegador
- Útil para links de detalhes, aprovações, etc.

#### HttpPOST
- Realiza uma requisição POST para um endpoint
- Requer configuração adicional no conector

## 6. Exemplo Completo de Payload

Exemplo de notificação para exceção crítica:

```json
{
  "@type": "MessageCard",
  "@context": "http://schema.org/extensions",
  "themeColor": "FF0000",
  "summary": "Exceção crítica linter-ignore-rule detectada por João Silva",
  "title": "🚨 Exceção Crítica Detectada",
  "text": "Uma exceção de severidade crítica foi detectada no pipeline de governança do Liquibase e requer atenção imediata.",
  "sections": [
    {
      "activityTitle": "**Liquibase Governance System**",
      "activitySubtitle": "Notificação Automática",
      "activityImage": "https://company.com/assets/liquibase-icon.png",
      "activityText": "Detalhes da exceção identificada:",
      "facts": [
        {
          "name": "Exceção ID:",
          "value": "EXC-2026-001"
        },
        {
          "name": "Changeset:",
          "value": "database/migrations/V1_0_1__create_user_table.sql"
        },
        {
          "name": "Autor:",
          "value": "João Silva"
        },
        {
          "name": "Severidade:",
          "value": "Crítica"
        },
        {
          "name": "Regra Ignorada:",
          "value": "table-naming-convention"
        },
        {
          "name": "Data/Hora Detecção:",
          "value": "2026-02-04 14:00:00 UTC"
        },
        {
          "name": "Pipeline:",
          "value": "main-pipeline #1234"
        }
      ],
      "markdown": true
    }
  ],
  "potentialAction": [
    {
      "@type": "OpenUri",
      "name": "📊 Visualizar Detalhes",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance-system.company.com/exceptions/EXC-2026-001"
        }
      ]
    },
    {
      "@type": "OpenUri",
      "name": "✅ Aprovar Exceção",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance-system.company.com/approvals/EXC-2026-001/approve"
        }
      ]
    },
    {
      "@type": "OpenUri",
      "name": "❌ Rejeitar Exceção",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance-system.company.com/approvals/EXC-2026-001/reject"
        }
      ]
    }
  ]
}
```

## 7. Considerações Técnicas

### Tamanho Máximo
- Payload total: 28 KB
- Summary: 1024 caracteres
- Title: 256 caracteres
- Text: 1024 caracteres
- Cada fato (name/value): 1024 caracteres

### Caracteres Especiais
- Escapar aspas duplas (\")
- Codificar corretamente caracteres Unicode
- Evitar caracteres de controle

### Performance
- Manter payloads enxutos
- Otimizar imagens para carregamento rápido
- Agrupar informações relevantes em seções

## 8. Boas Práticas

### Organização de Informações
1. Título claro e conciso
2. Resumo informativo
3. Seção principal com detalhes estruturados
4. Ações relevantes no final

### Clareza Visual
- Usar formatação consistente
- Cores apropriadas ao contexto
- Imagens relevantes e de boa qualidade
- Hierarquia visual clara

### Usabilidade
- Ações com nomes descritivos
- Links funcionais e relevantes
- Informações suficientes sem sobrecarregar
- Consistência com outras notificações do sistema