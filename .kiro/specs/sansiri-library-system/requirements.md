# Requirements Document

## Introduction

The Sansiri Library System is a web-based library book rental management system. The system provides member management, book borrowing and returning with fine calculation, and reporting capabilities. The architecture consists of a separate front-end and back-end, organized in `front/` and `back/` directories respectively.

## Glossary

- **Member**: A registered user of the Sansiri Library System who can borrow and return books.
- **Book**: A physical library item that can be borrowed by a Member.
- **Librarian**: A staff user who manages Members, Books, and library operations.
- **Loan**: A record representing a Book borrowed by a Member, including borrow date, due date, and return date.
- **Fine**: A monetary penalty applied to a Member for returning a Book after the due date.
- **Catalog**: The collection of all Books available in the Sansiri Library System.
- **Member_Service**: The back-end module responsible for managing Member registration, updates, and queries.
- **Book_Service**: The back-end module responsible for managing the Book catalog.
- **Loan_Service**: The back-end module responsible for processing borrow and return transactions.
- **Fine_Service**: The back-end module responsible for calculating and managing Fines.
- **Report_Service**: The back-end module responsible for generating reports and analytics.
- **Frontend_App**: The front-end web application that provides the user interface for the Sansiri Library System.

## Requirements

### Requirement 1: Member Registration

**User Story:** As a Librarian, I want to register new Members, so that they can borrow Books from the library.

#### Acceptance Criteria

1. WHEN a Librarian submits valid Member details (name, email, phone number, address), THE Member_Service SHALL create a new Member record and return a unique member ID.
2. WHEN a Librarian submits Member details with an email that already exists, THE Member_Service SHALL reject the registration and return a duplicate email error.
3. WHEN a Librarian submits Member details with missing required fields, THE Member_Service SHALL reject the registration and return a validation error listing the missing fields.
4. THE Frontend_App SHALL provide a Member registration form with fields for name, email, phone number, and address.

### Requirement 2: Member Management

**User Story:** As a Librarian, I want to view, update, and deactivate Members, so that I can maintain accurate Member records.

#### Acceptance Criteria

1. WHEN a Librarian searches for a Member by name or member ID, THE Member_Service SHALL return all matching Member records.
2. WHEN a Librarian updates a Member's details, THE Member_Service SHALL save the changes and return the updated Member record.
3. WHEN a Librarian deactivates a Member, THE Member_Service SHALL set the Member status to inactive and prevent the Member from borrowing Books.
4. WHILE a Member has outstanding Loans, THE Member_Service SHALL prevent deactivation of that Member.
5. THE Frontend_App SHALL display a searchable list of all Members with their current status.

### Requirement 3: Book Catalog Management

**User Story:** As a Librarian, I want to manage the Book catalog, so that Members can discover and borrow available Books.

#### Acceptance Criteria

1. WHEN a Librarian adds a new Book with valid details (title, author, ISBN, category, total copies), THE Book_Service SHALL create a new Book record in the Catalog.
2. WHEN a Librarian adds a Book with an ISBN that already exists, THE Book_Service SHALL increment the total copies count for that existing Book.
3. WHEN a Librarian updates Book details, THE Book_Service SHALL save the changes and return the updated Book record.
4. WHEN a Member searches the Catalog by title, author, ISBN, or category, THE Book_Service SHALL return all matching Book records with their availability status.
5. THE Book_Service SHALL calculate availability as total copies minus currently active Loans for each Book.

### Requirement 4: Book Borrowing

**User Story:** As a Member, I want to borrow Books from the library, so that I can read them at home.

#### Acceptance Criteria

1. WHEN a Member requests to borrow an available Book, THE Loan_Service SHALL create a new Loan record with the borrow date set to the current date and the due date set to 14 days from the borrow date.
2. WHILE a Member has 5 or more active Loans, THE Loan_Service SHALL reject new borrow requests from that Member.
3. WHILE a Member has unpaid Fines, THE Loan_Service SHALL reject new borrow requests from that Member.
4. WHEN a Member requests to borrow a Book with zero available copies, THE Loan_Service SHALL reject the request and return a not-available error.
5. IF the Loan_Service fails to create a Loan record due to a system error, THEN THE Loan_Service SHALL return an error message and leave the Book availability unchanged.
6. THE Frontend_App SHALL display a borrow button for each available Book and show the due date after a successful borrow.

