C4Context
    title InvestmentDataLoaderService System Context
    
    Person(developer, "Developer/Analyst", "Loads market data")
    System(dataLoader, "InvestmentDataLoaderService", "Loads/aggregates data from T-Invest to PG")
    System_Ext(tinkoff, "Tinkoff Invest API", "Market data source")
    System_Ext(postgres, "PostgreSQL DB", "Persistent storage")
    
    developer --> dataLoader : API calls
    dataLoader --> tinkoff : Fetch prices/candles
    dataLoader --> postgres : Store aggregated data

C4Container
    title InvestmentDataLoaderService Containers
    
    Person_Ext(developer, "Developer", "Manages service")
    
    System_Boundary(dataLoader, "InvestmentDataLoaderService") {
        Container(webapp, "Spring Boot App", "Java/Spring Boot", "REST API, Swagger")
        ContainerDb(pgdb, "PostgreSQL", "Relational DB", "prices, candles, aggregates")
        Container(scheduler, "Scheduler", "Spring @Scheduled", "Cron jobs for data load")
    }
    
    System_Ext(tinkoffAPI, "T-Invest API", "REST/Streaming", "Instruments, market data")
    
    developer --> webapp : HTTP /admin/load-prices
    webapp --> scheduler : Trigger loads
    scheduler --> tinkoffAPI : GET /market-data
    scheduler --> pgdb : INSERT/UPDATE data
    webapp --> pgdb : Queries

