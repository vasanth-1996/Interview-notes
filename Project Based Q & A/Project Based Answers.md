# Project Based Answers

## Q1: Explain the Complete Architecture of Your Recent Project

> "Vasanth, take one of your recent projects and explain the complete architecture to me, from the client request until the response"

### Project Overview

This project was an enterprise **batch-processing system** for a health insurance platform. We had multiple batch jobs that needed to process data periodically and integrate with systems such as Oracle HUGO and SQL databases.

### High-Level Architecture Flow

```
Batch Trigger → Azure Function → Durable Function orchestration → Activity/Business Logic → Oracle/SQL → Logging & Monitoring
```

### Technology Stack & Architecture Breakdown

#### 1. **Core Execution Layer**
- **Azure Functions** with .NET Core as the entry point
- Functions triggered periodically based on schedule
- Handled the job execution responsibility

#### 2. **Orchestration & Workflow Management**
- **Azure Durable Functions** for complex workflows
- Coordinated multiple activities without putting all logic in the orchestrator
- Supported long-running, multi-step processing
- Built-in retry and state management

#### 3. **Business Logic & Clean Architecture**
- Business logic kept separate from the function trigger
- Service and business-layer components for reusability
- Function responsibility: trigger + orchestration
- Service layer responsibility: actual business logic

#### 4. **Data Layer**
- Communication with backend systems:
  - Oracle HUGO database
  - SQL databases
- Data processing based on batch requirements

#### 5. **Configuration & Secrets Management**
- **Azure Key Vault** for sensitive information
- Credentials and configuration secured (no hardcoding)
- Environment-based configuration

#### 6. **Reliability & Error Handling**
- Failure and retry handling at appropriate processing levels
- Durable Functions automatically retried failed activities
- Workflow could continue without full restart

#### 7. **Monitoring & Logging**
- **Application Insights** for monitoring and diagnostics
- **Serilog** for application logging
- Production troubleshooting and performance analysis
- Failure investigation and execution details tracking

#### 8. **Quality Assurance**
- **Unit & Integration Tests** using xUnit and Moq
- **SonarQube** for code quality analysis
- **Checkmarx** for security vulnerability scanning

#### 9. **Deployment**
- Deployed as **Azure Function Apps**
- Runs periodic workloads independently from client-facing APIs

### Architecture Summary

| Component | Responsibility |
|-----------|-----------------|
| Azure Functions | Job execution & trigger handling |
| Durable Functions | Workflow orchestration & state management |
| Business/Service Layers | Core processing logic |
| Oracle/SQL | Data persistence |
| Key Vault | Secure configuration |
| Application Insights + Serilog | Monitoring & logging |

---

## Q2: Why Azure Functions Instead of .NET Web API for Batch Jobs?

> "You said Azure Function handles the job. Why didn't you just create a separate .NET Web API for the batch job? What's the actual advantage of Azure Functions here?"

### The Core Difference

While technically a .NET Web API could execute the same logic, Azure Functions was chosen based on **workload characteristics**, not execution capability.

### Workload Nature

**Batch Processing Requirements:**
- Periodic/scheduled execution (not continuous request-response)
- Background processing without client waiting
- No requirement for user-facing interaction
- Long-running, multi-step workflows

### Key Advantages of Azure Functions

#### 1. **Serverless Execution Model**
- Focus on business logic without managing application servers
- No infrastructure management overhead
- Automatic scaling based on demand

#### 2. **Platform Integration**
- Seamless integration with Azure ecosystem:
  - Durable Functions for orchestration
  - Application Insights for monitoring
  - Key Vault for secrets management
- Built-in bindings and triggers

#### 3. **Orchestration & State Management**
- **No Custom Implementation Needed:**
  - Scheduling (built-in)
  - State management (Durable Functions)
  - Retry logic (built-in)
  - Long-running workflow tracking (built-in)
  
- **With a Web API, you'd need to build all of this yourself**

#### 4. **Infrastructure Alignment**
- With Web API, you'd still need something to trigger it periodically (Task Scheduler, separate orchestrator, etc.)
- With Azure Functions, everything is unified in the platform

### Decision Rationale

**Both can execute code. The decision was based on:**

