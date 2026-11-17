```mermaid
erDiagram
    USER {
        string id PK
        string name
        string email UK
    }

    STATION {
        string id PK
        string location_name
        int capacity "greater than 0"
    }

    SCOOTER {
        string id PK
        string model
        int battery_level "0 to 100"
        string station_id FK "nullable"
    }

    RENTAL {
        string id PK
        string user_id FK
        string scooter_id FK
        datetime start_time
        datetime end_time "nullable"
        decimal total_cost
    }

    STATION |o--o{ SCOOTER : hosts
    USER ||--o{ RENTAL : makes
    SCOOTER ||--o{ RENTAL : used_in