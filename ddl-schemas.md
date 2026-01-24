# DDL описания схем базы данных InvestmentDataLoaderService

Данный документ содержит DDL (Data Definition Language) описания всех схем базы данных, полученные из реальной БД.

**Дата создания:** 2026-01-24  
**Источник:** PostgreSQL база данных через MCP

---

## Содержание

1. [invest_ref - Справочные данные](#1-invest_ref---справочные-данные)
2. [invest_prices - Ценовые данные](#2-invest_prices---ценовые-данные)
3. [invest_candles - Свечные данные](#3-invest_candles---свечные-данные)
4. [invest_utils - Служебные таблицы](#4-invest_utils---служебные-таблицы)
5. [invest - Основная схема](#5-invest---основная-схема)
6. [invest_views - Представления](#6-invest_views---представления)

---

## 1. invest_ref - Справочные данные

Схема содержит справочную информацию об инструментах, дивидендах и фундаментальных показателях.

### 1.1. CREATE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS invest_ref;
```

### 1.2. Таблицы

#### 1.2.1. shares (Акции)

```sql
CREATE TABLE invest_ref.shares (
    figi VARCHAR(255) NOT NULL,
    ticker VARCHAR(255),
    name VARCHAR(255),
    currency VARCHAR(255),
    exchange VARCHAR(255),
    sector VARCHAR(255),
    trading_status VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    short_enabled BOOLEAN,
    asset_uid VARCHAR(255),
    lot INTEGER,
    min_price_increment NUMERIC,
    CONSTRAINT shares_pkey PRIMARY KEY (figi)
);
```

#### 1.2.2. futures (Фьючерсы)

```sql
CREATE TABLE invest_ref.futures (
    figi VARCHAR(255) NOT NULL,
    ticker VARCHAR(255),
    asset_type VARCHAR(255),
    basic_asset VARCHAR(255),
    currency VARCHAR(255),
    exchange VARCHAR(255),
    short_enabled BOOLEAN,
    expiration_date TIMESTAMP WITHOUT TIME ZONE,
    min_price_increment NUMERIC,
    lot INTEGER,
    basic_asset_size NUMERIC(18,9),
    created_at TIMESTAMP(6) NOT NULL,
    updated_at TIMESTAMP(6) NOT NULL,
    CONSTRAINT futures_pkey PRIMARY KEY (figi)
);
```

#### 1.2.3. indicatives (Индикативы)

```sql
CREATE TABLE invest_ref.indicatives (
    figi VARCHAR(255) NOT NULL,
    ticker VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    currency VARCHAR(255) NOT NULL,
    exchange VARCHAR(255) NOT NULL,
    class_code VARCHAR(255),
    uid VARCHAR(255),
    sell_available_flag BOOLEAN DEFAULT FALSE,
    buy_available_flag BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    CONSTRAINT indicatives_pkey PRIMARY KEY (figi)
);
```

#### 1.2.4. dividends (Дивиденды)

```sql
CREATE TABLE invest_ref.dividends (
    id BIGSERIAL NOT NULL,
    figi VARCHAR(255) NOT NULL,
    declared_date DATE,
    record_date DATE NOT NULL,
    payment_date DATE,
    dividend_value NUMERIC(18,9),
    currency VARCHAR(10) NOT NULL,
    dividend_type VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    CONSTRAINT dividends_pkey PRIMARY KEY (id)
);

CREATE INDEX idx_dividends_figi ON invest_ref.dividends(figi);
```

#### 1.2.5. fundamentals (Фундаментальные показатели)

```sql
CREATE TABLE invest_ref.fundamentals (
    id BIGSERIAL NOT NULL,
    figi VARCHAR(255) NOT NULL,
    asset_uid VARCHAR(255),
    domicile_indicator_code VARCHAR(10),
    currency VARCHAR(10),
    -- Дивидендные показатели
    dividend_yield_daily_ttm NUMERIC(18,9),
    dividend_rate_ttm NUMERIC(18,9),
    dividend_payout_ratio_fy NUMERIC(18,9),
    forward_annual_dividend_yield NUMERIC(18,9),
    five_years_average_dividend_yield NUMERIC(18,9),
    dividends_per_share NUMERIC(18,9),
    five_year_annual_dividend_growth_rate NUMERIC(18,9),
    -- Показатели оценки
    price_to_sales_ttm NUMERIC(18,9),
    price_to_book_ttm NUMERIC(18,9),
    price_to_free_cash_flow_ttm NUMERIC(18,9),
    pe_ratio_ttm NUMERIC(18,9),
    ev_to_sales NUMERIC(18,9),
    ev_to_ebitda_mrq NUMERIC(18,9),
    -- Показатели прибыльности
    eps_ttm NUMERIC(18,9),
    diluted_eps_ttm NUMERIC(18,9),
    net_income_ttm NUMERIC(18,9),
    ebitda_ttm NUMERIC(18,9),
    free_cash_flow_ttm NUMERIC(18,9),
    revenue_ttm NUMERIC(18,9),
    net_margin_mrq NUMERIC(18,9),
    -- Показатели рентабельности
    roe NUMERIC(18,9),
    roa NUMERIC(18,9),
    roic NUMERIC(18,9),
    -- Показатели роста
    revenue_change_five_years NUMERIC(18,9),
    five_year_annual_revenue_growth_rate NUMERIC(18,9),
    one_year_annual_revenue_growth_rate NUMERIC(18,9),
    three_year_annual_revenue_growth_rate NUMERIC(18,9),
    eps_change_five_years NUMERIC(18,9),
    ebitda_change_five_years NUMERIC(18,9),
    -- Показатели долга
    total_debt_mrq NUMERIC(18,9),
    total_debt_to_equity_mrq NUMERIC(18,9),
    total_debt_to_ebitda_mrq NUMERIC(18,9),
    net_debt_to_ebitda NUMERIC(18,9),
    total_debt_change_five_years NUMERIC(18,9),
    -- Показатели ликвидности
    current_ratio_mrq NUMERIC(18,9),
    fixed_charge_coverage_ratio_fy NUMERIC(18,9),
    net_interest_margin_mrq NUMERIC(18,9),
    -- Рыночные показатели
    market_capitalization NUMERIC(18,9),
    total_enterprise_value_mrq NUMERIC(18,9),
    shares_outstanding NUMERIC(18,9),
    free_float NUMERIC(18,9),
    beta NUMERIC(18,9),
    -- Ценовые показатели
    high_price_last_52_weeks NUMERIC(18,9),
    low_price_last_52_weeks NUMERIC(18,9),
    -- Объемы торгов
    average_daily_volume_last_4_weeks NUMERIC(18,9),
    average_daily_volume_last_10_days NUMERIC(18,9),
    -- Показатели компании
    number_of_employees NUMERIC(18,9),
    adr_to_common_share_ratio NUMERIC(18,9),
    buy_back_ttm NUMERIC(18,9),
    free_cash_flow_to_price NUMERIC(18,9),
    -- Даты
    fiscal_period_start_date TIMESTAMP WITH TIME ZONE,
    fiscal_period_end_date TIMESTAMP WITH TIME ZONE,
    ex_dividend_date TIMESTAMP WITH TIME ZONE,
    -- Служебные поля
    created_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    updated_at TIMESTAMP WITH TIME ZONE DEFAULT timezone('Europe/Moscow'::text, CURRENT_TIMESTAMP),
    CONSTRAINT fundamentals_pkey PRIMARY KEY (id)
);

CREATE INDEX idx_fundamentals_figi ON invest_ref.fundamentals(figi);
```

---

## 2. invest_prices - Ценовые данные

Схема содержит ценовые данные: цены открытия, закрытия, последние цены и сделки.

### 2.1. CREATE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS invest_prices;
```

### 2.2. Таблицы

#### 2.2.1. close_prices (Цены закрытия) - Партиционированная

**Родительская таблица:**

```sql
CREATE TABLE invest_prices.close_prices (
    figi VARCHAR(255) NOT NULL,
    price_date DATE NOT NULL,
    close_price NUMERIC(18,9) NOT NULL,
    currency VARCHAR(255) NOT NULL,
    exchange VARCHAR(255) NOT NULL,
    instrument_type VARCHAR(255) NOT NULL,
    created_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    CONSTRAINT close_prices_pkey PRIMARY KEY (figi, price_date)
) PARTITION BY RANGE (price_date);
```

**Индексы:**

```sql
CREATE INDEX idx_close_prices_date ON ONLY invest_prices.close_prices(price_date);
CREATE INDEX idx_close_prices_figi_date ON ONLY invest_prices.close_prices(figi, price_date);
```

**Пример создания партиции:**

```sql
CREATE TABLE invest_prices.close_prices_2025_01 
PARTITION OF invest_prices.close_prices
FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');
```

#### 2.2.2. open_prices (Цены открытия) - Партиционированная

```sql
CREATE TABLE invest_prices.open_prices (
    figi VARCHAR(255) NOT NULL,
    price_date DATE NOT NULL,
    open_price NUMERIC(18,9) NOT NULL,
    currency VARCHAR(255) NOT NULL,
    exchange VARCHAR(255) NOT NULL,
    instrument_type VARCHAR(255) NOT NULL,
    created_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    CONSTRAINT open_prices_pkey PRIMARY KEY (figi, price_date)
) PARTITION BY RANGE (price_date);
```

#### 2.2.3. close_prices_evening_session (Цены закрытия вечерней сессии) - Партиционированная

```sql
CREATE TABLE invest_prices.close_prices_evening_session (
    figi VARCHAR(255) NOT NULL,
    price_date DATE NOT NULL,
    close_price NUMERIC(18,9) NOT NULL,
    currency VARCHAR(255) NOT NULL,
    exchange VARCHAR(255) NOT NULL,
    instrument_type VARCHAR(255) NOT NULL,
    created_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    CONSTRAINT close_prices_evening_session_pkey PRIMARY KEY (figi, price_date)
) PARTITION BY RANGE (price_date);
```

#### 2.2.4. last_prices (Последние цены) - Партиционированная

```sql
CREATE TABLE invest_prices.last_prices (
    figi VARCHAR(255) NOT NULL,
    time TIMESTAMP WITHOUT TIME ZONE NOT NULL,
    price NUMERIC(18,9) NOT NULL,
    currency VARCHAR(255) NOT NULL,
    exchange VARCHAR(255) NOT NULL,
    CONSTRAINT last_prices_pkey PRIMARY KEY (figi, time)
) PARTITION BY RANGE (time);
```

**Пример создания дневной партиции:**

```sql
CREATE TABLE invest_prices.last_prices_2025_01_15 
PARTITION OF invest_prices.last_prices
FOR VALUES FROM ('2025-01-15 00:00:00') TO ('2025-01-16 00:00:00');
```

#### 2.2.5. trades (Сделки) - Партиционированная

```sql
CREATE TABLE invest_prices.trades (
    figi VARCHAR(255) NOT NULL,
    time TIMESTAMP WITHOUT TIME ZONE NOT NULL,
    price NUMERIC(18,9) NOT NULL,
    quantity BIGINT NOT NULL,
    direction VARCHAR(255),
    trade_direction VARCHAR(255),
    trade_source VARCHAR(255),
    currency VARCHAR(255),
    exchange VARCHAR(255),
    CONSTRAINT trades_pkey PRIMARY KEY (figi, time)
) PARTITION BY RANGE (time);
```

---

## 3. invest_candles - Свечные данные

Схема содержит свечные данные: дневные и минутные свечи, анализ паттернов и результаты бэктестинга.

### 3.1. CREATE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS invest_candles;
```

### 3.2. Таблицы

#### 3.2.1. daily_candles (Дневные свечи) - Партиционированная

**Родительская таблица:**

```sql
CREATE TABLE invest_candles.daily_candles (
    figi VARCHAR(255) NOT NULL,
    time TIMESTAMP(6) WITH TIME ZONE NOT NULL,
    open NUMERIC(18,9) NOT NULL,
    high NUMERIC(18,9) NOT NULL,
    low NUMERIC(18,9) NOT NULL,
    close NUMERIC(18,9) NOT NULL,
    volume BIGINT NOT NULL,
    is_complete BOOLEAN NOT NULL,
    -- Вычисляемые поля (заполняются триггером)
    price_change NUMERIC(18,9),
    price_change_percent NUMERIC(18,4),
    candle_type VARCHAR(20),
    body_size NUMERIC(18,9),
    upper_shadow NUMERIC(18,9),
    lower_shadow NUMERIC(18,9),
    high_low_range NUMERIC(18,9),
    average_price NUMERIC(18,2),
    -- Служебные поля
    created_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    CONSTRAINT daily_candles_pkey PRIMARY KEY (figi, time)
) PARTITION BY RANGE (time);
```

**Индексы:**

```sql
CREATE INDEX idx_daily_candles_time ON ONLY invest_candles.daily_candles(time);
CREATE INDEX idx_daily_candles_figi_time ON ONLY invest_candles.daily_candles(figi, time);
```

**Пример создания месячной партиции:**

```sql
CREATE TABLE invest_candles.daily_candles_2025_01 
PARTITION OF invest_candles.daily_candles
FOR VALUES FROM ('2024-12-31 21:00:00+00') TO ('2025-01-31 21:00:00+00');
```

**Триггер:**

```sql
CREATE TRIGGER calculate_daily_candle_statistics
BEFORE INSERT OR UPDATE ON invest_candles.daily_candles
FOR EACH ROW
EXECUTE FUNCTION invest_candles.calculate_daily_candle_statistics();
```

#### 3.2.2. minute_candles (Минутные свечи) - Партиционированная

**Родительская таблица:**

```sql
CREATE TABLE invest_candles.minute_candles (
    figi VARCHAR(255) NOT NULL,
    time TIMESTAMP(6) WITH TIME ZONE NOT NULL,
    open NUMERIC(18,9) NOT NULL,
    high NUMERIC(18,9) NOT NULL,
    low NUMERIC(18,9) NOT NULL,
    close NUMERIC(18,9) NOT NULL,
    volume BIGINT NOT NULL,
    is_complete BOOLEAN NOT NULL,
    -- Вычисляемые поля (заполняются триггером)
    price_change NUMERIC(18,9),
    price_change_percent NUMERIC(18,4),
    candle_type VARCHAR(20),
    body_size NUMERIC(18,9),
    upper_shadow NUMERIC(18,9),
    lower_shadow NUMERIC(18,9),
    high_low_range NUMERIC(18,9),
    average_price NUMERIC(18,2),
    -- Служебные поля
    created_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    updated_at TIMESTAMP(6) WITH TIME ZONE DEFAULT NOW() NOT NULL,
    CONSTRAINT minute_candles_pkey PRIMARY KEY (figi, time)
) PARTITION BY RANGE (time);
```

**Индексы:**

```sql
CREATE INDEX idx_minute_candles_time ON ONLY invest_candles.minute_candles(time);
CREATE INDEX idx_minute_candles_figi_time ON ONLY invest_candles.minute_candles(figi, time);
```

**Пример создания дневной партиции:**

```sql
CREATE TABLE invest_candles.minute_candles_2025_01_15 
PARTITION OF invest_candles.minute_candles
FOR VALUES FROM ('2025-01-14 21:00:00+00') TO ('2025-01-15 21:00:00+00');
```

**Триггер:**

```sql
CREATE TRIGGER calculate_minute_candle_statistics
BEFORE INSERT OR UPDATE ON invest_candles.minute_candles
FOR EACH ROW
EXECUTE FUNCTION invest_candles.calculate_minute_candle_statistics();
```

#### 3.2.3. candle_pattern_analysis (Анализ свечных паттернов)

```sql
CREATE TABLE invest_candles.candle_pattern_analysis (
    id BIGINT NOT NULL,
    figi VARCHAR(255),
    analysis_date DATE,
    pattern_start_date DATE,
    pattern_end_date DATE,
    candle_type VARCHAR(20),
    consecutive_days INTEGER,
    avg_volume BIGINT,
    avg_price_change NUMERIC(18,9),
    total_price_change NUMERIC(18,9),
    strategy_applicable CHAR(1),
    instrument_name VARCHAR(255),
    instrument_ticker VARCHAR(255),
    instrument_type TEXT,
    created_at TIMESTAMP WITH TIME ZONE,
    CONSTRAINT pk_candle_pattern_analysis PRIMARY KEY (id)
);

CREATE INDEX idx_candle_pattern_analysis_figi ON invest_candles.candle_pattern_analysis(figi);
CREATE INDEX idx_candle_pattern_analysis_date ON invest_candles.candle_pattern_analysis(analysis_date);
```

#### 3.2.4. backtest_results (Результаты бэктестинга)

```sql
CREATE TABLE invest_candles.backtest_results (
    id BIGINT NOT NULL,
    pattern_analysis_id BIGINT,
    figi VARCHAR(255),
    analysis_date DATE,
    entry_type VARCHAR(20),
    entry_price NUMERIC(18,9),
    amount NUMERIC(18,2),
    stop_loss_percent NUMERIC(5,2),
    take_profit_percent NUMERIC(5,2),
    stop_loss_price NUMERIC(18,9),
    take_profit_price NUMERIC(18,9),
    exit_type VARCHAR(30),
    result VARCHAR(20),
    exit_price NUMERIC(18,9),
    exit_time TIMESTAMP WITH TIME ZONE,
    profit_loss NUMERIC(18,2),
    profit_loss_percent NUMERIC(8,4),
    duration_minutes INTEGER,
    instrument_name VARCHAR(255),
    instrument_ticker VARCHAR(255),
    instrument_type TEXT,
    created_at TIMESTAMP WITH TIME ZONE,
    CONSTRAINT backtest_results_pkey PRIMARY KEY (id)
);

CREATE INDEX idx_backtest_results_figi ON invest_candles.backtest_results(figi);
CREATE INDEX idx_backtest_results_pattern_analysis_id ON invest_candles.backtest_results(pattern_analysis_id);
CREATE INDEX idx_backtest_results_date ON invest_candles.backtest_results(analysis_date);
```

#### 3.2.5. special_trading_hours (Специальные торговые часы)

```sql
CREATE TABLE invest_candles.special_trading_hours (
    id BIGSERIAL NOT NULL,
    figi VARCHAR(255) NOT NULL,
    instrument_type VARCHAR(255),
    day_type VARCHAR(255),
    start_hour INTEGER,
    start_minute INTEGER,
    end_hour INTEGER,
    end_minute INTEGER,
    description VARCHAR(255),
    created_by VARCHAR(255),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT special_trading_hours_pkey PRIMARY KEY (id)
);

CREATE INDEX idx_special_trading_hours_figi ON invest_candles.special_trading_hours(figi);
```

---

## 4. invest_utils - Служебные таблицы

Схема содержит служебные таблицы для логирования и контроля качества данных.

### 4.1. CREATE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS invest_utils;
```

### 4.2. Таблицы

#### 4.2.1. system_logs (Системные логи)

```sql
CREATE TABLE invest_utils.system_logs (
    id BIGSERIAL NOT NULL,
    task_id VARCHAR(255) NOT NULL,
    endpoint VARCHAR(255) NOT NULL,
    method VARCHAR(255) NOT NULL,
    status VARCHAR(255) NOT NULL,
    message TEXT NOT NULL,
    start_time TIMESTAMP WITH TIME ZONE NOT NULL,
    end_time TIMESTAMP WITH TIME ZONE,
    duration_ms BIGINT,
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT system_logs_pkey PRIMARY KEY (id)
);

CREATE INDEX idx_system_logs_task_id ON invest_utils.system_logs(task_id);
CREATE INDEX idx_system_logs_created_at ON invest_utils.system_logs(created_at);
```

**Триггер:**

```sql
CREATE TRIGGER calculate_duration_ms
BEFORE INSERT OR UPDATE ON invest_utils.system_logs
FOR EACH ROW
EXECUTE FUNCTION invest_utils.calculate_duration_ms();
```

#### 4.2.2. data_quality_issues (Проблемы качества данных)

```sql
CREATE TABLE invest_utils.data_quality_issues (
    id BIGSERIAL NOT NULL,
    check_type VARCHAR(255) NOT NULL,
    check_name VARCHAR(255) NOT NULL,
    figi VARCHAR(255),
    issue_date DATE,
    issue_description TEXT NOT NULL,
    severity VARCHAR(50),
    status VARCHAR(50),
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
    CONSTRAINT data_quality_issues_pkey PRIMARY KEY (id)
);

CREATE INDEX idx_data_quality_issues_figi ON invest_utils.data_quality_issues(figi);
CREATE INDEX idx_data_quality_issues_date ON invest_utils.data_quality_issues(issue_date);
CREATE INDEX idx_data_quality_issues_type ON invest_utils.data_quality_issues(check_type);
```

---

## 5. invest - Основная схема

Схема содержит представления (views) для удобного доступа к данным из других схем и некоторые таблицы.

### 5.1. CREATE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS invest;
```

### 5.2. Таблицы

#### 5.2.1. last_prices (Последние цены) - Партиционированная

**Примечание:** Таблица находится в схеме `invest`, но партиционирована аналогично `invest_prices.last_prices`.

```sql
CREATE TABLE invest.last_prices (
    figi VARCHAR(255) NOT NULL,
    time TIMESTAMP WITHOUT TIME ZONE NOT NULL,
    price NUMERIC(18,9) NOT NULL,
    currency VARCHAR(255) NOT NULL,
    exchange VARCHAR(255) NOT NULL,
    CONSTRAINT last_prices_pkey PRIMARY KEY (figi, time)
) PARTITION BY RANGE (time);
```

### 5.3. Представления (Views)

#### 5.3.1. Синонимы таблиц из других схем

```sql
-- Справочные таблицы
CREATE VIEW invest.shares AS SELECT * FROM invest_ref.shares;
CREATE VIEW invest.futures AS SELECT * FROM invest_ref.futures;
CREATE VIEW invest.indicatives AS SELECT * FROM invest_ref.indicatives;
CREATE VIEW invest.dividends AS SELECT * FROM invest_ref.dividends;
CREATE VIEW invest.fundamentals AS SELECT * FROM invest_ref.fundamentals;

-- Ценовые таблицы
CREATE VIEW invest.close_prices AS SELECT * FROM invest_prices.close_prices;
CREATE VIEW invest.open_prices AS SELECT * FROM invest_prices.open_prices;
CREATE VIEW invest.close_prices_evening_session AS SELECT * FROM invest_prices.close_prices_evening_session;
CREATE VIEW invest.trades AS SELECT * FROM invest_prices.trades;

-- Свечные таблицы
CREATE VIEW invest.daily_candles AS SELECT * FROM invest_candles.daily_candles;
CREATE VIEW invest.minute_candles AS SELECT * FROM invest_candles.minute_candles;

-- Служебные таблицы
CREATE VIEW invest.system_logs AS SELECT * FROM invest_utils.system_logs;
CREATE VIEW invest.data_quality_issues AS SELECT * FROM invest_utils.data_quality_issues;
```

#### 5.3.2. Представления с обогащением данных

```sql
-- Анализ паттернов с информацией об инструментах
CREATE VIEW invest.candle_pattern_analysis AS
SELECT 
    cpa.*,
    COALESCE(s.name, f.ticker, ind.name) AS instrument_name,
    COALESCE(s.ticker, f.ticker, ind.ticker) AS instrument_ticker,
    CASE 
        WHEN s.figi IS NOT NULL THEN 'share'
        WHEN f.figi IS NOT NULL THEN 'future'
        WHEN ind.figi IS NOT NULL THEN 'indicative'
        ELSE 'unknown'
    END AS instrument_type
FROM invest_candles.candle_pattern_analysis cpa
LEFT JOIN invest_ref.shares s ON cpa.figi = s.figi
LEFT JOIN invest_ref.futures f ON cpa.figi = f.figi
LEFT JOIN invest_ref.indicatives ind ON cpa.figi = ind.figi;

-- Результаты бэктестинга с информацией об инструментах
CREATE VIEW invest.backtest_results AS
SELECT 
    br.*,
    COALESCE(s.name, f.ticker, ind.name) AS instrument_name,
    COALESCE(s.ticker, f.ticker, ind.ticker) AS instrument_ticker,
    CASE 
        WHEN s.figi IS NOT NULL THEN 'share'
        WHEN f.figi IS NOT NULL THEN 'future'
        WHEN ind.figi IS NOT NULL THEN 'indicative'
        ELSE 'unknown'
    END AS instrument_type
FROM invest_candles.backtest_results br
LEFT JOIN invest_ref.shares s ON br.figi = s.figi
LEFT JOIN invest_ref.futures f ON br.figi = f.figi
LEFT JOIN invest_ref.indicatives ind ON br.figi = ind.figi;

-- Специальные торговые часы с информацией об инструментах
CREATE VIEW invest.special_trading_hours AS
SELECT 
    sth.*,
    COALESCE(s.name, f.ticker, ind.name) AS instrument_name,
    COALESCE(s.ticker, f.ticker, ind.ticker) AS instrument_ticker
FROM invest_candles.special_trading_hours sth
LEFT JOIN invest_ref.shares s ON sth.figi = s.figi
LEFT JOIN invest_ref.futures f ON sth.figi = f.figi
LEFT JOIN invest_ref.indicatives ind ON sth.figi = ind.figi;

-- Исторические экстремумы цен
CREATE VIEW invest.historical_price_extremes AS
SELECT * FROM invest_views.historical_price_extremes;

-- Историческая агрегация объемов
CREATE VIEW invest.history_volume_aggregation AS
SELECT * FROM invest_views.history_volume_aggregation;
```

---

## 6. invest_views - Представления

Схема содержит материализованные представления и обычные представления для аналитики.

### 6.1. CREATE SCHEMA

```sql
CREATE SCHEMA IF NOT EXISTS invest_views;
```

### 6.2. Материализованные представления

#### 6.2.1. daily_volume_aggregation (Дневная агрегация объемов)

```sql
CREATE MATERIALIZED VIEW invest_views.daily_volume_aggregation AS
SELECT 
    minute_candles.figi,
    CASE
        WHEN s.figi IS NOT NULL THEN 'share'
        WHEN f.figi IS NOT NULL THEN 'future'
        ELSE 'unknown'
    END AS instrument_type,
    DATE(timezone('Europe/Moscow', minute_candles.time)) AS trade_date,
    SUM(minute_candles.volume) AS total_volume,
    COUNT(*) AS total_candles,
    ROUND(AVG(minute_candles.volume), 2) AS avg_volume_per_candle,
    -- Утренняя сессия (06:59-09:59 MSK, Пн-Пт)
    SUM(CASE 
        WHEN EXTRACT(DOW FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 1 AND 5
        AND (
            (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 6 
             AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) = 59)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 7 AND 8)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 9 
                AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) <= 59)
        ) THEN minute_candles.volume
        ELSE 0
    END) AS morning_session_volume,
    -- Основная сессия (10:00-18:59 MSK, Пн-Пт)
    SUM(CASE 
        WHEN EXTRACT(DOW FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 1 AND 5
        AND (
            (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 10 AND 17)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 18 
                AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) <= 59)
        ) THEN minute_candles.volume
        ELSE 0
    END) AS main_session_volume,
    -- Вечерняя сессия (19:00-23:50 MSK, Пн-Пт)
    SUM(CASE 
        WHEN EXTRACT(DOW FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 1 AND 5
        AND (
            (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 19 AND 22)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 23 
                AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) <= 50)
        ) THEN minute_candles.volume
        ELSE 0
    END) AS evening_session_volume,
    -- Выходная биржевая сессия (10:00-18:59 MSK, Сб-Вс)
    SUM(CASE 
        WHEN EXTRACT(DOW FROM timezone('Europe/Moscow', minute_candles.time)) IN (0, 6)
        AND (
            (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 10 AND 17)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 18 
                AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) <= 59)
        ) THEN minute_candles.volume
        ELSE 0
    END) AS weekend_exchange_session_volume,
    -- Выходная OTC сессия (02:00-09:59 и 19:00-23:50 MSK, Сб-Вс)
    SUM(CASE 
        WHEN EXTRACT(DOW FROM timezone('Europe/Moscow', minute_candles.time)) IN (0, 6)
        AND (
            (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 2 AND 8)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 9 
                AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) <= 59)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) BETWEEN 19 AND 22)
            OR (EXTRACT(HOUR FROM timezone('Europe/Moscow', minute_candles.time)) = 23 
                AND EXTRACT(MINUTE FROM timezone('Europe/Moscow', minute_candles.time)) <= 50)
        ) THEN minute_candles.volume
        ELSE 0
    END) AS weekend_otc_session_volume,
    MIN(timezone('Europe/Moscow', minute_candles.time)) AS first_candle_time,
    MAX(timezone('Europe/Moscow', minute_candles.time)) AS last_candle_time,
    NOW() AT TIME ZONE 'Europe/Moscow' AS last_updated
