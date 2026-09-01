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

#### Важно помнить

* Каждая `subnet` использует одну Route Table.

* Одна Route Table может быть ассоциирована с несколькими `subnets`.

* Если `subnet` не ассоциирована с конкретной Route Table, она использует **Main Route Table** VPC.

* По умолчанию одна Route Table может содержать **до 50 IPv4 routes и 50 IPv6 routes**.

* Quotas для IPv4 и IPv6 маршрутов считаются **отдельно** и могут быть увеличены через Service Quotas.

* `Public subnet` — это subnet, Route Table которой содержит маршрут к `Internet Gateway`, например:

  `0.0.0.0/0 → Internet Gateway`

* При этом ресурс внутри `public subnet` должен иметь **public IP**, чтобы иметь прямой доступ в Internet.

* Наличие маршрута к `Internet Gateway` само по себе не назначает ресурсам public IP.
