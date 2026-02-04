# Módulo de Testes

## Estrutura de Diretórios

```
/tests/
├── unit/
│   ├── test_changeset_processor.py
│   ├── test_rule_engine.py
│   ├── test_validators.py
│   └── test_utils.py
├── integration/
│   ├── test_liquibase_integration.py
│   ├── test_ci_integration.py
│   └── test_approval_workflow.py
├── e2e/
│   ├── test_full_approval_process.py
│   └── test_exception_handling.py
├── fixtures/
│   ├── sample_changesets/
│   ├── mock_configs/
│   └── test_data/
└── conftest.py
```

## Componentes

### Unit Tests (Testes Unitários)
Testes para unidades individuais de código:

- `test_changeset_processor.py`: Testes para o processador de changesets
- `test_rule_engine.py`: Testes para o motor de regras
- `test_validators.py`: Testes para os componentes de validação
- `test_utils.py`: Testes para funções utilitárias

### Integration Tests (Testes de Integração)
Testes que verificam a integração entre componentes:

- `test_liquibase_integration.py`: Testes de integração com arquivos do Liquibase
- `test_ci_integration.py`: Testes de integração com sistemas CI/CD
- `test_approval_workflow.py`: Testes do fluxo completo de aprovação

### End-to-End Tests (Testes de Ponta a Ponta)
Testes que simulam cenários completos de uso:

- `test_full_approval_process.py`: Teste do processo completo de aprovação
- `test_exception_handling.py`: Teste do tratamento completo de exceções

### Fixtures and Test Data
Dados e configurações usados nos testes:

- `sample_changesets/`: Exemplos de changesets para testes
- `mock_configs/`: Configurações simuladas para testes
- `test_data/`: Dados de teste variados
- `conftest.py`: Configurações comuns para o pytest