FROM invest_candles.minute_candles
LEFT JOIN invest_ref.shares s ON minute_candles.figi = s.figi
LEFT JOIN invest_ref.futures f ON minute_candles.figi = f.figi
GROUP BY minute_candles.figi, s.figi, f.figi, DATE(timezone('Europe/Moscow', minute_candles.time))
ORDER BY minute_candles.figi, trade_date;
```

#### 6.2.2. history_volume_aggregation (Историческая агрегация объемов)

**Примечание:** Полное определение см. в разделе о материализованных представлениях выше. Это представление содержит агрегированные данные за весь период.

#### 6.2.3. historical_price_extremes (Исторические экстремумы цен)

**Примечание:** Полное определение см. в разделе о материализованных представлениях выше. Это представление содержит исторические максимумы и минимумы цен для каждого инструмента.

---

## 7. Функции управления партициями

### 7.1. invest_utils.create_minute_candles_partition()

```sql
CREATE OR REPLACE FUNCTION invest_utils.create_minute_candles_partition(p_date DATE)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    partition_name TEXT;
    start_time TIMESTAMP WITH TIME ZONE;
    end_time TIMESTAMP WITH TIME ZONE;
BEGIN
    partition_name := 'minute_candles_' || TO_CHAR(p_date, 'YYYY_MM_DD');
    
    -- Определяем диапазон: от предыдущего дня 21:00 UTC до текущего дня 21:00 UTC
    start_time := (p_date - INTERVAL '1 day')::TIMESTAMP AT TIME ZONE 'Europe/Moscow' + INTERVAL '21 hours';
    end_time := p_date::TIMESTAMP AT TIME ZONE 'Europe/Moscow' + INTERVAL '21 hours';
    
    -- Проверяем существование партиции
    IF EXISTS (
        SELECT 1 FROM pg_class c
        JOIN pg_namespace n ON n.oid = c.relnamespace
        WHERE n.nspname = 'invest_candles' AND c.relname = partition_name
    ) THEN
        RETURN 'Partition ' || partition_name || ' already exists';
    END IF;
    
    -- Создаем партицию
    EXECUTE format('
        CREATE TABLE invest_candles.%I 
        PARTITION OF invest_candles.minute_candles
        FOR VALUES FROM (%L) TO (%L)',
        partition_name, start_time, end_time
    );
    
    -- Создаем индексы на партиции
    EXECUTE format('CREATE INDEX %I ON invest_candles.%I(time)',
        partition_name || '_time_idx', partition_name);
    EXECUTE format('CREATE INDEX %I ON invest_candles.%I(figi, time)',
        partition_name || '_figi_time_idx', partition_name);
    
    RETURN 'Partition ' || partition_name || ' created successfully';
