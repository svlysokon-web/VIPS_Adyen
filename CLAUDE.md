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
- **Стиль текста (реестр, письма и любые тексты для коллег):** не используй точку с
  запятой (;), вместо неё точка, запятая или «and». Без длинных тире (—). Избегай фраз,
  по которым текст узнают как написанный AI: «I hope this email finds you well»,
  «delve», «leverage», «seamless», «streamline», «robust», «comprehensive», «ensure»,
  «it's worth noting», «game-changer», «in today's fast-paced…», «I'm excited to…»,
  а также пафоса и шаблонных троек прилагательных. Пиши просто, как человек в рабочей
  переписке.
- **Даты** в тексте и в колонках дат пиши как `05.09` (ДД.ММ, формат ячеек `dd.mm`),
  не `05-Sep`. Месяц целиком можно словом (Aug, September).
- **Двоеточие** только там, где без него нельзя (время 00:04, цитата кода). Не пиши
  «Rama: open ticket», «Decided: …», «Proposal: …». Пиши «Rama to open a ticket»,
  «Decided to …», «Proposal is to …».
- Структура утверждена Stas 06-Nov-2026 («последняя версия»): **один лист**
  `Adyen Case Register`. Листов Actions, Summary, Patterns, Guide, Lists нет. Формул нет.
- Колонки (14, в этом порядке): ID, Problem, Category, Priority, Status, Initiated,
  Last update, Owner, Details, Resolution, Root cause, RCA, Next step, Blocker.
  - Problem — короткий симптом («что не так»), не причина.
  - Details — только факты, которых нет в Problem (отель, суммы, номера US/Solvers,
    даты). Не повторяй Problem.
  - Resolution — принятое решение или выполненный фикс («Decided 30.10 to …», «Fixed 19.10. …», «No decision yet»).
    Не цитируй переписку и не пиши, кто что сказал («Kondalu: …», «Jan proposes …»);
    исполнитель — в колонке Owner. Root cause — почему возникло (не дублируй в Resolution).
  - Пиши в формате отчёта: вывод своими словами, а не копия вопросов или фраз из
    письма. Root cause = найденные причины + что ещё не объяснено. Blocker — только
    реальная зависимость (не приписывай людей, которые в треде не участвуют).
  - Next step — следующее действие с именем («Jan to create the US»); Blocker — что/кто мешает (пусто, если ничего).
- Сортировка: по дате добавления (Initiated) по возрастанию; новый кейс — внизу.
- ID — числа 1, 2, 3… по порядку строк (на 06-Nov-2026 последний ID 12). При ссылке
  на другой кейс (#9) проверяй, что номер актуален.
- Выпадающие списки заданы прямо в валидации: Status, Priority, RCA, Category.
- Status (только 4): **Open** — ждём ответа/никто ещё не взял в работу;
  **In progress** — решение согласовано, идёт работа или анализ;
  **Blocked** — нельзя двигаться дальше из-за внешней зависимости или решения
  (указано в Blocker); **Closed** — решено и проверено.
  Статусы ставлю я по этим правилам для всех кейсов. Правки Stas в присланном файле
  могут быть просто примером, а не решением: если не уверен, уточни.
- RCA (4): **Confirmed** — причина найдена и подтверждена; **Partial** — причина
  найдена частично; **Not investigated** — причину ещё не искали; **Not a defect** —
  это вопрос, решение или новая функция, анализировать причину нечего (заменило N/A).
  Partial и Not investigated выделяются красным.
- Кейс закрывается только при проверенном фиксе и `RCA = Confirmed` (или Not a defect).
- Повторяющиеся темы держи в голове при анализе (в файл отдельным листом не выносить):
  надёжность загрузки данных (webhooks/payout reports), разные даты
  (ReportDate/PayoutDate/CreatedOn), нет общего референса (PSP Reference),
  не определены бизнес-процессы (chargebacks, FX), PCI/токенизация.

### Как обновлять реестр

1. Найди новые файлы в `Email cases` (по `createdTime`) и прочитай их (см. выше).
   Если Stas прислал свою версию xlsx — бери её за основу (он мог править вручную).
2. Новый тред → новая строка внизу со следующим номером. Продолжение существующего треда →
   обнови Resolution, Status, Root cause, RCA, Next step, Blocker, Last update.
3. Правь xlsx через openpyxl, стиль копируй с соседней строки (Arial 10, рамки, перенос
   текста, шапка #1F3864). Форматирование подготовлено до строки 149.
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
