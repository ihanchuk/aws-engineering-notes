# AWS SAM policy templates

AWS Serverless Application Model (AWS SAM) позволяет выбирать из набора **policy templates**, чтобы ограничивать permissions ваших Lambda-функций и state machines AWS Step Functions ресурсами, которые используются приложением.

Приложения AWS SAM в AWS Serverless Application Repository, использующие policy templates, не требуют каких-либо специальных подтверждений со стороны пользователя для их развёртывания из AWS Serverless Application Repository.

Если вы хотите запросить добавление нового policy template:

1. Создайте pull request для исходного файла `policy_templates.json` в ветке `develop` проекта AWS SAM на GitHub.
2. Создайте issue в проекте AWS SAM, указав причины для pull request и ссылку на него.

## Syntax

Для каждого policy template, который указывается в AWS SAM template, необходимо передать объект со значениями его placeholders.

Если policy template **не требует placeholder values**, необходимо передать пустой объект `{}`.

### YAML

```yaml
MyFunction:
  Type: AWS::Serverless::Function
  Properties:
    Policies:
      - PolicyTemplateName1:        # Policy template с placeholder value
          Key1: Value1
      - PolicyTemplateName2: {}     # Policy template без placeholder values
```

> [!NOTE]
> Если вы используете обычную IAM policy или managed policies, настроенные через Lambda, policy можно указать без пустого объекта `{}`.

## Examples

### Example 1: Policy template с placeholder values

В следующем примере `SQSPollerPolicy` требует параметр `QueueName`, который указывает на используемый ресурс.

AWS SAM template получает имя очереди `MyQueue`. Эту очередь можно создать в том же приложении либо передать её имя в приложение в качестве параметра.

```yaml
MyFunction:
  Type: 'AWS::Serverless::Function'
  Properties:
    CodeUri: ${codeuri}
    Handler: hello.handler
    Runtime: python2.7
    Policies:
      - SQSPollerPolicy:
          QueueName:
            !GetAtt MyQueue.QueueName
```

### Example 2: Policy template без placeholder values

В следующем примере используется `CloudWatchPutMetricPolicy`, которому не требуются placeholder values.

> [!NOTE]
> Даже если policy template не имеет placeholder values, необходимо указать пустой объект `{}`. В противном случае возникнет ошибка.

```yaml
MyFunction:
  Type: 'AWS::Serverless::Function'
  Properties:
    CodeUri: ${codeuri}
    Handler: hello.handler
    Runtime: python2.7
    Policies:
      - CloudWatchPutMetricPolicy: {}
```

### Example 3: Policy template с placeholder values и обычной IAM policy

В этом примере одновременно используются `AmazonSQSFullAccess` и `DynamoDBCrudPolicy`.

`AmazonSQSFullAccess` — обычная IAM policy, а не AWS SAM policy template. Поэтому для неё не нужно указывать пустой объект: policy передаётся непосредственно в CloudFormation.

`DynamoDBCrudPolicy` — SAM-specific policy template с определённой структурой.

```yaml
MyFunction:
  Type: 'AWS::Serverless::Function'
  Properties:
    CodeUri: ${codeuri}
    Handler: hello.handler
    Runtime: python2.7
    Policies:
      - AmazonSQSFullAccess # IAM policy можно указать без пустого объекта
      - DynamoDBCrudPolicy: # SAM-specific policy с определённой структурой
          TableName:
            !Ref SampleTable
```

## Policy template table

Ниже перечислены доступные AWS SAM policy templates.