END;
$$;
```

### 7.2. invest_utils.create_daily_candles_partition()

```sql
CREATE OR REPLACE FUNCTION invest_utils.create_daily_candles_partition(p_date DATE)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    partition_name TEXT;
    start_time TIMESTAMP WITH TIME ZONE;
    end_time TIMESTAMP WITH TIME ZONE;
    month_start DATE;
    month_end DATE;
BEGIN
    -- Определяем начало месяца
    month_start := DATE_TRUNC('month', p_date)::DATE;
    month_end := (DATE_TRUNC('month', p_date) + INTERVAL '1 month')::DATE;
    
    partition_name := 'daily_candles_' || TO_CHAR(month_start, 'YYYY_MM');
    
    -- Определяем диапазон: от последнего дня предыдущего месяца 21:00 UTC 
    -- до последнего дня текущего месяца 21:00 UTC
    start_time := (month_start - INTERVAL '1 day')::TIMESTAMP AT TIME ZONE 'Europe/Moscow' + INTERVAL '21 hours';
    end_time := (month_end - INTERVAL '1 day')::TIMESTAMP AT TIME ZONE 'Europe/Moscow' + INTERVAL '21 hours';
    
    -- Проверяем существование партиции
    IF EXISTS (
        SELECT 1 FROM pg_class c
        JOIN pg_namespace n ON n.oid = c.relnamespace
        WHERE n.nspname = 'invest_candles' AND c.relname = partition_name
    ) THEN
        RETURN 'Partition ' || partition_name || ' already exists';
    END IF;
    
    -- Создаем партицию
    EXECUTE format('
        CREATE TABLE invest_candles.%I 
        PARTITION OF invest_candles.daily_candles
        FOR VALUES FROM (%L) TO (%L)',
        partition_name, start_time, end_time
    );
    
    -- Создаем индексы на партиции
    EXECUTE format('CREATE INDEX %I ON invest_candles.%I(time)',
        partition_name || '_time_idx', partition_name);
    EXECUTE format('CREATE INDEX %I ON invest_candles.%I(figi, time)',
        partition_name || '_figi_time_idx', partition_name);
    
    RETURN 'Partition ' || partition_name || ' created successfully';
