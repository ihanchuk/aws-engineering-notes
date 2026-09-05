| Resource Type | Описание |
|---|---|
| `AWS::SQS::Queue` | Очередь сообщений SQS. |
| `AWS::SQS::QueuePolicy` | Политика доступа к очереди SQS. |
| `AWS::SQS::RedriveAllowPolicy` | Настройки повторной обработки сообщений (DLQ allow). |
| `AWS::SQS::RedrivePolicy` | Настройки Dead Letter Queue (DLQ). |
| `AWS::SNS::Topic` | SNS-топик для pub/sub. |
| `AWS::SNS::Subscription` | Подписка на SNS-топик. |
| `AWS::SNS::TopicPolicy` | Политика доступа к SNS-топику. |
| `AWS::Events::Rule` | EventBridge Rule (триггеры и маршрутизация событий). |
| `AWS::Events::Bus` | EventBridge шина событий. |
| `AWS::Events::EventBusPolicy` | Политика EventBus. |
| `AWS::AmazonMQ::Broker` | Message broker (RabbitMQ / ActiveMQ). |
| `AWS::AmazonMQ::Configuration` | Конфигурации Amazon MQ. |


| Resource Type | Описание |
|---|---|
| `AWS::EC2::Instance` | EC2 виртуальная машина. |
| `AWS::EC2::SecurityGroup` | Firewall-группа безопасности. |
| `AWS::EC2::SecurityGroupIngress` | Входящие правила SG. |
| `AWS::EC2::SecurityGroupEgress` | Исходящие правила SG. |
| `AWS::EC2::LaunchTemplate` | Настройки запуска EC2. |
| `AWS::EC2::Subnet` | Подсеть в VPC. |
| `AWS::EC2::VPC` | Виртуальная сеть. |
| `AWS::Lambda::Function` | Lambda-функция. |
| `AWS::Lambda::Permission` | Разрешение доступа к Lambda (например API Gateway). |
| `AWS::Lambda::EventSourceMapping` | Подписка Lambda на SQS/SNS/Kinesis/DynamoDB Streams. |
| `AWS::Lambda::Alias` | Alias Lambda (prod / dev). |
| `AWS::Lambda::Version` | Версия Lambda. |
| `AWS::ECS::Cluster` | ECS кластер. |
| `AWS::ECS::TaskDefinition` | Task Definition для ECS/Fargate. |
| `AWS::ECS::Service` | ECS/Fargate сервис. |
| `AWS::ECS::TaskSet` | Набор задач (используется для blue/green). |
| `AWS::ECS::CapacityProvider` | Capacity provider ECS. |
| `AWS::ElasticLoadBalancingV2::LoadBalancer` | ALB/NLB. |
| `AWS::ElasticLoadBalancingV2::TargetGroup` | Target group для ALB/NLB. |
| `AWS::ElasticLoadBalancingV2::Listener` | Listener для ALB/NLB. |
| `AWS::AutoScaling::AutoScalingGroup` | Группа автоскейлинга EC2. |
| `AWS::AutoScaling::LaunchConfiguration` | Настройки запуска ASG (устаревшее). |
| `AWS::AutoScaling::ScalingPolicy` | Политика масштабирования ASG. |
| `AWS::AutoScaling::ScheduledAction` | Планировщик автоскейлинга. |

| Resource Type | Описание |
|---|---|
| `AWS::DynamoDB::Table` | NoSQL таблица DynamoDB. |
| `AWS::DynamoDB::GlobalTable` | Глобальная таблица DynamoDB. |
| `AWS::RDS::DBInstance` | RDS база данных (Postgres/MySQL/etc). |
| `AWS::RDS::DBCluster` | Кластер RDS/Aurora. |
| `AWS::RDS::DBSubnetGroup` | Сетка подсетей для RDS. |
| `AWS::RDS::DBParameterGroup` | Параметры DB instance. |
| `AWS::RDS::DBClusterParameterGroup` | Параметры DB cluster. |
| `AWS::RDS::OptionGroup` | Доп. фичи RDS (например OEM плагины). |
| `AWS::ElastiCache::CacheCluster` | Redis/Memcached кластер. |
| `AWS::ElastiCache::ReplicationGroup` | Репликация для Redis. |
| `AWS::DocDB::DBCluster` | DocumentDB кластер. |
| `AWS::DocDB::DBInstance` | DocumentDB инстанс. |
| `AWS::DocDB::DBClusterParameterGroup` | Параметры кластера DocDB. |
| `AWS::EFS::FileSystem` | EFS файловая система (shared storage). |
| `AWS::EFS::MountTarget` | Точка монтирования EFS. |
