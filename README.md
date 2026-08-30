He seems to be saying:

“Before Spring Boot hides all the plumbing from you, build a tiny application where you personally deal with HTTP → Servlet → Java → JDBC → DB → response → UI. Then you will understand what Spring Boot is automating.”

That interpretation fits your notes very well.

One correction to the specification you received: “no dependencies/libraries” and “H2 + Servlets + Tomcat” cannot literally all be true. A servlet application needs a servlet API/container, and H2 needs its JDBC driver. So I would phrase the requirement as “No application frameworks such as Spring Boot, Hibernate/JPA, Lombok, frontend frameworks, etc.; use raw Servlets + JDBC.”

1. The project you should convey to your teammate

You can send your friend this:

Mini Project — Employee & Company Management System

We need to build a small Java web application without Spring Boot or other frameworks, mainly to understand what frameworks normally do for us.

The application manages two entities:

Company
Employee

An employee belongs to one company, so there is a simple Company → Employee relationship.

UI

The page will have a simple two-part layout:

Left side

Company
Employee

Right side

Displays the records for whichever tab is selected.

For example, clicking Company displays:

ID	Company Name	Location
1	Citi	Chennai
2	ABC Corp	Bangalore

Clicking Employee displays:

ID	Name	Role	Email	Company ID
1	Arun	Developer	arun@email.com	1
2	Priya	Analyst	priya@email.com	2

No sophisticated UI or styling is required. Functionality and code structure are the focus.

CRUD operations
1. Create

An Add New button will be available outside the table.

Clicking it opens an empty form.

For example, for Employee:

Name:       __________
Role:       __________
Email:      __________
Company ID: __________

[Create] [Reset] [Cancel]

Clicking Create sends the data to the backend, which inserts a new employee into the database.

2. Read

When Company or Employee is selected:

UI
 ↓
JavaScript
 ↓
Servlet
 ↓
DAO
 ↓
JDBC
 ↓
Database

The database records are retrieved and displayed in the table.

3. Update

Clicking an existing row opens the form populated with that record.

For example:

Name:       Priya
Role:       Analyst
Email:      priya@email.com
Company ID: 2

[Update] [Delete] [Reset] [Cancel]

The user can modify the values and click Update.

The backend performs an SQL UPDATE.

4. Delete

When an existing record is opened, clicking Delete removes that record from the database.

For the initial version, keep deletion simple. If a company has employees associated with it, we can either prevent deletion or define the FK behavior explicitly rather than introducing complicated cascading logic.

5. Reset

Reset should have a clear, consistent meaning.

For a new record:

Name:       ABC
Role:       Developer

→ Reset

Name:
Role:

It clears the entered values.

For an existing record:

Name: Priya
Role: Developer

→ user changes Role to Manager

→ Reset

Name: Priya
Role: Analyst

It restores the values originally loaded from the database.

Reset should not itself update the database.

Cancel simply closes the form without saving anything.

2. Backend — the important part of this exercise

This is where your manager is probably evaluating you.

We should deliberately not use Spring Boot.

The flow should be something like:

Browser
   ↓
HTML + Vanilla JavaScript
   ↓
HTTP request
   ↓
Servlet
   ↓
Service / business logic
   ↓
DAO
   ↓
JDBC
   ↓
Database

And the response comes back:

Database
   ↓
DAO
   ↓
Service
   ↓
Servlet
   ↓
JSON response
   ↓
JavaScript
   ↓
HTML table
Important separation
webapp/

Contains:

index.html
app.js

Responsible only for:

displaying UI
handling button clicks
opening/closing forms
calling backend APIs using fetch()
rendering returned data

No SQL here.

servlet/

Example:

CompanyServlet.java
EmployeeServlet.java

Responsible for:

receiving HTTP requests
identifying GET/POST/PUT/DELETE operations
reading request parameters/body
calling appropriate backend logic
returning HTTP/JSON responses

No SQL queries here.

service/

Example:

CompanyService.java
EmployeeService.java

Responsible for:

