FROM mysql/mysql-server:8.0

USER 0

COPY .docker/mysql/my.cnf /etc/my.cnf.d/
COPY .docker/mysql/create-testing-database.sh /docker-entrypoint-initdb.d/10-create-testing-database.sh
