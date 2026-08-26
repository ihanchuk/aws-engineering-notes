### SAM Policy Example

```yaml
Resources:

  MyQueue:
    Type: AWS::SQS::Queue

  MyTable:
    Type: AWS::DynamoDB::Table
    Properties:
      AttributeDefinitions:
        - AttributeName: id
          AttributeType: S
      KeySchema:
        - AttributeName: id
          KeyType: HASH
      BillingMode: PAY_PER_REQUEST

  MyBucket:
    Type: AWS::S3::Bucket

  MyFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handler.handler
      Runtime: nodejs22.x

      Policies:
        - SQSSendMessagePolicy:
            QueueName: !GetAtt MyQueue.QueueName

        - DynamoDBReadPolicy:
            TableName: !Ref MyTable

        - S3WritePolicy:
            BucketName: !Ref MyBucket
```
