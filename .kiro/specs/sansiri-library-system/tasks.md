# Tasks

## Task 1: Initialize Back-End Repository and Project Structure

Set up the `back/` directory as an independent git repository with the .NET Core Web API project scaffold.

- [ ] 1.1 Initialize `back/` as a git repository with `.gitignore` for .NET
- [ ] 1.2 Create the .NET Core Web API project (`back.csproj`, `Program.cs`, `appsettings.json`)
- [ ] 1.3 Add NuGet packages: Entity Framework Core, SQLite provider, FsCheck for testing
- [ ] 1.4 Create folder structure: `Controllers/`, `Services/`, `Models/`, `DTOs/`, `Data/`
- [ ] 1.5 Configure `Program.cs` with CORS (allow front-end origin), EF Core with SQLite, and service DI registration

## Task 2: Implement Data Models and Database Context

Create EF Core entity models and the `LibraryDbContext`.

- [ ] 2.1 Create `Models/Member.cs` with fields: Id, Name, Email, Phone, Address, IsActive, CreatedAt
- [ ] 2.2 Create `Models/Book.cs` with fields: Id, Title, Author, Isbn, Category, TotalCopies, CreatedAt
- [ ] 2.3 Create `Models/Loan.cs` with fields: Id, MemberId, BookId, BorrowDate, DueDate, ReturnDate, IsReturned
- [ ] 2.4 Create `Models/Fine.cs` with fields: Id, LoanId, MemberId, Amount, IsPaid, PaymentDate, CreatedAt
- [ ] 2.5 Create `Data/LibraryDbContext.cs` with DbSets and entity configurations (unique email, unique ISBN, foreign keys)
- [ ] 2.6 Create initial EF Core migration and apply to SQLite database

## Task 3: Implement DTOs

Create Data Transfer Objects for API request/response payloads.

- [ ] 3.1 Create `DTOs/MemberDto.cs` (Name, Email, Phone, Address)
- [ ] 3.2 Create `DTOs/BookDto.cs` (Title, Author, Isbn, Category, TotalCopies)
- [ ] 3.3 Create `DTOs/LoanDto.cs` (MemberId, BookId)
- [ ] 3.4 Create `DTOs/FineDto.cs`
- [ ] 3.5 Create `DTOs/ReportDtos.cs` (BorrowingHistoryEntry, PopularBookEntry, OverdueEntry, FineRevenueReport)

## Task 4: Implement Member Service and Controller

Build member registration, search, update, and deactivation.

- [ ] 4.1 Create `Services/IMemberService.cs` interface
- [ ] 4.2 Create `Services/MemberService.cs` with registration (email uniqueness, required field validation), search, update, and deactivation (guard against active loans)
- [ ] 4.3 Create `Controllers/MembersController.cs` with endpoints: POST /api/members, GET /api/members, GET /api/members/{id}, PUT /api/members/{id}, PATCH /api/members/{id}/deactivate
- [ ] 4.4 Write unit tests for MemberService (registration, duplicate email, missing fields, deactivation guard)
- [ ] 4.5 Write property tests: Property 6 (email uniqueness), Property 13 (validation rejects missing fields), Property 14 (update round-trip for members)

## Task 5: Implement Book Service and Controller

Build book catalog management with ISBN duplicate handling and availability calculation.

- [ ] 5.1 Create `Services/IBookService.cs` interface
- [ ] 5.2 Create `Services/BookService.cs` with add (ISBN duplicate → increment copies), update, search with availability calculation
- [ ] 5.3 Create `Controllers/BooksController.cs` with endpoints: POST /api/books, GET /api/books, GET /api/books/{id}, PUT /api/books/{id}
- [ ] 5.4 Write unit tests for BookService (add, ISBN duplicate, search, availability)
- [ ] 5.5 Write property tests: Property 1 (book availability invariant), Property 7 (ISBN duplicate handling), Property 14 (update round-trip for books)

## Task 6: Implement Loan Service and Controller

Build borrowing and returning with business rule enforcement.

- [ ] 6.1 Create `Services/ILoanService.cs` interface
- [ ] 6.2 Create `Services/LoanService.cs` with borrow (due date = +14 days, 5-loan limit, unpaid fine check, availability check, deactivated member check) and return (mark returned, trigger fine creation if overdue)
- [ ] 6.3 Create `Controllers/LoansController.cs` with endpoints: POST /api/loans, POST /api/loans/{id}/return, GET /api/loans/member/{memberId}
- [ ] 6.4 Write unit tests for LoanService (borrow success, limit exceeded, unpaid fines, zero availability, return on-time, return overdue)
- [ ] 6.5 Write property tests: Property 2 (due date invariant), Property 4 (borrowing eligibility), Property 5 (deactivation guard), Property 8 (overdue return creates fine), Property 9 (on-time return no fine)

## Task 7: Implement Fine Service and Controller

Build fine calculation, payment, and querying.

