# Объекты базы данных: функции, триггеры, представления

## 1. Обзор

Данный документ описывает все программные объекты базы данных:
- Функции и процедуры
- Триггеры
- Материализованные представления
- Обычные представления (views)

**Примечание:** Функции для проверки качества данных и сверок вынесены в отдельный документ: [data-quality-checks.md](./data-quality-checks.md)

---

## 2. Функции и процедуры

### 2.1. Функции управления партициями

#### 2.1.1. `invest_utils.create_minute_candles_partition(p_date DATE)`
**Схема:** `invest_utils`  
**Тип:** FUNCTION  
**Возвращает:** TEXT

**Описание:** Создает дневную партицию для таблицы `invest_candles.minute_candles` на указанную дату.

**Параметры:**
- `p_date` (DATE) - Дата, для которой создается партиция

**Логика:**
- Формирует имя партиции: `minute_candles_YYYY_MM_DD`
- Определяет диапазон: от предыдущего дня 21:00 UTC до текущего дня 21:00 UTC
- Проверяет существование партиции перед созданием
- Создает партицию с установкой владельца

**Пример использования:**
```sql
SELECT invest_utils.create_minute_candles_partition('2025-01-15');
```

**Возвращаемое значение:** Сообщение о результате операции

---

#### 2.1.2. `invest_utils.create_daily_candles_partition(p_date DATE)`
**Схема:** `invest_utils`  
**Тип:** FUNCTION  
**Возвращает:** TEXT

**Описание:** Создает месячную партицию для таблицы `invest_candles.daily_candles` на указанную дату.

**Параметры:**
- `p_date` (DATE) - Дата, для которой создается партиция (определяется месяц)

**Логика:**
- Определяет начало месяца указанной даты
- Формирует имя партиции: `daily_candles_YYYY_MM`
- Определяет диапазон: от последнего дня предыдущего месяца 21:00 UTC до последнего дня текущего месяца 21:00 UTC
- Проверяет существование партиции перед созданием

**Пример использования:**
```sql
SELECT invest_utils.create_daily_candles_partition('2025-01-15');
```

---

#### 2.1.3. `invest_utils.create_close_prices_partition(p_date DATE)`
**Схема:** `invest_utils`  
**Тип:** FUNCTION  
**Возвращает:** TEXT

**Описание:** Создает месячную партицию для таблицы `invest_prices.close_prices` на указанную дату.

**Параметры:**
- `p_date` (DATE) - Дата, для которой создается партиция

**Логика:** Аналогична `create_daily_candles_partition`, но для таблицы цен закрытия.

---

#### 2.1.4. `invest_utils.create_open_prices_partition(p_date DATE)`
**Схема:** `invest_utils`  
**Тип:** FUNCTION  
**Возвращает:** TEXT

**Описание:** Создает месячную партицию для таблицы `invest_prices.open_prices` на указанную дату.

**Параметры:**
- `p_date` (DATE) - Дата, для которой создается партиция

---

#### 2.1.5. `invest.create_last_prices_daily_partition(p_date DATE)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** TEXT

**Описание:** Создает дневную партицию для таблицы `invest.last_prices` на указанную дату.

**Параметры:**
- `p_date` (DATE) - Дата, для которой создается партиция

---

#### 2.1.6. `invest_utils.cleanup_old_daily_partitions(retention_days INTEGER DEFAULT 730)`
**Схема:** `invest_utils`  
**Тип:** FUNCTION  
**Возвращает:** VOID

**Описание:** Удаляет старые дневные партиции таблицы `last_prices`, старше указанного количества дней.

**Параметры:**
- `retention_days` (INTEGER) - Количество дней хранения (по умолчанию 730 дней = 2 года)

**Логика:**
- Определяет дату отсечения: `CURRENT_DATE - retention_days`
- Находит все партиции старше этой даты
- Удаляет найденные партиции
- Логирует операции через RAISE NOTICE

**Пример использования:**
```sql
SELECT invest_utils.cleanup_old_daily_partitions(365); -- Удалить партиции старше года
```

---

### 2.2. Функции вставки данных

#### 2.2.1. `invest.bulk_insert_last_prices(p_figi VARCHAR, p_time TIMESTAMP, p_price NUMERIC, p_currency VARCHAR, p_exchange VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** VOID

