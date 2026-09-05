### Контейнеризированная лямба

#### Пример контейнера
```
FROM public.ecr.aws/lambda/nodejs:20

# Copy app code into container
COPY app.js package.json ${LAMBDA_TASK_ROOT}/

# Install npm deps
RUN npm install

# Your Lambda handler
CMD ["app.handler"]
```
#### Код лямбды
```
exports.handler = async (event) => {
  return {
    statusCode: 200,
    body: JSON.stringify({ message: "Hello from Lambda Container!" })
  };
};
```

#### Билдим Контейнер из нашего файлы

```
docker build -t my-lambda-image .
```

#### Локальный запуск с Докером

```
docker run -p 9000:8080 my-lambda-image
```

#### Обращаемся к лямбде 
```
curl -XPOST "http://localhost:9000/2015-03-31/functions/function/invocations" \
    -d '{ "name": "Igor" }'
```