END;
$$;
```

### 7.3. invest_utils.create_close_prices_partition()

```sql
CREATE OR REPLACE FUNCTION invest_utils.create_close_prices_partition(p_date DATE)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    partition_name TEXT;
    month_start DATE;
    month_end DATE;
BEGIN
    month_start := DATE_TRUNC('month', p_date)::DATE;
    month_end := (DATE_TRUNC('month', p_date) + INTERVAL '1 month')::DATE;
    
    partition_name := 'close_prices_' || TO_CHAR(month_start, 'YYYY_MM');
    
    IF EXISTS (
        SELECT 1 FROM pg_class c
        JOIN pg_namespace n ON n.oid = c.relnamespace
        WHERE n.nspname = 'invest_prices' AND c.relname = partition_name
    ) THEN
        RETURN 'Partition ' || partition_name || ' already exists';
    END IF;
    
    EXECUTE format('
        CREATE TABLE invest_prices.%I 
        PARTITION OF invest_prices.close_prices
        FOR VALUES FROM (%L) TO (%L)',
        partition_name, month_start, month_end
    );
    
    RETURN 'Partition ' || partition_name || ' created successfully';
END;
$$;
```

### 7.4. invest.create_last_prices_daily_partition()

```sql
CREATE OR REPLACE FUNCTION invest.create_last_prices_daily_partition(p_date DATE)
RETURNS TEXT
LANGUAGE plpgsql
AS $$
DECLARE
    partition_name TEXT;
    start_time TIMESTAMP WITHOUT TIME ZONE;
    end_time TIMESTAMP WITHOUT TIME ZONE;