**Описание:** Оптимизированная вставка данных в таблицу `last_prices` с автоматическим созданием партиции при необходимости.

**Параметры:**
- `p_figi` (VARCHAR) - FIGI инструмента
- `p_time` (TIMESTAMP) - Время сделки
- `p_price` (NUMERIC) - Цена
- `p_currency` (VARCHAR) - Валюта
- `p_exchange` (VARCHAR) - Биржа

**Логика:**
- Определяет дату из `p_time`
- Проверяет существование дневной партиции
- Создает партицию через `create_last_prices_daily_partition`, если не существует
- Вставляет данные с обработкой конфликтов (`ON CONFLICT DO NOTHING`)

**Пример использования:**
```sql
SELECT invest.bulk_insert_last_prices(
    'BBG004S68DD6',
    '2025-01-15 10:30:00',
    150.50,
    'RUB',
    'MOEX'
);
```

---

### 2.3. Функции анализа данных

**Примечание:** Функции для анализа паттернов свечей (`analyze_candle_patterns`, `get_candle_pattern_stats`) описаны в отдельном документе: [db-strategy-analysis-objects.md](./db-strategy-analysis-objects.md)

#### 2.3.1. `invest.get_backtest_stats(p_analysis_date_from DATE, p_analysis_date_to DATE, p_figi VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** RECORD

**Описание:** Получает статистику результатов бэктестинга за указанный период.

**Параметры:**
- `p_analysis_date_from` (DATE) - Начальная дата периода
- `p_analysis_date_to` (DATE) - Конечная дата периода
- `p_figi` (VARCHAR) - FIGI инструмента (опционально)

**Возвращает:** Статистику по результатам бэктестинга

---

#### 2.3.2. `invest.get_session_analytics(p_figi VARCHAR, p_start_date DATE, p_end_date DATE)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** RECORD

**Описание:** Получает аналитику по торговым сессиям за указанный период.

**Параметры:**
- `p_figi` (VARCHAR) - FIGI инструмента (опционально, NULL = все инструменты)
- `p_start_date` (DATE) - Начальная дата (опционально)
- `p_end_date` (DATE) - Конечная дата (опционально)

**Возвращает:** Данные об объемах по сессиям (утренняя, основная, вечерняя, выходные)

**Пример использования:**
```sql
SELECT * FROM invest.get_session_analytics('BBG004S68DD6', '2025-01-01', '2025-01-31');
```

---

#### 2.3.3. `invest.get_today_session_analytics(p_figi VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** RECORD

**Описание:** Получает аналитику по торговым сессиям за сегодняшний день.

**Параметры:**
- `p_figi` (VARCHAR) - FIGI инструмента (опционально, NULL = все инструменты)

**Возвращает:** Данные об объемах за сегодня по сессиям

---

### 2.4. Функции работы с MT5 данными

#### 2.4.1. `invest.fill_session_prices_for_date(p_date DATE)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** VOID

**Описание:** Заполняет цены открытия/закрытия для всех символов за указанную дату на основе тиков MT5.

**Параметры:**
- `p_date` (DATE) - Дата в московском времени

**Логика:**
- Определяет временные метки сессий в UTC:
  - Premarket open: 11:00 MSK (08:00 UTC)
  - Main open: 16:30 MSK (13:30 UTC)
  - Main close: 23:00 MSK (20:00 UTC)
  - Postmarket close: 03:00 MSK следующего дня (00:00 UTC)
- Для каждого символа с тиками за день:
  - Находит последний тик до начала каждой сессии
  - Вычисляет среднюю цену (bid + ask) / 2
  - Вставляет/обновляет цены в соответствующие таблицы

**Таблицы, которые заполняются:**
- `mt5_premarket_open_price`
- `mt5_open_price`
- `mt5_close_price`
- `mt5_postmarket_close_price`

**Пример использования:**
```sql
SELECT invest.fill_session_prices_for_date('2025-01-15');
```

---

#### 2.4.2. `invest.fill_session_prices_for_symbol(p_symbol VARCHAR, p_date DATE)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** VOID

**Описание:** Аналогична `fill_session_prices_for_date`, но для одного символа.

