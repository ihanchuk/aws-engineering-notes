# Конфигурации деплоя AWS SAM

AWS SAM поддерживает 9 предопределённых конфигураций для постепенного переключения трафика между версиями Lambda.

## 1. Canary-деплой

Canary-деплой направляет небольшую долю трафика на новую версию Lambda, ждёт заданное время, а затем переключает на неё весь трафик.

| Конфигурация | Начальная доля трафика | Время ожидания |
|---|---:|---:|
| `Canary10Percent5Minutes` | 10% | 5 минут |
| `Canary10Percent10Minutes` | 10% | 10 минут |
| `Canary10Percent15Minutes` | 10% | 15 минут |
| `Canary10Percent30Minutes` | 10% | 30 минут |

## 2. Линейный деплой

Линейный деплой постепенно переключает трафик на новую версию шагами по 10 процентных пунктов.

| Конфигурация | Увеличение доли трафика | Интервал |
|---|---:|---:|
| `Linear10PercentEvery1Minute` | 10% | 1 минута |
| `Linear10PercentEvery2Minutes` | 10% | 2 минуты |
| `Linear10PercentEvery3Minutes` | 10% | 3 минуты |
| `Linear10PercentEvery10Minutes` | 10% | 10 минут |

Примерная продолжительность переключения от первых 10% до 100% составляет соответственно 9, 18, 27 или 90 минут.

## 3. Мгновенный деплой

| Конфигурация | Поведение |
|---|---|
| `AllAtOnce` | Сразу переключает весь трафик на новую версию. |

## Пример конфигурации

```yaml
AutoPublishAlias: live
DeploymentPreference:
  Type: Canary10Percent5Minutes
```

- `AutoPublishAlias: live` — SAM публикует новую версию Lambda и управляет alias `live`.
- `Canary10Percent5Minutes` — сначала 10% трафика направляется на новую версию, а через пять минут — весь трафик, если развёртывание проходит успешно.

**Важно:** эти конфигурации управляют переключением трафика между версиями Lambda. Они не создают stage API Gateway, например `Prod` или `Stage`. Для canary- и линейного деплоя используется интеграция с AWS CodeDeploy.


```yaml

AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31
Description: API Gateway with canary Lambda deployment

Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello_world/
      Handler: app.lambda_handler
      Runtime: python3.11
      AutoPublishAlias: live
      DeploymentPreference:
        Type: Canary10Percent5Minutes
      Events:
        HelloWorld:
          Type: Api
          Properties:
            Path: /hello
            Method: get

Outputs:
  ApiUrl:
    Description: API Gateway endpoint
    Value: !Sub
      - "https://${ApiId}.execute-api.${AWS::Region}.${AWS::URLSuffix}/Prod/hello/"
      - ApiId: !Ref ServerlessRestApi

```
