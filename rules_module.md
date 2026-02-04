# Módulo de Regras

## Estrutura de Diretórios

```
/rules/
├── approval_rules/
│   ├── __init__.py
│   ├── critical_changes.py
│   ├── database_object_naming.py
│   └── security_compliance.py
├── schema_validation/
│   ├── __init__.py
│   ├── xml_structure.py
│   └── yaml_structure.py
├── naming_conventions/
│   ├── __init__.py
│   ├── table_names.py
│   └── column_names.py
└── rule_engine.py
```

## Componentes

### Approval Rules
Regras para determinar se um changeset pode ser aprovado automaticamente com base em critérios predefinidos:

- `critical_changes.py`: Regras para identificar e tratar mudanças críticas
- `database_object_naming.py`: Regras de nomenclatura para objetos de banco de dados
- `security_compliance.py`: Regras para conformidade com requisitos de segurança

### Schema Validation
Validação da estrutura dos changesets do Liquibase:

- `xml_structure.py`: Validação da estrutura XML dos changesets
- `yaml_structure.py`: Validação da estrutura YAML dos changesets (se aplicável)

### Naming Conventions
Regras e validações para convenções de nomenclatura:

- `table_names.py`: Regras para nomes de tabelas
- `column_names.py`: Regras para nomes de colunas

### Rule Engine
Motor central que aplica todas as regras:

- `rule_engine.py`: Orquestrador que executa todas as regras configuradas