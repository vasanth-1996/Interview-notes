# Project Based Answers 2

## Index

- [Q1: Tell me about one query you optimized and how (SQL)](#q1-tell-me-about-one-query-you-optimized-and-how-sql)
- [Q2: What is an N+1 query problem?](#q2-what-is-an-n1-query-problem)
- [Q3: How would you optimize an API returning 1 million records?](#q3-how-would-you-optimize-an-api-returning-1-million-records)
- [Q4: What does SonarQube do?](#q4-what-does-sonarqube-do)
- [Q5: What does Checkmarx do?](#q5-what-does-checkmarx-do)
- [Q6: What is SQL Injection?](#q6-what-is-sql-injection)
- [Q7: What is XSS?](#q7-what-is-xss)
- [Q8: What is insecure deserialization?](#q8-what-is-insecure-deserialization)
- [Q9: What are the OWASP Top 10 guidelines?](#q9-what-are-the-owasp-top-10-guidelines)
- [Q10: How do you pass data between two microservices using Entra ID?](#q10-how-do-you-pass-data-between-two-microservices-using-entra-id)
- [Q11: Why should we not simply suppress security warnings?](#q11-why-should-we-not-simply-suppress-security-warnings)
- [Q12: Cursor plus MCP used in project development](#q12-cursor-plus-mcp-used-in-project-development)
- [Q13: Which AWS services did you actually use?](#q13-which-aws-services-did-you-actually-use)

## Q1: Tell me about one query you optimized and how (SQL)

In one project, I worked on a SQL query that was taking too long to return results. The main reason was that SQL Server had to scan a large number of rows before finding the matching data, so the query became slow when the table grew bigger.

### What the problem was

The query was doing a few things that made it inefficient:

- It was reading more rows than necessary.
- It was joining tables without using the best index path.
- It was selecting extra columns that were not actually needed.
- It was applying a function on the filter column, which made the index harder to use.

In simple terms, the database was doing extra work just to find the same result.

### What was the original problem?

The original problem was that the report or screen was taking too long to load because the SQL query behind it was slow. When the data size increased, the same query started taking more time and put extra load on the database.

So the real issue was not the application itself. The issue was the database query becoming the slow part of the flow.

### How did I identify the bottleneck?

I identified the bottleneck by checking the execution plan and the query behavior.

- The execution plan showed a table scan instead of a quick index seek.
- The query was touching too many rows before reaching the required data.
- The filter condition was not using the index properly because of the way the query was written.
- The query time was higher than expected compared to similar queries.

In simple terms, I saw that SQL Server was spending time searching through a lot of data instead of jumping straight to the required rows.

### Did I know it was the database from Application Insights?

Yes. I used Application Insights to compare the overall request time with the dependency time.

### Steps I followed in Application Insights

- I opened the slow request in the `Requests` view.
- I checked the total duration of the request.
- I looked at the `Dependencies` section to see how long the SQL call took.
- I compared the request time with the database call time.
- I noticed the SQL dependency was taking most of the total time.
- I also checked for retries, timeouts, or unusually slow dependency calls.

### How that helped

If the request took, for example, 5 seconds and the SQL dependency itself took 4.5 seconds, that clearly showed the database was the slow part, not the application logic.

So Application Insights helped me narrow it down quickly, and then I used the SQL execution plan to confirm the exact query issue.

### What I changed

- I added an index on the columns used in the `WHERE` clause and join condition, so SQL Server could find the rows faster.
- I changed the query to select only the columns the screen or report actually needed, instead of pulling unnecessary data.
- I removed functions from the filter column, because wrapping a column in a function often prevents the index from being used properly.
- I checked the execution plan before and after the change to confirm that the database was using the index instead of scanning the whole table.

### Simple example

Before optimization, the query was more like this:

```sql
SELECT *
FROM Orders
WHERE CONVERT(VARCHAR, OrderDate, 23) = '2026-09-16'
```

This is slow because SQL Server has to convert every row before comparing it.

After optimization, I changed it to a more index-friendly form:

```sql
SELECT OrderId, CustomerId, OrderDate
FROM Orders
WHERE OrderDate >= '2026-09-16'
	AND OrderDate < '2026-09-17'
```

This is faster because SQL Server can use the index on `OrderDate` directly.

### Result

The query became much faster because SQL Server stopped doing a full table scan and started reading only the rows it needed. That reduced the time taken by the query and also lowered the load on the database.

### How I explained it in the project

I told the team that the query was slow because the database was searching too much data. I confirmed this by looking at the execution plan and seeing a table scan. Then I fixed it by making the filter easier for SQL Server to use, adding the right index, and removing unnecessary columns from the result. After that, the query ran noticeably faster.

### Simple interview answer

I optimized a SQL query by adding the right index, reducing the selected columns, and making the filter index-friendly. In simple terms, I helped the database find the data faster by avoiding a full table scan. This improved performance and reduced query time noticeably.

## Q2: What is an N+1 query problem?

The N+1 query problem happens when the application makes one query to get a list of records, and then makes one extra query for each record in that list.

### Simple example

If I load 1 list of customers and then run another query for each customer to get their orders, that becomes:

- 1 query to load customers
- N queries to load related data for each customer

So if there are 100 customers, the app ends up making 101 queries. That is why it is called the N+1 problem.

### Why it is a problem

- It increases database round trips.
- It makes the application slower.
- It puts more load on the database.
- It gets worse as the number of records grows.

### How to fix it

- Use `Include()` in Entity Framework when appropriate.
- Use joins or projection to fetch all needed data in one query.
- Avoid loading child data inside a loop.
- Check the generated SQL when using ORM tools.

### Simple interview answer

An N+1 query problem happens when one query loads the main list and then one extra query runs for each item in that list. It is slow because it creates too many database calls. I usually fix it by loading the related data in a single query using `Include()`, joins, or projection.

## Q3: How would you optimize an API returning 1 million records?

If an API needs to return 1 million records, I would not try to send all of them in one response. That would be too slow, too heavy on memory, and bad for the client as well.

### What I would do first

I would ask one simple question: does the user really need all 1 million records at once?

Usually the answer is no. Most of the time, the user only needs a page of data, a filtered result, or an export file.

### Step-by-step approach

#### 1. Add pagination

I would return the data in small pages instead of one huge response.

- Example: 50 or 100 records per request
- Use `pageNumber` and `pageSize`
- This reduces load on the API and the database

In simple words, I would give the client only what it needs right now.

#### 2. Let the client filter the data

I would allow filters like:

- date range
- status
- customer id
- search text

This way, the API fetches only the needed records instead of everything.

#### 3. Return only required columns

I would not send full entities if the screen needs only a few fields.

- Instead of `SELECT *`
- Return only the columns that are actually used

This makes the response smaller and faster.

#### 4. Optimize the database query

I would check the SQL query behind the API.

- Add the right indexes
- Avoid scanning the whole table
- Avoid functions on filter columns
- Check the execution plan

If the database query is slow, the API will also be slow.

#### 5. Use projection instead of loading full objects

In Entity Framework, I would project only the needed fields into a DTO.

This avoids loading extra data into memory.

#### 6. Compress the response

If the response is still large, I would enable response compression.

That helps reduce the network size and makes transfer faster.

#### 7. Use async calls

I would make sure the API uses async database and I/O calls.

That does not make the query magically faster, but it helps the server handle more requests efficiently.

#### 8. For export scenarios, use background processing

If the business really needs all 1 million records, I would not return them directly in the API response.

Instead, I would:

- start a background job
- generate a file like CSV or Excel
- store it in blob storage or a shared location
- return a download link or job status

This is much safer than keeping one HTTP request open for too long.

### Simple example

Instead of this:

```text
GET /api/orders -> returns 1,000,000 rows
```

I would do this:

```text
GET /api/orders?pageNumber=1&pageSize=100&status=active
```

Or for export:

```text
POST /api/orders/export
```

Then the API processes the file in the background and gives the user a link later.

### Why this works

- The API response becomes smaller
- The database does less work
- The application uses less memory
- The client gets data faster
- The system becomes more stable under load

### Simple interview answer

If an API had to return 1 million records, I would not send them all in one response. I would use pagination, filtering, projection, indexing, compression, and async calls. If the full data set was still needed, I would move it to a background export process instead of keeping one huge API request open.

## Q4: What does SonarQube do?

SonarQube is a code quality tool. It checks source code and helps find problems before the code goes to production.

### In simple words

It acts like an automatic reviewer for code.

It can find things like:

- bugs
- code smells
- duplicated code
- security issues
- complex code that is hard to maintain

### What we usually check in SonarQube

From the dashboard, I usually check:

- Quality Gate status - shows whether the code meets the minimum quality rules.
- Security rating - shows how safe the code is from security problems.
- Reliability rating - shows how likely the code is to have bugs or runtime issues.
- Maintainability rating - shows how easy the code is to read, fix, and improve.
- Test coverage - shows how much of the code is covered by automated tests.
- Duplications - shows repeated code that should be cleaned up.
- Security hotspots - shows risky code areas that need a manual review.
- Open issues and accepted issues - shows the problems still not fixed and the ones already accepted by the team.

These tell me if the code is healthy or if it still needs fixes.

### Why it is useful

- It helps improve code quality.
- It catches problems early.
- It makes the code easier to maintain.
- It helps teams follow coding standards.

### Example

If a method is too long, repeated many times, or written in a risky way, SonarQube can flag it.

### Simple interview answer

SonarQube is a tool that scans code and points out bugs, bad patterns, duplicate code, and security issues. In the dashboard, we usually check quality gate, security, reliability, maintainability, coverage, duplications, security hotspots, and open issues. It helps developers write cleaner, safer, and easier-to-maintain code before the application goes live.

## Q5: What does Checkmarx do?

Checkmarx is a security scanning tool. It checks code and related project artifacts to find security risks before the application goes live.

### In simple words

It helps us find weak points in the application before attackers can use them.

### What we usually check in Checkmarx

From the scanners in your screenshot, I usually explain them like this:

- SAST - scans source code for security bugs in the code itself.
- IaC Security - scans infrastructure code like Terraform or ARM/Bicep for security misconfigurations.
- SCA - checks third-party libraries and packages for known vulnerabilities.
- API Security - checks APIs for security weaknesses in endpoints, requests, and responses.

### What each one means in simple terms

- SAST means Static Application Security Testing. It reads the source code without running the app and looks for risky patterns like SQL injection, hardcoded secrets, unsafe input handling, and weak validation.
- IaC Security means Infrastructure as Code Security. It checks deployment files like Terraform, ARM, or Bicep to find unsafe cloud settings such as open ports, public storage, missing encryption, or weak access rules.
- SCA means Software Composition Analysis. It checks the third-party packages and libraries used in the project and tells us if any of them have known security vulnerabilities or outdated versions.
- API Security tells us if the API itself is exposed to common attacks or weak validation.

### Why it is useful

- It finds security problems early.
- It reduces the chance of production issues.
- It helps teams follow secure coding practices.
- It protects both code and dependencies.

### How did I fix them?

- For SAST, I changed the unsafe code pattern, added proper input validation, removed hardcoded secrets, and fixed risky SQL or string handling.
- For IaC Security, I changed the infrastructure file to close open ports, restrict public access, enable encryption, and follow secure cloud settings.
- For SCA, I upgraded the vulnerable package to a safe version or removed the package if it was not needed.
- For API Security, I added proper authentication, validation, and input checks, and made sure the API did not expose unsafe data.

### Simple interview answer

Checkmarx is a security scanning tool that helps find vulnerabilities before release. In the scanners, SAST checks source code, IaC Security checks infrastructure code, SCA checks third-party packages, and API Security checks API-level risks. It helps us catch security problems early and keep the application safer.

## Q6: What is SQL Injection?

SQL Injection is a security attack where an attacker puts harmful SQL code into an input field, and the application passes it to the database without checking it properly.

### In simple words

The attacker tricks the database into running a query that should not run.

### Example

If an application builds a SQL query by directly joining user input into the string, a user can inject extra SQL and change the query behavior.

### Why it is dangerous

- It can expose sensitive data.
- It can change or delete records.
- It can bypass login checks.
- It can damage the database.

### How to prevent it

- Use parameterized queries.
- Use ORM tools safely.
- Validate user input.
- Avoid building SQL with string concatenation.

### Simple interview answer

SQL Injection is a security issue where unsafe user input changes the SQL query. It is dangerous because it can expose or change database data. I prevent it by using parameterized queries and proper input validation.

## Q7: What is XSS?

XSS means Cross-Site Scripting. It happens when an attacker puts harmful script into a web page, and the browser runs it.

### In simple words

The attacker injects JavaScript into the page so it runs in another user's browser.

### Example

If a website shows user input on the page without cleaning it, an attacker can enter script code instead of normal text.

### Why it is dangerous

- It can steal cookies or session data.
- It can show fake content.
- It can redirect users to bad sites.
- It can perform actions as the logged-in user.

### How to prevent it

- Encode output before showing it on the page.
- Sanitize user input.
- Use safe templating frameworks.
- Apply Content Security Policy where possible.

### Simple interview answer

XSS is a security issue where harmful script is injected into a web page and runs in the browser. It is dangerous because it can steal data or misuse the user's session. I prevent it by encoding output and sanitizing input properly.

## Q8: What is insecure deserialization?

Insecure deserialization is a security issue where an application takes data from an untrusted source and converts it back into an object without checking it safely.

### In simple words

The application trusts data that may have been changed by an attacker.

### Why it is dangerous

- An attacker can change the data before the app reads it.
- It can lead to code execution or privilege abuse in some cases.
- It can make the app load unsafe or fake object values.

### Example

If an app stores a serialized object and later reads it back without validation, an attacker may tamper with that object and cause unexpected behavior.

### How to prevent it

- Do not deserialize untrusted data directly.
- Use safe formats like JSON with validation.
- Sign or encrypt data when needed.
- Allow only known types and check input carefully.

### Simple interview answer

Insecure deserialization happens when an application trusts serialized data from an untrusted source. It is dangerous because attackers can modify the data and change how the app behaves. I prevent it by avoiding unsafe object deserialization and validating or signing the data properly.

## Q9: What are the OWASP Top 10 guidelines?

OWASP Top 10 is a list of the most common web application security risks. It helps developers understand the main areas where applications usually become unsafe.

### In simple words

It is like a security checklist for web applications.

### The main OWASP Top 10 risks and how I would fix them

#### 1. Broken Access Control

This means users can access data or pages they should not see.

Fix:

- Check user roles on every request.
- Do not trust hidden UI buttons alone.
- Add server-side authorization.

Example: A normal user should not be able to open another user's order by changing the URL. I would fix this by validating ownership on the server.

#### 2. Cryptographic Failures

This means sensitive data is not protected properly.

Fix:

- Use HTTPS.
- Encrypt sensitive data.
- Do not store passwords in plain text.

Example: If a system stores passwords, I would hash them with a strong hashing algorithm instead of saving them directly.

#### 3. Injection

This happens when untrusted input is sent into SQL, commands, or other queries.

Fix:

- Use parameterized queries.
- Validate input.
- Avoid string concatenation.

Example: For SQL input, I would use parameters instead of building the query with user text.

#### 4. Insecure Design

This means the application design itself is weak from a security point of view.

Fix:

- Add security in the design phase.
- Think about abuse cases early.
- Use threat modeling.

Example: If an API allows unlimited payment attempts, I would add rate limits and stronger validation in the design.

#### 5. Security Misconfiguration

This means the app or server is configured in an unsafe way.

Fix:

- Remove default passwords.
- Disable unused features.
- Set secure headers.
- Use safe cloud and server settings.

Example: I would not keep debug mode enabled in production.

#### 6. Vulnerable and Outdated Components

This means the app uses libraries with known security problems.

Fix:

- Scan packages regularly.
- Update vulnerable libraries.
- Remove unused dependencies.

Example: If a NuGet package has a security issue, I would upgrade it to a safe version.

#### 7. Identification and Authentication Failures

This means login and user identity handling are weak.

Fix:

- Use strong passwords and MFA.
- Lock accounts after repeated failures.
- Protect tokens and sessions.

Example: I would add multi-factor authentication for sensitive systems.

#### 8. Software and Data Integrity Failures

This means the app trusts unsafe code, updates, or data.

Fix:

- Verify package and build integrity.
- Sign important data.
- Do not trust tampered inputs.

Example: I would make sure deployment packages and config files are verified before use.

#### 9. Security Logging and Monitoring Failures

This means the system is not logging or monitoring important security events.

Fix:

- Log failed logins and suspicious actions.
- Monitor alerts.
- Keep logs protected.

Example: I would log repeated failed login attempts so the security team can review them.

#### 10. Server-Side Request Forgery

This happens when the server is tricked into calling unsafe internal or external URLs.

Fix:

- Validate allowed URLs.
- Block internal IPs.
- Use allowlists.

Example: If an API fetches a URL from user input, I would allow only trusted domains.

### Simple interview answer

OWASP Top 10 is a list of the most common web security risks. It covers problems like broken access control, injection, weak authentication, security misconfiguration, and unsafe components. I fix them by using proper authorization, parameterized queries, secure passwords, safe configuration, updated libraries, logging, and input validation.

## Q10: How do you pass data between two microservices using Entra ID?

If two microservices use Entra ID, I do not pass data by sharing credentials or calling each other anonymously. I use secure service-to-service authentication.

### Simple flow

- Service A needs to call Service B.
- Service A gets an access token from Entra ID.
- Service A sends the token in the `Authorization` header.
- Service B validates the token.
- Service B returns or accepts only the required data.

### In simple words

Entra ID proves that Service A is allowed to call Service B.

### How I usually do it

#### 1. Register both services in Entra ID

- Create an app registration for Service A.
- Create an app registration for Service B.
- Expose an API scope or app role on Service B.

#### 2. Give Service A permission to call Service B

- Assign the required app role or scope.
- Use client credentials flow or managed identity for service-to-service access.

#### 3. Service A gets a token

- Service A requests an access token from Entra ID.
- The token is meant for Service B.

#### 4. Service A calls Service B

- Send the token in `Authorization: Bearer <token>`.
- Send only the business data needed for that call in the request body.

#### 5. Service B validates the token

- Check the issuer, audience, and expiry.
- Check the scope or role.
- Reject the call if the token is not valid.

### Example

If Service A wants to create an order in Service B, it sends a secure API request like this:

```text
POST /orders
Authorization: Bearer <access-token>
```

And the body contains only the order data, not the security credentials.

### If the data is large or not needed immediately

I would not send everything in one API call.

Instead, I would use:

- a message queue
- a service bus
- an event

This is better for async communication and loose coupling.

### Simple interview answer

When two microservices use Entra ID, I use service-to-service authentication. Service A gets an access token from Entra ID and sends it to Service B in the bearer header. Service B validates the token and then processes only the required data. If the data is large or asynchronous, I use messaging like Service Bus instead of a direct API call.

## Q11: Why should we not simply suppress security warnings?

We should not simply suppress security warnings because the warning may point to a real security risk. If we hide it without checking properly, the problem can still go to production.

### In simple words

Suppressing a warning does not fix the issue. It only hides it.

### What does suppressing mean?

Suppressing means telling the tool to ignore a warning and stop showing it.

In simple terms, it means saying, "do not report this warning again," even though the code may still be the same.

### Why suppression is risky

- A real vulnerability may remain in the code.
- The issue can later become a production incident.
- It gives a false feeling that the application is safe.
- Other developers may think the issue was already fixed.

### What I do instead

- First, I understand why the tool raised the warning.
- Then I check whether it is a real issue or a false positive.
- If it is real, I fix the code properly.
- If it is a false positive, only then I suppress it with a clear reason.

### When suppression is acceptable

Suppression is acceptable only when:

- the warning is confirmed as a false positive
- the risk is understood and accepted by the team
- the reason is documented clearly
- there is no safer practical fix

### Example

If a tool flags hardcoded input handling and I confirm user input is already validated and cannot reach a dangerous path, then suppression may be acceptable with proper comments and review. But if the warning is about unsafe SQL or missing authorization, I should fix the code instead of hiding the warning.

### Simple interview answer

We should not suppress security warnings blindly because they may represent real vulnerabilities. Suppression only hides the warning, it does not solve the problem. My approach is to investigate first, fix real issues properly, and suppress only confirmed false positives with a documented reason.

## Q12: Cursor plus MCP used in project development

In one project, we used an AI coding tool like Cursor together with MCP to help developers work faster, but in a controlled and secure way.

### What problem were we solving?

The main problem was that developers were spending a lot of time switching between tools and doing repetitive work.

For example:

- reading Jira tickets manually
- opening GitHub to understand code changes
- searching the codebase for the right files
- creating boilerplate code or test skeletons
- collecting context from multiple systems before starting development

In simple words, the problem was not just writing code. The real problem was the time lost in gathering context from Jira, GitHub, and the project itself.

### End-to-end flow

- A developer opens the project in Cursor.
- Cursor can ask MCP for external context.
- MCP talks to approved systems like Jira and GitHub.
- MCP returns only the allowed data back to Cursor.
- Cursor uses that context to help with explanations, code suggestions, summaries, or draft changes.
- The developer reviews everything before applying or committing changes.

### How does it read Jira issues?

Cursor itself does not directly log in to Jira. It uses the MCP server.

The flow is usually:

- the developer asks for a Jira ticket summary
- Cursor sends that request to MCP
- MCP calls Jira APIs using approved credentials
- MCP reads fields like title, description, acceptance criteria, comments, or linked items
- MCP sends the safe response back to Cursor

In simple words, MCP acts like a controlled bridge between Cursor and Jira.

### How does it access GitHub?

Again, Cursor does not directly use a random GitHub session. MCP connects to GitHub in a controlled way.

The flow is:

- Cursor asks for a PR, branch, file, or repository context
- MCP calls GitHub APIs with a restricted token or GitHub App
- MCP reads only the repositories and actions it is allowed to use
- MCP returns the requested metadata, code diff, file content, or PR summary

### MCP

### Where does MCP fit?

MCP sits between the AI tool and external systems.

So the design is:

- Cursor is the AI client
- MCP is the tool gateway
- Jira and GitHub are external systems

MCP makes sure the AI does not directly access everything on its own.

### Why did we use MCP?

We used MCP because it gave us control, security, and standard integration.

Main reasons:

- one standard way to connect AI with tools
- better access control
- easier auditing
- safer handling of credentials
- ability to expose only approved operations

### What tools did the MCP server expose?

Depending on the project, the MCP server can expose tools like:

- get Jira issue details
- search Jira tickets
- read pull request summary
- read repository files
- search code in allowed repositories
- create draft code suggestions
- create PR summaries or release notes

In a stricter setup, write actions are separated from read actions.

### Security

### How did we secure GitHub access?

We secured GitHub access by using a GitHub App or restricted token with minimum permissions.

That means:

- access only to approved repositories
- read-only access where possible
- no broad admin permissions
- token storage in a secure secret store
- token rotation when needed

### How did we secure Jira credentials?

We did not hardcode Jira credentials in prompts or source code.

We secured them by:

- storing secrets in a vault or secret manager
- using service accounts or approved API tokens
- limiting access to only required Jira projects
- rotating credentials regularly

### Can the AI modify the repository?

Yes, technically it can be allowed to suggest or even make changes, but we should not give uncontrolled write access.

### If yes, how did we control that?

We controlled it by using guardrails such as:

- write access only in approved repositories
- branch protection rules
- pull request approval process
- limited MCP tools for write operations
- human review before merge
- audit logs for tool usage

In simple words, the AI could help create changes, but it could not freely push anything to production.

### How did we prevent prompt injection from a Jira ticket?

This is important because a Jira ticket may contain text like "ignore all previous instructions" or malicious content.

We handled that by:

- treating Jira content as untrusted input
- not allowing ticket text to override system instructions
- sanitizing or filtering unsafe content
- limiting tool actions based on policy, not ticket text
- keeping MCP tools permission-based

In simple words, a Jira ticket could provide business context, but it could not control the AI system.

### How did we prevent the AI from accessing unauthorized repositories?

We prevented that by restricting access at the MCP and credential level.

That means:

- GitHub token or app was scoped only to approved repositories
- MCP checked repository allowlists
- requests outside approved repos were rejected
- access was logged and auditable

### Simple interview answer

We used Cursor with MCP to reduce the time developers spent switching between Jira, GitHub, and the codebase. Cursor handled the AI interaction, and MCP acted as the secure bridge to external tools. MCP exposed only approved Jira and GitHub operations, used restricted credentials, and returned controlled data back to the AI. For security, we used scoped access, secret storage, branch protection, approval workflows, audit logs, and treated Jira content as untrusted input to prevent prompt injection or unauthorized repository access.

## Q13: Which AWS services did you actually use?

This answer should be given carefully, because my OSP project and Bonmojo project were not exactly the same in terms of cloud platform.

### Short honest answer

In the OSP project, I used AWS services directly. In Bonmojo, the documented setup was mainly Azure-based, so I would not claim AWS there unless the interviewer is asking only at a pattern level.

### AWS services I actually used in OSP

- AWS API Gateway
- Amazon SQS
- Amazon ECS with Fargate
- Amazon RDS
- Amazon ElastiCache for Redis
- Amazon S3
- Amazon CloudFront
- AWS Lambda
- Amazon CloudWatch
- AWS X-Ray
- Amazon SES
- Amazon SNS

### What I used them for in simple words

### Beginner-friendly explanation of each AWS service

#### 1. AWS API Gateway

AWS API Gateway is the front door for your APIs.

What it does:

- Receives HTTP requests from the frontend or another service.
- Routes the request to the right backend service.
- Can check authentication, rate limits, and request rules before forwarding.

Why a .NET developer uses it:

- Your Angular app should not call many services directly.
- API Gateway gives one clean endpoint, like `https://api.company.com`.
- It helps keep the microservices hidden and easier to manage.

Simple example:

- `GET /users` goes to User Service.
- `POST /orders` goes to Order Service.

In practice, I think of it as the traffic controller for the system.

#### 2. Amazon SQS

Amazon SQS is a message queue.

What it does:

- Stores messages temporarily until another service is ready to process them.
- Lets services talk to each other without waiting in real time.

Why a .NET developer uses it:

- It helps when one service should not block another.
- A controller or API can send a message and return fast.
- A background worker in .NET can read the queue later.

Simple example:

- Payment Service sends a message: "payment completed".
- Notification Service reads that message and sends an email.

Important concept:

- SQS is good for decoupling services and handling spikes in traffic.
- It is not for instant request/response style communication.

#### 3. Amazon ECS with Fargate

Amazon ECS Fargate is where Docker containers run without managing servers.

What it does:

- Hosts your containerized .NET APIs.
- Automatically handles the underlying infrastructure.

Why a .NET developer uses it:

- You build a Docker image for your ASP.NET Core app.
- ECS Fargate runs that container for you.
- You do not need to patch or manage EC2 machines directly.

Simple example:

- Build a container for `UserService`.
- Deploy it to ECS Fargate.
- Scale it up if traffic increases.

Important concept:

- Each service can run in its own container.
- This makes deployments and scaling easier than one big monolith.

#### 4. Amazon RDS

Amazon RDS is a managed relational database service.

What it does:

- Gives you a SQL database without manual server administration.
- Handles backups, patching, monitoring, and failover options.

Why a .NET developer uses it:

- .NET apps commonly use Entity Framework Core or Dapper with SQL databases.
- RDS gives a stable database endpoint that your app connects to through a connection string.

Simple example:

- Store users, orders, transactions, or logs in RDS.
- Use EF Core migrations to update the schema.

Important concept:

- RDS is still a relational database, so you use tables, joins, indexes, and transactions.
- It is useful when you need structured data and strong consistency.

#### 5. Amazon ElastiCache for Redis

Amazon ElastiCache Redis is an in-memory cache.

What it does:

- Stores frequently used data in memory so it can be fetched very fast.
- Reduces pressure on the database.

Why a .NET developer uses it:

- It is great for caching data that changes slowly.
- It can store session-like data, rate limits, or short-lived values.

Simple example:

- Cache product or profile data for 5 minutes.
- Read from Redis first.
- If the data is not there, fetch from the database and store it in Redis.

Important concept:

- Redis is not your main long-term database.
- It is a speed layer.

#### 6. Amazon S3

Amazon S3 is object storage for files.

What it does:

- Stores files like images, PDFs, videos, CSVs, and backups.
- Keeps files in buckets.

Why a .NET developer uses it:

- Your API can upload files to S3 using the AWS SDK for .NET.
- You can generate pre-signed URLs so users can upload or download safely.

Simple example:

- User uploads a profile image.
- .NET API sends the file to S3.
- Database stores only the file URL or key.

Important concept:

- S3 is for files, not relational rows.
- You usually do not store the actual file bytes inside SQL tables.

#### 7. Amazon CloudFront

Amazon CloudFront is a CDN.

What it does:

- Delivers static content from edge locations close to the user.
- Makes downloads and page loads faster.

Why a .NET developer uses it:

- If your app stores images or static files in S3, CloudFront can sit in front of S3.
- Users get faster access and lower latency.

Simple example:

- A user in one region opens an image.
- CloudFront serves it from a nearby edge location instead of the origin bucket every time.

Important concept:

- CloudFront is not the storage itself.
- It is the delivery layer in front of storage or an origin server.

#### 8. AWS Lambda

AWS Lambda runs code without a server.

What it does:

- Executes small pieces of code when an event happens.
- Scales automatically.

Why a .NET developer uses it:

- Great for small background jobs, event handlers, or file processing.
- You write a function, connect a trigger, and AWS runs it.

Simple example:

- A file is uploaded to S3.
- Lambda triggers and creates a thumbnail.

Important concept:

- Lambda is best for short, event-driven tasks.
- It is not ideal for long-running heavy workloads.

#### 9. Amazon CloudWatch

Amazon CloudWatch is for monitoring and logging.

What it does:

- Collects logs, metrics, and alarms.
- Helps you see what your app is doing in production.

Why a .NET developer uses it:

- You can send app logs from containers or Lambda to CloudWatch Logs.
- You can track CPU, memory, queue length, errors, and custom business metrics.

Simple example:

- Set an alarm if error rate becomes too high.
- Check logs when a request fails.

Important concept:

- Monitoring is how you know a service is healthy.
- Logging is how you debug when something goes wrong.

#### 10. AWS X-Ray

AWS X-Ray is for distributed tracing.

What it does:

- Shows how one request moves through multiple services.
- Helps find slow steps or failing calls.

Why a .NET developer uses it:

- In microservices, one request may touch many APIs.
- X-Ray helps you see where time is spent.

Simple example:

- Frontend calls API Gateway.
- API Gateway calls Service A.
- Service A calls Service B.
- X-Ray shows which part was slow.

Important concept:

- Metrics tell you something is wrong.
- Traces tell you where it is wrong.

#### 11. Amazon SES

Amazon SES is an email sending service.

What it does:

- Sends transactional and bulk emails.
- Handles deliverability features like reputation and bounce tracking.

Why a .NET developer uses it:

- Your application can send password reset emails, receipts, alerts, and verification emails.
- You can call SES from .NET using the AWS SDK or SMTP.

Simple example:

- User registers.
- .NET app sends a verification email through SES.

Important concept:

- SES is better than building your own SMTP mail server.

#### 12. Amazon SNS

Amazon SNS is a publish/subscribe notification service.

What it does:

- Sends a message to many subscribers at once.
- Can fan out to email, SQS, Lambda, HTTP endpoints, and more.

Why a .NET developer uses it:

- Use SNS when one event needs to reach multiple systems.
- It works well for notifications and event broadcasting.

Simple example:

- Order completed event is published to SNS.
- One subscriber sends email.
- Another subscriber writes to a queue.
- Another triggers a Lambda function.

Important concept:

- SQS is a queue for one consumer pattern.
- SNS is a broadcast model for multiple consumers.

### Quick mental model for a .NET developer

- API Gateway = entry point and router
- SQS = queue for async work
- ECS Fargate = container hosting
- RDS = relational database
- Redis = fast cache
- S3 = file storage
- CloudFront = global delivery/CDN
- Lambda = serverless event handler
- CloudWatch = logs and monitoring
- X-Ray = tracing
- SES = email
- SNS = pub/sub notifications

### What about Bonmojo?

For Bonmojo, the material in my notes is mainly Azure-based, such as App Service, Azure SQL, Service Bus, Blob Storage, Application Insights, Key Vault, CDN, and Redis.

So if the interviewer asks, "Did you use AWS in Bonmojo?" the safer and more correct answer is:

"No, Bonmojo was mainly Azure-based. My stronger AWS usage was in the OSP-style project."

### Best interview answer

In my OSP project, I used AWS API Gateway, SQS, ECS Fargate, RDS, ElastiCache Redis, S3, CloudFront, Lambda, CloudWatch, X-Ray, SES, and SNS. These covered API routing, messaging, container hosting, database, caching, file storage, CDN, background processing, monitoring, tracing, email, and notifications. Bonmojo, on the other hand, was mainly Azure-based, so I would not incorrectly claim the same AWS stack there.