business logic
validation
deciding what operation should happen
calling DAO

This layer is useful because later you can see how Spring's service-related features simplify this kind of structure.

dao/

Example:

CompanyDAO.java
EmployeeDAO.java

This is where JDBC and SQL live.

For example:

SELECT * FROM COMPANY
INSERT INTO COMPANY ...
UPDATE COMPANY ...
DELETE FROM COMPANY ...

The Servlet should never contain these SQL statements.

model/

Example:

Company.java
Employee.java

These are plain Java classes representing database records.

For example:

Employee
---------
id
name
role
email
companyId
3. Suggested project structure

A simple version could look like:

employee-company-app/
│
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/app/
│       │       │
│       │       ├── model/
│       │       │   ├── Company.java
│       │       │   └── Employee.java
│       │       │
│       │       ├── dao/
│       │       │   ├── CompanyDAO.java
│       │       │   └── EmployeeDAO.java
│       │       │
│       │       ├── service/
│       │       │   ├── CompanyService.java
│       │       │   └── EmployeeService.java
│       │       │
│       │       ├── servlet/
│       │       │   ├── CompanyServlet.java
│       │       │   └── EmployeeServlet.java
│       │       │
│       │       └── util/
│       │           └── DBConnection.java
│       │
│       └── webapp/
│           ├── index.html
│           └── app.js
│
└── WEB-INF/
    └── web.xml

Depending on how your manager wants the project configured, web.xml may be replaced/partially replaced by servlet annotations such as @WebServlet.

And if Tomcat's connection pool is specifically part of the exercise, there may also be:

META-INF/
    context.xml
4. Database

Keep the database very small.

COMPANY
COMPANY
-------
ID
NAME
LOCATION
EMPLOYEE
EMPLOYEE
--------
ID
NAME
ROLE
EMAIL
COMPANY_ID

Relationship:

COMPANY
   1
   |
   |
   N
EMPLOYEE

Example:

Company
1 | Citi | Chennai

Employees
1 | Arun  | Developer | ... | 1
2 | Priya | Analyst   | ... | 1

That's enough.

You don't need authentication, pagination, search, sorting, fancy validation, dashboards, charts, etc.

5. The final AI prompt I'd actually give to an AI coding tool

I would not give the AI the previous specification verbatim because it is already making some implementation decisions for you.

Give it this instead:

Build a small Java web application called Employee-Company Management System.

The purpose of this project is educational: I want to understand the plumbing that frameworks such as Spring Boot normally abstract away.

Constraints
Do NOT use Spring Boot.
Do NOT use Spring MVC.
Do NOT use Hibernate/JPA.
Do NOT use Lombok.
Do NOT use frontend frameworks such as React or Angular.
Use plain HTML and vanilla JavaScript for the frontend.
Use Java Servlets for handling HTTP requests.
Use JDBC for database access.
Use a relational database with two tables: COMPANY and EMPLOYEE.
Use only the minimum required runtime/container/JDBC dependencies. Do not introduce additional frameworks or libraries.
No effort is required for visual design; keep the UI simple.
Functional requirements

The application has two tabs on the left:

Company
Employee

Selecting a tab displays the corresponding database records on the right.

Company fields:

ID
Name
Location

Employee fields:

ID
Name
Role
Email
Company ID

Provide an Add New button outside the table.

Clicking Add New opens an empty form.

Clicking an existing row opens a form populated with that record.

Existing records must support:

Update
Delete
Reset
Cancel

New records must support:

Create
Reset
Cancel

Reset on a new record clears the entered values.

Reset on an existing record restores the values originally loaded into the form.

Cancel closes the form without saving changes.

Architecture

Maintain clear separation of concerns:

HTML/JavaScript
       ↓
Servlet
       ↓
Service
       ↓
DAO
       ↓
JDBC
       ↓
Database

SQL must exist only in the DAO layer.

Servlet classes must not contain SQL.

UI JavaScript must not contain SQL or database logic.

Model classes should represent Company and Employee.

Error handling