**Параметры:**
- `p_symbol` (VARCHAR) - Символ инструмента
- `p_date` (DATE) - Дата в московском времени

---

#### 2.4.3. `invest.bt_get_reference_price(p_symbol VARCHAR, p_date_msk DATE, p_type VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** NUMERIC

**Описание:** Получает опорную цену для бэктестинга стратегии mean reversion.

**Параметры:**
- `p_symbol` (VARCHAR) - Символ инструмента
- `p_date_msk` (DATE) - Дата в московском времени
- `p_type` (VARCHAR) - Тип цены: 'postmarket', 'close_price', 'premarket', 'open_price'

**Логика:**
- Определяет временную метку в зависимости от типа:
  - `postmarket`: 03:00 MSK следующего дня
  - `close_price`: 23:00 MSK предыдущего дня
  - `premarket`: 11:00 MSK текущего дня
  - `open_price`: 16:30 MSK текущего дня
- Получает цену из соответствующей таблицы MT5
- Если цена не найдена, ищет последний тик до указанного времени
- Вычисляет среднюю цену (bid + ask) / 2
- Округляет до 2 знаков после запятой

**Возвращает:** Опорную цену или исключение, если цена не найдена

---

#### 2.4.4. `invest.bt_run_mean_reversion(p_symbol VARCHAR, p_date_text VARCHAR, p_type VARCHAR, p_direction VARCHAR, p_amount NUMERIC, p_level1 NUMERIC, p_level2 NUMERIC, p_level3 NUMERIC)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** UUID

**Описание:** Запускает бэктестинг стратегии mean reversion (возврат к среднему).

**Параметры:**
- `p_symbol` (VARCHAR) - Символ инструмента
- `p_date_text` (VARCHAR) - Дата в формате 'DD.MM.YYYY'
- `p_type` (VARCHAR) - Тип опорной цены
- `p_direction` (VARCHAR) - Направление торговли: 'buy', 'sell', 'both'
- `p_amount` (NUMERIC) - Общая сумма для торговли
- `p_level1`, `p_level2`, `p_level3` (NUMERIC) - Процентные уровни отклонения от опорной цены

**Логика:**
- Генерирует уникальный `run_id` (UUID)
- Определяет временные рамки торговой сессии:
  - Начало: 03:00 MSK
  - Дедлайн входа: 16:25 MSK
  - Конец: 16:35 MSK
- Получает опорную цену через `bt_get_reference_price`
- Создает временную таблицу уровней торговли
- Для каждого уровня вычисляет:
  - `buy_limit` = опорная_цена * (1 - процент/100)
  - `sell_limit` = опорная_цена * (1 + процент/100)
  - `qty` = amount / опорная_цена
- Проходит по всем тикам за сессию:
  - До 16:25 MSK: проверяет достижение уровней для входа
  - После входа: проверяет возврат к опорной цене для выхода
- Сохраняет результаты в таблицу `bt_mean_reversion_trades`
- Принудительно закрывает открытые позиции в 16:35 MSK

**Возвращает:** UUID запуска бэктестинга

**Пример использования:**
```sql
SELECT invest.bt_run_mean_reversion(
    'SBER',
    '15.01.2025',
    'close_price',
    'both',
    100000,
    0.5,  -- 0.5% отклонение
    1.0,  -- 1.0% отклонение
    1.5   -- 1.5% отклонение
);
```

---

### 2.5. Функции работы с данными

#### 2.5.1. `invest.check_evening_session_close_price(p_date_str VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** RECORD

**Описание:** Проверяет цены закрытия вечерней сессии за указанную дату.

**Параметры:**
- `p_date_str` (VARCHAR) - Дата в строковом формате

**Логика:** Вызывает функцию из схемы `invest_prices`

---

#### 2.5.2. `invest.add_special_trading_hours(p_figi VARCHAR, p_instrument_type VARCHAR, p_day_type VARCHAR, p_start_hour INTEGER, p_start_minute INTEGER, p_end_hour INTEGER, p_end_minute INTEGER, p_description VARCHAR, p_created_by VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** BIGINT

**Описание:** Добавляет запись о специальных торговых часах для инструмента.

