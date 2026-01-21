erDiagram
    instruments {
        varchar figi PK
        varchar ticker
        varchar name
        varchar type
        int lot_size
        varchar currency
    }
    
    exchanges {
        varchar exchange PK
        varchar name
        varchar mic
    }

    daily_candles {
        varchar figi FK
        date time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
    }
    
    minute_candles {
        varchar figi FK
        timestamp time PK
        numeric open
        numeric high
        numeric low
        numeric close
        bigint volume
    }

    last_prices {
        varchar figi PK
        timestamp time PK
        numeric price
        varchar currency
        varchar exchange FK
    }
    
    evening_close {
        varchar figi PK
        date session_date PK
        numeric evening_price
    }

    system_logs {
        varchar task_id PK
        varchar endpoint
        varchar status
        text message
        timestamp created_at
    }

    today_volume_view {
        varchar figi
        date trade_date
        numeric total_volume
        numeric morning_volume
        numeric evening_volume
    }

    mt5_trades {
        int ticket PK
        varchar symbol
        timestamp time
        varchar type
        numeric volume
        numeric price
    }

    %% Связи
    instruments ||--o{ daily_candles : содержит
    instruments ||--o{ minute_candles : содержит
    instruments ||--o{ last_prices : цены
    exchanges ||--o{ last_prices : биржа
    minute_candles ||--o{ today_volume_view : агрегация
