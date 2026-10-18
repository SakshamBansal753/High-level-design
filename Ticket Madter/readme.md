## System Architecture

```mermaid
flowchart TD
    Client[Client] <--> CDN((CDN))
    CDN <--> API[API Gateway<br/>Load Balancer<br/>Routing<br/>Rate Limit]

    API <--> Search[Search]
    API <--> Event[Event CRUD Service]
    API <--> Queue[Virtual Waiting Queue]
    API <--> Booking[Booking]

    Search <--> ES[(Elasticsearch / OpenSearch)]
    Event <--> Redis[(Redis)]
    Event <--> DB[(PostgreSQL)]

    Queue <--> Booking

    Booking <--> DB
    Booking <--> Lock((Ticket Lock<br/>Redis, TTL))
    Booking <--> Stripe[Stripe]

    classDef service fill:#eaf2ff,stroke:#2457a7,color:#12356b,stroke-width:1.5px
    classDef storage fill:#e8f7ee,stroke:#24834a,color:#14532d,stroke-width:1.5px
    classDef external fill:#fff3df,stroke:#c47a12,color:#7c4600,stroke-width:1.5px

    class Client,API,Search,Event,Queue,Booking service
    class ES,Redis,DB,Lock storage
    class CDN,Stripe external
```

### Components

- **Client:** User-facing application.
- **CDN:** Caches and delivers static content.
- **API Gateway:** Handles load balancing, request routing, and rate limiting.
- **Search:** Finds events using Elasticsearch/OpenSearch.
- **Event CRUD Service:** Creates, reads, updates, and deletes event information.
- **Virtual Waiting Queue:** Manages traffic during high-demand ticket sales.
- **Booking:** Handles ticket reservations and booking workflows.
- **PostgreSQL:** Stores persistent event and booking data.
- **Redis:** Supports caching and fast data access.
- **Ticket Lock:** Uses Redis with a TTL (time to live) to temporarily reserve tickets and help prevent double booking.
- **Stripe:** Processes payments.

### Booking Flow

1. The client sends a request through the CDN and API Gateway.
2. The API Gateway routes the request to the appropriate service.
3. The waiting queue controls access during periods of high demand.
4. The Booking service checks ticket availability and acquires a temporary ticket lock.
5. Booking information is stored in PostgreSQL.
6. Stripe processes the payment.
7. The booking is confirmed and the temporary lock is released or expires.

