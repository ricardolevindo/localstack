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
      - SERVICES=s3,sqs,lambda,cloudwatch,logs,dynamodb,eventbridge
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

4. Instalando AWS CLI
  Linux
    sudo apt install awscli -y

    (método recomendado pela AWS)
      curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
      unzip awscliv2.zip
      sudo ./aws/install

      📌 Para confirmar a instalação:
      aws --version
      Saída esperada:
        aws-cli/2.x.x Python/3.x.x Linux/x86_64

macOS (Homebrew)
brew install awscli

Windows (Chocolatey)
choco install awscli

5. Configurando AWS CLI para LocalStack
aws configure
AWS Access Key ID: test
AWS Secret Access Key: test
Default region name: us-east-1
Default output format: json

💡 Essas credenciais são fictícias e válidas apenas para o LocalStack.


6. Comandos úteis para S3
Criar bucket
aws --endpoint-url=http://localhost:4566 s3 mb s3://meu-bucket

Listar buckets
aws --endpoint-url=http://localhost:4566 s3 ls

Enviar arquivo
aws --endpoint-url=http://localhost:4566 s3 cp arquivo.txt s3://meu-bucket/

Baixar arquivo
aws --endpoint-url=http://localhost:4566 s3 cp s3://meu-bucket/arquivo.txt .

6. Testando SQS
Criar fila
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name minha-fila

Listar filas
aws --endpoint-url=http://localhost:4566 sqs list-queues

7. Observações

Para evitar digitar --endpoint-url=http://localhost:4566 sempre, crie um perfil:

aws configure --profile localstack


E depois use:

aws --profile localstack s3 ls


A UI antiga do LocalStack (/_localstack/ui) não está mais disponível. Use ferramentas externas como LocalStack Cockpit (paga) ou scripts AWS CLI.


6. Testando SQS
Criar fila
aws --endpoint-url=http://localhost:4566 sqs create-queue --queue-name minha-fila

Listar filas
aws --endpoint-url=http://localhost:4566 sqs list-queues


7. Testando Lambda
7.1 Criar função Lambda de teste

Crie um arquivo lambda_function.py:

def handler(event, context):
    print("Evento recebido:", event)
    return {"statusCode": 200, "body": "Olá do Lambda Local!"}


Compacte em ZIP:

zip function.zip lambda_function.py

7.2 Criar a função no LocalStack
aws --endpoint-url=http://localhost:4566 lambda create-function \
  --function-name hello-lambda \
  --runtime python3.9 \
  --role arn:aws:iam::000000000000:role/lambda-execution-role \
  --handler lambda_function.handler \
  --zip-file fileb://function.zip

7.3 Invocar a função
aws --endpoint-url=http://localhost:4566 lambda invoke \
  --function-name hello-lambda \
  output.json
cat output.json

8. Testando CloudWatch

No LocalStack, o CloudWatch Logs é integrado automaticamente ao Lambda.

8.1 Listar grupos de logs
aws --endpoint-url=http://localhost:4566 logs describe-log-groups

8.2 Listar streams de logs
aws --endpoint-url=http://localhost:4566 logs describe-log-streams \
  --log-group-name /aws/lambda/hello-lambda

8.3 Ler eventos de log
aws --endpoint-url=http://localhost:4566 logs get-log-events \
  --log-group-name /aws/lambda/hello-lambda \
  --log-stream-name <NOME_DO_STREAM>

9. Dicas úteis

Para evitar digitar --endpoint-url=http://localhost:4566 sempre, crie um perfil:

aws configure --profile localstack


Depois use:

aws --profile localstack s3 ls


A UI antiga do LocalStack (/_localstack/ui) não está mais disponível.
Para interface gráfica, use:

LocalStack Cockpit (paga)

Scripts AWS CLI

Ferramentas como awslocal (pip install awscli-local)


```
# Exemplo NodeJS + TypeScript com LocalStack simulando:

  ### 1. API de entrada (Express) → recebe POST com dados do cliente → adiciona UUID curto → envia para SQS.
  ### 2. Lambda → disparada por evento da SQS → acrescenta dataHora → POSTa para outra API consumidora.
  ### 2. API consumidora → recebe POST → grava no DynamoDB.

```
localstack-node-ts/
├── docker-compose.yml
├── src/
│   ├── api-produtora.ts
│   ├── lambda-consumer.ts
│   ├── api-consumidora.ts
│   ├── dynamo.ts
├── package.json
├── tsconfig.json
└── scripts/
    ├── setup-resources.sh
```
1. docker-compose.yml
2. package.json
3. tsconfig.json
4. scripts/setup-resources.sh
5. src/api-produtora.ts — API que envia para SQS
6. src/lambda-consumer.ts — Lambda que lê da SQS e envia para API consumidora
7. src/api-consumidora.ts — API que grava no DynamoDB
8. Criando a Lambda no LocalStack


