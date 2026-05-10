# AWS SNS

---

# SECTION A: SNS (Simple Notification Service)

## Part 1: What Problem Does SNS Solve?

*"Your production server just went down. CloudWatch detected it. Now what? How does the on-call engineer get woken up at 3 AM? How does the team Slack channel get notified? How does the incident management system create a ticket?"*

*"You need a way to BROADCAST a message to multiple places at once. That's SNS."*

###  The Radio Station Analogy

*"SNS works like a radio station:"*

```
CloudWatch Alarm triggers
        │
        ▼
  ┌──────────────┐
  │  SNS TOPIC   │  ← The radio station (broadcasts to all listeners)
  │ "alerts"     │
  └──────┬───────┘
         │
    ┌────┼────────────────┐
    │    │                │
    ▼    ▼                ▼
  📧    📱              🔗
 Email  SMS         Lambda/SQS/HTTP
 (Dev)  (On-call)    (Auto-remediation)
```

*"One event, many listeners. You don't send individual emails and texts. You publish ONE message to a topic, and every subscriber gets it simultaneously."*

---

## Part 2: SNS Core Concepts

###  Topics

*"A Topic is a named channel. Think of it as a mailing list."*

| Feature | Detail |
|---------|--------|
| **Name** | Any string (e.g., `production-alerts`, `order-notifications`) |
| **Type** | Standard (default) or FIFO (ordered) |
| **ARN** | `arn:aws:sns:us-east-1:123456789012:production-alerts` |
| **Max topics** | 100,000 per account |

###  Standard vs FIFO Topics

| Feature | Standard Topic | FIFO Topic |
|---------|---------------|------------|
| **Ordering** | Best-effort (no guarantee) | Strict order |
| **Deduplication** | No (you might get duplicates) | Yes (content or ID-based) |
| **Throughput** | Unlimited | 300 msg/sec (batching: 3,000) |
| **Subscribers** | All types | Only SQS FIFO queues |
| **Use case** | Alerts, notifications | Financial transactions, order processing |

*"Standard is what you'll use 95% of the time. FIFO is for when ORDER MATTERS — like processing bank transactions in sequence."*

###  Publishers (Who sends messages?)

*"Anyone can publish to a topic if they have permission:"*

| Publisher | Example |
|-----------|---------|
| **CloudWatch Alarms** | CPU > 80% → publish to alerts topic |
| **S3 Events** | New file uploaded → publish to processing topic |
| **Lambda** | Function publishes result to topic |
| **EventBridge** | EC2 stopped → publish to ops topic |
| **Your Application** | User signs up → publish to welcome-email topic |
| **AWS CLI** | `aws sns publish --topic-arn ... --message "Hello"` |
| **CloudFormation** | Stack creation complete → notify |
| **Budgets** | Cost exceeded $100 → alert |
| **CodePipeline** | Deployment failed → notify team |

###  Subscribers (Who receives messages?)

| Subscriber | What It Does | Use Case |
|------------|-------------|----------|
| **Email** | Sends plain text or JSON email | Human notifications |
| **Email-JSON** | Sends raw JSON payload | Parsing by email rules |
| **SMS** | Text message to phone | On-call alerts |
| **HTTP/HTTPS** | POST to a URL | Webhooks (Slack, PagerDuty) |
| **Lambda** | Invokes a function | Auto-remediation |
| **SQS** | Puts message in a queue | Async processing |
| **Kinesis Firehose** | Streams to S3/Redshift | Log archival |
| **Platform Push** | iOS/Android push notifications | Mobile apps |

###  Fan-Out Pattern

*"The most important SNS pattern. One message → many consumers."*

```
[Order Placed]
      │
      ▼
 SNS Topic: "new-order"
      │
      ├──► SQS: payment-queue      → Payment service processes it
      ├──► SQS: inventory-queue    → Inventory service reserves stock
      ├──► SQS: email-queue        → Email service sends confirmation
      ├──► Lambda: analytics        → Records in analytics database
      └──► HTTP: partner-api        → Notifies shipping partner
```

*"Without SNS, your order service would have to call 5 different services directly — tightly coupled, slow, fragile. With SNS fan-out, the order service publishes ONE message and walks away. Each subscriber processes independently, at their own pace."*

---

## Part 3: Message Filtering

###  The Problem

*"In the fan-out example, every subscriber gets EVERY message. But what if the payment service only cares about orders over $100? And the email service only cares about new customers?"*

###  Subscription Filter Policies

*"You can attach a filter policy to each subscription. The subscriber ONLY receives messages that match the filter."*

