# Critérios para Classificação de Exceções e Regras Ignoradas

## 1. Categorias de Regras do Liquibase Linter

### 1.1 Regras de Segurança
Estas regras previnem operações potencialmente destrutivas ou inseguras:

| Nome da Regra | Descrição | Severidade Base |
|---------------|-----------|-----------------|
| no-drop-table | Impede comandos DROP TABLE | Alta |
| no-delete-all | Impede DELETE sem cláusula WHERE | Alta |
| no-truncate | Impede comandos TRUNCATE | Média |
| no-grant-on-public | Impede GRANT para PUBLIC role | Alta |
| disallowed-sql | Bloqueia SQLs específicos configurados | Variável |

### 1.2 Regras de Governança
Estas regras garantem conformidade e rastreabilidade:

| Nome da Regra | Descrição | Severidade Base |
|---------------|-----------|-----------------|
| has-author | Exige autor definido no changeset | Média |
| has-id | Exige ID único no changeset | Média |
| has-description | Exige descrição do changeset | Baixa |
| forbid-sale | Impede alterações em schemas sensíveis | Alta |
| forbid-production | Impede alterações em ambientes produtivos | Alta |

### 1.3 Regras de Qualidade
Estas regras mantêm padrões de qualidade e consistência:

| Nome da Regra | Descrição | Severidade Base |
|---------------|-----------|-----------------|
| naming-convention | Força convenções de nomenclatura | Baixa |
| no-illegal-words | Bloqueia palavras-chave específicas | Média |
| object-name-length | Limita tamanho de nomes de objetos | Baixa |
| minimal-changes | Limita número de mudanças por changeset | Baixa |

### 1.4 Regras de Desempenho
Estas regras previnem operações que podem impactar negativamente o desempenho:

| Nome da Regra | Descrição | Severidade Base |
|---------------|-----------|-----------------|
| no-unbounded-update | Impede UPDATE sem cláusula WHERE | Alta |
| no-unbounded-delete | Impede DELETE sem cláusula WHERE | Alta |
| modify-data-type | Alerta sobre alterações de tipo de dados | Média |
| rename-column | Alerta sobre renomeação de colunas | Média |

## 2. Critérios de Classificação de Exceções

### 2.1 Por Tipo de Ignore

#### 2.1.1 Ignore Específico (--linter-ignore-rule:nome-da-regra)
- **Classificação**: Baseada na regra ignorada
- **Severidade**: Herdada da regra original + fator de atenção

#### 2.1.2 Ignore Geral (--linter-ignore-all)
- **Classificação**: Crítica
- **Severidade**: Máxima
- **Justificativa**: Desativa todas as proteções do linter

#### 2.1.3 Ignore Mal Formado
- **Classificação**: Alerta
- **Severidade**: Baixa
- **Justificativa**: Possível erro de digitação ou má prática

### 2.2 Por Severidade

#### 2.2.1 Severidade Alta
Aplicável quando:
- Regras de segurança crítica são ignoradas (no-drop-table, no-delete-all)
- Regras de governança crítica são ignoradas (forbid-sale, forbid-production)
- É usado --linter-ignore-all
- Há múltiplos ignores críticos no mesmo changeset

**Ações**:
- Requer aprovação de Administrador de Dados (AD)
- Gera alerta imediato no CI/CD
- Bloqueia deploy automático

#### 2.2.2 Severidade Média
Aplicável quando:
- Regras de segurança moderada são ignoradas (no-truncate)
- Regras de governança são ignoradas (has-author, has-id)
- Regras de desempenho são ignoradas (no-unbounded-update)

**Ações**:
- Requer revisão durante Code Review
- Gera notificação no log de exceções
- Pode permitir deploy com warning

#### 2.2.3 Severidade Baixa
Aplicável quando:
- Regras de qualidade são ignoradas (naming-convention)
- Regras de estilo são ignoradas (object-name-length)

**Ações**:
- Registra para auditoria futura
- Permite deploy automático com registro
- Relatório periódico para equipe de desenvolvimento

### 2.3 Por Frequência de Uso

#### 2.3.1 Primeiro Uso de uma Exceção
- Mais rigoroso na avaliação
- Requer justificativa mais detalhada
- Monitoramento mais atento

#### 2.3.2 Uso Repetido de Padrões
- Permite padronização de exceções aprovadas
- Reduz burocracia para casos conhecidos
- Mantém rastreabilidade

## 3. Fatores de Ajuste de Classificação

### 3.1 Contexto do Ambiente
- **Desenvolvimento**: Redução de uma categoria de severidade
- **Homologação**: Manutenção da severidade base
- **Produção**: Aumento de uma categoria de severidade

### 3.2 Justificativa Fornecida
- **Justificativa Clara**: Redução de severidade (máximo uma categoria)
- **Justificativa Genérica**: Manutenção da severidade
- **Sem Justificativa**: Aumento de severidade (máximo uma categoria)

### 3.3 Histórico do Autor
- **Autor com bom histórico**: Redução de severidade (máximo uma categoria)
- **Autor novo ou com histórico misto**: Manutenção da severidade
- **Autor com histórico problemático**: Aumento de severidade (máximo uma categoria)

## 4. Escalonamento de Exceções

### 4.1 Exceções Aprovadas Automaticamente
- Severidade Baixa
- Justificativa clara
- Ambiente de desenvolvimento

### 4.2 Exceções Requerendo Revisão
- Severidade Média
- Qualquer ignore-all
- Ambientes de homologação/produção

### 4.3 Exceções Requerendo Aprovação Especial
- Severidade Alta
- Múltiplos ignores críticos
- Ignore-all em ambientes sensíveis

## 5. Estrutura de Dados para Armazenamento

### 5.1 Modelo de Exceção
```json
{
  "exceptionId": "uuid",
  "changeset": {
    "author": "string",
    "id": "string",
    "fileName": "string",
    "lineNumber": "integer"
  },
  "ignoredRule": {
    "name": "string",
    "category": "string",
    "baseSeverity": "string"
  },
  "exceptionType": "string",
  "calculatedSeverity": "string",
  "detectedAt": "timestamp",
  "environment": "string",
  "justification": "string",
  "status": "string",
  "approver": "string"
}
```

### 5.2 Campos de Classificação
- **exceptionType**: "specific", "all", "malformed"
- **baseSeverity**: "low", "medium", "high"
- **calculatedSeverity**: "low", "medium", "high", "critical"
- **status**: "pending", "approved", "rejected", "expired"

## 6. Métricas de Monitoramento

### 6.1 Métricas por Categoria
- Total de exceções por categoria de regra
- Taxa de aprovação por severidade
- Tempo médio para resolução de exceções

### 6.2 Métricas por Autor
- Número total de exceções solicitadas
- Taxa de aprovação por autor
- Padrões de solicitação de exceções

### 6.3 Métricas por Ambiente
- Distribuição de exceções por ambiente
- Comparativo de severidade por ambiente
- Tendências de uso de exceções ao longo do tempo