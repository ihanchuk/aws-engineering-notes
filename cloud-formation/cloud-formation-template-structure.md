| Секция | Подсекции / Возможности | Краткое описание | Обязательна? |
|--------|------------------------|-----------------|--------------|
| AWSTemplateFormatVersion | — | Версия формата шаблона (например "2010-09-09"). | Нет |
| Description | — | Текстовое описание шаблона, помогает документации. | Нет |
| Metadata | — | Произвольные метаданные шаблона (используются SAM, CDK, Designer, инструменты CI/CD). | Нет |
| Parameters | Type, Default, Description, AllowedValues, AllowedPattern, MinLength, MaxLength, MinValue, MaxValue, ConstraintDescription | Входные параметры для передачи значений при деплое. | Нет |
| Mappings | TopLevelKey → SecondLevelKey → Value | Статические таблицы соответствий, используются с `!FindInMap`. | Нет |
| Conditions | Создаются через функции: !If, !Equals, !And, !Or, !Not | Логические условия для управления созданием ресурсов или их свойств. | Нет |
| Transform | Имя трансформера (например `AWS::Serverless-2016-10-31`) | Подключение SAM или других макросов для расширенной функциональности. | Нет |
| Resources | Type, Properties, DependsOn, Metadata, DeletionPolicy, UpdateReplacePolicy | Основная секция с ресурсами AWS, которые создаются стеком. | Да |
| Outputs | Value, Description, Export, Condition | Выводимые значения после деплоя (например ARN, URL, имя ресурса). | Нет |
| Hooks | TypeName, Parameters | Предварительные и пост-деплойные проверки (CF Hooks). | Нет |