BEGIN
    partition_name := 'last_prices_' || TO_CHAR(p_date, 'YYYY_MM_DD');
    
    start_time := p_date::TIMESTAMP;
    end_time := (p_date + INTERVAL '1 day')::TIMESTAMP;
    
    IF EXISTS (
        SELECT 1 FROM pg_class c
        JOIN pg_namespace n ON n.oid = c.relnamespace
        WHERE n.nspname = 'invest' AND c.relname = partition_name
    ) THEN
        RETURN 'Partition ' || partition_name || ' already exists';
    END IF;
    
    EXECUTE format('
        CREATE TABLE invest.%I 
        PARTITION OF invest.last_prices
        FOR VALUES FROM (%L) TO (%L)',
        partition_name, start_time, end_time
    );
    
    RETURN 'Partition ' || partition_name || ' created successfully';
END;
$$;
```

---

## 8. Триггеры

### 8.1. invest_candles.calculate_daily_candle_statistics()

```sql
CREATE OR REPLACE FUNCTION invest_candles.calculate_daily_candle_statistics()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Вычисляем статистику свечи
    NEW.price_change := NEW.close - NEW.open;
    
    IF NEW.open > 0 THEN
        NEW.price_change_percent := (NEW.price_change / NEW.open) * 100;
    ELSE
        NEW.price_change_percent := NULL;
    END IF;
    
    IF NEW.close > NEW.open THEN
        NEW.candle_type := 'BULLISH';
    ELSIF NEW.close < NEW.open THEN
        NEW.candle_type := 'BEARISH';
    ELSE
        NEW.candle_type := 'DOJI';
    END IF;
    
    NEW.body_size := ABS(NEW.price_change);
    NEW.upper_shadow := NEW.high - GREATEST(NEW.open, NEW.close);
    NEW.lower_shadow := LEAST(NEW.open, NEW.close) - NEW.low;
    NEW.high_low_range := NEW.high - NEW.low;
    NEW.average_price := (NEW.high + NEW.low + NEW.close) / 3;
    
    RETURN NEW;
