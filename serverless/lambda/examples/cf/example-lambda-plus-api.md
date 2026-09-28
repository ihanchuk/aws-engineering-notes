# Пример на SAM + CloudFormation 

AWS SAM (Serverless Application Model) — это надстройка над AWS CloudFormation, которая упрощает описание serverless-инфраструктуры. 
В этом пример мы создадим Лямбду. 

Давайте расссмотрим характеристики нашей будуще Лямбды.

- __Lambda-функция__: HelloFunction
- __API__: HTTP API через API Gateway
- __Mаршрут__: GET /hello
- __Cвязи__: API → Lambda
- __Output__ с URL созданного API

### Пример:

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

### Разбор кода

```yaml
Globals:
  Function:
    Runtime: nodejs18.x
    Timeout: 10
    Handler: app.handler
```

#### Глобальные настройки Лямбды

Эти настройки автоматически задаются для всех Лямбд. Итак, все наши Лямбды будут иметь:

- __Runtime__: — Node.js 18.
- __Timeout__: — Lambda может выполняться максимум 10 секунд.
- __Handler__: app.handler — AWS ищет файл app.js и вызывает экспортированную функцию handler.

### Локальные настройки

```yaml
HelloFunction:
  Type: AWS::Serverless::Function
  Properties:
    CodeUri: src/
    Policies: AWSLambdaBasicExecutionRole
```

- `CodeUri: src/` - код берётся из src/;
- `AWSLambdaBasicExecutionRole` - Предопределенная самим АВС роль с правами для записи логов в CloudWatch.

#### Связь с API

```yaml
Events:
  Api:
    Type: HttpApi
    Properties:
      Path: /hello
      Method: GET
```

SAM автоматически создаёт HTTP API и связывает его с Lambda. В этом примере нет отдельного описания самого HTTP API.

#### Outputs

```yaml
Outputs:
  ApiEndpoint:
    Description: "HTTP API endpoint"
    Value: !Sub "https://${ServerlessHttpApi}.execute-api.${AWS::Region}.amazonaws.com"
```

`!Sub` подставляет значения переменных в строку:

- `${ServerlessHttpApi}` — созданный SAM HTTP API.
- `${AWS::Region} `— текущий AWS Region.

В результате получится URL примерно такого вида:

`https://abc123.execute-api.eu-central-1.amazonaws.com`
