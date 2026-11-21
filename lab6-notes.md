Activity 1
1. What I Learned

In this lab, I learned how to connect a PHP application to an SQLite database using PDO. I learned how to create and store student tutoring requests in a database instead of only using form data.

2. Database Connection
I used config/db.php to create a PDO connection to the SQLite database. This allows the PHP application to communicate with the database.
I created the SQLite database using the following command:
sqlite3 database/tutoring.db < database/setup-database.sql
I opened the database using sqlite3 database/tutoring.db and ran .tables to verify that the tables were created successfully.
The output showed two tables:
students
tutoring_requests
This confirmed that both required database tables were created successfully.
I started the PHP application using the command:
php -S 0.0.0.0:8000
I opened the forwarded port 8000 in my browser. The application loaded successfully and displayed the tutoring request form. I confirmed that the application was running correctly.
I submitted a test tutoring request using sample student information. The application accepted the submission and displayed a confirmation message. I then opened the requests page at /php/requests.php and confirmed that my test request appeared in the list.
| Filter requests | Selected the course and Open status filters. | Only requests matching the selected filters should appear. | My test request appeared in the filtered results. | Pass |
| Edit Open request | Opened my test request, changed the topic or reason, and saved the changes. | The updated information should be saved and displayed. | My request was updated successfully. | Pass |
| Cancel Open request | Selected my Open test request and cancelled it. | The request status should change to Cancelled. | The request was successfully marked as Cancelled. | Pass |
| Check Cancelled request | Opened Request #3 after cancelling it and checked the available options. | A cancelled request should not be editable or cancellable again. | The request details page displayed Cancelled and only provided navigation back to the requests list. | Pass |
3. Database Tables

The database contains two tables:

students: Stores student names and email addresses.

tutoring_requests: Stores tutoring requests, including the course, topic, preferred date, preferred time, and status.

The tutoring requests table uses a foreign key to connect each request to a student.

I used a SQL JOIN query to display tutoring requests together with the student information connected to each request.

```sql
SELECT
    r.request_id,
    s.student_name,
    s.student_email,
    r.course,
    r.topic,
    r.preferred_date,
    r.preferred_time,
    r.status
FROM tutoring_requests AS r
JOIN students AS s
    ON r.student_id = s.student_id
ORDER BY r.request_id;
```

The JOIN connects the `students` and `tutoring_requests` tables using `student_id`. This allows the database to show each tutoring request with the student's name and email address.

4. PHP Files and Their Purpose

config/db.php: Opens the PDO connection to the SQLite database.

services/requestService.php: Validates submitted information and manages database transactions.

repositories/studentPDO.php: Finds students by email and prevents duplicate student records.

repositories/requestPDO.php: Contains SQL statements for creating, finding, filtering, updating, and cancelling tutoring requests.

The PHP application files have these responsibilities:

php/request-form.php: Displays the tutoring request form.

php/process-request.php: Receives and processes submitted form information.

php/requests.php: Displays and filters tutoring requests.

php/request-details.php: Displays an individual tutoring request.

php/edit-request.php: Displays the edit form.

php/update-requests.php: Processes request updates.

php/delete-request.php: Handles request cancellation.

php/router.php: Routes application requests.

Why SQL belongs in repository files:

SQL belongs in repository files because they keep database operations separate from the pages users interact with. This makes the application easier to organize, maintain, and update. It also keeps database queries from being mixed directly into the form pages.
5. Testing

I ran the integration tests using:

php tests/integration-tests.php

All 14 integration checks passed. These tests checked creating requests, reusing student records, filtering requests, rejecting invalid information, editing requests, canceling requests, and enforcing foreign keys.

PASS UC01 create and Open
PASS Lab 4 checkbox values preserved
PASS Duplicate email reuses student
PASS UC02 course and status filter
PASS Invalid reason rejected
PASS Invalid submission writes nothing
PASS Edit Open request
PASS Updated reason persisted
PASS Cancel Open request
PASS Cancelled row retained
PASS Second cancellation rejected
PASS Unrelated request unchanged
PASS FK enforced
PASS UC02 empty filter result
All integration checks passed

6. AI Skills Pack

Activity 1: Application Builder

Prompt used:

"Explain how a tutoring request moves from php/request-form.php through PHP, PDO, and SQLite in this project. Identify the responsibility of each file."

AI Response:

A student starts by filling out the form in php/request-form.php. The form sends the information to php/process-request.php, which passes it to services/requestService.php. The service validates the information, starts a database transaction, and uses the repository files to save the student and tutoring request. The database connection is handled by config/db.php, and the information is stored in the SQLite students and tutoring_requests tables.

My Evaluation:

The AI response correctly explained how the request moves through the application. I checked the actual PHP files and confirmed that the form, service, repositories, and database connection each have separate responsibilities.

7. Challenges

One challenge was making sure student records were not duplicated when the same email address was used. I used INSERT OR IGNORE and looked up the student by email before creating a tutoring request.

Another challenge was making sure the database and application worked together correctly. Running the integration tests helped me find and fix problems.

Activity 2

**Prompt used:**

"Trace the requirement 'a student can submit a tutoring request' through the PHP files, service, repositories, SQLite tables, and integration tests in this project."

**AI Response:**

A student submits a request using `php/request-form.php`. The information is sent to `php/process-request.php` and validated by `services/requestService.php`. The repository files, `studentPDO.php` and `requestPDO.php`, handle the database operations. The information is stored in the SQLite `students` and `tutoring_requests` tables. The integration tests check that requests can be created successfully and that invalid information is rejected.

**My Evaluation:**

The AI response correctly identified the main files and database tables involved in submitting a request. I compared the explanation with my project files and confirmed that the integration tests check request creation and validation.

**Requirement Traceability Table:**

| Requirement | Application Page | Service or Validation | Repository or SQL | Database Table | Test Evidence |
|---|---|---|---|---|---|
| Student can submit a tutoring request | `request-form.php`, `process-request.php` | `requestService.php` | `studentPDO.php`, `requestPDO.php` | `students`, `tutoring_requests` | PASS UC01 create and Open |
| Student information is not duplicated | `process-request.php` | `requestService.php` | `studentPDO.php` | `students` | PASS Duplicate email reuses student |
| Invalid requests are rejected | `process-request.php` | `requestService.php` | No database write | None | PASS Invalid reason rejected; PASS Invalid submission writes nothing |
| Requests can be filtered | `requests.php` | Request service and filtering logic | `requestPDO.php` | `tutoring_requests` | PASS UC02 course and status filter |
| Open requests can be edited | `edit-request.php`, `update-requests.php` | `requestService.php` | `requestPDO.php` | `tutoring_requests` | PASS Edit Open request |
| Open requests can be cancelled | `delete-request.php` | Cancellation logic | `requestPDO.php` | `tutoring_requests` | PASS Cancel Open request |

Activity 3: Test Harness Generator

**Prompt used:**

"Suggest one useful test case for the Peer Tutoring Hub application. The test must relate to request creation, filtering, editing, cancellation, validation, or database integrity."

**AI Response:**

One useful test would be to try editing a tutoring request after it has been cancelled. The application should reject the update because only Open requests should be editable.

**My Evaluation:**

This is a useful test because cancelled requests should not be changed. My integration tests already check that a second cancellation is rejected, but they do not specifically test editing a cancelled request. I could test this manually by opening a cancelled request and trying to edit it, or add another test to the integration test file.

**Skills Pack Reflection:**

Using AI helped me understand how the different PHP files work together and how to connect the assignment requirements to my application. It also helped me identify useful test cases. I checked the AI suggestions against my actual project files and integration tests to make sure the information was accurate.