**Параметры:**
- `p_figi` (VARCHAR) - FIGI инструмента
- `p_instrument_type` (VARCHAR) - Тип инструмента
- `p_day_type` (VARCHAR) - Тип дня
- `p_start_hour`, `p_start_minute` (INTEGER) - Время начала
- `p_end_hour`, `p_end_minute` (INTEGER) - Время окончания
- `p_description` (VARCHAR) - Описание
- `p_created_by` (VARCHAR) - Создатель записи

**Возвращает:** ID созданной записи

---

#### 2.5.3. `invest.execute_saved_script(script_name_param VARCHAR)`
**Схема:** `invest`  
**Тип:** FUNCTION  
**Возвращает:** TEXT

**Описание:** Выполняет сохраненный SQL-скрипт из таблицы `sql_scripts`.

**Параметры:**
- `script_name_param` (VARCHAR) - Имя скрипта

**Логика:**
- Получает содержимое скрипта из таблицы `sql_scripts`
- Проверяет, что скрипт начинается с `SELECT` (только для безопасности)
- Выполняет скрипт и возвращает результат

**Ограничения:** Выполняются только SELECT-запросы

---

## 3. Триггеры

### 3.1. Триггеры вычисления статистики свечей

#### 3.1.1. `calculate_daily_candle_statistics()`
**Схема:** `invest_candles`  
**Таблица:** `daily_candles`  
**Событие:** BEFORE INSERT OR UPDATE  
**Тип:** TRIGGER FUNCTION

**Описание:** Автоматически вычисляет статистические показатели дневной свечи при вставке или обновлении.

**Вычисляемые поля:**
- `price_change` = `close` - `open`
- `price_change_percent` = (`price_change` / `open`) * 100 (если `open` > 0)
- `candle_type`:
  - `'BULLISH'` если `close` > `open`
  - `'BEARISH'` если `close` < `open`
  - `'DOJI'` если `close` = `open`
- `body_size` = ABS(`price_change`)
- `upper_shadow` = `high` - GREATEST(`open`, `close`)
- `lower_shadow` = LEAST(`open`, `close`) - `low`
- `high_low_range` = `high` - `low`
- `average_price` = (`high` + `low` + `close`) / 3

**Применение:** Автоматически при каждой вставке или обновлении записи в `daily_candles`

---

#### 3.1.2. `calculate_minute_candle_statistics()`
**Схема:** `invest_candles`  
**Таблица:** `minute_candles`  
**Событие:** BEFORE INSERT OR UPDATE  
**Тип:** TRIGGER FUNCTION

**Описание:** Автоматически вычисляет статистические показатели минутной свечи при вставке или обновлении.

**Вычисляемые поля:** Аналогичны `calculate_daily_candle_statistics`, но:
- `average_price` = (`high` + `low` + `open` + `close`) / 4 (включает все 4 цены)

**Применение:** Автоматически при каждой вставке или обновлении записи в `minute_candles`

---

### 3.2. Триггеры вычисления длительности

#### 3.2.1. `calculate_duration_ms()`
**Схема:** `invest_utils`  
**Таблица:** `system_logs`  
**Событие:** BEFORE INSERT OR UPDATE  
**Тип:** TRIGGER FUNCTION

**Описание:** Вычисляет длительность выполнения операции в миллисекундах.

**Логика:**
- Если `end_time` и `start_time` не NULL
- Вычисляет: `duration_ms` = (`end_time` - `start_time`) * 1000

**Применение:** Автоматически при вставке или обновлении записи в `system_logs`

---

## 4. Материализованные представления

### 4.1. `daily_volume_aggregation`
**Схема:** `invest_views` (доступ через `invest`)  
**Тип:** MATERIALIZED VIEW

**Описание:** Материализованное представление с агрегированными данными об объемах торгов по дням и сессиям.

**Источник данных:** `minute_candles`, `shares`, `futures`

**Атрибуты:**

