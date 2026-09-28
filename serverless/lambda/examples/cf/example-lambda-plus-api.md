# Пример на SAM + CloudFormation 

AWS SAM (Serverless Application Model) — это надстройка над AWS CloudFormation, которая упрощает описание serverless-инфраструктуры. 
В этом пример мы создадим Лямбду. 

Давайте расссмотрим характеристики нашей будуще Лямбды.

- __Lambda-функция__: HelloFunction
- __API__: HTTP API через API Gateway
- __Mаршрут__: GET /hello
- __Cвязи__: API → Lambda
- __Output__ с URL созданного API


```yml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: Минимальный SAM-пример — Lambda + API

Globals:
  Function:
    Runtime: nodejs18.x
    Timeout: 10
    Handler: app.handler

Resources:
  HelloFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: src/
      Policies: AWSLambdaBasicExecutionRole
      Events:
        Api:
          Type: HttpApi     # HTTP API (v2)
          Properties:
            Path: /hello
            Method: GET

Outputs:
  ApiEndpoint:
    Description: "HTTP API endpoint"
    Value: !Sub "https://${ServerlessHttpApi}.execute-api.${AWS::Region}.amazonaws.com"
```
