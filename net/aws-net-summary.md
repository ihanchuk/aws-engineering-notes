### Сетевые ресурсы кратко

__VPC__ (Virtual Private Cloud) — твоя логическая сеть в AWS. Внутри неё находятся subnets, route tables, security groups, NACL и сетевые интерфейсы ресурсов.

```
VPC
├── Subnet
├── Route Table
├── Security Group
├── NACL
├── Internet Gateway
├── NAT Gateway
└── VPC Endpoints
```

__Subnet__ — это диапазон IP-адресов внутри конкретной Availability Zone.

Она сама по себе не является public или private.

Public subnet → её route table содержит маршрут наружу через IGW.

Private subnet → нет маршрута напрямую через IGW. При этом она может иметь интернет-egress через NAT Gateway.

Public:
```
Subnet → Route Table → IGW → Internet
```

Private:
```
Subnet → Route Table → NAT Gateway → IGW → Internet
```

__Route Table__

Route Table отвечает на вопрос:

«Куда отправить пакет?»

Например:
```
0.0.0.0/0 → igw-123
```

или:
```
0.0.0.0/0 → nat-123
```

Route Table ассоциируется с subnet. Именно маршруты определяют, куда subnet отправляет traffic.

__Internet Gateway__ (IGW)

IGW — шлюз между VPC и Internet.

Он attach-ится к VPC, а не к subnet.

Но чтобы конкретная subnet считалась public, её Route Table должна иметь маршрут:
```
0.0.0.0/0 → IGW
```

То есть сам факт наличия IGW в VPC не делает все subnet public.

__NAT Gateway__

NAT Gateway позволяет ресурсам из private subnet делать outbound connections в Internet, не принимая входящие соединения из Internet.

Ключевой момент:

NAT Gateway находится в public subnet.

Типичная схема:
```
Private EC2
    ↓
Private Route Table
    ↓
NAT Gateway
    ↓
Public Route Table
    ↓
IGW
    ↓
Internet
```

NAT Gateway сам по себе интернет не создаёт — ему нужен путь через IGW.

__Security Group__ (SG)

SG — stateful firewall, привязанный к сетевому интерфейсу ресурса.
```
Internet
   ↓
NACL
   ↓
Security Group
   ↓
EC2 / ECS / Lambda ENI
```

Особенности:

- работает на уровне resource/ENI;
- только ALLOW;
- stateful — ответный traffic автоматически разрешается;
можно ссылаться на другой Security Group.

__NACL__

NACL — stateless firewall на уровне subnet.

В отличие от SG:

- ALLOW + DENY;
- stateless;
- inbound и outbound проверяются независимо;
- правила имеют номера;
- меньший номер проверяется первым;
- срабатывает первое совпавшее правило.

```
100 DENY  1.2.3.4
200 ALLOW 0.0.0.0/0
```

Для 1.2.3.4 → Rule 100 → DENY. Правило 200 уже не рассматривается.

__VPC Endpoint__

VPC Endpoint позволяет обращаться к AWS-сервисам без выхода через Internet Gateway или NAT Gateway.

Например:
```
Private EC2
    ↓
VPC Endpoint
    ↓
S3
```

Особенно полезен для private subnet, потому что можно убрать зависимость от NAT для доступа к определённым AWS services.
