# agent-cost-bench Java verification image — Eclipse Temurin JDK 17 + Maven + warmed ~/.m2.
# Base: Eclipse Temurin 17 (Adoptium), a cloud-neutral OpenJDK build from Docker
# Hub. Maven is installed on top from the Debian package repo.
FROM eclipse-temurin:17-jdk

# Install Maven.
RUN apt-get update \
 && apt-get install -y --no-install-recommends maven \
 && rm -rf /var/lib/apt/lists/*

# Warm the local Maven repo with JUnit + plugins + the Surefire test provider.
WORKDIR /warm
COPY warm-pom.xml ./pom.xml
RUN mkdir -p src/test/java/warm && \
    printf 'package warm;\nimport org.junit.jupiter.api.Test;\nclass WarmTest { @Test void ok() {} }\n' \
      > src/test/java/warm/WarmTest.java && \
    mvn -B -q test

WORKDIR /work