Handle database/SQL errors in the backend and return appropriate HTTP status codes and simple JSON error responses to the frontend.

Code quality

Use meaningful Java class, method and variable names.

Follow standard Java naming conventions.

Keep responsibilities separated into appropriate packages.

Avoid unnecessary abstraction because this is a small learning project.

Important: Do not generate the entire project blindly. First explain the responsibility of each file and the request flow for Create, Read, Update and Delete. Then generate the implementation one layer at a time so I can understand each part.

That last paragraph is particularly important for you.

6. How big is this actually?

It's small from a business perspective, but not tiny for a beginner because you're deliberately removing Spring Boot.

I'd roughly break it down like this:

Part	Difficulty
HTML layout	🟢 Easy
Vanilla JS tab switching	🟢 Easy
Form/modal handling	🟢 Easy–Medium
Java model classes	🟢 Easy
SQL tables	🟢 Easy
Basic JDBC	🟡 Medium
DAO	🟡 Medium
Servlets	🟡 Medium
HTTP request/response	🟡 Medium
JSON handling	🟡 Medium
Tomcat configuration	🟠 Medium
Connection pooling	🟠 Medium–Hard
Debugging entire flow	🟠 Medium–Hard

For you as a beginner with Java/Spring Boot knowledge but limited raw Servlet/JDBC experience, I wouldn't consider this a huge project.

If you code it yourself

Roughly:

3–5 focused days to build it while actually understanding what you're doing.

Potentially 5–7 days if you're learning Servlets, JDBC and Tomcat from scratch.

If AI generates everything

You could probably get something running in a few hours.

But you'd miss the entire point of your manager's exercise.

7. So should you use AI or code everything yourself?

I'd do a hybrid approach, especially because your manager explicitly said he doesn't want blind AI usage.

Don't do:

"AI, build this entire application."

Then copy → paste → run → done.

Instead:

Phase 1 — You write it

Write these yourself:

Company.java
Employee.java
database tables
basic HTML
basic JavaScript

These should be straightforward.

Phase 2 — Learn raw JDBC

Before asking AI to generate anything, understand this flow:

Connection
   ↓
PreparedStatement
   ↓
executeQuery / executeUpdate
   ↓
ResultSet
   ↓
Java object

You should be able to explain what every one of those does.

Phase 3 — Write one DAO yourself

For example:

CompanyDAO.getAllCompanies()

Then ask AI:

"Review my DAO. Don't rewrite it. Tell me what is wrong and why."

That's much better learning than generating it.

Phase 4 — Write one Servlet

Understand:

HTTP request
      ↓
Servlet
      ↓
Service
      ↓
DAO
      ↓
DB
      ↓
response

Once you've understood one, Employee becomes largely repetition.

Phase 5 — Use AI as a reviewer

Instead of:

"Generate EmployeeServlet."

Ask:

"Here is my EmployeeServlet. Review it for Java/Servlet best practices. Don't modify the code. Explain each problem and let me fix it."

That will make you much stronger when you eventually start using Spring Boot.

8. The really important thing your manager is trying to teach

Later, when you use Spring Boot, you'll see something like:

@RestController
public class EmployeeController {

    @GetMapping("/employees")
    public List<Employee> getEmployees() {
        return employeeService.getEmployees();
    }
}

It looks ridiculously simple.

But right now, you're going to discover that underneath it there is actually:

Browser
   ↓
HTTP
   ↓
Tomcat
   ↓
Servlet
   ↓
Request parsing
   ↓
Controller
   ↓
Service
   ↓
DAO
   ↓
Connection
   ↓
PreparedStatement
   ↓
SQL
   ↓
Database
   ↓
ResultSet
   ↓
Java objects
   ↓
JSON
   ↓
HTTP response
   ↓
Browser

That is the real value of this exercise.

Spring Boot later essentially lets you say:

"I want an endpoint here."

instead of manually wiring a large portion of that plumbing.

And similarly, JDBC teaches you why things such as:

connection management
connection pooling
prepared statements
exception handling
dependency injection
controllers
services
repositories
object mapping

exist in the first place.

One final recommendation

I would not start with connection pooling.

Build the first version in this order:

1. Database
      ↓
2. Company/Employee Java models
      ↓
3. JDBC connection
      ↓
4. DAO
      ↓
5. Servlet
      ↓
6. Test API manually
      ↓
7. HTML
      ↓
8. JavaScript fetch()
      ↓
9. Connect UI + backend
      ↓
10. Create/Update/Delete
      ↓
11. Error handling
      ↓
12. Connection pooling

That way, when something breaks, you know which layer is responsible rather than having a 1,000-line AI-generated project that you don't understand.

And yes: this is a very reasonable first project before moving into Spring Boot. It is small enough to finish, but it exposes exactly the kind of underlying plumbing your manager wants you to appreciate.

so you are using h2 database or smtg else here and relationship is 1 emp belongs to 1 company or 1 company has n employees or is it both, and give me suggestions of how i build this myself with minimal ai used to my advantage so that i learn, like form where i get code see and write or ask ai to generate file by file or how

Yes. I would use H2 for this exercise if your manager is okay with it. H2 is a small relational database that is convenient for a local learning project. The important thing is that you interact with it through plain JDBC, not through JPA/Hibernate.

And the relationship is both ways conceptually, but it is one-to-many overall:

Company
   1
   │
   │ has
   │
   N
Employee

Meaning:

One Company can have many Employees
Each Employee belongs to one Company

So in the EMPLOYEE table:

COMPANY
----------------
ID
NAME
LOCATION


EMPLOYEE
----------------
ID
NAME
ROLE
EMAIL
COMPANY_ID  ← foreign key

For example:

COMPANY
1 | Citi     | Chennai
2 | Microsoft| Bangalore


EMPLOYEE
1 | Arun  | Developer | ... | 1
2 | Priya | Analyst   | ... | 1
3 | Ravi  | Developer | ... | 2

Here Arun and Priya belong to Citi, while Ravi belongs to Microsoft.

You don't need a separate relationship table because this is a simple 1:N relationship.

How I recommend YOU build this

Given what your manager told you, I would not ask AI to generate the whole application file-by-file and then read it.

That still becomes:

AI thinks → AI codes → you understand afterwards.

Instead, make it:

You think → you attempt → AI helps when stuck → you write → AI reviews.

That's much more valuable.

The approach I'd recommend
🟢 Stage 1 — Understand the architecture before coding

First, make yourself understand this:

             Browser
                │
          HTML + JS
                │
             HTTP
                ↓
           Servlet
                │
             Service
                │
              DAO
                │
             JDBC
                │
              H2

Before writing anything, you should be able to answer:

"If I click Create Employee, what exactly happens?"

Answer:

Click Create
     ↓
JavaScript gets form values
     ↓
fetch() sends HTTP request
     ↓
EmployeeServlet receives request
     ↓
EmployeeService validates/processes it
     ↓
EmployeeDAO executes INSERT using JDBC
     ↓
H2 stores employee
     ↓
DAO returns result
     ↓
Servlet sends response
     ↓
JavaScript refreshes employee table

If you understand this, the actual coding becomes much easier.

🟢 Stage 2 — Build the database yourself

Don't ask AI to create the database.

Write the SQL yourself.

Something like:

CREATE TABLE COMPANY (
    ID INT PRIMARY KEY,
    NAME VARCHAR(100),
    LOCATION VARCHAR(100)
);

Then:

CREATE TABLE EMPLOYEE (
    ID INT PRIMARY KEY,
    NAME VARCHAR(100),
    ROLE VARCHAR(100),
    EMAIL VARCHAR(100),
    COMPANY_ID INT,
    FOREIGN KEY (COMPANY_ID) REFERENCES COMPANY(ID)
);

Insert 2–3 companies and employees manually.

At this point, understand:

primary key
foreign key
one-to-many relationship
why COMPANY_ID is in Employee
what happens if you give an employee a non-existent company ID

