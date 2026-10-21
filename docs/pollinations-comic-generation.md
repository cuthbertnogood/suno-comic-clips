# Интеграция Pollinations.ai для генерации комикс-страниц (AG_Clip_maker)

Инструкция для агента **AG_Clip_maker** (Suno2_Agent) по генерации картинок в стиле комиксов через бесплатный API Pollinations.ai.

## Почему Pollinations

- Полностью бесплатно, без регистрации и API-ключа (анонимный тир).
- Простой GET-запрос — идеально для скриптов бота.
- Поддерживает Flux и другие модели.
- Легко генерировать панели в стиле комиксов через промпт.

## Базовый эндпоинт

```
https://image.pollinations.ai/prompt/{URL_ENCODED_PROMPT}
```

Или более новый:
```
https://gen.pollinations.ai/image/{URL_ENCODED_PROMPT}
```

### Параметры (через query string)
- `width`, `height` — размер (например 1024)
- `seed` — для воспроизводимости
- `nologo=true` — убрать логотип Pollinations
- `model=flux` — выбрать модель (если поддерживается)

## Пример для бота (bash / curl)

```bash
PROMPT="comic book style, bold black outlines, halftone dots, dynamic action scene, superhero in noir style, vibrant colors, single panel, no text"
ENCODED=$(python3 -c "import urllib.parse; print(urllib.parse.quote('''$PROMPT'''))")

curl -s -o page01.jpg \
  "https://image.pollinations.ai/prompt/${ENCODED}?width=1024&height=1024&nologo=true&seed=42"
```

Или на Python (рекомендуется для серии панелей):

```python
import urllib.parse
import urllib.request
import time
import os

def generate_comic_panel(prompt: str, filename: str, width=1024, height=1024, seed=None):
    full_prompt = f"comic book style, bold black outlines, halftone shading, {prompt}, high quality, detailed line art"
    encoded = urllib.parse.quote(full_prompt)
    url = f"https://image.pollinations.ai/prompt/{encoded}?width={width}&height={height}&nologo=true"
    if seed is not None:
        url += f"&seed={seed}"
    
    urllib.request.urlretrieve(url, filename)
    print(f"Saved: {filename}")
    time.sleep(20)  # соблюдать рате-лимит анонима (~15-45 сек)

# Пример использования для страниц комикса
panels = [
    "hero standing on rooftop at night, city lights below",
    "villain appearing in shadows, dramatic lighting",
    "intense fight scene, motion lines",
]

os.makedirs("pages", exist_ok=True)
for i, p in enumerate(panels, 1):
    generate_comic_panel(p, f"pages/page{i:02d}.jpg", seed=1000 + i)
```

## Рекомендации по промптам для стиля комиксов

Всегда добавляй в начало или конец:
- `comic book style`
- `bold black outlines`
- `halftone dots` или `Ben-Day dots`
- `dynamic pose`
- `panel border` (если нужна рамка)
- Конкретный стиль: `Sin City noir`, `Marvel comic`, `manga style`, `Franco-Belgian comic`

Для консистентности персонажа используй один и тот же `seed` и описывай внешность детально.

## Лимиты

- Анонимный тир: ~1 запрос в 15–45 секунд.
- При превышении приходит 402 с `Retry-After`.
- Для большего объёма — бесплатная регистрация на enter.pollinations.ai (получить ключ).

## Интеграция в существующий пайплайн

1. Сгенерировать страницы в `pages/pageXX.jpg` в папке клипа.
2. Сохранить `lyrics.txt` и `suno_prompt.md` как обычно.
3. Для видео-клипа использовать существующие скрипты склейки (через ffmpeg) или пайплайн бота.
4. Обновить QUEUE.md и README папки.

## Примечания

- Если нужна лучшая консистентность персонажа — рассмотреть AI Horde или Hugging Face Spaces с референсами.
- Pollinations хорошо подходит для быстрой генерации панелей без дополнительной инфраструктуры.
- Бот может вызывать это напрямую из своего workspace через bash/python.

Документ добавлен: 2026-10-21. Для агента AG_Clip_maker / Suno2_Agent.
