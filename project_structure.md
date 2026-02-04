# Estrutura de Diretórios do Projeto

## Visão Geral

Esta estrutura foi projetada para suportar a governança automatizada de changesets do Liquibase com foco em aprovação condicional baseada em regras e integração com CI/CD.

```mermaid
graph TD
    A[Projeto Raiz] --> B[src]
    A --> C[rules]
    A --> D[ci-scripts]
    A --> E[validators]
    A --> F[docs]
    A --> G[tests]
    A --> H[logs]
    A --> I[exceptions]
    
    B --> B1[core]
    B --> B2[utils]
    B --> B3[models]
    
    C --> C1[approval_rules]
    C --> C2[schema_validation]
    C --> C3[naming_conventions]
    
    D --> D1[jenkins]
    D --> D2[gitlab_ci]
    D --> D3[generic_pipelines]
    
    E --> E1[schema_validators]
    E --> E2[content_validators]
    E --> E3[security_validators]
    
    H --> H1[audit_logs]
    H --> H2[execution_logs]
    H --> H3[error_logs]
    
    I --> I1[detection]
    I --> I2[handling]
    I --> I3[reporting]
```

## Diretórios Principais

### `/src`
Código fonte principal em Python:

- `core/` - Lógica central do sistema
- `utils/` - Funções utilitárias
- `models/` - Modelos de dados

### `/rules`
Definições de regras para aprovação condicional:

- `approval_rules/` - Regras para aprovação automática
- `schema_validation/` - Regras de validação de esquema
- `naming_conventions/` - Convenções de nomenclatura

### `/ci-scripts`
Scripts de integração com sistemas de CI/CD:

- `jenkins/` - Scripts específicos para Jenkins
- `gitlab_ci/` - Scripts para GitLab CI
- `generic_pipelines/` - Scripts genéricos para outros pipelines

### `/validators`
Componentes de validação de changesets:

- `schema_validators/` - Validadores de esquema
- `content_validators/` - Validadores de conteúdo
- `security_validators/` - Validadores de segurança

### `/docs`
Documentação técnica:

- `architecture/` - Documentos de arquitetura
- `processes/` - Processos e fluxos de trabalho
- `api/` - Documentação da API
- `guides/` - Guias de uso

### `/tests`
Testes automatizados:

- `unit/` - Testes unitários
- `integration/` - Testes de integração
- `e2e/` - Testes end-to-end

### `/logs`
Registros de auditoria e logs:

- `audit_logs/` - Logs de auditoria de ações
- `execution_logs/` - Logs de execução
- `error_logs/` - Logs de erros

### `/exceptions`
Tratamento de exceções e casos especiais:

- `detection/` - Componentes de detecção de exceções
- `handling/` - Componentes de tratamento de exceções
- `reporting/` - Relatórios de exceções

## Arquivos de Configuração

Arquivos de configuração raiz que afetam todo o projeto:

- `requirements.txt` - Dependências do Python
- `config.yaml` - Configurações gerais
- `.gitignore` - Arquivos ignorados pelo Git
- `README.md` - Documentação principal