END;
$$;
```

### 8.2. invest_candles.calculate_minute_candle_statistics()

```sql
CREATE OR REPLACE FUNCTION invest_candles.calculate_minute_candle_statistics()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    -- Аналогично daily_candles, но average_price включает все 4 цены
    NEW.price_change := NEW.close - NEW.open;
    
    IF NEW.open > 0 THEN
        NEW.price_change_percent := (NEW.price_change / NEW.open) * 100;
    ELSE
        NEW.price_change_percent := NULL;
    END IF;
    
    IF NEW.close > NEW.open THEN
        NEW.candle_type := 'BULLISH';
    ELSIF NEW.close < NEW.open THEN
        NEW.candle_type := 'BEARISH';
    ELSE
        NEW.candle_type := 'DOJI';
    END IF;
    
    NEW.body_size := ABS(NEW.price_change);
    NEW.upper_shadow := NEW.high - GREATEST(NEW.open, NEW.close);
    NEW.lower_shadow := LEAST(NEW.open, NEW.close) - NEW.low;
    NEW.high_low_range := NEW.high - NEW.low;
    NEW.average_price := (NEW.high + NEW.low + NEW.open + NEW.close) / 4;
    
    RETURN NEW;
END;
$$;
```

### 8.3. invest_utils.calculate_duration_ms()

```sql
CREATE OR REPLACE FUNCTION invest_utils.calculate_duration_ms()
RETURNS TRIGGER
LANGUAGE plpgsql
AS $$
BEGIN
    IF NEW.end_time IS NOT NULL AND NEW.start_time IS NOT NULL THEN
        NEW.duration_ms := EXTRACT(EPOCH FROM (NEW.end_time - NEW.start_time)) * 1000;
    END IF;
    RETURN NEW;
