# Data Encoding & Data Flow --- 15-Minute Business Guide

> **Goal:** Understand how business data is **packaged, moved, stored,
> and understood** by different systems --- without needing a technical
> background.

------------------------------------------------------------------------

## 1. The Big Picture

Imagine a food-delivery app. A customer places an order: **Chicken Rice
--- \$8**.

Inside the app, this exists as data in the computer's memory. If another
system needs it, the data must be packaged into a transferable format,
moved, and unpackaged again.

``` text
Customer App
    │
    │ ① ENCODE — Package the data
    ▼
Transferable Data
    │
    │ ② DATA FLOW — Move the data
    ▼
Another System
    │
    │ ③ DECODE — Unpack the data
    ▼
Usable data in memory
```

The module is mainly about two questions:

1.  **Data Encoding:** How should we package the data?
2.  **Data Flow:** How should we move the data between systems?

------------------------------------------------------------------------

# PART A --- DATA ENCODING

## 2. Why Does Data Need Encoding?

Programs work with data in **memory** as objects, arrays and values. To
save or transfer it, the data must be converted into a transferable
sequence of bytes.

``` text
IN MEMORY
Order object
     │
     │ Encode / Serialize / Marshal
     ▼
TRANSFERABLE OR STORABLE DATA
     │
     │ Decode / Deserialize / Unmarshal
     ▼
IN MEMORY
Order object
```

  -----------------------------------------------------------------------
  Term                                Layman meaning
  ----------------------------------- -----------------------------------
  **Encoding / Serialization /        Package in-memory data so it can be
  Marshalling**                       stored or transferred

  **Decoding / Deserialization /      Turn the packaged data back into
  Unmarshalling**                     usable in-memory data
  -----------------------------------------------------------------------

## 3. Textual vs Binary Encoding

``` text
ENCODING
│
├── TEXTUAL
│   ├── CSV
│   ├── JSON
│   └── XML
│
└── BINARY
    ├── Thrift
    ├── Protobuf
    ├── Avro
    ├── Parquet
    ├── ORC
    └── Arrow
```

**Textual formats** are relatively easy for humans to read and debug.
**Binary formats** are designed primarily for machines and can provide
smaller data size, faster serialization/deserialization, clearer data
types and stronger schema support.

> **Textual = easier for humans. Binary = optimized for machines.**

## 4. Schema and Schema Evolution

A **schema** describes the expected structure of data. Think of it as
the template for a business form.

``` text
ORDER SCHEMA

order_id   → integer
item       → text
price      → decimal
```

Business requirements change, so schemas change too:

``` text
VERSION 1                   VERSION 2

order_id                    order_id
item                        item
price                       price
                            delivery_type  ← NEW
```

This is **schema evolution**.

### Backward compatibility

**New code can understand old data.**

``` text
OLD DATA ─────► NEW CODE
```

> New software can look **backward**.

### Forward compatibility

**Old code can understand newer data.**

``` text
NEW DATA ─────► OLD CODE
```

> Old software can handle data coming from the **future**.

------------------------------------------------------------------------

# PART B --- BINARY FORMATS

## 5. The Six Formats Solve Different Problems

``` text
SERVICE COMMUNICATION
├── Thrift
└── Protobuf

DATA / EVENT EXCHANGE
└── Avro

ANALYTICAL STORAGE
├── Parquet
└── ORC

IN-MEMORY ANALYTICS
└── Arrow
```

  -----------------------------------------------------------------------
  Format                  Main business value     Simple preferred
                                                  scenario
  ----------------------- ----------------------- -----------------------
  **Apache Thrift**       Efficient               Python service
                          cross-language service  communicating with Java
                          communication           service

  **Protocol Buffers      Compact, fast           High-performance
  (Protobuf)**            structured messages     service communication

  **Apache Avro**         Efficient data exchange Large event/data
                          with evolving schemas   systems such as
                                                  Kafka-based pipelines

  **Apache Parquet**      Efficient column-based  Analyse years of sales
                          analytical storage      data

  **Apache ORC**          Efficient column-based  Large Hive-oriented
                          analytics, especially   analytics platform
                          associated with Hive    

  **Apache Arrow**        Fast in-memory          Share analytical data
                          analytics with reduced  already in memory
                          copying/serialization   
  -----------------------------------------------------------------------

One company can use several at once:

``` text
Food-delivery company

Microservices talking ─────────► Protobuf / Thrift
Order events moving around ────► Avro
Historical orders stored ──────► Parquet / ORC
Data being analysed in RAM ────► Arrow
```

They are complementary tools for different jobs.

------------------------------------------------------------------------

# PART C --- DATA FLOW

## 6. Once Data Is Packaged, How Does It Move?

``` text
                    DATA FLOW
                       │
          ┌────────────┼────────────┐
          ▼            ▼            ▼
      DATABASE     SERVICE CALL    MESSAGE
                       │              │
                 ┌─────┼─────┐   ┌───┴────┐
                REST GraphQL RPC Queue   Pub/Sub
                              │
                             gRPC
```

### Database

Provides **persistence and history**.

``` text
Order Service ── Save ──► DATABASE ── Retrieve later ──► Order Service
```

> **Database = store the information so we can use it later.**

## 7. Service Calls

A **service call** is when one application/service communicates with
another because it needs something done or returned.

> **Analogy: a phone call --- "I need something from you now."**

