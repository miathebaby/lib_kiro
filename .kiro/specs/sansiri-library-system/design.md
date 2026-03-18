# Design Document

## Overview

The Sansiri Library System is a full-stack web application split into two independent git repositories:

- **`front/`** — Vue.js front-end (its own git repository)
- **`back/`** — .NET Core back-end API (its own git repository)

The back-end exposes RESTful API endpoints for member management, book catalog management, loan processing, fine management, and reporting. The front-end consumes these APIs and provides the Librarian and Member user interfaces. Each repository is developed, versioned, and deployed independently.

## Architecture

### High-Level Architecture

The system is composed of two independent git repositories that communicate over HTTP/JSON:

```
┌──────────────────────────────┐       HTTP/JSON        ┌──────────────────────────────────┐
│  front/ (separate git repo)  │ ◄────────────────────► │   back/ (separate git repo)      │
│  Vue.js Frontend             │                         │   .NET Core Backend API          │
│                              │                         │                                  │
│  - Vue Router                │                         │  - Controllers                   │
│  - Pinia Store               │                         │  - Services                      │
│  - Axios HTTP Client         │                         │  - Entity Framework Core         │
│  - Vue Components            │                         │  - SQLite Database               │
└──────────────────────────────┘                         └──────────────────────────────────┘
```

> **Repository Separation Note:** Each directory (`front/` and `back/`) is an independent git repository with its own `.git`, version history, CI/CD pipeline, and dependency management. They share no common parent repository. The API contract (endpoints, DTOs) is the integration boundary between the two repos.

### Back-End Architecture (.NET Core)

The back-end follows a layered architecture:

```
Controllers (API Layer)
    ↓
Services (Business Logic Layer)
    ↓
DbContext / Repositories (Data Access Layer)
    ↓
SQLite Database
```

#### Project Structure (back/ — independent git repo)

```
back/
├── .git/
├── .gitignore
├── Program.cs
├── appsettings.json
├── back.csproj
├── Controllers/
│   ├── MembersController.cs
│   ├── BooksController.cs
│   ├── LoansController.cs
│   ├── FinesController.cs
│   └── ReportsController.cs
├── Services/
│   ├── IMemberService.cs
│   ├── MemberService.cs
│   ├── IBookService.cs
│   ├── BookService.cs
│   ├── ILoanService.cs
│   ├── LoanService.cs
│   ├── IFineService.cs
│   ├── FineService.cs
│   ├── IReportService.cs
│   └── ReportService.cs
├── Models/
│   ├── Member.cs
│   ├── Book.cs
│   ├── Loan.cs
│   └── Fine.cs
├── DTOs/
│   ├── MemberDto.cs
│   ├── BookDto.cs
│   ├── LoanDto.cs
│   ├── FineDto.cs
│   └── ReportDtos.cs
├── Data/
│   └── LibraryDbContext.cs
└── Migrations/
```

### Front-End Architecture (Vue.js)

The front-end uses Vue 3 with Composition API, Vue Router, and Pinia for state management.

#### Project Structure (front/ — independent git repo)

```
front/
├── .git/
├── .gitignore
├── package.json
├── vite.config.ts
├── index.html
├── src/
│   ├── main.ts
│   ├── App.vue
│   ├── router/
│   │   └── index.ts
│   ├── stores/
│   │   ├── memberStore.ts
│   │   ├── bookStore.ts
│   │   ├── loanStore.ts
│   │   └── fineStore.ts
│   ├── services/
│   │   └── api.ts
│   ├── views/
│   │   ├── MemberRegistration.vue
│   │   ├── MemberList.vue
│   │   ├── BookCatalog.vue
│   │   ├── BookManagement.vue
│   │   ├── MyLoans.vue
│   │   ├── BorrowingHistoryReport.vue
│   │   ├── PopularBooksReport.vue
│   │   ├── OverdueBooksReport.vue
│   │   └── FineRevenueReport.vue
│   └── components/
│       ├── MemberForm.vue
│       ├── BookForm.vue
│       ├── LoanCard.vue
│       ├── FineList.vue
│       └── ReportFilters.vue
```