```json
// Message published to "orders" topic:
{
  "order_id": "ORD-123",
  "amount": 250,
  "customer_type": "new",
  "category": "electronics"
}

// Subscriber: high-value-orders queue
// Filter Policy:
{
  "amount": [{"numeric": [">=", 100]}]
}
// → Receives this message ✅ (250 >= 100)

// Subscriber: new-customer-welcome queue
// Filter Policy:
{
  "customer_type": ["new"]
}
// → Receives this message ✅ (customer_type = "new")

// Subscriber: clothing-inventory queue
// Filter Policy:
{
  "category": ["clothing", "shoes"]
}
// → Does NOT receive this message ❌ (electronics ≠ clothing)
```

###  Filter Policy Operators

| Operator | Example | Matches |
|----------|---------|---------|
| **Exact match** | `["electronics"]` | Only "electronics" |
| **Multiple values** | `["electronics", "books"]` | Either value |
| **Prefix** | `[{"prefix": "order-"}]` | Strings starting with "order-" |
| **Numeric** | `[{"numeric": [">=", 100]}]` | Numbers ≥ 100 |
| **Numeric range** | `[{"numeric": [">", 0, "<=", 100]}]` | 0 < x ≤ 100 |
| **Anything-but** | `[{"anything-but": ["test"]}]` | Everything except "test" |
| **Exists** | `[{"exists": true}]` | Attribute must be present |

*"Filter policies reduce noise AND cost. SQS charges per message. If you filter at the SNS level, unwanted messages never reach SQS — you don't pay for them."*

---

## Part 4: SNS Message Format & Attributes

###  Message Structure

*"When SNS delivers a message, it wraps your content in a JSON envelope:"*

```json
{
  "Type": "Notification",
  "MessageId": "a1b2c3d4-5678-90ab-cdef-11111EXAMPLE",
  "TopicArn": "arn:aws:sns:us-east-1:123456789012:production-alerts",
  "Subject": "ALARM: High CPU",
  "Message": "CPU utilization exceeded 80% on instance i-abc123",
  "Timestamp": "2024-04-18T10:30:00.000Z",
  "SignatureVersion": "1",
  "Signature": "EXAMPLEsignature==",
  "UnsubscribeURL": "https://sns.us-east-1.amazonaws.com/?Action=Unsubscribe&..."
}
```

###  Message Attributes

*"You can attach key-value metadata to messages. These are used for filtering:"*

```python
import boto3

sns = boto3.client('sns')
sns.publish(
    TopicArn='arn:aws:sns:us-east-1:123456789012:orders',
    Message='{"order_id": "ORD-123", "amount": 250}',
    MessageAttributes={
        'customer_type': {
            'DataType': 'String',
            'StringValue': 'new'
        },
        'amount': {
            'DataType': 'Number',
            'StringValue': '250'
        }
    }
)
```

*"Filter policies match against MessageAttributes, NOT the Message body (by default). You can also filter on the body with 'message body' filter policy scope."*

---

## Part 5: SNS Security & Access Control

###  Who Can Publish/Subscribe?

| Control | How |
|---------|-----|
| **IAM Policy** | IAM user/role needs `sns:Publish` or `sns:Subscribe` permissions |
| **Topic Policy** | Resource-based policy on the topic itself (like S3 bucket policy) |
| **Encryption** | SSE with KMS key (data encrypted at rest) |
| **VPC Endpoint** | PrivateLink endpoint — SNS calls stay in your VPC |

###  Common Topic Policy — Allow CloudWatch

```json
{
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "cloudwatch.amazonaws.com"},
    "Action": "sns:Publish",
    "Resource": "arn:aws:sns:us-east-1:123456789012:alerts"
  }]
}
```

*"This allows CloudWatch Alarms to publish to your topic. Without this, the alarm fires but the notification fails silently."*

---

## Part 6: SNS Pricing

| Component | Free Tier | Paid |
|-----------|-----------|------|
| **Publishes** | 1 million/month | $0.50 per million |
| **HTTP/S deliveries** | 100,000/month | $0.60 per million |
| **Email deliveries** | 1,000/month | $2 per 100,000 |
| **SMS** | 100 (to US) | $0.00645/msg (India), $0.00581/msg (US) |
| **SQS deliveries** | First 1M free | Free (SQS charges apply) |
| **Lambda deliveries** | First 1M free | Free (Lambda charges apply) |
| **Data transfer** | First 1 GB free | $0.09/GB after |

