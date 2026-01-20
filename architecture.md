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
    
    User --> API
    User --> LoaderApp
    
    SCHED --> TinkoffScanner["Tinkoff Invest API"]
    LoaderSched --> TinkoffLoader["Tinkoff Invest API"]
    
    SCHED --> PG
    LoaderSched --> PG
    API --> PG
    
    SCANNERS --> Alerts["Alerts/Notifications"]


