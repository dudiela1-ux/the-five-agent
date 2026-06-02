---
name: gpt-image-gen
description: מעטפת ל-OpenAI Images API. מקבל PROMPT ו-OUTPUT_PATH, יוצר תמונה עם gpt-image-2 ושומר כ-PNG.
---

# gpt-image-gen

יוצר תמונה דרך OpenAI Images API ושומר אותה לדיסק.

## תשומות נדרשות

| משתנה | תיאור |
|-------|-------|
| `PROMPT` | תיאור התמונה המלא (string) |
| `OUTPUT_PATH` | נתיב שמירה כולל שם קובץ וסיומת `.png` |

> `OPENAI_API_KEY` חייב להיות מוגדר ב-`.env` לפני הקריאה.

---

## ביצוע — שיטה ראשית (curl + jq)

```bash
PROMPT="תיאור התמונה כאן"
OUTPUT_PATH="yuval/outputs/2026-06-02-example.png"

curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"gpt-image-2\",
    \"prompt\": \"$PROMPT\",
    \"size\": \"1024x1024\",
    \"quality\": \"medium\",
    \"output_format\": \"png\"
  }" | jq -r '.data[0].b64_json' | base64 --decode > "$OUTPUT_PATH"
```

---

## ביצוע — Python fallback (כשאין jq, למשל ב-Git Bash)

```bash
PROMPT="תיאור התמונה כאן"
OUTPUT_PATH="yuval/outputs/2026-06-02-example.png"

curl -s -X POST "https://api.openai.com/v1/images/generations" \
  -H "Authorization: Bearer $OPENAI_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"model\": \"gpt-image-2\",
    \"prompt\": \"$PROMPT\",
    \"size\": \"1024x1024\",
    \"quality\": \"medium\",
    \"output_format\": \"png\"
  }" | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
b64 = data['data'][0]['b64_json']
with open('$OUTPUT_PATH', 'wb') as f:
    f.write(base64.b64decode(b64))
"
```

---

## אימות

```bash
[ -s "$OUTPUT_PATH" ] && echo "OK: $OUTPUT_PATH" || echo "ERROR: file empty or missing"
```

---

## הערות חשובות

- **המודל הוא `gpt-image-2`** — יצא ב-21 באפריל 2026. אל תחליף ל-`dall-e-3` או `gpt-image-1`.
- אם יש שגיאת API → הבעיה ב-`OPENAI_API_KEY` או ב-parameters, לא בשם המודל.
- הפרמטר `output_format: "png"` מחזיר את התמונה כ-base64 בתוך ה-JSON.
- `size`: ניתן לשנות ל-`512x512` או `1024x1792` לפי הצורך.
- `quality`: `low` | `medium` | `high` — `medium` כברירת מחדל לאיזון מחיר/איכות.
