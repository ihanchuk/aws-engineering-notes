# Основные игроки AWS SAM

AWS SAM (Serverless Application Model) — это фреймворк, который упрощает создание, тестирование и деплой безсерверных приложений в AWS.  
Ниже — ключевые компоненты (или “игроки”), которые участвуют в работе SAM.

---

## 1. AWS SAM CLI

**Назначение:**  
CLI-инструмент для локальной разработки, тестирования и деплоя SAM-приложений.

**Возможности:**
- `sam init` — создание нового проекта.  
- `sam build` — сборка артефактов (кода, зависимостей).  
- `sam local invoke` / `sam local start-api` — локальное тестирование Lambda и API Gateway.  
- `sam deploy` — деплой приложения в AWS CloudFormation.

---

## 2. AWS CloudFormation

**Назначение:**  
SAM фактически является надстройкой над CloudFormation.  
При деплое SAM-шаблон транслируется в CloudFormation-шаблон.

**Роль:**
- Управление стеком инфраструктуры (создание/удаление ресурсов).
- Обеспечение идемпотентности при деплое.
- Поддержка параметров, маппингов и зависимостей ресурсов.

---

## 3. AWS Lambda

**Назначение:**  
Исполняет серверный код без управления серверами.

**В SAM:**  
Определяется через ресурс `AWS::Serverless::Function`.

**Пример:**
```yaml
Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: app.handler
      Runtime: nodejs20.x
      Events:
        Api:
          Type: Api
          Properties:
            Path: /products
            Method: get
```

---

## 4. Amazon API Gateway

**Назначение:**  
Предоставляет HTTP/REST/WebSocket API-интерфейсы для Lambda и других сервисов.

**В SAM:**  
Можно определить как:
- встроенное событие `Api` или `HttpApi` внутри `Function`,
- отдельный ресурс `AWS::Serverless::Api`.

**Особенности:**
- Поддержка CORS, авторизации, кастомных доменов.
- Синхронный триггер для Lambda.

---

## 5. Amazon EventBridge

**Назначение:**  
Асинхронная маршрутизация событий между сервисами (event bus).

**В SAM:**  
Используется через тип события `EventBridgeRule`.

**Пример:**
```yaml
Events:
  ProductCreated:
    Type: EventBridgeRule
    Properties:
      Pattern:
        source:
          - "app.products"
```

---

## 6. Amazon S3

**Назначение:**  
Хранилище исходников Lambda и артефактов сборки.

**В SAM:**  
Используется при `sam package` и `sam deploy` — код Lambda загружается в S3, откуда CloudFormation его разворачивает.

---

## 7. Amazon DynamoDB

**Назначение:**  
Безсерверная NoSQL база данных, часто используется совместно с SAM.

**Пример:**
- Lambda вызывает DynamoDB через AWS SDK.
- Или генерируется событие `DynamoDB` при изменении таблицы.

---

## 8. Amazon Step Functions

**Назначение:**  
Оркестрация выполнения нескольких функций Lambda в виде workflow.

**В SAM:**  
Определяется через `AWS::Serverless::StateMachine`.

---

## 9. AWS IAM

**Назначение:**  
Контролирует доступ между компонентами приложения.

**В SAM:**  
Для каждой Lambda можно задать роль через `Policies` или `Role`.

---

## 10. AWS X-Ray

**Назначение:**  
Трассировка вызовов и анализ производительности Lambda и API.

**Интеграция:**
```yaml
Tracing: Active
```

---

## 11. AWS CloudWatch

**Назначение:**  
Логирование и метрики.

**В SAM:**  
Все Lambda автоматически шлют логи в CloudWatch Logs, а метрики — в CloudWatch Metrics.

---

## 12. AWS Secrets Manager / Parameter Store

**Назначение:**  
Безопасное хранение конфигураций и секретов.

**Применение:**
- Lambda может получать значения во время выполнения.
- SAM позволяет подключать переменные окружения из SSM параметров.

---

## Итого

AWS SAM — это декларативная и автоматизируемая оболочка вокруг AWS-инфраструктуры для безсерверных приложений.  
Основная ценность — **простота разработки, тестирования и деплоя** в экосистеме AWS.
