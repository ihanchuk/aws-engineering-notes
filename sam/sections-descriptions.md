# Основные секции шаблона AWS SAM (template.yaml)

Шаблон AWS SAM (`template.yaml`) основан на CloudFormation и расширяет его возможностями для безсерверных приложений.  
Ниже приведено описание всех основных секций, их назначение и примеры использования.

---

## 1. AWSTemplateFormatVersion

**Назначение:**  
Указывает версию синтаксиса CloudFormation.  
Для всех современных шаблонов используется фиксированное значение:

```yaml
AWSTemplateFormatVersion: '2010-09-09'
```

**Комментарий:**  
Эта строка не обязательна для SAM, но помогает обозначить совместимость с CloudFormation.

---

## 2. Transform

**Назначение:**  
Сообщает CloudFormation, что шаблон должен быть обработан трансформацией SAM.

```yaml
Transform: AWS::Serverless-2016-10-31
```

**Комментарий:**  
Без этой строки SAM-шаблон не будет распознан.  
Она включает поддержку `AWS::Serverless::*` ресурсов (Function, Api, SimpleTable и т.д.).

---

## 3. Description

**Назначение:**  
Краткое описание проекта или назначения стека.

```yaml
Description: >
  Приложение для управления товарами (AWS SAM пример)
```

**Комментарий:**  
Информационная секция — отображается в AWS Console в описании стека CloudFormation.

---

## 4. Globals

**Назначение:**  
Определяет **глобальные параметры**, применяемые ко всем ресурсам данного типа (например, Function или Api).  
Позволяет избежать дублирования конфигурации.

**Пример:**
```yaml
Globals:
  Function:
    Runtime: nodejs20.x
    Timeout: 5
    MemorySize: 256
    Environment:
      Variables:
        LOG_LEVEL: info
```

**Комментарий:**  
Переопределяется локально внутри конкретного ресурса, если заданы другие значения.

---

## 5. Parameters

**Назначение:**  
Позволяет задавать **входные параметры** для шаблона, которые можно менять при деплое (`sam deploy --parameter-overrides`).

**Пример:**
```yaml
Parameters:
  StageName:
    Type: String
    Default: dev
    Description: Имя окружения (dev/stage/prod)
  TableName:
    Type: String
    Default: products
```

**Комментарий:**  
Используется для настройки окружений, имён ресурсов и логики шаблона без его изменения.

---

## 6. Mappings

**Назначение:**  
Определяет статические таблицы значений (ключ-значение), которые можно использовать через функцию `!FindInMap`.

**Пример:**
```yaml
Mappings:
  RegionMap:
    us-east-1:
      AMI: ami-123
    eu-west-1:
      AMI: ami-456
```

**Комментарий:**  
Редко используется в SAM, но полезна, если нужно задавать разные параметры для разных регионов.

---

## 7. Conditions

**Назначение:**  
Позволяет создавать логические условия для включения/исключения ресурсов из шаблона.

**Пример:**
```yaml
Conditions:
  IsProd: !Equals [ !Ref StageName, prod ]

Resources:
  AnalyticsFunction:
    Type: AWS::Serverless::Function
    Condition: IsProd
```

**Комментарий:**  
Удобно, если одни ресурсы нужны только в определённой среде (например, в `prod`).

---

## 8. Resources

**Назначение:**  
Главный раздел — описывает **все создаваемые ресурсы**: Lambda-функции, API, таблицы, очереди и т.д.

**Пример:**
```yaml
Resources:
  GetProductsFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: src/handlers/get-products.handler
      Events:
        ApiEvent:
          Type: Api
          Properties:
            Path: /products
            Method: get

  ProductsTable:
    Type: AWS::Serverless::SimpleTable
    Properties:
      PrimaryKey:
        Name: id
        Type: String
```

**Комментарий:**  
Каждый ресурс имеет:
- `Type` — тип (например, `AWS::Serverless::Function`);
- `Properties` — набор свойств;
- `Events` — события-триггеры (API Gateway, S3, EventBridge и др.).

---

## 9. Outputs

**Назначение:**  
Определяет **значения, возвращаемые после деплоя**.  
Обычно сюда выносят URL API, имена таблиц, ARN функций и т.д.

**Пример:**
```yaml
Outputs:
  ApiUrl:
    Description: URL API Gateway
    Value: !Sub "https://${ServerlessRestApi}.execute-api.${AWS::Region}.amazonaws.com/${StageName}/"
  ProductsTableName:
    Description: Имя таблицы
    Value: !Ref ProductsTable
```

**Комментарий:**  
Эти значения отображаются в AWS Console и доступны для использования в других стеках через `!ImportValue`.

---

## 10. Metadata

**Назначение:**  
Дополнительная информация о шаблоне, не влияющая на деплой.

**Пример:**
```yaml
Metadata:
  License: MIT
  Version: 1.0.0
```

**Комментарий:**  
Можно использовать для аннотаций, CI/CD или внутренних инструментов.

---

## 11. Outputs (подробно)

**Применение:**  
- Для отладки и отображения информации о развернутом окружении.
- Для передачи значений в другие CloudFormation-стеки.

**Дополнительно:**  
Можно экспортировать значения:
```yaml
Outputs:
  TableName:
    Value: !Ref ProductsTable
    Export:
      Name: ProductsTableName
```

---

## 12. Globals (дополнение)

**Типы ресурсов, поддерживающие Globals:**
- `Function`
- `Api`
- `HttpApi`
- `SimpleTable`
- `StateMachine`

---

## 13. Итого

| Секция | Назначение |
|--------|-------------|
| **AWSTemplateFormatVersion** | Версия формата CloudFormation |
| **Transform** | Подключает обработку SAM |
| **Description** | Краткое описание проекта |
| **Globals** | Глобальные настройки для ресурсов |
| **Parameters** | Параметры, задаваемые при деплое |
| **Mappings** | Статические таблицы значений |
| **Conditions** | Условия для создания ресурсов |
| **Resources** | Определение всех AWS-ресурсов |
| **Outputs** | Значения, экспортируемые после деплоя |
| **Metadata** | Дополнительные сведения о шаблоне |

---

**Вывод:**  
Шаблон SAM объединяет в себе всю инфраструктуру и конфигурацию приложения в одном YAML-файле.  
Он компилируется в CloudFormation и обеспечивает **повторяемость, управляемость и безопасность деплоя**.
