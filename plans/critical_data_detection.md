# Detecção de Dados Críticos em Changesets

## 1. Visão Geral

Este documento define os mecanismos de detecção de dados críticos em changesets do Liquibase, que será uma capacidade importante dos agentes de IA no sistema de governança. A detecção de dados críticos é essencial para classificar adequadamente o risco associado às mudanças propostas.

## 2. Objetivos

- Identificar automaticamente operações envolvendo dados críticos
- Classificar o nível de criticidade baseado no tipo de dado
- Integrar análise com o fluxo de governança existente
- Prover informações contextuais para tomada de decisão

## 3. Tipos de Dados Críticos

### 3.1. Dados Pessoais Sensíveis
- Informações de identificação pessoal (PII)
  - CPF/CNPJ
  - RG, passaporte
  - Endereços completos
  - Números de telefone
  - Datas de nascimento

### 3.2. Dados Financeiros
- Informações bancárias
  - Números de conta
  - Números de cartão de crédito/débito
  - Chaves PIX
- Dados de transações financeiras
- Salários e compensações

### 3.3. Dados de Saúde
- Informações médicas
- Histórico de tratamentos
- Resultados de exames
- Prescrições médicas

### 3.4. Dados de Segurança
- Credenciais de acesso
  - Senhas
  - Chaves de API
  - Tokens de autenticação
- Informações de infraestrutura crítica

### 3.5. Dados Empresariais Sensíveis
- Informações proprietárias
- Dados de clientes
- Informações estratégicas
- Dados de propriedade intelectual

## 4. Mecanismos de Detecção

### 4.1. Análise de Nomes de Colunas/Tabelas
- Identificar padrões conhecidos de dados críticos
- Utilizar dicionário de palavras-chave
- Analisar contexto semântico

### 4.2. Análise de Valores Literais
- Detectar padrões de dados sensíveis em valores hard-coded
- Utilizar expressões regulares específicas
- Aplicar técnicas de ofuscação para evitar falsos positivos

### 4.3. Análise de Comandos SQL
- Identificar operações de DML em tabelas sensíveis
- Detectar operações de DDL que afetam estrutura de dados críticos
- Analisar joins e relacionamentos com dados sensíveis

### 4.4. Detecção de Contexto Funcional
- Identificar operações em módulos de negócio críticos
- Analisar contexto de aplicação
- Considerar ambiente de implantação

## 5. Níveis de Criticidade

### 5.1. Baixo Risco
- Operações em tabelas não sensíveis
- Consultas sem restrição a dados críticos
- Mudanças estruturais em objetos não sensíveis

### 5.2. Médio Risco
- Operações em tabelas com dados potencialmente sensíveis
- Modificações em estruturas que podem afetar dados críticos indiretamente
- Acesso a views ou stored procedures que manipulam dados críticos

### 5.3. Alto Risco
- Operações diretas em dados críticos identificados
- Modificações em estrutura de tabelas com dados sensíveis
- Criação ou modificação de mecanismos de acesso a dados críticos

### 5.4. Crítico
- Exclusão em massa de dados críticos
- Modificação irreversível de dados sensíveis
- Quebra de mecanismos de proteção de dados
- Violação de compliance (LGPD, GDPR, etc.)

## 6. Estrutura de Dados para Detecção

### 6.1. Modelo de Dado Crítico
```python
@dataclass
class CriticalDataElement:
    element_type: str  # "column", "table", "schema", "procedure"
    element_name: str  # Nome do elemento
    data_category: str  # Categoria de dado crítico
    sensitivity_level: str  # "low", "medium", "high", "critical"
    confidence_score: float  # Grau de certeza na detecção (0.0 - 1.0)
    context_info: dict  # Informações contextuais adicionais
    
@dataclass
class CriticalDataOperation:
    operation_type: str  # "SELECT", "INSERT", "UPDATE", "DELETE", "DDL"
    affected_elements: List[CriticalDataElement]
    risk_level: str  # "low", "medium", "high", "critical"
    justification: str  # Justificativa para o risco atribuído
```

