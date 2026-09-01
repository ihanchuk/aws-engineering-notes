### Route Tables

__Route Table__ — это набор правил, определяющих, куда направлять сетевой трафик.

#### Кратко

- отвечают за маршрутизацию трафика
- ассоциируются с subnet
- определяют, куда направлять трафик на основе destination IP
- могут направлять трафик в Internet Gateway, NAT Gateway, VPC Endpoint, VPC Peering, Transit Gateway и другие targets

> [!IMPORTANT]
> Подсеть считается публичной, если её Route Table содержит маршрут к Internet Gateway, например:
>
> `0.0.0.0/0 → Internet Gateway`
>
> При этом ресурс внутри public subnet должен иметь public IP, чтобы иметь прямой доступ в Internet.
