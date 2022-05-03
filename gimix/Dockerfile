#--------- Generic stuff all our Dockerfiles should start with so we get caching ------------
ARG IMAGE_VERSION=buster
ARG IMAGE_VARIANT=-slim
FROM debian:$IMAGE_VERSION$IMAGE_VARIANT

# Reset ARG for version
ARG IMAGE_VERSION

RUN set -eux \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get update \
    && apt-get -y --no-install-recommends install \
        locales gnupg2 wget ca-certificates rpl pwgen software-properties-common gdal-bin iputils-ping \
    && sh -c "echo \"deb http://apt.postgresql.org/pub/repos/apt/ ${IMAGE_VERSION}-pgdg main\" > /etc/apt/sources.list.d/pgdg.list" \
    && wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc -O- | apt-key add - \
    && apt-get -y --purge autoremove \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/* \
    && dpkg-divert --local --rename --add /sbin/initctl

#-------------Application Specific Stuff ----------------------------------------------------

# We add postgis as well to prevent build errors (that we dont see on local builds)
# on docker hub e.g.
# The following packages have unmet dependencies:
RUN set -eux \
    && export DEBIAN_FRONTEND=noninteractive \
    && apt-get update \
    && apt-get -y  install postgresql-11 \
    && apt-get -y  install netcat postgresql-11-postgis-2.5 postgresql-common \
    && apt-get -y install postgresql-11-pgrouting postgresql-11-ogr-fdw  \
         postgresql-plpython3-11 postgresql-11-cron python3-pip libpq-dev\
    && apt-get -y --purge autoremove  \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*
RUN apt-get remove -y postgresql-11-postgis-3 postgresql-11-postgis-3-scripts || true; \
    dpkg-query -W -f='${Status}' postgresql-11-postgis-2.5 | grep -xq 'install ok installed' && \
    dpkg-query -W -f='${Status}' postgresql-11-postgis-2.5-scripts | grep -xq 'install ok installed'
RUN pip3 install psycopg2
# Open port 5432 so linked containers can see them

USER postgres

RUN /etc/init.d/postgresql start && \
    psql --command "ALTER USER postgres WITH PASSWORD 'ssvtat2';" && \
    createdb -O postgres gimix && \
    psql -d gimix --command "CREATE EXTENSION postgis;" && \
    psql -d gimix --command "CREATE EXTENSION postgis_topology;"

RUN echo "host all  all    0.0.0.0/0  md5" >> /etc/postgresql/11/main/pg_hba.conf

RUN echo "listen_addresses='*'" >> /etc/postgresql/11/main/postgresql.conf

CMD ["/usr/lib/postgresql/11/bin/postgres", "-D", "/var/lib/postgresql/11/main", "-c", "config_file=/etc/postgresql/11/main/postgresql.conf"]