- [ ] 7.1 Create `Services/IFineService.cs` interface
- [ ] 7.2 Create `Services/FineService.cs` with fine creation (overdue days × 10 baht), payment recording, and member fine querying
- [ ] 7.3 Create `Controllers/FinesController.cs` with endpoints: GET /api/fines/member/{memberId}, POST /api/fines/{id}/pay
- [ ] 7.4 Write unit tests for FineService (fine calculation, payment, query)
- [ ] 7.5 Write property tests: Property 3 (fine calculation correctness)

## Task 8: Implement Report Service and Controller

Build all four reports: borrowing history, popular books, overdue, fine revenue.

- [ ] 8.1 Create `Services/IReportService.cs` interface
- [ ] 8.2 Create `Services/ReportService.cs` with borrowing history (date range + member filter), popular books (sorted by loan count desc), overdue books, and fine revenue (totals + breakdown)
- [ ] 8.3 Create `Controllers/ReportsController.cs` with endpoints: GET /api/reports/borrowing-history, GET /api/reports/popular-books, GET /api/reports/overdue, GET /api/reports/fine-revenue
- [ ] 8.4 Write unit tests for ReportService (each report with sample data)
- [ ] 8.5 Write property tests: Property 11 (popular books ordering), Property 12 (fine revenue totals invariant), Property 15 (borrowing history correctness), Property 16 (overdue report correctness)

## Task 9: Implement JSON Serialization Round-Trip Tests

- [ ] 9.1 Write property tests: Property 10 (JSON serialization round-trip for Member, Book, Loan, Fine) using FsCheck with minimum 100 iterations

## Task 10: Initialize Front-End Repository and Project Structure

Set up the `front/` directory as an independent git repository with Vue 3 + Vite + TypeScript.

- [ ] 10.1 Initialize `front/` as a git repository with `.gitignore` for Node.js
- [ ] 10.2 Scaffold Vue 3 project with Vite and TypeScript (`package.json`, `vite.config.ts`, `index.html`, `src/main.ts`, `src/App.vue`)
- [ ] 10.3 Install dependencies: vue-router, pinia, axios, vitest, fast-check, @vue/test-utils
- [ ] 10.4 Create folder structure: `src/router/`, `src/stores/`, `src/services/`, `src/views/`, `src/components/`
- [ ] 10.5 Configure `vite.config.ts` with API proxy to back-end during development

## Task 11: Implement Front-End API Service and Stores

Create the Axios API client and Pinia stores.

- [ ] 11.1 Create `src/services/api.ts` with Axios instance and API functions for members, books, loans, fines, and reports
- [ ] 11.2 Create `src/stores/memberStore.ts` with actions: register, search, update, deactivate
- [ ] 11.3 Create `src/stores/bookStore.ts` with actions: add, search, update
- [ ] 11.4 Create `src/stores/loanStore.ts` with actions: borrow, return, getActiveLoans
- [ ] 11.5 Create `src/stores/fineStore.ts` with actions: getMemberFines, payFine

## Task 12: Implement Front-End Router and Views

Create Vue Router configuration and all page views.

- [ ] 12.1 Create `src/router/index.ts` with routes for all views
- [ ] 12.2 Create `src/views/MemberRegistration.vue` with registration form (name, email, phone, address)
- [ ] 12.3 Create `src/views/MemberList.vue` with searchable member list, edit, and deactivate actions
- [ ] 12.4 Create `src/views/BookCatalog.vue` with search by title/author/ISBN/category and availability display
- [ ] 12.5 Create `src/views/BookManagement.vue` with add/edit book forms
- [ ] 12.6 Create `src/views/MyLoans.vue` with active loans list and return buttons
- [ ] 12.7 Create report views: `BorrowingHistoryReport.vue`, `PopularBooksReport.vue`, `OverdueBooksReport.vue`, `FineRevenueReport.vue`

## Task 13: Implement Front-End Shared Components

Create reusable components used across views.

- [ ] 13.1 Create `src/components/MemberForm.vue` (reusable form for registration and editing)
- [ ] 13.2 Create `src/components/BookForm.vue` (reusable form for adding and editing books)
- [ ] 13.3 Create `src/components/LoanCard.vue` (displays loan info with return button)
- [ ] 13.4 Create `src/components/FineList.vue` (displays fines with pay button)
- [ ] 13.5 Create `src/components/ReportFilters.vue` (date range and category/member filters)

## Task 14: Front-End Testing

Write unit and property tests for the front-end.

- [ ] 14.1 Write unit tests for Pinia stores (memberStore, bookStore, loanStore, fineStore)
- [ ] 14.2 Write unit tests for API service functions (mock Axios)
- [ ] 14.3 Write property tests using fast-check: Property 13 (client-side validation rejects missing/whitespace fields) with minimum 100 iterations
- [ ] 14.4 Write component tests for MemberForm, BookForm (form submission, validation display)