| Policy template                                  | Описание                                                                                                                                                          |
| ------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `AcmGetCertificatePolicy`                        | Даёт permission на чтение сертификата из AWS Certificate Manager.                                                                                                 |
| `AMIDescribePolicy`                              | Даёт permission на получение информации об Amazon Machine Images (AMI).                                                                                           |
| `AthenaQueryPolicy`                              | Даёт permissions на выполнение запросов Athena.                                                                                                                   |
| `AWSSecretsManagerGetSecretValuePolicy`          | Даёт permission на получение значения указанного secret из AWS Secrets Manager.                                                                                   |
| `AWSSecretsManagerRotationPolicy`                | Даёт permission на выполнение rotation secret в AWS Secrets Manager.                                                                                              |
| `CloudFormationDescribeStacksPolicy`             | Даёт permission на получение информации о CloudFormation stacks.                                                                                                  |
| `CloudWatchDashboardPolicy`                      | Даёт permissions для работы с CloudWatch dashboards и отправки metrics.                                                                                           |
| `CloudWatchDescribeAlarmHistoryPolicy`           | Даёт permission на получение истории CloudWatch alarms.                                                                                                           |
| `CloudWatchPutMetricPolicy`                      | Даёт permission на отправку metrics в CloudWatch.                                                                                                                 |
| `CodeCommitCrudPolicy`                           | Даёт permissions на создание, чтение, изменение и удаление объектов в указанном CodeCommit repository.                                                            |
| `CodeCommitReadPolicy`                           | Даёт permissions на чтение объектов в указанном CodeCommit repository.                                                                                            |
| `CodePipelineLambdaExecutionPolicy`              | Даёт Lambda-функции, вызываемой CodePipeline, permission сообщать статус job.                                                                                     |
| `CodePipelineReadOnlyPolicy`                     | Даёт read permission для получения информации о CodePipeline pipeline.                                                                                            |
| `ComprehendBasicAccessPolicy`                    | Даёт permissions для обнаружения entities, key phrases, languages и sentiments.                                                                                   |
| `CostExplorerReadOnlyPolicy`                     | Даёт read-only permissions для API Cost Explorer, связанных с историей billing.                                                                                   |
| `DynamoDBBackupFullAccessPolicy`                 | Даёт permissions на чтение и запись on-demand backups DynamoDB table.                                                                                             |
| `DynamoDBCrudPolicy`                             | Даёт permissions на создание, чтение, изменение и удаление данных DynamoDB table.                                                                                 |
| `DynamoDBReadPolicy`                             | Даёт read-only permission для DynamoDB table.                                                                                                                     |
| `DynamoDBReconfigurePolicy`                      | Даёт permission на изменение конфигурации DynamoDB table.                                                                                                         |
| `DynamoDBRestoreFromBackupPolicy`                | Даёт permission на восстановление DynamoDB table из backup.                                                                                                       |
| `DynamoDBStreamReadPolicy`                       | Даёт permissions на описание и чтение DynamoDB streams и records.                                                                                                 |
| `DynamoDBWritePolicy`                            | Даёт write-only permission для DynamoDB table.                                                                                                                    |
| `EC2CopyImagePolicy`                             | Даёт permission на копирование Amazon EC2 images.                                                                                                                 |
| `EC2DescribePolicy`                              | Даёт permission на получение информации об Amazon EC2 instances.                                                                                                  |
| `EcsRunTaskPolicy`                               | Даёт permission на запуск новой task для указанного task definition.                                                                                              |
| `EFSWriteAccessPolicy`                           | Даёт permission на подключение к Amazon EFS file system с правами записи.                                                                                         |
| `EKSDescribePolicy`                              | Даёт permissions на получение информации или списка Amazon EKS clusters.                                                                                          |
| `ElasticMapReduceAddJobFlowStepsPolicy`          | Даёт permission на добавление новых steps в работающий cluster.                                                                                                   |
| `ElasticMapReduceCancelStepsPolicy`              | Даёт permission на отмену ожидающего step или нескольких steps в работающем cluster.                                                                              |
| `ElasticMapReduceModifyInstanceFleetPolicy`      | Даёт permissions на получение информации и изменение capacity instance fleets внутри cluster.                                                                     |
| `ElasticMapReduceModifyInstanceGroupsPolicy`     | Даёт permissions на получение информации и изменение настроек instance groups внутри cluster.                                                                     |
| `ElasticMapReduceSetTerminationProtectionPolicy` | Даёт permission на установку termination protection для cluster.                                                                                                  |
| `ElasticMapReduceTerminateJobFlowsPolicy`        | Даёт permission на остановку cluster.                                                                                                                             |
| `ElasticsearchHttpPostPolicy`                    | Даёт POST permission для Amazon OpenSearch Service.                                                                                                               |
| `EventBridgePutEventsPolicy`                     | Даёт permissions на отправку events в EventBridge.                                                                                                                |
| `FilterLogEventsPolicy`                          | Даёт permission на фильтрацию CloudWatch Logs events из указанной log group.                                                                                      |
| `FirehoseCrudPolicy`                             | Даёт permissions на создание, запись, изменение и удаление Firehose delivery stream.                                                                              |
| `FirehoseWritePolicy`                            | Даёт permission на запись в Firehose delivery stream.                                                                                                             |
| `KinesisCrudPolicy`                              | Даёт permissions на создание, публикацию и удаление Amazon Kinesis stream.                                                                                        |
| `KinesisStreamReadPolicy`                        | Даёт permissions на получение списка и чтение Amazon Kinesis stream.                                                                                              |
| `KMSDecryptPolicy`                               | Даёт permission на расшифровку с использованием AWS KMS key.                                                                                                      |
| `KMSEncryptPolicy`                               | Даёт permission на шифрование с использованием AWS KMS key.                                                                                                       |
| `LambdaInvokePolicy`                             | Даёт permission на вызов AWS Lambda function, alias или version.                                                                                                  |
| `MobileAnalyticsWriteOnlyAccessPolicy`           | Даёт write-only permission для отправки event data для всех application resources.                                                                                |
| `OrganizationsListAccountsPolicy`                | Даёт read-only permission на получение списка имён и IDs дочерних AWS accounts.                                                                                   |
| `PinpointEndpointAccessPolicy`                   | Даёт permissions на получение и обновление endpoints для Amazon Pinpoint application.                                                                             |
| `PollyFullAccessPolicy`                          | Даёт полный доступ к ресурсам Amazon Polly lexicon.                                                                                                               |
| `RekognitionDetectOnlyPolicy`                    | Даёт permissions для обнаружения faces, labels и text.                                                                                                            |
| `RekognitionFacesManagementPolicy`               | Даёт permissions на добавление, удаление и поиск faces в Amazon Rekognition collection.                                                                           |
| `RekognitionFacesPolicy`                         | Даёт permissions на сравнение и обнаружение faces и labels.                                                                                                       |
| `RekognitionLabelsPolicy`                        | Даёт permissions на обнаружение object и moderation labels.                                                                                                       |
| `RekognitionNoDataAccessPolicy`                  | Даёт permissions на сравнение и обнаружение faces и labels.                                                                                                       |
| `RekognitionReadPolicy`                          | Даёт permissions на получение списка и поиск faces.                                                                                                               |
| `RekognitionWriteOnlyAccessPolicy`               | Даёт permissions на создание collection и индексирование faces.                                                                                                   |
| `Route53ChangeResourceRecordSetsPolicy`          | Даёт permission на изменение resource record sets в Route 53.                                                                                                     |
| `S3CrudPolicy`                                   | Даёт permissions на создание, чтение, изменение и удаление объектов в Amazon S3 bucket.                                                                           |
| `S3FullAccessPolicy`                             | Даёт полный доступ к объектам в Amazon S3 bucket.                                                                                                                 |
| `S3ReadPolicy`                                   | Даёт read-only permission на чтение объектов в Amazon S3 bucket.                                                                                                  |
| `S3WritePolicy`                                  | Даёт permission на запись объектов в Amazon S3 bucket.                                                                                                            |
| `SageMakerCreateEndpointConfigPolicy`            | Даёт permission на создание endpoint configuration в SageMaker AI.                                                                                                |
| `SageMakerCreateEndpointPolicy`                  | Даёт permission на создание endpoint в SageMaker AI.                                                                                                              |
| `ServerlessRepoReadWriteAccessPolicy`            | Даёт permissions на создание и получение списка applications в AWS Serverless Application Repository.                                                             |
| `SESBulkTemplatedCrudPolicy`                     | Даёт permissions на отправку email, templated email, templated bulk emails и verification identity.                                                               |
| `SESBulkTemplatedCrudPolicy_v2`                  | Даёт permissions на отправку Amazon SES email, templated email и templated bulk emails, а также verification identity.                                            |
| `SESCrudPolicy`                                  | Даёт permissions на отправку email и verification identity.                                                                                                       |
| `SESEmailTemplateCrudPolicy`                     | Даёт permissions на создание, получение, перечисление, изменение и удаление Amazon SES email templates.                                                           |
| `SESSendBouncePolicy`                            | Даёт `SendBounce` permission для Amazon SES identity.                                                                                                             |
| `SNSCrudPolicy`                                  | Даёт permissions на создание, публикацию и подписку на Amazon SNS topics.                                                                                         |
| `SNSPublishMessagePolicy`                        | Даёт permission на публикацию message в Amazon SNS topic.                                                                                                         |
| `SQSPollerPolicy`                                | Даёт permission на polling Amazon SQS queue.                                                                                                                      |
| `SQSSendMessagePolicy`                           | Даёт permission на отправку message в Amazon SQS queue.                                                                                                           |
| `SSMParameterReadPolicy`                         | Даёт permission на доступ к parameter в Amazon EC2 Systems Manager Parameter Store для загрузки secrets. Используется, когда имя parameter не имеет `/` в начале. |
| `SSMParameterWithSlashPrefixReadPolicy`          | Даёт permission на доступ к parameter в Amazon EC2 Systems Manager Parameter Store для загрузки secrets. Используется, когда имя parameter начинается с `/`.      |
| `StepFunctionsExecutionPolicy`                   | Даёт permission на запуск execution для Step Functions state machine.                                                                                             |
| `TextractDetectAnalyzePolicy`                    | Даёт permissions на обнаружение и анализ документов с помощью Amazon Textract.                                                                                    |
| `TextractGetResultPolicy`                        | Даёт permissions на получение результатов обнаружения и анализа документов из Amazon Textract.                                                                    |
| `TextractPolicy`                                 | Даёт полный доступ к Amazon Textract.                                                                                                                             |
| `VPCAccessPolicy`                                | Даёт permissions на создание, удаление, описание и отсоединение Elastic Network Interfaces.                                                                       |

