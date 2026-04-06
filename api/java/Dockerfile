# Etapa 1: Construção do projeto com Maven e Java 23
FROM amazoncorretto:23 AS builder
WORKDIR /app

# Instalar Maven
RUN dnf install -y maven && dnf clean all

# Copiar o arquivo pom.xml e baixar as dependências
COPY pom.xml ./
RUN mvn dependency:go-offline -B

# Copiar o restante do código do projeto
COPY src ./src

# Fazer o build do projeto
RUN mvn clean package -DskipTests

# Etapa 2: Imagem final de produção com Java 23
FROM amazoncorretto:23
WORKDIR /app

COPY --from=builder /app/target/notification-0.0.1-SNAPSHOT.jar ./app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
