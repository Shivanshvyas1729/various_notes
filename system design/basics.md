What we learned:-
1) There is no right answer in system design, there are only trade offs. Trade off means to optimize something we need to sacrifice or break something else. 
2) Client-Server Architecture --> req and response cycle
3) HLD v/s LLD --> LLD is asked less in interview now (practical coding), HLD is more important(no coding) 
4) HLD has 9 principles:-
    a) Load / Scalability
    b) Performance / Latency
    c) Availability  (it can never be 100% but can be 99%, 99.9%, 99.99%, 99.999% --> it means the system will be available for 99% time of the year i.e. 365 days)
    d) Reliability / fault tolerance
    e) Consistency
    f) Durability
    g) Security
    h) Cost
    i) Maintainability 
5) Example of instagram and bank system (bank needs to be more reliable than insta, insta needs to be more available than bank)



# System Design – Introduction & Fundamentals

## What We Learned

### 1. There Is No Right Answer in System Design

- In System Design, there is no single correct or perfect answer. There are only **trade-offs**.
- Trade-off means that to optimize one thing, we might have to sacrifice or compromise on something else.
- We cannot make everything perfect in a system.

**Example: Travelling from Delhi to Patna**

We have three options:

- Flight
- AC 3-Tier Train
- General Train

Each option has its own advantages and disadvantages.

| Factor  | Flight    | AC 3-Tier | General  |
| ------- | --------- | --------- | -------- |
| Time    | Best      | Medium    | Medium   |
| Cost    | Expensive | Medium    | Cheapest |
| Comfort | Medium    | Best      | Worst    |

Now, which option is the best?

There is no single answer.

- If time is important, choose a flight.
- If saving money is important, choose the general train.
- If comfort is important, choose AC 3-Tier.

**Key takeaway:**

Before designing any system, we need to understand the requirements and priorities.

We need to ask:

- What is most important for the user?
- What can we compromise on?
- What are the business requirements?

We cannot optimize everything at the same time.

This is called a **Trade-off**.

Remember: There are no perfect system designs, only designs that are suitable for particular use cases.

---

### 2. Client-Server Architecture

Before understanding System Design, we need to understand how the internet works.

Generally, there are two types of machines connected to the internet:

1. **Client:** The machine that sends a request and asks for some data.
2. **Server:** The machine that receives the request, processes it and sends a response.

These machines communicate with each other using a **Request-Response Cycle**.

**Example: YouTube**

1. We open YouTube on our laptop.
2. We click on a particular video.
3. Our laptop sends a request to YouTube's server.
4. The server processes the request.
5. The server sends the video data as a response.
6. The video starts playing on our laptop.

**Important points:**

- Request means asking another machine for some data or service.
- Response means sending data or an answer back.
- A response can be correct, incorrect, delayed or even a denial.
- No response or timeout is also an outcome that the client needs to handle.

**Key takeaway:** Client-Server Architecture is the basic foundation of how applications communicate over the internet.

---

### 3. HLD vs LLD (High-Level Design vs Low-Level Design)

System Design is generally divided into two types:

#### A. High-Level Design (HLD)

HLD focuses on the overall architecture and planning of a software system.

Before writing code, we need to decide:

- What components will be present in the system?
- Which database will we use?
- What APIs will be required?
- How will different components communicate?
- How will data flow between components?
- How will the system handle users and requests?

**Example: Building a 40-floor apartment**

Before constructing the building, we need a proper plan.

We need to decide:

- Where will the lifts be?
- Where will the entrances be?
- How will water reach the top floors?
- Where will the security room be?
- How many people can the building accommodate?

This overall planning is similar to HLD.

**Important:** HLD focuses on architecture and system-level decisions. Generally, we do not need to write actual implementation code during an HLD interview.

#### B. Low-Level Design (LLD)

LLD focuses on the internal implementation and detailed design of individual components.

It includes:

- Classes and objects
- Methods and functions
- Internal component structure
- Design patterns
- Detailed implementation

**Example: Building a 40-floor apartment**

Once the overall building plan is ready, we decide:

- Which tiles will be used in each room?
- What will be the design of the windows?
- What paint will be used?
- What will the internal structure look like?

