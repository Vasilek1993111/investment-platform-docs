C4Context
title InvestmentDataLoaderService Context
Person(user, "User", "Developers")
System(svc, "DataLoaderService")
External_System(tinkoff, "Tinkoff API")
External_Db(pg, "PostgreSQL")
user )-- svc
svc --> tinkoff
svc --> pg

C4Container
title Containers
Person_Ext(user, "User")
System_Boundary(svc, "Service") {
  Container(app, "Spring Boot")
  ContainerDb(db, "PG DB")
}
External_System(tinkoff, "Tinkoff")
Rel(user, app, "API")
Rel(app, tinkoff, "Fetch data")
Rel(app, db, "Store")
