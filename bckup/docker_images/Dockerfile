FROM maven:3-jdk-8-alpine
WORKDIR /code
COPY ./Code /code
RUN mvn clean package

EXPOSE 8000
CMD ["mvn spring-boot:run"]