This detailed planning is similar to LLD.

**Important Difference:**

| HLD                                                                     | LLD                                                      |
| ----------------------------------------------------------------------- | -------------------------------------------------------- |
| Overall system architecture                                             | Internal component design                                |
| Focuses on planning                                                     | Focuses on implementation                                |
| How different components connect                                        | How individual components work                           |
| Generally no coding required                                            | Often involves coding                                    |
| More important in the context of this course's system design interviews | Also relevant, particularly in machine-coding interviews |

**Key takeaway:** HLD tells us how to build the overall system, whereas LLD tells us how to implement its individual components.

---

### 4. Nine Important Principles of HLD

Whenever we design a system, we need to consider nine important principles.

These principles help us decide what matters most for a particular system.

#### 1. Load / Scalability

**Load:** How many users or requests can our system handle?

**Scalability:** How can we increase the capacity of our system when the number of users grows?

Example:

Suppose we build an application for 100 users.

Initially, everything works perfectly.

But suddenly, 10,000 users start using the application.

Now we need to think:

- Can our system handle the increased traffic?
- Will the application crash?
- How can we increase its capacity?
- Will we need additional servers?

Two important concepts:

- Vertical Scaling
- Horizontal Scaling

We will study these concepts in detail later.

**Key takeaway:** We need to understand how much load the system should handle and how it can grow in the future.

#### 2. Performance / Latency

Latency means the time taken by the system to respond to a user's request.

Example:

Suppose a user sends a query to ChatGPT.

How much time should the system take to generate the response?

- 1 second?
- 5 seconds?
- 20 seconds?

This depends on the system requirements.

Some applications require very fast responses, while other applications can tolerate longer processing times.

**Key takeaway:** We need to define how quickly our system should respond to user requests.

#### 3. Availability

Availability means how much time our application or system remains operational and accessible to users.

For example:

- 99% availability
- 99.9% availability
- 99.99% availability
- 99.999% availability

100% availability is generally not realistic because servers, hardware, networks and other components can fail.

**Example:**

Suppose a website is expected to run throughout the year.

If the website is unavailable for some time due to server failure or other issues, its availability decreases.

Availability is generally expressed using percentages.

**Key takeaway:** We need to decide how much downtime our system can tolerate.

#### 4. Reliability / Fault Tolerance

Reliability means that the system should provide correct and expected results consistently.

A system being available does not necessarily mean that it is reliable.

**Example: ATM Machine**

Suppose we request ₹500 from an ATM.

But the ATM gives us only ₹400.

The ATM is working and is available, but it is not reliable because it has provided an incorrect result.

**Fault Tolerance:**

Fault tolerance means that even if one or more components fail, the entire system should continue functioning.

Example:

Suppose we have 10 servers.

One server goes down.

The entire application should not crash because of the failure of a single server.

The remaining servers should continue handling requests.

**Key takeaway:**

- Availability means the system is operational.
- Reliability means the system provides correct results consistently.
- Fault tolerance means the system can continue working even when some components fail.

#### 5. Consistency

Consistency means that users should see the correct and consistent data across the system.

**Example: Bank Transaction**

Suppose Person A transfers ₹100 to Person B.

After the transaction:

- ₹100 should be deducted from Person A's account.
- ₹100 should be added to Person B's account.

The transaction should not result in money being deducted from one account without being credited to the other.

We cannot afford incorrect or inconsistent financial data.

**Key takeaway:** Data should remain correct and consistent across the system.

#### 6. Durability

Durability means that once data is successfully stored, it should remain stored even if the system crashes or experiences a failure.

**Example: Instagram**

Suppose we create an Instagram account and gain 263 followers.

One day, Instagram's servers go down.

When Instagram becomes operational again, our account and its 263 followers should still be there.

The system should not lose previously stored information.

**Key takeaway:** Stored data should not disappear because of system crashes, power failures or other unexpected issues.

#### 7. Security

Security means protecting our system and its data from unauthorized access and attacks.

Important security concerns include:

- Data leakage
- Unauthorized access
- Hacking
- Data breaches
- Protecting user information

**Key takeaway:** We need to ensure that our application and user data remain secure.

#### 8. Cost

Cost means the overall expense required to build, operate and maintain our system.