``` text
Order Service
      │
      │ "Calculate delivery price"
      ▼
Pricing Service
      │
      │ "$4.50"
      ▼
Order Service
```

### REST

Uses standard HTTP methods and commonly JSON/XML.

**Good fit:** web applications and public APIs.

### GraphQL

Lets the client request **precisely the data it needs**, reducing over-
or under-fetching.

**Good fit:** complex, client-driven data retrieval.

### RPC

**Remote Procedure Call** focuses on executing a function on another
service.

``` text
Order Service ── calculateDeliveryFee(...) ──► Pricing Service
```

**Good fit:** high-performance service-to-service communication.

## 8. Microservices

A **microservice** is a small, independent application responsible for a
specific business function within a larger system.

``` text
Order Service ───── Payment Service
      │
      ├──────────── Pricing Service
      │
      └──────────── Driver Service
```

These services need to communicate, which is why technologies such as
**REST, GraphQL, RPC/gRPC and messaging** exist.

## 9. gRPC + Protobuf

**gRPC** is a high-performance implementation of RPC. The slides
describe it as using **Protocol Buffers** for serialization and HTTP/2
for transport.

``` text
MICROSERVICE A
      │
      │ gRPC = communication approach
      │ Protobuf = encoding
      ▼
MICROSERVICE B
```

> **Data flow and data encoding are different decisions that work
> together.**

------------------------------------------------------------------------

# PART D --- MESSAGES

## 10. Service Call vs Message

Both allow systems to communicate, but the interaction is different.

### Service call

``` text
System A ───── request ─────► System B
         ◄──── response ─────
```

The sender typically needs an immediate interaction.

### Message

``` text
System A
   │
   │ "Order completed"
   ▼
MESSAGE BROKER
   │
   └────────► System B processes it independently
```

The sender **does not wait for a reply**. A broker sits between sender
and receiver and can temporarily store messages.

                  Service Call              Message
  --------------- ------------------------- ------------------
  Analogy         Phone call                Send a message
  Communication   More direct               Through a broker
  Sender waits?   Usually yes               No
  Style           Typically synchronous     Asynchronous
  Examples        REST, GraphQL, RPC/gRPC   Queue, Pub/Sub

## 11. Message Queue vs Publish-Subscribe

### Message Queue --- distribute work

``` text
Producer
   │
   ▼
QUEUE
│ Task 1
│ Task 2
│ Task 3
   │
   ▼
Consumer
```

Typically, each message is processed by **one consumer**, commonly in
FIFO order.

Good for task distribution, event handling and workload management.

Example from the slides: **RabbitMQ**.

> **Queue = "Someone please do this job."**

### Publish-Subscribe --- broadcast an event

``` text
             "Order completed"
                    │
                    ▼
                  TOPIC
          ┌─────────┼─────────┐
          ▼         ▼         ▼
      Analytics   Email    Loyalty
```

Multiple subscribers can receive the same message.

Good for real-time event broadcasting, notifications and log
aggregation.

Examples from the slides: **Kafka** and **Cloud Pub/Sub**.

> **Pub/Sub = "Something happened; everyone interested can react."**

------------------------------------------------------------------------

# 12. Put Everything Together

One customer order can involve nearly the entire ecosystem:

``` text
CUSTOMER APP
     │
     │ REST + JSON
     │ "Place my order"
     ▼
ORDER MICROSERVICE
     │
     │ gRPC + Protobuf
     │ "Process payment"
     ▼
PAYMENT MICROSERVICE
     │
     │ success
     ▼
ORDER MICROSERVICE
     │
     │ Publish "Order Created"
     ▼
MESSAGE BROKER / TOPIC
     │
     ├────────► Driver Service
     ├────────► Notification Service
     └────────► Analytics System
                       │
                       │ Historical data
                       ▼
                 Parquet / ORC
                       │
                       ▼
                    ANALYTICS
                       │
                     Arrow
              while data is in memory
```

These are **layers of the same ecosystem**, not isolated technologies.

------------------------------------------------------------------------

# 13. The 15-Minute Memory Map

``` text
BUSINESS DATA
     │
     ▼
① PACKAGE IT = ENCODING

   Human-readable
   └── JSON / XML / CSV

   Machine-efficient
   ├── Protobuf / Thrift → service communication
   ├── Avro              → evolving data/events
   ├── Parquet / ORC     → analytical storage
   └── Arrow             → in-memory analytics

     │
     ▼
② MOVE / USE IT = DATA FLOW

   ├── Database
   │      └── store for later
   │
   ├── Service Call
   │      ├── REST
   │      ├── GraphQL
   │      └── RPC → gRPC
   │
   └── Message
          ├── Queue   → distribute work
          └── Pub/Sub → broadcast events

     │
     ▼
③ UNPACKAGE IT = DECODING
```

## Final Mental Model

-   **Encoding** = How is the data packaged?
-   **Schema** = What should the package contain?
-   **Schema evolution** = How does that structure change over time?
-   **Compatibility** = Can old and new software/data still understand
    each other?
-   **Database** = Where can we persist data?
-   **Service call** = I need another system to interact with me now.
-   **Message** = I will send this and continue without waiting.
-   **Queue** = Someone should process this job.
-   **Pub/Sub** = Something happened; tell everyone interested.
-   **Microservices** = Small applications performing specific business
    functions.

> **One-sentence summary:** Modern systems encode business data into an
> appropriate format, move it between databases and services using
> synchronous or asynchronous communication, and decode it so another
> system can use it.
