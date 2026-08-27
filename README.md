# 📘 AWS SAM — [Work in Progress] Serverless Application Model

## 📑 Оглавление
### Глава 0. AWS короткое интро
- Регионы, зоны доступности, эдж локации
- Политики безопасности AWS
- Cloud Formation
  - [Структура шаблона](cloud-formation-template-structure.md) 
  - [Пример шаблона](cloud-formation-templatexample.md)
  - [Базовые ресурсы](cloud-fromation-basic-resources.md)
### Глава 1. Шаблоны AWS SAM
- Общая структура шаблона  
- Параметры, ресурсы и выходные данные
- Serverless ресурсы
  - [Перечень](serverless-resources.md)   
- [Встроенные функции CloudFormation](template-funcs.md)
- Примеры и практические советы
  - [Lambda+API](example-lambda-plus-api.md) 

### Глава 2. Введение в Serverless
- Что такое Serverless и зачем он нужен  
- Обзор AWS Lambda  
- Как SAM упрощает работу с CloudFormation  

### Глава 3. Установка и настройка окружения
- Установка AWS CLI и SAM CLI  
- Настройка профиля и кредов  
- Проверка и локальный запуск
- SAM CLI
  - [Список комманд](sam-cli-commands.md) 

### Глава 4. Структура проекта SAM
- Обзор типичного проекта  
- `template.yaml` и `samconfig.toml`  
- Как работает `Transform: AWS::Serverless-2016-10-31`

### Глава 5. Работа с ресурсами
- [Определение Lambda-функций](lambda-definition.md)
  - Общие принципы работ Лямбд
  - [Lambda Managed Instances !!!!](lambda-managed-instances.md)
  - [Жизненный цикл](lambda-lifecycle.md)
  - [Контекст выполнения](lambda-execution-context.md)
  - [Слои ***](lambda-layers.md)
  - Конфигурация
    - [Concurency (конкурентность)](concurency.md)
      - [Provisioned concurency](lambda-provisioned-concurency.md)
      - [Reserved Concurency](lambda-reserved-concurency.md)
    - Triggers   
  - Версии, альясы и энвайрмент и Canary релизы
  - Контейнеры
    - Cold vs Worm start
    - Контейнеризированная Лямбда
    - RIC vs RIE
    - [Пример](lambda-container-example.md)
    - [Типичные ошибки при работе с Лямбдой в Контейнере](labmda-container-mistakes.md)
  - [Лимиты ресурсов и ограничения лямбды](lambda-limits.md)
  - [Авторизация доступа посредством Лямбды](lambda-auth.md)
  - [Extensions !!!](lambda-extensions,md)
- API Gateway
  - [Общее описание](api-gateway.md)
    - [API эндпоинты](api-gateway-endpoints.md)
    - Rest Api vs HTTP Api   
    - Поддерживаемые сервисы
  - Интеграция
    - Интеграция с Лямбдой
      - [Lambda Proxy Integration] (lambda-proxy-integration.md) 
      - Типы интеграции
        - Mock (fake data)
        - AWS Proxy || Lambda Proxy (All original request & response data transfer)
        - HHTP_PROXY (when endpoint is NOT a Lambda)
        - AWS (Lambda get nit real request but template mapping and response to client is transformed)
        - HTTP (Same as AWS but not for AWS Services! Only for http endpoints. for integration with 3-d party services)
    - Интеграция с Aws Fargate
    - CORS
    - Open API
  - Auth
    - Lambda Authorizer
    - Cognito
  - Api Keys  
  - Api Gateway vs Load Balancer    
- Интеграция с DynamoDB, S3, SNS, SQS
- Брокеры Сообщений
  - SNS
    - Обшее описание
    - [Fanout паттерн](sns-fanout.md)
  - SQS
    - Общее описание
    - [Настройки очереди](sqs-params.md)
    - DLQ (Dead Letter Que)
  - Event Bridge
    - Назначение
    - Конфигурация
    - Сравнение с SQS и SNS
    - Примеры
      - [Custom Events example](aws-eb-custom-event.md)  

### Глава 6. Безопасность и IAM
- Минимизация прав Lambda  
- Примеры ролей и политик  
- Использование `Policies:` в SAM  

### Глава 7. Локальное тестирование
- `sam local invoke`  
- `sam local start-api`  
- Использование Docker для эмуляции окружения  

### Глава 8. CI/CD и деплой
- `sam build` и `sam deploy`  
- Интеграция с GitHub Actions  
- Отладка ошибок при деплое  

### Глава 9. Расширенные темы
- Layers и Reuse кода  
- Custom Resources  
- Nested Stacks и экспорт значений  

### Глава 10. Практика
- Разворачиваем REST API  
- Интеграция с Aurora Serverless  
- Событийная архитектура (EventBridge, SNS, SQS)

### Глава 11. Отладка и мониторинг
- CloudWatch Logs  
- X-Ray и tracing  
- Ошибки и перезапуски  

### Глава 12. Часто встречающиеся ошибки и best practices
- Типовые проблемы при сборке  
- Как избегать hardcoding  
- Лучшие практики SAM  

---

📘 **Дополнительно**
- [AWS SAM официальная документация](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)  
- [AWS Workshops по SAM](https://catalog.workshops.aws/serverless)
