# Componentes de Formatação para Microsoft Teams

## 1. Tipos de Cartões no Teams

### MessageCard (Usado para Webhooks)
- Tipo principal para integrações via Incoming Webhooks
- Estrutura baseada em seções e fatos
- Suporta ações básicas

### Adaptive Card
- Mais flexível e moderno
- Melhor para integrações via Microsoft Graph API
- Suporta layouts complexos e ações avançadas

## 2. Seções (Sections)

As seções são os blocos de construção principais para conteúdo estruturado:

```json
{
  "sections": [
    {
      "activityTitle": "Título da Atividade",
      "activitySubtitle": "Subtítulo",
      "activityImage": "https://exemplo.com/avatar.png",
      "activityText": "Texto da atividade",
      "heroImage": {
        "image": "https://exemplo.com/hero-image.png",
        "title": "Imagem Principal"
      },
      "text": "Texto descritivo da seção",
      "facts": [],
      "images": [],
      "markdown": true
    }
  ]
}
```

### Propriedades da Seção

#### activityTitle
- Título principal da seção
- Aceita formatação Markdown
- Exibido de forma proeminente

#### activitySubtitle
- Subtítulo ou descrição secundária
- Menos destacado que o activityTitle
- Útil para metadados

#### activityImage
- Imagem de avatar ou ícone pequeno
- Dimensões sugeridas: 48x48 pixels
- Formatos: PNG, JPEG, GIF

#### activityText
- Texto descritivo principal da seção
- Mais detalhado que o subtitle
- Suporta Markdown

#### heroImage
- Imagem destacada (banner) da seção
- Dimensões sugeridas: largura máxima de 480px
- Bom para ilustrações ou diagramas

#### text
- Texto adicional da seção
- Complementa o activityText
- Suporta Markdown

## 3. Campos de Informação (Facts)

Estrutura para apresentar informações estruturadas como pares nome-valor:

```json
{
  "facts": [
    {
      "name": "Propriedade:",
      "value": "Valor da propriedade"
    }
  ]
}
```

### Características
- Máximo de 50 facts por seção
- Cada fato limitado a 1024 caracteres (nome + valor)
- Exibidos em duas colunas quando possível
- Úteis para mostrar métricas e atributos

### Formatação em Facts
- Nome termina com dois-pontos (:)
- Valor pode conter Markdown básico
- Alinhamento automático para melhor legibilidade

## 4. Imagens

Suporte a diferentes tipos de imagens:

```json
{
  "images": [
    {
      "image": "https://exemplo.com/imagem.jpg",
      "title": "Descrição da Imagem"
    }
  ]
}
```

### Tipos de Imagens

#### Imagem Hero (Banner)
- Exibida em destaque na parte superior da seção
- Dimensões recomendadas: largura máxima de 480px

#### Imagens Inline
- Exibidas após o texto da seção
- Limitadas a 5 imagens por seção
- Dimensões recomendadas: 400x200px

#### Avatar (Activity Image)
- Pequena imagem circular associada à atividade
- Dimensões: 48x48 pixels

## 5. Ações Potenciais (Potential Actions)

Interações disponíveis para os usuários:

### OpenUri
Abre uma URL em um navegador:
```json
{
  "@type": "OpenUri",
  "name": "Abrir Link",
  "targets": [
    {
      "os": "default",
      "uri": "https://exemplo.com"
    }
  ]
}
```

### HttpPOST
Realiza uma requisição POST (requer configuração adicional):
```json
{
  "@type": "HttpPOST",
  "name": "Executar Ação",
  "target": "https://exemplo.com/api/action",
  "body": "{\"param\":\"value\"}",
  "headers": [
    {
      "name": "Authorization",
      "value": "Bearer token"
    }
  ]
}
```

### ActionCard
Apresenta um formulário inline (suporte limitado):
```json
{
  "@type": "ActionCard",
  "name": "Responder",
  "inputs": [
    {
      "@type": "TextInput",
      "id": "comment",
      "title": "Comentário",
      "isMultiline": true
    }
  ],
  "actions": [
    {
      "@type": "HttpPOST",
      "name": "Enviar",
      "target": "https://exemplo.com/api/comment"
    }
  ]
}
```

## 6. Formatação de Texto

### Markdown Suportado

