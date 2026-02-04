# Módulo de Validação

## Estrutura de Diretórios

```
/validators/
├── schema_validators/
│   ├── __init__.py
│   ├── liquibase_xml_validator.py
│   └── liquibase_yaml_validator.py
├── content_validators/
│   ├── __init__.py
│   ├── sql_statement_validator.py
│   └── object_dependency_validator.py
├── security_validators/
│   ├── __init__.py
│   ├── ddl_command_validator.py
│   └── data_access_validator.py
└── validation_engine.py
```

## Componentes

### Schema Validators
Validadores responsáveis por verificar a conformidade estrutural dos changesets:

- `liquibase_xml_validator.py`: Valida a estrutura XML dos changesets do Liquibase
- `liquibase_yaml_validator.py`: Valida a estrutura YAML dos changesets do Liquibase (se utilizado)

### Content Validators
Validadores que analisam o conteúdo e a lógica dos changesets:

- `sql_statement_validator.py`: Valida comandos SQL dentro dos changesets
- `object_dependency_validator.py`: Verifica dependências entre objetos de banco de dados

### Security Validators
Validadores que verificam aspectos de segurança nos changesets:

- `ddl_command_validator.py`: Analisa comandos DDL em busca de potenciais problemas de segurança
- `data_access_validator.py`: Verifica operações que acessam ou modificam dados sensíveis

### Validation Engine
Motor central que coordena todas as validações:

- `validation_engine.py`: Orquestrador que executa todos os validadores configurados