*"Email and SMS cost money. SQS and Lambda deliveries from SNS are free — you only pay the SQS/Lambda costs. That's another reason to use the fan-out pattern."*

---

# SECTION B: SQS (Simple Queue Service)

## Part 7: What Problem Does SQS Solve

###  The Problem — Tight Coupling

*"Imagine you run an e-commerce site. When someone places an order, your Order Service has to:"*

```
1. Charge the credit card     → Payment Service
2. Reserve inventory          → Inventory Service
3. Send confirmation email    → Email Service
4. Update analytics           → Analytics Service
```

*"Without a queue, the Order Service calls each one DIRECTLY and WAITS for a response:"*

```
Order Service → [calls Payment Service] → waits 2 sec
              → [calls Inventory Service] → waits 1 sec
              → [calls Email Service] → waits 0.5 sec
              → [calls Analytics Service] → waits 0.5 sec
Total: 4 seconds before customer sees "Order Placed!"

What if Payment Service is DOWN?
→ "Internal Server Error" 
→ Customer thinks order failed
→ Customer leaves
```

###  The Solution — Decouple with a Queue

```
Order Service → [drop message in SQS] → 50ms → "Order Placed!" ✅

Meanwhile, in the background:
  Payment Service ←── polls SQS ──── processes at its own pace
  Inventory Service ←── polls SQS ──── processes at its own pace
  Email Service ←── polls SQS ──── processes at its own pace
```

*"Fastest analogy: SQS is a post office. You drop a letter in the mailbox and walk away. The recipient picks it up when they're ready. You don't stand at their door waiting."*

###  Why Queues Are Essential

| Without Queue | With Queue |
|---------------|------------|
| If consumer is down → message lost | If consumer is down → message waits in queue |
| Producer must wait for response | Producer drops message and moves on |
| Spike in traffic → producer overwhelmed | Spike in traffic → queue absorbs it, consumer processes at its pace |
| 1 producer : 1 consumer | 1 producer : many consumers |
| Tightly coupled | Loosely coupled |

---

## Part 8: SQS Core Concepts

###  Queue Types — Standard vs FIFO

| Feature | Standard Queue | FIFO Queue |
|---------|---------------|------------|
| **Throughput** | Unlimited | 300 msg/sec (3,000 with batching) |
| **Ordering** | Best-effort | Strict FIFO (guaranteed) |
| **Duplicates** | Possible (at-least-once) | No duplicates (exactly-once) |
| **Name** | Any name | Must end in `.fifo` |
| **Use case** | Notifications, async tasks | Financial, ordering, inventory |

*"Standard: fast, might deliver messages twice or out of order. FIFO: slower, guaranteed order and no duplicates. Choose based on whether your application can handle duplicates."*

---

###  Message Lifecycle

```
1. PRODUCER sends message to queue
   │
   ▼
2. Message sits in queue (WAITING)
   │ ← Can wait up to 14 days (retention period)
   ▼
3. CONSUMER polls the queue → receives message
   │ ← Message becomes INVISIBLE (visibility timeout)
   │ ← Other consumers can't see it
   ▼
4. Consumer processes the message
   │
   ├── SUCCESS → Consumer DELETES the message from queue ✅
   │
   └── FAILURE → Visibility timeout expires
                 → Message becomes VISIBLE again
                 → Another consumer can try ♻️
                 → After N failures → Dead Letter Queue ☠️
```

###  Key Settings

#### Visibility Timeout

*"When a consumer picks up a message, it becomes invisible to other consumers for X seconds. This prevents two consumers from processing the same message."*

```
Default: 30 seconds
Range: 0 seconds to 12 hours
```

*"If the consumer finishes in 5 seconds, it deletes the message. Done. If the consumer takes longer than 30 seconds (maybe it crashed), the message reappears and another consumer can try."*

*"Rule: Set visibility timeout to 6× your average processing time. If processing takes 10 seconds, set it to 60 seconds."*

#### Message Retention

*"How long a message stays in the queue if nobody processes it."*

```
Default: 4 days
Range: 1 minute to 14 days
```

*"After 14 days, unprocessed messages are permanently deleted. If you need longer, you're doing something wrong — fix your consumers."*

#### Max Message Size

```
Maximum: 256 KB per message
```

*"If you need to send larger data, use the Extended Client Library — it stores the payload in S3 and sends a reference through SQS. Up to 2 GB."*

#### Delivery Delay

*"Delay delivery of new messages for X seconds."*

```
Default: 0 seconds (immediate)
Range: 0 to 900 seconds (15 minutes)
```

