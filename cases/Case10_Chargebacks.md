# Case 10. Chargeback process not defined

Action Plan, вопросы и User Stories. Подготовлено 09.11 по письму «Adyen Chargebacks» (04.11), ответу Rama (05.11) и ответам Stas (09.11).

## Источники (факты)
- Письмо «Adyen Chargebacks», Jan Paauwe, 04.11. To Rama, Kondalu. CC Vijay, Stefan, Stas, Chen, Tessa.
  Вопросы Jan: документировал ли Charles или разработано ли что-то по чарджбэкам, что происходит
  сегодня при чарджбэке (пока 0), уведомит ли кто-то клиента и будет ли follow up, получит ли
  уведомление Support. Страница Disputes пустая.
- Ответ Rama 05.11: по чарджбэкам ничего не разработано, от Charles ничего не получено.
- Ответы Stas 09.11: страница Disputes = Adyen Customer Area. Сумму чарджбэка несёт отель
  (списание с balance account отеля). С гостем говорит отель. Кто обрабатывает диспут, пока не решено.
- Реестр, строка 10: Open, High, RCA Confirmed, Blocker «Process definition».
- В документах Drive (гайды, Checklist, Status Update) чарджбэки не упоминаются.
- Сроки ответа по документации Adyen (зависят от схемы):
  - [Visa](https://docs.adyen.com/risk-management/chargeback-guidelines/visa-chargebacks): 18 дней на защиту (9 дней только для локальных платежей US/Canada с 21.07.2025).
  - [Mastercard](https://docs.adyen.com/risk-management/chargeback-guidelines/mastercard-chargebacks): 40 дней.
  - [American Express](https://docs.adyen.com/risk-management/chargeback-guidelines/american-express-chargebacks): 14 дней на ответ на RFI. Без ответа на RFI следующий чарджбэк оспорить нельзя.
- У Adyen для платформ есть [Dispute components](https://docs.adyen.com/platforms/components-overview/after-payment/disputes-component) (Dispute Overview, Dispute Management). Их встраивают в свой интерфейс: список диспутов, сроки, accept, загрузка документов. Доступны ли они для нашей интеграции, не известно.

## Суть кейса (для Stas, по-русски)
Чарджбэк означает, что гость оспаривает карточный платёж через свой банк. Adyen списывает сумму
(по ответу Stas, с balance account отеля) и даёт срок на защиту. Сейчас в VIPS для этого нет
ничего: ни процесса, ни разработки, ни документации. Никто не получает уведомление, никто не
отвечает отелю и гостю, Support не знает. Пока чарджбэков 0, поэтому проблема не видна, но первый
же диспут пройдёт незамеченным: отель увидит меньшую выплату без объяснения, а срок на защиту
истечёт. Root cause подтверждён: функцию не разрабатывали и процесс не описан.

Общая справка по Adyen (не факт по VIPS, проверить с Adyen): диспут проходит стадии Request for
Information, Notification of Chargeback, Chargeback, при защите Second chargeback / Pre-arbitration.
Adyen шлёт webhooks (CHARGEBACK, NOTIFICATION_OF_CHARGEBACK, REQUEST_FOR_INFORMATION,
CHARGEBACK_REVERSED, SECOND_CHARGEBACK, NOTIFICATION_OF_FRAUD и др.). Для платформ в split
configuration задаётся, с какого balance account списывается чарджбэк. Защита идёт через Customer
Area или Disputes API (accept, defend, supply documents). У каждой стадии есть срок ответа.

## Action Plan
| # | Шаг | Кто (предложение) | Результат |
|---|-----|-------------------|-----------|
| 1 | Собрать недостающие факты (вопросы ниже) | Stas | Ответы Adyen, Rama, Jan |
| 2 | Решение: кто ведёт диспут (отель / VIPS Support), кого уведомляем, канал, срок реакции | Jan, Stefan, Chen | Зафиксированное решение |
| 3 | Описать процесс (RACI + шаги от webhook до закрытия) | Stas, Jan | Документ процесса, ссылка в реестре |
| 4 | Spike: проверить конфигурацию Adyen и что сейчас приходит в webhooks | Rama / Kondalu | Отчёт: события, split, тест в test env |
| 5 | US в Azure DevOps (ниже) и оценка | Jan создаёт US, Rama оценивает | US в спринте |
| 6 | Разработка и тест в Adyen test environment (тестовый диспут) | Kondalu | Пройденные AC |
| 7 | Инструкция для Support и отелей, обновление гайдов | Stas / Support | Гайд |
| 8 | Закрыть кейс в реестре после проверенного теста | Stas | Status Closed |

## Нужна информация (вопросы)
**Jan / Stefan / Chen (бизнес-решения)**
1. Кто обрабатывает диспут: отель сам или VIPS Support от имени отеля?
2. Где отели управляют диспутами: в VIPS Cloud PMS (например, через Adyen Dispute components) или на странице Disputes в Adyen как отдельном инструменте?
3. Кого и как уведомлять (отель, Support, email, экран в PMS)? В какой срок реагировать?
4. Кто платит chargeback fee Adyen, если он есть? Перевыставляем отелю? Есть ли это в договоре с Adyen (Chen)?
5. В scope и терминалы, и Payment Page, и PayLink?

**Rama / Kondalu (техника)**
6. Какие dispute webhooks включены, принимаем ли их сейчас и где они логируются?
7. Попадают ли чарджбэки в payout report import (AdyenPayoutReportDetails) и в Central Admin?
8. Как сейчас связан платёж с бронью (PSP Reference)? (связано с кейсом 8)
9. Можно ли создать тестовый диспут в Adyen test environment?

**Jan**
10. Area / Iteration path и родительская Feature в Azure DevOps.

**Adyen (Kobe / York), в последнюю очередь**
Пункты 4, 6 и 9, если внутри VIPS ответа нет.

## User Stories (EN, черновик для Azure DevOps)

**US 0 (Spike). Check Adyen dispute setup and current webhook handling**
- Story: As the dev team, we want to know which dispute events Adyen sends us and how we handle them today, so that we can size the chargeback work.
- Problem: Nothing was developed for chargebacks (Rama, 05.11). It is not known which dispute webhooks are enabled and what happens with them.
- Value: Estimates and design based on facts, not guesses.
- AC:
  1. List of dispute event codes enabled for the VIPS merchant account.
  2. Confirmation of which balance account is debited for a chargeback and for the fee.
  3. Current handling of these webhooks in the code is described (stored, logged or ignored).
  4. A test dispute is created in the Adyen test environment and the received webhooks are documented.

**US 1. Receive and store Adyen dispute notifications**
- Story: As VIPS, I want every Adyen dispute notification stored and linked to the original payment, so that no chargeback is missed.
- Problem: If a chargeback happens today nobody in VIPS knows about it.
- Value: Every dispute is visible in our data from day one.
- AC:
  1. All dispute events from US 0 are accepted and acknowledged to Adyen.
  2. Each event is stored with PSP Reference, original PSP Reference, hotel, amount, currency, reason code, stage, defense deadline and event date.
  3. The dispute is linked to the original payment and reservation by PSP Reference.
  4. A repeated webhook does not create a duplicate record.
  5. Failed processing is logged and can be retried.

**US 2. Notify hotel and Support about a new dispute**
- Story: As a hotel (and Support), I want to get a notice when a guest disputes a payment, so that we can act before the deadline.
- Problem: Nobody notifies the hotel or Support (Jan, 04.11).
- Value: No dispute is lost because nobody saw it.
- AC:
  1. On a new dispute or a new stage, a notice goes to the recipients defined in the process (decision pending, see question 1 and 2).
  2. The notice shows reservation, guest payment date, amount, reason, stage and deadline.
  3. A reminder is sent X days before the deadline if the dispute is still open (X to be decided).
  4. Notice texts are approved by Jan.

**US 3. Disputes overview in PMS**
- Story: As a hotel user, I want a Disputes screen in PMS, so that I see all disputes and their status without Adyen Customer Area.
- Problem: Disputes are only in Adyen Customer Area and that page is not used today.
- Value: The hotel and Support see open disputes, deadlines and outcome in one place.
- AC:
  1. List per hotel with date, reservation, amount, reason, stage, deadline, status (open, won, lost, accepted).
  2. Filter by status and period. Open disputes with a near deadline are highlighted.
  3. Click opens the reservation and original payment.
  4. Status updates when Adyen sends a new stage or the outcome.

**US 4. Show chargebacks in balance, payouts and reports**
- Story: As a hotel and as Finance, I want chargeback debits and reversals to show in the reservation, payout and reports, so that totals match Adyen.
- Problem: The chargeback amount is debited from the hotel balance account. Without it in PMS and Central Admin, payouts will not match (see cases 4 and 9).
- Value: The hotel understands why a payout is lower and reports stay correct.
- AC:
  1. A chargeback debit and a reversal are shown on the reservation payment history.
  2. The payout screen and payout report show chargeback lines separately with the PSP Reference.
  3. Central Admin totals include chargebacks and match the Adyen report for the same period.
  4. Chargeback fee is shown separately (if Adyen charges one).

**US 5. Accept or defend a dispute from PMS (only if decided that disputes are handled in PMS)**
- Story: As the dispute owner, I want to accept or defend a dispute and upload documents in PMS, so that we do not need Adyen Customer Area.
- Problem: Depends on the decision who handles disputes. Not decided yet.
- Value: Higher chance to win disputes, one tool for the hotel.
- AC:
  1. User can accept a dispute or choose a defense reason from the list Adyen returns.
  2. User can upload the documents Adyen requires (invoice, reservation, signed terms, etc.).
  3. Defense is sent via Adyen Disputes API before the deadline and the result is shown.
  4. Action is blocked after the deadline with a clear message.

**Task (not dev). Document the chargeback process**
- Process document (RACI, steps from notice to closing, deadlines) and a short guide for Support and hotels. Link in register case 10.

## Email to colleagues (09.11, v2)

### Кому переадресовать вопросы Adyen внутри VIPS (предложение, не факт)
| Вопрос Adyen | Кому внутри VIPS | Почему |
|---|---|---|
| Кто несёт chargeback fee, что в договоре | Chen | Решения по фактурации и ценам, договор с Adyen |
| Какие dispute webhooks включены | Rama | Principal Engineer, настройка webhooks |
| Как сымитировать диспут в тестовой среде | Kondalu | Работает с test environment |
Если внутри ответа нет, Stas пересылает эти пункты Kobe и York.

### Email text

**To** Jan Paauwe, Rama Kiran Seru, Eedukondalu Dupati, Chen Arets
**CC** Stefan van Welie
**Subject** Adyen chargebacks, open questions before we define the process

Hi all,

I am picking up the chargeback topic from Jan's email of 04.11. Rama confirmed on 05.11 that nothing has been built for chargebacks yet and there is no documentation from Charles. We have had 0 chargebacks so far, but when the first one comes, nobody will be notified and the hotel will only see a lower payout.

What we know so far. The chargeback amount is taken from the hotel balance account, and the hotel is the one who talks to the guest. Deadlines to respond depend on the card scheme. Visa gives 18 days, Mastercard 40 days, and for American Express we have 14 days to answer a request for information. If we miss that request, we cannot defend the chargeback later.

To describe the process and write the user stories, I need a few answers first.

**Jan, Stefan, Chen (process decisions)**
1. Who handles a dispute, the hotel itself or VIPS Support on behalf of the hotel?
2. Where will hotels manage disputes, in VIPS Cloud PMS or in the Adyen Disputes page as a separate tool? Adyen has ready dispute components for platforms that can be added to our own screens (https://docs.adyen.com/platforms/components-overview/after-payment/disputes-component).
3. Who should be notified (hotel, Support, both), how (email, PMS screen) and how fast do we need to react?
4. Who pays the Adyen chargeback fee, if there is one? Do we charge it to the hotel? Chen, is this in our agreement with Adyen?
5. Is this for all channels, terminals, Payment Page and PayLink?

**Rama**
6. Which dispute webhooks are turned on for our account, and do we receive them today? If yes, where are they stored or logged?
7. Do chargebacks show up in AdyenPayoutReportDetails and in Central Admin?
8. How do we link a payment to a reservation today? Is it by PSP Reference? (this is also open in the payment references case)

**Kondalu**
9. Can we create a test dispute in the Adyen test environment, so we can see the webhooks and the flow?

**Jan**
10. For the user stories in Azure DevOps, which Area path and parent Feature should I use?

If we cannot answer 4, 6 or 9 ourselves, I will ask Kobe and York at Adyen.

Thanks,
Stas
