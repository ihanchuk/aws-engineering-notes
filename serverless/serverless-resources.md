### Serverless Resources

| Resource Type | Описание |
|---|---|
| `AWS::Serverless::Function` | Серверлесс-функция (обёртка над AWS Lambda). Поддерживает встроенные события (SQS, SNS, API Gateway, EventBridge, Cron). |
| `AWS::Serverless::Api` | API Gateway REST API с сокращённым синтаксисом (routes, auth, CORS). |
| `AWS::Serverless::HttpApi` | HTTP API (новое поколение API Gateway) с лёгкой конфигурацией. |
| `AWS::Serverless::StateMachine` | Step Functions state machine в YAML-сокращённом виде. |
| `AWS::Serverless::SimpleTable` | Упрощённое создание DynamoDB таблицы (ключ, чтение/запись). |
| `AWS::Serverless::LayerVersion` | Lambda Layer (упрощённый синтаксис). |
| `AWS::Serverless::Application` | Вложенное SAM-приложение (SAM nested stacks). |
| `AWS::Serverless::Connector` | Связи между ресурсами (например Function → DynamoDB). Упрощает IAM. |
| `AWS::Serverless::GlueJob` | Glue Job для ETL (serverless Spark/ETL). |

#### Дополнительное описание

### `AWS::Serverless::Function`
- Обёртка над Lambda.
- Автоматически создаёт IAM роли при необходимости.
- Поддерживает Events: SQS, SNS, Dynamo Streams, Kinesis, API, HttpApi, S3, EventBridge, IoT и др.
- Поддерживает AutoPublishAlias (канареечные деплои).
- Поддерживает inline code или указание CodeUri.

### `AWS::Serverless::Api`
- Создаёт REST API Gateway.
- Поддерживает implicit routes через event: Api в функциях.
- Может использовать OpenAPI внутри.
- Поддерживает Cognito/JWT/AWS_IAM авторизацию.
- Поддерживает CORS.

### `AWS::Serverless::HttpApi`
- Новый тип API Gateway (HTTP API).
- Лёгкая конфигурация.
- Поддерживает JWT authorizers.
- Более дешёвый и быстрый, чем REST API.

### `AWS::Serverless::StateMachine`
- Step Functions в сокращённом синтаксисе.
- Поддерживает встроенный IAM.
- Может использовать ASL (Amazon States Language) inline или из файлов.

### `AWS::Serverless::SimpleTable`
- Упрощённый DynamoDB.
- Создаёт:
  - PrimaryKey (Name + Type)
  - ProvisionedThroughput (по желанию)
  - GlobalSecondaryIndexes (в упрощённом виде)

### `AWS::Serverless::LayerVersion`
- Создаёт Lambda Layer.
- Упрощённый синтаксис по сравнению с `AWS::Lambda::LayerVersion`.

### `AWS::Serverless::Application`
- Позволяет импортировать сторонние SAM приложения (через S3 или SAR).
- Аналог nested stacks, но проще.

### `AWS::Serverless::Connector`
- Связывает ресурсы и автоматически создаёт IAM разрешения.
- Пример: Function → Queue, Function → Bucket.
- Поддерживает несколько шаблонов “connection”.