✓ **Workload type** — Scheduled, background, event-driven  
✓ **Platform fit** — Durable Functions ecosystem  
✓ **Operational overhead** — Minimal infrastructure management  
✓ **Built-in capabilities** — Retry, state, scheduling  
✓ **Cost efficiency** — Pay-per-execution model  

### What a Web API Approach Would Require

If we had used a Web API:
- Separate scheduler/orchestrator service
- Custom scheduling implementation
- Custom state management for long-running workflows
- Custom retry logic
- Custom infrastructure provisioning and management
- Continuous server availability (higher costs)

### Conclusion

Azure Functions with Durable Functions was a **better architectural fit** for a periodic, scheduled, background batch workload compared to a continuously-running Web API.


----------------

You said it's a background workload. But I can create a Worker Service or Web API with a scheduler. Why exactly is Azure Functions better?


Yes 👍. Let's forget the **interview-answer paragraph** for now. I'll explain it to you like a beginner, so you understand the concept first.

# Azure Functions vs Web API vs Worker Service

The interviewer asks:

> **"Why did you use Azure Function? Couldn't you just create a Web API or Worker Service?"**

The simple answer is:

**Yes, we could.**

Azure Function is **not magically capable of something that Web API cannot do**.

The difference is mainly **how the application is hosted and how the work is triggered/managed**.

---

## 1. First understand a Web API

Imagine you create:

```text
.NET Web API
     ↓
POST /process-batch
     ↓
Process 100,000 records
```

Someone has to call this API.

For example:

```text
Control-M
    ↓
calls API
    ↓
POST /process-batch
    ↓
Web API
    ↓
Process records
```

So yes, you can absolutely build your batch as a Web API.

---

# 2. Then what is a Worker Service?

A Worker Service is designed specifically for background work.

For example:

```text
Worker Service
      ↓
Wait
      ↓
Run batch
      ↓
Wait
      ↓
Run batch
      ↓
Wait...
```

It is normally a continuously running application.

You might host it on:

* VM
* Kubernetes
* App Service
* Container
* etc.

So:

```text
Server
  ↓
Worker Service
  ↓
Background processing
```

---

# 3. Now Azure Function

Azure Function is also capable of running background code.

But Azure gives you a **managed way of running that code based on triggers**.

For example:

```text
Trigger
   ↓
Azure Function
   ↓
Your C# code
```

The trigger could be:

```text
Timer
Queue
HTTP
Event
etc.
```

In **your Bupa project**, you told me the scheduling was handled by **Control-M**.

So your conceptual flow is:

```text
Control-M
    ↓
Azure Function
    ↓
Batch processing
```

---

# 4. So where is the actual difference?

This is the important part.

Imagine you have a batch that runs once every night.

### Option A — Worker Service

You need a Worker Service continuously running:

```text
        Server
          ↓
   Worker Service
          ↓
       waiting
          ↓
       waiting
          ↓
     10:00 PM
          ↓
    Run batch
```

The application is basically sitting there waiting for work.

---

### Option B — Web API

You have:

```text
Control-M
    ↓
HTTP request
    ↓
Web API
    ↓
Run batch
```

You need to maintain the API application and the mechanism that invokes it.

---

### Option C — Azure Function

You have:

```text
Control-M
    ↓
Azure Function
    ↓
Run batch
```

Azure manages much of the underlying hosting/execution infrastructure.

That's one reason Functions can be attractive for this type of workload.

---

# 5. But wait... Control-M already exists!

Exactly! 👏

This is where your interviewer may challenge you.

They might say:

> **"Vasanth, Control-M is already scheduling the job. So why do you need Azure Functions? Why not Control-M → Web API?"**

That's a **very good question**.

And the answer is:

You **could** do:

```text
Control-M
    ↓
Web API
    ↓
Batch
```

There is nothing technically wrong with that.

Your team chose:

```text
Control-M
    ↓
Azure Function
    ↓
Batch
```

because the batch workload was suitable for the **serverless/managed execution model** and the project was already using Azure services.

---

# 6. Think of it like this

Suppose you need someone to clean a room once a day.

### Worker Service

You hire a person and keep them inside the room **24 hours a day**, waiting.

```text
Person
  ↓
Waiting...
  ↓
Waiting...
  ↓
Clean room
```

