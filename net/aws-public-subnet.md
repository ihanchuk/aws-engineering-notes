# AWS Public Subnet: что делает подсеть публичной?

**Public Subnet** — это subnet, Route Table которой содержит маршрут через **Internet Gateway (IGW)**, прикреплённый к VPC.

## Условия

Чтобы subnet была публичной, выполняются два условия:

### 1. Internet Gateway прикреплён к VPC

```text
VPC
└── Internet Gateway
```

Само по себе наличие IGW в VPC **не делает все subnet публичными**.

### 2. Route Table subnet содержит маршрут к Internet Gateway

Например:

```text
Destination       Target
-----------------------------
10.0.0.0/16       local
0.0.0.0/0         Internet Gateway
```

Именно этот маршрут делает subnet **public**.

---

## Пример

```text
VPC
│
├── Internet Gateway
│
├── Public Subnet
│   │
│   └── Route Table
│       └── 0.0.0.0/0 → IGW
│
└── Private Subnet
    │
    └── Route Table
        └── 0.0.0.0/0 → NAT Gateway
```

Обе subnet находятся в одной VPC и используют один Internet Gateway, но:

* **Public Subnet** имеет прямой маршрут к IGW.
* **Private Subnet** такого маршрута не имеет.

---

## Важно: Public Subnet ≠ ресурс автоматически имеет Internet

Например, EC2 в Public Subnet должен иметь **public IPv4 address или Elastic IP**, чтобы иметь возможность напрямую взаимодействовать с Internet по IPv4.

Получается:

```text
Public Subnet
     │
     ▼
Route Table
     │
     ▼
Internet Gateway
     │
     ▼
Internet
```

Но сам EC2 должен иметь публичный IP:

```text
EC2
├── Private IP
└── Public IP / Elastic IP
```

---

## Главное правило

> [!IMPORTANT]
> **Public Subnet — это subnet, чья Route Table имеет маршрут к Internet Gateway, прикреплённому к её VPC.**

> [!NOTE]
> **Наличие NAT Gateway не является обязательным условием для Public Subnet.**

NAT Gateway нужен для другого сценария:

```text
Private Resource
      ↓
NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

А публичный ресурс может ходить напрямую:

```text
Public Resource
      ↓
Internet Gateway
      ↓
Internet
```
