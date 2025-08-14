# Guia Rápido: Usando LocalStack com AWS CLI

Este guia explica como configurar e testar serviços AWS localmente usando **LocalStack** via Docker e AWS CLI.

---

## 1. O que é o LocalStack?
LocalStack é uma ferramenta que simula serviços da AWS localmente, permitindo testar aplicações sem custo e sem afetar recursos reais na nuvem.

---

## 2. Subindo o ambiente com Docker Compose

Crie o arquivo `docker-compose.yml`:

```yaml
version: "3.8"

services:
  localstack:
    image: localstack/localstack:3.6
    container_name: localstack
    ports:
      - "4566:4566"  # Endpoint unificado para todos os serviços
      - "4571:4571"
    environment:
      - SERVICES=s3,sqs,lambda,cloudwatch,logs
      - DEBUG=1
      - LAMBDA_EXECUTOR=docker-reuse
      - LAMBDA_REMOVE_CONTAINERS=true
      - DATA_DIR=/tmp/localstack/data
      - DEFAULT_REGION=us-east-1
      - AWS_ACCESS_KEY_ID=test
      - AWS_SECRET_ACCESS_KEY=test
      - AWS_DEFAULT_REGION=us-east-1
      - LOCALSTACK_UI=1
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock" # Necessário para execução de Lambdas

networks:
  default:
    name: localstack-net
```
---

## 3. Execute
```
  docker compose up -d
