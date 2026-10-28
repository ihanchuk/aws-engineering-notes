# AWS CodeBuild и CodePipeline: настройка сборки и развёртывания SAM-приложения

## Введение

В этом руководстве описана настройка AWS CodeBuild и AWS CodePipeline для автоматизации сборки и развёртывания SAM-приложения.

Исходный код уже упакован в архив `my-repository.zip` и загружен в S3 bucket. Pipeline использует этот архив как источник, передаёт его в CodeBuild для сборки и упаковки приложения, а затем использует полученный CloudFormation template для развёртывания.

Общий процесс состоит из четырёх этапов:

1. **Source:** CodePipeline получает `my-repository.zip` из S3.
2. **Build:** CodePipeline запускает проект CodeBuild, который выполняет команды из BuildSpec.
3. **Package:** SAM CLI собирает приложение, создаёт `outputtemplate.yaml` и загружает необходимые артефакты в S3.
4. **Deploy:** CloudFormation использует шаблон для создания или обновления AWS stack.

У каждого сервиса своя роль: CodePipeline управляет последовательностью этапов, CodeBuild выполняет команды сборки, SAM CLI подготавливает приложение, а CloudFormation управляет AWS-ресурсами.

## Часть 1. AWS CodeBuild

### Шаг 1. Создание проекта сборки

1. Откройте **AWS CodeBuild → Build projects → Create build project**.
2. Укажите имя проекта.
3. Выберите `Default project`.

### Шаг 2. Настройка источника

Укажите следующие параметры:

- `Source provider: S3`
- `Bucket`: выберите bucket, в котором находится архив.
- `S3 object key or S3 folder: my-repository.zip`

### Шаг 3. Настройка окружения

Укажите следующие параметры:

- `Compute: EC2`
- `Provisioning model: On Demand`
- `Environment image: Managed Image`
- `Running mode: Running on Docker Container`
- `Operating system: Ubuntu`
- `Runtime(s): Standard`
- `Image: aws/codebuild/standard:5.0`
- `Image Version: Always use the latest version`

### Шаг 4. Настройка BuildSpec

1. Выберите `Insert build commands`.
2. Выберите `Switch to editor`.
3. Вставьте команды сборки из файла `aws-codebuild-steps.md`.

### Шаг 5. Создание проекта

Нажмите `Create build project`.

## Часть 2. AWS CodePipeline

### Шаг 1. Выбор параметров создания

1. Откройте **CodePipeline → Pipelines → Pipelines**.
2. Нажмите `Create pipeline`.
3. Выберите `Build Custom Pipeline`.
4. Нажмите `Next`.

### Шаг 2. Настройка pipeline

Укажите следующие параметры:

- `Execution mode: Superseded`
- `Service role: Existing Service Role`
- `Role ARN`: выберите ARN роли из выпадающего списка.
- Укажите имя pipeline.

### Шаг 3. Добавление этапа Source

Настройте источник:

- `Source provider: S3`
- `Bucket`: выберите bucket, в котором находится архив.
- `S3 Object Key: my-repository.zip`

### Шаг 4. Добавление этапа Build

Настройте провайдера сборки:

- `Build provider: Other build providers`
- В расположенном ниже выпадающем списке выберите `AWS CodeBuild`.
- `Project name`: выберите ранее созданный проект CodeBuild.
- Нажмите `Next`.

### Шаг 5. Добавление этапа Test

Выберите `Skip test stage`.

### Шаг 6. Настройка этапа Deploy

Укажите параметры развёртывания:

- `Deploy provider: AWS CloudFormation`
- `Region: us-east-1` (пример региона)
- `Action mode: Create or update a stack`
- `Stack name: pipeline-stack`

В разделе `Template` укажите:

- `Artifact name: BuildArtifact`
- `Filename: my-repository/outputtemplate.yaml`

В разделе `Capabilities` выберите:

- `CAPABILITY_IAM`
- `CAPABILITY_AUTO_EXPAND`

### Шаг 7. Создание pipeline

Нажмите `Create pipeline`.

## Итоговый процесс

После создания pipeline автоматически выполняет следующие действия:

1. **Source:** CodePipeline получает исходный архив из S3.
2. **Build:** CodePipeline запускает настроенный проект CodeBuild.
3. **Package:** CodeBuild выполняет команды BuildSpec. SAM CLI собирает приложение, создаёт CloudFormation template и загружает необходимые артефакты.
4. **Deploy:** CloudFormation использует созданный шаблон для создания или обновления stack.

Таким образом, этапы получения исходного кода, сборки и развёртывания объединены в единый автоматизированный процесс.
