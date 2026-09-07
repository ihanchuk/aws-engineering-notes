### Lambda Micro VM

Lambda MicroVM — serverless-среда выполнения (container) для запуска пользовательского кода в изолированных, сохраняющих состояние execution environments. В основе используется Firecracker microVM, обеспечивающая VM-уровень изоляции при быстром запуске.

**Stateful** — это не просто «может сохранять данные». Речь о том, что само execution environment может сохраняться между вызовами, включая состояние памяти и файловой системы, а не обязательно создаваться заново для каждого запуска.

```text
request
  ↓
есть готовый execution environment?
  │
  ├── да ──► resume/frozen environment ──► handler
  │
  └── нет ─► create/initialize environment ─► handler
```

## MicroVM Lifecycle

Во время работы MicroVM проходит следующие состояния:

#### 1. Run — запуск

Вы вызываете `run-microvm`.

Lambda восстанавливает MicroVM из **image snapshot**, присваивает ей уникальный ID и создаёт endpoint для доступа к приложению.

```
PENDING → RUNNING
```

В отличие от классического запуска VM, MicroVM восстанавливается из заранее подготовленного snapshot, поэтому запуск происходит очень быстро.

#### 2. Running — выполнение

MicroVM находится в состоянии `RUNNING`.

Приложение внутри MicroVM принимает и обрабатывает запросы через предоставленный endpoint URL.

В этом состоянии сохраняются:

- состояние памяти;
- состояние диска;
- состояние приложения;
- runtime state.

MicroVM может обрабатывать множество запросов, пока остаётся активной.

#### 3. Suspend — приостановка

После заданного периода бездействия MicroVM может быть автоматически приостановлена. Также её можно приостановить явно через API `suspend-microvm`.

```
RUNNING → SUSPENDING → SUSPENDED
```

Ключевой момент — **MicroVM не уничтожается**.

Сохраняются:

- содержимое памяти;
- состояние диска;
- состояние приложения.

Поэтому это не аналог обычного cold start: среда может быть продолжена практически с того же состояния, в котором была приостановлена.

#### 4. Resume — возобновление

Когда снова поступает трафик, MicroVM может автоматически возобновиться, если включён:

```
autoResumeEnabled=true
```

Либо её можно явно восстановить через `resume-microvm`.

```
SUSPENDED → RUNNING
```

MicroVM продолжает выполнение с сохранённого состояния памяти и диска.

#### 5. Terminate — завершение

MicroVM окончательно уничтожается, когда вызывается `terminate-microvm` или истекает максимально допустимая продолжительность её существования.

```
RUNNING / SUSPENDED
       │
       ▼
  TERMINATING
       │
       ▼
   TERMINATED
```

После `TERMINATED` состояние MicroVM больше не является активной execution environment.

### Lifecycle в целом

```
run-microvm
     │
     ▼
  PENDING
     │
     ▼
  RUNNING ◄──────────────┐
     │                   │
     │ idle / suspend    │ resume
     ▼                   │
 SUSPENDING              │
     │                   │
     ▼                   │
 SUSPENDED ──────────────┘
     │
     │ terminate / max duration
     ▼
 TERMINATING
     │
     ▼
 TERMINATED
```

### Главное отличие от классической Lambda mental model

Классическая модель часто выглядит так:

```
Request
   ↓
Create environment
   ↓
Initialize
   ↓
Handler
   ↓
Environment может быть reused
```

MicroVM меняет акцент:

```
Run
 ↓
RUNNING
 ↓
Suspend
 ↓
SUSPENDED
 ↓
Resume
 ↓
RUNNING
```

То есть execution environment становится **явно управляемой, сохраняющей состояние вычислительной средой**, которую можно быстро запускать, приостанавливать и возобновлять благодаря snapshot-based lifecycle и Firecracker.

### Спецификации

#### Compute

- 0.5-32 GB Memory
- 0.25 - 16 vCPU
- 4x burst
- 32 GB disk

#### Platform

- ARM64 Graviton
- Amazon Linux
- FireCracker VM

#### Lifecycle

- 8 hours max
- Suspend/Resume
- Auto-resume
- Per second biling

#### Connectivity

- HTTPS per MicroVM
- HHTP/2, gRPC, WS
- JWE auth tokens
- VPC egress

#### Conecctivity in details:

**HTTPS** per MicroVM

Каждая MicroVM получает свой HTTPS endpoint. Это означает, что HTTP(S)-трафик может быть направлен не просто на Lambda function, а на конкретную запущенную MicroVM.

**HTTP/2, gRPC, WebSocket**

Здесь важное отличие от классической Lambda-модели.

Если MicroVM предоставляет долгоживущий network endpoint, приложение внутри неё может работать не только с обычными HTTP/1.1 request/response, но и с протоколами, рассчитанными на постоянное соединение:

- HTTP/2
- gRPC
- WebSocket

Это принципиально отличается от привычной Lambda-модели, где invocation — это отдельный вызов функции

**JWE auth tokens**

JWE (JSON Web Encryption) — это зашифрованный JSON Web Token.

В контексте MicroVM это можно понимать как механизм передачи credentials/auth context при обращении к endpoint.

Почему именно JWE, а не просто JWT?

У обычного подписанного JWT содержимое токена можно прочитать а JWE дополнительно шифрует payload.

> [!IMPORTANT]
> JWE не является обязательным механизмом аутентификации Lambda вообще.

**VPC egress**

Это уже исходящий трафик из MicroVM.

Например, приложение внутри MicroVM хочет обратиться к:

- RDS
- DynamoDB
- S3
- internal API
- Internet

```text
                  AWS VPC
┌──────────────────────────────────────────┐
│                                          │
│  MicroVM                                  │
│     │                                    │
│     │ outbound traffic                  │
│     ▼                                    │
│  VPC networking                           │
│     │                                    │
│     ├──► private service                 │
│     │                                    │
│     └──► NAT Gateway ──► Internet       │
│                                          │
└──────────────────────────────────────────┘
```

То есть MicroVM может иметь VPC network connectivity и использовать обычные AWS механизмы маршрутизации, security groups/NACLs и NAT для выхода наружу.

> [!IMPORTANT]
> Обычная Lambda тоже может использовать HTTP/2/gRPC/WebSocket для исходящих соединений

> [!IMPORTANT]
> MicroVM может кратковременно обработать нагрузку примерно в 4 раза выше своей обычной/базовой производительности.
