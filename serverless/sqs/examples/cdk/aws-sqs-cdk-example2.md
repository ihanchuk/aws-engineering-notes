# SQS + Lambda + DLQ

пример настройки, где сообщения, которые Lambda не смогла обработать несколько раз, автоматически отправляются в Dead Letter Queue (DLQ).В AWS CDK это делается через настройку свойства deadLetterQueue у основной очереди.

### Stack definition

```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as sqs from 'aws-cdk-lib/aws-sqs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as lambdaEventSources from 'aws-cdk-lib/aws-lambda-event-sources';
import * as path from 'path';

export class SqsDlqLambdaStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // 1. Создаем саму Мертвую Очередь (Dead Letter Queue)
    const deadLetterQueue = new sqs.Queue(this, 'MyDLQ', {
      retentionPeriod: cdk.Duration.days(14), // Храним «сломанные» сообщения дольше (макс. 14 дней)
    });

    // 2. Создаем Основную Очередь и связываем её с DLQ
    const mainQueue = new sqs.Queue(this, 'MainQueue', {
      visibilityTimeout: cdk.Duration.seconds(180), // 3 минуты (должно быть >= 6 * таймаут Лямбды)
      deadLetterQueue: {
        queue: deadLetterQueue,
        maxReceiveCount: 3, // Если Лямбда упадет 3 раза на одном сообщении, оно уйдет в DLQ
      },
    });

    // 3. Создаем функцию Lambda
    const workerLambda = new lambda.Function(this, 'WorkerLambda', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset(path.join(__dirname, '../lambda')),
      timeout: cdk.Duration.seconds(30), // Таймаут Лямбды — 30 секунд
    });

    // 4. Подписываем Лямбду на Основную Очередь
    workerLambda.addEventSource(new lambdaEventSources.SqsEventSource(mainQueue, {
      batchSize: 10,
    }));
  }
}
```

### Lambda handler

```js
exports.handler = async (event) => {
    for (const record of event.Records) {
        try {
            console.log("Обработка сообщения: ", record.body);
            
            // Имитация валидации данных
            const data = JSON.parse(record.body);
            if (!data.id) {
                throw new Error("Неверный формат данных: отсутствует ID");
            }
            
            // Успешная бизнес-логика...
            
        } catch (error) {
            console.error(`Ошибка при обработке сообщения ${record.messageId}:`, error);
            // Пробрасываем ошибку наружу, чтобы AWS Lambda зафейлила этот батч
            // и SQS вернул сообщение обратно в очередь для повторной попытки
            throw error;
        }
    }
};
```
