### Route Tables

**Route Table** — это набор правил, определяющих, куда направлять сетевой трафик.

#### Кратко

* отвечают за маршрутизацию трафика
* ассоциируются с `subnet`
* определяют маршрут на основе `destination IP`
* могут направлять трафик в `Internet Gateway`, `NAT Gateway`, `VPC Endpoint`, `VPC Peering`, `Transit Gateway` и другие targets

> [!IMPORTANT]
> При создании `VPC` автоматически создаётся **Main Route Table**.
>
> Каждая новая `subnet` автоматически ассоциируется с `Main Route Table`, если для неё не указана другая Route Table.
>
> `VPC → Main Route Table → Subnets`

#### Public Subnet

`Subnet` считается **public**, если её Route Table содержит маршрут к `Internet Gateway`.

Например:

```text
0.0.0.0/0 → Internet Gateway
```

При этом ресурс внутри `public subnet` должен иметь **public IP**, чтобы иметь прямой доступ в Internet.

#### Private Subnet

`Subnet` считается **private**, если её Route Table **не содержит маршрута к Internet Gateway**.

Для исходящего доступа в Internet обычно используется:

```text
0.0.0.0/0 → NAT Gateway
```

При этом `NAT Gateway` должен находиться в **public subnet** и иметь маршрут к `Internet Gateway`.

#### Важно помнить

* Каждая `subnet` использует одну Route Table.
* Одна Route Table может быть ассоциирована с несколькими `subnets`.
* Если `subnet` не ассоциирована с конкретной Route Table, она использует **Main Route Table** VPC.
* По умолчанию одна Route Table может содержать **до 50 IPv4 routes и 50 IPv6 routes**.
* Quotas для IPv4 и IPv6 маршрутов считаются **отдельно** и могут быть увеличены через Service Quotas.
* Наличие маршрута к `Internet Gateway` делает subnet **public**.
* Отсутствие маршрута к `Internet Gateway` делает subnet **private**.
* Наличие `public IP` у ресурса само по себе не делает subnet public.
* Наличие `NAT Gateway` в Route Table не делает subnet public — наоборот, это типичный способ дать **private subnet** исходящий доступ в Internet.
