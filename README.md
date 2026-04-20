# Grand Mebel — Лендинг-калькулятор кухонь

Одностраничный калькулятор стоимости кухни с двухэтапным захватом лида, отправкой в WhatsApp и записью в Google Sheets через Apps Script.

---

## Структура

```
grand-mebel/
├── index.html              # Лендинг (всё в одном файле)
└── apps-script/
    └── webhook.gs          # Google Apps Script — приём лидов в таблицу
```

---

## Быстрый старт

### 1. Настроить `index.html`

Открыть файл и заменить три константы в начале `<script>`:

```js
const WA_PHONE       = '77XXXXXXXXX';          // номер WhatsApp без +
const LEAD_WEBHOOK   = 'https://script.google.com/macros/s/XXXXX/exec';
const WEBHOOK_SECRET = 'ваш-секретный-ключ';   // произвольная строка
```

Также заменить номер в trust-баре (строка `href="https://wa.me/77XXXXXXXXX"`).

### 2. Настроить Google Apps Script

1. Создать Google Sheet, добавить лист `Leads` с заголовками:

   | Время | Имя | Телефон | Пакет | Длина | Опции | Сроки | Чертёж | Min | Max | 3D | Горячий | UTM Source | UTM Medium | UTM Campaign | UTM Term | Landing URL | Referrer | User Agent | Экран | Timestamp |

2. Открыть **Extensions → Apps Script**

3. Вставить содержимое `apps-script/webhook.gs`

4. Заменить константы в начале файла:
   ```js
   const SPREADSHEET_ID = 'ID вашей таблицы'; // из URL таблицы
   const WEBHOOK_SECRET = 'ваш-секретный-ключ'; // тот же, что в index.html
   ```

5. **Deploy → New deployment → Web app**
   - Execute as: `Me`
   - Who has access: `Anyone`

6. Скопировать URL деплоя → вставить в `LEAD_WEBHOOK` в `index.html`

---

## Чеклист перед публикацией

- [ ] `WA_PHONE` — реальный номер
- [ ] `LEAD_WEBHOOK` — URL из Apps Script Deploy
- [ ] `WEBHOOK_SECRET` — одинаковый в HTML и в Apps Script
- [ ] Номер в `href="https://wa.me/..."` в trust-баре
- [ ] Проверить 6 сценариев (см. ниже)

### 6 сценариев для тест-прогона

1. Пустое имя → поле подсвечивается, отправки нет
2. Кривой номер → форма не уходит
3. Валидная заявка → строка в листе `Leads`
4. Тестовое имя (`test`, `тест`) → строка в листе `Rejected`
5. WhatsApp открывается с корректным текстом (пакет, длина, допы, квиз)
6. Мобильный: sticky CTA не перекрывает поля формы

---

## Аналитика

Страница автоматически вызывает события в Яндекс.Метрику, GA4 и Meta Pixel (если счётчики подключены через `<head>`):

| Событие | Когда |
|---|---|
| `view_calculator` | Загрузка страницы |
| `change_package` | Выбор пакета |
| `change_length` | Изменение длины |
| `toggle_extra` | Включение/выключение опции |
| `quiz_answer` | Ответ на вопрос квиза |
| `open_form_step2` | Клик по тизеру формы |
| `sticky_cta_click` | Клик по мобильной кнопке |
| `submit_form` | Отправка формы |
| `open_whatsapp` | Открытие WhatsApp |

---

## Технические детали

- **Чистый HTML/CSS/JS** — без фреймворков и сборки
- **Webhook timeout**: 2.5 сек — если Apps Script не ответил, WhatsApp всё равно открывается
- **Горячие лиды**: `is_hot: true` если выбрано "В этом месяце", пишется в таблицу отдельной колонкой
- **Антиспам**: валидация имени, телефона, enum-полей на стороне Apps Script; лист `Rejected` для отклонённых лидов
- **UTM**: `source`, `medium`, `campaign`, `term`, `content` читаются из URL автоматически