## Data Models

### Member

| Field     | Type     | Constraints                    |
|-----------|----------|--------------------------------|
| Id        | int      | Primary key, auto-increment    |
| Name      | string   | Required                       |
| Email     | string   | Required, unique               |
| Phone     | string   | Required                       |
| Address   | string   | Required                       |
| IsActive  | bool     | Default: true                  |
| CreatedAt | DateTime | Set on creation                |

### Book

| Field       | Type   | Constraints                 |
|-------------|--------|-----------------------------|
| Id          | int    | Primary key, auto-increment |
| Title       | string | Required                    |
| Author      | string | Required                    |
| Isbn        | string | Required, unique            |
| Category    | string | Required                    |
| TotalCopies | int    | Required, >= 1              |
| CreatedAt   | DateTime | Set on creation           |

### Loan

| Field      | Type      | Constraints                          |
|------------|-----------|--------------------------------------|
| Id         | int       | Primary key, auto-increment          |
| MemberId   | int       | Foreign key → Member                 |
| BookId     | int       | Foreign key → Book                   |
| BorrowDate | DateTime  | Set on creation                      |
| DueDate    | DateTime  | BorrowDate + 14 days                 |
| ReturnDate | DateTime? | Null until returned                  |
| IsReturned | bool      | Default: false                       |

### Fine

| Field       | Type      | Constraints                          |
|-------------|-----------|--------------------------------------|
| Id          | int       | Primary key, auto-increment          |
| LoanId      | int       | Foreign key → Loan                   |
| MemberId    | int       | Foreign key → Member                 |
| Amount      | decimal   | OverdueDays × 10                     |
| IsPaid      | bool      | Default: false                       |
| PaymentDate | DateTime? | Null until paid                      |
| CreatedAt   | DateTime  | Set on creation                      |

## API Endpoints

### Members API

| Method | Endpoint              | Description                  | Request Body     | Response         |
|--------|-----------------------|------------------------------|------------------|------------------|
| POST   | /api/members          | Register a new member        | MemberDto        | Member (with Id) |
| GET    | /api/members          | List/search members          | ?search=query    | Member[]         |
| GET    | /api/members/{id}     | Get member by ID             | -                | Member           |
| PUT    | /api/members/{id}     | Update member details        | MemberDto        | Member           |
| PATCH  | /api/members/{id}/deactivate | Deactivate a member   | -                | Member           |

### Books API

| Method | Endpoint              | Description                  | Request Body     | Response         |
|--------|-----------------------|------------------------------|------------------|------------------|
| POST   | /api/books            | Add a book                   | BookDto          | Book             |
| GET    | /api/books            | Search catalog               | ?search=&category= | BookWithAvailability[] |
| GET    | /api/books/{id}       | Get book by ID               | -                | BookWithAvailability |
| PUT    | /api/books/{id}       | Update book details          | BookDto          | Book             |

### Loans API

| Method | Endpoint              | Description                  | Request Body     | Response         |
|--------|-----------------------|------------------------------|------------------|------------------|
| POST   | /api/loans            | Borrow a book                | { memberId, bookId } | Loan          |
| POST   | /api/loans/{id}/return | Return a book               | -                | Loan (+ Fine if overdue) |
| GET    | /api/loans/member/{memberId} | Get active loans for member | -          | Loan[]           |

### Fines API

| Method | Endpoint              | Description                  | Request Body     | Response         |
|--------|-----------------------|------------------------------|------------------|------------------|
| GET    | /api/fines/member/{memberId} | Get fines for a member | -              | Fine[]           |
| POST   | /api/fines/{id}/pay   | Record fine payment          | -                | Fine             |

### Reports API

