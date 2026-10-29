```mermaid
erDiagram
    USER {
        string id PK
        string name
        string email
    }

    STATION {
        string id PK
        string location_name
        int capacity
    }

    SCOOTER {
        string id PK
        string model
        int battery_level
        string station_id FK
    }

    RENTAL {
        string id PK
        string user_id FK
        string scooter_id FK
        datetime start_time
        datetime end_time
        decimal total_cost
    }

    STATION ||--o{ SCOOTER : hosts
    USER ||--o{ RENTAL : makes
    SCOOTER ||--o{ RENTAL : used_in