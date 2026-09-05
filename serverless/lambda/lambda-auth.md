```JSON
{
  "principalId": "user-123",
  "policyDocument": {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Action": "execute-api:Invoke",
        "Effect": "Allow",
        "Resource": "arn:aws:execute-api:us-east-1:123456789012:abcd1234/*/*"
      }
    ]
  },
  "context": {
    "role": "admin",
    "email": "admin@example.com"
  }
}
```
