# Segurança

TODO

## Grupos e Usuários

````sql
CREATE ROLE rl_admin WITH
  NOLOGIN
  NOSUPERUSER
  NOINHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

CREATE ROLE rl_super WITH
  NOLOGIN
  SUPERUSER
  NOINHERIT
  CREATEDB
  CREATEROLE
  NOREPLICATION;

CREATE ROLE rl_view WITH
  NOLOGIN
  NOSUPERUSER
  NOINHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;
````

````sql
CREATE ROLE superuser WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

GRANT rl_super TO superuser;

CREATE ROLE admin WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

GRANT rl_admin TO admin;

CREATE ROLE user1 WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

GRANT rl_view TO user1;

CREATE ROLE user2 WITH
  LOGIN
  NOSUPERUSER
  INHERIT
  NOCREATEDB
  NOCREATEROLE
  NOREPLICATION;

GRANT rl_view TO user2;
````

## Redaction

TODO