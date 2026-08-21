FROM ubuntu:24.04

RUN apt update

RUN apt install curl gnupg -y

RUN curl -fsSL 'https://packages.clickhouse.com/rpm/lts/repodata/repomd.xml.key' | \
    gpg --dearmor -o /usr/share/keyrings/clickhouse-keyring.gpg

RUN echo "deb [signed-by=/usr/share/keyrings/clickhouse-keyring.gpg] \
    https://packages.clickhouse.com/deb stable main" | \
    tee /etc/apt/sources.list.d/clickhouse.list

WORKDIR /app

RUN apt update && apt upgrade -y

RUN apt install -y openjdk-17-jdk apt-transport-https wget postgresql-client-common postgresql-client-16 clickhouse-client zstd

RUN wget https://bao-products-builds.s3.us-east-1.amazonaws.com/analytics-platform/master/bao-data-pipeline.jar

RUN chmod +x bao-data-pipeline.jar

RUN addgroup --system --gid 1001 bao-admin
RUN adduser --system --uid 1001 bao-admin

USER bao-admin

EXPOSE 8084

ENV JAVA_OPTS="-Xms1024M -Xmx2048M"

ENV CACHE_DIR="/opt/bao-data-pipeline/data-pipeline"

CMD ["java", "-jar", "bao-data-pipeline.jar"]
