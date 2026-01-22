# Инструкция по настройке серверного API

## Обзор

Для безопасной работы с данными о покупках необходимо создать серверный endpoint, который будет использовать токен бота на сервере, а не в клиентском коде.

## Варианты реализации

### Вариант 1: Python (Flask) - Рекомендуется

**Файл:** `server_example_flask.py`

**Установка:**
```bash
pip install flask flask-cors
```

**Запуск:**
```bash
python server_example_flask.py
```

**Для продакшена используйте gunicorn:**
```bash
pip install gunicorn
gunicorn -w 4 -b 0.0.0.0:5000 server_example_flask:app
```

### Вариант 2: Node.js (Express)

**Файл:** `server_example_nodejs.js`

**Установка:**
```bash
npm install express cors
```

**Запуск:**
```bash
node server_example_nodejs.js
```

**Для продакшена используйте PM2:**
```bash
npm install -g pm2
pm2 start server_example_nodejs.js
```

## Настройка в приложении

После запуска сервера обновите конфигурацию в `minitg.html`:

```javascript
const PURCHASES_API_URL = 'https://ваш-домен.com/api/purchases';
const USE_API_SYNC = true;
```

## Развертывание

### Варианты хостинга:

1. **Heroku** (бесплатный тариф)
   - Поддержка Python и Node.js
   - Простое развертывание через Git

2. **Railway** (бесплатный тариф)
   - Автоматическое развертывание
   - Поддержка множества языков

3. **VPS** (DigitalOcean, AWS, etc.)
   - Полный контроль
   - Требует настройки сервера

4. **Vercel/Netlify** (для Node.js)
   - Бесплатный хостинг
   - Простое развертывание

## Безопасность

⚠️ **ВАЖНО:**

1. **Никогда не храните токен бота в клиентском коде!**
2. Используйте переменные окружения для токена:
   ```python
   import os
   BOT_TOKEN = os.getenv('BOT_TOKEN')
   ```
3. Реализуйте проверку подписи `initData` от Telegram
4. Используйте HTTPS в продакшене
5. Ограничьте доступ к API (rate limiting, IP whitelist)

## Интеграция с базой данных

В примерах используется словарь в памяти. В реальном приложении используйте:

- **PostgreSQL** - для сложных запросов
- **MongoDB** - для гибкой схемы
- **SQLite** - для простых приложений
- **Firebase** - для быстрого старта

## Пример интеграции с PostgreSQL (Python)

```python
import psycopg2

def get_user_purchases(user_id, last_sync=None):
    conn = psycopg2.connect("dbname=mydb user=myuser")
    cur = conn.cursor()
    
    if last_sync:
        cur.execute(
            "SELECT * FROM purchases WHERE user_id = %s AND purchase_date > %s",
            (user_id, datetime.fromtimestamp(last_sync / 1000))
        )
    else:
        cur.execute(
            "SELECT * FROM purchases WHERE user_id = %s",
            (user_id,)
        )
    
    purchases = cur.fetchall()
    cur.close()
    conn.close()
    
    return purchases
```

## Тестирование API

Используйте curl или Postman для тестирования:

```bash
# Получение покупок
curl "http://localhost:5000/api/purchases?userId=123456789"

# С lastSync
curl "http://localhost:5000/api/purchases?userId=123456789&lastSync=1704067200000"

# Проверка работоспособности
curl "http://localhost:5000/health"
```

## Следующие шаги

1. Выберите язык/фреймворк
2. Настройте базу данных
3. Реализуйте проверку подписи Telegram
4. Разверните на хостинге
5. Обновите URL в `minitg.html`
6. Протестируйте интеграцию
