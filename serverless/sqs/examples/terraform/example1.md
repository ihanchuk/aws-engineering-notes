# Пример Terraform: SQS + Lambda подписчик

Этот пример демонстрирует создание стандартной очереди SQS, функции AWS Lambda и настройку триггера, который автоматически вызывает Lambda при появлении сообщений в очереди.

## Код Terraform (`main.tf`)

```hcl
provider "aws" {
  region = "us-east-1" # Замените на ваш регион
}

# 1. Создаем SQS очередь
resource "aws_sqs_queue" "my_queue" {
  name                      = "my-lambda-trigger-queue"
  delay_seconds             = 0
  max_message_size          = 262144
  message_retention_seconds = 86400 # 1 день
  receive_wait_time_seconds = 20    # Включаем Long Polling
}

# 2. Создаем IAM роль для Lambda
resource "aws_iam_role" "lambda_role" {
  name = "sqs-lambda-execution-role"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      {
        Action = "sts:AssumeRole"
        Effect = "Allow"
        Principal = {
          Service = "://amazonaws.com"
        }
      }
    ]
  })
}

# 3. Политика для IAM роли (разрешаем логи и чтение из SQS)
resource "aws_iam_policy" "lambda_policy" {
  name        = "sqs-lambda-policy"
  description = "IAM policy for AWS Lambda to read from SQS and log to CloudWatch"

  policy = jsonencode({
    Version = "2012-10-17"
    Statement = [
      # Разрешение на логирование в CloudWatch
      {
        Effect = "Allow"
        Action = [
          "logs:CreateLogGroup",
          "logs:CreateLogStream",
          "logs:PutLogEvents"
        ]
        Resource = "arn:aws:logs:*:*:*"
      },
      # Разрешения для работы с SQS очередью
      {
        Effect = "Allow"
        Action = [
          "sqs:ReceiveMessage",
          "sqs:DeleteMessage",
          "sqs:GetQueueAttributes"
        ]
        Resource = aws_sqs_queue.my_queue.arn
      }
    ]
  })
}

# Привязываем политику к роли
resource "aws_iam_role_policy_attachment" "lambda_policy_attach" {
  role       = aws_iam_role.lambda_role.name
  policy_arn = aws_iam_policy.lambda_policy.arn
}

# 4. Подготавливаем архив с кодом функции Lambda (Node.js пример)
# Для работы этого блока создайте файл index.js рядом с main.tf
data "archive_file" "lambda_zip" {
  type        = "zip"
  source_file = "index.js"
  output_path = "lambda_function.zip"
}

# 5. Создаем саму AWS Lambda функцию
resource "aws_lambda_function" "my_lambda" {
  filename         = data.archive_file.lambda_zip.output_path
  function_name    = "sqs_message_processor"
  role             = aws_iam_role.lambda_role.arn
  handler          = "index.handler"
  source_code_hash = data.archive_file.lambda_zip.output_base64sha256
  runtime          = "nodejs18.x"

  # Таймаут должен быть меньше или равен Visibility Timeout у SQS
  timeout = 30 
}

# 6. Подписка (Event Source Mapping) — связываем SQS и Lambda
resource "aws_lambda_event_source_mapping" "sqs_trigger" {
  event_source_arn = aws_sqs_queue.my_queue.arn
  function_name    = aws_lambda_function.my_lambda.arn
  batch_size       = 10 # За раз Lambda может забирать до 10 сообщений
  enabled          = true
}
```

## Пример кода Lambda (`index.js`)

Создайте этот файл в той же папке, что и `main.tf`, перед запуском `terraform apply`:

```javascript
exports.handler = async (event) => {
    // В event.Records приходит массив сообщений (размером до batch_size)
    for (const record of event.Records) {
        console.log("Обработка сообщения ID:", record.messageId);
        console.log("Тело сообщения (Body):", record.body);
        
        // Ваша бизнес-логика здесь
    }
    return `Успешно обработано сообщений: ${event.Records.length}`;
};
```

---

### Важные нюансы конфигурации:

1. **Удаление сообщений:** Когда Lambda работает через `aws_lambda_event_source_mapping`, вам **не нужно вручную вызывать `DeleteMessage`** в коде функции. Если Lambda завершилась успешно (`status 200 / без ошибок`), AWS сама автоматически удалит эти сообщения из очереди.
2. **Таймаут видимости (Visibility Timeout):** По умолчанию у SQS он равен 30 секундам. Убедитесь, что параметр `timeout` у Lambda выставлен **меньше или равен** таймауту видимости очереди, иначе начнется параллельная обработка («зомби-сообщения»).
3. **Обработка ошибок:** Если выполнение Lambda падает с ошибкой (`throw Error`), сообщения из этого батча вернутся в очередь. Для защиты от бесконечных повторов рекомендуется добавить к ресурсу `aws_sqs_queue` блок `redrive_policy` для отправки битых сообщений в **DLQ (Dead Letter Queue)**.
