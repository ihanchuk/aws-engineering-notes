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
