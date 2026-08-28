### Summary

| Ресурс                     | Уровень          | Где находится / к чему привязан | Назначение                                        | Нужен для Internet? |
| -------------------------- | ---------------- | ------------------------------- | ------------------------------------------------- | ------------------- |
| **VPC**                    | Network          | Регион                          | Изолированная виртуальная сеть AWS                | Косвенно            |
| **Subnet**                 | Network          | Availability Zone внутри VPC    | Делит VPC на сегменты                             | —                   |
| **Route Table**            | Routing          | Ассоциирована с Subnet          | Определяет, куда направлять traffic               | **Да**              |
| **Internet Gateway (IGW)** | VPC / Internet   | Прикреплён к VPC                | Соединяет VPC с Internet                          | **Да**              |
| **NAT Gateway**            | Network / NAT    | Public Subnet*                  | Даёт Private Resources outbound-доступ в Internet | **Да**              |
| **Network ACL (NACL)**     | Network Security | Subnet                          | Фильтрует traffic на уровне subnet                | Нет                 |
| **Security Group**         | Network Security | ENI / Resource                  | Фильтрует traffic конкретного resource            | Нет                 |