### Requirement 5: Book Returning

**User Story:** As a Member, I want to return borrowed Books, so that other Members can borrow them.

#### Acceptance Criteria

1. WHEN a Member returns a Book on or before the due date, THE Loan_Service SHALL mark the Loan as returned with the return date set to the current date.
2. WHEN a Member returns a Book after the due date, THE Loan_Service SHALL mark the Loan as returned and THE Fine_Service SHALL create a Fine record.
3. THE Frontend_App SHALL display a list of active Loans for the logged-in Member with a return button for each Loan.

### Requirement 6: Fine Calculation and Management

**User Story:** As a Librarian, I want the system to calculate and track Fines for overdue Books, so that Members are accountable for late returns.

#### Acceptance Criteria

1. WHEN a Book is returned after the due date, THE Fine_Service SHALL calculate the Fine as the number of overdue days multiplied by a daily fine rate of 10 baht per day.
2. WHEN a Librarian records a Fine payment, THE Fine_Service SHALL update the Fine status to paid and record the payment date.
3. THE Fine_Service SHALL track each Fine with the associated Loan, Fine amount, payment status, and payment date.
4. WHEN a Librarian queries Fines for a specific Member, THE Fine_Service SHALL return all Fine records for that Member including paid and unpaid Fines.
5. THE Frontend_App SHALL display unpaid Fines for a Member with the total outstanding amount.

### Requirement 7: Borrowing History Report

**User Story:** As a Librarian, I want to view borrowing history reports, so that I can analyze library usage patterns.

#### Acceptance Criteria

1. WHEN a Librarian requests a borrowing history report for a date range, THE Report_Service SHALL return all Loan records within that date range including Member name, Book title, borrow date, due date, return date, and Fine status.
2. WHEN a Librarian requests a borrowing history report for a specific Member, THE Report_Service SHALL return all Loan records for that Member.
3. THE Frontend_App SHALL provide date range filters and Member filters for the borrowing history report.

### Requirement 8: Popular Books Report

**User Story:** As a Librarian, I want to see which Books are most popular, so that I can make informed purchasing decisions.

#### Acceptance Criteria

1. WHEN a Librarian requests a popular Books report for a date range, THE Report_Service SHALL return a list of Books sorted by total number of Loans in descending order within that date range.
2. THE Report_Service SHALL include the Book title, author, category, and total Loan count in each entry of the popular Books report.
3. THE Frontend_App SHALL display the popular Books report as a ranked list with filtering by date range and category.

### Requirement 9: Overdue Books Report

**User Story:** As a Librarian, I want to see all currently overdue Books, so that I can follow up with Members.

#### Acceptance Criteria

1. THE Report_Service SHALL return all active Loans where the current date exceeds the due date when a Librarian requests an overdue Books report.
2. THE Report_Service SHALL include the Member name, Member contact information, Book title, borrow date, due date, and number of overdue days in each entry of the overdue Books report.
3. THE Frontend_App SHALL display the overdue Books report as a sortable table.

### Requirement 10: Fine Revenue Report

**User Story:** As a Librarian, I want to view Fine revenue reports, so that I can track library Fine income.

#### Acceptance Criteria

1. WHEN a Librarian requests a Fine revenue report for a date range, THE Report_Service SHALL return the total Fine amount, total paid amount, and total unpaid amount within that date range.
2. THE Report_Service SHALL include a breakdown of individual Fine records with Member name, Book title, Fine amount, and payment status in the Fine revenue report.
3. THE Frontend_App SHALL display the Fine revenue report with summary totals and a detailed breakdown table.

### Requirement 11: Data Serialization Round-Trip

**User Story:** As a developer, I want data serialization to be reliable, so that no data is lost during API communication between the Frontend_App and back-end services.

#### Acceptance Criteria

1. THE Member_Service SHALL serialize Member records to JSON and deserialize JSON to Member records.
2. THE Book_Service SHALL serialize Book records to JSON and deserialize JSON to Book records.
3. THE Loan_Service SHALL serialize Loan records to JSON and deserialize JSON to Loan records.
4. THE Fine_Service SHALL serialize Fine records to JSON and deserialize JSON to Fine records.
5. FOR ALL valid Member, Book, Loan, and Fine records, serializing to JSON then deserializing back SHALL produce an equivalent object (round-trip property).
