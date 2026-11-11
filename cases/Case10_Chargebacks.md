# Case 10. Chargeback process not defined

Статус на 10.11: **In progress**. Процесс согласован (Jan 10.11), технические вопросы закрыты (Rama 10.11).
Следующий шаг: Stas заводит US 0 to 7 в Azure DevOps, Rama начинает с US 0.

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
- В отчётах Balance Platform чарджбэк идёт как category `platformPayment` с type `chargeback`, возврат как `chargebackReversal`, второй чарджбэк как `secondChargeback`. Fee идёт как `paymentCost`, в payout report есть строки «Chargeback Fee». Исходный платёж, диспут и списание связывает ARN ([Accounting Report](https://docs.adyen.com/marketplaces/reports-and-fees/balance-platform-accounting-report), [Payout report](https://docs.adyen.com/platforms/reports-and-fees/payout-report/)).
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
| 1 | Завести US 0 to 7 в Azure DevOps | Stas | US в бэклоге |
| 2 | US 0: включить webhooks, проверить настройки списания и fee, тестовый чарджбэк | Rama / Kondalu | Отчёт по AC US 0 |
| 3 | По итогам US 0 уточнить US 3 и US 4 (Adyen components или своя разработка) | Stas, Rama | Обновлённые US |
| 4 | Разработка US 1 to 6 | Kondalu | Пройденные AC |
| 5 | Сквозной тест в Adyen test: диспут от открытия до закрытия по каждому каналу | Kondalu, Stas | Протокол теста |
| 6 | US 7: гайды для отелей и Support | Stas | Гайды в Drive |
| 7 | Закрыть кейс 10 в реестре | Stas | Status Closed |

**Критерий закрытия кейса:** US 0 to 7 выполнены, сквозной тест пройден в Adyen test, первый диспут в live виден отелю в PMS.

---

## Feature (Azure DevOps)

**Title:** Adyen disputes and chargebacks

**Description:**

**Problem.** VIPS has no chargeback process. Dispute webhooks are not enabled, so nobody is notified when a guest disputes a payment. The amount is taken from the hotel balance account and the hotel only sees a lower payout. A missed deadline means the dispute is lost.

**Task.** Receive Adyen dispute events and notify the hotel in PMS and by email. Let hotels accept or defend disputes in PMS. Show chargebacks and fees in hotel payouts and in Central Admin. Covers all card payments.

---

## User Stories (final, EN, for Azure DevOps)

### US 0 . Enable dispute webhooks and check chargeback setup
- **Story:** As the dev team, we want dispute events from Adyen turned on and the chargeback setup checked, so that we build the feature on facts.
- **Problem:** No dispute webhooks are enabled for our account. Not checked from which balance account Adyen takes the chargeback amount and the fee, and if chargeback lines reach AdyenPayoutReportDetails.
- **Value:** We know what Adyen sends, who pays, and what we already receive, before we build screens.
- **Reference:** [Split chargebacks](https://docs.adyen.com/platforms/online-payments/split-transactions/split-chargebacks)
- **AC:**
  1. Standard webhook has dispute events enabled in test and live. The list of enabled event codes is documented.
  2. An API credential with the dispute management role exists (needed for the Disputes API and dispute components).
  3. The chargeback amount is taken from the hotel balance account (`chargeback`).
  4. The chargeback fee goes to the hotel balance account (`chargebackCostAllocation` checked).
  5. A test chargeback is created in the test environment. Received webhooks are saved as examples.
  6. Chargeback lines from the Adyen payout report are present in AdyenPayoutReportDetails after the test.

### US 1. Receive and store dispute events
- **Story:** As VIPS, I want every Adyen dispute event stored and linked to the original payment, so that no chargeback is missed.
- **Problem:** Today a chargeback reaches nobody in VIPS.
- **Value:** Every dispute is in our data from the first event.
- **Reference:** [Dispute webhooks](https://docs.adyen.com/risk-management/disputes-api/dispute-notifications)
- **AC:**
  1. VIPS accepts all Adyen dispute webhooks and confirms each one to Adyen with [accepted].
  2. Each event is stored with PSP Reference, original PSP Reference, hotel, amount, currency, reason, stage, `defensePeriodEndsAt` and event date.
  3. The dispute is linked to the original payment and reservation by the original PSP Reference.
  4. A repeated webhook does not create a duplicate.
  5. Failed processing is logged and can be retried.
  6. Works for terminals, Payment Page, Pay Link, CC and VCC.

### US 2. Dispute notifications
- **Story:** As a hotel user, I want to see a notice in VIPS PMS and get an email as soon as a guest disputes a payment, so that we act before the deadline.
- **Problem:** Nobody is notified about a dispute today.
- **Value:** No dispute is missed and no deadline passes because nobody saw it.
- **Notification logic:**

  | Adyen event | PMS notice | Email to subscribers |
  |---|---|---|
  | New dispute or request for information (NOTIFICATION_OF_CHARGEBACK, REQUEST_FOR_INFORMATION) | Right away | Right away |
  | Amount taken from the hotel balance (CHARGEBACK) | Same notice is updated | No |
  | Reminder, 3 days before the deadline, only if not yet accepted or defended | Once | Once |
  | Result (CHARGEBACK_REVERSED, SECOND_CHARGEBACK, PREARBITRATION_WON) | Right away | Right away |

- **Reference:** [NOTIFICATION_OF_CHARGEBACK webhook (defensePeriodEndsAt)](https://docs.adyen.com/api-explorer/Webhooks/1/post/NOTIFICATION_OF_CHARGEBACK)
- **AC:**
  1. PMS notices go to all users of the hotel who have the Disputes permission in their user role.
  2. On a new dispute or a request for information, the notice shows right away in the PMS notification list, and the Disputes menu shows a counter of disputes that need action.
  3. The notice shows reservation, payment date, amount, reason, stage and deadline, and opens the dispute.
  4. Read status is per user. The counter stays until the dispute is accepted, defended or closed.
  5. One notice per event from the table. The same Adyen event received twice does not create a second notice or email.
  6. 3 days before the deadline, if the dispute is still not accepted or defended, one reminder notice and one reminder email are sent. No reminder if the hotel already answered.
  7. In hotel settings, one or more employees (email addresses) can subscribe to dispute emails. Emails are an extra channel, PMS notices always show.
  8. If nobody is subscribed to dispute emails, hotel settings show a warning.
  9. Email texts are approved by PO.

### US 3. Disputes screen in VIPS Cloud PMS
- **Story:** As a hotel user, I want a Disputes screen in PMS, so that I see all disputes and their status.
- **Problem:** Disputes are not visible to hotel managers in the PMS.
- **Value:** The hotel sees open disputes, deadlines and results in one place.
- **Reference:** [Dispute components](https://docs.adyen.com/platforms/components-overview/after-payment/disputes-component)
- **AC:**
  1. List per hotel with date, reservation, amount, reason, stage, deadline and status (open, won, lost, accepted).
  2. Filter by status and period. Disputes with a deadline in 3 days or less are highlighted.
  3. Click opens the reservation and the original payment.
  4. Status updates when Adyen sends a new stage or the result.
  5. Built with Adyen Dispute components.

### US 4. Accept or defend a dispute in PMS
- **Story:** As a hotel user, I want to accept or defend a dispute and upload supporting documents directly in the PMS, so that I can manage dispute process in one place.
- **Problem:** Hotels cannot defend disputes or upload supporting documents directly in PMS.
- **Value:** Hotels can win disputes on time from PMS.
- **Reference:** [Disputes API](https://docs.adyen.com/risk-management/disputes-api)
- **AC:**
  1. The user can accept a dispute or pick a defense reason from the list Adyen returns.
  2. The user can upload the documents Adyen asks for (invoice, reservation, terms). Files follow Adyen limits on type and size.
  3. The defense is sent to Adyen before the deadline and the result is shown on the dispute.
  4. After the deadline, accept and defend are blocked with a clear message.
  5. Built with Adyen Dispute components.

### US 5. Chargebacks in hotel payouts and reports in PMS
- **Story:** As a hotel, I want chargebacks, reversals and fees visible in PMS, so that I understand why a payout is lower.
- **Problem:** The chargeback amount is taken from the hotel balance account. If it is missing in PMS, the hotel sees a lower payout without explanation.
- **Value:** The hotel can match its payout in PMS without asking Support.
- **Reference:** [Balance Platform Payout report](https://docs.adyen.com/platforms/reports-and-fees/payout-report/)
- **AC:**
  1. The chargeback and the reversal are shown in the payment history of the reservation.
  2. The hotel sees chargeback lines with the PSP Reference on the payout screen and in the payout report in PMS.
  3. The chargeback fee is shown as a separate line.
  4. PMS payout totals match the Adyen payout report for the same period.

### US 6. Disputes and chargebacks in Central Admin
- **Story:** As VIPS Support, I want all disputes and chargebacks of all hotels in Central Admin, with export, so that I can answer a hotel and VIPS totals and invoices are correct.
- **Problem:** Support does not work in PMS and gets no dispute notices. 
- **Value:** Support can quickly find any dispute without needing access to external systems, and VIPS invoices to hotels are correct.
- **Reference:** [Balance Platform Accounting Report](https://docs.adyen.com/marketplaces/reports-and-fees/balance-platform-accounting-report)
- **AC:**
  1. Central Admin has a list of all disputes for all hotels with hotel, date, reservation, PSP Reference, amount, reason, stage, deadline and status.
  2. Filter by hotel, status and period.
  3. Export of the filtered list to Excel (xlsx or csv).
  4. Central Admin totals include chargebacks, reversals and fees and match the Adyen report for the same period.

### US 7. User guides for hotels and Support
- **Story:** As a hotel user and as VIPS Support, I want a short guide on chargebacks, so that I know what to do when a dispute comes in and before the deadline.
- **Problem:** Chargebacks are new for hotels and Support. Without a guide, hotels will miss deadlines and Support will not know where to look.
- **Value:** Hotels handle disputes on their own and on time. Support can answer hotel questions without the dev team.
- **AC:**
  1. Hotel guide covers what a chargeback is, deadlines per card scheme (Visa 18 days, Mastercard 40 days, American Express 14 days for a request for information), and what happens if the deadline is missed.
  2. Hotel guide explains PMS notices and how to subscribe employees to dispute emails.
  3. Hotel guide explains the Disputes screen, how to accept or defend a dispute, which documents to upload, and where the chargeback and fee show in payouts.
  4. Support guide explains the dispute list in Central Admin, filters and export, and how to answer typical hotel questions such as a lower payout.
  5. Both guides use the format of the existing Adyen user guides.
