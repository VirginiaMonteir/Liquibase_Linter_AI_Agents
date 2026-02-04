# Módulo de Integração CI/CD

## Estrutura de Diretórios

```
/ci-scripts/
├── jenkins/
│   ├── __init__.py
│   ├── pipeline_steps.py
│   ├── approval_gate.py
│   └── notification_hooks.py
├── gitlab_ci/
│   ├── __init__.py
│   ├── approval_job.py
│   └── integration_script.py
├── generic_pipelines/
│   ├── __init__.py
│   ├── cli_interface.py
│   └── webhook_handler.py
└── ci_adapter.py
```

## Componentes

### Jenkins Integration
Componentes específicos para integração com Jenkins:

- `pipeline_steps.py`: Steps personalizados para pipelines Jenkins
- `approval_gate.py`: Gate de aprovação para ser usado em pipelines
- `notification_hooks.py`: Hooks para notificações em diferentes etapas

### GitLab CI Integration
Componentes para integração com GitLab CI:

- `approval_job.py`: Job de aprovação automática para .gitlab-ci.yml
- `integration_script.py`: Script de integração com o pipeline do GitLab

### Generic Pipeline Support
Componentes para suporte a outros sistemas de CI/CD:

- `cli_interface.py`: Interface de linha de comando para execução em qualquer pipeline
- `webhook_handler.py`: Handler para webhooks de sistemas de controle de versão

### CI Adapter
Adaptador central que facilita a integração com diferentes sistemas:

- `ci_adapter.py`: Camada de abstração para integração com múltiplos sistemas CI/CD