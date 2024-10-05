FROM flink:1.18

COPY smt.tar.gz /opt/flink/smt.tar.gz
COPY connectors/flink-connector-starrocks-1.2.9_flink-1.18.jar /opt/flink/lib/flink-connector-starrocks-1.2.9_flink-1.18.jar
COPY connectors/flink-sql-connector-mysql-cdc-3.0.1.jar /opt/flink/lib/flink-sql-connector-mysql-cdc-3.0.1.jar

RUN set -ex; \
    tar -xzf /opt/flink/smt.tar.gz;

RUN rm /opt/flink/smt.tar.gz

VOLUME /opt/flink/smt
VOLUME /opt/flink/lib