Don't move on until this makes sense.

🟢 Stage 3 — Write the Model classes yourself

You don't need AI for this.

Create:

Company.java
Employee.java

And try writing:

public class Company {
    private int id;
    private String name;
    private String location;

    // constructors
    // getters
    // setters
}

And similarly Employee.

This is easy enough that you should experience writing it yourself.

If you get stuck on something, ask me:

"I'm writing Employee.java. Why do I need getters and setters here?"

rather than:

"Generate Employee.java."

🟡 Stage 4 — Learn JDBC BEFORE creating DAO

This is probably the most important learning part.

Don't immediately create EmployeeDAO.

First make a tiny standalone Java program that does:

Java program
    ↓
JDBC
    ↓
H2

Learn this sequence:

Connection
    ↓
PreparedStatement
    ↓
executeQuery()
    ↓
ResultSet

and for INSERT/UPDATE/DELETE:

Connection
    ↓
PreparedStatement
    ↓
executeUpdate()

You should manually write a small program that:

connects to H2
inserts an employee
retrieves employees
prints them
updates one
deletes one

Once you can do that, DAO will make sense rather than looking like mysterious boilerplate.

🟡 Stage 5 — Now create DAO

Now create:

CompanyDAO.java
EmployeeDAO.java

You try writing methods such as:

getAllCompanies()
getCompanyById()
createCompany()
updateCompany()
deleteCompany()

and:

getAllEmployees()
getEmployeeById()
createEmployee()
updateEmployee()
deleteEmployee()

You will notice that a lot of JDBC code repeats.

This is exactly the kind of pain your manager wants you to experience.

You might think:

"Why am I repeatedly opening connections, creating PreparedStatements, handling ResultSets and SQLExceptions?"

Good. 😂

That's part of the lesson.

🟡 Stage 6 — THEN use AI strategically

This is where AI becomes extremely useful.

Suppose you've written:

public List<Employee> getAllEmployees() {
    // your attempt
}

and you're stuck.

Don't say:

"Generate EmployeeDAO."

Instead ask:

"I'm learning JDBC and implementing EmployeeDAO myself. I attempted getAllEmployees() below. Don't rewrite the complete method. Explain what steps are missing and give me hints so I can finish it myself."

Then you write it.

Afterwards:

"Review my implementation. Don't rewrite it. Point out bugs, bad practices and anything I should improve."

This is probably the best use of AI for this project.

🟡 Stage 7 — Learn Servlets

Once DAO works independently, introduce:

EmployeeServlet
CompanyServlet

Start with only GET.

For example:

GET /employees

should eventually do:

Browser
   ↓
EmployeeServlet
   ↓
EmployeeService
   ↓
EmployeeDAO
   ↓
H2

Don't build CRUD all at once.

Get READ working first.

Then:

GET → Read
POST → Create
PUT → Update
DELETE → Delete

You'll learn HTTP semantics naturally this way.

🟢 Stage 8 — Only then build the frontend

Start with ugly HTML. Seriously. 😂

Something like:

Company | Employee

-----------------------------

ID | Name | Role | Email | Company
1  | Arun | Dev  | ...   | 1
2  | Priya| QA   | ...   | 1

[Add New]

Don't spend time making it beautiful.

First make:

HTML
 ↓
JavaScript
 ↓
fetch()
 ↓
Servlet

work.

Then add the modal/form.

🟢 Stage 9 — Finally implement the complete flow

Your final application becomes:

Read
Click Employee
      ↓
GET /employees
      ↓
EmployeeServlet
      ↓
EmployeeService
      ↓
EmployeeDAO
      ↓
JDBC
      ↓
H2
      ↓
JSON
      ↓
JavaScript
      ↓
Table
Create
Add New
 ↓
empty form
 ↓
Create
 ↓
POST
 ↓
Servlet
 ↓
Service
 ↓
DAO
 ↓
INSERT
 ↓
H2
Update
Click row
 ↓
populate form
 ↓
modify
 ↓