*"Use case: 'After someone places an order, wait 60 seconds before sending the confirmation email — in case they cancel immediately.'"*

---

###  Long Polling vs Short Polling

| Feature | Short Polling (Default) | Long Polling (Recommended) |
|---------|------------------------|---------------------------|
| **How it works** | Consumer asks, queue responds immediately (even if empty) | Consumer asks, queue waits up to 20 sec for a message |
| **Empty responses** | Yes (many) | Rarely |
| **API calls** | Many (costs money) | Fewer |
| **Latency** | Higher (polling loop) | Lower (immediate on arrival) |
| **Setting** | `ReceiveMessageWaitTimeSeconds = 0` | `ReceiveMessageWaitTimeSeconds = 20` |

```
Short Polling:
  Consumer: "Any messages?"  → Queue: "No."     $$$
  Consumer: "Any messages?"  → Queue: "No."     $$$
  Consumer: "Any messages?"  → Queue: "No."     $$$
  Consumer: "Any messages?"  → Queue: "Yes! Here." $

Long Polling:
  Consumer: "Any messages? I'll wait 20 seconds."
  ...(queue waits)...
  ...(message arrives at second 8)...
  Queue: "Yes! Here."  $
```

*"ALWAYS use long polling. Set `ReceiveMessageWaitTimeSeconds` to 20. It reduces cost by 90% and reduces latency."*

---

###  Dead Letter Queue (DLQ)

*"What happens when a message keeps failing? Maybe it has bad data. A consumer picks it up, crashes, the message reappears, another consumer picks it up, crashes again... infinite loop."*

*"A Dead Letter Queue catches messages that have failed too many times."*

```
Main Queue: orders-queue
  │
  │ Message fails 3 times (maxReceiveCount = 3)
  │
  ▼
Dead Letter Queue: orders-dlq
  │
  │ Message sits here for investigation
  │ An engineer reviews it, fixes the bug,
  │ and optionally replays the message
```

| Setting | What It Does |
|---------|-------------|
| **maxReceiveCount** | How many times a message can be received before going to DLQ |
| **DLQ ARN** | Which queue to send failed messages to |

*"EVERY production queue should have a DLQ. Without it, poison messages (messages that always fail) clog your queue forever."*

###  DLQ Redrive

*"After fixing the bug, you can REDRIVE messages from the DLQ back to the main queue:"*

1. **SQS Console** → Select DLQ → **Start DLQ redrive**
2. Choose: Redrive to source queue or a custom queue
3. **Start redrive**

*"This replays all the failed messages. If you fixed the bug, they'll process successfully this time."*

---

## Part 9: SNS + SQS Together — The Power Combo

###  Why Use Both?

*"SNS is for BROADCASTING (one-to-many push). SQS is for BUFFERING (reliable processing)."*

*"Used together:"*

```
SNS: "Hey everyone, a new order arrived!"     ← BROADCAST
  │
  ├── SQS: payment-queue      ← BUFFER for payment service
  ├── SQS: inventory-queue    ← BUFFER for inventory service
  └── SQS: email-queue        ← BUFFER for email service

Each service processes at its own speed.
If one service is slow or down, its queue just fills up.
Other services are unaffected.
```

###  SNS vs SQS — When to Use Which

| Feature | SNS | SQS |
|---------|-----|-----|
| **Model** | Pub/Sub (push) | Queue (pull) |
| **Consumers** | Multiple (fan-out) | Typically one (or competing consumers) |
| **Persistence** | No (message gone after delivery) | Yes (message stays until deleted) |
| **Retry** | Limited (3 for Lambda) | Built-in (visibility timeout + DLQ) |
| **Ordering** | FIFO topics available | FIFO queues available |
| **Use case** | Broadcasting alerts | Async job processing |

*"Decision tree:*
- *Need to notify MULTIPLE systems? → **SNS** (fan-out)*
- *Need reliable async processing? → **SQS** (buffer)*
- *Need both? → **SNS → SQS** (fan-out + buffer)"*

---

## Part 10: SQS Security & Encryption

###  Access Control

| Control | What It Does |
|---------|-------------|
| **IAM Policy** | Who can Send/Receive/Delete messages |
| **Queue Policy** | Resource-based policy (like S3 bucket policy) |
| **Encryption (SSE-SQS)** | AWS-managed encryption at rest (free) |
| **Encryption (SSE-KMS)** | Customer-managed KMS keys ($1/month + API) |
| **VPC Endpoint** | PrivateLink — SQS calls stay in your VPC |

###  Common Queue Policy — Allow SNS to Send

