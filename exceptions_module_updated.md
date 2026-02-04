# Módulo de Tratamento de Exceções

## Estrutura de Diretórios

```
/exceptions/
├── detection/
│   ├── __init__.py
│   ├── pattern_detector.py
│   ├── anomaly_scanner.py
│   └── liquibase_ignore_detector.py
├── handling/
│   ├── __init__.py
│   ├── exception_resolver.py
│   └── manual_review_processor.py
├── reporting/
│   ├── __init__.py
│   ├── exception_reporter.py
│   └── summary_generator.py
├── classification/
│   ├── __init__.py
│   └── exception_classifier.py
├── storage/
│   ├── __init__.py
│   └── exception_repository.py
├── integration/
│   ├── __init__.py
│   └── validation_adapter.py
└── exception_engine.py
```

Este módulo está relacionado diretamente ao reminder #2: "Criar estrutura para componentes de detecção de exceções"

## Componentes

### Detection (Detecção)
Componentes responsáveis por identificar situações excepcionais nos changesets:

- `pattern_detector.py`: Detector de padrões incomuns ou potencialmente problemáticos
- `anomaly_scanner.py`: Scanner para identificar anomalias em relação ao comportamento esperado
- `liquibase_ignore_detector.py`: Detector específico para exceções linter-ignore-rule em changesets Liquibase

### Classification (Classificação)
Componentes responsáveis por classificar as exceções detectadas:

- `exception_classifier.py`: Classifica exceções com base em critérios de severidade e categoria

### Storage (Armazenamento)
Componentes responsáveis por armazenar e gerenciar exceções:

- `exception_repository.py`: Repositório para persistência de registros de exceções

### Integration (Integração)
Componentes responsáveis por integrar o tratamento de exceções com outros módulos:

- `validation_adapter.py`: Adaptador para integração com o módulo de validação

### Handling (Tratamento)
Componentes que definem como lidar com as exceções detectadas:

- `exception_resolver.py`: Resolve exceções automaticamente quando possível
- `manual_review_processor.py`: Encaminha exceções que requerem revisão humana

### Reporting (Relatórios)
Componentes para gerar relatórios sobre exceções detectadas:

- `exception_reporter.py`: Gera relatórios detalhados sobre exceções encontradas
- `summary_generator.py`: Cria resumos periódicos das exceções para acompanhamento

### Exception Engine
Motor central que coordena todas as operações de detecção e tratamento de exceções:

- `exception_engine.py`: Orquestrador que gerencia o fluxo completo de tratamento de exceções