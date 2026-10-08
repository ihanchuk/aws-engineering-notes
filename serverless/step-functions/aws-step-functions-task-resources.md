# AWS Step Functions — Таски процесс обработки состояния

# Task

Состояние `Task` выполняет определенную работу.

```json
{
  "Type": "Task",
  "Resource": "...",
  "Parameters": {},
  "Next": "Next State"
}
```

## Resource

Определяет службу или операцию для вызова.

### Lambda

```json
"Resource": "arn:aws:states:::lambda:invoke"
```

### SNS

```json
"Resource": "arn:aws:states:::sns:publish"
```

### SQS

```json
"Resource": "arn:aws:states:::sqs:sendMessage"
```

### DynamoDB

```json
"Resource": "arn:aws:states:::dynamodb:getItem"
```

---

# Data Processing

Поля, используемые для фильтрации и манипулирования данными при их прохождении через состояние:

```text
Input     ->  [ InputPath ]
                 ↓
              [ Parameters ]
                 ↓
              [ (State Execution) ]
                 ↓
              [ ResultSelector ]
                 ↓
              [ ResultPath ]
                 ↓
              [ OutputPath ]  -> Output
```

## Data Manipulation Fields

| Field            | Purpose                                                                                 | Example                   |
| :--------------- | :-------------------------------------------------------------------------------------- | :------------------------ |
| `InputPath`      | Выбирает часть входных данных для передачи в состояние.                                 | `"$.user"`                |
| `Parameters`     | Создает кастомный запрос (набор пар ключ-значение) для ресурса.                         | `{"Id.$": "$.userId"}`    |
| `ResultSelector` | Фильтрует чистый результат выполнения состояния перед применением `ResultPath`.         | `{"data.$": "$.Payload"}` |
| `ResultPath`     | Определяет, куда поместить результат в исходных данных состояния.                       | `"$.taskResult"`          |
| `OutputPath`     | Фильтрует финальные объединенные данные состояния перед передачей следующему состоянию. | `"$.taskResult"`          |

> 💡 **Примечание о суффиксе `.$`:** Любой ключ, заканчивающийся на `.$`, указывает Step Functions вычислять значение как **выражение JSONPath**, а не обрабатывать его как обычную строку.