```json
{
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "sns.amazonaws.com"},
    "Action": "sqs:SendMessage",
    "Resource": "arn:aws:sqs:us-east-1:123456789012:payment-queue",
    "Condition": {
      "ArnEquals": {
        "aws:SourceArn": "arn:aws:sns:us-east-1:123456789012:orders"
      }
    }
  }]
}
```

*"Without this policy, SNS can't deliver to SQS even if they're in the same account. This is the #1 reason SNS→SQS fails."*

---

## Part 11: SQS Pricing

| Component | Free Tier | Paid |
|-----------|-----------|------|
| **Standard requests** | 1 million/month | $0.40 per million |
| **FIFO requests** | 1 million/month | $0.50 per million |
| **Data transfer** | First 1 GB free | $0.09/GB out |

*"Each API call = 1 request: SendMessage, ReceiveMessage, DeleteMessage, etc."*

```
Example: 10 million messages/month (standard)
  Send: 10M × $0.40/M    = $4.00
  Receive: 10M × $0.40/M = $4.00
  Delete: 10M × $0.40/M  = $4.00
  Total:                    $12.00/month
```

*"SQS is dirt cheap. Even at massive scale. Long polling dramatically reduces ReceiveMessage calls."*

---

## Part 12: SNS & SQS Interview Questions

###  Top 20 Interview Questions

**SNS:**

1. **What is SNS?**
   → A fully managed pub/sub messaging service that sends messages to multiple subscribers simultaneously.

2. **What's the difference between SNS Standard and FIFO topics?**
   → Standard: unlimited throughput, best-effort ordering, possible duplicates. FIFO: 300 msg/sec, strict ordering, exactly-once delivery.

3. **What subscriber types does SNS support?**
   → Email, SMS, HTTP/S, Lambda, SQS, Kinesis Firehose, platform push (mobile).

4. **What is SNS fan-out?**
   → One message published to a topic is delivered to ALL subscribers. Used to broadcast events to multiple systems.

5. **What is a subscription filter policy?**
   → A JSON policy on a subscription that only delivers messages matching certain attributes. Reduces noise and SQS costs.

6. **Can SNS guarantee message delivery?**
   → For SQS/Lambda: yes (with retries). For HTTP: retry policy (up to 100K retries). For email/SMS: best-effort.

7. **How does SNS work with CloudWatch?**
   → CloudWatch Alarms publish notifications to SNS topics, which then deliver to email/SMS/Lambda/etc.

**SQS:**

8. **What is SQS?**
   → A fully managed message queue service for decoupling and scaling microservices.

9. **Standard vs FIFO queue?**
   → Standard: unlimited throughput, at-least-once, best-effort ordering. FIFO: 300 msg/sec, exactly-once, strict ordering.

10. **What is visibility timeout?**
    → The period during which a message is invisible to other consumers after being received. Default: 30 seconds. If the consumer doesn't delete it in time, it reappears.

11. **What is a Dead Letter Queue?**
    → A queue that receives messages that have failed processing N times (maxReceiveCount). Used to isolate problematic messages.

12. **Long polling vs Short polling?**
    → Short: returns immediately (even if empty). Long: waits up to 20 seconds for messages. Long polling reduces empty responses and costs.

13. **Max message size?**
    → 256 KB. Use Extended Client Library with S3 for up to 2 GB.

14. **Max retention period?**
    → 14 days. Default is 4 days.

15. **Can multiple consumers read from the same SQS queue?**
    → Yes. This is "competing consumers" pattern. Each message is delivered to ONE consumer (visibility timeout prevents duplicates). Great for parallel processing.

**SNS + SQS:**

16. **Why use SNS + SQS together?**
    → SNS broadcasts to multiple SQS queues (fan-out). Each queue buffers messages for its consumer. Combines broadcasting with reliable processing.

17. **What happens if SNS delivers to SQS but no consumer is processing?**
    → Messages pile up in the queue (up to 14 days retention). They'll be processed when the consumer comes back.

18. **Can SNS deliver to SQS in a different AWS account?**
    → Yes. Add a queue policy allowing the SNS topic ARN from the other account to call `sqs:SendMessage`.

19. **How do you prevent SNS from overwhelming a slow consumer?**
    → Put SQS between them. SNS delivers to SQS instantly. The consumer reads from SQS at its own pace. SQS absorbs the spike.

20. **What is the difference between SNS and EventBridge?**
    → SNS: simple pub/sub, good for fan-out to SQS/Lambda/email. EventBridge: event bus with content-based filtering, scheduling, 100+ AWS service integrations, cross-account routing. EventBridge is more powerful but more complex.

