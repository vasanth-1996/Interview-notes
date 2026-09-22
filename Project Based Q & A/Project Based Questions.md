## 🎯 If I were your interviewer, these are the questions I'd ask

### 1. First: "Explain your current/recent project"

I'd start with:

> **"Vasanth, take one of your recent projects and explain the complete architecture to me, from the client request until the response."**

Then I would drill down:

1. What is the business problem?
2. What are the different components?
3. Why did you choose .NET Core?
4. Is it monolith or microservices?
5. How many services are there?
6. How do services communicate?
7. Where is authentication handled?
8. Where is authorization handled?
9. Where is the database?
10. How does the application get configuration?
11. How are secrets stored?
12. How do you deploy it?
13. How do you monitor it?
14. What happens if one service goes down?
15. What happens if the database goes down?
16. How do you handle retries?
17. How do you handle logging?
18. How do you handle exceptions?

**This is the most important area to prepare.**

---

# 🟦 Bupa — Azure Functions / Batch Jobs

Your resume says you worked on enterprise Azure Functions, Oracle HUGO, Durable Functions, Application Insights, Serilog, Clean Architecture, xUnit/Moq, SonarQube and Checkmarx. 

An interviewer will almost certainly attack this area.

### Basic

1. **Why did you use Azure Function instead of a normal .NET Web API?**
2. What type of Azure Function did you use?
3. What triggered the function?
4. Was it Timer Trigger, Queue Trigger, HTTP Trigger, etc.?
5. Why did you choose that trigger?
6. How frequently does the batch run?
7. What happens if the function fails?
8. How do you retry failed jobs?
9. How do you prevent duplicate processing?

### Durable Functions

10. **Why did you use Durable Functions?**
11. What is an Orchestrator Function?
12. What is an Activity Function?
13. Why shouldn't you put normal business logic directly inside the orchestrator?
14. What happens if an activity fails?
15. How does Durable Functions maintain state?
16. How would you process 100,000 records?
17. Would you process them sequentially or in parallel?
18. How do you control concurrency?
19. How do you make the batch idempotent?

### Production scenarios

20. **Suppose your batch processes 50,000 records and crashes after 30,000. What happens?**
21. How would you resume from 30,001?
22. What if the same record gets processed twice?
23. What if Oracle is temporarily unavailable?
24. What if SQL Server is unavailable?
25. What if one record is invalid?
26. Should the entire batch fail?
27. How do you identify which records failed?

### Monitoring

28. What did you monitor using Application Insights?
29. What is the difference between logs, metrics and traces?
30. How would you investigate a production batch that suddenly takes 3 hours instead of 20 minutes?
31. How do you correlate logs across multiple functions?

---

# 🟩 Azure Security

Because you have Azure + Key Vault on your resume, I would ask:

32. **Where do you store database passwords?**
33. Why not store them in `appsettings.json`?
34. How does Azure Function access Key Vault?
35. What is Managed Identity?
36. System-assigned vs user-assigned Managed Identity?
37. Does your application know the Key Vault password?
38. How do you give only required permissions?
39. What happens if somebody gets the Function configuration?
40. How do you rotate secrets?

And because you have been studying **Entra ID/APIM**, I would extend this into:

41. How does Entra ID issue a JWT?
42. Who validates the JWT?
43. Authentication vs authorization?
44. What is a scope?
45. What is a role/claim?
46. Where would APIM fit?
47. Why use APIM if the API already validates JWT?
48. How would you secure service-to-service communication?

---

# 🟨 OSP — Online School Payments

This is probably the **best project on your resume for a senior-level interview** because it has payments, users, roles, districts, schools, transactions, refunds and reporting. Your resume describes it as a multi-district K-12 payment platform with role-based access and district-level configuration. 

I'd ask:

### Architecture

49. **Explain the architecture of OSP.**

50. Who are the users?

51. What are the major modules?

52. How do you model:

```text
District
   ↓
School
   ↓
Student
   ↓
Parent
   ↓
Fee
   ↓
Payment
```

53. How do you ensure one district cannot access another district's data?

54. What is multi-tenancy in your project?

55. How did you implement tenant isolation?

---

### Payment scenarios

56. **A parent clicks Pay twice. What happens?**

57. How do you prevent duplicate payments?

58. What is idempotency?

59. Payment succeeds but your API crashes before saving the transaction. What happens?

60. Database transaction vs payment transaction — are they the same?

61. What happens if payment provider says SUCCESS but your database says FAILED?

62. How would you reconcile them?

63. How do you handle refunds?

64. How do you maintain an audit trail?

These are **excellent senior-level questions**.

---

# 🔐 Authentication / Authorization

Your resume specifically mentions custom authorization and ASP.NET Identity customization. 

I'd ask:

65. Why did you customize ASP.NET Identity?

66. How does ASP.NET Identity work?

67. Where are users stored?

68. How is the password stored?

69. What happens during login?

70. Explain the complete JWT flow.

```text
User
 ↓
Login
 ↓
Authentication
 ↓
JWT
 ↓
API
 ↓
JWT Validation
 ↓
Authorization
 ↓
Controller
```

71. What is inside a JWT?

72. Should we store sensitive information inside JWT?

73. What is the difference between authentication and authorization?

74. Role-based vs policy-based authorization?

75. How did you implement authorization in OSP?

---

# 🗄️ SQL Server / PostgreSQL

Your resume specifically mentions complex query design and performance tuning. 

So I would definitely test whether that is real experience.

