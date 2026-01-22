# ERD диаграмма базы данных InvestmentDataLoaderService

## Диаграмма сущностей и связей

```mermaid
erDiagram
    %% Справочные сущности (инструменты)
    SHARES {
        varchar figi PK
        varchar ticker
        varchar name
        varchar currency
        varchar exchange
        varchar sector
        varchar trading_status
        boolean short_enabled
        varchar asset_uid
        numeric min_price_increment
        integer lot
        timestamptz created_at
        timestamptz updated_at
    }
    
    FUTURES {
        varchar figi PK
        varchar ticker
        varchar asset_type
        varchar basic_asset
        varchar currency
        varchar exchange
        boolean short_enabled
        timestamp expiration_date
        numeric min_price_increment
        integer lot
        numeric basic_asset_size
        timestamp created_at
        timestamp updated_at
    }
    
    INDICATIVES {
        varchar figi PK
        varchar ticker
        varchar name
        varchar currency
        varchar exchange
        varchar class_code
        varchar uid
        boolean sell_available_flag
        boolean buy_available_flag
        timestamptz created_at
        timestamptz updated_at
    }
    
    %% Фундаментальные данные
    DIVIDENDS {
        bigint id PK
        varchar figi FK
        date declared_date
        date record_date
        date payment_date
        numeric dividend_value
        varchar currency
        varchar dividend_type
        timestamptz created_at
        timestamptz updated_at
    }
    
    FUNDAMENTALS {
        bigint id PK
        varchar figi FK
        varchar asset_uid
        varchar domicile_indicator_code
        varchar currency
        numeric dividend_yield_daily_ttm
        numeric pe_ratio_ttm
        numeric eps_ttm
        numeric revenue_ttm
        numeric market_capitalization
        timestamptz created_at
        timestamptz updated_at
    }
    
    %% Ценовые данные
    CLOSE_PRICES {
        varchar figi PK,FK
        date price_date PK
        numeric close_price
        varchar currency
        varchar exchange
        varchar instrument_type
        timestamptz created_at
        timestamptz updated_at
    }
    
    OPEN_PRICES {
        varchar figi PK,FK
        date price_date PK
        numeric open_price
        varchar currency
        varchar exchange
        varchar instrument_type
        timestamptz created_at
        timestamptz updated_at
    }
    
    LAST_PRICES {
        varchar figi PK,FK
        timestamp time PK
        numeric price
        varchar currency
        varchar exchange
    }
    
    %% Свечные данные
    DAILY_CANDLES {
        varchar figi PK,FK
        timestamptz time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
        boolean is_complete
        numeric price_change
        numeric price_change_percent
        varchar candle_type
        numeric body_size
        numeric upper_shadow
        numeric lower_shadow
        timestamptz created_at
        timestamptz updated_at
    }
    
    MINUTE_CANDLES {
        varchar figi PK,FK
        timestamptz time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
        boolean is_complete
        numeric price_change
        numeric price_change_percent
        varchar candle_type
        numeric body_size
        timestamptz created_at
        timestamptz updated_at
    }
    
    %% Торговые данные
    TRADES {
        varchar figi PK,FK
        timestamp time PK
        numeric price
        bigint quantity
        varchar direction
        varchar trade_direction
        varchar trade_source
        varchar currency
        varchar exchange
    }
    
    %% Аналитические данные
    CANDLE_PATTERN_ANALYSIS {
        bigint id PK
        varchar figi FK
        date analysis_date
        date pattern_start_date
        date pattern_end_date
        varchar candle_type
        integer consecutive_days
        bigint avg_volume
        numeric avg_price_change
        numeric total_price_change
        char strategy_applicable
        varchar instrument_name
        varchar instrument_ticker
        text instrument_type
        timestamptz created_at
    }
    
    BACKTEST_RESULTS {
        bigint id PK
        bigint pattern_analysis_id FK
        varchar figi FK
        date analysis_date
        varchar entry_type
        numeric entry_price
        numeric amount
        numeric stop_loss_percent
        numeric take_profit_percent
        numeric stop_loss_price
        numeric take_profit_price
        varchar exit_type
        varchar result
        numeric exit_price
        timestamptz exit_time
        numeric profit_loss
        numeric profit_loss_percent
        integer duration_minutes
        varchar instrument_name
        varchar instrument_ticker
        text instrument_type
        timestamptz created_at
    }
    
    %% Служебные данные
    SYSTEM_LOGS {
        bigint id PK
        varchar task_id
        varchar endpoint
        varchar method
        varchar status
        text message
        timestamptz start_time
        timestamptz end_time
        bigint duration_ms
        timestamptz created_at
    }
    
    %% Связи через FIGI (логические, не через FOREIGN KEY)
    SHARES ||--o{ CLOSE_PRICES : "has"
    SHARES ||--o{ OPEN_PRICES : "has"
    SHARES ||--o{ LAST_PRICES : "has"
    SHARES ||--o{ DAILY_CANDLES : "has"
    SHARES ||--o{ MINUTE_CANDLES : "has"
    SHARES ||--o{ TRADES : "has"
    SHARES ||--o{ DIVIDENDS : "has"
    SHARES ||--|| FUNDAMENTALS : "has"
    SHARES ||--o{ CANDLE_PATTERN_ANALYSIS : "has"
    SHARES ||--o{ BACKTEST_RESULTS : "has"
    
    FUTURES ||--o{ CLOSE_PRICES : "has"
    FUTURES ||--o{ OPEN_PRICES : "has"
    FUTURES ||--o{ LAST_PRICES : "has"
    FUTURES ||--o{ DAILY_CANDLES : "has"
    FUTURES ||--o{ MINUTE_CANDLES : "has"
    FUTURES ||--o{ TRADES : "has"
    FUTURES ||--o{ DIVIDENDS : "has"
    FUTURES ||--|| FUNDAMENTALS : "has"
    FUTURES ||--o{ CANDLE_PATTERN_ANALYSIS : "has"
    FUTURES ||--o{ BACKTEST_RESULTS : "has"
    
    INDICATIVES ||--o{ CLOSE_PRICES : "has"
    INDICATIVES ||--o{ OPEN_PRICES : "has"
    INDICATIVES ||--o{ LAST_PRICES : "has"
    INDICATIVES ||--o{ DAILY_CANDLES : "has"
    INDICATIVES ||--o{ MINUTE_CANDLES : "has"
    INDICATIVES ||--o{ TRADES : "has"
    INDICATIVES ||--o{ DIVIDENDS : "has"
    INDICATIVES ||--|| FUNDAMENTALS : "has"
    INDICATIVES ||--o{ CANDLE_PATTERN_ANALYSIS : "has"
    INDICATIVES ||--o{ BACKTEST_RESULTS : "has"
    
    %% Связи между аналитическими сущностями
    CANDLE_PATTERN_ANALYSIS ||--o{ BACKTEST_RESULTS : "generates"
```

