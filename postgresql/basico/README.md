# Básico

Usar *psql* para conectar com o banco por ser mais simples, mas pode ser usado o pgAdmin.

```
psql -d postgres -h localhost
```

## Criando um novo banco

```
postgres=# CREATE DATABASE sql_101;
```

## Conectando com o banco

```
postgres=# \c sql_101
```

## Criando uma estrutra simples

```
CREATE TABLE public.consumer
(
    id_consumer character varying(32) NOT NULL,
    ds_name character varying(100),
    ds_document character varying(20),
    ds_country character(3),
    dt_created timestamp with time zone,
    dt_modified timestamp with time zone,
    CONSTRAINT pk_consumer PRIMARY KEY (id_consumer)
);

CREATE TYPE public."phone type" AS ENUM
    ('phone', 'cellphone', 'smartphone', 'fax');

CREATE TABLE public.consumer_phone
(
    id_phone bigint NOT NULL DEFAULT nextval('consumer_phone_id_phone_seq'::regclass),
    id_consumer character varying(32) NOT NULL,
    no_phone character varying(12) COLLATE pg_catalog."default",
    tp_phone "phone type",
    dt_creation timestamp with time zone,
    CONSTRAINT pk_phone PRIMARY KEY (id_phone)
);

ALTER TABLE public.consumer_phone
    ADD CONSTRAINT fk_consumer FOREIGN KEY (id_consumer_fk)
    REFERENCES public.consumer (id_consumer) MATCH SIMPLE
    ON UPDATE CASCADE
    ON DELETE CASCADE;

CREATE TABLE public.consumer_order
(
    id_consumer_order character varying(32) COLLATE pg_catalog."default" NOT NULL,
    id_consumer_fk character varying(32) COLLATE pg_catalog."default" NOT NULL,
    ds_consumer_order character varying(100) COLLATE pg_catalog."default",
    dt_creation timestamp with time zone,
    CONSTRAINT pk_consumer_order PRIMARY KEY (id_consumer_order),
    CONSTRAINT fk_consumer FOREIGN KEY (id_consumer_order)
        REFERENCES public.consumer (id_consumer) MATCH SIMPLE
        ON UPDATE CASCADE
        ON DELETE CASCADE
        NOT VALID
)

```