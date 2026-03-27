# our base image
FROM postgres:17.2-alpine

# Use the virtual environment for all commands
# ENV POSTGRES_DB=db \
# POSTGRES_USER=usr \
# POSTGRES_PASSWORD=pwd

COPY ./sql /docker-entrypoint-initdb.d
