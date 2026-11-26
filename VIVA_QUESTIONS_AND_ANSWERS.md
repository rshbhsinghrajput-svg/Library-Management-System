# 🎓 Library Management System — Viva & Defense Preparation Guide
**Institution:** Babasaheb Bhimrao Ambedkar University (BBAU), Lucknow  
**Project:** Gautam Buddha Central Library Management System  
**Tech Stack:** Java 17+, JavaFX 21, MySQL 8.0 (JDBC), Maven  

---

## 📌 Section 1: Architecture & Technology Choices

### Q1. Why did you use JavaFX instead of Java Applets or Swing?
* **Ideal Answer:**
  > *"Applets were deprecated in Java 9 and permanently removed in modern Java (Java 17/21/26) because web browsers no longer support NPAPI plugins. Swing, while functional, relies on older rendering pipelines. JavaFX is the modern, official desktop GUI toolkit for Java. It offers hardware-accelerated graphics, native CSS styling (`-fx-...`), clean separation of layout and design, and reactive UI controls like `TableView` and `TabPane`."*

---

### Q2. Why did you use pure JDBC with `PreparedStatement` instead of an ORM like Hibernate or Spring Data JPA?
* **Ideal Answer:**
  > *"Using standard JDBC provides complete, low-level control over SQL queries and transactions without the heavy overhead and configuration complexity of an ORM. Furthermore, `PreparedStatement` pre-compiles SQL queries on the database server and automatically sanitizes user inputs, which eliminates 100% of SQL Injection vulnerabilities."*

---

### Q3. What design pattern does your project follow?
* **Ideal Answer:**
  > *"The project follows a **Layered MVC (Model-View-Controller)** architecture:
  > - **Model Layer:** `Book.java` and `LoanRecord.java` encapsulate data entities.
  > - **View / Presentation Layer:** `UIHeader.java`, `LoginView.java`, `StudentView.java`, and `LibrarianView.java` handle JavaFX UI rendering.
  > - **Controller / Service Layer:** `LibraryManager.java` acts as the business engine managing rules, fine calculations, and database coordination.
  > - **Data Access Layer:** `DatabaseConnection.java` manages MySQL JDBC connections and initial DDL table creation."*

---

## 📌 Section 2: Database, SQL & ACID Transactions

### Q4. What is your database schema? Name your tables and foreign key relationships.
* **Ideal Answer:**
  > *"We have 4 normalized relational tables in MySQL `library_db`:
  > 1. `librarians` (Primary Key: `librarian_id`) — stores admin credentials.
  > 2. `students` (Primary Key: `roll_no`) — stores student profiles.
  > 3. `books` (Primary Key: `book_id`) — stores book catalog and stock counts (`total_copies`, `available_copies`).
  > 4. `loans` (Primary Key: `loan_id`) — tracks borrowing history with two Foreign Keys:
  >    - `FOREIGN KEY (roll_no) REFERENCES students(roll_no)`
  >    - `FOREIGN KEY (book_id) REFERENCES books(book_id)`"*

---

### Q5. Where did you use ACID transactions in your code, and why?
* **Ideal Answer:**
  > *"We used explicit transactions during **Book Issuing** and **Book Returning** in `LibraryManager.java`:
  > - When issuing a book, two operations must happen together: inserting the loan record into `loans` AND decrementing `available_copies` in `books`.
  > - We call `connection.setAutoCommit(false)`. If both operations succeed, we call `connection.commit()`. If an exception occurs, `connection.rollback()` is executed inside a catch block to prevent data inconsistency (like creating a loan without deducting the book from the shelf)."*

---

### Q6. How do you prevent SQL Injection?
* **Ideal Answer:**
  > *"We never concatenate user inputs directly into SQL strings. Instead, we use parameterized `PreparedStatement` queries with positional placeholders (`?`):
  > ```java
  > String sql = \"SELECT * FROM books WHERE book_id = ?\";
  > PreparedStatement stmt = connection.prepareStatement(sql);
  > stmt.setString(1, bookId);
  > ```
  > The database driver treats the parameters strictly as literal data, never as executable SQL commands."*

