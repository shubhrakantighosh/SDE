# 📨 AWS SQS - Key Concepts

## 1️⃣ Types of Queues

### Standard Queue
- High throughput.
- Allows multiple messages to be sent simultaneously.
- **Ordering is not guaranteed**.
- Best for scenarios where the exact order of processing is not critical.

### FIFO Queue (First-In-First-Out)
- Guarantees **strict message ordering**.
- Prevents **duplicate messages**.
- Use when order and exactly-once processing are required.

---

## 2️⃣ Core Concepts

### Delay
- Delay before a message becomes visible in the queue.
- **Default:** `0 seconds`
- **Maximum:** `15 minutes`

### Consumer
- Application/component that **retrieves and processes** messages from the queue.

### Producer
- Application/component that **sends** messages to the queue.

### Message Capacity
- Maximum message size is **256 KB**.

### Purge
- Deletes **all messages** from the queue.
- ⚠️ **Cannot delete individual messages.**

### Polling Types
- **Short Polling:**  
  Immediately returns messages if available, even if the queue is empty.  
  More API calls, more cost.

- **Long Polling:**  
  Waits for a message before responding.  
  Reduces cost and unnecessary API requests.  
  Configurable up to **20 seconds**.

---

## 3️⃣ Visibility Timeout

- Duration for which a message becomes **invisible** after being consumed.
- If not deleted during this time, it becomes visible again and can be reprocessed.
- Prevents **duplicate processing** if handled properly.

- **Default:** `30 seconds`
- **Maximum:** `12 hours`

> 🔁 Use `ChangeMessageVisibility` to extend timeout if processing takes longer.

---

## 4️⃣ Concurrency and Workers

- **Concurrency:** Number of consumers polling the queue at the same time.
- **Per Concurrency Worker:** Number of messages each worker can process concurrently.

### Example:
- `Concurrency = 2`, `Per Concurrency Worker = 2`  
  ➡️ Total 4 workers processing messages in parallel.

---

## 5️⃣ FIFO Queue & MessageGroupId

- **MessageGroupId** enables parallel processing in FIFO queues.
- Messages with the **same MessageGroupId** are processed **in order** (within a single partition).
- Messages with **different MessageGroupIds** can be processed **concurrently**.
- Proper use of GroupId can **greatly improve throughput** by increasing parallelism.

---

## 6️⃣ Dead Letter Queue (DLQ)

- Stores messages that **fail to process** after a specified number of retries.
- Helps with **debugging** and monitoring failed messages.
- If DLQ is **not set**, SQS retries **indefinitely**.

---

## 7️⃣ Additional Key Points

### ✅ Message Retention Period
- Duration a message is retained in the queue **whether processed or not**.
- **Default:** `4 days`
- **Maximum:** `14 days`

### ✅ Batching
- **Send/Receive up to 10 messages** in a single API call.
- Reduces **cost** and improves **throughput**.

### ✅ Maximum In-Flight Messages
- Messages being processed but **not yet deleted**.
- **Standard Queue:** `120,000` messages
- **FIFO Queue:** `20,000` messages

> ⚠️ New messages will not be delivered if this limit is reached until others are deleted or timeout.

### ✅ Deduplication in FIFO Queues
- Automatically removes duplicates within a **5-minute deduplication window**.
- Supports **content-based deduplication** and **explicit `MessageDeduplicationId`**.

### ✅ Throughput Limits (TPS)
- **Standard Queue:**
  - Nearly unlimited TPS.
- **FIFO Queue:**
  - `300 TPS` (without batching)
  - `3,000 TPS` (with batching)

---

## 8️⃣ Advanced Topics

### 🔁 Message Lifecycle
1. Message is sent to the queue.
2. Becomes visible to consumers.
3. Consumer processes the message.
4. Message is **deleted** after successful processing.
5. If not deleted in time, it becomes visible again.

### 🧠 Idempotency
- Make sure your message processing logic is **idempotent** (safe to process multiple times) to handle retries or duplicates.

### 📊 Monitoring & Metrics
Use **CloudWatch** to monitor:
- Number of messages sent/received/deleted
- Messages in-flight
- Approximate queue length
- DLQ metrics

### 🔐 Security
- Use **IAM policies** to control access to queues.
- Enable **encryption (SSE)** for sensitive data.

---

## 9️⃣ Frequently Asked Questions (FAQs)

### ❓ What happens if my processing takes longer than the Visibility Timeout?
Another consumer may pick up the same message, leading to **duplicate processing**.

### ❓ Can I control the number of partitions in a FIFO queue?
No. **Partitions are managed automatically** based on `MessageGroupId`. More distinct group IDs = more concurrency.

### ❓ Is Webhook-based delivery possible in SQS?
No. SQS is **pull-based** (consumer polls messages). For **push-based**, consider **SNS** (Simple Notification Service).

---
