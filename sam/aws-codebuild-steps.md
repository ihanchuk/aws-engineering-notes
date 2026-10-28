# Aws Codebuild Steps demo

```yaml
version: 0.2

phases:
  install:
    runtime-versions:
       python: 3.12
  build:
    commands:
      - pip install aws-sam-cli
      - aws s3 cp s3://sam-bucket-e6ecd1f0/my-repository.zip .
      - unzip -o my-repository.zip
      - cd my-repository
      - sam build -t sam-app/template.yaml
      - sam package --template-file sam-app/template.yaml --output-template-file outputtemplate.yaml --s3-bucket sam-bucket-e6ecd1f0
artifacts:
  files: 
    - my-repository/outputtemplate.yaml
```
