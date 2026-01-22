# Логическая модель базы данных InvestmentDataLoaderService

## 1. Обзор

Данный документ описывает логическую модель базы данных на уровне сущностей, атрибутов, связей, ключей и ограничений целостности.

**Схемы базы данных:**
- `invest_ref` - справочники
- `invest_prices` - ценовые данные
- `invest_candles` - свечные данные
- `invest_utils` - служебные таблицы
- `invest` - основная схема с представлениями (views)

---

## 2. Сущности (Таблицы) и их атрибуты

### 2.1. Справочные сущности

#### 2.1.1. SHARES (Акции)
**Схема:** `invest_ref` (доступ через view `invest.shares`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | Уникальный идентификатор инструмента (FIGI) |
| ticker | VARCHAR(255) | NULL | Тикер акции |
| name | VARCHAR(255) | NULL | Название компании |
| currency | VARCHAR(255) | NULL | Валюта торговли |
| exchange | VARCHAR(255) | NULL | Биржа торговли |
| sector | VARCHAR(255) | NULL | Сектор экономики |
| trading_status | VARCHAR(255) | NULL | Статус торговли |
| short_enabled | BOOLEAN | NULL | Флаг возможности коротких продаж |
| asset_uid | VARCHAR(255) | NULL | Уникальный идентификатор актива |
| min_price_increment | NUMERIC | NULL | Минимальный шаг цены |
| lot | INTEGER | NULL | Лотность инструмента |
| created_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата создания записи |
| updated_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата обновления записи |

**Первичный ключ:** `figi`

**Индексы:**
- PRIMARY KEY на `figi`

---

#### 2.1.2. FUTURES (Фьючерсы)
**Схема:** `invest_ref` (доступ через view `invest.futures`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | Уникальный идентификатор инструмента |
| ticker | VARCHAR(255) | NULL | Тикер фьючерса |
| asset_type | VARCHAR(255) | NULL | Тип базового актива |
| basic_asset | VARCHAR(255) | NULL | Базовый актив фьючерса |
| currency | VARCHAR(255) | NULL | Валюта инструмента |
| exchange | VARCHAR(255) | NULL | Биржа торговли |
| short_enabled | BOOLEAN | NULL | Флаг возможности коротких продаж |
| expiration_date | TIMESTAMP WITHOUT TIME ZONE | NULL | Дата экспирации |
| min_price_increment | NUMERIC | NULL | Минимальный шаг цены |
| lot | INTEGER | NULL | Лотность инструмента |
| basic_asset_size | NUMERIC(18,9) | NULL | Размер базового актива |
| created_at | TIMESTAMP(6) | NOT NULL | Дата создания записи |
| updated_at | TIMESTAMP(6) | NOT NULL | Дата обновления записи |

**Первичный ключ:** `figi`

---

#### 2.1.3. INDICATIVES (Индикативы)
**Схема:** `invest_ref` (доступ через view `invest.indicatives`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | Уникальный идентификатор инструмента |
| ticker | VARCHAR(255) | NOT NULL | Тикер инструмента |
| name | VARCHAR(255) | NOT NULL | Название инструмента |
| currency | VARCHAR(255) | NOT NULL | Валюта инструмента |
| exchange | VARCHAR(255) | NOT NULL | Биржа/площадка |
| class_code | VARCHAR(255) | NULL | Код класса инструмента |
| uid | VARCHAR(255) | NULL | Уникальный идентификатор |
| sell_available_flag | BOOLEAN | DEFAULT FALSE | Флаг доступности для продажи |
| buy_available_flag | BOOLEAN | DEFAULT FALSE | Флаг доступности для покупки |
| created_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата создания записи |
| updated_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата обновления записи |

**Первичный ключ:** `figi`

---

#### 2.1.4. DIVIDENDS (Дивиденды)
**Схема:** `invest_ref` (доступ через view `invest.dividends`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **id** | BIGSERIAL | PRIMARY KEY | Уникальный идентификатор записи |
| figi | VARCHAR(255) | NOT NULL | FIGI инструмента |
| declared_date | DATE | NULL | Дата объявления дивидендов |
| record_date | DATE | NOT NULL | Дата фиксации реестра |
| payment_date | DATE | NULL | Дата выплаты дивидендов |
| dividend_value | NUMERIC(18,9) | NULL | Размер дивиденда на акцию |
| currency | VARCHAR(10) | NOT NULL | Валюта дивиденда |
| dividend_type | VARCHAR(50) | NULL | Тип дивиденда |
| created_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата создания записи |
| updated_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата обновления записи |

**Первичный ключ:** `id`

**Индексы:**
- PRIMARY KEY на `id`
- Рекомендуется индекс на `figi` для связи с инструментами

---

#### 2.1.5. FUNDAMENTALS (Фундаментальные показатели)
**Схема:** `invest_ref` (доступ через view `invest.fundamentals`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **id** | BIGSERIAL | PRIMARY KEY | Уникальный идентификатор записи |
| figi | VARCHAR(255) | NOT NULL | FIGI инструмента |
| asset_uid | VARCHAR(255) | NULL | Уникальный идентификатор актива |
| domicile_indicator_code | VARCHAR(10) | NULL | Код страны регистрации |
| currency | VARCHAR(10) | NULL | Валюта показателей |

**Дивидендные показатели:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| dividend_yield_daily_ttm | DECIMAL(18,9) | Дневная дивидендная доходность за TTM |
| dividend_rate_ttm | DECIMAL(18,9) | Ставка дивиденда за TTM |
| dividend_payout_ratio_fy | DECIMAL(18,9) | Коэффициент выплаты дивидендов за FY |
| forward_annual_dividend_yield | DECIMAL(18,9) | Прогнозируемая годовая дивидендная доходность |
| five_years_average_dividend_yield | DECIMAL(18,9) | Средняя дивидендная доходность за 5 лет |
| dividends_per_share | DECIMAL(18,9) | Дивиденды на акцию |
| five_year_annual_dividend_growth_rate | DECIMAL(18,9) | Годовой темп роста дивидендов за 5 лет |

**Показатели оценки:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| price_to_sales_ttm | DECIMAL(18,9) | P/S за TTM |
| price_to_book_ttm | DECIMAL(18,9) | P/B за TTM |
| price_to_free_cash_flow_ttm | DECIMAL(18,9) | P/FCF за TTM |
| pe_ratio_ttm | DECIMAL(18,9) | P/E за TTM |
| ev_to_sales | DECIMAL(18,9) | EV/Sales |
| ev_to_ebitda_mrq | DECIMAL(18,9) | EV/EBITDA на MRQ |

**Показатели прибыльности:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| eps_ttm | DECIMAL(18,9) | EPS за TTM |
| diluted_eps_ttm | DECIMAL(18,9) | Разводненная EPS за TTM |
| net_income_ttm | DECIMAL(18,9) | Чистая прибыль за TTM |
| ebitda_ttm | DECIMAL(18,9) | EBITDA за TTM |
| free_cash_flow_ttm | DECIMAL(18,9) | Свободный денежный поток за TTM |
| revenue_ttm | DECIMAL(18,9) | Выручка за TTM |
| net_margin_mrq | DECIMAL(18,9) | Чистая маржа на MRQ |

**Показатели рентабельности:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| roe | DECIMAL(18,9) | ROE |
| roa | DECIMAL(18,9) | ROA |
| roic | DECIMAL(18,9) | ROIC |

**Показатели роста:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| revenue_change_five_years | DECIMAL(18,9) | Изменение выручки за 5 лет |
| five_year_annual_revenue_growth_rate | DECIMAL(18,9) | Годовой темп роста выручки за 5 лет |
| one_year_annual_revenue_growth_rate | DECIMAL(18,9) | Годовой темп роста выручки за 1 год |
| three_year_annual_revenue_growth_rate | DECIMAL(18,9) | Годовой темп роста выручки за 3 года |
| eps_change_five_years | DECIMAL(18,9) | Изменение EPS за 5 лет |
| ebitda_change_five_years | DECIMAL(18,9) | Изменение EBITDA за 5 лет |

**Показатели долга:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| total_debt_mrq | DECIMAL(18,9) | Общий долг на MRQ |
| total_debt_to_equity_mrq | DECIMAL(18,9) | Отношение долга к капиталу на MRQ |
| total_debt_to_ebitda_mrq | DECIMAL(18,9) | Отношение долга к EBITDA на MRQ |
| net_debt_to_ebitda | DECIMAL(18,9) | Отношение чистого долга к EBITDA |
| total_debt_change_five_years | DECIMAL(18,9) | Изменение долга за 5 лет |

**Показатели ликвидности:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| current_ratio_mrq | DECIMAL(18,9) | Коэффициент текущей ликвидности на MRQ |
| fixed_charge_coverage_ratio_fy | DECIMAL(18,9) | Покрытие фиксированных платежей за FY |
| net_interest_margin_mrq | DECIMAL(18,9) | Чистая процентная маржа на MRQ |

**Рыночные показатели:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| market_capitalization | DECIMAL(18,9) | Рыночная капитализация |
| total_enterprise_value_mrq | DECIMAL(18,9) | Стоимость предприятия на MRQ |
| shares_outstanding | DECIMAL(18,9) | Количество акций в обращении |
| free_float | DECIMAL(18,9) | Свободно обращающиеся акции |
| beta | DECIMAL(18,9) | Бета-коэффициент |

**Ценовые показатели:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| high_price_last_52_weeks | DECIMAL(18,9) | Максимальная цена за 52 недели |
| low_price_last_52_weeks | DECIMAL(18,9) | Минимальная цена за 52 недели |

**Объемы торгов:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| average_daily_volume_last_4_weeks | DECIMAL(18,9) | Средний дневной объем за 4 недели |
| average_daily_volume_last_10_days | DECIMAL(18,9) | Средний дневной объем за 10 дней |

**Показатели компании:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| number_of_employees | DECIMAL(18,9) | Количество сотрудников |
| adr_to_common_share_ratio | DECIMAL(18,9) | Соотношение ADR к обыкновенным акциям |
| buy_back_ttm | DECIMAL(18,9) | Выкуп акций за TTM |
| free_cash_flow_to_price | DECIMAL(18,9) | Отношение FCF к цене |

**Даты:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| fiscal_period_start_date | TIMESTAMP WITH TIME ZONE | Дата начала финансового периода |
| fiscal_period_end_date | TIMESTAMP WITH TIME ZONE | Дата окончания финансового периода |
| ex_dividend_date | TIMESTAMP WITH TIME ZONE | Дата отсечки дивидендов |

**Служебные поля:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| created_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата создания |
| updated_at | TIMESTAMP WITH TIME ZONE | DEFAULT (CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow') | Дата обновления |

**Первичный ключ:** `id`

**Индексы:**
- PRIMARY KEY на `id`
- INDEX на `figi` (idx_fundamentals_figi)

---

### 2.2. Ценовые сущности

#### 2.2.1. CLOSE_PRICES (Цены закрытия)
**Схема:** `invest_prices` (доступ через view `invest.close_prices`)

**Партиционирование:** По месяцам (RANGE по `price_date`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | FIGI инструмента |
| **price_date** | DATE | PRIMARY KEY, NOT NULL | Дата торгов |
| close_price | NUMERIC(18,9) | NOT NULL | Цена закрытия |
| currency | VARCHAR(255) | NOT NULL | Валюта |
| exchange | VARCHAR(255) | NOT NULL | Биржа |
| instrument_type | VARCHAR(255) | NOT NULL | Тип инструмента |
| created_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата создания |
| updated_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата обновления |

**Первичный ключ:** `(figi, price_date)` - составной

**Индексы:**
- PRIMARY KEY на `(figi, price_date)`
- INDEX на `price_date` (idx_close_prices_date)
- INDEX на `(figi, price_date)` (idx_close_prices_figi_date)

---

#### 2.2.2. OPEN_PRICES (Цены открытия)
**Схема:** `invest_prices` (доступ через view `invest.open_prices`)

**Партиционирование:** По месяцам (RANGE по `price_date`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | FIGI инструмента |
| **price_date** | DATE | PRIMARY KEY, NOT NULL | Дата торгов |
| open_price | NUMERIC(18,9) | NOT NULL | Цена открытия |
| currency | VARCHAR(255) | NOT NULL | Валюта |
| exchange | VARCHAR(255) | NOT NULL | Биржа |
| instrument_type | VARCHAR(255) | NOT NULL | Тип инструмента |
| created_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата создания |
| updated_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата обновления |

**Первичный ключ:** `(figi, price_date)` - составной

---

#### 2.2.3. LAST_PRICES (Последние цены)
**Схема:** `invest` (партиционированная таблица)

**Партиционирование:** По дням (RANGE по `time`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | FIGI инструмента |
| **time** | TIMESTAMP WITHOUT TIME ZONE | PRIMARY KEY, NOT NULL | Время сделки |
| price | NUMERIC(18,9) | NOT NULL | Последняя цена |
| currency | VARCHAR(255) | NOT NULL | Валюта |
| exchange | VARCHAR(255) | NOT NULL | Биржа |

**Первичный ключ:** `(figi, time)` - составной

---

### 2.3. Свечные сущности

#### 2.3.1. DAILY_CANDLES (Дневные свечи)
**Схема:** `invest_candles` (доступ через view `invest.daily_candles`)

**Партиционирование:** По месяцам (RANGE по `time`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | FIGI инструмента |
| **time** | TIMESTAMP(6) WITH TIME ZONE | PRIMARY KEY, NOT NULL | Время начала дня |
| open | NUMERIC(18,9) | NOT NULL | Цена открытия |
| high | NUMERIC(18,9) | NOT NULL | Максимальная цена |
| low | NUMERIC(18,9) | NOT NULL | Минимальная цена |
| close | NUMERIC(18,9) | NOT NULL | Цена закрытия |
| volume | BIGINT | NOT NULL | Объем торгов |
| is_complete | BOOLEAN | NOT NULL | Флаг завершенности свечи |

**Вычисляемые атрибуты (триггеры):**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| price_change | NUMERIC(18,9) | Изменение цены (close - open) |
| price_change_percent | NUMERIC(18,4) | Процентное изменение цены |
| candle_type | VARCHAR(20) | Тип свечи: BULLISH, BEARISH, DOJI |
| body_size | NUMERIC(18,9) | Размер тела свечи |
| upper_shadow | NUMERIC(18,9) | Верхняя тень |
| lower_shadow | NUMERIC(18,9) | Нижняя тень |
| high_low_range | NUMERIC(18,9) | Диапазон цен (high - low) |
| average_price | NUMERIC(18,2) | Средняя цена (high + low + close) / 3 |

**Служебные поля:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| created_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата создания |
| updated_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата обновления |

**Первичный ключ:** `(figi, time)` - составной

**Триггеры:**
- `calculate_daily_candle_statistics()` - вычисляет статистику свечи при INSERT/UPDATE

---

#### 2.3.2. MINUTE_CANDLES (Минутные свечи)
**Схема:** `invest_candles` (доступ через view `invest.minute_candles`)

**Партиционирование:** По дням (RANGE по `time`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | FIGI инструмента |
| **time** | TIMESTAMP(6) WITH TIME ZONE | PRIMARY KEY, NOT NULL | Время начала минуты |
| open | NUMERIC(18,9) | NOT NULL | Цена открытия |
| high | NUMERIC(18,9) | NOT NULL | Максимальная цена |
| low | NUMERIC(18,9) | NOT NULL | Минимальная цена |
| close | NUMERIC(18,9) | NOT NULL | Цена закрытия |
| volume | BIGINT | NOT NULL | Объем торгов |
| is_complete | BOOLEAN | NOT NULL | Флаг завершенности свечи |

**Вычисляемые атрибуты (триггеры):**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| price_change | NUMERIC(18,9) | Изменение цены (close - open) |
| price_change_percent | NUMERIC(18,4) | Процентное изменение цены |
| candle_type | VARCHAR(20) | Тип свечи: BULLISH, BEARISH, DOJI |
| body_size | NUMERIC(18,9) | Размер тела свечи |
| upper_shadow | NUMERIC(18,9) | Верхняя тень |
| lower_shadow | NUMERIC(18,9) | Нижняя тень |
| high_low_range | NUMERIC(18,9) | Диапазон цен (high - low) |
| average_price | NUMERIC(18,2) | Средняя цена (high + low + open + close) / 4 |

**Служебные поля:**
| Атрибут | Тип данных | Описание |
|---------|------------|----------|
| created_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата создания |
| updated_at | TIMESTAMP(6) WITH TIME ZONE | DEFAULT NOW(), NOT NULL | Дата обновления |

**Первичный ключ:** `(figi, time)` - составной

**Индексы:**
- PRIMARY KEY на `(figi, time)`
- INDEX на `time` (idx_minute_candles_time)
- INDEX на `(figi, time)` (idx_minute_candles_figi_time)

**Триггеры:**
- `calculate_minute_candle_statistics()` - вычисляет статистику свечи при INSERT/UPDATE

---

### 2.4. Торговые сущности

#### 2.4.1. TRADES (Сделки)
**Схема:** `invest`

**Партиционирование:** По дням (RANGE по `time`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **figi** | VARCHAR(255) | PRIMARY KEY, NOT NULL | FIGI инструмента |
| **time** | TIMESTAMP WITHOUT TIME ZONE | PRIMARY KEY, NOT NULL | Время сделки |
| price | NUMERIC(18,9) | NOT NULL | Цена сделки |
| quantity | BIGINT | NOT NULL | Количество (объем) |
| direction | VARCHAR(255) | NULL | Направление сделки |
| trade_direction | VARCHAR(255) | NULL | Направление торговли |
| trade_source | VARCHAR(255) | NULL | Источник данных |
| currency | VARCHAR(255) | NULL | Валюта |
| exchange | VARCHAR(255) | NULL | Биржа |

**Первичный ключ:** `(figi, time)` - составной

---

#### 2.4.2. ORDERS (Заявки)
**Схема:** `invest`

**Примечание:** Структура требует дополнительного анализа.

---

### 2.5. Аналитические сущности

#### 2.5.1. CANDLE_PATTERN_ANALYSIS (Анализ свечных паттернов)
**Схема:** `invest`

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **id** | BIGINT | PRIMARY KEY | Уникальный идентификатор |
| figi | VARCHAR(255) | NULL | FIGI инструмента |
| analysis_date | DATE | NULL | Дата анализа |
| pattern_start_date | DATE | NULL | Дата начала паттерна |
| pattern_end_date | DATE | NULL | Дата окончания паттерна |
| candle_type | VARCHAR(20) | NULL | Тип свечи (BULLISH/BEARISH) |
| consecutive_days | INTEGER | NULL | Количество последовательных дней |
| avg_volume | BIGINT | NULL | Средний объем |
| avg_price_change | NUMERIC(18,9) | NULL | Среднее изменение цены |
| total_price_change | NUMERIC(18,9) | NULL | Общее изменение цены |
| strategy_applicable | CHAR(1) | NULL | Применимость стратегии (Y/N) |
| instrument_name | VARCHAR(255) | NULL | Название инструмента |
| instrument_ticker | VARCHAR(255) | NULL | Тикер инструмента |
| instrument_type | TEXT | NULL | Тип инструмента |
| created_at | TIMESTAMP WITH TIME ZONE | NULL | Дата создания |

**Первичный ключ:** `id`

---

#### 2.5.2. BACKTEST_RESULTS (Результаты бэктестинга)
**Схема:** `invest`

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **id** | BIGINT | PRIMARY KEY | Уникальный идентификатор |
| pattern_analysis_id | BIGINT | NULL | Ссылка на анализ паттернов |
| figi | VARCHAR(255) | NULL | FIGI инструмента |
| analysis_date | DATE | NULL | Дата анализа |
| entry_type | VARCHAR(20) | NULL | Тип входа (LONG/SHORT) |
| entry_price | NUMERIC(18,9) | NULL | Цена входа |
| amount | NUMERIC(18,2) | NULL | Объем позиции |
| stop_loss_percent | NUMERIC(5,2) | NULL | Процент стоп-лосса |
| take_profit_percent | NUMERIC(5,2) | NULL | Процент тейк-профита |
| stop_loss_price | NUMERIC(18,9) | NULL | Цена стоп-лосса |
| take_profit_price | NUMERIC(18,9) | NULL | Цена тейк-профита |
| exit_type | VARCHAR(30) | NULL | Тип выхода |
| result | VARCHAR(20) | NULL | Результат (PROFIT/LOSS) |
| exit_price | NUMERIC(18,9) | NULL | Цена выхода |
| exit_time | TIMESTAMP WITH TIME ZONE | NULL | Время выхода |
| profit_loss | NUMERIC(18,2) | NULL | Прибыль/убыток |
| profit_loss_percent | NUMERIC(8,4) | NULL | Процент прибыли/убытка |
| duration_minutes | INTEGER | NULL | Длительность позиции в минутах |
| instrument_name | VARCHAR(255) | NULL | Название инструмента |
| instrument_ticker | VARCHAR(255) | NULL | Тикер инструмента |
| instrument_type | TEXT | NULL | Тип инструмента |
| created_at | TIMESTAMP WITH TIME ZONE | NULL | Дата создания |

**Первичный ключ:** `id`

**Внешний ключ (логический):**
- `pattern_analysis_id` → `candle_pattern_analysis.id` (0..1 : N)

---

### 2.6. Служебные сущности

#### 2.6.1. SYSTEM_LOGS (Системные логи)
**Схема:** `invest_utils` (доступ через view `invest.system_logs`)

| Атрибут | Тип данных | Ограничения | Описание |
|---------|------------|-------------|----------|
| **id** | BIGSERIAL | PRIMARY KEY | Уникальный идентификатор |
| task_id | VARCHAR(255) | NOT NULL | ID задачи |
| endpoint | VARCHAR(255) | NOT NULL | Название эндпоинта |
| method | VARCHAR(255) | NOT NULL | HTTP метод |
| status | VARCHAR(255) | NOT NULL | Статус выполнения |
| message | TEXT | NOT NULL | Текстовое сообщение |
| start_time | TIMESTAMP WITH TIME ZONE | NOT NULL | Время начала |
| end_time | TIMESTAMP WITH TIME ZONE | NULL | Время завершения |
| duration_ms | BIGINT | NULL | Длительность в миллисекундах |
| created_at | TIMESTAMP WITH TIME ZONE | DEFAULT NOW() | Дата создания |

**Первичный ключ:** `id`

**Триггеры:**
- `calculate_duration_ms()` - вычисляет длительность при установке end_time

---

## 3. ERD - Описание связей между сущностями

### 3.1. Диаграмма связей (текстовое описание)

```
┌─────────────┐
│   SHARES    │
│   FUTURES   │◄──────┐
│ INDICATIVES │       │
└─────────────┘       │
      │               │
      │ (1)           │ (1)
      │               │
      ▼               │
┌─────────────┐       │
│ DIVIDENDS   │       │
└─────────────┘       │
      │               │
      │ (N)           │
      │               │
      ▼               │
┌─────────────┐       │
│FUNDAMENTALS │       │
└─────────────┘       │
      │               │
      │ (1)           │
      │               │
      ▼               │
┌─────────────┐       │
│CLOSE_PRICES │       │
│OPEN_PRICES  │       │
│LAST_PRICES  │       │
│DAILY_CANDLES│       │
│MINUTE_CANDLES│      │
│   TRADES    │       │
└─────────────┘       │
      │               │
      │ (N)           │
      │               │
      ▼               │
┌─────────────┐       │
│CANDLE_PATTERN│      │
│   ANALYSIS   │      │
└─────────────┘       │
      │               │
      │ (1)           │
      │               │
      ▼               │
┌─────────────┐       │
│BACKTEST_    │       │
│  RESULTS    │       │
└─────────────┘       │
                      │
                      │
┌─────────────┐       │
│ SYSTEM_LOGS │       │
└─────────────┘       │
      (независимая)   │
                      │
      Все связи через FIGI
```

### 3.2. Детальное описание связей

#### 3.2.1. Связи через FIGI (инструменты)

**SHARES/FUTURES/INDICATIVES → CLOSE_PRICES**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество цен закрытия
- **Ключ связи:** `shares.figi` = `close_prices.figi`
- **Ограничение:** Нет явного внешнего ключа (связь через логику приложения)

**SHARES/FUTURES/INDICATIVES → OPEN_PRICES**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество цен открытия
- **Ключ связи:** `shares.figi` = `open_prices.figi`

**SHARES/FUTURES/INDICATIVES → LAST_PRICES**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество последних цен
- **Ключ связи:** `shares.figi` = `last_prices.figi`

**SHARES/FUTURES/INDICATIVES → DAILY_CANDLES**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество дневных свечей
- **Ключ связи:** `shares.figi` = `daily_candles.figi`

**SHARES/FUTURES/INDICATIVES → MINUTE_CANDLES**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество минутных свечей
- **Ключ связи:** `shares.figi` = `minute_candles.figi`

**SHARES/FUTURES/INDICATIVES → TRADES**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество сделок
- **Ключ связи:** `shares.figi` = `trades.figi`

**SHARES/FUTURES/INDICATIVES → DIVIDENDS**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество дивидендных событий
- **Ключ связи:** `shares.figi` = `dividends.figi`

**SHARES/FUTURES/INDICATIVES → FUNDAMENTALS**
- **Тип:** Один к одному (1:1)
- **Кардинальность:** Один инструмент имеет один набор фундаментальных показателей
- **Ключ связи:** `shares.figi` = `fundamentals.figi`

**SHARES/FUTURES/INDICATIVES → CANDLE_PATTERN_ANALYSIS**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество анализов паттернов
- **Ключ связи:** `shares.figi` = `candle_pattern_analysis.figi`

**SHARES/FUTURES/INDICATIVES → BACKTEST_RESULTS**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один инструмент может иметь множество результатов бэктестинга
- **Ключ связи:** `shares.figi` = `backtest_results.figi`

#### 3.2.2. Связи между аналитическими сущностями

**CANDLE_PATTERN_ANALYSIS → BACKTEST_RESULTS**
- **Тип:** Один ко многим (1:N)
- **Кардинальность:** Один анализ паттернов может иметь множество результатов бэктестинга
- **Ключ связи:** `candle_pattern_analysis.id` = `backtest_results.pattern_analysis_id`
- **Ограничение:** Внешний ключ не определен на уровне БД (логическая связь)

---

## 4. Ключи

### 4.1. Первичные ключи (Primary Keys)

| Таблица | Первичный ключ | Тип | Описание |
|---------|----------------|-----|----------|
| shares | figi | VARCHAR(255) | Простой ключ |
| futures | figi | VARCHAR(255) | Простой ключ |
| indicatives | figi | VARCHAR(255) | Простой ключ |
| dividends | id | BIGSERIAL | Автоинкрементный |
| fundamentals | id | BIGSERIAL | Автоинкрементный |
| close_prices | (figi, price_date) | Составной | Составной ключ |
| open_prices | (figi, price_date) | Составной | Составной ключ |
| last_prices | (figi, time) | Составной | Составной ключ |
| daily_candles | (figi, time) | Составной | Составной ключ |
| minute_candles | (figi, time) | Составной | Составной ключ |
| trades | (figi, time) | Составной | Составной ключ |
| candle_pattern_analysis | id | BIGINT | Простой ключ |
| backtest_results | id | BIGINT | Простой ключ |
| system_logs | id | BIGSERIAL | Автоинкрементный |

### 4.2. Внешние ключи (Foreign Keys)

**Примечание:** В базе данных не определены явные внешние ключи (FOREIGN KEY constraints). Связи реализованы на уровне приложения через логику использования FIGI.

**Логические внешние ключи:**

| Таблица | Атрибут | Ссылается на | Описание |
|---------|---------|--------------|----------|
| dividends | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| fundamentals | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| close_prices | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| open_prices | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| last_prices | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| daily_candles | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| minute_candles | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| trades | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| candle_pattern_analysis | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| backtest_results | figi | shares.figi / futures.figi / indicatives.figi | Логическая связь |
| backtest_results | pattern_analysis_id | candle_pattern_analysis.id | Логическая связь |

### 4.3. Уникальные индексы

| Таблица | Индекс | Атрибуты | Описание |
|---------|--------|----------|----------|
| shares | PRIMARY KEY | figi | Уникальность FIGI |
| futures | PRIMARY KEY | figi | Уникальность FIGI |
| indicatives | PRIMARY KEY | figi | Уникальность FIGI |
| close_prices | PRIMARY KEY | (figi, price_date) | Уникальность цены закрытия на дату |
| open_prices | PRIMARY KEY | (figi, price_date) | Уникальность цены открытия на дату |
| daily_candles | PRIMARY KEY | (figi, time) | Уникальность дневной свечи |
| minute_candles | PRIMARY KEY | (figi, time) | Уникальность минутной свечи |

---

## 5. Кардинальности связей

### 5.1. Определение кардинальностей

| Связь | Тип | Минимум | Максимум | Описание |
|-------|-----|---------|----------|----------|
| SHARES → CLOSE_PRICES | 1:N | 0 | N | Один инструмент может иметь 0 или много цен закрытия |
| SHARES → OPEN_PRICES | 1:N | 0 | N | Один инструмент может иметь 0 или много цен открытия |
| SHARES → LAST_PRICES | 1:N | 0 | N | Один инструмент может иметь 0 или много последних цен |
| SHARES → DAILY_CANDLES | 1:N | 0 | N | Один инструмент может иметь 0 или много дневных свечей |
| SHARES → MINUTE_CANDLES | 1:N | 0 | N | Один инструмент может иметь 0 или много минутных свечей |
| SHARES → TRADES | 1:N | 0 | N | Один инструмент может иметь 0 или много сделок |
| SHARES → DIVIDENDS | 1:N | 0 | N | Один инструмент может иметь 0 или много дивидендов |
| SHARES → FUNDAMENTALS | 1:1 | 0 | 1 | Один инструмент может иметь 0 или 1 набор фундаментальных данных |
| SHARES → CANDLE_PATTERN_ANALYSIS | 1:N | 0 | N | Один инструмент может иметь 0 или много анализов паттернов |
| SHARES → BACKTEST_RESULTS | 1:N | 0 | N | Один инструмент может иметь 0 или много результатов бэктестинга |
| CANDLE_PATTERN_ANALYSIS → BACKTEST_RESULTS | 1:N | 0 | N | Один анализ паттернов может иметь 0 или много результатов бэктестинга |

---

## 6. Бизнес-правила и ограничения целостности

### 6.1. Правила уникальности

1. **FIGI уникален в рамках типа инструмента:**
   - Каждый FIGI уникален в таблице `shares`
   - Каждый FIGI уникален в таблице `futures`
   - Каждый FIGI уникален в таблице `indicatives`
   - Один FIGI может существовать только в одной из этих таблиц

2. **Уникальность ценовых данных:**
   - Одна цена закрытия на инструмент в день: `(figi, price_date)` уникальны
   - Одна цена открытия на инструмент в день: `(figi, price_date)` уникальны
   - Множество последних цен на инструмент: `(figi, time)` уникальны

3. **Уникальность свечных данных:**
   - Одна дневная свеча на инструмент в день: `(figi, time)` уникальны (time = начало дня)
   - Одна минутная свеча на инструмент в минуту: `(figi, time)` уникальны (time = начало минуты)

4. **Уникальность сделок:**
   - Одна сделка на инструмент в момент времени: `(figi, time)` уникальны

### 6.2. Правила целостности данных

1. **Ссылочная целостность (логическая):**
   - Все `figi` в ценовых таблицах должны существовать в `shares`, `futures` или `indicatives`
   - `pattern_analysis_id` в `backtest_results` должен существовать в `candle_pattern_analysis`
   - **Примечание:** Не реализовано через FOREIGN KEY constraints

2. **Целостность временных данных:**
   - `price_date` в `close_prices` и `open_prices` не может быть в будущем
   - `time` в свечах должен соответствовать торговым сессиям
   - `pattern_start_date` ≤ `pattern_end_date` в `candle_pattern_analysis`
   - `entry_time` ≤ `exit_time` в `backtest_results`

3. **Целостность ценовых данных:**
   - `close_price` > 0
   - `open_price` > 0
   - `high` ≥ `low` в свечах
   - `high` ≥ `open` и `high` ≥ `close` в свечах
   - `low` ≤ `open` и `low` ≤ `close` в свечах
   - `volume` ≥ 0

4. **Целостность вычисляемых полей:**
   - `price_change` = `close` - `open` (проверяется триггером)
   - `body_size` = ABS(`close` - `open`) (проверяется триггером)
   - `upper_shadow` = `high` - GREATEST(`open`, `close`) (проверяется триггером)
   - `lower_shadow` = LEAST(`open`, `close`) - `low` (проверяется триггером)

5. **Целостность бэктестинга:**
   - `stop_loss_price` < `entry_price` для LONG позиций
   - `stop_loss_price` > `entry_price` для SHORT позиций
   - `take_profit_price` > `entry_price` для LONG позиций
   - `take_profit_price` < `entry_price` для SHORT позиций
   - `profit_loss` = (`exit_price` - `entry_price`) * `amount` для LONG
   - `profit_loss` = (`entry_price` - `exit_price`) * `amount` для SHORT

### 6.3. Правила валидации

1. **Валидация типов данных:**
   - `candle_type` ∈ {'BULLISH', 'BEARISH', 'DOJI'}
   - `entry_type` ∈ {'LONG', 'SHORT'}
   - `exit_type` ∈ {'STOP_LOSS', 'TAKE_PROFIT', 'TIME', 'MANUAL'}
   - `result` ∈ {'PROFIT', 'LOSS'}
   - `strategy_applicable` ∈ {'Y', 'N'}

2. **Валидация диапазонов:**
   - `stop_loss_percent` > 0 и < 100
   - `take_profit_percent` > 0 и < 1000
   - `consecutive_days` ≥ 1
   - `duration_minutes` ≥ 0

3. **Валидация валют:**
   - `currency` ∈ {'RUB', 'USD', 'EUR', 'CNY', ...} (список валют биржи)

4. **Валидация бирж:**
   - `exchange` ∈ {'MOEX', 'SPB', 'NASDAQ', 'NYSE', ...} (список бирж)

### 6.4. Правила партиционирования

1. **Партиционирование по времени:**
   - `close_prices`: партиции по месяцам (RANGE по `price_date`)
   - `open_prices`: партиции по месяцам (RANGE по `price_date`)
   - `daily_candles`: партиции по месяцам (RANGE по `time`)
   - `minute_candles`: партиции по дням (RANGE по `time`)
   - `last_prices`: партиции по дням (RANGE по `time`)
   - `trades`: партиции по дням (RANGE по `time`)

2. **Правила создания партиций:**
   - Партиции создаются автоматически через функции
   - Партиции для будущих периодов создаются заранее
   - Старые партиции могут архивироваться или удаляться

### 6.5. Правила обновления данных

1. **Аудит изменений:**
   - `created_at` устанавливается при создании записи
   - `updated_at` обновляется при изменении записи
   - Временная зона: `Europe/Moscow`

2. **Правила обновления свечей:**
   - Незавершенные свечи (`is_complete = false`) могут обновляться
   - Завершенные свечи (`is_complete = true`) не должны изменяться
   - Статистика свечей пересчитывается автоматически через триггеры

3. **Правила обновления цен:**
   - Цены закрытия/открытия обновляются один раз в день
   - Последние цены обновляются в реальном времени
   - Конфликты при вставке обрабатываются через `ON CONFLICT DO NOTHING`

---

## 7. Глоссарий терминов (Glossary)

### 7.1. Общие термины

| Термин | Определение |
|--------|-------------|
| **FIGI** | Financial Instrument Global Identifier - уникальный глобальный идентификатор финансового инструмента |
| **Тикер** | Краткое буквенное обозначение финансового инструмента на бирже |
| **Биржа** | Организованный рынок для торговли финансовыми инструментами (MOEX, SPB, NASDAQ и др.) |
| **Валюта** | Валюта торговли инструмента (RUB, USD, EUR и др.) |
| **Лот** | Минимальное количество единиц инструмента для одной сделки |
| **Минимальный шаг цены** | Минимальное изменение цены инструмента (tick size) |

### 7.2. Типы инструментов

| Термин | Определение |
|--------|-------------|
| **Акция (Share)** | Долевая ценная бумага, представляющая долю собственности в компании |
| **Фьючерс (Future)** | Производный финансовый инструмент, контракт на покупку/продажу базового актива в будущем |
| **Индикатив (Indicative)** | Индикативный инструмент, используемый для расчетов и индексов |
| **Базовый актив** | Актив, лежащий в основе производного инструмента (фьючерса) |

### 7.3. Ценовые данные

| Термин | Определение |
|--------|-------------|
| **Цена закрытия (Close Price)** | Цена последней сделки в торговой сессии за день |
| **Цена открытия (Open Price)** | Цена первой сделки в торговой сессии за день |
| **Последняя цена (Last Price)** | Цена последней совершенной сделки в реальном времени |
| **Максимальная цена (High)** | Максимальная цена за период |
| **Минимальная цена (Low)** | Минимальная цена за период |

### 7.4. Свечные данные

| Термин | Определение |
|--------|-------------|
| **Свеча (Candle)** | Графическое представление ценовых данных за период (OHLC) |
| **Дневная свеча (Daily Candle)** | Свеча, представляющая торговлю за один торговый день |
| **Минутная свеча (Minute Candle)** | Свеча, представляющая торговлю за одну минуту |
| **OHLC** | Open, High, Low, Close - четыре основные цены свечи |
| **Объем (Volume)** | Количество лотов, проданных/купленных за период |
| **Бычья свеча (Bullish)** | Свеча, где цена закрытия выше цены открытия |
| **Медвежья свеча (Bearish)** | Свеча, где цена закрытия ниже цены открытия |
| **Доджи (Doji)** | Свеча, где цена открытия равна цене закрытия |
| **Тело свечи (Body)** | Разница между ценой открытия и закрытия |
| **Верхняя тень (Upper Shadow)** | Разница между максимальной ценой и максимумом из open/close |
| **Нижняя тень (Lower Shadow)** | Разница между минимумом из open/close и минимальной ценой |

### 7.5. Фундаментальные показатели

| Термин | Определение |
|--------|-------------|
| **TTM** | Trailing Twelve Months - показатели за последние 12 месяцев |
| **MRQ** | Most Recent Quarter - показатели за последний квартал |
| **FY** | Fiscal Year - финансовый год |
| **EPS** | Earnings Per Share - прибыль на акцию |
| **P/E** | Price-to-Earnings - отношение цены к прибыли |
| **P/B** | Price-to-Book - отношение цены к балансовой стоимости |
| **P/S** | Price-to-Sales - отношение цены к выручке |
| **EV/EBITDA** | Enterprise Value to EBITDA - отношение стоимости предприятия к EBITDA |
| **ROE** | Return on Equity - рентабельность собственного капитала |
| **ROA** | Return on Assets - рентабельность активов |
| **ROIC** | Return on Invested Capital - рентабельность инвестированного капитала |
| **EBITDA** | Earnings Before Interest, Taxes, Depreciation and Amortization |
| **Free Cash Flow (FCF)** | Свободный денежный поток |
| **Market Capitalization** | Рыночная капитализация компании |
| **Beta** | Бета-коэффициент - мера волатильности относительно рынка |

### 7.6. Дивиденды

| Термин | Определение |
|--------|-------------|
| **Дивиденд** | Выплата акционерам из прибыли компании |
| **Дата объявления (Declared Date)** | Дата, когда компания объявляет о выплате дивидендов |
| **Дата фиксации реестра (Record Date)** | Дата, на которую определяется список акционеров для получения дивидендов |
| **Дата выплаты (Payment Date)** | Дата фактической выплаты дивидендов |
| **Дивидендная доходность** | Отношение дивиденда к цене акции |

### 7.7. Торговля

| Термин | Определение |
|--------|-------------|
| **Сделка (Trade)** | Отдельная операция покупки/продажи инструмента |
| **Заявка (Order)** | Приказ на покупку/продажу инструмента |
| **Направление (Direction)** | Покупка (BUY) или продажа (SELL) |
| **Объем (Quantity)** | Количество единиц инструмента в сделке |
| **Короткая продажа (Short)** | Продажа инструмента, которым трейдер не владеет |

### 7.8. Аналитика

| Термин | Определение |
|--------|-------------|
| **Паттерн (Pattern)** | Узнаваемая формация на графике цен |
| **Бэктестинг (Backtesting)** | Тестирование торговой стратегии на исторических данных |
| **Стоп-лосс (Stop Loss)** | Приказ на закрытие позиции при достижении определенного уровня убытка |
| **Тейк-профит (Take Profit)** | Приказ на закрытие позиции при достижении определенного уровня прибыли |
| **Вход в позицию (Entry)** | Момент открытия торговой позиции |
| **Выход из позиции (Exit)** | Момент закрытия торговой позиции |
| **Прибыль/Убыток (P/L)** | Финансовый результат торговой операции |

### 7.9. Технические термины

| Термин | Определение |
|--------|-------------|
| **Партиционирование (Partitioning)** | Разделение таблицы на несколько физических частей по определенному критерию |
| **Индекс (Index)** | Структура данных для ускорения поиска |
| **Триггер (Trigger)** | Автоматически выполняемая процедура при определенных событиях |
| **Представление (View)** | Виртуальная таблица, основанная на результате SQL-запроса |
| **Материализованное представление (Materialized View)** | Представление, данные которого физически хранятся в базе данных |
| **Схема (Schema)** | Логическая группа объектов базы данных |

---

## 8. Определения данных (Data Dictionary)

### 8.1. Типы данных

| Тип данных | Диапазон/Формат | Использование |
|------------|-----------------|---------------|
| **VARCHAR(n)** | Строка переменной длины до n символов | Текстовые данные (тикеры, названия) |
| **CHAR(n)** | Строка фиксированной длины n символов | Короткие коды (Y/N флаги) |
| **TEXT** | Строка неограниченной длины | Длинные текстовые данные |
| **INTEGER** | -2,147,483,648 до 2,147,483,647 | Целые числа (количество дней, длительность) |
| **BIGINT** | -9,223,372,036,854,775,808 до 9,223,372,036,854,775,807 | Большие целые числа (объемы, ID) |
| **NUMERIC(p,s)** | Точные числа с p цифрами, s после запятой | Финансовые значения (цены, проценты) |
| **DECIMAL(p,s)** | Аналогично NUMERIC | Финансовые значения |
| **BOOLEAN** | TRUE / FALSE | Логические флаги |
| **DATE** | ГГГГ-ММ-ДД | Даты без времени |
| **TIMESTAMP** | ГГГГ-ММ-ДД ЧЧ:ММ:СС | Дата и время |
| **TIMESTAMP WITH TIME ZONE** | ГГГГ-ММ-ДД ЧЧ:ММ:СС+TZ | Дата и время с часовым поясом |
| **BIGSERIAL** | Автоинкрементное BIGINT | Автоматические ID |

### 8.2. Соглашения об именовании

1. **Таблицы:** нижний регистр, множественное число, подчеркивания для разделения слов
   - Примеры: `shares`, `close_prices`, `daily_candles`

2. **Атрибуты:** нижний регистр, подчеркивания для разделения слов
   - Примеры: `figi`, `price_date`, `created_at`

3. **Первичные ключи:** обычно `id` для суррогатных ключей, или естественный ключ (`figi`)

4. **Внешние ключи:** имя таблицы в единственном числе + `_id` или имя атрибута родительской таблицы
   - Примеры: `pattern_analysis_id`, `figi`

5. **Индексы:** префикс `idx_` + имя таблицы + имя атрибута
   - Примеры: `idx_close_prices_date`, `idx_fundamentals_figi`

6. **Функции:** нижний регистр, подчеркивания
   - Примеры: `calculate_daily_candle_statistics()`, `create_minute_candles_partition()`

7. **Триггеры:** обычно совпадают с именем функции

### 8.3. Значения по умолчанию

| Атрибут | Значение по умолчанию | Описание |
|---------|----------------------|----------|
| `created_at` | `CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow'` | Текущее время в московской таймзоне |
| `updated_at` | `CURRENT_TIMESTAMP AT TIME ZONE 'Europe/Moscow'` | Текущее время в московской таймзоне |
| `sell_available_flag` | `FALSE` | По умолчанию продажа недоступна |
| `buy_available_flag` | `FALSE` | По умолчанию покупка недоступна |
| `id` (BIGSERIAL) | Автоинкремент | Автоматически генерируемый ID |

### 8.4. Ограничения NULL

| Категория | Правило |
|-----------|---------|
| **Первичные ключи** | Всегда NOT NULL |
| **Обязательные бизнес-атрибуты** | NOT NULL (figi, price_date, time, цены) |
| **Вычисляемые поля** | Могут быть NULL до вычисления |
| **Опциональные атрибуты** | Могут быть NULL (сектор, статус торговли) |
| **Внешние ключи (логические)** | Могут быть NULL (опциональные связи) |

---

## 9. Примечания

1. **Отсутствие явных внешних ключей:** В базе данных не определены FOREIGN KEY constraints. Связи реализованы на уровне приложения. Это может быть сделано для:
   - Гибкости при работе с партиционированными таблицами
   - Производительности при массовых операциях
   - Возможности работы с данными из разных источников

2. **Партиционирование:** Большинство таблиц с временными данными партиционированы. Это требует специального подхода к созданию партиций и управлению ими.

3. **Вычисляемые поля:** Многие атрибуты вычисляются автоматически через триггеры. Это обеспечивает целостность данных, но требует понимания логики триггеров.

4. **Временные зоны:** Все временные метки хранятся с учетом временной зоны `Europe/Moscow`. Это важно учитывать при работе с данными.

5. **Представления (Views):** Многие таблицы доступны через представления в схеме `invest`, которые являются синонимами таблиц из других схем.

---
