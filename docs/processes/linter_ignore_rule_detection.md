# Processo de Detecção de linter-ignore-rule

## Visão Geral

Este documento descreve o processo completo de detecção e tratamento de exceções `linter-ignore-rule` em changesets do Liquibase. O processo é projetado para identificar, classificar e gerenciar adequadamente situações onde desenvolvedores optam por ignorar regras específicas do Liquibase Linter.

## Objetivos

1. **Identificação Precisa**: Detectar corretamente todas as instâncias de `linter-ignore-rule` em changesets
2. **Classificação Adequada**: Categorizar exceções com base em critérios de severidade
3. **Integração Transparente**: Integrar-se suavemente com o processo de validação existente
4. **Auditoria Completa**: Manter registros detalhados de todas as exceções para fins de auditoria
5. **Controle de Acesso**: Implementar mecanismos apropriados de aprovação para exceções de alto risco

## Componentes Envolvidos

### Detector de Exceções
Responsável pela identificação inicial de comentários `linter-ignore-rule` nos changesets.

### Classificador de Exceções
Aplica critérios de classificação para determinar a severidade e categorização de cada exceção.

### Repositório de Exceções
Armazena e gerencia registros persistentes de todas as exceções detectadas.

### Integrador com Validação
Coordena a comunicação entre o processo de detecção de exceções e o motor de validação principal.

## Processo Detalhado

### 1. Inicialização
O processo é iniciado quando um arquivo de changeset é submetido para validação, geralmente como parte de um pipeline de CI/CD.

### 2. Parse do Changeset
O parser divide o arquivo em changesets individuais, identificando:
- Cabeçalhos de changeset (`--changeset autor:id`)
- Limites lógicos de cada changeset
- Estrutura geral do arquivo

### 3. Detecção de Exceções
Para cada changeset, o detector examina as linhas imediatamente após o cabeçalho em busca de padrões:

```sql
--linter-ignore-rule:<nome-da-regra>
--linter-ignore-all
```

### 4. Extração de Contexto
Quando uma exceção é detectada, as seguintes informações são extraídas:
- **Autor do Changeset**: De `--changeset autor:id`
- **ID do Changeset**: De `--changeset autor:id`
- **Nome da Regra**: Da declaração de ignore
- **Localização**: Número da linha no arquivo original
- **Timestamp**: Momento da detecção

### 5. Classificação da Exceção
Usando os critérios definidos em `exception_classification_criteria.md`, a exceção é classificada por:
- **Categoria da Regra**: Segurança, Governança, Qualidade, Desempenho
- **Severidade Base**: Baseada na criticidade da regra original
- **Severidade Calculada**: Ajustada por contexto e fatores ambientais

### 6. Armazenamento
A exceção é registrada no repositório com:
- Status inicial (normalmente "pendente")
- Metadados completos de contexto
- Informações de rastreabilidade

### 7. Integração com Validação
O sistema de validação é notificado sobre as exceções para:
- Ignorar as regras especificadas durante a validação
- Aplicar mecanismos de aprovação conforme necessário
- Continuar o processo de validação normal para regras não ignoradas

### 8. Relatório e Auditoria
Informações sobre exceções são registradas em:
- Logs especializados para auditoria
- Relatórios de exceções para revisão manual
- Métricas agregadas para monitoramento contínuo

## Padrões de Implementação

### Convenções de Nomenclatura
- Classes e funções seguem padrão `CamelCase`
- Variáveis e parâmetros seguem padrão `snake_case`
- Constantes são nomeadas em `UPPER_SNAKE_CASE`

### Estrutura de Arquivos
```
/exceptions/
├── detection/
│   ├── liquibase_ignore_detector.py
│   ├── context_extractor.py
│   └── pattern_matcher.py
├── classification/
│   ├── exception_classifier.py
│   └── severity_calculator.py
├── storage/
│   ├── exception_repository.py
│   └── exception_model.py
└── integration/
    ├── validation_adapter.py
    └── approval_coordinator.py
```

### Interfaces de Comunicação
Todas as interfaces entre componentes são explicitamente definidas usando contratos claros e tipagem estática quando possível.

## Considerações de Segurança

### Proteção contra Abuso
- Limite máximo de exceções por autor em um período
- Monitoramento de padrões suspeitos de uso
- Revisão obrigatória para certas categorias de exceções

### Privacidade de Dados
- Informações sensíveis são tratadas conforme políticas da organização
- Logs contêm apenas informações necessárias para auditoria
- Acesso ao repositório de exceções é controlado por permissões

## Monitoramento e Métricas

### Métricas em Tempo Real
- Número de exceções detectadas por hora
- Taxa de aprovação por categoria
- Tempo médio para resolução de exceções pendentes

### Alertas
- Exceções de alta severidade detectadas
- Padrões anômalos de uso de exceções
- Falhas no processo de detecção

### Relatórios
- Relatórios diários de atividade de exceções
- Análises mensais de tendências
- Audits completos sob demanda

## Resolução de Problemas

### Problemas Comuns

#### Padrões Não Reconhecidos
**Sintoma**: Exceções válidas não estão sendo detectadas  
**Solução**: Verificar padrões de regex configurados e atualizar conforme necessário

#### Falsos Positivos
**Sintoma**: Comentários que não são exceções estão sendo detectados  
**Solução**: Refinar regras de detecção e contexto

#### Integração com Validação Falhando
**Sintoma**: Regras ignoradas ainda estão causando falhas de validação  
**Solução**: Verificar comunicação entre componentes e ordem de execução

### Diagnóstico
1. Verificar logs de detecção para erros
2. Confirmar conectividade com repositório de exceções
3. Validar formato dos arquivos de changeset
4. Testar componentes isoladamente

## Evolução e Manutenção

### Atualizações de Padrões
Novos padrões de `linter-ignore-rule` devem ser adicionados através de:
1. Análise de necessidade de negócio
2. Atualização de padrões de detecção
3. Testes abrangentes
4. Documentação atualizada

### Expansão de Categorias
Novas categorias de regras exigem:
1. Definição de critérios de classificação
2. Atualização da lógica de severidade
3. Ajustes nos mecanismos de aprovação
4. Treinamento da equipe

## Casos de Uso Exemplares

### Exceção de Segurança Justificada
```sql
--changeset dba.operations:cleanup-temp-tables-v1
--linter-ignore-rule:no-drop-table
-- Justificativa: Remoção necessária de tabelas temporárias criadas durante processos batch
DROP TABLE temp_batch_processing_2023_01;
DROP TABLE temp_user_sessions_cleanup;
```

### Exceção de Governança para Ambiente Controlado
```sql
--changeset developer.intern:migration-setup-dev
--linter-ignore-rule:has-description
-- Desenvolvimento em progresso - descrição será adicionada antes do merge
CREATE INDEX idx_user_lookup ON users(username);
```

## Melhores Práticas

### Para Desenvolvedores
1. Sempre forneça justificativas claras para exceções
2. Use exceções específicas em vez de `linter-ignore-all`
3. Revise exceções durante processos de code review
4. Considere alternativas antes de usar exceções

### Para Revisores
1. Verifique a adequação da severidade calculada
2. Avalie se a justificativa é suficientemente detalhada
3. Considere contexto de ambiente e projeto
4. Documente decisões de aprovação/rejeição

### Para Administradores
1. Monitore métricas de uso de exceções
2. Revise periodicamente padrões de detecção
3. Atualize critérios de classificação conforme necessário
4. Mantenha documentação atualizada