## Упрощенная диаграмма (основные сущности)

```mermaid
erDiagram
    INSTRUMENT {
        varchar figi PK
        varchar type
        varchar ticker
        varchar name
    }
    
    DIVIDENDS {
        bigint id PK
        varchar figi FK
        date record_date
        numeric dividend_value
    }
    
    FUNDAMENTALS {
        bigint id PK
        varchar figi FK
        numeric pe_ratio_ttm
        numeric eps_ttm
        numeric market_capitalization
    }
    
    CLOSE_PRICES {
        varchar figi PK,FK
        date price_date PK
        numeric close_price
    }
    
    OPEN_PRICES {
        varchar figi PK,FK
        date price_date PK
        numeric open_price
    }
    
    DAILY_CANDLES {
        varchar figi PK,FK
        timestamptz time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
    }
    
    MINUTE_CANDLES {
        varchar figi PK,FK
        timestamptz time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
    }
    
    TRADES {
        varchar figi PK,FK
        timestamp time PK
        numeric price
        bigint quantity
    }
    
    CANDLE_PATTERN_ANALYSIS {
        bigint id PK
        varchar figi FK
        date analysis_date
        integer consecutive_days
    }
    
    BACKTEST_RESULTS {
        bigint id PK
        bigint pattern_analysis_id FK
        varchar figi FK
        varchar entry_type
        numeric entry_price
        numeric profit_loss
    }
    
    INSTRUMENT ||--o{ CLOSE_PRICES : "has"
    INSTRUMENT ||--o{ OPEN_PRICES : "has"
    INSTRUMENT ||--o{ DAILY_CANDLES : "has"
    INSTRUMENT ||--o{ MINUTE_CANDLES : "has"
    INSTRUMENT ||--o{ TRADES : "has"
    INSTRUMENT ||--o{ DIVIDENDS : "has"
    INSTRUMENT ||--|| FUNDAMENTALS : "has"
    INSTRUMENT ||--o{ CANDLE_PATTERN_ANALYSIS : "has"
    INSTRUMENT ||--o{ BACKTEST_RESULTS : "has"
    CANDLE_PATTERN_ANALYSIS ||--o{ BACKTEST_RESULTS : "generates"
```

## Диаграмма по категориям данных

