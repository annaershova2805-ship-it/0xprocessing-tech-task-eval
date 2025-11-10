# Отчёт о проверке интеграции и платежей 0xProcessing  
**Автор:** Anna Milovanova  
**Дата:** 10 ноября 2025  
**Среда:** sandbox 0xProcessing  
**Цель:** проверить корректность интеграции, вебхуков, платежей и выводов.

---

# 1. Депозит с фиксированной суммой
Документация:  
**Payment form with fixed amount**  
https://docs.0xprocessing.com/0xprocessing-api/deposits/payment-form-with-fixed-amount

### Проверки
- Корректная генерация формы оплаты
- Отображение QR-кода
- Правильные параметры (`amount`, `currency`, `clientId`, `merchantId`)
- Таймер оплаты 30-50 минут
- Правильный переход по `SuccessURL` / `CancelURL`

После оплаты:
- Вебхук приходит со статусом **success**
- Поле `totalAmount` учитывает network fee  
Webhook:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks/payment-form-with-fixed-amount

**Недоплата / Переплата:**  
Статус `insufficient`, возможность доплаты:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks/payment-form-with-fixed-amount#payment-underpayment

---

# 2. Депозит без фиксированной суммы
Документация:  
**Payment without fixed amount**  
https://docs.0xprocessing.com/0xprocessing-api/deposits/payment-without-fixed-amount

Webhook:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks/payment-form-without-fixed-amount

### Проверки
- Генерация уникального адреса и `minimumAmount`
- Платёж выше минимального → success
- Платёж ниже минимального → `insufficient=true`

---

# 3. Статический кошелёк
Документация:  
**Static Wallet**  
https://docs.0xprocessing.com/0xprocessing-api/deposits/static-wallet

Webhook:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks/static-wallet

### Проверки
- Генерация постоянного адреса
- Корректность QR-кода
- Несколько платежей → один адрес, правильные суммы
- Поддержка `DestinationTag` для XRP и Memo для TON

---

# 4. Обработка вебхуков  
Общий раздел вебхуков:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks

Алгоритм MD5 подписи:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks#signature-validation

### Проверки
- Ответ сервера ≤ 3 сек → иначе до 31 повторной попытки
- MD5-подпись: сортировка, конкатенация, вычисление
- Правильность статусов:
  - success  
  - canceled  
  - insufficient  
- Наличие полей: `txHashes`, `billingID`, `clientId`

---

# 5. Вывод средств
Документация:  
**Withdrawals main page**  
https://docs.0xprocessing.com/0xprocessing-api/withdrawals/

**Withdrawal by client**  
https://docs.0xprocessing.com/0xprocessing-api/withdrawals/withdrawal-by-client

**Withdrawal status**  
https://docs.0xprocessing.com/0xprocessing-api/withdrawals/withdrawal-status-by-id

Webhook:  
https://docs.0xprocessing.com/0xprocessing-api/webhooks/withdrawals

### Проверки
- Создание вывода с валидными параметрами
- Статус `pending`
- Подтверждение, если `autoWithdraw=false`
- Статусы: success, aborted, rejected
- Ошибки: неверный адрес, low balance

---

# 6. Система VRCS (автоконвертация)

Документация:  
https://docs.0xprocessing.com/quick-start/vrcs-setup

### Проверки
- Активация VRCS
- Платежи с автоконвертацией → поля:  
  `isVRCS`, `convertedCurrency`, `amountUSD`
- Отключение VRCS → платежи в исходной валюте

---

# 7. Информационные запросы и конвертации

**GetBalances**  
https://docs.0xprocessing.com/quick-start/balance-management

**CoinInfo**  
https://docs.0xprocessing.com/0xprocessing-api/informational-commands/coin-info

**Fiat/Crypto Conversion**  
https://docs.0xprocessing.com/0xprocessing-api/crypto-or-fiat-equivalent/

### Проверки
- Актуальные балансы + VRCS баланс
- Минимальные суммы
- Commission / network fee
- /Convert, /ConvertToCrypto, /ConvertCryptoToFiat  
  Ограничение: 500 мс между запросами

---

# 8. Обработка ошибок

**Errors section**  
https://docs.0xprocessing.com/0xprocessing-api/errors

### Проверки
- MerchantIdRequired  
- Incorrect amount  
- UnsupportedCurrency  
- Некорректная подпись  
- Пустые поля
- Соответствие HTTP-кодов  
- Читаемые сообщения для логирования

---

# 9. Безопасность

**Account security**  
https://docs.0xprocessing.com/quick-start/account-security

**Regex для проверок адресов**  
https://docs.0xprocessing.com/quick-start/match-crypto-wallet-addresses

### Проверки
- API key хранится в переменных окружения  
- HTTPS  
- IP restrictions  
- 2FA  
- Правильность регулярных выражений:  
  - Tron  
  - Dogecoin  
  - Litecoin  
  - Solana  
  - TON  
  - XRP  

---

# Итоги тестирования

Платёжные и выводные сценарии работают корректно:  
- депозиты фикс / нефикс  
- статические кошельки  
- VRCS  
- вебхуки  
- информационные команды  
- обработка ошибок  

---

# Предложения по улучшению документации

## 1. Структура
- Создать единый справочник валют  
- Добавить глоссарий  
- Улучшить сквозные ссылки между разделами
- Перелинковать Deposits ↔ Webhooks ↔ Errors

## 2. Содержание
- Расширить описание VRCS  
- Добавить примеры curl/Node/Python  
- Визуализировать потоки (диаграммы)

## 3. Локализация
- Полный русский перевод  
- Указывать дату последнего обновления страниц

## 4. Негативные сценарии
- Расширить описание `insufficient` / `canceled`  
- Добавить практические кейсы

## 5. Безопасность
- Добавить раздел о хранении API-ключей  
- Пример расчёта MD5 подписи  
- Перенести regex ближе к Withdrawals  
- Добавить советы по 2FA/IP ограничению

---

# Конец отчёта  
**Anna Milovanova | 2025**

