FROM maven AS builder
WORKDIR /opt/server
COPY pom.xml .
COPY src /opt/server/src
RUN mvn clean package

FROM eclipse-temurin:17-jre-alpine
EXPOSE 8080
WORKDIR /opt/server
# we need to create system user and 
RUN addgroup -S roboshop && adduser -S -G roboshop roboshop
ENV CART_ENDPOINT="cart:8080"\
    DB_HOST="mysql"
COPY --from=builder /opt/server/target/shipping-*.jar /opt/server/shipping.jar
CMD ["java", "-jar", "shipping.jar"] 