# Promotions Infra — local development

This repo contains Docker Compose configuration for the infrastructure used by the Promotions project. It is intended for local development and integration testing.

## What is included
- cassandra_db — Apache Cassandra 4.1 (CQL port 9042). Provided and mounted to a named volume for local persistence.

Optional (commented out in docker-compose.yml; enable by uncommenting):
- kafka & zookeeper — Confluent images for Kafka development (requires Zookeeper for older Kafka images).
- redis_cache — Redis for caching / ephemeral storage.
- kafka_producer_api — Example API service for local integration with Kafka and Redis.

## Ports
- Cassandra: 9042 -> host:9042
- (If enabled) Kafka: 9092 -> host:9092
- (If enabled) Redis: 6379 -> host:6379
- (If enabled) API (example): 8080 -> host:8080

## Volumes
- cassandra_data — persisted Cassandra data
- redis_data — (commented) persisted Redis data

## Quick start

From this directory:

1. Bring up infra:
```bash
docker compose up -d
```

2. Watch Cassandra logs until it is ready:
```bash
docker compose logs -f cassandra_db
# or
docker logs -f cassandra_db
```

3. Connect to Cassandra (containerized cqlsh):
```bash
docker exec -it cassandra_db cqlsh
# inside cqlsh you can run: DESCRIBE KEYSPACES; CREATE KEYSPACE ...; etc.
```

4. Tear down (remove containers; keep volumes):
```bash
docker compose down
```

5. Tear down and remove volumes:
```bash
docker compose down -v
```

## How to connect your Spring Boot app

When running the Promotions API locally (outside Docker) you can point it at the Cassandra container. Example environment variables you can set when running the app:

```bash
# If your app is running on the host and Cassandra is in Docker
SPRING_DATA_CASSANDRA_CONTACT_POINTS=cassandra_db
SPRING_DATA_CASSANDRA_PORT=9042

# or with full Spring property names
spring.data.cassandra.contact-points=cassandra_db
spring.data.cassandra.port=9042
```

If you run your API inside Docker Compose (add the service), use the service name `cassandra_db` as the host. The provided docker-compose.yml contains an example `kafka_producer_api` service (commented) showing how to refer to other services by name.

## Enabling Kafka / Redis / API service
Coming Soon.

## Troubleshooting
- Cassandra can take a minute or two to become ready. Check logs (`docker compose logs -f cassandra_db`) and wait for the node to finish startup.
- If you get connection failures from your app, ensure you are using the correct host (service name inside Compose) and port.
- To inspect dependency images / volumes:
```bash
docker ps
docker volume ls
docker volume inspect promotions-infra_cassandra_data
```

## Notes for production
This repository is for local development. For production deployment, use IaC (CloudFormation / Terraform) to provision managed services — do not run production Cassandra in a single Docker instance. A future CFN / Terraform playbook may be added here.
