# Especificação de Detecção de linter-ignore-rule em Changesets Liquibase

## 1. Padrões de Detecção

### 1.1 Sintaxe do Comentário
Os comentários de ignore rule seguem estes padrões:

```
--linter-ignore-rule:<nome-da-regra>
```

Onde `<nome-da-regra>` corresponde às regras definidas no arquivo de configuração `liquibase-linter.json`.

### 1.2 Variações Aceitas
- `--linter-ignore-rule:<nome-da-regra>` (formato padrão)
- `--linter-ignore-all` (ignora todas as regras, uso não recomendado)
- Pode haver espaços antes do comentário, mas deve estar imediatamente após o cabeçalho do changeset

### 1.3 Exemplos Válidos
```sql
--liquibase formatted sql
--changeset autor:id
--linter-ignore-rule:no-drop-table
DROP TABLE tabela_temporaria;

--changeset autor:id
--  linter-ignore-rule:has-author
CREATE TABLE nova_tabela (id INT);
```

## 2. Contexto de Detecção

### 2.1 Localização Esperada
- Imediatamente após o cabeçalho do changeset (`--changeset`)
- Antes de quaisquer comandos SQL reais
- Dentro do mesmo changeset onde a regra será ignorada

### 2.2 Limitações
- O escopo é exclusivamente local ao changeset onde foi declarado
- Não afeta outros changesets no mesmo arquivo
- Não pode estar fora de um changeset

## 3. Extração de Informações Contextuais

### 3.1 Informações Obrigatórias
1. **Autor do Changeset**: Extraído do cabeçalho `--changeset autor:id`
2. **ID do Changeset**: Extraído do cabeçalho `--changeset autor:id`
3. **Nome da Regra Ignorada**: Extraído de `--linter-ignore-rule:nome-da-regra`
4. **Localização no Arquivo**: Número da linha onde o ignore foi encontrado
5. **Nome do Arquivo**: Caminho completo do arquivo do changeset

### 3.2 Informações Adicionais Úteis
1. **Timestamp de Detecção**: Quando a exceção foi identificada
2. **Motivo Presumido**: Análise heurística do contexto próximo
3. **Tipo de Changeset**: SQL, XML, YAML, etc.
4. **Comentários Adjacentes**: Outros comentários que possam fornecer contexto

## 4. Critérios de Classificação de Exceções

### 4.1 Tipos de Exceções
1. **Ignore Específico**: `--linter-ignore-rule:nome-da-regra`
2. **Ignore Geral**: `--linter-ignore-all` (alerta elevado)
3. **Ignore Mal Formado**: Comentários que se assemelham a ignore mas estão incorretos

### 4.2 Níveis de Severidade
1. **Alto**: Uso de `--linter-ignore-all`
2. **Médio**: Ignore de regras críticas como `no-drop-table`
3. **Baixo**: Ignore de regras não-críticas como convenções de nomenclatura

### 4.3 Categorias de Regras
1. **Segurança**: `no-drop-table`, `no-delete-all`, etc.
2. **Governança**: `has-author`, `has-id`, etc.
3. **Qualidade**: `naming-convention`, etc.
4. **Desempenho**: `no-unbounded-update`, etc.

## 5. Integração com Módulo de Validação Existente

### 5.1 Ponto de Integração
O detector de linter-ignore-rule deve se integrar como um componente no pipeline de validação, executado antes da validação real das regras.

### 5.2 Fluxo de Processamento
1. Parse do arquivo de changeset
2. Identificação de todos os changesets
3. Detecção de comentários linter-ignore-rule
4. Extração de informações contextuais
5. Classificação das exceções
6. Armazenamento para revisão e auditoria
7. Continuação do processo de validação normal