END;
$$;
```

---

## 9. Примечания

### 9.1. Партиционирование

- **minute_candles**: Партиционирование по дням (RANGE по `time`)
- **daily_candles**: Партиционирование по месяцам (RANGE по `time`)
- **close_prices**: Партиционирование по месяцам (RANGE по `price_date`)
- **open_prices**: Партиционирование по месяцам (RANGE по `price_date`)
- **close_prices_evening_session**: Партиционирование по месяцам (RANGE по `price_date`)
- **last_prices**: Партиционирование по дням (RANGE по `time`)
- **trades**: Партиционирование по дням (RANGE по `time`)

### 9.2. Индексы

- Все партиционированные таблицы имеют индексы на каждой партиции отдельно
- Индексы на родительских таблицах создаются с `ONLY` и не наследуются партициями
- Каждая партиция имеет свои собственные индексы для оптимизации запросов

### 9.3. Временные зоны

- Все временные метки хранятся с учетом временной зоны `Europe/Moscow`
- Функции создания партиций учитывают временные зоны при определении границ партиций

### 9.4. Вычисляемые поля

- Поля `price_change`, `price_change_percent`, `candle_type`, `body_size`, `upper_shadow`, `lower_shadow`, `high_low_range`, `average_price` вычисляются автоматически через триггеры
- Поле `duration_ms` в `system_logs` вычисляется автоматически через триггер

### 9.5. Внешние ключи

- В базе данных не определены явные FOREIGN KEY constraints
- Связи реализованы на уровне приложения через использование FIGI

---

## 10. Примеры использования

### 10.1. Создание партиций на месяц вперед

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

### 10.2. Обновление материализованных представлений

```sql
REFRESH MATERIALIZED VIEW invest_views.daily_volume_aggregation;
REFRESH MATERIALIZED VIEW invest_views.history_volume_aggregation;
REFRESH MATERIALIZED VIEW invest_views.historical_price_extremes;
```

### 10.3. Очистка старых партиций

```sql
-- Удалить партиции старше года
SELECT invest_utils.cleanup_old_daily_partitions(365);
```
