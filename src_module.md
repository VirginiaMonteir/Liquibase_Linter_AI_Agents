# Módulo Fonte Principal

## Estrutura de Diretórios

```
/src/
├── core/
│   ├── __init__.py
│   ├── changeset_processor.py
│   ├── approval_engine.py
│   └── governance_orchestrator.py
├── utils/
│   ├── __init__.py
│   ├── liquibase_parser.py
│   ├── file_handler.py
│   └── config_loader.py
├── models/
│   ├── __init__.py
│   ├── changeset.py
│   ├── rule.py
│   └── validation_result.py
├── cli/
│   ├── __init__.py
│   ├── main.py
│   └── commands.py
└── __init__.py
```

Este módulo está relacionado diretamente ao reminder #1: "Definir diretórios principais do projeto"

## Componentes

### Core (Núcleo)
Componentes centrais que implementam a lógica principal do sistema:

- `changeset_processor.py`: Processa changesets recebidos do Liquibase
- `approval_engine.py`: Motor de aprovação que coordena regras e validações
- `governance_orchestrator.py`: Orquestrador que coordena todo o fluxo de governança

### Utils (Utilitários)
Funções utilitárias usadas por diversos componentes:

- `liquibase_parser.py`: Parser para interpretar arquivos de changesets do Liquibase
- `file_handler.py`: Manipulação de arquivos e diretórios
- `config_loader.py`: Carregamento e processamento de arquivos de configuração

### Models (Modelos)
Modelos de dados usados no sistema:

- `changeset.py`: Representação de um changeset do Liquibase
- `rule.py`: Representação de uma regra de aprovação
- `validation_result.py`: Resultado da validação de um changeset

### CLI (Interface de Linha de Comando)
Interface de linha de comando para interagir com o sistema:

- `main.py`: Ponto de entrada principal para o CLI
- `commands.py`: Implementação dos comandos disponíveis