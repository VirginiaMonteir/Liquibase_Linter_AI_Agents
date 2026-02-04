# Módulo de Auditoria e Logging

## Estrutura de Diretórios

```
/logs/
├── audit_logs/
│   ├── changeset_approvals.log
│   ├── changeset_rejections.log
│   └── exception_decisions.log
├── execution_logs/
│   ├── validator_executions.log
│   ├── rule_evaluations.log
│   └── ci_integrations.log
├── error_logs/
│   ├── validation_errors.log
│   ├── rule_evaluation_errors.log
│   └── integration_failures.log
└── log_manager.py
```

Este módulo está relacionado diretamente ao reminder #4: "Configurar espaço para armazenamento de logs e auditoria"

## Componentes

### Audit Logs (Logs de Auditoria)
Registros detalhados de decisões tomadas pelo sistema:

- `changeset_approvals.log`: Registra todos os changesets aprovados automaticamente
- `changeset_rejections.log`: Registra todos os changesets rejeitados e seus motivos
- `exception_decisions.log`: Registra decisões tomadas sobre exceções identificadas

### Execution Logs (Logs de Execução)
Registros detalhados da execução dos componentes do sistema:

- `validator_executions.log`: Registra a execução de todos os validadores
- `rule_evaluations.log`: Registra a avaliação de todas as regras
- `ci_integrations.log`: Registra interações com sistemas de CI/CD

### Error Logs (Logs de Erro)
Registros de erros e falhas encontrados durante a execução:

- `validation_errors.log`: Erros ocorridos durante processos de validação
- `rule_evaluation_errors.log`: Erros ocorridos durante avaliação de regras
- `integration_failures.log`: Falhas na integração com sistemas externos

### Log Manager
Gerenciador central dos logs:

- `log_manager.py`: Componente responsável por configurar e gerenciar todos os logs