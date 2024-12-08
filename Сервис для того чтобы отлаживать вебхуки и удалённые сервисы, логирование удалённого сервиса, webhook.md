---
Добавлена в базу: 25.07.2024 в 02:34
Ссылка: https://webhook.site/
Тип: "[[Ссылка на полезное]]"
---
Для отладки удалённых сервисов — можно долбить в уникальный выделенный URL и смотреть в веб-интерфейсе данные всех пришедших на URL запросов.

```python
import json
from datetime import datetime
import http.client

def datetime_converter(o):
	if isinstance(o, datetime):
		return o.isoformat()

url = "webhook.site"
endpoint = "/3dfd26c3-6f42-4e94-bb72-3bfc6e0fb989"

json_data = json.dumps(rows, default=datetime_converter)
conn = http.client.HTTPSConnection(url)
headers = {'Content-type': 'application/json'}
conn.request("POST", endpoint, body=json_data, headers=headers)
response = conn.getresponse()
conn.close()
```

[[tools]]
[[Python]]