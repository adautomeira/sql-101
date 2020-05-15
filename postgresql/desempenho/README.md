# Desempenho

Para fazermos alguns testes de desempenho, precisaremos de alguma carga considerável de dados.

````sql
DO $$
DECLARE
total integer := 200000;
unity integer;
BEGIN
    FOR unity IN 1..total LOOP
	    BEGIN
		    INSERT INTO consumer
		    VALUES (substring('PERSON:'||UPPER(MD5(random()::text)),1,32)
			     , MD5(random()::text) || ' ' || MD5(random()::text) || ' ' || MD5(random()::text)
			     , substring(MD5(random()::text),1,20)
			     , UPPER(substring(MD5(random()::text),1,3))
			     , (NOW() - '1 year'::INTERVAL * ROUND(RANDOM() * 100))::date);
		EXCEPTION
            WHEN unique_violation THEN
		END;
		COMMIT;
    END LOOP;
END$$;
````

Com a tabela consumer_phone teremos variabilidade e um volume maior de informações. Será útil para verificarmos a necessidade de índices.

````sql
INSERT INTO consumer_phone (id_consumer_fk
						  , no_phone
						  , tp_phone
						  , dt_creation)
	   (SELECT id_consumer
			         , SUBSTRING(floor(random()*(999999999999-100000000000+1))||'000',1,12)
			         , 'smartphone'
			         , (NOW() - '1 year'::INTERVAL * ROUND(RANDOM() * 100))::date
			   FROM consumer);
````

````sql
DO $$
DECLARE
BEGIN
	    BEGIN
		    INSERT INTO consumer_order_old (id_consumer_fk, id_consumer_order_old, ds_consumer_order, dt_creation)
		    (SELECT co.id_consumer
			         , substring('ORDER:'||UPPER(MD5((row_number() OVER (ORDER BY co.id_consumer) * random())::text)),1,32)
			         , substring(MD5(random()::text) || ' ' || MD5(random()::text) ||' '||MD5(random()::text) ||' '||MD5(random()::text),1,100)
			         , timestamp '2019-01-02 00:00:00' + (random() * (timestamp '2019-01-31 23:59:59' - timestamp '2019-01-02 00:00:00'))
			   FROM consumer co
			   JOIN (SELECT c.id_consumer, COUNT(coo.*) FROM consumer c LEFT JOIN consumer_order_old coo ON c.id_consumer = coo.id_consumer_fk GROUP BY c.id_consumer HAVING COUNT(coo.*) < 4)as p ON p.id_consumer = co.id_consumer);
		EXCEPTION
            WHEN unique_violation THEN
		END;
END$$;
````

## Índices

TODO

### Índice por valores delimitados

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



### Por id_consumer_fk

TODO

````sql
CREATE TABLE public.consumer_order_c
(
    id_consumer_order_c character varying(32) NOT NULL,
    id_consumer_fk character varying(32) NOT NULL,
    ds_consumer_order character varying(100),
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
    id_consumer_order_d character varying(32) NOT NULL,
    id_consumer_fk character varying(32) NOT NULL,
    ds_consumer_order character varying(100),
    dt_creation timestamp with time zone,
    CONSTRAINT pk_consumer_order_d PRIMARY KEY (id_consumer_order_d, dt_creation),
    CONSTRAINT fk_consumer_d FOREIGN KEY (id_consumer_fk)
        REFERENCES public.consumer (id_consumer) MATCH SIMPLE
        ON UPDATE NO ACTION
        ON DELETE NO ACTION
) PARTITION BY RANGE (dt_creation);
````