| Атрибут | Тип | Описание |
|---------|-----|----------|
| `figi` | VARCHAR(255) | FIGI инструмента |
| `instrument_type` | TEXT | Тип инструмента (share/future/unknown) |
| `trade_date` | DATE | Дата торгов |
| `total_volume` | BIGINT | Общий объем за день |
| `total_candles` | BIGINT | Общее количество свечей |
| `avg_volume_per_candle` | NUMERIC | Средний объем на свечу |
| `morning_session_volume` | BIGINT | Объем утренней сессии |
| `morning_session_candles` | INTEGER | Количество свечей утренней сессии |
| `morning_avg_volume_per_candle` | NUMERIC | Средний объем на свечу утренней сессии |
| `main_session_volume` | BIGINT | Объем основной сессии |
| `main_session_candles` | INTEGER | Количество свечей основной сессии |
| `main_avg_volume_per_candle` | NUMERIC | Средний объем на свечу основной сессии |
| `evening_session_volume` | BIGINT | Объем вечерней сессии |
| `evening_session_candles` | INTEGER | Количество свечей вечерней сессии |
| `evening_avg_volume_per_candle` | NUMERIC | Средний объем на свечу вечерней сессии |
| `weekend_exchange_session_volume` | BIGINT | Объем выходной биржевой сессии |
| `weekend_exchange_session_candles` | INTEGER | Количество свечей выходной биржевой сессии |
| `weekend_exchange_avg_volume_per_candle` | NUMERIC | Средний объем на свечу выходной биржевой сессии |
| `weekend_otc_session_volume` | BIGINT | Объем выходной OTC сессии |
| `weekend_otc_session_candles` | INTEGER | Количество свечей выходной OTC сессии |
| `weekend_otc_avg_volume_per_candle` | NUMERIC | Средний объем на свечу выходной OTC сессии |
| `first_candle_time` | TIMESTAMP WITH TIME ZONE | Время первой свечи |
| `last_candle_time` | TIMESTAMP WITH TIME ZONE | Время последней свечи |
| `last_updated` | TIMESTAMP WITH TIME ZONE | Время последнего обновления |

**Определение сессий (московское время):**
- **Утренняя сессия:** Пн-Пт, 06:59-09:59
- **Основная сессия:** Пн-Пт, 10:00-18:59
- **Вечерняя сессия:** Пн-Пт, 19:00-23:50
- **Выходная биржевая:** Сб-Вс, 10:00-18:59
- **Выходная OTC:** Сб-Вс, 02:00-09:59 и 19:00-23:50

**Обновление:** Требует ручного обновления через `REFRESH MATERIALIZED VIEW`

**Пример использования:**
```sql
REFRESH MATERIALIZED VIEW invest.daily_volume_aggregation;
SELECT * FROM invest.daily_volume_aggregation WHERE figi = 'BBG004S68DD6';
```

---

### 4.2. `history_volume_aggregation`
**Схема:** `invest_views` (доступ через `invest`)  
**Тип:** MATERIALIZED VIEW

**Описание:** Материализованное представление с исторической агрегацией объемов торгов за весь период.

**Источник данных:** `minute_candles`, `shares`, `futures`

**Атрибуты:** Аналогичны `daily_volume_aggregation`, но дополнительно включает:
- `total_days` - Общее количество дней
- `working_days` - Количество рабочих дней
- `weekend_days` - Количество выходных дней
- Средние объемы на день по сессиям

**Обновление:** Требует ручного обновления

---

### 4.3. `historical_price_extremes`
**Схема:** `invest_views` (доступ через `invest`)  
**Тип:** MATERIALIZED VIEW

**Описание:** Материализованное представление с историческими экстремумами цен инструментов.

**Источник данных:** `daily_candles`

**Атрибуты:**

| Атрибут | Тип | Описание |
|---------|-----|----------|
| `figi` | VARCHAR(255) | FIGI инструмента |
| `ticker` | VARCHAR(255) | Тикер инструмента |
| `instrument_type` | TEXT | Тип инструмента |
| `historical_high` | NUMERIC | Исторический максимум цены |
| `historical_high_date` | TIMESTAMP WITH TIME ZONE | Дата исторического максимума |
| `historical_low` | NUMERIC | Исторический минимум цены |
| `historical_low_date` | TIMESTAMP WITH TIME ZONE | Дата исторического минимума |

**Логика вычисления:**
- `historical_high` = MAX(`high` + `upper_shadow`)
- `historical_low` = MIN(`low` - `lower_shadow`)

**Обновление:** Требует ручного обновления

---

## 5. Обычные представления (Views)

