# AWS Step Functions — Базовая структура

```json
{
  "Comment": "Описание машины состояний",
  "StartAt": "First State",
  "TimeoutSeconds": 3600,

  "States": {
    "First State": {
      "Type": "Task",
      "Resource": "arn:aws:states:::lambda:invoke",

      "Parameters": {
        "FunctionName": "my-function",
        "Payload.$": "$"
      },

      "InputPath": "$",

      "ResultSelector": {
        "result.$": "$.Payload"
      },

      "ResultPath": "$.lambda",

      "OutputPath": "$",

      "Next": "Next State"
    },

    "Next State": {
      "Type": "Pass",
      "End": true
    }
  }
}
```

---

## State Machine

Поля верхнего уровня:

| Field            | Purpose                          |
| ---------------- | -------------------------------- |
| `Comment`        | Описание машины состояний        |
| `StartAt`        | Имя первого состояния            |
| `TimeoutSeconds` | Максимальное время выполнения    |
| `States`         | Все состояния в машине состояний |

`States` содержит отдельные состояния:

```text
Task
Pass
Choice
Wait
Parallel
Map
Succeed
Fail
```

---

## State

Обычно состояние выглядит так:

```json
"Имя состояния": {
  "Type": "Task",
  "Next": "Another State"
}
```

### Main State Types

| Type       | Purpose                        |
| ---------- | ------------------------------ |
| `Task`     | Выполнение работы              |
| `Pass`     | Передача или преобразование данных состояния |
| `Choice`   | Условное ветвление             |
| `Wait`     | Ожидание в течение определенного периода или времени |
| `Parallel` | Параллельный запуск веток      |
| `Map`      | Обработка нескольких элементов |
| `Succeed`  | Успешное завершение выполнения |
| `Fail`     | Завершение выполнения с ошибкой|
