# Project Based Answers 2

## Index

- [Q1: Tell me about one query you optimized and how (SQL)](#q1-tell-me-about-one-query-you-optimized-and-how-sql)
- [Q2: What is an N+1 query problem?](#q2-what-is-an-n1-query-problem)
- [Q3: How would you optimize an API returning 1 million records?](#q3-how-would-you-optimize-an-api-returning-1-million-records)
- [Q4: What does SonarQube do?](#q4-what-does-sonarqube-do)

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
