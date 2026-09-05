### Краткий список комманд

| Команда | Что делает | Пример |
|--------|-------------|---------|
| `sam init` | Создаёт новый SAM-проект | `sam init --runtime nodejs20.x --name my-app` |
| `sam build` | Собирает функции и зависимости | `sam build` |
| `sam validate` | Проверяет шаблон на ошибки | `sam validate --lint` |
| `sam deploy` | Деплой в AWS (через CloudFormation) | `sam deploy --guided` |
| `sam delete` | Удаляет стек SAM | `sam delete --stack-name my-stack` |
| `sam local invoke` | Локальный вызов Lambda | `sam local invoke MyFunc -e event.json` |
| `sam local start-api` | Локальная эмуляция API Gateway | `sam local start-api` |
| `sam local start-lambda` | Локальная эмуляция Lambda Runtime API | `sam local start-lambda` |
| `sam local generate-event` | Генератор тестовых событий | `sam local generate-event s3 put` |
| `sam local start-dynamodb` | Запуск локальной DynamoDB | `sam local start-dynamodb` |
| `sam logs` | Просмотр CloudWatch логов | `sam logs -n MyFunc --tail` |
| `sam traces` | Просмотр X-Ray трейсинга | `sam traces --name MyFunc` |
| `sam sync` | Быстрая синхронизация кода | `sam sync --watch` |
| `sam package` | Пакует артефакты в S3 | `sam package --s3-bucket my-bucket --output-template packaged.yaml` |
| `sam publish` | Публикация в SAR | `sam publish --template packaged.yaml` |
| `sam pipeline init` | Генерация CI/CD пайплайна | `sam pipeline init` |
| `sam pipeline bootstrap` | Создаёт роли/бакеты для CI/CD | `sam pipeline bootstrap --region eu-central-1` |
| `sam pipeline deploy` | Деплой через пайплайн | `sam pipeline deploy` |
| `sam list endpoints` | Показывает публичные API URL | `sam list endpoints --stack-name my-stack` |
| `sam list resources` | Список ресурсов стека | `sam list resources --stack-name my-stack` |
