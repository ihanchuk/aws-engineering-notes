# AWS Step Functions — Choice & Error Handling

# Choice

Логика условного ветвления на основе данных состояния.

```json
{
  "Type": "Choice",
  "Choices": [
    {
      "Variable": "$.status",
      "StringEquals": "SUCCESS",
      "Next": "ProcessSuccess"
    },
    {
      "Variable": "$.attempts",
      "NumericGreaterThan": 3,
      "Next": "FailState"
    }
  ],
  "Default": "AlternativeState"
}
```

## Common Choice Operators

- **String:** `StringEquals`, `StringLessThan`, `StringGreaterThan`, `StringMatches`
- **Numeric:** `NumericEquals`, `NumericLessThan`, `NumericGreaterThan`
- **Boolean:** `BooleanEquals`
- **Timestamp:** `TimestampEquals`, `TimestampLessThan`
- **Logical:** `And`, `Or`, `Not`

---

# Error Handling

Состояния типов `Task`, `Parallel` и `Map` могут перехватывать ошибки и повторять невыполненные операции.

```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::lambda:invoke",
  "Parameters": { "FunctionName": "my-function" },

  "Retry": [
    {
      "ErrorEquals": ["Lambda.ServiceException", "Lambda.AWSLambdaException"],
      "IntervalSeconds": 2,
      "MaxAttempts": 3,
      "BackoffRate": 2.0
    }
  ],

  "Catch": [
    {
      "ErrorEquals": ["States.ALL"],
      "Next": "FallbackState"
    }
  ],
  "End": true
}
```

## Configuration Breakdown

### Retry

- `ErrorEquals`: Список названий ошибок, которые запускают это правило повтора. Используйте `States.ALL` для перехвата любых ошибок.
- `IntervalSeconds`: Начальная задержка в секундах перед первой попыткой повтора.
- `MaxAttempts`: Общее количество попыток повтора перед остановкой.
- `BackoffRate`: Множитель, используемый для увеличения интервала между повторами каждый следующий раз.

### Catch

- `ErrorEquals`: Список названий ошибок для перехвата.
- `Next`: Состояние, в которое нужно перейти в случае возникновения ошибки.
- `ResultPath`: (Опционально) Куда именно внедрить детали ошибки в выходные данные состояния.
