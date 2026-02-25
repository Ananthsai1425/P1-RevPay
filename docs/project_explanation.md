# RevPay Project Walkthrough

This guide provides a detailed explanation of the **RevPay** project structure and code. Use this to prepare for your review.

## 🏗️ Project Architecture
RevPay follows a **Layered Architecture**. This means the code is organized into different "layers," each with a specific responsibility.

**Flow of Data:**
1.  **User** interacts with the **Console (Main.java)**.
2.  **Console** sends commands to the **Service Layer** (Business Logic).
3.  **Service Layer** checks rules and asks the **DAO Layer** for data.
4.  **DAO Layer** runs SQL queries on the **Database**.

---

## 📂 File-by-File Explanation

### 1. The Entry Point (Presentation Layer)
This is where the application starts and where the user interacts.

*   **`src/main/java/com/revpay/Main.java`**
    *   **Purpose**: The "Brain" of the console interface. It contains the `public static void main` method.
    *   **What it does**:
        *   Shows the "Welcome" menu (Register, Login).
        *   Handles user input using a `Scanner`.
        *   Directs users to different "Dashboards" based on their role (Personal vs. Business).
        *   Calls methods in the `Service` layer to do the actual work (like `userService.login()`).

### 2. Configuration (`com.revpay.config`)
Infrastructure settings for the application.

*   **`DatabaseConnection.java`**
    *   **Purpose**: Connects your Java code to the MySQL database.
    *   **Key Code**: uses `DriverManager.getConnection(URL, USER, PASSWORD)`.
    *   **Important**: It uses the **Singleton Pattern** (implicitly via static methods) to ensure there is a central place to get connections.

### 3. Models (`com.revpay.model`)
These are simple Java classes (POJOs) that represent your data. They don't have "logic"; they just hold variables.

*   **`User.java`**: Represents a registered user (ID, Name, Email, PasswordHash, Role).
*   **`Role.java`**: An `Enum` (Enumeration) defining the two user types: `PERSONAL` and `BUSINESS`.
*   **`Wallet.java`**: Holds the user's balance.
*   **`Transaction.java`**: Records a money transfer (Sender, Receiver, Amount, Timestamp).
*   **`TransactionType.java`**: Enum for `CREDIT` (money coming in) vs `DEBIT` (money going out).
*   **`Invoice.java`**: A bill sent by a Business user to a Personal user.
*   **`Loan.java`**: A request for money from the bank.
*   **`PaymentMethod.java`**: Represents a Credit/Debit card linked to an account.

### 4. Data Access Object (DAO) Layer (`com.revpay.dao`)
These files communicate directly with the Database. **"DAO" stands for Data Access Object.**

*   **`UserDAO.java`**
    *   **functions**: `registerUser()` (INSERT), `getUserByEmail()` (SELECT), `deleteUser()` (DELETE).
    *   **Key Concept**: uses **`PreparedStatement`** to safely execute SQL queries and prevent SQL Injection attacks.
*   **`WalletDAO.java`**
    *   Updates the user's balance in the database.
*   **`TransactionDAO.java`**
    *   Saves every transaction into the `transactions` table so users can see their history.
*   **`InvoiceDAO.java`, `LoanDAO.java`, `RequestDAO.java`**
    *   Handle database operations for their specific features.

### 5. Service Layer (`com.revpay.service`)
The "Brain" of the business logic. It sits between the User (Main) and the Database (DAO).

*   **`UserService.java`**
    *   **Login Logic**: Gets user from DAO -> Hashes the input password -> Compares it with the stored hash.
    *   **Register Logic**: Calls `UserDAO` to save the user, then immediately calls `WalletDAO` to create an empty wallet for them.
*   **`TransactionService.java`**
    *   **Transfer Logic**: Checks if the sender has enough money -> Subtracts from Sender -> Adds to Receiver -> Records the transaction.
    *   **Transactional**: Ensures that if one part fails (e.g., money deducted but not added), the whole thing is cancelled (Rollback).

### 6. Utilities (`com.revpay.util`)
Helper classes used across the project.

*   **`SecurityUtil.java`**
    *   **Purpose**: Handles passwords securely.
    *   **Key Library**: Uses **BCrypt** (`at.favre.lib.bcrypt`).
    *   **Functions**: `hashPassword()` (turns "password123" into a scrambled string) and `verifyPassword()` (checks if a password matches the hash). **Never store plain text passwords!**

### 7. Other Files
*   **`pom.xml`**: The **Maven** configuration file. It lists the "Dependencies" (external libraries) your project needs:
    *   `mysql-connector-java`: To talk to MySQL.
    *   `log4j`: For logging messages.
    *   `bcrypt`: For password security.
    *   `junit`: For testing.
*   **`log4j2.xml`**: Configuration for logging. It tells the app where to save logs and what level of detail (INFO, ERROR) to record.
*   **`database_setup.sql`**: The SQL script used to create your tables initially.

---

## 🔑 Key Concepts for your Review
If functionality questions come up, remember these points:

1.  **Security**: "We use **BCrypt** to hash passwords so even if the DB is stolen, user passwords are safe."
2.  **Safety**: "We use **PreparedStatements** in DAOs to prevent SQL Injection."
3.  **Architecture**: "The **Service Layer** ensures business rules (like checking balance) are met before modifying data in the **DAO Layer**."
4.  **Transactions**: "Money transfers use database transactions to ensure money isn't lost if the system crashes halfway through."
