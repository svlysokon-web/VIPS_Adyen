# Case 10. Chargeback process not defined

Статус на 10.11: **In progress**. Процесс согласован (Jan 10.11), технические вопросы закрыты (Rama 10.11).
Следующий шаг: Stas заводит US 0 to 6 в Azure DevOps, Rama начинает с US 0.

## Переписка (факты)
- 04.11 Jan: что происходит при чарджбэке, уведомят ли клиента и Support. Страница Disputes в Adyen Customer Area пустая, чарджбэков 0.
- 05.11 Rama: по чарджбэкам ничего не разработано, от Charles ничего не получено.
- 10.11 Stas: предложение по процессу и 3 вопроса разработчикам.
- 10.11 Jan: «Sounds like a good approach to me». Новое требование: в отеле можно зарегистрировать одного или нескольких сотрудников, которые получают уведомление по email.
- 10.11 Rama:
  1. Dispute webhooks пока не включены. Нужно разобраться, что включать для этой функции.
  2. Импорт payout report в AdyenPayoutReportDetails идёт без фильтров, поэтому строки чарджбэков «should already import». Не проверено, Rama предлагает проверить в Adyen test.
  3. PSP Reference сохраняется для платежей во всех каналах.

## Решения (согласованы 10.11)
1. Диспут ведёт сам отель. Сумма списывается с balance account отеля, с гостем говорит отель.
2. Отели управляют диспутами в VIPS Cloud PMS. Вариант реализации: Adyen Dispute components.
3. Уведомление всегда показывается в VIPS PMS. Дополнительно email сотрудникам отеля, которые подписаны. Напоминание за 3 дня до срока, если отель ещё не ответил.
4. Support уведомления не получает. Support видит все диспуты и чарджбэки в Central Admin и выгружает их (решение Stas 10.11).
5. Chargeback fee Adyen (если есть) перевыставляется отелю.
6. Процесс покрывает все каналы: терминалы, Payment Page, Pay by Link, CC и VCC.