### Web API

You have a person available and someone calls:

> "Please clean the room."

```text
Caller
  ↓
Person
  ↓
Clean room
```

### Azure Function

You tell the building management:

> "Whenever this job needs to happen, provide the execution environment and run this code."

```text
Trigger
  ↓
Managed execution
  ↓
Run code
```

That's the basic idea.

---

# 7. Very important: Don't say "Functions are better"

In an interview, don't think:

```text
Azure Function > Web API
```

Think:

```text
Which one fits my workload better?
```

For example:

| Requirement                             | Possible choice   |
| --------------------------------------- | ----------------- |
| Client request/response                 | Web API           |
| Continuously running background process | Worker Service    |
| Event/scheduled/serverless workload     | Azure Function    |
| Complex multi-step serverless workflow  | Durable Functions |

There can be overlap.

---

# 8. Then why did Bupa use Durable Functions?

That's the **next level**.

Suppose your batch isn't just:

```text
Start
 ↓
Process
 ↓
Finish
```

Instead:

```text
Start
 ↓
Get data from Oracle
 ↓
Validate
 ↓
Process records
 ↓
Update SQL
 ↓
Generate result
 ↓
Finish
```

Now you have a **workflow with multiple steps**.

That's where Durable Functions becomes useful.

```text
Azure Function
      ↓
Durable Function
      ↓
 ┌────┼────┬────┐
 ↓    ↓    ↓    ↓
 A    B    C    D
```

We'll cover this in **Question 3**.

---

## 🧠 Remember just these 3 things

**Control-M**

> "When should the job run?"

**Azure Function**

> "Run my code."

**Durable Function**

> "Manage this multi-step workflow."


-------------------


Suppose the batch takes 2 hours. What happens if the Function execution is interrupted?



### 2. **Unexpected Crash** ⚠️ — Your Server Dies

**What happens:**
- Your function app crashes (bad code, out of memory, etc.)
- No warning, no goodbye

**The sequence:**
```
Your job is running...
    ↓
ERROR! Server crashes
    ↓
Function stops immediately
    ↓
Everything in memory is LOST
    ↓
But progress is saved to Azure Storage
```

**Real-world analogy:**
Like your laptop dying mid-work. You lose what's on screen, but if you had autosaved, you recover from that checkpoint.

**What's lost:**
- ❌ Work still being done in memory
- ❌ Temporary files that weren't flushed to disk
- ❌ Emails that were in the process of being sent

**What's saved:**
- ✅ Job progress (stored in Azure Table Storage)
- ✅ Reports already written to disk
- ✅ Database commits that already finished

**The recovery:**
- Someone needs to manually restart the job (it doesn't auto-restart)
- The system knows the job stopped via the stored progress

---

### 3. **Network Connection Dies** 🚨 — The Most Common Problem!

**The tricky scenario:**
Your job is running fine, but the connection between your Function App and the database breaks. This is the real issue your team documented.

**What happens:**
```
Your job: "Hey Database, process this huge report for me"
    ↓
Database: "Sure, I'm working on it... (processing for 90 minutes)"
    ↓
[SOMEWHERE IN THE MIDDLE]
Network component (Firewall/VPN/Router) says:
"This connection has been idle for 60 minutes. I'm closing it."
    ↓
Your job is STILL WAITING for the answer
    ↓
Database is STILL PROCESSING
    ↓
But the communication line is CUT
    ↓
Your job hangs (waits forever for an answer that will never come)
```

**Real-world analogy:**
You call a pizza shop and ask for a special order. They say "Sure, we're making it!" But then the phone line goes dead. You're holding an empty line, the pizza shop is still cooking, but you can't hear each other.

**The danger:**
- ⚠️ Your job **hangs** (sits idle doing nothing)
- ⚠️ The database **continues working** (you don't know this!)
- ⚠️ After 6 hours (your timeout), the job finally gives up
- ⚠️ Database finishes work, but nobody is there to receive it

**Who's at fault?**

From your team's investigation:
- ❌ NOT Azure Function App (can run 6 hours)
- ❌ NOT Database (keeps running fine)
- ✅ **Probably network security** (Firewall, NAT Gateway, VPN has a ~60-minute idle timeout)

---
