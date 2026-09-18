# Python examples

```python
import boto3
import json

# Инициализируем клиент SQS
sqs = boto3.client('sqs', region_name='eu-central-1')

# URL вашей очереди SQS
queue_url = 'https://amazonaws.com'

# Тело сообщения (обычно JSON)
message_body = json.dumps({
    "order_id": 98765,
    "amount": 150.00
})

# Атрибуты сообщения (метаданные)
message_attributes = {
    'EventType': {
        'DataType': 'String',
        'StringValue': 'OrderCreated'
    },
    'Priority': {
        'DataType': 'Number',
        'StringValue': '1'  # Даже числа в SQS передаются как строки внутри StringValue
    }
}

# Отправка в SQS
response = sqs.send_message(
    QueueUrl=queue_url,
    MessageBody=message_body,
    MessageAttributes=message_attributes
)

print(f"Сообщение успешно отправлено. MessageId: {response['MessageId']}")
```
