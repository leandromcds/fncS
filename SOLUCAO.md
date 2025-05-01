# Solução Definitiva para o Erro de Despesas Fixas

## Problema Identificado

O erro `Could not find the 'periodicidade' column of 'expenses' in the schema cache` está ocorrendo porque a tabela `expenses` no banco de dados Supabase não possui as colunas necessárias para armazenar informações das despesas fixas, que são:

- `fixa` (boolean): indica se a despesa é fixa ou não
- `pago` (boolean): indica se a despesa fixa foi paga
- `periodicidade` (enum): indica a periodicidade da despesa fixa (semanal, mensal ou anual)

## Solução Definitiva

Para resolver este problema de forma definitiva, é necessário atualizar o esquema do banco de dados no Supabase. Fornecemos dois scripts SQL para isso, escolha a opção que preferir:

### Opção 1: Script com verificação (recomendado)

O script `src/db/schema_update.sql` verifica se as colunas já existem antes de criá-las, o que torna o processo mais seguro:

```sql
-- Script para adicionar as colunas necessárias à tabela "expenses"
-- Execute este script no Editor SQL do Supabase

-- Primeiro, vamos verificar se as colunas já existem
DO $$
DECLARE
  fixa_exists BOOLEAN;
  pago_exists BOOLEAN;
  periodicidade_exists BOOLEAN;
  periodicidade_type_exists BOOLEAN;
BEGIN
  -- Verificar se as colunas existem
  SELECT EXISTS (
    SELECT FROM information_schema.columns 
    WHERE table_schema = 'public' AND table_name = 'expenses' AND column_name = 'fixa'
  ) INTO fixa_exists;
  
  SELECT EXISTS (
    SELECT FROM information_schema.columns 
    WHERE table_schema = 'public' AND table_name = 'expenses' AND column_name = 'pago'
  ) INTO pago_exists;
  
  SELECT EXISTS (
    SELECT FROM information_schema.columns 
    WHERE table_schema = 'public' AND table_name = 'expenses' AND column_name = 'periodicidade'
  ) INTO periodicidade_exists;
  
  -- Verificar se o tipo enum existe
  SELECT EXISTS (
    SELECT FROM pg_type 
    WHERE typname = 'periodicidade_type'
  ) INTO periodicidade_type_exists;
  
  -- Criar tipo enum se não existir
  IF NOT periodicidade_type_exists THEN
    EXECUTE 'CREATE TYPE periodicidade_type AS ENUM (''semanal'', ''mensal'', ''anual'')';
  END IF;
  
  -- Adicionar coluna fixa se não existir
  IF NOT fixa_exists THEN
    EXECUTE 'ALTER TABLE public.expenses ADD COLUMN fixa BOOLEAN DEFAULT false';
  END IF;
  
  -- Adicionar coluna pago se não existir
  IF NOT pago_exists THEN
    EXECUTE 'ALTER TABLE public.expenses ADD COLUMN pago BOOLEAN DEFAULT false';
  END IF;
  
  -- Adicionar coluna periodicidade se não existir
  IF NOT periodicidade_exists THEN
    EXECUTE 'ALTER TABLE public.expenses ADD COLUMN periodicidade periodicidade_type DEFAULT ''mensal''';
  END IF;
  
  -- Atualizar as permissões da API RESTful para incluir as novas colunas
  EXECUTE 'GRANT ALL ON public.expenses TO authenticated';
  EXECUTE 'GRANT ALL ON public.expenses TO service_role';
  
  RAISE NOTICE 'Atualização concluída com sucesso!';
END $$;
```

### Opção 2: Script simples e direto

O script `src/db/simple_update.sql` é mais simples e direto, mas pode gerar erros se as colunas ou tipos já existirem:

```sql
-- Script simples para adicionar as colunas necessárias à tabela "expenses"
-- Execute este script no Editor SQL do Supabase

-- Criar tipo enum para periodicidade
CREATE TYPE periodicidade_type AS ENUM ('semanal', 'mensal', 'anual');

-- Adicionar colunas à tabela expenses
ALTER TABLE public.expenses ADD COLUMN fixa BOOLEAN DEFAULT false;
ALTER TABLE public.expenses ADD COLUMN pago BOOLEAN DEFAULT false;
ALTER TABLE public.expenses ADD COLUMN periodicidade periodicidade_type DEFAULT 'mensal';

-- Atualizar permissões da API RESTful
GRANT ALL ON public.expenses TO authenticated;
GRANT ALL ON public.expenses TO service_role;
```

## Como executar a solução

1. Acesse o painel de controle do Supabase: https://supabase.com/dashboard
2. Selecione seu projeto (`hoekgixcyzyicxndsavm`)
3. No menu lateral, clique em "SQL Editor"
4. Clique em "New Query" (Nova Consulta)
5. Copie e cole o conteúdo de um dos scripts acima
6. Clique em "Run" (Executar)
7. Verifique se a execução foi concluída sem erros

## Verificação

Após executar o script, você pode verificar se as colunas foram adicionadas corretamente com a seguinte consulta SQL:

```sql
SELECT column_name, data_type, is_nullable 
FROM information_schema.columns 
WHERE table_schema = 'public' AND table_name = 'expenses'
ORDER BY ordinal_position;
```

Você deve ver as novas colunas `fixa`, `pago` e `periodicidade` na lista.

## Importante

Esta é uma solução definitiva que corrige o esquema do banco de dados para suportar despesas fixas. O código da aplicação já está preparado para usar estas colunas assim que elas existirem no banco de dados.

Caso você precise adicionar mais valores ao tipo de periodicidade no futuro, pode usar:

```sql
ALTER TYPE periodicidade_type ADD VALUE 'trimestral' AFTER 'mensal';
``` 