---

## 📌 Section 3: Core Business Rules & Calculations

### Q7. How does the 30-day due date calculation and renewal policy work?
* **Ideal Answer:**
  > *"When a book is issued, the due date is set to exactly 30 calendar days from today using `LocalDate.now().plusDays(30)`.
  > - **Renewal Rule:** A student can only renew an active loan on or before the due date (`!today.isAfter(dueDate)`).
  > - If allowed, the due date is extended by +30 days from the current due date, and `renewal_count` is incremented.
  > - If today is past the due date, renewal is strictly blocked because the book is already overdue."*

---

### Q8. How is the overdue fine calculated in Java?
* **Ideal Answer:**
  > *"We use Java's `java.time.temporal.ChronoUnit.DAYS` API:
  > ```java
  > if (returnDate.isAfter(dueDate)) {
  >     long overdueDays = ChronoUnit.DAYS.between(dueDate, returnDate);
  >     fineAmount = overdueDays * 50.0; // ₹50 per day
  > } else {
  >     fineAmount = 0.0;
  > }
  > ```
  > This guarantees exact calendar day math regardless of month lengths or leap years."*

---

### Q9. Can a student borrow two copies of the same book at the same time?
* **Ideal Answer:**
  > *"No. In `LibraryManager.java`, prior to issuing, we run a check:
  > ```sql
  > SELECT COUNT(*) FROM loans WHERE roll_no = ? AND book_id = ? AND returned = FALSE
  > ```
  > If count > 0, the system blocks the issue and informs the student that an active copy is already in their possession."*

---

## 📌 Section 4: JavaFX UI & BBAU Lucknow Branding

### Q10. How did you implement the BBAU logo and the background watermark?
* **Ideal Answer:**
  > *"We implemented the BBAU logo in two places:
  > 1. **Header Banner:** In `UIHeader.java`, the official BBAU crest is placed inside a gold-bordered (`#DDA853`) rounded container beside the bilingual institutional titles.
  > 2. **Background Watermark:** In `LibraryApp.java`, the root layout is a `StackPane`. Layer 0 contains a centered `ImageView` with the BBAU logo set to `opacity = 0.09` and `mouseTransparent = true`. Layer 1 holds the interactive UI. This allows the watermark to show subtly in the background without blocking mouse events."*

---

### Q11. How does the JavaFX TableView update when a book is issued or returned?
* **Ideal Answer:**
  > *"We use JavaFX `ObservableList<Book>` and `ObservableList<LoanRecord>`. When an action completes (like check-in or check-out), `refreshAllData()` queries the database and calls `tableView.setItems(FXCollections.observableArrayList(list))`, which automatically repopulates the UI table rows."*

---

## 📌 Section 5: "Live Test" Questions (Teachers Love Asking These!)

### Q12. "Can you show me where to change the fine from ₹50 to ₹100 per day?"
* **Where to Look:**
  > Open [LibraryManager.java](file:///c:/Users/ANKITA%20KUMARI/OneDrive/Desktop/LMS/src/main/java/com/library/LibraryManager.java):
  > - In `returnBook()`: line where `overdueDays * 50.0` is computed.
  > - In `updateLiveOverdueFines()`: line where `overdueDays * 50.0` is computed.
  > - Simply change `50.0` to `100.0`.

---

### Q13. "What happens if MySQL server is stopped and someone opens the app?"
* **Ideal Answer:**
  > *"In `DatabaseConnection.java`, the connection attempt is wrapped in a `try-catch (SQLException e)` block. If MySQL is unreachable, the system displays a clear error alert informing the user that the MySQL service is stopped on port 3306 rather than crashing abruptly with an unhandled exception."*

---

### Q14. "How do you test your code without manually clicking every button?"
* **Ideal Answer:**
  > *"We have an automated test suite with **25 unit and integration tests** using JUnit 5 and an in-memory H2 database in MySQL compatibility mode. Running `mvn test` validates all business rules (30-day calculations, renewals, returns, fines, and duplicate borrow prevention) in under 4 seconds."*
