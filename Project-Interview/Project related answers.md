# Project Based Answers

## Index

1. [Explain the Complete Architecture of Your Recent Project](#q1-explain-the-complete-architecture-of-your-recent-project)
2. [Explain the Previous Project Structure](#explain-the-previous-project-structure)
3. [Why did you customize ASP.NET Identity?](#why-did-you-customize-aspnet-identity)
4. [Why Azure Functions Instead of .NET Web API for Batch Jobs?](#q2-why-azure-functions-instead-of-net-web-api-for-batch-jobs)
5. [Azure Functions vs Web API vs Worker Service](#azure-functions-vs-web-api-vs-worker-service)
6. [Suppose the batch takes 2 hours. What happens if the Function execution is interrupted?](#suppose-the-batch-takes-2-hours-what-happens-if-the-function-execution-is-interrupted)
7. [Suppose Your Batch Processes 100,000 Records and Fails After 60,000. How Would You Resume It?](#q3-suppose-your-batch-processes-100000-records-and-fails-after-60000-how-would-you-resume-it)
8. [Why did you use both Serilog and Application Insights?](#why-did-you-use-both-serilog-and-application-insights-arent-they-doing-the-same-thing)
9. [What is the difference between logs, metrics, and traces?](#what-is-the-difference-between-logs-metrics-and-traces)
10. [What exactly would you look at in Application Insights?](#what-exactly-would-you-look-at-in-application-insights)
11. [How would you investigate a production batch that suddenly takes 3 hours instead of 20 minutes?](#how-would-you-investigate-a-production-batch-that-suddenly-takes-3-hours-instead-of-20-minutes)
12. [Where do you store database passwords? Why not store them in appsettings.json?](#where-do-you-store-database-passwords-why-not-store-them-in-appsettingsjson)
13. [How does Azure Function access Key Vault? What is Managed Identity?](#how-does-azure-function-access-key-vault-what-is-managed-identity)
14. [If multiple batch projects use the same Key Vault, is the identity system-assigned or user-assigned?](#if-multiple-batch-projects-use-the-same-key-vault-is-the-identity-system-assigned-or-user-assigned)
15. [How do you rotate secrets?](#how-do-you-rotate-secrets)
16. [Tell me one production issue you personally faced in this project and how you solved it](#tell-me-one-production-issue-you-personally-faced-in-this-project-and-how-you-solved-it)
17. [Saga and Outbox POC](#1-what-are-we-trying-to-prove-with-the-poc)
18. [How does Entra ID issue a JWT?](#how-does-entra-id-issue-a-jwt)
19. [How do you secure a Web API application?](#how-do-you-secure-a-web-api-application)
20. [How would you secure service-to-service communication?](#how-would-you-secure-service-to-service-communication)
21. [How does the batch make a real-time call to the Claims Domain API?](#how-does-the-batch-make-a-real-time-call-to-the-claims-domain-api)
22. [How is the Azure Function deployed?](#how-is-the-azure-function-deployed)
23. [How did CI/CD work in the previous project?](#how-did-cicd-work-in-the-previous-project)

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

## Explain the Previous Project Structure

The previous project was an **Edlio-like online school platform**. It was a multi-tenant SaaS application, which means one platform could serve many schools while keeping each school's data isolated.

### High-level structure

```text
Student Portal / School Admin Portal / Edlio Admin Portal
          ↓
          API Gateway
          ↓
   ┌────────────────┼────────────────┐
   ↓                ↓                ↓
 Identity Service   School Service   Student Service
   ↓                ↓                ↓
   Fee Service    Activity Service   Payment Service
          ↓
       Order / Notification
          ↓
       Message Broker
```

### Main layers

1. **User portals**

   Students could view information, enroll in activities, and make payments. School administrators could manage students, activities, fees, and school settings. Edlio super administrators could manage all schools and platform-level operations.

2. **API Gateway**

   All client requests entered through one gateway. It routed each request to the correct service and could handle authentication, authorization, rate limiting, request aggregation, and API versioning.

3. **Microservices**

   Each service owned one business area:
   - **Identity and Access Service:** login, JWT tokens, roles, permissions, and tenant-aware access
   - **School Management Service:** school profiles, onboarding, and settings
   - **Student Enrollment Service:** students, enrollments, status, and enrollment history
   - **Activity Service:** activities, schedules, capacity, and waitlists
   - **Fee Management Service:** fee structures, discounts, and fee calculations
   - **Order and Payment Services:** orders, payments, refunds, and transaction status
   - **Notification Service:** email, SMS, and in-app notifications

4. **Database per service**

   Each microservice owned its own database. Other services did not directly update that database. This kept data isolated and allowed services to be deployed or scaled independently.

5. **Communication between services**

   Immediate operations used synchronous REST calls. For work that did not need an immediate response, services published events through a message broker such as AWS SQS/SNS or RabbitMQ. For example, an `OrderCreated` event could trigger payment processing and a notification without tightly coupling those services.

6. **AWS infrastructure and deployment**

   The platform could use API Gateway for entry and protection, ECS with Fargate for container hosting, RDS for relational data, ElastiCache for frequently accessed data, S3 for files and images, and SQS/SNS for messaging. Docker images were deployed independently to the required services.

### Example request flow: creating a fee

```text
School Admin Portal
   ↓ POST /api/fees with JWT
API Gateway validates token and routes request
   ↓
Fee Management Service validates the request
   ↓
Image is stored in S3, if provided
   ↓
Fee is saved in the Fee database
   ↓
FeeCreated event is published
   ↓
Notification or Reporting Service handles it asynchronously
```

### Interview-ready answer

> **My previous project was a multi-tenant online school platform built with microservices. Students, school administrators, and super administrators used separate portals, and all requests entered through an API Gateway. The gateway handled routing and initial security checks. Each domain, such as Identity, School, Student Enrollment, Activities, Fees, Orders, Payments, and Notifications, had its own service and database. We used REST for operations requiring an immediate response and a message broker for asynchronous events. The services were containerized and deployed independently using AWS services such as ECS with Fargate, RDS, S3, ElastiCache, and SQS/SNS.**

---

## Why did you customize ASP.NET Identity?

We customized ASP.NET Identity because the standard user fields were not enough for our school platform. We needed to support multiple schools, different user types, fine-grained permissions, and secure refresh-token handling.

### Why customize it?

- **Multi-tenancy:** Every user needed a `TenantId` or `SchoolId` so the API could isolate one school's data from another school's data.
- **Business roles:** We needed roles such as `Student`, `SchoolAdmin`, and `SuperAdmin`.
- **Permissions:** Roles needed specific permissions, such as `enrollment.read` or `payment.write`.
- **Refresh tokens:** Access tokens were short-lived, so refresh tokens needed database storage, expiry, revocation, and rotation.
- **Security controls:** We configured password rules, account lockout, and audit-related fields.

### 1. Extend the user model

Instead of using Identity's default user class, we created our own class and inherited from `IdentityUser`:

```csharp
public class ApplicationUser : IdentityUser
{
   public string FirstName { get; set; } = string.Empty;
   public string LastName { get; set; } = string.Empty;
   public Guid TenantId { get; set; }
   public ICollection<RefreshToken> RefreshTokens { get; set; } = new List<RefreshToken>();
}
```

`TenantId` was important because it was taken from the authenticated user's claims and used when accessing school data.

### 2. Add a refresh-token entity

Refresh tokens were stored separately so they could be expired, revoked, and rotated:

```csharp
public class RefreshToken
{
   public Guid Id { get; set; }
   public string Token { get; set; } = string.Empty;
   public DateTime ExpiresAtUtc { get; set; }
   public DateTime? RevokedAtUtc { get; set; }
   public string? ReplacedByToken { get; set; }
   public string UserId { get; set; } = string.Empty;
   public ApplicationUser User { get; set; } = null!;
}
```

The service checked that the token existed, had not expired, and had not been revoked before issuing a new access token.

### 3. Use a custom Identity DbContext

The DbContext inherited from `IdentityDbContext` so Identity's standard tables were still available, while our application-specific data was added:

```csharp
public class ApplicationDbContext
   : IdentityDbContext<ApplicationUser, IdentityRole, string>
{
   public DbSet<RefreshToken> RefreshTokens => Set<RefreshToken>();

   protected override void OnModelCreating(ModelBuilder builder)
   {
      base.OnModelCreating(builder);

      builder.Entity<ApplicationUser>()
         .HasIndex(user => user.TenantId);

      builder.Entity<RefreshToken>()
         .HasIndex(token => token.Token)
         .IsUnique();
   }
}
```

This produced the normal Identity tables such as `AspNetUsers`, `AspNetRoles`, and `AspNetUserClaims`, plus our `RefreshTokens` table.

### 4. Configure Identity and JWT authentication

In `Program.cs`, we registered the custom user and database context, then configured password and lockout rules:

```csharp
builder.Services.AddIdentity<ApplicationUser, IdentityRole>(options =>
{
   options.Password.RequiredLength = 12;
   options.Password.RequireDigit = true;
   options.Lockout.MaxFailedAccessAttempts = 5;
})
.AddEntityFrameworkStores<ApplicationDbContext>()
.AddDefaultTokenProviders();

builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
   .AddJwtBearer(options =>
   {
      options.Authority = configuration["Jwt:Authority"];
      options.Audience = configuration["Jwt:Audience"];
      options.RequireHttpsMetadata = true;
   });
```

The exact values came from environment-specific configuration, not hardcoded secrets.

### 5. Add tenant and permission claims to the JWT

When the user logged in, the token service added claims such as:

```csharp
new Claim("userId", user.Id),
new Claim("tenantId", user.TenantId.ToString()),
new Claim(ClaimTypes.Role, "SchoolAdmin"),
new Claim("permission", "fee.write")
```

The API then used those claims in authorization policies and service logic. For example, a fee endpoint could require the `fee.write` permission and also verify that the request's school matches the user's `tenantId`.

### 6. Keep Identity's standard features

Customization did not mean rewriting Identity. We continued to use its built-in support for password hashing, login, roles, claims, account lockout, password reset, and email confirmation. We extended it only where the platform had additional business requirements.

### Interview-ready answer

> **We customized ASP.NET Identity because the default user model did not support our multi-tenant school platform completely. We created an `ApplicationUser` derived from `IdentityUser` and added `FirstName`, `LastName`, `TenantId`, and refresh tokens. We added a `RefreshToken` entity and configured a custom `IdentityDbContext` with indexes and relationships. In `Program.cs`, we registered Identity with Entity Framework stores, configured password and lockout rules, and added JWT Bearer authentication. During login, we included the user ID, tenant ID, role, and permissions in the JWT. The API used those claims to enforce tenant isolation and authorization. We kept Identity's built-in password hashing, role management, lockout, and password reset features instead of rewriting them.**

---

## Where do you store database passwords? Why not store them in appsettings.json?

Database passwords and other secrets should be stored in **Azure Key Vault**, not directly in `appsettings.json`.

### Why not store passwords in appsettings.json?

`appsettings.json` is normally part of the application source code or deployment package. If a password is placed there:

- It may be committed to Git accidentally.
- It may appear in a pull request, build log, or artifact.
- Developers or other users with repository access may see it.
- Rotating the password requires changing and redeploying application configuration.
- The same secret may be copied across development, test, and production environments.

Even if the file is not committed, it can still be exposed through a deployment package or incorrect access permissions.

### Safer approach with Azure Key Vault

The application stores only a reference or configuration name, while the actual password remains in Key Vault.

```text
Azure Function
   ↓
Managed Identity
   ↓
Azure Key Vault
   ↓
Database password
   ↓
Oracle / SQL connection
```

The Function App uses a **managed identity** to access Key Vault. The identity is given the minimum permission required to read the specific secret. No password or Key Vault access key needs to be hardcoded in the application.

### What would be in appsettings.json?

Non-sensitive configuration can be stored there, for example:

```json
{
  "Database": {
    "Server": "database-server-name",
    "Name": "BatchDatabase"
  },
  "KeyVault": {
    "SecretName": "Oracle-Connection-Password"
  }
}
```

The actual password should remain in Key Vault. In local development, we can use **User Secrets** or environment variables instead of placing the password in `appsettings.json`.

### Interview-ready answer

> **We stored database passwords in Azure Key Vault and accessed them using the Function App's managed identity. We did not store them in `appsettings.json` because that file can be committed to source control, copied into deployment artifacts, or exposed to people who should not see production secrets. `appsettings.json` can contain non-sensitive configuration or a secret reference, while the actual password remains protected in Key Vault.**

---

## How does Azure Function access Key Vault? What is Managed Identity?

### Simple explanation

The Azure Function does not log in to Key Vault with a username and password. Azure gives the Function App an identity, and Key Vault trusts that identity.

```text
Azure Function
   ↓
Managed Identity requests an Entra ID token
   ↓
Azure Key Vault verifies the token
   ↓
Key Vault checks the identity's permissions
   ↓
Secret is returned to the Function
```

### What is Managed Identity?

**Managed Identity** is an identity created and managed by Azure for an Azure resource, such as a Function App. It allows the application to authenticate to other Azure services without storing another password, client secret, or certificate in the code.

Think of it as an Azure-managed identity card for the Function App.

There are two common types:

- **System-assigned identity:** Created together with the Function App and deleted when the resource is deleted.
- **User-assigned identity:** Created as a separate Azure resource and reusable by multiple applications.

### How the access works in practice

1. Enable a managed identity on the Function App.
2. Give that identity permission to read secrets in Key Vault, using Azure RBAC or a Key Vault access policy.
3. Store the database password as a secret in Key Vault.
4. The Function uses the Azure SDK or a Key Vault configuration reference.
5. Azure authenticates the Function through its managed identity, and Key Vault returns the secret only if the identity has permission.

The permission should follow **least privilege**. For example, the Function may be allowed to read secrets but not delete or modify them.

### Interview-ready answer

> **The Azure Function accesses Key Vault using its managed identity. The identity obtains an Entra ID access token, and Key Vault uses that identity to check whether it has permission to read the required secret. If permission is granted, Key Vault returns the database password to the application. This avoids storing a username, password, or client secret in the code or in `appsettings.json`.**

---

## If multiple batch projects use the same Key Vault, is the identity system-assigned or user-assigned?

Using one shared Key Vault does **not** automatically tell us which type of managed identity is used. The identity type depends on how the Function Apps were configured.

### Option 1: Separate system-assigned identity for each Function App

Each Function App has its own Azure-managed identity:

```text
Batch Function A ── System-assigned identity A ──┐
Batch Function B ── System-assigned identity B ──┼── Shared Key Vault
Batch Function C ── System-assigned identity C ──┘
```

The shared Key Vault grants each Function App identity permission to read only the secrets it needs. This gives better isolation and is often the safer default.

### Option 2: One user-assigned identity shared by multiple Function Apps

A separate user-assigned identity is created and attached to several Function Apps:

```text
Batch Function A ──┐
Batch Function B ──┼── Shared user-assigned identity ── Shared Key Vault
Batch Function C ──┘
```

This can be useful when the same identity must be reused across applications or recreated independently from the Function Apps. However, all attached applications share that identity's permissions, so its access must be designed carefully.

### What can we confirm from the batch project documentation?

The available project documentation shows `DefaultAzureCredential` and Managed Identity support, but it does not show an infrastructure declaration such as `SystemAssigned`, `UserAssigned`, or a user-assigned identity client ID. Therefore, we should not claim the exact type without checking the Azure Function App or deployment configuration.

### How would we verify it?

Check the Function App in Azure:

- **Identity > System assigned:** If enabled, the Function App has its own system-assigned identity.
- **Identity > User assigned:** If an identity is listed, the Function App uses a user-assigned identity.
- Check the deployment template, Bicep, Terraform, or Helm values for `SystemAssigned`, `UserAssigned`, `userAssignedIdentities`, or a managed identity client ID.

### Interview-ready answer

> **All batch projects can use the same Key Vault with either identity model. Sharing the vault does not mean the identity must be user-assigned. Each Function App could have its own system-assigned identity, and the shared Key Vault would grant access to each one. A user-assigned identity would be used only if the same identity was deliberately attached to multiple Function Apps. In our available project documentation, I can confirm Managed Identity support, but I cannot confirm the exact type without checking the Azure or deployment configuration.**

---

## How do you rotate secrets?

Secret rotation means replacing an old password, connection string, API key, or certificate with a new one before the old value expires or becomes unsafe.

### Basic rotation process

1. Generate a new strong secret.
2. Update the database or external system with the new secret.
3. Store the new value in Azure Key Vault under the same secret name. Key Vault creates a new secret version.
4. Allow the Function App to load the latest version.
5. Run a health check or test batch to confirm that the Function can connect successfully.
6. Revoke or disable the old secret after all applications are using the new value.
7. Record the rotation and monitor for authentication failures.

```text
Old secret
   ↓
Generate new secret
   ↓
Update database / external system
   ↓
Store new Key Vault secret version
   ↓
Function loads the new value
   ↓
Test connection
   ↓
Revoke old secret
```

### Important points

- The Key Vault secret name can remain the same while its value and version change.
- The Function's Managed Identity normally does not change during secret rotation. It still provides access to Key Vault; only the stored secret value changes.
- The application should not log the secret, connection string, or token during rotation.
- Configuration refresh or a Function restart may be needed, depending on how the application loads Key Vault values.
- For zero-downtime rotation, the database or external service may temporarily accept both the old and new credentials. After all applications are updated and tested, revoke the old credential.
- Rotation should be automated with a scheduled process or Key Vault rotation policy where supported, rather than relying only on manual updates.

### Interview-ready answer

> **We rotate secrets by generating a new value, updating the database or external system, and storing the new value in Azure Key Vault as a new version under the same secret name. The Function continues to use its Managed Identity to access Key Vault, so the identity itself does not need to change. After refreshing the application configuration, we run a health check or test batch, monitor for failures, and then revoke the old secret. We never write the secret value to logs or source control.**

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

---

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

- VM

- Kubernetes

- App Service

- Container

- etc.

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

| Requirement | Possible choice |

| --------------------------------------- | ----------------- |

| Client request/response | Web API |

| Continuously running background process | Worker Service |

| Event/scheduled/serverless workload | Azure Function |

| Complex multi-step serverless workflow | Durable Functions |

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

---

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

## Suppose the batch takes 2 hours. What happens if the Function execution is interrupted?

---

## Q3: Suppose Your Batch Processes 100,000 Records and Fails After 60,000. How Would You Resume It?

> "Suppose your batch processes 100,000 records and fails after processing 60,000. How would you resume it?"

### Simple Answer

I would not restart the whole batch from the beginning.

I would design the batch with a **checkpoint/resume mechanism**.

### Did Our Project Already Handle This?

In our Bupa batch project, we clearly handled **job-level tracking**, but I should be careful not to overclaim **record-level resume** unless I have seen that exact implementation.

What the project had:

- Durable Function `runId` to track one job execution
- Status endpoint to check whether the job is Running, Completed, or Failed
- Azure Storage/Azurite behind Durable Functions to store orchestration state
- SQL/Application Insights logging with `RunId`, `JobRequestId`, and status
- Summary logs like `Processed=X; Failed=Y; Total=Z` for investigation

So if the Function host restarts, Durable Functions can remember the **workflow state** and continue/replay the orchestration.

But that is not exactly the same as saying:

```text
Record 60,000 completed
Restart from record 60,001
```

For that exact behavior, the processor or database procedure must save record-level progress, such as:

- each record status: Pending / Processed / Failed
- last processed record ID
- batch chunk status
- output already generated flag
- retry count / error reason

So the honest interview answer is:

> In our project, Durable Functions gave us job-level durability and status tracking using `runId`, and our logging helped identify processed/failed counts. For true record-level resume, the business processor or database layer should maintain record status or checkpoint information. If that was not already present, I would add a tracking table and make processing idempotent so the restarted job can safely continue from the remaining records.

That means after processing a set of records successfully, the job saves its progress somewhere permanent, usually in a database or job status table.

So if the batch fails after 60,000 records, the system already knows:

```text
Total records       = 100,000
Processed records   = 60,000
Pending records     = 40,000
Next record to start = 60,001
```

When the job is restarted, it reads the last saved checkpoint and continues from the remaining records.

```text
Start batch
    ↓
Process records 1 to 10,000
    ↓
Save checkpoint: 10,000 completed
    ↓
Process records 10,001 to 20,000
    ↓
Save checkpoint: 20,000 completed
    ↓
...
    ↓
Failure after 60,000
    ↓
Restart job
    ↓
Read checkpoint: 60,000 completed
    ↓
Continue from 60,001
```

### How It Works in Practice

Instead of treating 100,000 records as one huge operation, I would process them in smaller chunks.

For example:

```text
Batch 1  → records 1 to 10,000
Batch 2  → records 10,001 to 20,000
Batch 3  → records 20,001 to 30,000
...
```

After each chunk completes successfully, I update a tracking table.

Example tracking table:

| JobId             | LastProcessedRecord | Status |
| ----------------- | ------------------- | ------ |
| TaxJob-2026-09-11 | 60000               | Failed |

When the job runs again, it checks this table first.

If `LastProcessedRecord = 60000`, then the next query should pick records greater than 60,000.

```sql
SELECT *
FROM TaxRecords
WHERE RecordId > 60000
ORDER BY RecordId
```

### Very Important Point: Idempotency

The batch should also be **idempotent**.

In simple terms, idempotent means:

> If the same record is processed again by mistake, it should not create duplicate output or corrupt data.

For example, before generating a tax statement, I can check:

```text
Has this record already been processed?
    ↓
Yes → skip it
No  → process it
```

This protects us if the job fails after processing a record but before updating the checkpoint.

### Interview-Ready Answer

> If a batch has 100,000 records and fails after 60,000, I would resume it using checkpointing. The job should save progress after each successful chunk or record, for example in a job tracking table. When the job restarts, it reads the last successful checkpoint and continues from record 60,001 instead of starting from record 1 again. I would also make the processing idempotent, so even if a few records are retried, duplicate records or duplicate files are not created.

### One-Line Answer

> Save progress regularly, restart from the last successful checkpoint, and make each record safe to retry.

## **Unexpected Crash** ⚠️ — Your Server Dies

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

## Why did you use both Serilog and Application Insights? Aren't they doing the same thing?

No — they are related, but **they are not the same thing**. Think of them as solving two different parts of observability.

## 1. What is Serilog?

**Serilog is a logging library.**

Your application uses Serilog to **create structured log messages**.

For example:

```csharp
Log.Information(
    "Started processing batch {BatchId}",
    batchId);
```

It can produce information like:

```text
Started processing batch B123
```

or structured data:

```text
BatchId = B123
Status  = Started
```

So think:

> **Serilog = How my application creates/writes logs.**

---

## 2. What is Application Insights?

**Application Insights is an Azure monitoring/telemetry service.**

It helps you collect and investigate things such as:

```text
Requests
Exceptions
Dependencies
Performance
Traces
Failures
Execution duration
```

You can then go into Azure and investigate:

```text
Application Insights
       ↓
Search logs
       ↓
Find failed Function
       ↓
See exception
       ↓
See database dependency
       ↓
Check execution time
```

So think:

> **Application Insights = Where Azure collects and helps you analyze telemetry.**

---

# 3. How can they work together?

This is the important part.

You can have:

```text
Your Azure Function
       │
       ↓
    Serilog
       │
       ↓
Structured log
       │
       ↓
Application Insights
       │
       ↓
Azure monitoring
```

So they're **not necessarily competing tools**.

One is primarily a **logging framework**, while the other is a **monitoring/telemetry platform**.

---

# 4. Real example from your batch

Suppose your batch starts:

```text
Batch ID = B123
```

You might log:

```csharp
Log.Information(
    "Batch {BatchId} started",
    batchId);
```

Then during processing:

```csharp
Log.Information(
    "Processing {RecordCount} records for batch {BatchId}",
    count,
    batchId);
```

If something fails:

```csharp
Log.Error(
    exception,
    "Batch {BatchId} failed",
    batchId);
```

Those logs can be collected into your monitoring setup.

Then in **Application Insights**, you can investigate the execution and correlate the telemetry with the Function execution and dependencies.

---

# 5. Why would a team want Serilog?

Because you want your application's logs to be:

- Consistent
- Structured
- Searchable
- Useful for troubleshooting

Instead of:

```text
Something went wrong
```

you want:

```text
BatchId: B123
Step: CustomerProcessing
RecordId: 45678
Error: Oracle timeout
Duration: 120 seconds
```

That makes production troubleshooting much easier.

---

# 6. Why Application Insights then?

Imagine your Bupa batch normally takes:

```text
20 minutes
```

Today:

```text
2 hours
```

You need to investigate.

Application Insights can help you look at:

```text
Function execution
       ↓
Duration
       ↓
Exceptions
       ↓
Dependency calls
       ↓
Database calls
       ↓
Failures
```

Then your application logs can give you the **business-specific details**.

For example:

```text
Application Insights
       │
       ├── Function took 2 hours
       ├── Oracle dependency was slow
       └── Exception occurred

Serilog logs
       │
       ├── Batch ID = B123
       ├── Step = GenerateReport
       ├── Records = 100,000
       └── Processing reached record 60,000
```

Together, they give you a much better picture.

What exactly would you look at in Application Insights?

Let's understand this like a beginner first.

## What is the difference between logs, metrics, and traces?

These are three different ways to understand what is happening inside an application.

### 1. Logs: What happened?

Logs are written messages that describe an event or error.

Example:

```text
Batch B123 started processing 100,000 records
Oracle connection failed
Batch B123 completed with 60,000 records processed
```

Logs give you detailed context, such as the batch ID, record ID, error message, and processing step.

### 2. Metrics: How much or how often?

Metrics are numbers that show the health or performance of the system.

Example:

```text
Batch duration       = 2 hours
Records processed    = 60,000
Failed executions    = 3
Memory usage         = 75%
```

Metrics help you notice trends and unusual behavior quickly, such as a batch that normally takes 20 minutes but suddenly takes 2 hours.

### 3. Traces: Where did the request travel?

A trace follows one request or batch operation across multiple components. It shows the path and the time spent at each step.

Example:

```text
Function
    ↓ 2 seconds
Business service
    ↓ 10 minutes
Oracle database
    ↓ 1 hour 40 minutes
SQL database
    ↓ 5 minutes
```

Traces help identify which service or dependency made the overall operation slow or caused it to fail.

### Simple way to remember

```text
Logs    = What happened?
Metrics = How much, how often, or how fast?
Traces  = Where did the operation go?
```

### Interview-ready answer

> **Logs give detailed information about events and errors. Metrics are numbers that show system health and performance, such as duration, failure count, or memory usage. Traces follow one request across services and dependencies, helping us find where time was spent or where the failure occurred.**

# 🔍 What exactly would you look at in Application Insights?

Imagine your batch normally takes **20 minutes**, but today someone tells you:

> "Vasanth, today's batch took 2 hours. Find out why."

You open **Application Insights**. You don't just look at one screen. You investigate different types of telemetry.

---

## 1️⃣ First — Did the Function run successfully?

I'd first check:

```text
Function executions
       ↓
Success / Failed
       ↓
Duration
```

For example:

```text
Batch-Job
──────────────
Status: Failed ❌
Duration: 1h 58m
```

Now I know there is a problem.

---

## 2️⃣ Exceptions 🚨

Next I'd check **Exceptions**.

For example:

```text
Exception
   ↓
OracleException
   ↓
Connection timeout
```

This gives me an initial clue about **what went wrong**.

---

## 3️⃣ Logs / Traces 📝

Then I'd look at the application logs.

This is where your structured Serilog logging becomes useful.

For example:

```text
Batch started
Batch ID: B123

Fetching records...
100,000 records found

Processing records...
60,000 completed

Calling Oracle...
⛔ Timeout

Batch failed
```

Now I have much more context than just:

```text
Function failed
```

---

## 4️⃣ Dependencies 🔗

This is **very important for your Bupa project** because your Function communicates with databases/external systems.

I'd check:

```text
Function
   ↓
Dependencies
   ├── Oracle
   ├── SQL
   └── Other external calls
```

I want to know:

> **Which dependency was slow or failed?**

For example:

```text
Oracle
────────────
Duration: 1h 40m
Status: Timeout
```

That immediately gives me a strong lead.

---

## 5️⃣ Performance / Duration ⏱️

I'd compare the current execution with previous executions.

For example:

```text
Yesterday:  20 min
Monday:     22 min
Tuesday:    21 min
Today:      2 hours  ❌
```

Then I'd investigate **where the extra time was spent**.

Maybe:

```text
Function execution
       ↓
Business processing     10 min
Oracle call             1h 40 min  ❌
SQL processing           5 min
Other                    3 min
```

Now I know the Oracle interaction is the area to investigate.

---

# 6️⃣ End-to-end correlation

This is another useful concept.

Suppose your batch has:

```text
Batch ID = B123
```

You can use that identifier in your logs to follow the same batch through different processing steps.

```text
Batch B123
    ↓
Function started
    ↓
Oracle call
    ↓
Processing
    ↓
SQL update
    ↓
Function completed
```

That's why **structured logging** is valuable.

---

# 🧠 So what would YOU say in an interview?

Don't try to list 20 Application Insights features.

Think in this order:

```text
1. Did the Function execute?
          ↓
2. Did it succeed or fail?
          ↓
3. What exception occurred?
          ↓
4. What do the application logs say?
          ↓
5. Which dependency was involved?
          ↓
6. How long did each operation take?
          ↓
7. Can I correlate the complete batch execution?
```

### 🎤 Simple interview answer

> **"If a batch has a problem, I would first check the Function execution status and duration. Then I'd check exceptions and application traces to understand what happened. Since our batch interacts with Oracle and SQL, I'd also check the dependency telemetry to see whether a database or external call was slow or failed. I'd then correlate those logs using the batch or correlation ID to understand the complete execution flow and identify where the problem occurred."**

## How would you investigate a production batch that suddenly takes 3 hours instead of 20 minutes?

I would investigate it step by step, starting with the overall execution and then narrowing down to the slow component.

### Troubleshooting path

1. **Confirm the execution details**

   I would check the Function execution status, start time, end time, duration, and whether the batch completed, failed, or is still running.

2. **Compare it with a normal run**

   I would compare today's execution with previous runs. For example, if the batch normally takes 20 minutes but today takes 3 hours, I would identify when the slowdown started and whether it affects every run or only this batch.

3. **Check exceptions and application logs**

   I would review Application Insights exceptions and Serilog logs using the batch ID or correlation ID. I would look for timeout messages, retry loops, blocked processing, unusually large input, or a particular record that caused the delay.

4. **Check dependency duration**

   I would inspect Oracle, SQL, and other external dependency telemetry. I would compare the time spent in each call to determine whether the delay is in business processing, an Oracle stored procedure, a SQL query, or an external service.

5. **Check retries and waiting time**

   A batch may take longer because an operation is failing and being retried repeatedly. I would check retry counts, timeout settings, queue delays, connection-pool waits, and whether the application is waiting for a lock or a response.

6. **Check infrastructure and network changes**

   If the code and database look healthy, I would investigate recent changes to the Function App, AKS, Envoy/Istio, firewall, VPN, DNS, or database connectivity. I would also check CPU, memory, scaling, and platform alerts.

7. **Compare with a controlled test**

   I would run a smaller or equivalent operation, where safe, and compare the result from the production path with another known-good path. This helps separate an application problem from a database or network problem.

### Example conclusion

If the Function normally completes in 20 minutes, but today's telemetry shows that the Oracle call takes 2 hours and the connection closes at approximately 3600 seconds, I would investigate the network path and proxy timeout rather than immediately changing the business code.

### Interview-ready answer

> **I would start by checking the Function execution duration and comparing it with previous successful runs. Then I would review exceptions, Serilog logs, and correlation IDs to understand what happened during this specific batch. I would inspect dependency telemetry to see whether Oracle, SQL, or another external service consumed the extra time. I would also check retries, timeouts, resource usage, and recent infrastructure or network changes. Once I identify the slow component, I would reproduce or test that part in isolation and then apply a targeted fix.**

## Tell me one production issue you personally faced in this project and how you solved it

Yes. For this one, you should use the **network/long-running database connection issue you actually described**, because it gives you a realistic production story and connects directly to Azure Functions, Oracle, Application Insights, and troubleshooting.

## 🎤 Interviewer

> **"Tell me one production issue you personally faced in this project and how you solved it."**

Yes. I remember the **exact 3600-second Oracle failure** from your Bupa batch project. More importantly, I checked the uploaded interview document again so we keep the answer consistent with your project evidence.

## 🔴 The problem

You had a **long-running `LMembershipCriteriaJob`**.

The flow was:

```text
Control-M
    ↓
Azure Function
    ↓
Durable Function
    ↓
Batch Processor
    ↓
Oracle Stored Procedure
```

The stored procedure was taking **more than 1 hour**.

But the job was consistently failing at approximately:

```text
3600 seconds
     ↓
1 hour
     ↓
ORA-12537: TNS: connection closed
```

The important clue was:

> **It was failing at almost exactly 3600 seconds every time.**

That strongly suggested a **configured timeout**, rather than a random application or database failure.

---

# 🔍 How we investigated it

We didn't immediately say:

> "Oracle is timing out."

We checked each layer.

### 1️⃣ Check Azure Function timeout

First question:

> Is Azure Function itself killing the job after one hour?

We increased the Function timeout to allow the job to run for several hours.

For example:

```text
Function timeout
       ↓
6 hours
```

But the job **still failed around 3600 seconds**.

So:

```text
Azure Function timeout
        ↓
❌ Not the root cause
```

The Function host was still alive.

---

### 2️⃣ Check Oracle command timeout

Next:

> Is our application cancelling the Oracle command?

We increased the Oracle command timeout to:

```text
7200 seconds
= 2 hours
```

But the connection was still being closed around one hour.

So:

```text
Oracle command timeout
        ↓
❌ Not the root cause
```

---

### 3️⃣ Test the same operation locally

This was the **big clue**.

We ran the same application/database operation from the local environment.

```text
Local machine
      ↓
Same Oracle DB
      ↓
Stored procedure
      ↓
Runs beyond 1 hour ✅
```

But:

```text
Azure / AKS
      ↓
Same Oracle DB
      ↓
Around 3600 sec
      ↓
Connection closed ❌
```

So we now knew:

> **The stored procedure itself was capable of running longer than one hour.**

And the problem was specific to the **Azure/AKS path to Oracle**.

---

# 🔎 4️⃣ Investigate the network path

Then we looked at:

```text
Azure/AKS
     ↓
Network
     ↓
Oracle
```

We discovered that the application traffic was passing through the **Istio Envoy sidecar**.

Think of Envoy as a proxy sitting between your application and Oracle:

```text
Application
     ↓
Envoy Sidecar
     ↓
Network
     ↓
Oracle
```

And the important finding was:

> **Envoy had a default 1-hour TCP idle timeout.**

So our long-running Oracle connection was being dropped by Envoy after approximately:

```text
3600 seconds
```

That explained the:

```text
ORA-12537: TNS: connection closed
```

## Follow-up: How did you find that?

We followed the request path layer by layer instead of assuming that Oracle was the problem.

### Troubleshooting path

1. **Collect the failure pattern**

   From Application Insights and application logs, we compared several executions. The failure happened at almost exactly **3600 seconds**, and the error was `ORA-12537: TNS: connection closed`. A repeatable one-hour cutoff suggested an infrastructure timeout rather than a random database failure.

2. **Check the Azure Function timeout**

   We confirmed that the Function App timeout was configured to allow the batch to run for several hours. The host was still running when the Oracle connection was closed, so the Function timeout was not the cause.

3. **Check the Oracle command timeout**

   We verified that the Oracle command timeout was longer than one hour, for example **7200 seconds**. The application was not cancelling the command at 3600 seconds, so the client-side command timeout was also ruled out.

4. **Run the same operation outside Azure/AKS**

   We ran the same application operation against the same Oracle database from a local environment. It continued beyond one hour, which showed that the stored procedure and Oracle database could run for that duration. The difference was the network path used by Azure/AKS.

5. **Trace the AKS network path**

   We checked the pod configuration and service-mesh setup and confirmed that Istio sidecar injection was enabled. The application traffic was therefore being redirected through the Envoy sidecar before reaching Oracle.

6. **Compare the timeout with the proxy configuration**

   We reviewed the Istio/Envoy proxy configuration and found a one-hour TCP idle timeout. Its value matched the repeated failure time of approximately 3600 seconds. This connected the evidence: the database was still processing, but Envoy was closing the apparently idle connection.

7. **Confirm the diagnosis with a controlled change**

   We excluded the Oracle ports from Envoy interception and reran the batch. The long-running Oracle operation completed successfully, confirming that the Envoy timeout on that network path was the root cause.

### Interview-ready follow-up answer

> **I found it by isolating each layer. First, I noticed from the logs that the job failed at almost exactly 3600 seconds with `ORA-12537`, which suggested a timeout. I verified that neither the Function timeout nor the Oracle command timeout was set to one hour. The same operation worked locally against the same Oracle database for more than an hour, so the database procedure itself was not the problem. We then checked the AKS pod and found that Istio was routing the Oracle traffic through the Envoy sidecar. The Envoy TCP idle timeout matched the 3600-second failure. After excluding the Oracle ports from sidecar interception, the batch completed successfully, confirming the root cause.**

---

# 🛠️ 5️⃣ The actual fix

We changed the Kubernetes deployment configuration.

We made this change in the Kubernetes deployment manifest, usually the `deployment.yaml` file. If the project uses Helm, the annotation may instead be maintained in the chart's `values.yaml` file and rendered into the deployment manifest.

### What is Helm?

**Helm** is a package manager and templating tool for Kubernetes. It helps teams define, configure, and deploy Kubernetes resources as a reusable **chart** instead of maintaining every YAML file manually.

For example, a Helm chart may contain:

- `values.yaml` for environment-specific settings
- `templates/deployment.yaml` for the Kubernetes deployment template
- Other templates for services, configuration, secrets, and ingress

The deployment value is supplied through `values.yaml`, and Helm combines it with the template to generate the final Kubernetes manifest. Therefore, in a Helm-based project, we would normally update the chart values or deployment template rather than editing the generated YAML directly.

We added:

```yaml
podAnnotations:
  traffic.sidecar.istio.io/excludeOutboundPorts: "1521,1530,1624"
```

The important part is:

```yaml
excludeOutboundPorts
```

This tells Istio:

> **Do not intercept traffic going to these Oracle-related ports through the Envoy sidecar.**

So before:

```text
Application
     ↓
Envoy
     ↓
Oracle
```

After:

```text
Application
     ↓
Oracle
```

The Oracle traffic bypasses Envoy.

Therefore the **Envoy 1-hour TCP timeout no longer affects that Oracle connection**.

### What is Envoy and why is it used?

**Envoy** is a high-performance proxy that runs alongside an application as a sidecar. Instead of the application connecting directly to another service, Envoy can handle the network traffic on its behalf.

Teams use Envoy through Istio to provide common networking features without adding that logic to every application, such as:

- Traffic routing and service discovery
- Retries, timeouts, and circuit breaking
- Mutual TLS and service-to-service security
- Metrics, tracing, and request logging

In this incident, Envoy was useful for managing and observing service traffic, but its default TCP idle timeout was unsuitable for the long-running Oracle connection. That is why the Oracle ports were excluded from Envoy interception.

---

# 🧠 Very simple way to remember the whole story

```text
PROBLEM
   ↓
Oracle stored procedure takes > 1 hour
   ↓
Job fails at exactly ~3600 seconds
   ↓
ORA-12537: TNS connection closed
```

Then:

```text
INVESTIGATION
   ↓
Function timeout? ❌
   ↓
Oracle command timeout? ❌
   ↓
Oracle itself? ❌
   ↓
Local test works > 1 hour
   ↓
Therefore investigate Azure/AKS network path
```

Then:

```text
ROOT CAUSE
   ↓
Oracle traffic was going through Envoy
   ↓
Envoy had 1-hour TCP idle timeout
```

Then:

```text
FIX
   ↓
Exclude Oracle ports from Envoy
   ↓
1521, 1530, 1624
   ↓
Oracle traffic bypasses Envoy
   ↓
Long-running connection is not dropped by Envoy
```

---

# 🎯 Interview answer — memorize this

If the interviewer asks:

### **"Tell me about a production issue you faced and how you solved it."**

Say:

> **"One challenging issue I faced was with a long-running batch job called LMembershipCriteriaJob. It was calling an Oracle stored procedure, and the job was consistently failing at around 3600 seconds with an ORA-12537 TNS connection closed error.**
>
> **I investigated it layer by layer. First, I increased the Azure Function timeout and confirmed that the Function host was still running, so it wasn't the Function timeout. Then I increased the Oracle command timeout to 7200 seconds, but the connection was still getting closed after around one hour.**
>
> **I then ran the same operation locally against the same Oracle database, and it worked beyond one hour. That helped us narrow the issue to the Azure/AKS network path.**
>
> **We found that the Oracle traffic was passing through the Istio Envoy sidecar, which had a one-hour TCP idle timeout. So we configured the Kubernetes pod annotation to exclude the Oracle-related ports 1521, 1530 and 1624 from Envoy sidecar interception. This allowed the Oracle traffic to bypass Envoy and prevented the one-hour TCP connection drop."**

### ⭐ The key sentence

If you remember only one thing:

> **"The Function and Oracle were still running, but the Envoy sidecar was closing the TCP connection at the one-hour mark, so we excluded the Oracle ports from Envoy interception."**

That is the **root cause → fix** you should be able to explain confidently.

---

Yes. Let's treat this as **we actually created a small POC to prove the Saga + Outbox approach**. I’ll explain the architecture, components, flow, failure handling, and what you would see during testing. No code.

> **Important:** Your OSP project material describes Saga, Outbox, event-driven communication, and idempotent handlers. The following is a POC-style explanation of how those pieces fit together, without claiming specific implementation details that aren't in the project file.

# 1. What are we trying to prove with the POC?

We want to prove this scenario:

```text
Student enrolls in Football
          ↓
Enrollment created
          ↓
Seat reserved
          ↓
Fee calculated
          ↓
Everything successful
```

And more importantly:

```text
Student enrolls
      ↓
Enrollment created       ✅
      ↓
Seat reserved            ✅
      ↓
Fee calculation          ❌
      ↓
Can we recover safely?
```

The POC demonstrates that **we can recover from a failure without leaving incorrect business data behind**.

---

# 2. POC architecture

We can keep the POC very small.

### Services

```text
┌──────────────────────┐
│ Enrollment Service   │
│ .NET Web API         │
└──────────┬───────────┘
           │
           │ HTTP / Event
           ↓
┌──────────────────────┐
│ Activity Service     │
│ .NET Web API         │
└──────────────────────┘

           │
           │ Event
           ↓
┌──────────────────────┐
│ Fee Service          │
│ .NET Web API         │
└──────────────────────┘
```

Each service has its own database:

```text
Enrollment Service → Enrollment DB

Activity Service   → Activity DB

Fee Service        → Fee DB
```

And we introduce a message broker:

```text
Services
   ↓
Message Broker
   ↓
Other Services
```

For the POC, you could use something like:

- .NET 8
- ASP.NET Core Web API
- EF Core
- SQL Server
- RabbitMQ or Azure Service Bus
- Docker
- Postman for testing

The project material lists RabbitMQ/Azure Service Bus as possible messaging infrastructure.

---

# 3. First POC — without Saga

Before implementing Saga, I'd actually demonstrate the problem.

Imagine we have:

```text
Enrollment DB
Activity DB
Fee DB
```

We call:

```text
POST /enroll
```

The flow is:

```text
Client
  ↓
Enrollment Service
  ↓
Create Enrollment
  ↓
Activity Service
  ↓
Reserve Seat
  ↓
Fee Service
  ↓
Calculate Fee
```

Now intentionally make Fee Service fail.

For example:

```text
Enrollment       ✅
Activity Seat    ✅
Fee              ❌
```

Look at the databases:

```text
Enrollment DB
-------------------
Enrollment = 1001


Activity DB
-------------------
Seat = Reserved


Fee DB
-------------------
Fee = Not Created
```

That's our problem.

The POC clearly demonstrates:

> **There is no single transaction that can rollback all three databases.**

---

# 4. Now introduce Saga

Now we introduce the Saga concept.

The Saga maintains the business workflow:

```text
Step 1
Create Enrollment

      ↓

Step 2
Reserve Seat

      ↓

Step 3
Calculate Fee
```

Every step is a **local transaction**.

Meaning:

```text
Enrollment Service
     ↓
BEGIN
Create enrollment
COMMIT
```

Then:

```text
Activity Service
     ↓
BEGIN
Reserve seat
COMMIT
```

Then:

```text
Fee Service
     ↓
BEGIN
Calculate fee
COMMIT
```

There is no global database transaction.

---

# 5. Who controls the Saga?

For understanding the POC, I recommend using an **Orchestrator**.

Think of the orchestrator as a coordinator.

```text
             Saga Orchestrator
                    |
       ┌────────────┼────────────┐
       ↓            ↓            ↓
 Enrollment      Activity       Fee
 Service         Service       Service
```

It basically says:

> "Do this first."

Then:

> "Okay, that succeeded. Now do this."

Then:

> "That succeeded too. Now do the next one."

---

# 6. Step-by-step successful flow

Let's say:

```text
StudentId = 501
ActivityId = 10
```

The client sends:

```text
Enroll Student 501
in Activity 10
```

---

## Step 1 — Create Saga

The orchestrator creates something like:

```text
SagaId = SAGA-1001
```

This ID is important because we can track the complete workflow.

Conceptually:

```text
SAGA-1001

Status = Started
```

---

# 7. Step 2 — Create Enrollment

Orchestrator tells Enrollment Service:

```text
Create enrollment
Student = 501
Activity = 10
SagaId = SAGA-1001
```

Enrollment Service performs its local transaction.

```text
Enrollment DB

EnrollmentId | Student | Activity | Status
------------------------------------------------
1001         | 501     | 10       | Pending
```

Transaction commits.

Then:

```text
Create Enrollment
       ↓
SUCCESS
```

---

# 8. Step 3 — Reserve activity capacity

Orchestrator now calls Activity Service:

```text
Reserve seat
Activity = 10
SagaId = SAGA-1001
```

Activity Service checks:

```text
Maximum capacity = 30
Current = 29
```

One seat is available.

It changes:

```text
Current = 30
```

and commits its local transaction.

```text
Reserve Seat
     ↓
SUCCESS
```

---

# 9. Step 4 — Calculate fee

Now orchestrator calls Fee Service:

```text
Calculate fee
Student = 501
Activity = 10
SagaId = SAGA-1001
```

Fee Service calculates:

```text
Activity fee = $100
```

and stores it.

```text
Calculate Fee
      ↓
SUCCESS
```

Now the Saga is complete.

```text
Enrollment     ✅
Seat           ✅
Fee            ✅
```

Final state:

```text
Enrollment = Confirmed
Seat = Reserved
Fee = $100
```

---

# 10. Now the important POC — force a failure

This is where the POC becomes useful.

We intentionally configure Fee Service:

```text
FAIL_FEE_SERVICE = true
```

Now run the exact same enrollment.

Flow:

```text
Create Enrollment
       ↓
      ✅
       ↓
Reserve Seat
       ↓
      ✅
       ↓
Calculate Fee
       ↓
      ❌
```

Now the Saga Orchestrator knows:

> Step 3 failed.

---

# 11. What does the Saga do now?

It doesn't execute:

```text
ROLLBACK
```

because there is no global transaction.

Instead, it performs **compensation**.

The orchestrator says:

```text
Fee failed
   ↓
Undo previous business operations
```

---

# 12. Compensation #1 — Release seat

Activity Service receives:

```text
Release seat
Activity = 10
SagaId = SAGA-1001
```

Before:

```text
Capacity = 30 / 30
```

After:

```text
Capacity = 29 / 30
```

So:

```text
Reserve Seat
     ↓
Release Seat
```

---

# 13. Compensation #2 — Cancel enrollment

Now Enrollment Service receives:

```text
Cancel enrollment
EnrollmentId = 1001
SagaId = SAGA-1001
```

Instead of deleting the record, normally we'd maintain the business history and change status:

```text
Pending
   ↓
Cancelled
```

So:

```text
Enrollment = Cancelled
```

---

# 14. Final state after failure

Now check all databases.

### Enrollment DB

```text
Enrollment 1001
Status = Cancelled
```

### Activity DB

```text
Seat = Available
```

### Fee DB

```text
Fee = Not Created
```

That's a valid business state.

So even though the overall operation failed:

> **We didn't leave a half-completed enrollment behind.**

That's the key thing we're proving with the POC.

---

# 15. Now where does the Message Broker come in?

Now we make the POC more realistic.

Instead of every operation being tightly coupled through HTTP, we can use events for asynchronous communication.

For example:

```text
Enrollment Service
       ↓
EnrollmentCreated
       ↓
Message Broker
       ↓
Fee Service
```

The broker could be:

```text
RabbitMQ
```

or:

```text
Azure Service Bus
```

The project architecture specifically uses event-driven communication for operations that don't require an immediate response.

---

# 16. Why do we need Outbox?

Here's another failure scenario.

Suppose Enrollment Service does this:

```text
Save Enrollment
      ↓
Publish Event
```

What if:

```text
Save Enrollment       ✅
Publish Event         ❌
```

Maybe RabbitMQ/Azure Service Bus is temporarily unavailable.

Now:

```text
Enrollment DB
   ↓
Enrollment exists

Message Broker
   ↓
Event never arrived
```

Other services don't know that enrollment was created.

---

# 17. Add Outbox table

We add an Outbox table inside Enrollment DB.

Conceptually:

```text
Enrollment DB
----------------------------

Enrollment
----------------------------
1001 | Student 501 | Football


Outbox
----------------------------
EventId
EventType
Payload
Status
CreatedAt
```

When enrollment is created, we save **both**:

```text
Enrollment
+
EnrollmentCreated event
```

inside the same local database transaction.

So:

```text
BEGIN TRANSACTION

Create Enrollment
       +
Create Outbox Event

COMMIT
```

Now either both succeed or both fail.

That's the important part.

---

# 18. Background publisher

Then we have a background process.

It continuously checks:

```text
Outbox
```

for unpublished events.

For example:

```text
Outbox

EventId = E1001
Status = Pending
```

Publisher picks it up:

```text
Outbox
   ↓
Publish to RabbitMQ
   ↓
SUCCESS
   ↓
Mark event as Published
```

If broker is down:

```text
Outbox
   ↓
Publish
   ↓
FAIL
   ↓
Keep Pending
   ↓
Retry later
```

So the event isn't lost.

The project POC material describes exactly this Outbox concept: store the event with the local transaction, then have a background process publish it with retry handling.

---

# 19. Now another problem — duplicate events

Suppose:

```text
Publisher
   ↓
Publish Event
   ↓
Message Broker receives it ✅
```

But before the publisher knows that it succeeded, there is a network problem.

It retries.

Now the broker may contain:

```text
EnrollmentCreated E1001
EnrollmentCreated E1001
```

Two messages.

Will Fee Service calculate the fee twice?

That's where **idempotency** comes in.

---

# 20. Idempotent consumer

Fee Service receives:

```text
EventId = E1001
```

It checks:

```text
Have I already processed E1001?
```

If:

```text
NO
```

then process it.

And record:

```text
ProcessedEvent
----------------
E1001
```

If the same event arrives again:

```text
E1001
```

Fee Service checks:

```text
Already processed?
```

Answer:

```text
YES
```

So it ignores the duplicate.

That's why:

```text
Outbox      → prevents lost events

Idempotency → handles duplicate events

Saga        → handles business failure
```

The project material explicitly calls out idempotent event handlers to prevent duplicate processing.

---

# 21. What tools would we use in this POC?

A realistic simple POC could look like:

```text
.NET 8
   ↓
ASP.NET Core Web API
   ↓
EF Core
   ↓
SQL Server
```

Messaging:

```text
RabbitMQ
```

or:

```text
Azure Service Bus
```

Local environment:

```text
Docker
```

Testing:

```text
Postman
```

Monitoring/debugging:

```text
Serilog
Application Insights
```

The actual project architecture lists these types of technologies and patterns, although the exact POC implementation technology can vary.

---

# 22. What would we actually test in Postman?

### Test 1 — Successful enrollment

```text
POST /enroll
```

Expected:

```text
Enrollment created
Seat reserved
Fee calculated
Status = Confirmed
```

---

### Test 2 — Fee Service failure

Configure:

```text
Fee Service = unavailable
```

Call:

```text
POST /enroll
```

Expected:

```text
Enrollment created       ✅
Seat reserved            ✅
Fee calculation          ❌
       ↓
Release seat             ✅
Cancel enrollment        ✅
```

Final:

```text
Enrollment = Cancelled
Seat = Available
Fee = Not Created
```

---

### Test 3 — Message broker failure

Stop RabbitMQ.

Then:

```text
Create Enrollment
```

Expected:

```text
Enrollment DB
     ↓
Enrollment created

Outbox
     ↓
Event stored as Pending
```

Start RabbitMQ again.

Publisher retries:

```text
Pending event
     ↓
RabbitMQ
     ↓
SUCCESS
```

---

### Test 4 — Duplicate event

Send the same event twice:

```text
E1001
E1001
```

Expected:

```text
First E1001
   ↓
Process

Second E1001
   ↓
Already processed
   ↓
Ignore
```

No duplicate fee.

---

# 23. The complete POC picture

This is the mental model I want you to remember:

```text
                         Client
                           |
                           ↓
                  Saga Orchestrator
                           |
             ┌─────────────┼─────────────┐
             ↓             ↓             ↓
       Enrollment       Activity         Fee
        Service          Service        Service
             |             |              |
             ↓             ↓              ↓
        Enrollment      Activity         Fee
             DB             DB             DB
             |
             ↓
          Outbox
             |
             ↓
      Message Broker
             |
       ┌─────┴─────┐
       ↓           ↓
 Activity        Fee
 Service        Service
```

And the failure path:

```text
Enrollment
    ↓
   ✅
    ↓
Activity
    ↓
   ✅
    ↓
Fee
    ↓
   ❌
    |
    ↓
Saga Compensation
    |
    ├── Release Activity Seat
    |
    └── Cancel Enrollment
```

---

# 24. What is the actual problem each pattern solves?

This is the **most important part to memorize**:

| Problem                                                      | Pattern                        |
| ------------------------------------------------------------ | ------------------------------ |
| Multiple databases cannot share one transaction              | **Saga**                       |
| Previous successful operations need to be reversed           | **Compensating Transaction**   |
| Event could be lost between DB update and message publishing | **Outbox**                     |
| Same event could arrive multiple times                       | **Idempotency**                |
| Services shouldn't be tightly coupled                        | **Event-driven communication** |
| Need to coordinate multiple steps                            | **Saga Orchestrator**          |

### One-line mental model

> **Saga manages the workflow, compensation handles failure, Outbox makes events reliable, and idempotency makes retries safe.**

That's the complete story.

---

## How does Entra ID issue a JWT?

Entra ID issues a JWT after an application or user successfully authenticates and requests an access token for a specific API.

### Simple flow

```text
User or application
   ↓
Sends authentication request to Entra ID
   ↓
Entra ID verifies the identity
   ↓
Entra ID checks permissions and requested API scope
   ↓
Entra ID creates and signs a JWT
   ↓
JWT is returned to the client
   ↓
Client sends JWT to the API
```

### Step-by-step explanation

1. The client is registered in Entra ID and has a client ID. For a confidential application, it also has a secure credential such as a certificate or client secret.
2. The client authenticates with Entra ID using a supported OAuth 2.0 flow, such as Authorization Code flow for a user or Client Credentials flow for service-to-service communication.
3. The client requests an access token for a target API by specifying its scope or application permission.
4. Entra ID validates the identity and checks whether it is allowed to call that API.
5. Entra ID creates the JWT payload with claims such as issuer, audience, subject, tenant, permissions or scopes, and expiry time.
6. Entra ID signs the JWT with its private signing key and returns it to the client.
7. The client sends the token in the request header:

```http
Authorization: Bearer <access-token>
```

8. The API validates the token's signature, issuer, audience, expiry, and required scope or role. The API does not need to call Entra ID for every request because it can validate the signature using Entra ID's published public keys.

### What is inside a JWT?

A JWT has three parts separated by dots:

```text
Header.Payload.Signature
```

- **Header:** Identifies the token type and signing algorithm.
- **Payload:** Contains claims such as `iss`, `aud`, `exp`, `tid`, `scp`, or `roles`.
- **Signature:** Proves that the token was issued by Entra ID and was not changed.

The JWT is encoded, not encrypted. Therefore, sensitive passwords or secrets should never be placed in its claims.

### Interview-ready answer

> **The client authenticates with Entra ID and requests an access token for a specific API. Entra ID validates the identity and permissions, creates claims such as issuer, audience, scope, tenant, and expiry, and signs the JWT with its private key. The client sends the token as a Bearer token. The API validates the signature using Entra ID's public keys and then checks the token's issuer, audience, expiry, and required scope or role before allowing access.**

---

## How do you secure a Web API application?

A Web API should be protected in layers. Authentication confirms **who** is calling, and authorization confirms **what that caller is allowed to do**.

### Main ways to secure a Web API

1. **Use HTTPS**

   Encrypt all traffic between the client and API. Redirect or reject plain HTTP requests so passwords, tokens, and business data are not sent in clear text.

2. **Authenticate callers**

   Use Microsoft Entra ID, OAuth 2.0, and OpenID Connect instead of creating your own password system where possible. The client sends an access token:

   ```http
   Authorization: Bearer <access-token>
   ```

3. **Authorize every protected operation**

   Check scopes or application roles for each endpoint. For example, a caller with `Claims.Read` should not automatically be allowed to create or delete claims. Use policies such as:

   ```csharp
   [Authorize(Policy = "Claims.Read")]
   ```

4. **Validate tokens correctly**

   Validate the JWT signature, issuer, audience, expiry, and required scope or role. Do not trust a token only because it is present or because the request came from an internal network.

5. **Validate input**

   Check request models, required fields, lengths, formats, ranges, and allowed values. Reject unexpected input and use parameterized database queries or ORM APIs to reduce injection risks.

6. **Protect secrets and configuration**

   Do not store passwords, API keys, or client secrets in source code or committed `appsettings.json`. Use Managed Identity, Azure Key Vault, environment variables, or a secure secret store.

7. **Control traffic**

   Add rate limiting and request-size limits to reduce brute-force attempts, denial-of-service risk, and accidental overload. API Management or an ingress gateway can apply these controls consistently.

8. **Configure CORS carefully**

   Allow only trusted browser origins, methods, and headers. Do not use `AllowAnyOrigin` for a protected production API unless the design explicitly requires it.

9. **Handle errors safely**

   Return useful status codes such as `401 Unauthorized`, `403 Forbidden`, `400 Bad Request`, and `404 Not Found`, but do not expose stack traces, SQL details, tokens, or secret values to clients.

10. **Log and monitor security events**

    Log authentication failures, authorization failures, unusual traffic, and important administrative actions. Use correlation IDs for investigation, but never log passwords, access tokens, or sensitive personal data.

11. **Secure dependencies and deployment**

    Keep .NET, NuGet packages, containers, and servers patched. Use code-quality and security scanning in CI/CD, restrict network access with private endpoints or firewalls where appropriate, and give identities only the permissions they need.

### Simple request flow

```text
Client
   ↓ HTTPS + access token
API gateway / APIM
   ↓ rate limit and basic protection
Web API
   ↓ validate JWT and permissions
Controller / service layer
   ↓ validate input and business rules
Database or downstream API
```

### Authentication versus authorization

```text
Authentication = Who are you?
Authorization  = What are you allowed to do?
```

For example, Entra ID can authenticate a calling application, while the API's authorization policy decides whether that application has permission to read claims.

### Interview-ready answer

> **I would secure a Web API in layers. I would use HTTPS for encryption, Microsoft Entra ID and OAuth 2.0 for authentication, and scopes or roles for authorization. The API would validate the JWT signature, issuer, audience, expiry, and permissions. I would validate all inputs, use parameterized database access, keep secrets in Key Vault, apply rate limiting and strict CORS, return safe error messages, and monitor authentication or authorization failures. I would also patch dependencies and run security checks in the CI/CD pipeline.**

---

## How would you secure service-to-service communication?

I would use **OAuth 2.0 with Microsoft Entra ID**, HTTPS, and least-privilege permissions.

### Basic flow

```text
Service A
   ↓
Authenticates with Entra ID
   ↓
Receives an access token for Service B
   ↓
Calls Service B over HTTPS
   ↓
Service B validates the token
   ↓
Request is allowed or rejected
```

### Main security controls

1. **Use Entra ID authentication**

   Service A must prove its identity to Entra ID before calling Service B. There are two common ways to do that:
   - **Managed Identity:** If Service A runs on Azure, Azure gives it a managed identity. Service A uses that identity to request a token without storing a client secret. This is the preferred option for Azure-to-Azure communication.
   - **Client Credentials flow:** If Managed Identity is not available, Service A uses an Entra app registration with a client ID and a certificate or client secret. It sends those credentials to Entra ID and receives a token. The credential must be stored securely, such as in Key Vault, and never in source code.

   In both cases, Entra ID issues an access token for Service B. Service A then sends that token in the request. Service B does not trust the caller only because it is inside the same network.

2. **Use HTTPS/TLS**

   Encrypt traffic in transit so tokens and business data cannot be read or changed while travelling between services.

3. **Validate the access token**

   Service B validates the JWT signature, issuer, audience, expiry time, and required scope or application role.

4. **Use least privilege**

   Give Service A only the permission it needs, such as `Orders.Read`, instead of broad access to every endpoint in Service B.

5. **Use Managed Identity in Azure**

   When Service A runs on Azure, Managed Identity avoids storing client secrets in configuration. Azure identifies Service A, Entra ID issues the access token, and Service A sends it to Service B.

6. **Add network-level protection**

   Use private endpoints, VNets, firewall rules, API Management, or service-mesh controls where required. Network restrictions provide an additional boundary, but they do not replace token validation.

7. **Monitor and protect failures**

   Log authentication failures without logging tokens, apply rate limits where appropriate, use timeouts and retries carefully, and rotate any certificates or secrets that are still required.

### Interview-ready answer

> **I would secure service-to-service communication using OAuth 2.0 and Microsoft Entra ID. The calling service would use Managed Identity or another securely stored credential to obtain an access token for the target API. It would call the API over HTTPS, and the receiving service would validate the JWT signature, issuer, audience, expiry, and required scope or role. I would apply least-privilege permissions and add network controls such as private endpoints or firewall rules where needed.**

---

## How does the batch make a real-time call to the Claims Domain API?

In this flow, **real-time** means the batch sends an HTTP request to the Claims Domain API and waits for the response before continuing its current processing step. It does not mean a user is waiting on a screen; it describes the communication style between the two services.

### Real-time request-response flow

```text
MedicalClaimRecon batch
   ↓
Reads Claims Domain API base URL and scope from configuration
   ↓
Obtains an Entra ID access token
   ↓
Sends HTTPS request with Bearer token
   ↓
Claims API Management validates the JWT
   ↓
Claims Domain API checks permissions
   ↓
API returns response
   ↓
Batch processes the response and continues
```

### Example flow in simple terms

1. The batch identifies a claim that must be checked or updated.
2. It builds an HTTP request for the Claims Domain API.
3. It obtains an access token for the Claims API. In Azure, this may use the batch Function's Managed Identity; the exact implementation should be confirmed in the batch source or deployment configuration.
4. It sends the token in the request header:

```http
Authorization: Bearer <access-token>
```

5. APIM validates the token's signature, issuer, audience, expiry, and required permission.
6. The Claims API processes the request and returns a response such as success, validation failure, not found, or server error.
7. The batch handles the response, logs the result with a correlation or claim ID, and continues or retries according to the error-handling rules.

### Real-time versus asynchronous communication

```text
Real-time:
Batch → HTTP request → Claims API → HTTP response → Batch continues

Asynchronous:
Batch → Message broker → Claims API consumer processes later
```

The Claims Domain API documentation confirms OAuth2 and JWT Bearer authentication, with token validation at APIM and permission checks in the API. The available MedicalClaimRecon folder is empty, so the exact token acquisition class cannot be confirmed from the attached source.

### Interview-ready answer

> **The MedicalClaimRecon batch communicates with the Claims Domain API synchronously over HTTPS. It reads the API URL and scope from configuration, obtains an Entra ID access token, and sends an HTTP request with the token in the Authorization Bearer header. APIM validates the JWT, and the Claims API checks the required permission before processing the request. The batch waits for the response, logs the result using the claim or correlation ID, and then continues processing or applies retry and error-handling rules.**

---

## How is the Azure Function deployed?

In simple terms, the Function code is packaged into a container and deployed to **Azure Kubernetes Service (AKS)** using **Helm**.

### Simple deployment flow

```text
Developer commits code
   ↓
Azure DevOps pipeline starts
   ↓
Application is built and tested
   ↓
Docker image is created
   ↓
Image is pushed to a container registry
   ↓
Helm deploys the image to AKS
   ↓
Kubernetes starts the Function container
   ↓
Function is ready for Control-M or HTTP requests
```

### What happens step by step?

1. **Build the application**

   The pipeline restores dependencies, compiles the .NET Azure Function, and runs automated tests.

2. **Create a container image**

   The Function and its runtime are packaged into a Docker image. The image contains everything required to run the application.

3. **Push the image**

   The pipeline pushes the image to the organisation's container registry with a version or build tag.

4. **Deploy with Helm**

   Helm uses the chart for the batch Function, such as `batch-medicalclaimrecon-fa`. The chart creates or updates the Kubernetes Deployment, ConfigMap, environment variables, and other required resources.

5. **Run in an AKS namespace**

   Kubernetes starts the container in the appropriate environment namespace. For the documented D24 environment, the namespace is `hicbocoreservices-d24-ns`.

6. **Load configuration**

   Helm and Kubernetes provide settings such as the API URL, database configuration, logging configuration, and Key Vault references through environment variables or configuration resources. The application reads them using normal .NET configuration.

7. **Verify the deployment**

   The team checks that the pod is running, reviews logs and health information, and invokes the Function endpoint or runs the batch through Control-M.

### Simple interview answer

> **The Azure Function is deployed through an Azure DevOps pipeline. The pipeline builds and tests the .NET application, packages it as a Docker image, and pushes that image to a container registry. Helm then deploys the image to AKS, where Kubernetes runs it in the required namespace. Configuration is supplied through Helm and Kubernetes environment variables, and we verify the deployment by checking the pod, logs, health status, and Function endpoint.**

The available documentation confirms the AKS, Helm, and `batch-medicalclaimrecon-fa` deployment pattern. The exact pipeline name and container registry name are not available in the attached empty project folder.

---

## How did CI/CD work in the previous project?

In simple terms:

- **CI** means automatically building and checking the code.
- **CD** means automatically delivering and deploying the approved build to an environment.

### CI flow

```text
Developer pushes code
   ↓
Azure DevOps pipeline starts
   ↓
Restore .NET dependencies
   ↓
Build the Azure Function
   ↓
Run unit and integration tests
   ↓
Run code-quality and security checks
   ↓
Create a versioned Docker image
```

The CI stage helps catch compilation errors, failing tests, code-quality issues, and security vulnerabilities before deployment.

### CD flow

```text
Approved build
   ↓
Push Docker image to container registry
   ↓
Select environment values
   ↓
Helm applies the Kubernetes configuration
   ↓
Deploy image to AKS
   ↓
Kubernetes starts or updates the Function pod
   ↓
Run health checks and smoke tests
```

### What changes between environments?

The application image can remain the same, while environment-specific configuration changes, such as:

- API URLs
- Database endpoints
- Key Vault references
- Logging settings
- Namespace and replica settings

Helm values and pipeline variables provide those environment-specific settings. Secrets should come from Key Vault or secure pipeline variables rather than being committed to source control.

### Simple interview answer

> **In the previous project, CI/CD was handled through Azure DevOps. When code was pushed, the CI pipeline restored dependencies, built the Azure Function, ran tests, and performed code-quality and security checks. It then created a versioned Docker image. After approval, the CD pipeline pushed the image to the container registry and used Helm to deploy it to AKS. Kubernetes started the new Function pod, and we verified the deployment through health checks, logs, and a smoke test.**

The available documentation confirms the Azure DevOps, Docker, Helm, and AKS deployment pattern. The exact pipeline and registry names are not available in the attached project folder.
