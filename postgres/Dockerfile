FROM postgres:10.7

ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

COPY zombodb_jessie_pg10-10-1.0.3_amd64.deb /
RUN apt-get update \
 && apt-get install -y libcurl4-openssl-dev \
 && apt-get install -y libcurl4-nss-dev \
 && apt-get install -y libcurl4-gnutls-dev \
 && dpkg -i zombodb_jessie_pg10-10-1.0.3_amd64.deb \
 && rm -f zombodb_jessie_pg10-10-1.0.3_amd64.deb

ENV POSTGIS_MAJOR 2.5
ENV POSTGIS_VERSION 2.5.2+dfsg-1~exp1.pgdg90+1

RUN apt-get update \
      && apt-cache showpkg postgresql-$PG_MAJOR-postgis-$POSTGIS_MAJOR \
      && apt-get install -y --no-install-recommends \
           postgresql-$PG_MAJOR-postgis-$POSTGIS_MAJOR=$POSTGIS_VERSION \
           postgresql-$PG_MAJOR-postgis-$POSTGIS_MAJOR-scripts=$POSTGIS_VERSION \
           postgis=$POSTGIS_VERSION \
      && rm -rf /var/lib/apt/lists/*

ENV CONF /usr/share/postgresql/postgresql.conf.sample

RUN sed -i 's/^#log_destination = '\''stderr'\''/log_destination = '\''stderr'\''/g' $CONF \
 && sed -i 's/^#logging_collector = off/logging_collector = on/g' $CONF \
 && sed -i 's/^#log_directory = '\''log'\''/log_directory = '\''\/log'\''/g' $CONF \
 #&& sed -i 's/^#log_filename/log_filename/g' $CONF \
 && sed -i 's/^#log_rotation_age = 1d/log_rotation_age = 1d/g' $CONF \
 && sed -i 's/^#log_rotation_size = 10MB/log_rotation_size = 100MB/g' $CONF \
 #&& sed -i 's/^#log_error_verbosity = default/log_error_verbosity = verbose/g' $CONF \
 && sed -i 's/^#log_statement = '\''none'\''/log_statement = '\''mod'\''/g' $CONF \
 && sed -i 's/^#log_lock_waits = off/log_lock_waits = on/g' $CONF \
 && sed -i 's/^#log_min_duration_statement = -1/log_min_duration_statement = 0/g' $CONF \
 && sed -i 's/^#log_checkpoints = off/log_checkpoints = on/g' $CONF \
 && sed -i 's/^#log_duration = off/log_duration = on/g' $CONF \
 && sed -i 's/^#log_min_messages = warning/log_min_messages = warning/g' $CONF \
 && sed -i 's/^#log_min_error_statement = error/log_min_error_statement = error/g' $CONF \
 && sed -i 's/^#log_line_prefix = '\''%m \[%p\] '\''/log_line_prefix = '\''%m \[%p\] %q%u@%d '\''/g' $CONF

# -c 'shared_buffers=2GB' -c 'max_connections=1000' -c 'work_mem=1GB'

RUN sed -i 's/^#wal_level = replica/wal_level = hot_standby/g' $CONF \
 && sed -i 's/^#max_wal_senders = 10/max_wal_senders = 32/g' $CONF \
 && sed -i 's/^#wal_keep_segments = 0/wal_keep_segments = 64/g' $CONF \
 && sed -i 's/^#wal_sender_timeout = 60s/wal_sender_timeout = 60s/g' $CONF \
 && sed -i 's/^#hot_standby = on/hot_standby = on/g' $CONF \
 && sed -i 's/^#max_standby_streaming_delay = 30s/max_standby_streaming_delay = 30s/g' $CONF \
 && sed -i 's/^#wal_receiver_status_interval = 10s/wal_receiver_status_interval = 10s/g' $CONF \
 && sed -i 's/^#hot_standby_feedback = off/hot_standby_feedback = on/g' $CONF \
 # archive_mode archive_command
 && sed -i 's/^#archive_mode = off/archive_mode = on/g' $CONF \
 && sed -i 's/^#archive_command = '\'''\''/archive_command = '\''cp %p \/var\/lib\/postgresql\/data\/pg_archive\/%f'\''/g' $CONF

COPY start.sh /
COPY init.sh /docker-entrypoint-initdb.d/
COPY recovery.conf /docker-entrypoint-initdb.d
COPY docker-entrypoint.sh /usr/local/bin/
COPY postgresql.ex.conf /docker-entrypoint-initdb.d/

COPY ./initdb-postgis.sh /docker-entrypoint-initdb.d/postgis.sh
COPY ./update-postgis.sh /usr/local/bin

RUN chown postgres:root /root \
 && chown postgres:root /var/spool/cron/crontabs \
 && chmod +x /usr/local/bin/docker-entrypoint.sh \
 && chmod +x /start.sh \
 && mkdir -p /log && chown postgres:postgres /log \
 && mkdir -p /backups && chown postgres:postgres /backups

ENV BACKUPS=/backups