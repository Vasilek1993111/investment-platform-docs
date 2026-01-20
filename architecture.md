@startuml
!define RECTANGLE class
actor User
rectangle "DataLoaderService" as Svc {
  [Spring Boot] as App
  database PG
}
cloud "Tinkoff API"
User --> App
App --> PG
App --> "Tinkoff API"
@enduml
