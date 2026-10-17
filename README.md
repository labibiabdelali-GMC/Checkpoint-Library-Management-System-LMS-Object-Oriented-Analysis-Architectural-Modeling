# Library Management System (LMS)

Analysis and design of a simple library system that manages **books**, **members**, and **borrowing/returning**.

## Actors
- **Member**: searches books and views their loans
- **Librarian**: issues and returns books, manages books and members
- **System Timer**: checks for overdue loans

## Main Use Cases
Search book · Issue book · Return book · Add/remove book · Register member · View my loans · Check overdue loans

## Rules
- One book can be lent to one member at a time
- A member can borrow at most **3 books**
- A loan lasts **14 days**

## Diagrams (draw.io)

| # | File | Shows |
|---|---|---|
| 1 | [01-architecture](diagrams/01-architecture.drawio) | 3 layers: UI → Business Logic → Data Access |
| 2 | [02-use-case](diagrams/02-use-case.drawio) | Actors and what they can do |
| 3 | [03-class-diagram](diagrams/03-class-diagram.drawio) | Classes, attributes, methods, relations |
| 4 | [04-sequence-issue-book](diagrams/04-sequence-issue-book.drawio) | Steps to issue a book |
| 5 | [05-sequence-return-book](diagrams/05-sequence-return-book.drawio) | Steps to return a book |
| 6 | [06-sequence-search-book](diagrams/06-sequence-search-book.drawio) | Steps to search a book |
| 7 | [07-state-book](diagrams/07-state-book.drawio) | Book states: Available → Issued → Returned |
| 8 | [08-data-model-erd](diagrams/08-data-model-erd.drawio) | **Data model**: tables and keys |
| 9 | [09-functional-model-dfd](diagrams/09-functional-model-dfd.drawio) | **Functional model**: how data flows |
| 10 | [10-activity-issue-book](diagrams/10-activity-issue-book.drawio) | **Behavioral model**: issue book decisions |

Open them at [app.diagrams.net](https://app.diagrams.net) (**File → Open from → Device**) or in VS Code with the *Draw.io Integration* extension.

## More Details
[ANALYSIS.md](ANALYSIS.md) has the full analysis, with explanations and the same diagrams written in Mermaid.