---

# SECTION C: HANDS-ON LABS

## 🟢 Lab 1: BASIC — SNS Email Notifications

### Step 1: Create an SNS Topic

1. **SNS Console** → **Topics** → **Create topic**
2. **Type:** Standard
3. **Name:** `streamflix-alerts`
4. Leave all other settings default → **Create topic**

### Step 2: Create Email Subscription

1. Click **Create subscription**
2. **Protocol:** Email
3. **Endpoint:** Your email address
4. **Create subscription**

5. **CHECK YOUR EMAIL** → Click the confirmation link!
   
   ⚠️ *The subscription stays "Pending confirmation" until you click the link. This prevents spam.*

### Step 3: Publish a Test Message

**From the Console:**
1. Select your topic → **Publish message**
2. **Subject:** `Test Alert from StreamFlix`
3. **Message body:** `This is a test notification from our StreamFlix lab.`
4. **Publish message**

**From CLI:**
```bash
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:streamflix-alerts" \
  --subject "Test Alert" \
  --message "Hello from the CLI! If you see this, SNS is working."
```

### Step 4: Verify

- Check your email inbox → You should see the message
- Check the **topic dashboard** → see the delivery count increase

### Step 5: Add More Subscribers

1. Add another **Email** subscriber (a teammate or second email)
2. Add an **SMS** subscriber (your phone number with country code, e.g., `+91XXXXXXXXXX`)
3. Publish another message → both email AND SMS arrive!

*"One publish, multiple deliveries. That's the power of pub/sub."*

---

## 🟡 Lab 2: INTERMEDIATE — SQS Queue + SNS Fan-Out + DLQ

### Step 1: Create the Dead Letter Queue

1. **SQS Console** → **Create queue**
2. **Type:** Standard
3. **Name:** `streamflix-orders-dlq`
4. **Message retention:** 14 days
5. **Create queue**

### Step 2: Create the Main Queue

1. **Create queue**
2. **Type:** Standard
3. **Name:** `streamflix-orders-queue`
4. **Configure:**

| Setting | Value |
|---------|-------|
| Visibility timeout | `60` seconds |
| Message retention | `4` days |
| Receive message wait time | `20` seconds (long polling!) |
| Delivery delay | `0` seconds |

5. **Dead Letter Queue:**
   - Enabled: ✅
   - Queue: `streamflix-orders-dlq`
   - Max receives: `3` (after 3 failures → DLQ)

6. **Create queue**

### Step 3: Create SNS Topic for Orders

1. **SNS** → **Topics** → **Create topic**
2. **Name:** `new-orders`
3. **Create topic**

### Step 4: Subscribe SQS to SNS

1. In the SNS topic → **Create subscription**
2. **Protocol:** Amazon SQS
3. **Endpoint:** ARN of `streamflix-orders-queue`
4. Check ✅ **Enable raw message delivery** (sends your message directly, no SNS wrapper)
5. **Create subscription**

6. **IMPORTANT: Update SQS queue policy!**
   - Go to **SQS** → `streamflix-orders-queue` → **Access policy** tab → **Edit**
   - Add this statement (or use the auto-generated one):

```json
{
  "Statement": [{
    "Effect": "Allow",
    "Principal": {"Service": "sns.amazonaws.com"},
    "Action": "sqs:SendMessage",
    "Resource": "arn:aws:sqs:us-east-1:YOUR_ACCOUNT:streamflix-orders-queue",
    "Condition": {
      "ArnEquals": {
        "aws:SourceArn": "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders"
      }
    }
  }]
}
```

### Step 5: Publish an Order via SNS

```bash
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders" \
  --message '{"order_id":"ORD-001","item":"StreamFlix Premium","amount":499,"customer":"saikiran@example.com"}' \
  --message-attributes '{"customer_type":{"DataType":"String","StringValue":"premium"}}'
```

### Step 6: Read the Message from SQS

```bash
# Receive message (long poll)
aws sqs receive-message \
  --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-queue" \
  --wait-time-seconds 20 \
  --max-number-of-messages 1
```

**Expected output:**
```json
{
  "Messages": [{
    "MessageId": "abc-123",
    "ReceiptHandle": "xyz-789-...",
    "Body": "{\"order_id\":\"ORD-001\",\"item\":\"StreamFlix Premium\",\"amount\":499}",
    "MD5OfBody": "..."
  }]
}
```

### Step 7: Delete the Message (Acknowledge Processing)