| Method | Endpoint                     | Description              | Query Params              | Response              |
|--------|------------------------------|--------------------------|---------------------------|-----------------------|
| GET    | /api/reports/borrowing-history | Borrowing history report | ?from=&to=&memberId=    | BorrowingHistoryEntry[] |
| GET    | /api/reports/popular-books   | Popular books report     | ?from=&to=&category=      | PopularBookEntry[]    |
| GET    | /api/reports/overdue         | Overdue books report     | -                         | OverdueEntry[]        |
| GET    | /api/reports/fine-revenue    | Fine revenue report      | ?from=&to=                | FineRevenueReport     |

## Key Business Logic

### Book Availability Calculation
- `AvailableCopies = Book.TotalCopies - COUNT(Loans WHERE BookId = Book.Id AND IsReturned = false)`
- A Book is available for borrowing when `AvailableCopies > 0`

### Borrowing Rules
- Maximum 5 active (unreturned) Loans per Member
- Members with unpaid Fines cannot borrow
- Only active Members can borrow

### Fine Calculation
- `FineAmount = OverdueDays × 10` (baht)
- `OverdueDays = (ReturnDate - DueDate).Days`
- Fine is created automatically when a Book is returned after the DueDate

### Member Deactivation Rules
- A Member cannot be deactivated while they have active (unreturned) Loans
- Deactivated Members cannot borrow Books

### ISBN Duplicate Handling
- When adding a Book with an existing ISBN, the TotalCopies of the existing Book is incremented instead of creating a new record

## Correctness Properties

*A property is a characteristic or behavior that should hold true across all valid executions of a system — essentially, a formal statement about what the system should do. Properties serve as the bridge between human-readable specifications and machine-verifiable correctness guarantees.*

### Property 1: Book Availability Invariant

*For any* Book in the system, after any borrow or return operation, `AvailableCopies` must equal `TotalCopies - COUNT(active loans for that book)` and `AvailableCopies >= 0`. A borrow request for a Book with zero available copies must be rejected.

**Validates: Requirements 3.5, 4.4**

### Property 2: Loan Due Date Invariant

*For any* newly created Loan, `DueDate` must equal `BorrowDate + 14 days` exactly.

**Validates: Requirements 4.1**

### Property 3: Fine Calculation Correctness

*For any* Loan returned after its due date, the Fine amount must equal `(ReturnDate - DueDate).Days × 10` baht.

**Validates: Requirements 6.1**

### Property 4: Borrowing Eligibility Enforcement

*For any* Member, a new borrow request must be rejected if the Member has 5 or more active Loans, OR has any unpaid Fines, OR is deactivated.

**Validates: Requirements 4.2, 4.3, 2.3**

### Property 5: Member Deactivation Guard

*For any* Member with one or more active (unreturned) Loans, a deactivation request must be rejected and the Member must remain active.

**Validates: Requirements 2.4**

### Property 6: Email Uniqueness

*For any* two Member registration attempts with the same email address, the second attempt must be rejected with a duplicate email error.

**Validates: Requirements 1.2**

### Property 7: ISBN Duplicate Handling

*For any* Book addition where the ISBN already exists in the catalog, the system must increment the existing Book's `TotalCopies` rather than creating a new record. The total number of Book records with that ISBN must remain exactly one.

**Validates: Requirements 3.2**

### Property 8: Overdue Return Creates Fine Atomically

*For any* Book returned after the due date, the Loan must be marked as returned AND a corresponding Fine record must be created in the same operation.

**Validates: Requirements 5.2**

### Property 9: On-Time Return Produces No Fine

*For any* Book returned on or before the due date, the Loan must be marked as returned and no Fine record must be created.

**Validates: Requirements 5.1**

### Property 10: JSON Serialization Round-Trip

*For any* valid Member, Book, Loan, or Fine object, serializing to JSON and then deserializing back must produce an equivalent object.

**Validates: Requirements 11.1, 11.2, 11.3, 11.4, 11.5**

### Property 11: Popular Books Report Ordering

*For any* date range and set of loan data, the popular books report must return entries sorted by total loan count in descending order, and each entry must include book title, author, category, and loan count.

