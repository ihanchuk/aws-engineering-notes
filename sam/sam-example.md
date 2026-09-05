# Структура шаблона AWS SAM (template.yaml)

Файл **`template.yaml`** — это сердце SAM-приложения.  
Он описывает ресурсы (Lambda, API, таблицы, роли и т.д.), их связи и конфигурацию деплоя.

---

## Общая структура

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: >
  Пример шаблона AWS SAM приложения

Globals:
  Function:
    Timeout: 10
    MemorySize: 128
    Runtime: nodejs20.x
    Tracing: Active
    Environment:
      Variables:
        LOG_LEVEL: info

Parameters:
  StageName:
    Type: String
    Default: dev
    Description: Имя окружения (dev, staging, prod)

Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-products.handler
      Description: Получение списка товаров
      Policies:
        - AWSLambdaBasicExecutionRole
      Events:
        GetProductsApi:
          Type: Api
          Properties:
            Path: /products
            Method: get

  CreateProductFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/create-product.handler
      Description: Создание товара
      Policies:
        - AWSLambdaBasicExecutionRole
      Events:
        CreateProductApi:
          Type: Api
          Properties:
            Path: /products
            Method: post

  ProductsTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      TableName: products
      PrimaryKey:
        Name: id
        Type: String
      ProvisionedThroughput:
        ReadCapacityUnits: 5
        WriteCapacityUnits: 5

Outputs:
  ApiUrl:
    Description: URL REST API
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/${StageName}/"