```mermaid
erDiagram
    %% Категория: Справочники
    SHARES {
        varchar figi PK
        varchar ticker
        varchar name
    }
    
    FUTURES {
        varchar figi PK
        varchar ticker
        varchar basic_asset
    }
    
    INDICATIVES {
        varchar figi PK
        varchar ticker
        varchar name
    }
    
    %% Категория: Фундаментальные данные
    FUNDAMENTALS {
        bigint id PK
        varchar figi FK
        numeric pe_ratio_ttm
        numeric eps_ttm
    }
    
    DIVIDENDS {
        bigint id PK
        varchar figi FK
        date record_date
        numeric dividend_value
    }
    
    %% Категория: Ценовые данные
    CLOSE_PRICES {
        varchar figi PK,FK
        date price_date PK
        numeric close_price
    }
    
    OPEN_PRICES {
        varchar figi PK,FK
        date price_date PK
        numeric open_price
    }
    
    LAST_PRICES {
        varchar figi PK,FK
        timestamp time PK
        numeric price
    }
    
    %% Категория: Свечные данные
    DAILY_CANDLES {
        varchar figi PK,FK
        timestamptz time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
    }
    
    MINUTE_CANDLES {
        varchar figi PK,FK
        timestamptz time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
    }
    
    %% Категория: Торговые данные
    TRADES {
        varchar figi PK,FK
        timestamp time PK
        numeric price
        bigint quantity
    }
    
    %% Категория: Аналитика
    CANDLE_PATTERN_ANALYSIS {
        bigint id PK
        varchar figi FK
        date analysis_date
        integer consecutive_days
    }
    
    BACKTEST_RESULTS {
        bigint id PK
        bigint pattern_analysis_id FK
        varchar figi FK
        numeric profit_loss
    }
    
    %% Связи: Справочники -> Ценовые данные
    SHARES ||--o{ CLOSE_PRICES : "figi"
    SHARES ||--o{ OPEN_PRICES : "figi"
    SHARES ||--o{ LAST_PRICES : "figi"
    FUTURES ||--o{ CLOSE_PRICES : "figi"
    FUTURES ||--o{ OPEN_PRICES : "figi"
    FUTURES ||--o{ LAST_PRICES : "figi"
    INDICATIVES ||--o{ CLOSE_PRICES : "figi"
    INDICATIVES ||--o{ OPEN_PRICES : "figi"
    INDICATIVES ||--o{ LAST_PRICES : "figi"
    
    %% Связи: Справочники -> Свечные данные
    SHARES ||--o{ DAILY_CANDLES : "figi"
    SHARES ||--o{ MINUTE_CANDLES : "figi"
    FUTURES ||--o{ DAILY_CANDLES : "figi"
    FUTURES ||--o{ MINUTE_CANDLES : "figi"
    INDICATIVES ||--o{ DAILY_CANDLES : "figi"
    INDICATIVES ||--o{ MINUTE_CANDLES : "figi"
    
    %% Связи: Справочники -> Торговые данные
    SHARES ||--o{ TRADES : "figi"
    FUTURES ||--o{ TRADES : "figi"
    INDICATIVES ||--o{ TRADES : "figi"
    
    %% Связи: Справочники -> Фундаментальные данные
    SHARES ||--o{ DIVIDENDS : "figi"
    SHARES ||--|| FUNDAMENTALS : "figi"
    FUTURES ||--o{ DIVIDENDS : "figi"
    FUTURES ||--|| FUNDAMENTALS : "figi"
    INDICATIVES ||--o{ DIVIDENDS : "figi"
    INDICATIVES ||--|| FUNDAMENTALS : "figi"
    
    %% Связи: Справочники -> Аналитика
    SHARES ||--o{ CANDLE_PATTERN_ANALYSIS : "figi"
    SHARES ||--o{ BACKTEST_RESULTS : "figi"
    FUTURES ||--o{ CANDLE_PATTERN_ANALYSIS : "figi"
    FUTURES ||--o{ BACKTEST_RESULTS : "figi"
    INDICATIVES ||--o{ CANDLE_PATTERN_ANALYSIS : "figi"
    INDICATIVES ||--o{ BACKTEST_RESULTS : "figi"
    
    %% Связи: Аналитика
    CANDLE_PATTERN_ANALYSIS ||--o{ BACKTEST_RESULTS : "pattern_analysis_id"
```

## Легенда

### Обозначения связей

- `||--o{` - Один ко многим (1:N), где "много" может быть нулем
- `||--||` - Один к одному (1:1), где связь обязательна
- `}o--o{` - Многие ко многим (N:M)

### Обозначения ключей

- `PK` - Primary Key (Первичный ключ)
- `FK` - Foreign Key (Внешний ключ, логический)
- Составные ключи обозначены как `PK,FK` для полей, входящих в составной ключ

### Примечания

1. **Логические связи:** В базе данных не определены явные FOREIGN KEY constraints. Все связи реализованы на уровне приложения через использование FIGI.

2. **Партиционирование:** Таблицы `close_prices`, `open_prices`, `daily_candles`, `minute_candles`, `last_prices`, `trades` партиционированы по времени, что не отражено на диаграмме.

3. **Упрощения:** 
   - На полной диаграмме показаны только ключевые атрибуты
   - Многие вычисляемые поля (например, `price_change`, `candle_type`) не показаны
   - Служебные поля (`created_at`, `updated_at`) показаны только в первой диаграмме

4. **Типы инструментов:** SHARES, FUTURES, INDICATIVES являются разными типами инструментов, но имеют одинаковые связи с другими сущностями через FIGI.
