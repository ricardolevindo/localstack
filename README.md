# Guia Rápido: Usando LocalStack com AWS CLI

Este guia explica como configurar e testar serviços AWS localmente usando **LocalStack** via Docker e AWS CLI.

---

## 1. O que é o LocalStack?
LocalStack é uma ferramenta que simula serviços da AWS localmente, permitindo testar aplicações sem custo e sem afetar recursos reais na nuvem.

---

## 2. Subindo o ambiente com Docker Compose

Crie o arquivo `docker-compose.yml`:

```yaml
version: '3.8'
services:
  localstack:
    image: localstack/localstack:latest
    container_name: localstack
    ports:
      - "4566:4566" # Porta unificada para todos os serviços
    environment:
      - SERVICES=s3,sqs,lambda,cloudwatch
      - DEBUG=1
      - DATA_DIR=/var/lib/localstack/data
    volumes:
      - "./localstack-data:/var/lib/localstack"
      - "/var/run/docker.sock:/var/run/docker.sock"
