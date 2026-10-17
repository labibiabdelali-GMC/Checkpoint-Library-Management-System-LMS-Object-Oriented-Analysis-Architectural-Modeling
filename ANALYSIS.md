# Library Management System (LMS): Analysis & Modeling

A simplified library system for managing **books**, **members**, **borrowing/returning**, and **admin tasks**.
This document covers the analysis and design of the system.

> Diagrams are written in **Mermaid**. They render automatically on GitHub and in VS Code (with a Mermaid preview extension).

---

## 1. Requirement Analysis

### Actors

| Actor | Who they are | What they do |
|---|---|---|
| **Member** | A registered library user | Searches and borrows/returns books |
| **Librarian** | Library staff (admin) | Manages books and members, issues and returns books |
| **System (timer)** | Automatic | Marks loans as overdue |

### Key Use Cases

| # | Use case | Actor | Short description |
|---|---|---|---|
| UC1 | Search book | Member, Librarian | Find books by title or author |
| UC2 | Issue book | Librarian | Lend an available book to a member |
| UC3 | Return book | Librarian | Take a book back and close the loan |
| UC4 | Add / remove book | Librarian | Keep the catalog up to date |
| UC5 | Register member | Librarian | Create a new member account |
| UC6 | View my loans | Member | See borrowed books and due dates |
| UC7 | Check overdue loans | System | Flag loans past their due date |

### Business Rules
- A book can be lent to **only one member at a time**.
- A member can borrow **at most 3 books**.
- A loan lasts **14 days**.
- A book that is currently issued **cannot be removed**.

---

## 2. System Architecture

The system uses a classic **3-layer architecture**. Each layer talks only to the layer directly below it.

| Layer | Responsibility | Examples |
|---|---|---|
| **Presentation (UI)** | Shows screens and collects user input | Member portal, Librarian dashboard |
| **Business Logic** | Applies the library rules | `CatalogService`, `LoanService`, `MemberService` |
| **Data Access** | Saves and loads data | `BookRepository`, `MemberRepository`, `LoanRepository` |
| **Database** | Stores the data | Books, Members, Loans tables |

```mermaid
flowchart TB
    subgraph UI["Presentation Layer"]
        MP["Member Portal"]
        LD["Librarian Dashboard"]
    end

    subgraph BL["Business Logic Layer"]
        CS["Catalog Service<br/>(search, add, remove books)"]
        LS["Loan Service<br/>(issue, return, overdue)"]
        MS["Member Service<br/>(register, view loans)"]
    end

    subgraph DA["Data Access Layer"]
        BR["Book Repository"]
        LR["Loan Repository"]
        MR["Member Repository"]
    end

    DB[("Database")]

    MP --> CS
    MP --> MS
    LD --> CS
    LD --> LS
    LD --> MS

    CS --> BR
    LS --> BR
    LS --> LR
    LS --> MR
    MS --> MR

    BR --> DB
    LR --> DB
    MR --> DB
```

**Why layers?** The UI can change (web, mobile) without touching the rules, and the database can change without touching the UI.

---

## 3. Object-Oriented Analysis

### 3.1 Use Case Diagram

```mermaid
flowchart LR
    Member(["👤 Member"])
    Librarian(["👤 Librarian"])
    Timer(["⏰ System Timer"])

    subgraph LMS["Library Management System"]
        UC1(("Search book"))
        UC6(("View my loans"))
        UC2(("Issue book"))
        UC3(("Return book"))
        UC4(("Add / remove book"))
        UC5(("Register member"))
        UC7(("Check overdue loans"))
    end

    Member --- UC1
    Member --- UC6
    Librarian --- UC1
    Librarian --- UC2
    Librarian --- UC3
    Librarian --- UC4
    Librarian --- UC5
    Timer --- UC7
```

### 3.2 Classes (from the use cases)

