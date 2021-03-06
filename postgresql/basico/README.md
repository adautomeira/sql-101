# Básico

## Index

1. [Conectando no servidor](#conectando-no-servidor)
2. [Criando um novo banco](#criando-um-novo-banco)
3. [Conectando no banco](#conectando-no-banco)
4. [Schemas](#schemas)
5. [Tabelas](#tabelas)
6. [Relacionamentos](#relacionamentos)
7. [Índices](#índices)
8. [Funções](#funções)
9. [Consultas](#consultas)
10. [Clients](#clients)
11. [Migrations](#migrations)

Esse guia usará o _client_ *psql* para conectar com o banco por ser mais simples e ter alguns comandos a mais. No tópico [Clients](#clients) será sugerido algumas outras ferramentas gratuitas de conexão com o banco PostgreSQL.

## Conectando no servidor

```shell
psql -d postgres -h localhost
```

## Criando um novo banco

```shell
postgres=# CREATE DATABASE sql_101;
```

## Conectando no banco

```shell
postgres=# \c sql_101
```

## Schemas

No momento da criação de um novo banco, o PostgreSQL cria um schema com o nome _public_ com permissão de acesso a todos os usuários, por padrão.

```shell
sql_101=# \dn
```

|Name  |Owner   |
|------|--------|
|public|postgres|

```sql
CREATE SCHEMA app_main;
```

É possível modificar o proprietário (owner) do _schema_ usando o comando ALTER SCHEMA ... OWNER TO ...

```sql
ALTER SCHEMA app_main OWNER TO user1;
```

```shell
sql_101=# \dn
```

|Name    |Owner   |
|--------|--------|
|app_main|user1   |
|public  |postgres|

```shell
sql_101=# \dn
```

Did not find any relations

## Tabelas

```sql
CREATE TABLE app_main.consumer
(
    id_consumer character varying(32) NOT NULL,
    ds_name character varying(100),
    ds_document character varying(20),
    ds_country character(3),
    dt_created timestamp with time zone,
    dt_modified timestamp with time zone,
    CONSTRAINT pk_consumer PRIMARY KEY (id_consumer)
);

CREATE TYPE app_main."phone type" AS ENUM
    ('phone', 'cellphone', 'smartphone', 'fax');

CREATE TABLE app_main.consumer_phone
(
    id_phone bigint NOT NULL DEFAULT nextval('consumer_phone_id_phone_seq'::regclass),
    id_consumer character varying(32) NOT NULL,
    no_phone character varying(12),
    tp_phone "phone type",
    dt_creation timestamp with time zone,
    CONSTRAINT pk_phone PRIMARY KEY (id_phone)
);
```

## Relacionamentos

```sql
ALTER TABLE app_main.consumer_phone
    ADD CONSTRAINT fk_consumer FOREIGN KEY (id_consumer_fk)
    REFERENCES public.consumer (id_consumer) MATCH SIMPLE
    ON UPDATE CASCADE
    ON DELETE CASCADE;

CREATE TABLE app_main.consumer_order
(
    id_consumer_order character varying(32) NOT NULL,
    id_consumer_fk character varying(32) NOT NULL,
    ds_consumer_order character varying(100),
    dt_creation timestamp with time zone,
    CONSTRAINT pk_consumer_order PRIMARY KEY (id_consumer_order),
    CONSTRAINT fk_consumer FOREIGN KEY (id_consumer_order)
        REFERENCES public.consumer (id_consumer) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
        NOT VALID
)

```

## Índices

Criando um índice simples. A estrutura usada por padrão é a _btree_

```sql
CREATE INDEX consumer_phoneidx1 ON consumer_phone (no_phone);
```

Usando concurrently. Não gera lock para insert.

```sql
CREATE INDEX CONCURENTLY consumer_orderidx1 ON consumer_order (no_phone);
```

Usando funções na criação de index

```sql
CREATE INDEX CONCURENTLY consumeridx1 ON consumer (lower(ds_name));
```

Índices parciais

```sql
CREATE INDEX CONCURENTLY consumeridx2 ON consumer (id_consumer, ds_document) WHERE ds_country='BRA';
```

Criando colunas unique com regras

```sql
CREATE UNIQUE INDEX CONCURENTLY consumeridx2 ON consumer (ds_document) WHERE ds_country='BRA';
```

## Funções

Encapsular lógicas complexas ou repetitivas.

````sql
CREATE OR REPLACE FUNCTION 3chars(text character varying)
    RETURNS character varying AS $$
        BEGIN
            RETURN substring(text FOR 3);
        END;
$$ LANGUAGE plpgsql;
````

Usando o atributo _Security Definer_ é possível indicar o perfil acionador e, respectivamente, o grau de permissão que será usado na execução da função.

````sql
CREATE OR REPLACE FUNCTION customer_new_column(text character varying)
    SECURITY DEFINER
    RETURNS character varying AS $$
        BEGIN
            EXECUTE FORMAT('ALTER TABLE customer ADD COLUMN % character varying(255)','text');
        END;
$$ LANGUAGE plpgsql;
````

## Consultas

TODO

````sql

WITH sample as (
    SELECT '1234567890' as document
    UNION
    SELECT '0987654321' as document
)

SELECT *
FROM customer c
WHERE EXISTS (
    SELECT *
    FROM sample s
    WHERE c.ds_document = s.document
)

````

## Clients

[pgAdmin](https://www.pgadmin.org/)

[DBeaver](https://dbeaver.io/)

[HeidiSQL](https://www.heidisql.com/)

## Migrations

Fluxo automatizado para execução de scripts de banco de dados.

````sql
DO $BODY$
    DECLARE
        ds_column character varying;
    BEGIN

        -- Procurando por coluna 'nome_da_coluna' na tabela 'nome_da_tabela' no schema 'nome_do_schema'.

        SELECT column_name INTO ds_column
        FROM information_schema.columns
        WHERE table_schema = 'nome_do_schema' -- Não é obrigatório
          AND table_name = 'nome_da_tabela';  -- Não é obrigatório
          AND column_name = 'nome_da_coluna';

        -- Se a coluna não existir, faça...

        IF ds_column IS NULL THEN

            ALTER TABLE nome_da_tabela ADD COLUMN nome_da_coluna data_type;
            RAISE NOTICE 'nome_da_coluna criada com sucesso';

        -- Se a coluna já existir, faça...

        ELSE

            RAISE NOTICE 'A coluna ''nome_da_coluna'' já existe.';
            ds_column := NULL; -- Devolvendo o valor para nulo, caso seja necessário criar outras colunas.
        END IF;

        /* Aqui pode colocar outro conjunto de SELECT e IF*/

    END;
$BODY$
````