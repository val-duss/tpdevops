FROM postgres:11.6-alpine

ENV 	POSTGRES_DB=db \
	POSTGRES_USER=usr \
	POSTGRES_PASSWORD=pwd \
	PGDATA=/var/lib/postgresql/data/pgdata

COPY 01-sqlschema.sql /docker-entrypoint-initdb.d/
COPY 02-sqlschema.sql /docker-entrypoint-initdb.d/