## Что нашёл в документации Adyen (10.11)
- Для диспутов нужно подписаться на Standard webhook и включить dispute events. Для части событий нужна доп. настройка ([Dispute webhooks](https://docs.adyen.com/risk-management/disputes-api/dispute-notifications), [Manage disputes](https://docs.adyen.com/risk-management/manage-disputes)).
- Основные события:
  - NOTIFICATION_OF_CHARGEBACK: диспут открыт, начинается срок на защиту, в нём есть `defensePeriodEndsAt`.
  - CHARGEBACK: деньги списаны.
  - CHARGEBACK_REVERSED: защита принята, деньги вернулись (банк ещё может открыть second chargeback).
  - SECOND_CHARGEBACK: финальный проигрыш.
  - PREARBITRATION_WON: выигрыш в pre-arbitration (Visa allocation, UnionPay).
  - INFORMATION_SUPPLIED: документы загружены.

  Источник: [Webhook reference](https://docs.adyen.com/api-explorer/Webhooks/1/post/NOTIFICATION_OF_CHARGEBACK).
- Для Disputes API нужен API credential с ролью dispute management ([Disputes API](https://docs.adyen.com/risk-management/disputes-api)).
- Тестовый чарджбэк можно создать в test environment ([Test a chargeback scenario](https://docs.adyen.com/risk-management/disputes-api/test-applicable-dispute-reasons)).
- Сроки ответа: Visa 18 дней, Mastercard 40 дней, American Express 14 дней на RFI.
- **Важно для решений 1 и 5.** Для платформ списание чарджбэка задаётся настройкой `chargeback`: `deductFromLiableAccount` (с liable account платформы, то есть VIPS), `deductFromOneBalanceAccount` или `deductAccordingToSplitRatio`. Chargeback fee идёт на account из `chargebackCostAllocation`, по умолчанию это liable account платформы. Если balance account не найден, сумма и fee тоже уходят на liable account ([Split chargebacks](https://docs.adyen.com/platforms/online-payments/split-transactions/split-chargebacks)).
  Значит, «списание с отеля» и «fee на отель» работают, только если это так настроено. Иначе платит VIPS. Проверка вынесена в US 0.

## Что ещё не закрыто
1. **Pay by Link.** В отправленном письме 10.11 его нет в списке каналов, а 09.11 он был в scope. В US оставил Pay by Link.
2. **Chargeback fee.** Chen ответа не дал. Fee на отель по умолчанию не настроено (см. выше). Если fee списывается в Adyen с balance account отеля, перевыставлять в счёте VIPS не нужно. Это проверяется в US 0.
3. **Права в PMS (US 2).** Предложение: уведомления видят пользователи с правом Disputes в роли. Нужно подтвердить, есть ли в PMS такая настройка ролей или её надо добавить.
4. **Payout report.** Попадание строк чарджбэков в AdyenPayoutReportDetails не проверено, только ожидается. Проверка в US 0.

## Action Plan
| # | Шаг | Кто | Результат |
|---|-----|-----|-----------|
| 1 | Завести US 0 to 6 в Azure DevOps | Stas | US в бэклоге |
| 2 | US 0: включить webhooks, проверить настройки списания и fee, тестовый чарджбэк | Rama / Kondalu | Отчёт по AC US 0 |
| 3 | По итогам US 0 уточнить US 3 и US 4 (Adyen components или своя разработка) | Stas, Rama | Обновлённые US |
| 4 | Разработка US 1 to 6 | Kondalu | Пройденные AC |
| 5 | Сквозной тест в Adyen test: диспут от открытия до закрытия по каждому каналу | Kondalu, Stas | Протокол теста |
| 6 | Гайд для отелей и Support (Task) | Stas | Гайд в Drive |
| 7 | Закрыть кейс 10 в реестре | Stas | Status Closed |

**Критерий закрытия кейса:** US 0 to 6 и Task выполнены, сквозной тест пройден в Adyen test, первый диспут в live виден отелю в PMS.

---

## User Stories (final, EN, for Azure DevOps)

### US 0 (Spike). Enable dispute webhooks and check chargeback setup in Adyen
- **Story:** As the dev team, we want dispute events from Adyen turned on and the chargeback setup checked, so that we build the feature on facts.
- **Problem:** No dispute webhooks are enabled for our account (Rama, 10.11). It is not checked from which balance account Adyen takes the chargeback amount and the fee, and if chargeback lines reach AdyenPayoutReportDetails.
- **Value:** We know what Adyen sends, who pays, and what we already receive, before we build screens.
- **AC:**
  1. Standard webhook has dispute events enabled in test and live. The list of enabled event codes is documented.
  2. An API credential with the dispute management role exists (needed for the Disputes API and dispute components).
  3. The chargeback amount is taken from the hotel balance account (`chargeback` setting checked and documented).
  4. The chargeback fee goes to the hotel balance account (`chargebackCostAllocation` checked). If this is not possible, it is reported to Stas.
  5. A test chargeback is created in the Adyen test environment. Received webhooks are saved as examples.
  6. Chargeback lines from the Adyen payout report are present in AdyenPayoutReportDetails after the test.
  7. Short note if Adyen Dispute components can be used in VIPS Cloud PMS (prerequisites, effort).

### US 1. Receive and store dispute events
- **Story:** As VIPS, I want every Adyen dispute event stored and linked to the original payment, so that no chargeback is missed.
- **Problem:** Today a chargeback reaches nobody in VIPS.
- **Value:** Every dispute is in our data from the first event.
- **AC:**
  1. All dispute events from US 0 are accepted and acknowledged to Adyen.
  2. Each event is stored with PSP Reference, original PSP Reference, hotel, amount, currency, reason, stage, `defensePeriodEndsAt` and event date.
  3. The dispute is linked to the original payment and reservation by the original PSP Reference (stored for all channels, Rama 10.11).
  4. A repeated webhook does not create a duplicate.
  5. Failed processing is logged and can be retried.
  6. Works for terminals, Payment Page, Pay by Link, CC and VCC.

### US 2. Dispute notifications in PMS and by email
- **Story:** As a hotel user, I want to see a notice in VIPS PMS as soon as a guest disputes a payment, and get an email if I am subscribed, so that we act before the deadline.
- **Problem:** Nobody is notified about a dispute today (Jan, 04.11). Hotels handle their own disputes in PMS (decision 10.11).
- **Value:** No dispute is missed and no deadline passes because nobody saw it.
- **Notification logic:**

  | Adyen event | PMS notice | Email to subscribers |
  |---|---|---|
  | New dispute or request for information (NOTIFICATION_OF_CHARGEBACK, REQUEST_FOR_INFORMATION) | Right away | Right away |
  | Amount taken from the hotel balance (CHARGEBACK) | Same notice is updated | No |
  | Reminder, 3 days before the deadline, only if not yet accepted or defended | Once | Once |
  | Result (CHARGEBACK_REVERSED, SECOND_CHARGEBACK, PREARBITRATION_WON) | Right away | Right away |

- **AC:**
  1. PMS notices go to all users of the hotel who have the Disputes permission in their user role. Other hotels never see them.
  2. On a new dispute or a request for information, the notice shows right away in the PMS notification list, and the Disputes menu shows a counter of disputes that need action.
  3. The notice shows reservation, payment date, amount, reason, stage and deadline, and opens the dispute.
  4. Read status is per user. The counter stays until the dispute is accepted, defended or closed.
  5. One notice per event from the table. The same Adyen event received twice does not create a second notice or email.
  6. 3 days before the deadline, if the dispute is still not accepted or defended, one reminder notice and one reminder email are sent. No reminder if the hotel already answered.
  7. In hotel settings, one or more employees (email addresses) can subscribe to dispute emails (Jan, 10.11). Emails are an extra channel, PMS notices always show.
  8. If nobody is subscribed to dispute emails, hotel settings show a warning.
  9. Email texts are approved by Stas.

### US 3. Disputes screen in VIPS Cloud PMS
- **Story:** As a hotel user, I want a Disputes screen in PMS, so that I see all disputes and their status without Adyen Customer Area.
- **Problem:** Disputes are only visible in Adyen Customer Area, and hotels manage disputes in PMS (decision 10.11).
- **Value:** The hotel sees open disputes, deadlines and results in one place.
- **AC:**
  1. List per hotel with date, reservation, amount, reason, stage, deadline and status (open, won, lost, accepted).
  2. Filter by status and period. Disputes with a deadline in 3 days or less are highlighted.
  3. Click opens the reservation and the original payment.
  4. Status updates when Adyen sends a new stage or the result.
  5. Built with Adyen Dispute components if US 0 confirms they can be used, otherwise with our own screen.

### US 4. Accept or defend a dispute in PMS
- **Story:** As a hotel user, I want to accept or defend a dispute and upload documents in PMS, so that I do not need Adyen Customer Area.
- **Problem:** The hotel handles its own disputes in PMS (decision 10.11), but defending is only possible in Adyen Customer Area.
- **Value:** Hotels can win disputes on time from the tool they use every day.
- **AC:**
  1. The user can accept a dispute or pick a defense reason from the list Adyen returns.
  2. The user can upload the documents Adyen asks for (for example invoice, reservation, terms). Files follow Adyen limits on type and size.
  3. The defense is sent to Adyen before the deadline and the result is shown on the dispute.
  4. After the deadline, accept and defend are blocked with a clear message.
  5. Built with Adyen Dispute components if US 0 confirms they can be used.

### US 5. Chargebacks in hotel payouts and reports in PMS
- **Story:** As a hotel, I want chargebacks, reversals and fees visible in PMS, so that I understand why a payout is lower.
- **Problem:** The chargeback amount is taken from the hotel balance account. If it is missing in PMS, the hotel sees a lower payout without explanation.
- **Value:** The hotel can match its payout with Adyen without asking Support.
- **AC:**
  1. The chargeback and the reversal are shown in the payment history of the reservation.
  2. The hotel sees chargeback lines with the PSP Reference on the payout screen and in the payout report in PMS.
  3. The chargeback fee is shown as a separate line.
  4. PMS payout totals match the Adyen payout report for the same period.

### US 6. Disputes and chargebacks in Central Admin for Support
- **Story:** As VIPS Support, I want all disputes and chargebacks of all hotels in Central Admin, with export, so that I can answer a hotel and VIPS totals and invoices are correct.
- **Problem:** Support does not work in PMS and gets no dispute notices (decision 10.11). If chargebacks are missing in Central Admin, VIPS totals will not match Adyen (see cases 4 and 9).
- **Value:** Support finds any dispute fast without access to Adyen Customer Area, and VIPS invoices to hotels are correct.
- **AC:**
  1. Central Admin has a list of all disputes for all hotels with hotel, date, reservation, PSP Reference, amount, reason, stage, deadline and status.
  2. Filter by hotel, status and period.
  3. Export of the filtered list to Excel (xlsx or csv).
  4. Central Admin totals include chargebacks, reversals and fees and match the Adyen report for the same period.
  5. If Adyen does not take the chargeback fee from the hotel balance account (US 0), the fee is added to the VIPS invoice to the hotel.

### Task (not dev). Guide for hotels and Support
- Short guide: what a chargeback is, deadlines per scheme, how to register employees for notices, how to accept or defend in PMS, what Support can see. Saved in the Drive folder Adyen, link in register case 10.
