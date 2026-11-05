# VIPS × Adyen — project memory

## Источник данных проекта (Google Drive)

Главная папка проекта — **Google Drive «Adyen»**:

- URL: https://drive.google.com/drive/folders/1_-K5rVuj76l-TJELeV7mPlFwuzpAkj9w
- Folder ID: `1_-K5rVuj76l-TJELeV7mPlFwuzpAkj9w`
- Владелец: s.v.lysokon@gmail.com

Правила работы:

1. Это основной источник информации по проекту. Перед ответом на вопросы по Adyen
   (онбординг, терминалы, payment page, чарджбэки, отчётность, токенизация, статус)
   сначала смотри содержимое этой папки, а не отвечай по памяти.
2. Пользователь регулярно добавляет туда новые файлы. Список ниже — только снимок;
   актуальный состав всегда получай заново через Google Drive connector:
   - список файлов: `search_files` с запросом `parentId = '1_-K5rVuj76l-TJELeV7mPlFwuzpAkj9w'`
     (и рекурсивно для подпапок, пагинация через `pageToken`);
   - новые файлы: добавь `and modifiedTime > '<дата>'` или `createdTime > '<дата>'`;
   - содержимое: `read_file_content` (docx, xlsx, pdf, Google Docs/Sheets, изображения);
     для `.eml` — `download_file_content`.
3. Новые документы, которые создаются для проекта (отчёты, письма, гайды),
   по умолчанию сохраняй в эту же папку (или в подходящую подпапку), если пользователь
   не указал иное.

### Структура (снимок на 2026-11-05)

- `Email cases/` (ID `1QbKczi2zJMeHTNJ35J8MPdLMNYClDJHV`) — переписка по кейсам (.eml):
  чарджбэки, payment references в отчётах, расхождения reporting/invoicing PMS vs
  Centrals admin, tokenization endpoint в prod, показ только последних 4 цифр карты и др.
- `Adyen Status Update.xlsx` — статус проекта
- `Adyen_Onboarding_User_Guide_Final.docx`
- `Adyen_In-Person_Terminals_Final User guide.docx`
- `Adyen_Payment page_User Manual.docx`
- `Adyen_Complete_User_Guide-….docx`
- `Adyen Checklist-….docx` (2 версии)
- `image007.png`