### 5.1. Представления-синонимы (из других схем)

Все представления в схеме `invest` являются синонимами таблиц из других схем для удобства доступа:

#### 5.1.1. `invest.shares`
**Источник:** `invest_ref.shares`  
**Описание:** Синоним для таблицы акций

#### 5.1.2. `invest.futures`
**Источник:** `invest_ref.futures`  
**Описание:** Синоним для таблицы фьючерсов

#### 5.1.3. `invest.indicatives`
**Источник:** `invest_ref.indicatives`  
**Описание:** Синоним для таблицы индикативов

#### 5.1.4. `invest.dividends`
**Источник:** `invest_ref.dividends`  
**Описание:** Синоним для таблицы дивидендов

#### 5.1.5. `invest.fundamentals`
**Источник:** `invest_ref.fundamentals`  
**Описание:** Синоним для таблицы фундаментальных показателей

#### 5.1.6. `invest.close_prices`
**Источник:** `invest_prices.close_prices`  
**Описание:** Синоним для таблицы цен закрытия

#### 5.1.7. `invest.open_prices`
**Источник:** `invest_prices.open_prices`  
**Описание:** Синоним для таблицы цен открытия

#### 5.1.8. `invest.last_prices`
**Источник:** `invest_prices.last_prices`  
**Описание:** Синоним для таблицы последних цен

#### 5.1.9. `invest.daily_candles`
**Источник:** `invest_candles.daily_candles`  
**Описание:** Синоним для таблицы дневных свечей

#### 5.1.10. `invest.minute_candles`
**Источник:** `invest_candles.minute_candles`  
**Описание:** Синоним для таблицы минутных свечей

#### 5.1.11. `invest.trades`
**Источник:** `invest_prices.trades`  
**Описание:** Синоним для таблицы сделок

#### 5.1.12. `invest.system_logs`
**Источник:** `invest_utils.system_logs`  
**Описание:** Синоним для таблицы системных логов

---

### 5.2. Представления с обогащением данных

**Примечание:** Представление `invest.candle_pattern_analysis` описано в отдельном документе: [db-strategy-analysis-objects.md](./db-strategy-analysis-objects.md)

#### 5.2.1. `invest.backtest_results`
**Источник:** `invest_candles.backtest_results` + справочники  
**Описание:** Представление результатов бэктестинга с добавлением информации об инструментах.

**Дополнительные поля:**
- `instrument_name` - Название инструмента (из shares/futures/indicatives)
- `instrument_ticker` - Тикер инструмента
- `instrument_type` - Тип инструмента (shares/futures/indicatives)

---

#### 5.2.2. `invest.special_trading_hours`
**Источник:** `invest_candles.special_trading_hours` + справочники  
**Описание:** Представление специальных торговых часов с информацией об инструментах.

**Дополнительные поля:**
- `instrument_name` - Название инструмента
- `instrument_ticker` - Тикер инструмента

---

#### 5.2.3. `invest.historical_price_extremes`
**Источник:** `invest_views.historical_price_extremes`  
**Описание:** Синоним для материализованного представления исторических экстремумов

---

#### 5.2.4. `invest.history_volume_aggregation`
**Источник:** `invest_views.history_volume_aggregation`  
**Описание:** Синоним для материализованного представления исторической агрегации объемов

---

#### 5.2.5. `invest.today_volume_view`
**Источник:** `invest_views.today_volume_view`  
**Описание:** Представление с агрегированными объемами за сегодняшний день.

**Особенности:**
- Вычисляется динамически при каждом запросе
- Фильтрует данные по текущей дате (московское время)
- Группирует по инструментам и сессиям

**Пример использования:**
```sql
SELECT * FROM invest.today_volume_view WHERE figi = 'BBG004S68DD6';
```

---

#### 5.2.6. `invest.close_prices_evening_session`
**Источник:** `invest_prices.close_prices_evening_session`  
**Описание:** Синоним для таблицы цен закрытия вечерней сессии

---

#### 5.2.7. `invest.data_quality_issues`
**Источник:** `invest_utils.data_quality_issues`  
**Описание:** Синоним для таблицы проблем качества данных

---

### 5.3. Служебные представления

