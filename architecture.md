C4Context
    title InvestmentDataLoaderService System Context
    
    Person(developer, "Developer/Analyst", "Loads market data")
    System(dataLoader, "InvestmentDataLoaderService", "Loads/aggregates data from T-Invest to PG")
    System_Ext(tinkoff, "Tinkoff Invest API", "Market data source")
    System_Ext(postgres, "PostgreSQL DB", "Persistent storage")
    
    developer --> dataLoader : API calls
    dataLoader --> tinkoff : Fetch prices/candles
    dataLoader --> postgres : Store aggregated data
