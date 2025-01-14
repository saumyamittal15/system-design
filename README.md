# **Credit Card System Design**

This repository contains the high-level design (HLD) and detailed architecture for a scalable, reliable, and low-latency **Credit Card System**. The system is designed to handle core functionalities like credit card applications, transaction management, payment processing, and reporting while meeting strict non-functional requirements (NFRs) such as scalability, consistency, reliability, and availability.

---

## **Table of Contents**
1. [Overview](#overview)
2. [Functional Requirements (FRs)](#functional-requirements-frs)
3. [Non-Functional Requirements (NFRs)](#non-functional-requirements-nfrs)
4. [APIs](#apis)
5. [Entities and Database Design](#entities-and-database-design)
6. [High-Level Design (HLD)](#high-level-design-hld)
7. [Deep Dives](#deep-dives)
8. [Excalidraw Whiteboard](#excalidraw-whiteboard)

---

## **Overview**

The credit card system is designed to:
1. Allow users to apply for credit cards.
2. Enable users to create and access bank accounts linked to their cards.
3. Handle real-time posting of transactions using third-party APIs.
4. Process payments securely via external payment gateways.
5. Provide daily, weekly, and monthly reports of user activity.

The system is built using microservices architecture with clear separation of concerns for scalability, maintainability, and fault tolerance.

---

## **Functional Requirements (FRs)**

1. **Apply for Credit Card**:
   - Users can submit an application for a credit card.
   - Applications are verified asynchronously, and users are notified of success or failure.

2. **Create and Access Bank Account**:
   - Users can create a bank account linked to their credit card.
   - They can view account details like balance and transaction history.

3. **Post Transactions**:
   - Handle real-time posting of transactions using third-party APIs.
   - Update account balances after successful transactions.

4. **Process Payments**:
   - Users can make payments to repay their credit card bills.
   - Payments are processed securely via external payment gateways.

5. **View Reports**:
   - Generate daily, weekly, and monthly reports of user activity.
   - Provide aggregated statistics like total spending by category or merchant.

---

## **Non-Functional Requirements (NFRs)**

1. **Scalability**:
   - Support up to 300 million users in the US region.
   - Handle high transaction volumes with low latency (<1 second).

2. **Consistency**:
   - Ensure ACID compliance for critical operations like payments and balance updates.

3. **Reliability**:
   - Prevent overcharging or lost payment information.
   - Reconciliation jobs ensure data integrity.

4. **Availability**:
   - System downtime should not exceed 10 hours per year (<99.9% availability).

5. **Security**:
   - Protect sensitive user data with HTTPS, token-based authentication, and encryption.
   - Detect and reject suspicious activities (e.g., fraud).

---

## **APIs**

### 1. Credit Card Application
- **`POST /credit-card/apply`**
  - Input: User details (name, SSN, income).
  - Output: Application ID, success/failure notification.

### 2. Create/Access Bank Account
- **`POST /account/create`**
  - Input: User ID, initial deposit amount.
  - Output: Account ID.
- **`GET /account/{accountId}`**
  - Input: Account ID.
  - Output: Account details (balance, linked cards).

### 3. Post Transactions
- **`POST /transaction/post`**
  - Input: Account ID, transaction details (amount, merchant).
  - Output: Transaction ID, success/failure status.

### 4. Process Payments
- **`POST /payment/process`**
  - Input: User ID, payment amount.
  - Output: Payment ID, success/failure status.

### 5. View Reports
- **`GET /report/{userId}?type=daily|weekly|monthly`**
  - Input: User ID, report type.
  - Output: Aggregated user activity report.

---

## **Entities and Database Design**

### Core Entities

#### Users Table
| Field         | Type        | Description                 |
|---------------|-------------|-----------------------------|
| userId        | UUID        | Unique identifier for users |
| name          | String      | User's name                |
| email         | String      | User's email               |

#### Accounts Table
| Field         | Type        | Description                 |
|---------------|-------------|-----------------------------|
| accountId     | UUID        | Unique identifier for accounts |
| userId        | UUID        | Foreign key to Users table |
| balance       | Decimal     | Current account balance     |

#### Transactions Table
| Field         | Type        | Description                 |
|---------------|-------------|-----------------------------|
| transactionId | UUID        | Unique transaction ID       |
| accountId     | UUID        | Foreign key to Accounts table |
| amount        | Decimal     | Transaction amount          |
| timestamp     | Timestamp   | Time of transaction         |

#### Payments Table
| Field         | Type        | Description                 |
|---------------|-------------|-----------------------------|
| paymentId     | UUID        | Unique payment ID           |
| userId        | UUID        | Foreign key to Users table  |
| amount        | Decimal     | Payment amount              |
| status        | Enum        | WAITING/SUCCESS/FAILED      |

---

## **High-Level Design (HLD)**

The system is built using microservices architecture with the following components:

1. **Application Management Service**:
    Handles credit card applications and verification workflows.
    DB: MySQL (ACID-compliant).

2. **Account Management Service**:
    Manages accounts and balances.
    DB: MySQL sharded by `userId`.

3. **Transaction Management Service**:
    Posts transactions using third-party APIs and updates balances.
    DB: DynamoDB for fast writes by timestamp.

4. **Payment Processing Service**:
    Handles secure payment processing via Stripe API.
    DB: MySQL for payment orders; Kafka for event queues.

5. **Reporting Service**:
    Aggregates transaction data into reports.
    DB: Cassandra or MongoDB for storing report documents.

6. **API Gateway**:
    Routes all client requests to appropriate microservices.

---

## **Deep Dives**

### Caching for Low Latency
- Cache frequently accessed data like user profiles, account balances, recent transactions, and precomputed reports using Redis or Memcached.
- Use short TTLs for dynamic data (e.g., balances) and long TTLs for static data (e.g., reports).

### Scalability
- Use sharded MySQL databases for accounts and payments tables to handle high write loads efficiently.
- Use Kafka as an event streaming platform for asynchronous processing of payments and reporting tasks.

### Consistency
- ACID compliance ensured using MySQL for critical operations like payments and balances.
- Idempotency keys ensure no duplicate transactions or payments are processed.

### Reliability
- Dead-letter queues handle failed requests for retrying later.
- Nightly reconciliation jobs compare event logs with database values to detect inconsistencies.

### Fault Tolerance
- Load balancers distribute traffic across multiple instances of each microservice to avoid single points of failure.
- All services are stateless; state is managed in databases or message queues.

---

## **Excalidraw Whiteboard**

The high-level architecture diagram can be viewed on Excalidraw:

[View Credit Card System Design on Excalidraw](https://excalidraw.com/#json=-GnwuCVfxQY69RQNYD6dZ,vt3IDqoOWPMSal1mjJi44Q)



---

## Conclusion

This design ensures scalability, reliability, consistency, low latency (<1 second), and security while meeting all functional requirements effectively:

1. Modular microservices architecture ensures separation of concerns between components like TMS, PPS, Reporting Service, etc.
2. Caching strategies reduce latency by serving frequently accessed data from in-memory stores like Redis/Memcached.
3. Kafka-based event-driven architecture ensures scalability for asynchronous workflows like reporting and payment execution.

Feel free to explore the repository further or contribute improvements!