#### 5.3.1. `invest_views.last_prices_daily_partition_stats`
**Схема:** `invest_views`  
**Описание:** Статистика по дневным партициям таблицы `last_prices`.

**Атрибуты:**
- `schemaname` - Имя схемы
- `partition_name` - Имя партиции
- `partition_date` - Дата партиции (извлеченная из имени)
- `size` - Размер партиции (человекочитаемый формат)
- `size_bytes` - Размер партиции в байтах

**Логика:** Извлекает информацию из системных таблиц PostgreSQL о партициях

**Пример использования:**
```sql
SELECT * FROM invest_views.last_prices_daily_partition_stats 
ORDER BY partition_date DESC;
```

---

## 6. Рекомендации по использованию

### 6.1. Функции партиционирования

1. **Автоматическое создание партиций:**
   - Вызывать функции создания партиций заранее (например, на месяц вперед)
   - Использовать планировщик задач (cron, pg_cron) для автоматизации

2. **Очистка старых партиций:**
   - Регулярно вызывать `cleanup_old_daily_partitions` для освобождения места
   - Настроить политику хранения данных

### 6.2. Материализованные представления

1. **Обновление:**
   - Обновлять регулярно через `REFRESH MATERIALIZED VIEW`
   - Использовать `REFRESH MATERIALIZED VIEW CONCURRENTLY` для обновления без блокировки (требует уникального индекса)

2. **Производительность:**
   - Материализованные представления ускоряют запросы, но требуют места на диске
   - Балансировать между частотой обновления и актуальностью данных

### 6.3. Триггеры

1. **Производительность:**
   - Триггеры выполняются автоматически, но могут замедлять массовые операции
   - Для массовых вставок рассмотреть временное отключение триггеров

2. **Отладка:**
   - Использовать логирование в триггерах для отладки (RAISE NOTICE)

### 6.4. Функции бэктестинга

1. **Использование:**
   - Функции бэктестинга требуют наличия данных MT5
   - Убедиться, что данные за нужный период загружены

2. **Производительность:**
   - Бэктестинг может быть ресурсоемким
   - Выполнять в нерабочее время или на отдельном сервере

---

## 7. Примечания

1. **Отсутствие явных внешних ключей:** Многие функции работают с логическими связями через FIGI, а не через FOREIGN KEY constraints.

2. **Временные зоны:** Все функции работают с московским временем (`Europe/Moscow`), что важно учитывать при вызовах.

3. **Партиционирование:** Функции создания партиций должны вызываться заранее, до начала загрузки данных в соответствующий период.

4. **Безопасность:** Функция `execute_saved_script` ограничена только SELECT-запросами для безопасности.

5. **Материализованные представления:** Требуют ручного обновления, что может быть автоматизировано через планировщик задач.

---

## 8. Примеры использования

### 8.1. Создание партиций на месяц вперед

```sql
-- Создать партиции для минутных свечей на январь 2025
DO $$
DECLARE
    d DATE;
BEGIN
    FOR d IN SELECT generate_series('2025-01-01'::date, '2025-01-31'::date, '1 day'::interval)::date
    LOOP
        PERFORM invest_utils.create_minute_candles_partition(d);
    END LOOP;
END $$;
```

### 8.2. Обновление материализованных представлений

```sql
-- Обновить все материализованные представления
REFRESH MATERIALIZED VIEW invest.daily_volume_aggregation;
REFRESH MATERIALIZED VIEW invest.history_volume_aggregation;
REFRESH MATERIALIZED VIEW invest.historical_price_extremes;
```

### 8.3. Анализ паттернов и получение статистики

**Примечание:** Примеры использования функций анализа паттернов свечей см. в [db-strategy-analysis-objects.md](./db-strategy-analysis-objects.md)

```sql
-- Получить статистику бэктестинга
SELECT * FROM invest.get_backtest_stats(
    '2025-01-01'::date,
    '2025-01-31'::date,
    'BBG004S68DD6'
);
```

### 8.4. Заполнение цен сессий MT5

```sql
-- Заполнить цены для всех символов за дату
SELECT invest.fill_session_prices_for_date('2025-01-15');

-- Заполнить цены для конкретного символа
SELECT invest.fill_session_prices_for_symbol('SBER', '2025-01-15');
```

---
