```mermaid
C4Container
title DataLoaderService
Person(user, "User")
System_Boundary(svc, "Service") {
  Container(app, "Spring Boot API")
  ContainerDb(db, "PostgreSQL")
}
External(tinkoff, "Tinkoff API")
user --> app
app --> tinkoff : GET prices
app --> db : INSERT data
