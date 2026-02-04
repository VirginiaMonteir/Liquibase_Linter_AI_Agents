# Capacidades de Análise Contextual para Tabelas Temporárias/Staging

## 1. Visão Geral

Este documento especifica as capacidades de análise contextual automatizada para identificação e avaliação de mudanças em tabelas temporárias/staging em changesets do Liquibase. Essas capacidades serão implementadas como parte do sistema de agentes de IA no módulo de governança.

## 2. Objetivos

- Identificar automaticamente tabelas temporárias e de staging em changesets
- Analisar o contexto de uso dessas tabelas
- Avaliar riscos associados às operações nessas tabelas
- Integrar análise com o fluxo de detecção de exceções existente

## 3. Capacidades Específicas

### 3.1. Detecção de Tabelas Temporárias

#### 3.1.1. Padrões de Nomenclatura
- Identificar tabelas com prefixos/sufixos comuns:
  - `temp_`, `tmp_`, `staging_`, `stage_`
  - `_temp`, `_tmp`, `_staging`
  - Padrões configuráveis pelo usuário

#### 3.1.2. Comandos DDL Associados
- `CREATE TEMPORARY TABLE`
- `CREATE GLOBAL TEMPORARY TABLE`
- `CREATE TABLE AS` com destino em tabelas temporárias

### 3.2. Detecção de Tabelas de Staging

#### 3.2.1. Padrões de Nomenclatura
- Identificar tabelas de staging por convenções:
  - `stg_`, `stage_`, `staging_`
  - `_stg`, `_stage`
  - Espaços de nomes específicos (ex: `staging_schema.*`)

#### 3.2.2. Estrutura Típica
- Tabelas com estrutura semelhante às tabelas finais
- Uso em processos ETL/ELT
- Ciclo de vida temporário controlado

### 3.3. Análise de Contexto de Uso

#### 3.3.1. Propósito Identificado
- Processamento batch
- Importação de dados
- Transformações ETL
- Testes e desenvolvimento

#### 3.3.2. Ciclo de Vida
- Criação e população
- Processamento/transformação
- Limpeza/remoção
- Dependências entre etapas

### 3.4. Avaliação de Riscos

#### 3.4.1. Impacto Potencial
- Consumo de recursos (memória, disco)
- Bloqueios em tabelas finais
- Integridade referencial
- Conflitos de concorrência

#### 3.4.2. Complexidade Operacional
- Tamanho estimado das tabelas
- Volume de dados manipulados
- Duração esperada das operações
- Dependências críticas

## 4. Integração com Componentes Existentes

### 4.1. Módulo de Regras (Rules Engine)
```
/rules/
├── approval_rules/
│   ├── temporary_tables.py          # Novo: Regras específicas para tabelas temporárias
│   └── staging_tables.py            # Novo: Regras específicas para tabelas de staging
└── naming_conventions/
    ├── table_names.py               # Atualizar: Adicionar padrões para temp/staging tables
    └── temporary_staging_patterns.py # Novo: Padrões específicos de nomenclatura
```

### 4.2. Módulo de Validação (Validators)
```
/validators/
├── content_validators/
│   ├── temporary_table_validator.py  # Novo: Validador para operações em tabelas temporárias
│   └── staging_table_validator.py    # Novo: Validador para operações em tabelas de staging
└── security_validators/
    └── temporary_access_validator.py # Novo: Validador de acesso a tabelas temporárias
```

### 4.3. Módulo de Exceções (Exceptions)
```
/exceptions/
├── detection/
│   └── table_context_detector.py     # Novo: Detector de contexto para tabelas temp/staging
└── classification/
    └── table_risk_classifier.py      # Novo: Classificador de risco para operações em tables
```

## 5. Estrutura de Dados para Análise

### 5.1. Modelo de Tabela Temporária/Staging
```python
@dataclass
class TemporaryTable:
    name: str
    type: str  # "temporary" ou "staging"
    schema: str
    creation_ddl: str
    estimated_size: str  # Estimativa de tamanho
    purpose: str  # "batch_processing", "etl", "testing", etc.
    lifecycle_stage: str  # "created", "populated", "processed", "dropped"
    
@dataclass
class TableOperation:
    table_name: str
    operation_type: str  # "CREATE", "INSERT", "UPDATE", "DELETE", "DROP"
    complexity: str  # "low", "medium", "high"
    risk_level: str  # "low", "medium", "high", "critical"
    context: str  # Contexto de uso identificado
```

## 6. Critérios de Classificação

### 6.1. Tipos de Tabelas
1. **Tabelas Temporárias Tradicionais**
   - Criadas com `CREATE TEMPORARY TABLE`
   - Escopo de sessão ou transação

2. **Tabelas Temporárias Persistentes**
   - Criadas como tabelas regulares com prefixo temporário
   - Requerem limpeza explícita

3. **Tabelas de Staging**
   - Parte de processos ETL/ELT
   - Estrutura semelhante às tabelas finais

### 6.2. Níveis de Risco

#### 6.2.1. Baixo Risco
- Tabelas pequenas (< 100MB)
- Operações simples (SELECT, INSERT)
- Criação/limpeza automática

#### 6.2.2. Médio Risco
- Tabelas de tamanho médio (100MB - 10GB)
- Operações complexas (JOINs, agregações)
- Dependências com tabelas finais

#### 6.2.3. Alto Risco
- Tabelas grandes (> 10GB)
- Operações de modificação em massa
- Ausência de plano de limpeza
- Impacto em processos críticos

## 7. Recomendações de Implementação

### 7.1. Análise de Nomenclatura
- Implementar scanner lexico para identificar padrões de nome
- Correlacionar com comandos DDL associados
- Validar consistência com convenções definidas

### 7.2. Análise de Estrutura
- Identificar relacionamentos com tabelas permanentes
- Avaliar complexidade dos índices
- Verificar constraints e triggers

### 7.3. Análise de Ciclo de Vida
- Mapear sequência de operações
- Identificar pontos de limpeza
- Validar completude do processo

### 7.4. Integração com IA
- Utilizar modelos de linguagem para entender propósito
- Aplicar análise preditiva para riscos
- Fornecer explicações sobre decisões de classificação

## 8. Métricas de Monitoramento

### 8.1. Métricas de Detecção
- Total de tabelas temporárias identificadas
- Taxa de detecção precisa
- Tempo médio de análise por changeset

### 8.2. Métricas de Risco
- Distribuição de riscos identificados
- Taxa de intervenção manual necessária
- Tempo médio para resolução de alertas

### 8.3. Métricas de Performance
- Latência da análise de contexto
- Taxa de throughput de changesets
- Utilização de recursos durante análise