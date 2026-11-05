# VIPS × Adyen — project memory

Проект: внедрение Adyen (VIPS Pay) в VIPS PMS. Stanislav Lysokon (Stas, пользователь)
ведёт мониторинг всех Adyen-кейсов по поручению Stefan van Welie (Director of Technology & Product).

## Источник данных проекта (Google Drive)

Главная папка проекта — **Google Drive «Adyen»**:

- URL: https://drive.google.com/drive/folders/1_-K5rVuj76l-TJELeV7mPlFwuzpAkj9w
- Folder ID: `1_-K5rVuj76l-TJELeV7mPlFwuzpAkj9w`
- Подпапка `Email cases`: ID `1QbKczi2zJMeHTNJ35J8MPdLMNYClDJHV`. Сюда Stas сохраняет
  переписку по кейсам (.eml).
- Владелец: s.v.lysokon@gmail.com

Правила работы:

1. Это основной источник информации по проекту. По вопросам об Adyen (онбординг,
   терминалы, payment page, чарджбэки, отчётность, токенизация, статус) сначала смотри
   эту папку, а не отвечай по памяти.
2. Пользователь регулярно добавляет туда новые файлы. Состав папки всегда получай заново:
   - список: `search_files` с `parentId = '<folder id>'` (по подпапкам тоже, с пагинацией через `pageToken`);
   - новые файлы: добавь `and createdTime > '<дата>'` или `modifiedTime > '<дата>'`;
   - docx/xlsx/pdf/Google-файлы: `read_file_content`;
   - `.eml` через `read_file_content` не читаются. Используй `download_file_content`:
     письма тяжёлые (вложенные картинки, 0.3–4 МБ), и результат сохраняется в
     `tool-results/*.txt` (JSON `{content: base64}`). Декодируй его Python-скриптом
     (`base64` → `email.message_from_bytes` → text/plain body, вложения через
     `iter_attachments()`) и не печатай base64 в разговор. Маленькие письма (<~40 КБ)
     приходят прямо в разговор, поэтому сначала скачивай крупные.
3. Новые документы проекта по умолчанию сохраняй в эту папку или в подходящую подпапку,
   если пользователь не сказал иначе. Ограничение: Drive connector не может перезаписать
   содержимое существующего файла, а загрузка бинарного xlsx через base64 в вызове
   ненадёжна. Поэтому Excel отдавай пользователю файлом (и коммить в репозиторий), а
   загрузку в Drive предлагай сделать вручную.

## Adyen Case Register (главный рабочий документ)

- Файл: `registers/Adyen_Case_Register.xlsx` в этом репозитории (мастер-копия);
  копию для коллег пользователь кладёт в Drive-папку «Adyen».
- Цель: держать стейкхолдеров в курсе, что в работе, что без ответа, какая root cause и
  как именно исправили; искать повторяющиеся проблемы (patterns). Stefan в Teams
  (04-Nov-2026) жаловался, что India-команда «чинит по комментариям Jan», не делает RCA
  и не ищет паттерны. Реестр должен это закрывать.
- Язык содержимого: английский (коллеги NL/IN), общение со Stas на русском.
- Листы: `Summary` (KPI на формулах COUNTIF/COUNTIFS), `Case Register` (1 строка = 1 кейс),
  `Actions` (действия: owner, due, status), `Patterns & RCA` (P1–P5), `Guide`, `Lists`
  (выпадающие списки, базовый URL трекера в `Lists!B3`, дата обновления в `Lists!B4`).
- ID кейсов: `ADY-001`… (на 05-Nov-2026 последний ID ADY-013). Actions: `A-01`…
- Колонки Case Register: ID, Title, Category, Type, Hotel/Property, Priority, Status,
  Initiated, Initiated by, Question/Issue summary, Response/Current answer, Root cause,
  RCA status, Action plan/Fix, Next step, Owner, Involved, Waiting on, Last update,
  Target date, Closed date, Days open*, Days since update*, Work item type, Work item #,
  Work item link*, Sprint/Release, Recurring?, Pattern, Related cases, Related Status
  Update item, Source e-mail (Drive, hyperlink), Notes. (* = формулы, не перезаписывать.)
- Статусы: New, Awaiting response, In analysis, Awaiting decision, In progress,
  Waiting on Adyen, Fix deployed - verify, Closed. Кейс закрывается только при
  верифицированном фиксе и `RCA status = Confirmed` (или N/A для вопросов и решений).
- Паттерны: P1 надёжность загрузки данных (webhooks/payout reports, нет идемпотентности
  и мониторинга), P2 нет единой даты (CreatedOn/ReportDate/PayoutDate…), P3 нет общего
  референса (PSP Reference), P4 не определены бизнес-процессы (chargebacks, FX, invoicing),
  P5 PCI/токенизация.

### Как обновлять реестр

1. Найди новые файлы в `Email cases` (по `createdTime`) и прочитай их (см. выше).
2. Новый тред → новая строка `ADY-0NN`. Продолжение существующего треда → обнови
   Response, Status, Last update, Next step, Root cause / RCA status того же кейса.
3. Правь xlsx через openpyxl (`load_workbook`, без `data_only`), стиль копируй с
   соседней строки. Формулы в колонках Days open / Days since update / Work item link
   и на листе Summary не трогай; при добавлении строк за пределами форматированного
   диапазона (до строки 150) протяни формулы, валидацию и автофильтр.
4. Обнови `Lists!B4` (Last updated) и привяжи кейс к Pattern.
5. Запусти recalc (скилл xlsx, `scripts/recalc.py`), проверь 0 ошибок, закоммить и
   отправь файл пользователю. В ответе дай короткую сводку изменений (новые кейсы,
   смена статусов, что без ответа).

## Люди

- VIPS: Stefan van Welie (Director of Technology & Product), Jan Paauwe (Product
  Strategist, автор большинства кейсов, пишет US), Eedukondalu «Kondalu» Dupati
  (Software Engineer II, Hyderabad), Rama Kiran Seru (Principal Engineer),
  Vijay Kumar Oruganti, Shyambabu Kolipaka, Chen Arets (бизнес-решения по
  фактурации/ценам), Coen Vogelaar.
- Adyen: Kobe Stienaers (Account Manager), York Broekhuizen (Implementation Engineer).
- Клиент: Tessa Meerkerk (FreeTime Hospitality; аккаунт Adyen зарегистрирован на
  FreeTime Hospitality).
- Трекер: user stories (например US 27124) и Solvers cases (например 27119); спринты
  75–77, версия v77. Базовый URL трекера пока неизвестен.

## Структура Drive-папки (снимок на 2026-11-05)

- `Email cases/` — 12 тредов .eml (токенизация/PCI, маскирование карт, пропущенные
  транзакции, расхождения PMS vs Central Admin, invoice data, Monday meeting, payout
  screen, payment references, дубли payout report, chargebacks, currency exchange,
  consolidate pending issues).
- `Adyen Status Update.xlsx` — статус фич (13 пунктов: settlement/invoicing, PayLink,
  payout & account mgmt, payment methods).
- `Adyen_Onboarding_User_Guide_Final.docx`, `Adyen_In-Person_Terminals_Final User guide.docx`,
  `Adyen_Payment page_User Manual.docx`, `Adyen_Complete_User_Guide-….docx`,
  `Adyen Checklist-….docx` (2 версии), `image007.png`.
