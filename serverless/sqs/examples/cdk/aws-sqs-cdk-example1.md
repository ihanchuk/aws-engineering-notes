# SQS + Lambda example

```js
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as sqs from 'aws-cdk-lib/aws-sqs';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as lambdaEventSources from 'aws-cdk-lib/aws-lambda-event-sources';
import * as path from 'path';

export class SqsLambdaStack extends cdk.Stack {
  constructor(scope: Construct, id: string, props?: cdk.StackProps) {
    super(scope, id, props);

    // 1. Создаем очередь SQS
    const queue = new sqs.Queue(this, 'MyQueue', {
      visibilityTimeout: cdk.Duration.seconds(300), // Должно быть >= таймаута Лямбды
    });

    // 2. Создаем функцию Lambda
    const workerLambda = new lambda.Function(this, 'WorkerLambda', {
      runtime: lambda.Runtime.NODEJS_20_X,
      handler: 'index.handler',
      code: lambda.Code.fromAsset(path.join(__dirname, '../lambda')), // Путь к коду Лямбды
      timeout: cdk.Duration.seconds(30),
    });

    // 3. Подписываем Лямбду на очередь SQS (Добавляем триггер)
    workerLambda.addEventSource(new lambdaEventSources.SqsEventSource(queue, {
      batchSize: 10, // Лямбда будет получать до 10 сообщений за один вызов
    }));
  }
}
```