#### Formatação Básica
- **Negrito**: `**texto**` ou `__texto__`
- *Itálico*: `*texto*` ou `_texto_`
- `Monospace`: \`código\`
- ~~Tachado~~: `~~texto~~`

#### Listas
- Bullet points não ordenados: `- item` ou `* item`
- Listas numeradas: `1. item` ou `1) item`

#### Links
- Formato: `[texto](url)`
- URLs sozinhas são automaticamente convertidas em links

#### Citações
- Iniciar linha com `> `
- Bom para destacar trechos importantes

#### Headers (limitado)
- Somente `#` e `##` têm efeito visual
- Demais níveis ignorados na renderização

### Limitações Importantes
- Tabelas não são suportadas
- Blocos de código precisam de linguagem especificada
- HTML é ignorado (exceto `<at>` para menções)

## 7. Cores e Temas

### Cores Predefinidas
- themeColor: Cor da borda esquerda do cartão
- Baseada no código hexadecimal fornecido

### Recomendações de Cores por Contexto
| Contexto    | Cor Hexadecimal | Uso Sugerido                  |
|-------------|-----------------|-------------------------------|
| Informação  | #0076D7         | Notificações gerais           |
| Aviso       | #FFC000         | Alertas de média severidade   |
| Erro        | #FF0000         | Problemas críticos            |
| Sucesso     | #00FF00         | Operações concluídas com sucesso |
| Personalizado | Varia         | Cores da marca corporativa    |

## 8. Melhores Práticas de Layout

### Hierarquia Visual
1. Título principal do cartão (title)
2. Seções com títulos claros (activityTitle)
3. Informações estruturadas (facts)
4. Ações no final (potentialAction)

### Espaçamento e Organização
- Uma seção por tipo de informação
- Agrupar campos relacionados em facts
- Usar textos descritivos antes de listas de itens

### Responsividade
- Designs verticais funcionam melhor em dispositivos móveis
- Evitar larguras excessivas de imagens
- Textos longos devem ser quebrados em múltiplas linhas

## 9. Exemplo de Cartão Completo

```json
{
  "@type": "MessageCard",
  "@context": "http://schema.org/extensions",
  "themeColor": "0076D7",
  "summary": "Relatório diário de governança Liquibase",
  "title": "📊 Relatório Diário de Governança",
  "text": "Confira o resumo das atividades de hoje no sistema de governança do Liquibase.",
  "sections": [
    {
      "activityTitle": "**Estatísticas do Dia**",
      "activityImage": "https://company.com/assets/stats-icon.png",
      "facts": [
        {
          "name": "Changesets Processados:",
          "value": "42"
        },
        {
          "name": "Exceções Detectadas:",
          "value": "3 (1 crítica)"
        },
        {
          "name": "Aprovações Pendentes:",
          "value": "2"
        },
        {
          "name": "Taxa de Conformidade:",
          "value": "93%"
        }
      ],
      "markdown": true
    },
    {
      "activityTitle": "**Alertas Críticos**",
      "activityImage": "https://company.com/assets/alert-icon.png",
      "activityText": "Atenção necessária para as seguintes exceções:",
      "text": "- EXC-2026-015: Ignorado schema-validation rule\n- EXC-2026-018: Violado naming convention em tabela temporária",
      "markdown": true
    }
  ],
  "potentialAction": [
    {
      "@type": "OpenUri",
      "name": "📈 Dashboard Completo",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance.company.com/dashboard"
        }
      ]
    },
    {
      " "@type": "OpenUri",
      "name": "📋 Lista de Exceções",
      "targets": [
        {
          "os": "default",
          "uri": "https://governance.company.com/exceptions"
        }
      ]
    }
  ]
}
```

Nota: No exemplo acima, corrigi um erro de formatação no último bloco de potentialAction (espaço extra antes de "@type").

## 10. Validação e Depuração

### Ferramentas Úteis
- Message Card Playground da Microsoft
- Postman para testes de API
- Logs de erro detalhados

### Erros Comuns
- Excesso de caracteres nos campos
- URLs inválidas nas imagens
- JSON mal formatado
- Ações com sintaxe incorreta

### Depuração
1. Validar JSON com ferramentas online
2. Verificar limites de tamanho
3. Testar em ambiente de desenvolvimento
4. Monitorar logs de erro de integração