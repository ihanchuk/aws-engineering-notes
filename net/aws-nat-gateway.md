### AWS NAT Gateway: где он находится и как Private Resources выходят в Internet

> **Важно:** ниже рассматривается классический **Public NAT Gateway**. AWS также предоставляет другие варианты NAT Gateway, например Regional NAT Gateway, поэтому утверждение «NAT Gateway всегда находится в Public Subnet» не является универсальным.

## 1. Что делает NAT Gateway?

NAT Gateway позволяет ресурсам, находящимся в **Private Subnet**, устанавливать исходящие соединения с Internet, при этом внешние системы не могут напрямую инициировать новые соединения к этим ресурсам.

Например:

```text
EC2 в Private Subnet
        │
        │ HTTPS → Internet
        ▼
   NAT Gateway
        │
        ▼
     Internet
```

Главная задача NAT Gateway:

> **Дать private resources возможность выходить в Internet, не делая сами resources публичными.**

---

## 2. Может ли NAT Gateway самостоятельно выйти в Internet?

**Нет.**

Для классического Public NAT Gateway нужен **Internet Gateway (IGW)**.

Получается:

```text
Private Resource
       │
       ▼
NAT Gateway
       │
       ▼
Internet Gateway
       │
       ▼
Internet
```

Роли компонентов разные:

### NAT Gateway

* принимает исходящий трафик от private resources;
* выполняет Network Address Translation;
* использует свой Elastic IP для выхода в Internet;
* обеспечивает возврат ответов правильному private resource.

### Internet Gateway

* обеспечивает связь VPC с Internet;
* является точкой выхода VPC в Internet.

Можно запомнить:

> **NAT Gateway — "дай private resource выйти наружу".**

> **Internet Gateway — "соедини VPC с Internet".**

---

# 3. Почему NAT Gateway находится в Public Subnet?

Это главная ловушка.

Чтобы NAT Gateway мог отправить трафик в Internet, его subnet должна иметь маршрут к Internet Gateway:

```text
Public Subnet Route Table

Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         Internet Gateway
```

Поэтому классическая архитектура выглядит так:

```text
VPC
│
├── Public Subnet
│      │
│      └── NAT Gateway
│
└── Private Subnet
       │
       └── EC2
```

То есть:

> **NAT Gateway предназначен для Private Subnet, но сам находится в Public Subnet.**

---

# 4. Как Private Subnet знает, куда отправлять Internet Traffic?

Через **Route Table**.

У Private Subnet должна быть Route Table примерно такого вида:

```text
Private Route Table

Destination       Target
--------------------------------
10.0.0.0/16       local
0.0.0.0/0         NAT Gateway
```

То есть EC2 говорит:

> «Адрес назначения не находится внутри моей VPC. Отправляю трафик на NAT Gateway».

---

# 5. Получается, нужны две Route Tables

Да.

## Public Subnet

```text
Public Route Table

0.0.0.0/0 → Internet Gateway
```

## Private Subnet

```text
Private Route Table

0.0.0.0/0 → NAT Gateway
```

Вся конструкция:

```text
                    INTERNET
                        ▲
                        │
              ┌─────────┴─────────┐
              │  Internet Gateway │
              └─────────▲─────────┘
                        │
                 Public Subnet
                        │
                 ┌──────┴──────┐
                 │ NAT Gateway │
                 └──────▲──────┘
                        │
                 Private Subnet
                        │
                    ┌───┴───┐
                    │  EC2  │
                    └───────┘
```

Маршруты:

```text
PRIVATE ROUTE TABLE
0.0.0.0/0 → NAT Gateway
                    │
                    ▼
PUBLIC ROUTE TABLE
0.0.0.0/0 → Internet Gateway
```

---

# 6. Как проходит реальный запрос?

Допустим, EC2 в Private Subnet хочет обратиться к:

```text
https://github.com
```

## Шаг 1 — EC2 смотрит на destination IP

Если адрес назначения не относится к VPC CIDR:

```text
10.0.0.0/16
```

используется default route:

```text
0.0.0.0/0 → NAT Gateway
```

---

## Шаг 2 — трафик приходит на NAT Gateway

NAT Gateway преобразует source address.

Условно:

```text
До NAT:

10.0.2.15:54321
       ↓
140.82.x.x:443
```

После NAT:

```text
NAT EIP:xxxxx
       ↓
140.82.x.x:443
```

Private IP EC2 не используется как source IP в Internet.

---

## Шаг 3 — NAT Gateway отправляет traffic через IGW

```text
EC2
 ↓
NAT Gateway
 ↓
Internet Gateway
 ↓
Internet
```

Ответ возвращается обратно через NAT Gateway, который выполняет обратное преобразование адреса.

---

# 7. Почему Internet не может напрямую подключиться к EC2?

EC2 находится в **Private Subnet** и не имеет прямого маршрута через Internet Gateway.

NAT Gateway предназначен для исходящих соединений.

Поэтому:

```text
Private EC2
    │
    │ ────────→ Internet
    │
    │ ←──────── Ответ
```

Но:

```text
Internet
    │
    X ────────→ Private EC2
```

То есть NAT Gateway не превращает Private Subnet в Public Subnet.

---

# 8. Главные выводы

### 1. NAT Gateway не является Internet Connection

Для классического Public NAT Gateway нужен **Internet Gateway**.

### 2. Public NAT Gateway находится в Public Subnet

Потому что ему нужен маршрут:

```text
0.0.0.0/0 → Internet Gateway
```

### 3. Private Resources отправляют Internet Traffic на NAT Gateway

Для этого Private Route Table содержит:

```text
0.0.0.0/0 → NAT Gateway
```

### 4. NAT Gateway отправляет traffic через Internet Gateway

Получается:

```text
Private Subnet
      │
      │ Route Table
      ▼
NAT Gateway
      │
      │ Route Table
      ▼
Internet Gateway
      │
      ▼
Internet
```

---

# 9. Самая полезная ментальная модель

Не думай:

> **«NAT Gateway находится в Private Subnet, чтобы дать ей Internet».**

Думай:

> **«NAT Gateway — это шлюз, расположенный на public side, через который Private Resources получают исходящий доступ в Internet».**

Тогда становится понятно, зачем нужны обе Route Tables:

```text
PRIVATE SIDE

EC2
 │
 └── 0.0.0.0/0 → NAT
                       │
                       ▼
PUBLIC SIDE

NAT Gateway
 │
 └── 0.0.0.0/0 → IGW
                       │
                       ▼
                    INTERNET
```

---

# 10. Важная современная оговорка

В AWS существуют разные варианты NAT Gateway.

Поэтому утверждение:

> **«NAT Gateway всегда должен находиться в Public Subnet»**

нужно понимать как правило для **классического Public NAT Gateway**.

AWS также предоставляет **Regional NAT Gateway**, который работает иначе и не требует классической схемы с размещением NAT Gateway в Public Subnet.

Для изучения базовой VPC-архитектуры сначала стоит хорошо понять именно классический паттерн:

```text
Private Subnet
      ↓
Public NAT Gateway
      ↓
Internet Gateway
      ↓
Internet
```

Именно этот паттерн является фундаментом для понимания AWS networking.
