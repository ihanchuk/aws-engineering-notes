```
const mongoose = require('mongoose');

let cachedConnection = null;

async function connectToDatabase() {
  if (cachedConnection) {
    console.log('Используем кешированное соединение');
    return cachedConnection;
  }

  console.log('Создаём новое соединение');
  cachedConnection = await mongoose.connect(process.env.MONGO_URI, {
    useNewUrlParser: true,
    useUnifiedTopology: true,
  });

  return cachedConnection;
}

exports.handler = async (event) => {
  const db = await connectToDatabase();

  // теперь можно работать с коллекциями
  const users = await db.model('User').find();
  
  return {
    statusCode: 200,
    body: JSON.stringify(users),
  };
};
```