We cannot simply keep adding servers or infrastructure to improve performance and scalability.

Every additional resource increases the cost.

Example:

Suppose our application is receiving more traffic.

We can add more servers to handle the load, but we also need to consider whether the business can afford that infrastructure.

**Key takeaway:** We need to balance performance, scalability and reliability with the available budget.

#### 9. Maintainability

Maintainability means that our system should be easy to understand, modify, debug and maintain.

Our system should not become so complicated that only the original developer understands how it works.

Example:

Suppose a developer leaves the company.

Another developer should be able to understand the codebase and continue working on the system.

For this, we need:

- Clean architecture
- Proper code organization
- Readable code
- Documentation
- Clear separation of responsibilities

**Key takeaway:** A system should be designed in such a way that it can be maintained and improved easily in the future.

---

### 5. Instagram vs Bank System – Understanding Trade-offs

This is one of the most important examples for understanding HLD principles.

Different systems have different requirements.

Not every system needs to prioritize all nine principles equally.

Let's compare Instagram and a Banking System.

#### A. Reliability

**Banking System:**

Suppose our bank balance is ₹2,00,000.

But the banking application displays ₹500.

This is a serious issue.

A banking system cannot afford to provide incorrect financial information.

Therefore, reliability and data correctness are extremely important for banks.

**Instagram:**

Suppose our Instagram reel has 90,000 views.

But Instagram temporarily displays 80,000 views.

This is not ideal, but it generally does not have the same financial consequences as an incorrect bank balance.

Instagram may tolerate some temporary inaccuracies in non-critical metrics.

**Conclusion:** Reliability and correctness are more critical for banking systems.

#### B. Availability

**Banking System:**

Suppose the banking system detects that it cannot process transactions correctly because of a serious failure.

It may be better to temporarily stop transactions rather than allow incorrect transactions to happen.

In other words, it may prioritize correctness over remaining available.

**Instagram:**

Suppose Instagram experiences an issue with some metrics.

It may still be preferable to keep the application running so users can continue watching reels and using other features.

**Conclusion:** Instagram generally places a very high priority on availability and user experience, while a banking system may sacrifice availability temporarily to protect transaction correctness.

#### Comparison Table

| Principle       | Instagram                                                          | Banking System                              |
| --------------- | ------------------------------------------------------------------ | ------------------------------------------- |
| Availability    | Very important                                                     | Very important                              |
| Reliability     | Important                                                          | Extremely important                         |
| Consistency     | Some temporary inconsistency may be tolerable for certain features | Critical for financial transactions         |
| Durability      | Very important                                                     | Extremely important                         |
| Security        | Very important                                                     | Extremely important                         |
| Performance     | Fast user experience is important                                  | Fast and correct transactions are important |
| Cost            | Important                                                          | Important                                   |
| Scalability     | Extremely important                                                | Extremely important                         |
| Maintainability | Important                                                          | Important                                   |

**Key takeaway:**

When designing Instagram, we may prioritize availability and scalability.

When designing a banking system, we must give very high priority to reliability, consistency, security and durability.

There is no universal design that works perfectly for every system.

Everything depends on the use case and business requirements.

---

## Final Summary – Things to Remember

1. System Design has no single correct answer. There are only trade-offs.
2. Before designing a system, understand the requirements and priorities.
3. Client-Server Architecture works on a Request-Response cycle.
4. HLD focuses on overall architecture, whereas LLD focuses on detailed implementation.
5. HLD is about deciding what components are required and how they communicate.
6. There are nine important principles of HLD:
   - Load / Scalability
   - Performance / Latency
   - Availability
   - Reliability / Fault Tolerance
   - Consistency
   - Durability
   - Security
   - Cost
   - Maintainability
7. Different applications prioritize different principles.
8. A banking system prioritizes correctness and data consistency, while Instagram can tolerate certain temporary inaccuracies to keep the experience available.
9. During a System Design interview, don't immediately jump to the solution. First understand the requirements, discuss trade-offs and then design the system.

**Remember: System Design is not about finding a perfect solution. It is about making the right engineering decisions based on the requirements.**

Add a quick revision cheat sheetClarify HLD versus LLD interview context
