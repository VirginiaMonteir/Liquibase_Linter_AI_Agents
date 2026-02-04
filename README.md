# Projeto de Governança Automatizada de Changesets do Liquibase

Este projeto implementa um sistema automatizado de governança para changesets do Liquibase, com foco em aprovação condicional baseada em regras e integração com CI/CD.

## Estrutura do Projeto

- `src/` - Código fonte principal
- `rules/` - Definições de regras para aprovação condicional
- `ci-scripts/` - Scripts de integração com CI/CD (Jenkins, GitLab CI)
- `validators/` - Componentes de validação de changesets
- `docs/` - Documentação técnica
- `tests/` - Testes automatizados
- `logs/` - Registros de auditoria e logs
- `exceptions/` - Tratamento de exceções e casos especiais

## Funcionalidades Principais

- Validação automática de changesets do Liquibase
- Aprovação condicional baseada em regras configuráveis
- Integração com pipelines de CI/CD
- Auditoria completa de todas as ações
- Detecção e tratamento de exceções