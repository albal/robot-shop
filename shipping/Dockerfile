###############################################################################
# 🛠  Build stage – Debian 10 + Maven, with bug workaround
###############################################################################
FROM debian:10 AS build

# Prevent any interactive prompts
ARG DEBIAN_FRONTEND=noninteractive

# ---------------------------------------------------------------------------
# Work-around for Debian-10 bug #1030129
#   • create /etc/ssl/certs/java        – the post-install hook expects it
#   • install ca-certificates-java FIRST
#   • THEN install Maven (brings in openjdk-11-jre-headless)
# ---------------------------------------------------------------------------
RUN set -e \
 && apt-get update \
 && mkdir -p /etc/ssl/certs/java \
 && apt-get install -y --no-install-recommends ca-certificates ca-certificates-java \
 && apt-get install -y --no-install-recommends maven \
 && rm -rf /var/lib/apt/lists/*

WORKDIR /opt/shipping

# 1️⃣  Warm Maven cache
COPY pom.xml .
RUN mvn -q dependency:go-offline

# 2️⃣  Compile the application
COPY src ./src
RUN mvn -q package -DskipTests    # drop -DskipTests if tests must run

###############################################################################
# 🚀  Runtime stage – same JDK family, slimmer image
###############################################################################
FROM openjdk:8-jdk-slim

WORKDIR /opt/shipping
EXPOSE 8080

ENV CART_ENDPOINT=cart:8080 \
    DB_HOST=mysql

COPY --from=build /opt/shipping/target/shipping-1.0.jar shipping.jar

# JSON-array form avoids a shell and satisfies linter warnings
ENTRYPOINT ["java", "-Xmn256m", "-Xmx768m", "-jar", "shipping.jar"]
