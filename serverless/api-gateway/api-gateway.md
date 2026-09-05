### Возможности AWS API Gateway

Amazon API Gateway — это полностью управляемый сервис, который помогает разработчикам легко создавать, публиковать, защищать и контролировать API любого масштаба. Через такие API приложения получают доступ к данным и бизнес-логике backend-сервисов. API Gateway позволяет создавать как REST, так и WebSocket API, что подходит и для обычных веб-приложений, и для сценариев с двусторонним обменом данными в реальном времени. Он одинаково хорошо работает с контейнерами, серверлесс-функциями и классическими приложениями.

Сервис берет на себя весь сложный функционал обработки большого потока запросов: управление трафиком, CORS, авторизацию, контроль доступа, rate limiting, мониторинг и версионирование. Минимальных затрат нет — вы платите только за количество обращений к API и объем переданных данных. По мере роста нагрузки действует многоуровневая модель цен, позволяющая снижать стоимость.


#### Основные функции

- Создание REST API
- Создание HTTP API
- Создание WebSocket API
- Импорт и экспорт OpenAPI (Swagger)
- Proxy-интеграция

#### Типы интеграций Backend
- AWS Lambda
- ECS/Fargate через ALB или NLB
- EC2 через ALB или NLB
- Любой HTTP endpoint
- AWS Services (SQS, SNS, Step Functions, DynamoDB, Kinesis, EventBridge, S3 и др.)

#### Безопасность и авторизация

- IAM Authorization
- Lambda Authorizer (Custom Authorizer)
- Cognito JWT Authorizer
- Resource Policies
- WAF интеграция
- mTLS

#### Управление трафиком

- Throttling
- Quotas per API key
- Rate limiting per stage и per method
- Stage variables
- Версионирование API (stages)
- Роутинг и трансформации
- Mapping templates (VTL)
- Body transformation
- Header transformation
- Query params remapping
- Response mapping templates
- Custom error mapping

#### Мониторинг и логирование
- CloudWatch Logs
- CloudWatch Metrics
- X-Ray tracing
- Access logs
- Execution logs
- Монетизация
- API Keys
- Usage Plans
- Request metering

#### Edge и кэширование
- API Gateway Cache
- Интеграция с CloudFront
- Regional и Edge-optimized endpoints

#### DevOps / CI/CD
- Поддержка SAM
- Поддержка CDK
- Поддержка CloudFormation
- Автоматический генератор SDK
- Import/Export OpenAPI
- Mock интеграции

#### Дополнительные возможности

- WebSocket state management
- CORS
- VPC Links
- Custom domains
- Deployment stages
- Поддержка бинарных payloads