| Class | Attributes | Methods | Responsibility |
|---|---|---|---|
| **Book** | isbn, title, author, status | `is_available()`, `issue()`, `give_back()` | Knows its own state |
| **Member** | member_id, name, loans | `can_borrow()`, `add_loan()`, `remove_loan()` | Knows its borrowing limit |
| **Librarian** | staff_id, name | uses `Library` | Performs admin actions |
| **Loan** | book, member, issue_date, due_date, return_date | `close()`, `is_overdue()` | Records one borrowing |
| **Library** | books, members, loans | `search()`, `issue_book()`, `return_book()`, `add_book()`, `remove_book()`, `register_member()` | Applies the rules and coordinates the objects |

**Design principles applied**
- **Single responsibility:** `Book` manages its status, `Loan` manages dates, `Library` manages the rules.
- **Encapsulation:** `status` and `loans` are private. They change only through methods like `issue()` and `add_loan()`, never directly.

### 3.3 Class Diagram

```mermaid
classDiagram
    class Person {
        <<abstract>>
        -id : String
        -name : String
    }

    class Member {
        -loans : List~Loan~
        +can_borrow() bool
        +add_loan(loan)
        +remove_loan(loan)
    }

    class Librarian {
        -staff_id : String
    }

    class Book {
        -isbn : String
        -title : String
        -author : String
        -status : BookStatus
        +is_available() bool
        +issue()
        +give_back()
    }

    class Loan {
        -issue_date : Date
        -due_date : Date
        -return_date : Date
        +close()
        +is_overdue() bool
    }

    class Library {
        -books : Map~String, Book~
        -members : Map~String, Member~
        -loans : List~Loan~
        +search(keyword) List~Book~
        +issue_book(isbn, member_id) Loan
        +return_book(isbn, member_id) Loan
        +add_book(book)
        +remove_book(isbn)
        +register_member(member)
    }

    class BookStatus {
        <<enumeration>>
        AVAILABLE
        ISSUED
    }

    Person <|-- Member
    Person <|-- Librarian
    Library "1" o-- "*" Book
    Library "1" o-- "*" Member
    Library "1" *-- "*" Loan
    Loan "*" --> "1" Book
    Loan "*" --> "1" Member
    Librarian ..> Library : uses
    Book --> BookStatus
```

**Reading the diagram**
- A `Member` and a `Librarian` are both a kind of `Person` (inheritance).
- The `Library` holds many books, members and loans.
- Each `Loan` links exactly **one** book to **one** member.

### 3.4 Sequence Diagram: Issue Book (UC2)

```mermaid
sequenceDiagram
    actor L as Librarian
    participant UI as Librarian Dashboard
    participant Lib as Library
    participant B as Book
    participant M as Member
    participant Ln as Loan

    L->>UI: Enter ISBN + member ID
    UI->>Lib: issue_book(isbn, member_id)
    Lib->>B: is_available()
    B-->>Lib: true
    Lib->>M: can_borrow()
    M-->>Lib: true
    Lib->>B: issue()
    Note right of B: status = ISSUED
    Lib->>Ln: create Loan(book, member)
    Note right of Ln: due_date = today + 14 days
    Lib->>M: add_loan(loan)
    Lib-->>UI: loan
    UI-->>L: "Book issued, due on ..."

    alt Book not available OR limit reached
        Lib-->>UI: error
        UI-->>L: Show error message
    end
```

### 3.5 Sequence Diagram: Return Book (UC3)

```mermaid
sequenceDiagram
    actor L as Librarian
    participant UI as Librarian Dashboard
    participant Lib as Library
    participant Ln as Loan
    participant B as Book
    participant M as Member

    L->>UI: Scan returned book + member ID
    UI->>Lib: return_book(isbn, member_id)
    Lib->>Lib: find open loan
    Lib->>Ln: close()
    Note right of Ln: return_date = today
    Lib->>B: give_back()
    Note right of B: status = AVAILABLE
    Lib->>M: remove_loan(loan)
    Lib-->>UI: loan
    UI-->>L: "Book returned"

    alt No open loan found
        Lib-->>UI: error
        UI-->>L: Show error message
    end
```

### 3.6 Sequence Diagram: Search Book (UC1)

