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
> Например:
>
> `VPC → Main Route Table → Subnets`

#### Важно помнить

* каждая `subnet` использует одну Route Table.

* одна Route Table может быть ассоциирована с несколькими `subnets`.

* если `subnet` не ассоциирована с конкретной Route Table, она использует **Main Route Table** VPC.

* для IPv4 и IPv6 существуют отдельные quotas на количество маршрутов.

* `Public subnet` — это subnet, Route Table которой содержит маршрут к `Internet Gateway`, например:

  `0.0.0.0/0 → Internet Gateway`

* при этом ресурс внутри `public subnet` должен иметь **public IP**, чтобы иметь прямой доступ в Internet.

* наличие маршрута к `Internet Gateway` само по себе не назначает ресурсам public IP.
