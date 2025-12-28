FROM alpine:latest

RUN apk update

RUN apk add openjdk17 wget

WORKDIR /app

RUN wget -o bao-api-gateway.jar https://bao-products-builds.s3.us-east-1.amazonaws.com/dhis2-superset-gateway/master/dhis2-superset-gateway.jar


RUN chmod +x bao-api-gateway.jar

RUN addgroup --system --gid 1001 bao-admin
RUN adduser --system --uid 1001 bao-admin

RUN chown -R bao-admin:bao-admin /app

USER bao-admin

EXPOSE 8085

ENV JAVA_OPTS="-Xms256M -Xmx512M"

CMD ["java", "-jar", "bao-api-gateway.jar"]