```mermaid
sequenceDiagram
    actor M as Member
    participant UI as Member Portal
    participant Lib as Library

    M->>UI: Type "clean code"
    UI->>Lib: search("clean code")
    Lib->>Lib: match title or author
    Lib-->>UI: list of books
    UI-->>M: Show results with status
```

### 3.7 State Diagram: Book

```mermaid
stateDiagram-v2
    [*] --> Available : added to catalog

    Available --> Issued : issue()
    Issued --> Overdue : due date passed
    Issued --> Returned : give_back()
    Overdue --> Returned : give_back() (late)
    Returned --> Available : back on the shelf

    Available --> Removed : remove_book()
    Removed --> [*]
```

| State | Meaning |
|---|---|
| **Available** | On the shelf, can be borrowed |
| **Issued** | Lent to a member |
| **Overdue** | Still issued after the due date |
| **Returned** | Brought back, checked by the librarian |
| **Removed** | Taken out of the catalog (only possible when Available) |

> The `Book` class stores only `AVAILABLE` / `ISSUED`. **Overdue** is known from the loan's due date, and **Returned** is the short step before going back to `AVAILABLE`.

---

## 4. Data, Functional and Behavioral Models

### 4.1 Data Model: how data is structured (ERD)

```mermaid
erDiagram
    MEMBER ||--o{ LOAN : "borrows"
    BOOK   ||--o{ LOAN : "is borrowed in"

    MEMBER {
        string member_id PK
        string name
        string email
    }

    BOOK {
        string isbn PK
        string title
        string author
        string status
    }

    LOAN {
        int    loan_id PK
        string isbn FK
        string member_id FK
        date   issue_date
        date   due_date
        date   return_date "null while open"
    }

    LIBRARIAN {
        string staff_id PK
        string name
    }
```

- One member can have **many** loans; one book can appear in **many** loans over time.
- A loan is **open** while `return_date` is empty.

### 4.2 Functional Model: what the system does (Data Flow Diagram)

```mermaid
flowchart LR
    Member(["Member"])
    Librarian(["Librarian"])

    P1["1. Search catalog"]
    P2["2. Issue book"]
    P3["3. Return book"]
    P4["4. Manage catalog & members"]

    D1[("Books")]
    D2[("Members")]
    D3[("Loans")]

    Member -- keyword --> P1
    P1 -- results --> Member
    D1 --> P1

    Librarian -- isbn, member_id --> P2
    D1 <--> P2
    D2 --> P2
    P2 -- new loan --> D3

    Librarian -- isbn, member_id --> P3
    D3 <--> P3
    P3 -- status = AVAILABLE --> D1

    Librarian -- book / member info --> P4
    P4 --> D1
    P4 --> D2
```

| Function | Input | Output |
|---|---|---|
| Search catalog | keyword | list of matching books |
| Issue book | isbn, member_id | new loan (or error) |
| Return book | isbn, member_id | closed loan, book available again |
| Manage catalog | book / member data | updated catalog / member list |

### 4.3 Behavioral Model: how the system behaves over time (Activity Diagram: Issue Book)

```mermaid
flowchart TD
    A([Start]) --> B[Librarian enters ISBN + member ID]
    B --> C{Book available?}
    C -- No --> X[Show 'Book not available'] --> Z([End])
    C -- Yes --> D{Member under 3 books?}
    D -- No --> Y[Show 'Limit reached'] --> Z
    D -- Yes --> E[Set book status = ISSUED]
    E --> F[Create loan, due in 14 days]
    F --> G[Add loan to member]
    G --> H[Show due date]
    H --> Z
```

The **state diagram** (section 3.7) and the **sequence diagrams** (sections 3.4–3.6) are also part of the behavioral model.

---

## 5. Summary

| Requirement | Where |
|---|---|
| Actors & use cases | Section 1 |
| Architecture + diagram | Section 2 |
| Classes + class diagram | Sections 3.2, 3.3 |
| Use case diagram | Section 3.1 |
| Sequence diagrams (3 use cases) | Sections 3.4, 3.5, 3.6 |
| Book state diagram | Section 3.7 |
| Data / functional / behavioral models | Section 4 |
