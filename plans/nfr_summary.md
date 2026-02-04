# Resumo dos Requisitos Não-Funcionais
## Sistema de Governança de Changesets do Liquibase

Este documento fornece uma visão consolidada de todos os requisitos não-funcionais definidos para o sistema de governança de changesets do Liquibase.

## Índice

1. [Performance e Escalabilidade](#1-performance-e-escalabilidade)
2. [Disponibilidade e Confiabilidade](#2-disponibilidade-e-confiabilidade)
3. [Manutenibilidade e Evolutividade](#3-manutenibilidade-e-evolutividade)
4. [Compatibilidade e Portabilidade](#4-compatibilidade-e-portabilidade)
5. [Métricas e Critérios de Qualidade](#5-métricas-e-critérios-de-qualidade)

## 1. Performance e Escalabilidade

### Objetivos Principais
- Processamento individual de changesets em até 2 segundos (5 segundos em pico)
- Capacidade de processar 300 changesets/minuto em condições normais
- Suporte a 20 operações simultâneas com capacidade de escalar para 100

### Recursos
- Consumo máximo de 256MB por operação e 512MB em estado ocioso
- Utilização de CPU < 80% durante processamento em lote

## 2. Disponibilidade e Confiabilidade

### SLA
- 99,5% de uptime anual (máximo 43,8 horas/ano de downtime)
- Operação 24x7, todos os dias da semana

### Recuperação
- RTO (Recovery Time Objective): 4 horas
- RPO (Recovery Point Objective): 1 hora de perda de dados

### Confiabilidade
- Taxa de acerto em aprovações automáticas ≥ 95%
- Falsos positivos < 2%
- Reprodutibilidade de resultados para mesmos inputs

## 3. Manutenibilidade e Evolutividade

### Modularidade
- Arquitetura seguindo princípios SOLID
- Baixo acoplamento e alta coesão entre componentes
- Interfaces bem definidas entre módulos

### Evolução
- Suporte a múltiplas versões de regras simultaneamente
- Feature flags para novas funcionalidades
- Estratégia de versionamento semântico (Semantic Versioning)

### Qualidade de Código
- Conformidade com PEP 8 para Python
- Cobertura mínima de testes: 80%
- Análise estática integrada ao processo de desenvolvimento

## 4. Compatibilidade e Portabilidade

### Compatibilidade com Liquibase
- Suporte para versões 3.10.x a 4.20.x
- Formatos XML, YAML e JSON de changesets
- Compatibilidade com databases changelogs e formatted SQL changelogs

### Compatibilidade com Bancos de Dados
- PostgreSQL 10+, MySQL 5.7+, Oracle 12c+, SQL Server 2016+, MariaDB 10.3+

### Compatibilidade com CI/CD
- Jenkins 2.200+, GitLab CI/CD 12.0+, GitHub Actions, Azure DevOps Pipelines
- Interfaces CLI e REST para integrações genéricas

### Portabilidade
- Suporte a Linux, Windows Server, macOS
- Empacotamento Docker e manifests Kubernetes
- Configurações externalizadas em formatos agnósticos

## 5. Métricas e Critérios de Qualidade

### Métricas Principais
- **Performance**: Latência, throughput, utilização de recursos
- **Disponibilidade**: Uptime, MTTR (Mean Time to Recovery)
- **Confiabilidade**: Taxa de acerto, falsos positivos, integridade de dados
- **Manutenibilidade**: Tempo de resolução de bugs, complexidade de código

### Critérios de Aceitação
- **Latência**: ≤ 2 segundos para 95% das operações
- **Throughput**: ≥ 300 changesets/minuto em condições normais
- **Precisão**: ≥ 95% de aprovações automáticas corretas
- **Cobertura de Testes**: ≥ 80%

### Monitoramento
- Dashboards em tempo real para performance, disponibilidade e qualidade
- Sistema de alertas com níveis crítico, alto, médio e baixo
- Relatórios diários, semanais, mensais e trimestrais

---

*Documentos detalhados disponíveis em:*
*- [Requisitos de Performance e Escalabilidade](./nfr_performance_scalability.md)*
*- [Requisitos de Disponibilidade e Confiabilidade](./nfr_availability_reliability.md)*
*- [Requisitos de Manutenibilidade e Evolutividade](./nfr_maintainability_evolvability.md)*
*- [Requisitos de Compatibilidade e Portabilidade](./nfr_compatibility_portability.md)*
*- [Métricas e Critérios de Qualidade](./nfr_quality_metrics.md)*
*- [Requisitos de Segurança](./security_requirements.md)*