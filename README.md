# wam-health-digest

Зашифрованный дайджест состояния Ярослава Шуваева — для совместного планирования приёмов пищи с другим AI-агентом (диетологом жены).

## Что внутри

`digest.enc` — base64-кодированный шифр AES-256-GCM. Внутри JSON с актуальным контекстом:

- параметры (рост, текущий и целевой вес)
- сегодняшний баланс калорий и БЖУ
- цели по диете и тренировкам
- последние приёмы пищи (3–5 шт)
- активность за вчера (Apple Health)
- заметки коуча и рекомендации по следующему приёму пищи

Файл обновляется автоматически после каждого изменения в основном журнале (`yshuvaev/wam-health`).

## Как расшифровать

Формат шифра: `base64(salt[16] || nonce[12] || ciphertext+tag)`.
Ключ: PBKDF2-HMAC-SHA256, 200 000 итераций, выход 32 байта.

Python:

```python
from cryptography.hazmat.primitives.ciphers.aead import AESGCM
from cryptography.hazmat.primitives.kdf.pbkdf2 import PBKDF2HMAC
from cryptography.hazmat.primitives import hashes
import base64, json, urllib.request

PASSWORD = "<shared-password>"

raw = urllib.request.urlopen("https://yshuvaev.github.io/wam-health-digest/digest.enc").read()
data = base64.b64decode(raw)
salt, nonce, ct = data[:16], data[16:28], data[28:]

kdf = PBKDF2HMAC(algorithm=hashes.SHA256(), length=32, salt=salt, iterations=200_000)
key = kdf.derive(PASSWORD.encode())

plaintext = AESGCM(key).decrypt(nonce, ct, None)
digest = json.loads(plaintext)
```

Пароль — общий секрет между двумя агентами, передаётся вне репозитория.
