# Microservices Kafka Lab

Event-driven microservices lab using Spring Boot, Spring Cloud Gateway (MVC), Apache Kafka (KRaft), and Docker Compose.

## Architecture

- `api-gateway` (port `8080`): routes `/orders` requests to order service
- `order-service` (port `8081`): REST producer, publishes to Kafka topic `order-topic`
- `inventory-service` (port `8082`): Kafka consumer (`inventory-group`)
- `billing-service` (port `8083`): Kafka consumer (`billing-group`)
- `kafka` (port `9092`): Confluent Kafka single-node KRaft broker

## Tech Stack

- Java 17
- Spring Boot `4.0.3`
- Spring Cloud `2025.1.0`
- Maven Wrapper (`mvnw` / `mvnw.cmd`)
- Docker Compose + `confluentinc/cp-kafka:7.6.0`

## Project Structure

```text
microservices-kafka-lab/
|- api-gateway/
|- order-service/
|- inventory-service/
|- billing-service/
`- docker-compose.yml
```

## Prerequisites

- JDK 17
- Docker Desktop
- Maven (optional, wrapper is included)

## Configuration

### Kafka (Docker)

Kafka is configured in `docker-compose.yml` with KRaft mode and exposed on:

- `localhost:9092`

### Service Ports

- API Gateway: `8080`
- Order Service: `8081`
- Inventory Service: `8082`
- Billing Service: `8083`

### API Gateway Route (important)

This project uses Spring Cloud Gateway MVC (`spring-cloud-starter-gateway-server-webmvc`), so route properties must be under:

- `spring.cloud.gateway.server.webmvc.routes`

Current route:

- `Path=/orders,/orders/**` -> `http://localhost:8081`

## Run the System

### 1. Start Kafka

From project root:

```powershell
docker compose up -d
docker compose ps -a
```

### 2. Start Microservices

Open 4 terminals and run:

```powershell
cd order-service
.\mvnw spring-boot:run
```

```powershell
cd inventory-service
.\mvnw spring-boot:run
```

```powershell
cd billing-service
.\mvnw spring-boot:run
```

```powershell
cd api-gateway
.\mvnw spring-boot:run
```

## Test Flow

Send a POST request through API Gateway:

- URL: `http://localhost:8080/orders`
- Method: `POST`
- Body (raw JSON):

```json
{
  "orderId": "ORD-1001",
  "item": "Laptop",
  "quantity": 1
}
```

You can use Postman or:

```powershell
Invoke-RestMethod -Uri "http://localhost:8080/orders" -Method Post -ContentType "application/json" -Body '{"orderId":"ORD-1001","item":"Laptop","quantity":1}'
```

Expected response:

- `Order Created & Event Published`

Expected logs:

- Inventory Service: `Inventory updated for order: ...`
- Billing Service: `Invoice generated for order: ...`

## Common Issues

- `No static resource orders` from API Gateway:
  - Ensure gateway config is in `application.yml` under `spring.cloud.gateway.server.webmvc.routes`
  - Restart API Gateway after config changes
- Kafka container exits quickly:
  - Ensure `CLUSTER_ID` exists in `docker-compose.yml`
  - Check logs: `docker logs kafka --tail 100`

## Useful Commands

```powershell
docker compose down -v
docker compose up -d
docker logs kafka --tail 100
```

```powershell
cd api-gateway; .\mvnw test
cd order-service; .\mvnw test
cd inventory-service; .\mvnw test
cd billing-service; .\mvnw test
```
