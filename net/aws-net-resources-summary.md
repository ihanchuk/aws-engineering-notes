### AWS Networking — краткое summary

#### VPC (Virtual Private Cloud) — это изолированная виртуальная сеть AWS. ####
Внутри VPC ты создаёшь subnets, которые являются отдельными сетевыми сегментами и находятся в конкретных Availability Zones. Обычно subnets делят на public и private в зависимости от их маршрутизации.

#### Route Table определяет, куда отправляется traffic из subnet.####
Именно маршрут определяет, является ли subnet публичной: если её Route Table имеет маршрут вроде 0.0.0.0/0 → Internet Gateway, это Public Subnet. Private Subnet обычно имеет 0.0.0.0/0 → NAT Gateway.

#### Internet Gateway (IGW) подключает VPC к Internet.#### 
Он прикрепляется непосредственно к VPC, но сам по себе не делает все её subnets публичными. Для прямого выхода ресурсу также нужен public IPv4/EIP и соответствующий маршрут через IGW.

#### NAT Gateway#### 
используется для исходящего доступа в Internet из Private Subnet. В классической архитектуре Public NAT Gateway размещается в Public Subnet и отправляет traffic дальше через IGW. Поэтому типичная цепочка выглядит как Private Resource → NAT Gateway → Internet Gateway → Internet.

#### Network ACL (NACL) — фильтр traffic на уровне subnet. ####
Он stateless: inbound и outbound traffic контролируются независимо. Security Group работает на уровне конкретного resource/ENI и является stateful. В отличие от NACL, Security Group поддерживает только ALLOW, без явных DENY.

Таким образом, всю базовую сеть можно представить как цепочку: VPC создаёт сеть → Subnet делит её на сегменты → Route Table определяет путь → IGW даёт доступ в Internet → NAT Gateway даёт private resources исходящий Internet → NACL фильтрует subnet traffic → Security Group фильтрует traffic конкретного resource.

| Ресурс                     | Уровень          | Где находится / к чему привязан | Назначение                                        | Нужен для Internet? |
| -------------------------- | ---------------- | ------------------------------- | ------------------------------------------------- | ------------------- |
| **VPC**                    | Network          | Регион                          | Изолированная виртуальная сеть AWS                | Косвенно            |
| **Subnet**                 | Network          | Availability Zone внутри VPC    | Делит VPC на сегменты                             | —                   |
| **Route Table**            | Routing          | Ассоциирована с Subnet          | Определяет, куда направлять traffic               | **Да**              |
| **Internet Gateway (IGW)** | VPC / Internet   | Прикреплён к VPC                | Соединяет VPC с Internet                          | **Да**              |
| **NAT Gateway**            | Network / NAT    | Public Subnet*                  | Даёт Private Resources outbound-доступ в Internet | **Да**              |
| **Network ACL (NACL)**     | Network Security | Subnet                          | Фильтрует traffic на уровне subnet                | Нет                 |
| **Security Group**         | Network Security | ENI / Resource                  | Фильтрует traffic конкретного resource            | Нет                 |

* NAT Gateway размещается в публичной подсети, но обслуживает ресурсы находящиеся в приватной.
