```mermaid
flowchart TB
    subgraph scanner["investmentDataScannerService"]
        API["Spring Boot API"]
        SCHED["Scheduler Data Scanner"]
        SCANNERS["Сканеры Market/Instruments"]
        PG["(PostgreSQL DB)"]
    end
    
    subgraph loader["InvestmentDataLoaderService"]
        LoaderApp["Spring Boot API"]
        LoaderSched["Scheduler Data Loader"]
    end

    subgraph orderService["InvestmentTradingService"]
        OrderAPI["Spring Boot API"]
        PG
        Alerts["Alerts/Notifications"]
    end
    
    User --> API
    User --> LoaderApp
    User --> OrderAPI
    
    SCHED --> TinkoffScanner["Tinkoff Invest API"]
    LoaderSched --> TinkoffLoader["Tinkoff Invest API"]
    OrderService --> TinkoffOrders["Tinkoff Invest API<br/>OrdersService"]
    
    SCHED --> PG
    LoaderSched --> PG
    API --> PG
    OrderController --> OrderService
    OrderService --> PG
    SCANNERS --> Alerts
    OrderService --> Alerts
    
    style orderService fill:#e1f5fe


