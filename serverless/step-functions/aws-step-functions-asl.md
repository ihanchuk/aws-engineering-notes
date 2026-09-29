# ASL [Amazon State Language]

## Amazon States Language (ASL)

**Amazon States Language (ASL)** — это JSON-based язык, с помощью которого описывается логика **State Machine** в AWS Step Functions.

В ASL задаются:

- какие шаги выполняются;
- в каком порядке они запускаются;
- какие шаги могут выполняться параллельно;
- куда переходить после выполнения шага;
- что делать при ошибке;
- какие условия проверять;
- какие данные передавать между состояниями.

### Типы состояний

Каждое состояние в ASL имеет свой `Type`. Основные типы:

| Type | Назначение |
|---|---|
| `Task` | Выполнение работы: вызов Lambda, ECS, Batch или другого AWS-сервиса |
| `Choice` | Условное ветвление workflow |
| `Parallel` | Параллельное выполнение нескольких веток |
| `Map` | Обработка набора элементов с выполнением одной логики для каждого |
| `Pass` | Передача или преобразование данных без выполнения внешней операции |
| `Wait` | Ожидание определённого времени или момента |
| `Succeed` | Успешное завершение workflow |
| `Fail` | Завершение workflow с ошибкой |

### Task ###

`Task` используется для выполнения конкретной операции. Например, можно вызвать Lambda:

```json
{
  "ProcessOrder": {
    "Type": "Task",
    "Resource": "arn:aws:lambda:eu-central-1:123456789012:function:ProcessOrder",
    "End": true
  }
}
```

### Choice ###

Choice используется для условного ветвления workflow.

```json
{
  "CheckStatus": {
    "Type": "Choice",
    "Choices": [
      {
        "Variable": "$.status",
        "StringEquals": "COMPLETED",
        "Next": "Success"
      }
    ],
    "Default": "Failed"
  }
}
```

В данном случае Choice проверяет значение $.status и выбирает следующий шаг.

### Parallel ###

Parallel позволяет выполнять несколько веток одновременно.

```text

              ┌──→ GetDiscount ──────┐
              │                      │
Start ────────┼──→ GetShipmentInfo ──┼──→ Final
              │                      │
              └──→ GetStatus ────────┘
```

Workflow продолжит выполнение после завершения всех параллельных веток, если одна из них не завершит workflow ошибкой.

### Map ###

Map используется, когда одну и ту же операцию необходимо выполнить для каждого элемента коллекции.
```text
Orders
  │
  ▼
 Map
 ├── Order #1 → ProcessOrder
 ├── Order #2 → ProcessOrder
 ├── Order #3 → ProcessOrder
 └── Order #4 → ProcessOrder

```

Например, массив заказов может быть обработан одним и тем же Task для каждого элемента.