**Validates: Requirements 8.1, 8.2**

### Property 12: Fine Revenue Report Totals Invariant

*For any* date range and set of fine data, the fine revenue report must satisfy `TotalFineAmount = TotalPaidAmount + TotalUnpaidAmount`, and each fine entry must include member name, book title, fine amount, and payment status.

**Validates: Requirements 10.1, 10.2**

### Property 13: Validation Rejects Missing Required Fields

*For any* Member registration request with one or more missing required fields (name, email, phone, address), the system must reject the request and return a validation error listing the missing fields.

**Validates: Requirements 1.3**

### Property 14: Update Round-Trip

*For any* valid Member or Book update, the returned record must reflect exactly the submitted changes, and a subsequent GET request must return the same updated data.

**Validates: Requirements 2.2, 3.3**

### Property 15: Borrowing History Report Correctness

*For any* date range and optional member filter, the borrowing history report must return exactly the Loan records that fall within that date range (and match the member filter if specified), each including member name, book title, borrow date, due date, return date, and fine status.

**Validates: Requirements 7.1, 7.2**

### Property 16: Overdue Report Correctness

*For any* set of active Loans, the overdue books report must return exactly those Loans where the current date exceeds the due date, each including member name, contact info, book title, borrow date, due date, and overdue days count.

**Validates: Requirements 9.1, 9.2**

## Error Handling

### Back-End Error Handling

- All API endpoints return appropriate HTTP status codes:
  - `200 OK` for successful operations
  - `201 Created` for successful resource creation
  - `400 Bad Request` for validation errors (missing fields, invalid data)
  - `404 Not Found` for non-existent resources
  - `409 Conflict` for business rule violations (duplicate email, borrowing limit, unpaid fines, deactivation with active loans)
  - `500 Internal Server Error` for unexpected failures
- Validation errors return a structured JSON response with field-level error messages
- Business rule violations return a descriptive error message explaining why the operation was rejected
- Database errors are caught and logged; the client receives a generic error without internal details
- All write operations (borrow, return, fine creation) are wrapped in database transactions to ensure atomicity

### Front-End Error Handling

- Axios interceptors handle global HTTP error responses
- Form validation occurs client-side before submission (required fields, email format)
- API error responses are displayed as user-friendly messages in the UI
- Network errors show a retry prompt
- Loading states are shown during API calls to prevent duplicate submissions

## Testing Strategy

### Dual Testing Approach

The system uses both unit tests and property-based tests for comprehensive coverage:

- **Unit tests**: Verify specific examples, edge cases, integration points, and error conditions
- **Property tests**: Verify universal properties across randomly generated inputs

### Back-End Testing (.NET Core — `back/` repo)

- **Framework**: xUnit + FsCheck (property-based testing library for .NET)
- **Unit tests**: Test each service method with specific inputs, edge cases, and error scenarios
- **Property tests**: Implement each correctness property (Properties 1–16) as a single property-based test with minimum 100 iterations
- **Integration tests**: Test API endpoints end-to-end using `WebApplicationFactory<Program>` with an in-memory SQLite database
- **Tag format**: Each property test must include a comment: `// Feature: sansiri-library-system, Property {number}: {property_text}`

### Front-End Testing (Vue.js — `front/` repo)

- **Framework**: Vitest + fast-check (property-based testing library for TypeScript/JavaScript)
- **Unit tests**: Test Pinia store actions and getters, API service functions, and component rendering
- **Property tests**: Test client-side validation logic (e.g., Property 13 for missing fields) and serialization (Property 10) with minimum 100 iterations
- **Component tests**: Use Vue Test Utils for component behavior verification
- **Tag format**: Each property test must include a comment: `// Feature: sansiri-library-system, Property {number}: {property_text}`

### Test Organization

Since `front/` and `back/` are independent git repositories, each repo manages its own test suite independently:

- `back/` tests are run via `dotnet test`
- `front/` tests are run via `npx vitest --run`
- No shared test infrastructure between repos; the API contract is the integration boundary
