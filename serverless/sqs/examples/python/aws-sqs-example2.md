# Python examples

Пример на получение и чтение атрибутов (Consumer/Worker)

```yaml
import boto3
import json

sqs = boto3.client('sqs', region_name='eu-central-1')
queue_url = 'https://amazonaws.com'

# Читаем сообщения из очереди
response = sqs.receive_message(
    QueueUrl=queue_url,
    MaxNumberOfMessages=1,
    WaitTimeSeconds=20,          # Включаем Long Polling (ожидание до 20 сек)
    MessageAttributeNames=['All'] # КРИТИЧНО: без этого атрибуты будут пустыми!
)

if 'Messages' in response:
    for message in response['Messages']:
        # 1. Читаем тело сообщения
        body = json.loads(message['Body'])
        print(f"Получено тело заказа: ID {body['order_id']}")
        
        # 2. Проверяем и читаем атрибуты (если они есть)
        attrs = message.get('MessageAttributes', {})
        
        event_type = attrs.get('EventType', {}).get('StringValue', 'Unknown')
        priority = attrs.get('Priority', {}).get('StringValue', '0')
        
        print(f"Метаданные из атрибутов -> Тип события: {event_type}, Приоритет: {priority}")
        
        # 3. Наш код САМ решает, что делать на основе атрибутов
        if event_type == 'OrderCreated' and int(priority) > 0:
            print("Логика воркера: Обрабатываем как важный новый заказ...")
        else:
            print("Логика воркера: Обычная обработка...")

        # 4. Удаляем сообщение из очереди, чтобы оно не вернулось обратно
        sqs.delete_message(
            QueueUrl=queue_url,
            ReceiptHandle=message['ReceiptHandle']
        )
else:
    print("Очередь пуста, сообщений нет.")
```

> [!IMPORTANT]
> По умолчанию SQS при вызове receive_message не возвращает атрибуты, чтобы экономить трафик. Чтобы их получить, нужно явно передать параметр MessageAttributeNames=['All'].
