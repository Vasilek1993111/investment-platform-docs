```mermaid
flowchart TD
    User[User/Developer] --> App[InvestmentDataLoaderService]
    App --> Tinkoff[Tinkoff Invest API]
    App --> PG[(PostgreSQL DB)]
    Tinkoff -.->|prices, candles| App
    App -.->|store data| PG