## 7. Integração com Componentes Existentes

### 7.1. Módulo de Regras (Rules Engine)
```
/rules/
├── approval_rules/
│   └── critical_data_rules.py          # Novo: Regras específicas para dados críticos
└── security_compliance.py              # Atualizar: Adicionar regras de proteção de dados
```

### 7.2. Módulo de Validação (Validators)
```
/validators/
├── security_validators/
│   ├── data_sensitivity_validator.py   # Novo: Validador para sensibilidade de dados
│   └── pii_protection_validator.py     # Novo: Validador para proteção de PII
└── content_validators/
    └── critical_operation_validator.py # Novo: Validador para operações críticas
```

### 7.3. Módulo de Exceções (Exceptions)
```
/exceptions/
├── detection/
│   └── critical_data_detector.py       # Novo: Detector de dados críticos
└── classification/
    └── data_risk_classifier.py         # Novo: Classificador de risco de dados
```

## 8. Critérios de Classificação

### 8.1. Fatores de Classificação
1. **Tipo de Dado**: Categoria de dado crítico identificado
2. **Operação Realizada**: Tipo de operação SQL executada
3. **Volume de Dados**: Quantidade de registros afetados
4. **Ambiente**: Ambiente de implantação (dev, staging, produção)
5. **Histórico**: Padrões anteriores do desenvolvedor

### 8.2. Matriz de Risco
| Tipo Dado | Operação | Volume | Ambiente | Nível de Risco |
|-----------|----------|--------|----------|----------------|
| PII Baixo | SELECT   | < 100  | DEV      | Baixo          |
| PII Médio | UPDATE   | < 1000 | STAGING  | Médio          |
| PII Alto  | DELETE   | > 1000 | PROD     | Crítico        |
| Financeiro| INSERT   | > 100  | PROD     | Alto           |

## 9. Recomendações de Implementação

### 9.1. Análise Lexical
- Implementar tokenizer para análise de comandos SQL
- Utilizar bibliotecas de detecção de padrões
- Criar dicionário de palavras-chave sensíveis

### 9.2. Análise Semântica
- Utilizar modelos de linguagem para entender contexto
- Implementar análise de relacionamentos entre tabelas
- Considerar hierarquia de dados e dependências

### 9.3. Detecção de Padrões
- Criar expressões regulares para padrões conhecidos
- Implementar detecção de ofuscação (base64, hashing)
- Utilizar técnicas de machine learning para padrões complexos

### 9.4. Integração com IA
- Treinar modelos especializados para tipos de dados
- Utilizar embeddings para similaridade semântica
- Implementar mecanismos de explicabilidade para decisões

## 10. Métricas de Monitoramento

### 10.1. Métricas de Detecção
- Total de elementos críticos identificados
- Taxa de precisão da detecção
- Tempo médio de análise por changeset

### 10.2. Métricas de Classificação
- Distribuição de níveis de criticidade
- Acerto nas classificações manuais vs automáticas
- Número de intervenções humanas necessárias

### 10.3. Métricas de Performance
- Latência da análise de dados críticos
- Taxa de throughput de changesets
- Utilização de recursos durante análise

## 11. Considerações de Privacidade e Segurança

### 11.1. Proteção de Dados
- Não armazenar valores literais de dados sensíveis
- Utilizar hashing para identificação de padrões
- Implementar controle de acesso aos metadados

### 11.2. Conformidade Legal
- LGPD (Lei Geral de Proteção de Dados)
- GDPR (General Data Protection Regulation)
- SOX (Sarbanes-Oxley Act) para auditoria

### 11.3. Auditoria
- Registrar todas as detecções de dados críticos
- Manter histórico de decisões tomadas
- Garantir rastreabilidade completa