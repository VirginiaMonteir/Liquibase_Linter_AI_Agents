# Requisitos Não-Funcionais: Disponibilidade e Confiabilidade
## Sistema de Governança de Changesets do Liquibase

### 1. Requisitos de Disponibilidade

#### 1.1 SLA de Disponibilidade
- **Objetivo de Disponibilidade**: 99,5% de uptime anual (tempo máximo de downtime: 43,8 horas/ano)
- **Horário de Operação**: 24x7, todos os dias da semana
- **Janela de Manutenção Planejada**: Máximo de 4 horas por semana, preferencialmente fora do horário comercial

#### 1.2 Tolerância a Falhas
- **Componentes Críticos**: 
  - Motor de validação de changesets deve ter failover automático
  - Banco de dados de auditoria deve ter replicação e backup automático
  - Filas de mensagens devem ter persistência garantida

- **Recuperação Automática**: 
  - Sistema deve se recuperar automaticamente de falhas transitórias em até 30 segundos
  - Componentes devem reiniciar automaticamente após falhas críticas
  - Algoritmo de retry com exponential backoff para operações externas

#### 1.3 Redundância
- **Infraestrutura**: 
  - Deploy em múltiplas zonas de disponibilidade quando possível
  - Balanceamento de carga entre instâncias do serviço
  - Banco de dados com replicação síncrona/em espera quente

- **Armazenamento**: 
  - Backup diário automático dos dados de auditoria
  - Replicação geográfica para recuperação de desastres
  - Armazenamento redundante para logs críticos

### 2. Requisitos de Confiabilidade

#### 2.1 Integridade dos Dados
- **Consistência Transacional**: 
  - Todas as operações de aprovação/rejeição de changesets devem ser atomicamente registradas no sistema de auditoria
  - Estado consistente entre filas de processamento e registros de auditoria

- **Verificação de Integridade**: 
  - Checksums e hashes para changesets processados
  - Validação de integridade em tempo de leitura para dados críticos
  - Mecanismos de detecção e correção de inconsistências

#### 2.2 Precisão do Processo de Governaça
- **Taxa de Acerto em Aprovações Automáticas**: 
  - Mínimo de 95% de aprovações automáticas corretas em comparação com revisão manual
  - Taxa de falsos positivos em detecção de exceções < 2%

- **Reprodutibilidade**: 
  - Mesmo changeset deve sempre produzir o mesmo resultado quando as regras não mudarem
  - Ambientes de execução isolados para evitar interferências

#### 2.3 Robustez
- **Tratamento de Entradas Inválidas**: 
  - Sistema deve lidar graceful degradation com changesets malformados
  - Erros específicos e informativos para diferentes tipos de falha
  - Validação rigorosa de schemas do Liquibase antes do processamento

- **Limites de Operação**: 
  - Sistema deve rejeitar gracefully changesets acima de limite máximo de tamanho (10MB)
  - Proteção contra processamento infinito com timeouts configuráveis

### 3. Plano de Recuperação de Desastres

#### 3.1 RTO e RPO
- **Recovery Time Objective (RTO)**: 4 horas para restauração completa do serviço
- **Recovery Point Objective (RPO)**: 1 hora de perda máxima de dados aceitável

#### 3.2 Estratégias de Backup
- **Backup Completo**: Realizado semanalmente com retenção de 4 semanas
- **Backup Incremental**: Realizado diariamente com retenção de 30 dias
- **Backup de Configuração**: Versionado continuamente no repositório de código

#### 3.3 Failover e Failback
- **Detecção Automática de Falhas**: Monitoramento contínuo de saúde dos componentes
- **Failover Transparente**: Transição automática para sistemas de backup sem interrupção perceptível
- **Procedimento de Failback**: Processo documentado e testado para retorno ao ambiente primário

### 4. Monitoramento e Health Checks

#### 4.1 Endpoints de Saúde
- **Health Check Básico**: Verificação de disponibilidade do serviço (HTTP 200 OK)
- **Health Check Profundo**: Validação de conectividade com dependências (banco de dados, filas)
- **Métricas em Tempo Real**: Exposição de métricas de performance via endpoint dedicado

#### 4.2 Alertas de Disponibilidade
- **Alerta Crítico**: Sistema indisponível por mais de 5 minutos consecutivos
- **Alerta Alto**: Tempo de resposta acima de 10 segundos para 95% das requisições
- **Alerta Médio**: Taxa de erro acima de 5% nas últimas 15 minutos