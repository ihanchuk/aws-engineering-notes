❌ Запускаешь Express/HTTP сервер внутри Lambda-контейнера
→ Lambda сама вызывает runtime, Express не нужен

❌ Не используешь RIC → контейнер не получает invoke events

❌ CMD указано как npm start, node server.js
→ Lambda не вызовет твой handler

❌ Порт 3000, а не 8080
→ Lambda не сможет общаться с контейнером
