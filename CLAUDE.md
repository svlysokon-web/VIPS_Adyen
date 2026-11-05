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
- **Главное требование Stas: лаконично, конкретно, понятно.** По строке нужно за секунды
  понять: в чём проблема, решена ли, кто решил, какая root cause; если не решена, то
  чего ждём, что блокирует, какой следующий шаг. Каждая ячейка — 1–2 коротких
  предложения, без пересказа переписки, без лишних деталей и колонок. Не добавляй
  обратно удалённые листы и колонки без просьбы.
- Листы (версия от 05-Nov-2026, согласована со Stas): только `Case Register` и `Actions`.
  Summary, Patterns, Guide и Lists Stas удалил. Формул в файле нет.
- Колонки Case Register (14): ID, Problem, Category, Priority, Status, Initiated, Owner,
  Summary, Answer / Resolution, Root cause, RCA, Waiting on / Blocker, Next step, Last update.
  Колонку Answer / Resolution для решённого кейса заполняй так: как исправили и кто.
- Actions: #, Case ID, Action, Owner, Due date, Status (Open / Proposed / In progress /
  Done / Cancelled).
- ID кейсов — числа 1, 2, 3… (на 05-Nov-2026 последний ID 13). В Actions `Case ID` — тот же номер.
- Выпадающие списки заданы прямо в валидации (без листа Lists): Status, Priority, RCA, Category.
- Статусы: New, Awaiting response, In analysis, Awaiting decision, In progress,
  Waiting on Adyen, Fix deployed - verify, Closed. Кейс закрывается только при
  верифицированном фиксе и `RCA = Confirmed` (или N/A для вопросов и решений).
- Повторяющиеся темы держи в голове при анализе (в файл отдельным листом не выносить):
  надёжность загрузки данных (webhooks/payout reports), разные даты
  (ReportDate/PayoutDate/CreatedOn), нет общего референса (PSP Reference),
  не определены бизнес-процессы (chargebacks, FX), PCI/токенизация.

### Как обновлять реестр

1. Найди новые файлы в `Email cases` (по `createdTime`) и прочитай их (см. выше).
   Если Stas прислал свою версию xlsx — бери её за основу (он мог править вручную).
2. Новый тред → новая строка со следующим номером. Продолжение существующего треда →
   обнови Answer / Resolution, Status, Root cause, RCA, Waiting on, Next step, Last update.
3. Правь xlsx через openpyxl, стиль копируй с соседней строки (Arial 10, рамки, перенос
   текста, шапка #1F3864). Форматирование подготовлено до строки 150.
4. Закоммить и отправь файл пользователю. В ответе дай короткую сводку: новые кейсы,
   смена статусов, что без ответа.

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