```bash
aws sqs delete-message \
  --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-queue" \
  --receipt-handle "xyz-789-..."  # Use the actual ReceiptHandle from Step 6
```

*"If you don't delete it, after 60 seconds (visibility timeout) it reappears. After 3 reappearances, it goes to DLQ."*

### Step 8: Demo the Dead Letter Queue

```bash
# Publish a message
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders" \
  --message '{"order_id":"ORD-BAD","item":"CORRUPTED DATA"}'

# Receive it 3 times WITHOUT deleting (simulating failures)
for i in 1 2 3; do
  echo "=== Attempt $i ==="
  aws sqs receive-message \
    --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-queue" \
    --wait-time-seconds 5
  echo "Not deleting... waiting for visibility timeout..."
  sleep 65  # Wait for visibility timeout to expire
done

# Now check the DLQ — the message should be there!
echo "=== Checking DLQ ==="
aws sqs receive-message \
  --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-dlq" \
  --wait-time-seconds 5
```

*"The message failed 3 times and moved to the DLQ automatically. In production, you'd have a Lambda or alarm monitoring the DLQ."*

### Step 9: Create DLQ CloudWatch Alarm

1. **CloudWatch** → **Alarms** → **Create alarm**
2. Metric: **SQS** → `ApproximateNumberOfMessagesVisible` → `streamflix-orders-dlq`
3. Threshold: **Greater than** `0`
4. Action: SNS topic → `streamflix-alerts`
5. Name: `DLQ-Has-Messages`

*"Now any message that lands in the DLQ triggers an email alert. No poison messages go unnoticed."*

---

## 🔴 Lab 3: ADVANCED — Fan-Out Architecture with Lambda Processing
### Architecture

```
┌───────────┐
│   User    │
│ places    │
│ order     │
└─────┬─────┘
      │ (CLI publish)
      ▼
┌──────────────┐
│  SNS Topic:  │
│ "new-orders" │
└──────┬───────┘
       │
  ┌────┼────────────────┐
  │    │                │
  ▼    ▼                ▼
┌────┐ ┌─────────┐  ┌─────────┐
│SQS:│ │SQS:     │  │Lambda:  │
│pay │ │inventory│  │analytics│
│ment│ │-queue   │  │-func    │
└──┬─┘ └────┬────┘  └────┬────┘
   │        │             │
   ▼        ▼             ▼
Lambda   Lambda      CloudWatch
payment  inventory   Logs
-proc    -proc
```

### Step 1: Create Second SQS Queue (for Inventory)

1. **SQS** → Create queue → Standard
2. Name: `streamflix-inventory-queue`
3. Visibility timeout: 60, Long polling: 20
4. DLQ: Create `streamflix-inventory-dlq`, maxReceiveCount: 3

### Step 2: Subscribe Inventory Queue to SNS

1. **SNS** → `new-orders` topic → **Create subscription**
2. Protocol: SQS → ARN of `streamflix-inventory-queue`
3. **Add filter policy** (only electronics orders):

```json
{
  "category": ["electronics", "streaming"]
}
```

4. Update `streamflix-inventory-queue` access policy to allow SNS

### Step 3: Create Lambda — Direct SNS Subscriber

1. **Lambda** → **Create function**
2. Name: `order-analytics`
3. Runtime: Python 3.12
4. Code:

```python
import json

def lambda_handler(event, context):
    for record in event['Records']:
        message = json.loads(record['Sns']['Message'])
        print(f"📊 Analytics: New order received!")
        print(f"   Order ID: {message.get('order_id', 'unknown')}")
        print(f"   Amount: ${message.get('amount', 0)}")
        print(f"   Item: {message.get('item', 'unknown')}")
        print(f"   Category: {message.get('category', 'unknown')}")
    
    return {'statusCode': 200, 'body': 'Logged!'}
```

5. **Deploy**

### Step 4: Subscribe Lambda to SNS

1. **SNS** → `new-orders` → **Create subscription**
2. Protocol: **AWS Lambda**
3. Endpoint: ARN of `order-analytics` function
4. No filter (receives ALL orders)

### Step 5: Create Lambda — SQS Consumer (Payment Processing)

1. **Lambda** → Create function → `payment-processor`
2. Code:

```python
import json
import time

def lambda_handler(event, context):
    for record in event['Records']:
        body = json.loads(record['body'])
        order_id = body.get('order_id', 'unknown')
        amount = body.get('amount', 0)
        
        print(f"💳 Processing payment for {order_id}: ${amount}")
        
        # Simulate processing time
        time.sleep(2)
        
        # Simulate occasional failure for DLQ demo
        if 'FAIL' in str(body.get('item', '')):
            raise Exception(f"Payment failed for {order_id}!")
        
        print(f"✅ Payment successful for {order_id}")
    
    return {'statusCode': 200}
```