## Troubleshooting

### SAM CLI error: `Must specify valid parameter values for policy template '<policy-template-name>'`

При выполнении:

```bash
sam build
```

может появиться ошибка:

```text
Must specify valid parameter values for policy template '<policy-template-name>'
```

Это означает, что при объявлении policy template, у которого нет placeholder values, не был передан пустой объект `{}`.

Например, для `CloudWatchPutMetricPolicy` необходимо написать:

```yaml
MyFunction:
  Policies:
    - CloudWatchPutMetricPolicy: {}
```

Если написать policy template без `{}`, SAM CLI не сможет корректно определить значения параметров и `sam build` завершится ошибкой.

## Ключевая идея

В AWS SAM есть два разных случая:

### SAM policy template

```yaml
Policies:
  - SQSPollerPolicy:
      QueueName: !GetAtt MyQueue.QueueName
```

SAM знает структуру `SQSPollerPolicy` и на основании указанных параметров генерирует соответствующие IAM permissions.

### Обычная IAM policy

```yaml
Policies:
  - AmazonSQSFullAccess
```

Это уже готовая IAM policy, поэтому SAM не требует специального объекта с параметрами.

### Policy template без параметров

```yaml
Policies:
  - CloudWatchPutMetricPolicy: {}
```

Пустой `{}` здесь обязателен.

> [!IMPORTANT]
> **Наличие `{}` зависит не от того, является ли policy «SAM policy» вообще, а от конкретного способа объявления policy template. Если SAM policy template не имеет placeholders, передавай `{}`.**