Update
 ↓
PUT
 ↓
DAO
 ↓
UPDATE
Delete
Click row
 ↓
Delete
 ↓
DELETE
 ↓
DAO
 ↓
DELETE FROM EMPLOYEE
🟠 Stage 10 — Connection pooling comes LAST

Your original specification mentions manual connection pooling.

Don't start there.

Initially learn:

Request
 ↓
getConnection()
 ↓
use connection
 ↓
close connection

Then understand the problem:

"If every request creates a new DB connection, isn't repeatedly creating connections expensive?"

Then learn connection pooling:

Connection Pool
 ┌─────────────────────┐
 │ Connection 1        │
 │ Connection 2        │
 │ Connection 3        │
 │ Connection 4        │
 └─────────────────────┘
          ↑
          │ borrow
       DAO
          │
          ↓
       return

That moment is exactly when frameworks/container infrastructure starts making more sense.

Your AI usage strategy

I'd use this rule:

Situation	What you should do
Simple Java class	Write yourself
SQL table	Write yourself
Don't understand concept	Ask AI to explain
JDBC syntax forgotten	Ask AI for a small example
Stuck implementing a method	Ask AI for hints, not solution
You've written code	Ask AI to review
Error/exception	Give AI the error + your code and debug together
Repetitive boilerplate	AI can generate it, but understand every line
Entire project	❌ Don't generate at once
Entire DAO	❌ Avoid initially
Entire Servlet	❌ Avoid initially
One particularly useful technique

Use AI like a senior developer sitting beside you, not like a code generator.

For example:

You:

I need to implement getAllEmployees() using JDBC. I know I need Connection, PreparedStatement and ResultSet, but I'm not sure about the sequence. Give me the conceptual steps, not code.

AI explains.

Then you code it.

Then:

Here's my implementation. Review it but don't rewrite it.

AI reviews.

Then you fix it.

Then:

Why should I use PreparedStatement instead of Statement here?

You understand.

That's significantly better than copying a generated DAO.

How much AI should you actually use?

For this particular project, I'd aim roughly for:

70% you + 30% AI.

Not because AI is bad, but because the project itself is deliberately designed as a learning exercise.

For your actual Spring Boot project later, you can comfortably shift toward:

30% you + 70% AI, provided you can understand, test and explain the generated code.

And I would NOT use tutorials as your primary source

You can certainly learn from tutorials, but I'd use a small targeted learning → immediately implement approach.

For example:

Learn: JDBC basics
       ↓
Implement JDBC yourself
       ↓
Learn: Servlet basics
       ↓
Implement one Servlet yourself
       ↓
Learn: fetch()
       ↓
Implement frontend call
       ↓
Connect everything

Don't spend three days watching "Complete Java Servlet Course" before writing a line of code.

Your ideal development sequence

If I were guiding you through this project, I'd make you build exactly this:

DAY 1
─────
Understand architecture
↓
Create H2 DB
↓
Create COMPANY + EMPLOYEE
↓
Insert test data
↓
Write Company.java + Employee.java


DAY 2
─────
Learn JDBC
↓
Connect Java → H2
↓
INSERT
↓
SELECT
↓
UPDATE
↓
DELETE


DAY 3
─────
Create DAO
↓
Move JDBC code into DAO
↓
Create Service
↓
Create Servlet
↓
Test backend independently


DAY 4
─────
Create HTML
↓
Create vanilla JS
↓
GET employees
↓
GET companies
↓
Display tables


DAY 5
─────
Create
↓
Update
↓
Delete
↓
Reset
↓
Error handling


DAY 6
─────
Connection pooling
↓
Clean up code
↓
Understand what Spring Boot would
eventually automate

You don't necessarily need six full days; this is the learning sequence, not a strict schedule.

And honestly, I would build it this way rather than having AI generate file-by-file from the beginning. You will struggle more initially, but by the end you'll be able to look at a Spring Boot controller/repository/service and understand why those things exist, which is almost certainly what your manager is trying to achieve.
