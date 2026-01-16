FROM alpine:latest

RUN apk update && apk upgrade

RUN apk add --no-cache openjdk17-jdk wget

WORKDIR /app

ENV PULSAR_VER="3.3.3"

RUN wget -O apache-pulsar.tar.gz https://archive.apache.org/dist/pulsar/pulsar-${PULSAR_VER}/apache-pulsar-${PULSAR_VER}-bin.tar.gz

RUN tar xvfz apache-pulsar.tar.gz

RUN mv apache-pulsar-${PULSAR_VER} apache-pulsar

RUN chmod +x apache-pulsar/bin/pulsar

EXPOSE 6650

CMD ["apache-pulsar/bin/pulsar", "standalone"]


