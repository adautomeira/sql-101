# Básico

## Index

1. [Conectando no servidor](#conectando-no-servidor)
2. [Criando um novo banco](#criando-um-novo-banco)
3. [Conectando no banco](#conectando-no-banco)
4. [Schemas](#schemas)
4. [Tabelas](#tabelas)
5. [Relacionamentos](#relacionamentos)
6. [Índices](#índices)
7. [Funções](#funções)
7. [Consultas](#consultas)
8. [Clients](#clients)

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

Na criação de um novo banco o PostgreSQL cria um schema com o nome _public_ com permissão de acesso a todos os usuários, por padrão.

```shell
sql_101=# \dn
```

|Name  |Owner   |
|------|--------|
|public|postgres|

```sql
CREATE SCHEMA app_main AUTHORIZATION postgres;
```

```shell
sql_101=# \dn
```

|Name    |Owner   |
|--------|--------|
|app_main|postgres|
|public  |postgres|

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

TODO

## Funções

TODO

````sql
CREATE OR REPLACE FUNCTION 3chars(text character varying)
    RETURNS character varying AS $$
        BEGIN
            RETURN substring(text FOR 3);
        END;
$$ LANGUAGE plpgsql;
````

Usando o atributo _Security Definer_ é possível indicar o perfil e, respectivamente, o grau de permissão que será usado na execução da função.

EXECUTE

## Consultas

TODO

## Clients

TODO