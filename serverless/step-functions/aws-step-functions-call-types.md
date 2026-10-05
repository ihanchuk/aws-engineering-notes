### Режимы выполения запросов в Step Functions

| Pattern                 | Что делает                             | Пример                                                          |
| ----------------------- | -------------------------------------- | --------------------------------------------------------------- |
| **Request Response**    | Вызвать сервис и получить его ответ    | "Resource": "arn:aws:states:::sns:publish"                      |
| **`.sync`**             | Запустить работу и ждать её завершения | "Resource": "arn:aws:states:::ecs:runTask.sync"                 |
| **`.waitForTaskToken`** | Ждать callback с `TaskToken`           | "Resource": "arn:aws:states:::sqs:sendMessage.waitForTaskToken" |
