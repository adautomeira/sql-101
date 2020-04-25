# Desempenho

TODO

## Índices

TODO

### Índice por delimitados

TODO

## Particionamento

````sql
ALTER TABLE public.consumer_order
    RENAME TO consumer_order_old;
ALTER TABLE public.consumer_order_old
    RENAME CONSTRAINT pk_consumer_order TO pk_consumer_order_old;
ALTER TABLE public.consumer_order_old
    RENAME CONSTRAINT fk_consumer TO fk_consumer_old;
````

TODO

### Por id_consumer_fk

TODO

````sql
CREATE TABLE public.consumer_order_c
(
    id_consumer_order_c character varying(32) COLLATE pg_catalog."default" NOT NULL,
    id_consumer_fk character varying(32) COLLATE pg_catalog."default" NOT NULL,
    ds_consumer_order character varying(100) COLLATE pg_catalog."default",
    dt_creation timestamp with time zone,
    CONSTRAINT pk_consumer_order_c PRIMARY KEY (id_consumer_order_c, id_consumer_fk),
    CONSTRAINT fk_consumer_c FOREIGN KEY (id_consumer_fk)
        REFERENCES public.consumer (id_consumer) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
) PARTITION BY RANGE (id_consumer_fk);
````

### Por dt_creation

TODO

````sql
CREATE TABLE public.consumer_order_d
(
    id_consumer_order_d character varying(32) COLLATE pg_catalog."default" NOT NULL,
    id_consumer_fk character varying(32) COLLATE pg_catalog."default" NOT NULL,
    ds_consumer_order character varying(100) COLLATE pg_catalog."default",
    dt_creation timestamp with time zone,
    CONSTRAINT pk_consumer_order_d PRIMARY KEY (id_consumer_order_d, dt_creation),
    CONSTRAINT fk_consumer_d FOREIGN KEY (id_consumer_fk)
        REFERENCES public.consumer (id_consumer) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
) PARTITION BY RANGE (dt_creation);
````