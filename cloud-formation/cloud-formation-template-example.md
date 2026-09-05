#### Cloud Formation краткое введение

##### Минималистичный пример шаблона, описывающий создание S3

Этот шаблон создаст минимальный S3 без имени, настроек и тп

``` Yaml
Resources:
    MyDemoBucket:
        type: AWS::S3::Bucket
```

А теперь слегка усложним - добавим параметр для имени нашего бакета

``` Yaml
Parameters:
  MyBucketNameParameter:
    Type: String
    Description: This is how the bucket will be called

Resources:
  MyDemoBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !Ref MyBucketNameParameter
```

С использованием суффикса для динамических имен

```Yaml

Parameters:
  MyBucketNameParameter:
    Type: String
    Description: This is how the bucket will be called

  Environment:
    Type: String
    Description: Env to upload
    AllowedValues:
      - Dev
      - Prod
      - Stage

Mappings:
  EnvBucketSuffix:
    Dev:
      Suffix: "-env-deployment"
    Prod:
      Suffix: "-prod-deployment"
    Stage:
      Suffix: "-staging-deployment"

Resources:
  MyDemoBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: 
        !Join 
          - ""
          - - !Ref MyBucketNameParameter
            - !FindInMap [EnvBucketSuffix, !Ref Environment, Suffix]


```

####  Создание S3 - пример близкий к продакшшн

``` Yaml
AWSTemplateFormatVersion: "2010-09-09"

Description: Simple CF template

Parameters:
  Environment:
    Type: String

Mappings:
  EnvMap:
    Prod:
      Suffix: "-prod"

Conditions:
  IsProd: !Equals [!Ref Environment, "Prod"]

Resources:
  MyBucket:
    Type: AWS::S3::Bucket
    Properties:
      BucketName: !If [IsProd, !Join ["", ["mybucket", !FindInMap [EnvMap, Prod, Suffix]]], "mybucket"]

Outputs:
  BucketName:
    Value: !Ref MyBucket

```