76. **Tell me about one query you optimized.**

77. What was the original problem?

78. How did you identify the bottleneck?

79. What was the execution plan?

80. Did you add an index?

81. Why did you choose that index?

82. What is a clustered index?

83. Non-clustered index?

84. Composite index?

85. What happens if you create too many indexes?

86. What is an N+1 query problem?

87. How does EF Core generate SQL?

88. `IEnumerable` vs `IQueryable`?

89. Tracking vs `AsNoTracking()`?

90. Lazy loading vs eager loading?

91. How would you optimize an API returning 1 million records?

---

# 🟧 EF Core

I'd ask:

92. What is DbContext?

93. Is DbContext thread-safe?

94. What is Dependency Injection lifetime of DbContext?

95. Scoped vs Singleton vs Transient?

96. What is change tracking?

97. What is a migration?

98. Code First vs Database First?

99. What happens when you call:

```csharp
SaveChanges()
```

100. How do you handle transactions in EF Core?

101. How do you prevent SQL injection?

102. Does using EF Core automatically guarantee good performance?

---

# ☁️ AWS

Your OSP and Bonmojo projects mention AWS. 

I'd ask:

103. Which AWS services did you actually use?

104. Why did you use AWS instead of Azure?

105. How did your .NET API communicate with AWS?

106. What messaging service did you use?

107. Why asynchronous messaging?

108. Queue vs topic?

109. What happens if message processing fails?

110. How do you retry?

111. What is a dead-letter queue?

112. How do you ensure a message isn't processed twice?

---

# 🐳 Docker / Kubernetes

Since you're presenting yourself as someone familiar with DevOps and modern architecture, I'd ask:

113. Why Docker?

114. What goes inside a Docker image?

115. Image vs container?

116. What is a Dockerfile?

117. How do you configure environment-specific settings?

118. What is Kubernetes?

119. Pod vs Deployment vs Service?

120. Who creates a new pod when one crashes?

121. Readiness vs liveness probe?

122. What happens if readiness fails?

123. How would you deploy 5 replicas of your API?

---

# 🧱 Microservices

Your resume explicitly says microservices architecture and implementation. 

This is where a senior interviewer can go very deep.

124. **Why microservices?**

125. When would you NOT use microservices?

126. How do you identify service boundaries?

127. Should every microservice have its own database?

128. Why shouldn't all microservices share one database?

129. How do services communicate?

130. Synchronous vs asynchronous communication?

131. How do you handle distributed transactions?

132. What is Saga pattern?

133. What happens when Service A calls Service B and B is down?

134. Timeout?

135. Retry?

136. Circuit breaker?

137. What is eventual consistency?

138. How do you monitor a request traveling through 5 microservices?

---

# 🧪 Testing

Your resume says xUnit and Moq. 

I'd ask:

139. Unit test vs integration test?

140. Why Moq?

141. What should you mock?

142. What should you NOT mock?

143. How do you test a service that calls a database?

144. How do you test an external API?

145. What is code coverage?

146. Is 100% code coverage always good?

147. What makes a good unit test?

---

# 🔒 SonarQube / Checkmarx

Because you've explicitly put these on your resume, expect questions.

148. What does SonarQube do?

149. What is SAST?

150. What security issues did Checkmarx identify?

151. How did you fix them?

152. SQL injection?

153. XSS?

154. Hardcoded secrets?

155. Insecure deserialization?

156. Why shouldn't we simply suppress security warnings?

---

# 🤖 Cursor + Docker development tool

This is an interesting differentiator in your resume.

You mention building an internal development tool using Cursor and Docker to analyze Jira issues and GitHub repositories. 

I would **absolutely ask about this**, because it can reveal whether you genuinely built it.

157. **Explain this tool end-to-end.**

158. What problem were you solving?

159. How does it read Jira issues?

160. How does it access GitHub?

161. Where does MCP fit?

162. Why did you use MCP?

163. What tools did your MCP server expose?

164. How did you secure GitHub access?

165. How did you secure Jira credentials?

166. Can the AI modify the repository?

167. If yes, how did you control that?

168. How did you prevent prompt injection from a Jira ticket?

169. How did you prevent the AI from accessing unauthorized repositories?

170. Why Docker?

This could become a **very strong modern-project discussion** if you prepare it properly.

---

# 🔥 Finally, the questions I would use to detect "resume knowledge"

These are the questions I would ask when I suspect someone knows the technology but hasn't really worked on the project:

### Scenario 1

> **"Your API suddenly becomes 10x slower in production. What will you check first?"**

### Scenario 2

> **"Database CPU is 100%. Your API is timing out. Walk me through your investigation."**

### Scenario 3

> **"One of your microservices is down. What happens to the whole application?"**

### Scenario 4

> **"A batch processed the same customer twice. How would you investigate and fix it?"**

### Scenario 5

> **"Someone accidentally committed a database password to GitHub. What do you do?"**

### Scenario 6

> **"Your JWT is valid but the user should not be allowed to access this API. How do you stop them?"**

### Scenario 7

> **"Payment succeeded but your application crashed before saving the payment. How do you recover?"**

### Scenario 8

> **"An Azure Function works locally but fails in Azure. How do you troubleshoot it?"**

### Scenario 9

> **"A message is delivered twice to your consumer. How do you handle it?"**

### Scenario 10

> **"You have 1 million records to process. Your current batch takes 5 hours. How would you reduce it to 30 minutes?"**

---
