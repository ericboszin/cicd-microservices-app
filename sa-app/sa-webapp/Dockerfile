# Stage 1: Build App
FROM maven AS builder
COPY . /sa-webapp
WORKDIR /sa-webapp
RUN mvn install

# Instantiating env variable
ENV SA_LOGIC_API_URL http://localhost:5000

# Stage 2: Build Java App
FROM openjdk:8-jre-alpine
COPY --from=builder /sa-webapp/target/sentiment-analysis-web-0.0.1-SNAPSHOT.jar ./
EXPOSE 8080
CMD ["java", "-jar", "sentiment-analysis-web-0.0.1-SNAPSHOT.jar", "--sa.logic.api.url=${SA_LOGIC_API_URL}"]
