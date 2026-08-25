### SG Examples

#### Пример 1 RDS PostgreSQL — разрешить подключение к БД

Самый простой вариант: PostgreSQL принимает подключения только из нашей внутренней сети.

```hcl
resource "aws_security_group" "postgres" {
  name   = "postgres-sg"
  vpc_id = var.vpc_id

  ingress {
    description = "PostgreSQL from private network"
    protocol    = "tcp"
    from_port   = 5432
    to_port     = 5432
    cidr_blocks = ["10.0.0.0/16"]
  }
}
```
```yaml
Resources:

  PostgresSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow PostgreSQL from private network
      VpcId: !Ref VpcId

      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 5432
          ToPort: 5432
          CidrIp: 10.0.0.0/16
```

Создаёт Security Group для PostgreSQL и разрешает входящие подключения к порту 5432 только из VPC 10.0.0.0/16. Подключения к PostgreSQL из интернета будут заблокированы.

#### Пример 2 EC2 — SSH + обновление системы

Разрешает SSH-доступ к EC2 только с IP администратора 203.0.113.10. Сам сервер может устанавливать HTTPS-соединения наружу — например, чтобы скачивать обновления ОС и пакеты. 
Ответы на эти исходящие соединения разрешаются автоматически благодаря stateful behavior.

```hcl
resource "aws_security_group" "server" {
  name   = "server-sg"
  vpc_id = var.vpc_id

  ingress {
    description = "SSH from admin network"
    protocol    = "tcp"
    from_port   = 22
    to_port     = 22
    cidr_blocks = ["203.0.113.10/32"]
  }

  egress {
    description = "HTTPS for system updates"
    protocol    = "tcp"
    from_port   = 443
    to_port     = 443
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```
```yaml
Resources:

  ServerSecurityGroup:
    Type: AWS::EC2::SecurityGroup
    Properties:
      GroupDescription: Allow SSH and HTTPS outbound
      VpcId: !Ref VpcId

      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 203.0.113.10/32

      SecurityGroupEgress:
        - IpProtocol: tcp
          FromPort: 443
          ToPort: 443
          CidrIp: 0.0.0.0/0
```