3. **Add SQS trigger:**
   - Lambda → Configuration → Triggers → Add trigger
   - SQS → `streamflix-orders-queue`
   - Batch size: 1
   - Enable trigger

### Step 6: Test the Full Architecture

```bash
# Publish an order
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders" \
  --message '{"order_id":"ORD-100","item":"StreamFlix Premium Plan","amount":499,"category":"streaming","customer":"student@class.com"}' \
  --message-attributes '{
    "category":{"DataType":"String","StringValue":"streaming"},
    "amount":{"DataType":"Number","StringValue":"499"}
  }'

echo "✅ Order published! Check:"
echo "  1. Lambda 'order-analytics' CloudWatch logs (direct SNS → Lambda)"
echo "  2. Lambda 'payment-processor' CloudWatch logs (SNS → SQS → Lambda)"
echo "  3. SQS 'streamflix-inventory-queue' for the message (filtered: streaming)"
```

### Step 7: Verify Each Path

```bash
# Check Lambda analytics logs
aws logs tail /aws/lambda/order-analytics --since 2m

# Check Lambda payment logs
aws logs tail /aws/lambda/payment-processor --since 2m

# Check inventory queue (should have message since category=streaming matches filter)
aws sqs receive-message \
  --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-inventory-queue" \
  --wait-time-seconds 5
```

### Step 8: Test Filter Policy

```bash
# Publish a "books" order — inventory queue should NOT receive this
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders" \
  --message '{"order_id":"ORD-101","item":"AWS Textbook","amount":29,"category":"books"}' \
  --message-attributes '{"category":{"DataType":"String","StringValue":"books"}}'

# Check inventory queue — should be empty! (books ≠ electronics or streaming)
aws sqs get-queue-attributes \
  --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-inventory-queue" \
  --attribute-names ApproximateNumberOfMessagesVisible

# But analytics Lambda still receives it (no filter)
aws logs tail /aws/lambda/order-analytics --since 2m
```

*"See? The filter policy worked. The inventory service only gets electronics and streaming orders. Books are ignored. But analytics gets EVERYTHING because it has no filter."*

### Step 9: Test DLQ Path

```bash
# Publish a "bad" order that triggers the payment failure
aws sns publish \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders" \
  --message '{"order_id":"ORD-BAD","item":"FAIL_THIS_ORDER","amount":0}' \
  --message-attributes '{"category":{"DataType":"String","StringValue":"streaming"}}'

# Wait 2 minutes for 3 retries, then check DLQ
sleep 120
aws sqs get-queue-attributes \
  --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-dlq" \
  --attribute-names ApproximateNumberOfMessagesVisible
# → Should show 1 message in DLQ
```

---

## Cleanup

```bash
# Delete Lambda functions
aws lambda delete-function --function-name order-analytics
aws lambda delete-function --function-name payment-processor

# Delete SNS subscriptions (list them first)
aws sns list-subscriptions-by-topic \
  --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders"
# Delete each subscription ARN

# Delete SNS topics
aws sns delete-topic --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:new-orders"
aws sns delete-topic --topic-arn "arn:aws:sns:us-east-1:YOUR_ACCOUNT:streamflix-alerts"

# Delete SQS queues
aws sqs delete-queue --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-queue"
aws sqs delete-queue --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-orders-dlq"
aws sqs delete-queue --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-inventory-queue"
aws sqs delete-queue --queue-url "https://sqs.us-east-1.amazonaws.com/YOUR_ACCOUNT/streamflix-inventory-dlq"

# Delete CloudWatch alarms
aws cloudwatch delete-alarms --alarm-names "DLQ-Has-Messages"
```

---

## Summary: What Each Lab Teaches

| Lab | Level | Duration | Concepts |
|-----|-------|----------|----------|
| 🟢 **Lab 1** | Basic | 15 min | SNS topic, email subscription, confirm, publish, multi-subscriber |
| 🟡 **Lab 2** | Intermediate | 30 min | SQS queue, SNS→SQS fan-out, long polling, DLQ, CW alarm on DLQ |
| 🔴 **Lab 3** | Advanced | 35 min | Full fan-out: SNS → SQS + Lambda, filter policies, SQS→Lambda trigger, DLQ path, Logs verification |
