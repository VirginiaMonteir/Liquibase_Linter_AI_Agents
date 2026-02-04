# Requisitos Não-Funcionais: Performance e Escalabilidade
## Sistema de Governança de Changesets do Liquibase

### 1. Requisitos de Performance

#### 1.1 Tempo de Resposta
- **Processamento de Changesets Individuais**: 
  - Validação e aprovação automática de um único changeset deve ser concluída em no máximo 2 segundos em condições normais
  - Em cenários de pico, o tempo máximo aceitável é de 5 segundos por changeset

- **Processamento em Lote**: 
  - Validação e aprovação de até 100 changesets simultaneamente deve ser concluída em no máximo 60 segundos
  - Tempo de resposta percentil 95 (P95) para processamento em lote: 30 segundos

#### 1.2 Throughput
- **Taxa de Processamento**: 
  - Sistema deve ser capaz de processar no mínimo 300 changesets por minuto em condições normais
  - Em modo de alta carga, capacidade mínima de 150 changesets por minuto

- **Concorrência**: 
  - Suporte a no mínimo 20 operações simultâneas de validação/aprovação
  - Capacidade de escalar horizontalmente para suportar até 100 operações concorrentes

#### 1.3 Uso de Recursos
- **Memória**: 
  - Consumo máximo de memória por operação de validação: 256 MB
  - Consumo base do sistema em execução: 512 MB

- **CPU**: 
  - Utilização máxima de CPU durante processamento em lote: 80% em sistemas com 4+ cores
  - Utilização média em estado ocioso: < 10%

### 2. Requisitos de Escalabilidade

#### 2.1 Escalabilidade Vertical
- **Capacidade de Crescimento**: 
  - Sistema deve suportar aumento de 300% na carga de trabalho através de adição de recursos (CPU, memória) ao nó existente
  - Deve funcionar adequadamente com até 8 GB de RAM alocados

#### 2.2 Escalabilidade Horizontal
- **Arquitetura Distribuída**: 
  - Componentes devem ser projetados para funcionar em cluster com balanceamento de carga
  - Suporte para até 5 nós worker em configuração de cluster

- **Particionamento de Dados**: 
  - Dados de auditoria e histórico podem ser particionados por data para melhorar desempenho de consultas
  - Estratégia de sharding para dados de configuração em ambientes distribuídos

#### 2.3 Limites de Escala
- **Volume de Dados**: 
  - Sistema deve lidar com histórico de até 1 milhão de changesets avaliados
  - Banco de dados de auditoria deve suportar crescimento de 10 GB/ano

- **Taxa de Ingestão**: 
  - Capacidade de receber e processar picos de até 1000 changesets/hora
  - Buffer de filas para lidar com picos temporários de carga

### 3. Métricas de Monitoramento de Performance

#### 3.1 KPIs Principais
- Taxa de sucesso de aprovações automáticas (%)
- Tempo médio de processamento por changeset
- Latência do sistema de filas de integração CI/CD
- Utilização de recursos do sistema (CPU, memória, disco)

#### 3.2 Alertas de Performance
- **Alerta Médio**: Tempo de processamento acima de 3 segundos por changeset
- **Alerta Alto**: Taxa de throughput abaixo de 100 changesets/minuto
- **Alerta Crítico**: Falha em processar 10+ changesets consecutivos