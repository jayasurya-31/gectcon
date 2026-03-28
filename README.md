# GECT Connect — Team 1: Authentication Module
### Integration README for All Teams
**Version:** 1.1.0 | **Maintained by:** Team 1 — User Registration & Authentication
**Project:** GECT Connect — Campus Messaging Application, GEC Thrissur
**Primary Dev Environment:** Windows 10/11 + VS Code

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [What Team 1 Owns](#2-what-team-1-owns)
3. [Technology Stack](#3-technology-stack)
4. [Environment Setup — Windows + VS Code](#4-environment-setup--windows--vs-code)
5. [VS Code Configuration](#5-vs-code-configuration)
6. [Database Setup — Run This First](#6-database-setup--run-this-first)
7. [Database Schema Reference](#7-database-schema-reference)
8. [Shared Java Components](#8-shared-java-components)
9. [Package & Naming Conventions](#9-package--naming-conventions)
10. [How to Get the Logged-In User](#10-how-to-get-the-logged-in-user)
11. [How to Navigate From the Dashboard](#11-how-to-navigate-from-the-dashboard)
12. [How to Build a Screen (Template)](#12-how-to-build-a-screen-template)
13. [Integration Checklist Per Team](#13-integration-checklist-per-team)
14. [Common Mistakes to Avoid](#14-common-mistakes-to-avoid)
15. [File Submission Structure](#15-file-submission-structure)
16. [Contact / Coordination](#16-contact--coordination)

---

## 1. Project Overview

GECT Connect is a campus-exclusive chat messenger for students and staff of
Government Engineering College Thrissur. It is divided into 7 modules, each
owned by one team, all of which integrate into a single runnable Java Swing
application backed by a shared MySQL database.

**Team Responsibilities at a Glance:**

| Team | Module |
|------|--------|
| Team 1 | User Registration & Authentication — **YOU ARE HERE** |
| Team 2 | User Profile Management |
| Team 3 | Contact Management |
| Team 4 | Individual Chat |
| Team 5 | Group Chat |
| Team 6 | Academic Notifications & Media Sharing |
| Team 7 | Notifications, Settings & Campus Feed |

Team 1's module is the **entry point of the entire application.** The login
flow, session management, and the central `users` database table are all
owned by Team 1. Every other team depends on Team 1's output to function.

---

## 2. What Team 1 Owns

Team 1 is responsible for building and maintaining the following. Other teams
must not modify or duplicate any of these.

### Screens (UI Layer)
- `LoginScreen.java` — Email + password login
- `RegistrationScreen.java` — New user registration with role selection
- `OTPVerificationScreen.java` — Simulated OTP entry and verification
- `ForgotPasswordScreen.java` — Password reset flow
- `DashboardScreen.java` — Post-login hub screen (integration point for all teams)

### Shared Infrastructure (used by ALL teams)
- `DBConnection.java` — The one and only JDBC connection manager
- `SessionManager.java` — Holds the currently logged-in user in memory
- `User.java` — Base model class
- `Student.java` — Derived model class (for students)
- `Staff.java` — Derived model class (for staff)
- `ValidationUtil.java` — Email and input validators

### Database Tables (owned by Team 1)
- `users` — Master user table, referenced by all other teams
- `otp_records` — OTP log table

### SQL Scripts
- `sql/01_create_database.sql`
- `sql/02_create_tables.sql`
- `sql/03_seed_data.sql`

---

## 3. Technology Stack

| Component | Technology | Version |
|-----------|------------|---------|
| Language | Java (JDK) | 17 LTS (recommended) |
| GUI Framework | Java Swing | Built-in with JDK |
| Database | MySQL Community Server | 8.0+ |
| DB Driver | MySQL Connector/J | 8.x (.jar file) |
| Editor | Visual Studio Code | Latest |
| Java Extension | Extension Pack for Java | Microsoft (VS Code) |
| DB GUI Tool | MySQL Workbench | 8.0 (optional but recommended) |
| Terminal | Windows PowerShell or Git Bash | Built-in / Git for Windows |

---

## 4. Environment Setup — Windows + VS Code

Follow every step in order. Do not skip any step.

---

### Step 1 — Install Java JDK 17

1. Go to: https://adoptium.net
2. Download **Temurin JDK 17** — Windows x64 `.msi` installer
3. Run the installer. On the install options screen, make sure these are checked:
   - Set `JAVA_HOME` variable
   - Add to PATH
4. After install, open **PowerShell** and verify:

```powershell
java -version
javac -version
```

Expected output:
```
openjdk version "17.x.x" ...
javac 17.x.x
```

> If you see `'java' is not recognized`, Java is not on your PATH.
> Fix: Search "Environment Variables" in Windows Start Menu →
> Edit System Environment Variables → under System Variables find `Path` →
> Edit → New → paste `C:\Program Files\Eclipse Adoptium\jdk-17.x.x\bin`
> → OK all the way out → restart PowerShell.

---

### Step 2 — Install VS Code

1. Go to: https://code.visualstudio.com
2. Download and install the **Windows User Installer**
3. During install, check these options:
   - Add "Open with Code" action to Windows Explorer file context menu
   - Add "Open with Code" action to Windows Explorer directory context menu
   - Add to PATH
4. Launch VS Code

---

### Step 3 — Install the Java Extension Pack in VS Code

1. Open VS Code
2. Press `Ctrl + Shift + X` to open Extensions panel
3. Search: `Extension Pack for Java`
4. Install the one published by **Microsoft** (it shows 6 extensions inside)
5. Restart VS Code after install

This installs the Java Language Server, Debugger for Java, and project
management support all at once. After install, open any `.java` file —
you should see syntax highlighting and auto-complete working immediately.

---

### Step 4 — Install MySQL Community Server

1. Go to: https://dev.mysql.com/downloads/mysql/
2. Download **MySQL Community Server 8.0** — Windows MSI Installer
3. During setup, choose **Developer Default** installation type
4. When prompted to set root password, set it to: `root1234`
   (you only need this once to create the project user)
5. Keep MySQL running as a Windows Service (default option — leave it checked)

Verify in PowerShell:
```powershell
mysql --version
```

Expected output: `mysql  Ver 8.0.xx ...`

> If `mysql` is not recognized, add MySQL's bin folder to PATH:
> `C:\Program Files\MySQL\MySQL Server 8.0\bin`
> (Follow the same PATH editing steps from Step 1)

---

### Step 5 — Install MySQL Workbench (Recommended for All Team Members)

MySQL Workbench gives you a visual interface to run SQL, browse tables,
and debug database issues. It saves a lot of time — install it.

1. Go to: https://dev.mysql.com/downloads/workbench/
2. Download **MySQL Workbench 8.0** for Windows and install it
3. Launch Workbench, click the `+` next to "MySQL Connections"
4. Fill in:
   - Connection Name: `GECT Local`
   - Host: `127.0.0.1`
   - Port: `3306`
   - Username: `root`
5. Click "Test Connection" → enter password `root1234` → OK

---

### Step 6 — Create the Project Database and User

Open **PowerShell** and connect to MySQL as root:

```powershell
mysql -u root -p
```

Enter `root1234` when prompted. Then paste and run:

```sql
CREATE DATABASE IF NOT EXISTS gectconnect_db
  CHARACTER SET utf8mb4
  COLLATE utf8mb4_unicode_ci;

CREATE USER 'gectconnect'@'localhost' IDENTIFIED BY 'gect@1234';

GRANT ALL PRIVILEGES ON gectconnect_db.* TO 'gectconnect'@'localhost';

FLUSH PRIVILEGES;

EXIT;
```

> **All teams must use these exact credentials — no exceptions.**
> These are hardcoded in `DBConnection.java`. If you change them
> on your machine, your code will not work on anyone else's machine
> during integration.

| Setting | Value |
|---------|-------|
| Host | `localhost` |
| Port | `3306` |
| Database | `gectconnect_db` |
| Username | `gectconnect` |
| Password | `gect@1234` |

---

### Step 7 — Download the JDBC Driver

1. Go to: https://dev.mysql.com/downloads/connector/j/
2. Under "Select Operating System" choose **Platform Independent**
3. Download the ZIP file (not the MSI)
4. Extract the ZIP — find `mysql-connector-j-8.x.x.jar` inside
5. Copy this `.jar` file into your project's `lib/` folder

```
GECTConnect/
└── lib/
    └── mysql-connector-j-8.3.0.jar   ← place it here
```

VS Code will pick it up automatically via the settings in Section 5.

---

### Step 8 — Run Team 1's SQL Scripts

Open **PowerShell** inside the project folder
(`Shift + Right-click` on the folder → "Open PowerShell window here"):

```powershell
mysql -u gectconnect -pgect@1234 gectconnect_db < sql\02_create_tables.sql
mysql -u gectconnect -pgect@1234 gectconnect_db < sql\03_seed_data.sql
```

> Note the backslashes `\` — this is Windows PowerShell syntax.
> If you are using Git Bash, use forward slashes `/` instead.

Verify the tables were created in Workbench or PowerShell:
```sql
USE gectconnect_db;
SHOW TABLES;
-- Expected: users, otp_records

SELECT user_id, email, role, department FROM users;
-- Expected: 4 test seed accounts
```

---

### Step 9 — Run the Application

1. In VS Code: `File → Open Folder` → select the `GECTConnect` folder
2. Wait for the Java extension to finish indexing (progress bar at the bottom)
3. Open `src/gectconnect/auth/ui/LoginScreen.java`
4. Press `F5` to run with the debugger, or `Ctrl + F5` to run without
5. The Login screen window should appear

> **If you see "Build failed" or red underlines on import statements:**
> - Check that `lib/mysql-connector-j-8.x.x.jar` exists
> - Check that `.vscode/settings.json` has `"java.project.referencedLibraries": ["lib/**/*.jar"]`
> - Press `Ctrl + Shift + P` → type `Java: Clean Java Language Server Workspace` → restart

---

## 5. VS Code Configuration

This section sets up VS Code so every team member has the same zero-friction
development experience. Create these two files exactly as shown.

### Project Folder That VS Code Opens

```
GECTConnect/                    <- Open THIS folder in VS Code
├── .vscode/
│   ├── settings.json           <- Java classpath + formatting config
│   └── launch.json             <- Run/debug configuration
├── src/
│   └── gectconnect/
│       └── ...
├── lib/
│   └── mysql-connector-j-8.x.x.jar
└── sql/
    └── ...
```

---

### File: `.vscode/settings.json`

Create this file at `.vscode/settings.json` in your project root.

```json
{
    "java.project.sourcePaths": [
        "src"
    ],
    "java.project.referencedLibraries": [
        "lib/**/*.jar"
    ],
    "java.compile.nullAnalysis.mode": "automatic",
    "editor.tabSize": 4,
    "editor.insertSpaces": true,
    "files.encoding": "utf8",
    "files.eol": "\n"
}
```

What each line does:

- `java.project.sourcePaths` — tells VS Code your Java source root is `src/`
- `java.project.referencedLibraries` — automatically adds any `.jar` in `lib/` to classpath
- `editor.tabSize` / `insertSpaces` — everyone uses 4-space indentation, no tab characters
- `files.eol` — forces Unix line endings (`\n`) so SQL and Java files work on all machines

---

### File: `.vscode/launch.json`

Create this file at `.vscode/launch.json` in your project root.

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "java",
            "name": "Run GECT Connect",
            "request": "launch",
            "mainClass": "gectconnect.auth.ui.LoginScreen",
            "projectName": "GECTConnect",
            "classPaths": [
                "${workspaceFolder}/src",
                "${workspaceFolder}/lib/mysql-connector-j-8.x.x.jar"
            ]
        }
    ]
}
```

> Update the `.jar` filename in `classPaths` to match your exact downloaded
> version, for example `mysql-connector-j-8.3.0.jar`.

After both files are in place, pressing `F5` from any file in the project
will always launch `LoginScreen` correctly with the JDBC driver loaded.

---

### Recommended VS Code Extensions

Open the Extensions panel (`Ctrl + Shift + X`) and install all of these:

| Extension Name | Publisher | Why You Need It |
|----------------|-----------|-----------------|
| Extension Pack for Java | Microsoft | Java language, debugger, IntelliSense — **mandatory** |
| MySQL (Database Client) | cweijan | Run SQL queries inside VS Code without switching to Workbench |
| GitLens | GitKraken | See who changed what, line-by-line git blame |
| Indent Rainbow | oderwat | Color-coded indentation levels — makes Swing layout code readable |
| Java Code Generators | Sohibe | Right-click → Generate getters/setters/constructors automatically |
| Error Lens | usernamehw | Shows error messages inline next to the code line, not just in problems panel |

---

### Connecting to MySQL Inside VS Code

With the **MySQL (Database Client)** extension installed:

1. Click the database cylinder icon in the left sidebar
2. Click `+` → MySQL
3. Fill in:
   - Host: `localhost`
   - Port: `3306`
   - Username: `gectconnect`
   - Password: `gect@1234`
   - Database: `gectconnect_db`
4. Click Connect
5. To run a SQL file: right-click the `.sql` file in Explorer → **Run SQL File**

---

### Common Windows-Specific Issues and Fixes

**Problem: Spaces in the project folder path**

If your project is stored at `C:\Users\Your Name\Documents\GECTConnect`,
the space in `Your Name` can silently break Java classpath resolution.

Fix: Move the project to a path with no spaces:
```
C:\Projects\GECTConnect
```

**Problem: PowerShell blocks scripts**

Symptom: You see `running scripts is disabled on this system` in PowerShell.

Fix: Open PowerShell as Administrator and run once:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```

**Problem: MySQL password with `@` symbol in PowerShell**

The `@` in `gect@1234` may be interpreted as a variable in PowerShell.

Fix: Wrap the password in single quotes:
```powershell
mysql -u gectconnect -p'gect@1234' gectconnect_db < sql\02_create_tables.sql
```
Or use Git Bash instead — it handles this correctly without quoting.

**Problem: VS Code shows red squiggles on all imports**

The Java extension hasn't indexed the project yet, or the classpath is wrong.

Fix sequence:
1. Confirm `lib/mysql-connector-j-8.x.x.jar` exists
2. Confirm `.vscode/settings.json` has `"lib/**/*.jar"` in referencedLibraries
3. Press `Ctrl + Shift + P` → type `Java: Clean Language Server Workspace` → Restart

---

## 6. Database Setup — Run This First

> **CRITICAL ORDER:** Team 1's SQL creates the `users` table. Every other
> team's table has a FOREIGN KEY pointing to `users.user_id`. If you run
> your team's SQL before Team 1's, MySQL will throw:
> `ERROR 1215 (HY000): Cannot add foreign key constraint`
> and your tables will not be created.

**Always run in this order:**

```
1.  sql/02_create_tables.sql          <- Team 1  (users, otp_records)
2.  sql/03_seed_data.sql              <- Team 1  (test accounts)
3.  sql/0X_your_team_tables.sql       <- Your team (only after the above)
```

**PowerShell (Windows):**
```powershell
mysql -u gectconnect -p'gect@1234' gectconnect_db < sql\02_create_tables.sql
mysql -u gectconnect -p'gect@1234' gectconnect_db < sql\03_seed_data.sql
```

**MySQL Workbench:**
`File → Open SQL Script → select file → click the lightning bolt icon (Run)`

---

## 7. Database Schema Reference

### Table: `users`

The master table. Every other team's tables must foreign-key to `users.user_id`.

```sql
CREATE TABLE users (
    user_id          INT           NOT NULL AUTO_INCREMENT,
    email            VARCHAR(100)  NOT NULL UNIQUE,
    password_hash    VARCHAR(255)  NOT NULL,
    full_name        VARCHAR(100)  NOT NULL,
    role             ENUM('STUDENT','STAFF') NOT NULL,
    department       VARCHAR(50)   NOT NULL,
    mobile_number    VARCHAR(15)   DEFAULT NULL,
    roll_number      VARCHAR(20)   DEFAULT NULL,
    employee_id      VARCHAR(20)   DEFAULT NULL,
    profile_pic_path VARCHAR(255)  DEFAULT NULL,
    bio              VARCHAR(300)  DEFAULT NULL,
    is_active        TINYINT(1)    NOT NULL DEFAULT 1,
    last_seen        DATETIME      DEFAULT NULL,
    created_at       DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at       DATETIME      NOT NULL DEFAULT CURRENT_TIMESTAMP
                                   ON UPDATE CURRENT_TIMESTAMP,
    PRIMARY KEY (user_id)
);
```

### Table: `otp_records`

Owned by Team 1. Do not reference or modify from other modules.

```sql
CREATE TABLE otp_records (
    otp_id      INT         NOT NULL AUTO_INCREMENT,
    user_id     INT         NOT NULL,
    otp_code    VARCHAR(6)  NOT NULL,
    purpose     ENUM('REGISTRATION','PASSWORD_RESET') NOT NULL,
    is_used     TINYINT(1)  NOT NULL DEFAULT 0,
    expires_at  DATETIME    NOT NULL,
    created_at  DATETIME    NOT NULL DEFAULT CURRENT_TIMESTAMP,
    PRIMARY KEY (otp_id),
    FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
);
```

### Foreign Key Pattern — Copy This Into Your Tables

```sql
-- Every table that links to a user must do this:
FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE
```

### Key Columns — Quick Reference

| Column | Type | Notes |
|--------|------|-------|
| `user_id` | INT | **Primary key. Use as FK in all your tables.** |
| `email` | VARCHAR(100) | Always ends with `@gectcr.ac.in` |
| `full_name` | VARCHAR(100) | Use for display labels in UI |
| `role` | ENUM | Exactly `'STUDENT'` or `'STAFF'` — case-sensitive |
| `department` | VARCHAR(50) | e.g., `'CS'`, `'ECE'`, `'ME'` |
| `profile_pic_path` | VARCHAR(255) | **May be NULL — always null-check before use** |
| `is_active` | TINYINT(1) | `1` = active, `0` = disabled. Always filter by this. |
| `last_seen` | DATETIME | May be NULL for brand-new users |

### Test Seed Accounts (from `03_seed_data.sql`)

Use these to test your module without manually registering:

| Email | Password | Role | Department |
|-------|----------|------|------------|
| `student1@gectcr.ac.in` | `pass123` | STUDENT | CS |
| `student2@gectcr.ac.in` | `pass123` | STUDENT | ECE |
| `staff1@gectcr.ac.in` | `pass123` | STAFF | CS |
| `staff2@gectcr.ac.in` | `pass123` | STAFF | ME |

---

## 8. Shared Java Components

These files are written by Team 1 and shared with the entire project.
Do not rewrite them, do not create your own versions, do not copy-paste
them into your own packages. Import them directly.

---

### `DBConnection.java`
**Package:** `gectconnect.dao`

The single shared JDBC connection manager for the whole application.

```java
import gectconnect.dao.DBConnection;
import java.sql.Connection;

// In ANY DAO class across ANY team:
Connection conn = DBConnection.getConnection();
```

Never write this in your code:
```java
// WRONG — never create your own connection
Connection conn = DriverManager.getConnection("jdbc:mysql://localhost/gectconnect_db", "root", "");
```

---

### `SessionManager.java`
**Package:** `gectconnect.session`

Singleton. Stores the logged-in user for the entire app session.
Team 1 writes to it on login and clears it on logout.
All other teams only read from it.

```java
import gectconnect.session.SessionManager;
import gectconnect.model.User;

// Reading the logged-in user — do this in every screen constructor
User currentUser = SessionManager.getInstance().getCurrentUser();

// Checking login status
boolean isLoggedIn = SessionManager.getInstance().isLoggedIn();

// DO NOT call these from your module:
// SessionManager.getInstance().setCurrentUser(user);  <- Team 1 only
// SessionManager.getInstance().clearSession();         <- Team 1 only
```

---

### `User.java` — Abstract Base Class
**Package:** `gectconnect.model`

The base class. You will always receive either a `Student` or a `Staff`
from `SessionManager`. Both extend `User`, so these methods work on both:

```java
import gectconnect.model.User;
import gectconnect.model.Student;
import gectconnect.model.Staff;

User user = SessionManager.getInstance().getCurrentUser();

user.getUserId();           // int   — use as FK in your DB queries
user.getEmail();            // String
user.getFullName();         // String — use in display labels
user.getRole();             // String — "STUDENT" or "STAFF"
user.getDepartment();       // String
user.getMobileNumber();     // String — may be null
user.getProfilePicPath();   // String — may be null, always null-check
user.getBio();              // String — may be null
user.isActive();            // boolean
user.getLastSeen();         // java.sql.Timestamp — may be null

// Accessing role-specific data:
if (user.getRole().equals("STUDENT")) {
    Student s = (Student) user;
    String rollNo = s.getRollNumber();
} else {
    Staff st = (Staff) user;
    String empId = st.getEmployeeId();
}
```

---

### `ValidationUtil.java`
**Package:** `gectconnect.util`

```java
import gectconnect.util.ValidationUtil;

// Use these instead of writing your own validation logic:
ValidationUtil.isValidGECTEmail("john@gectcr.ac.in");  // true
ValidationUtil.isValidGECTEmail("john@gmail.com");      // false
ValidationUtil.isValidPassword("abc123");               // true  (6+ chars)
ValidationUtil.isValidPassword("ab1");                  // false (< 6 chars)
ValidationUtil.isNotEmpty("Hello");                     // true
ValidationUtil.isNotEmpty("");                          // false
ValidationUtil.isNotEmpty(null);                        // false
```

---

## 9. Package & Naming Conventions

### Root Package
All Java files in the entire project start with:
```
gectconnect
```

### Per-Team Packages

| Team | Package Root |
|------|-------------|
| Team 1 | `gectconnect.auth` |
| Team 2 | `gectconnect.profile` |
| Team 3 | `gectconnect.contacts` |
| Team 4 | `gectconnect.chat` |
| Team 5 | `gectconnect.groups` |
| Team 6 | `gectconnect.media` |
| Team 7 | `gectconnect.notifications` |

**Shared packages (Team 1 owns, all teams import):**
```
gectconnect.model
gectconnect.dao
gectconnect.session
gectconnect.util
```

Every team must have these four layers under their own root:
```
gectconnect.<yourmodule>.model      <- Entity/POJO classes
gectconnect.<yourmodule>.dao        <- Database operations
gectconnect.<yourmodule>.service    <- Business logic
gectconnect.<yourmodule>.ui         <- Swing screens
```

### Java Naming Rules

| Element | Style | Example |
|---------|-------|---------|
| Classes | PascalCase | `ChatScreen`, `MessageDAO` |
| Methods | camelCase | `sendMessage()`, `getUserById()` |
| Variables | camelCase | `currentUser`, `messageList` |
| Constants | UPPER_SNAKE_CASE | `MAX_MSG_LENGTH` |
| Packages | all lowercase | `gectconnect.chat.dao` |

### SQL Naming Rules

| Element | Style | Example |
|---------|-------|---------|
| Tables | snake_case, plural | `messages`, `group_members` |
| Columns | snake_case | `sender_id`, `sent_at` |
| Primary keys | `<singular>_id` | `message_id`, `group_id` |
| Foreign keys | same name as the PK it references | `user_id` → `users.user_id` |
| Timestamps | always these exact names | `created_at`, `updated_at` |

### SQL File Names Per Team

```
sql/01_create_database.sql              <- Team 1
sql/02_create_tables.sql                <- Team 1
sql/03_seed_data.sql                    <- Team 1
sql/04_team2_profile_tables.sql         <- Team 2
sql/05_team3_contacts_tables.sql        <- Team 3
sql/06_team4_chat_tables.sql            <- Team 4
sql/07_team5_groups_tables.sql          <- Team 5
sql/08_team6_media_tables.sql           <- Team 6
sql/09_team7_notifications_tables.sql   <- Team 7
```

---

## 10. How to Get the Logged-In User

This is the most important pattern in the project. Copy it exactly.

```java
package gectconnect.profile.ui;   // example: Team 2

import gectconnect.model.User;
import gectconnect.session.SessionManager;
import javax.swing.*;

public class ProfileScreen extends JFrame {

    private User currentUser;

    public ProfileScreen() {
        // Always get user from SessionManager in the constructor
        this.currentUser = SessionManager.getInstance().getCurrentUser();

        // Always guard against null session (safety net)
        if (this.currentUser == null) {
            JOptionPane.showMessageDialog(null,
                "Session expired. Please log in again.",
                "Session Error",
                JOptionPane.ERROR_MESSAGE);
            new gectconnect.auth.ui.LoginScreen().setVisible(true);
            dispose();
            return;
        }

        initComponents();
        loadData();
    }
}
```

---

## 11. How to Navigate From the Dashboard

`DashboardScreen.java` is owned by Team 1 but has one button per module
for all 7 teams. Your main screen class name needs to be given to Team 1
so they can wire the button.

**The navigation pattern used everywhere:**
```java
// Inside any ActionListener (button click):
private void openProfileModule() {
    new gectconnect.profile.ui.ProfileScreen().setVisible(true);
    this.dispose();   // always close the current screen
}
```

**Your main screen MUST have a no-argument constructor:**
```java
// Correct — screen fetches user from SessionManager internally
new YourMainScreen().setVisible(true);

// Wrong — do not require User in constructor
new YourMainScreen(someUserObject).setVisible(true);
```

**Every screen must have a "Back to Dashboard" button:**
```java
JButton btnBack = new JButton("<- Back to Dashboard");
btnBack.addActionListener(e -> {
    new gectconnect.auth.ui.DashboardScreen().setVisible(true);
    this.dispose();
});
```

---

## 12. How to Build a Screen (Template)

Copy this as the starting point for every new Swing screen you create.

```java
package gectconnect.<yourmodule>.ui;

import gectconnect.model.User;
import gectconnect.session.SessionManager;

import javax.swing.*;
import javax.swing.border.EmptyBorder;
import java.awt.*;

/**
 * [Screen Name]
 * Module  : [Your Module Name]
 * Team    : [Team Number]
 * Purpose : [One line description]
 */
public class YourScreen extends JFrame {

    private final User currentUser;

    // Declare Swing components here
    private JLabel  lblTitle;
    private JButton btnBack;

    /**
     * No-argument constructor.
     * Fetches logged-in user from SessionManager.
     */
    public YourScreen() {
        this.currentUser = SessionManager.getInstance().getCurrentUser();

        if (this.currentUser == null) {
            JOptionPane.showMessageDialog(null,
                "Session expired. Please log in again.",
                "Error", JOptionPane.ERROR_MESSAGE);
            new gectconnect.auth.ui.LoginScreen().setVisible(true);
            dispose();
            return;
        }

        setTitle("GECT Connect — [Screen Title]");
        setSize(800, 600);
        setLocationRelativeTo(null);
        setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);
        setResizable(false);

        initComponents();
        loadData();
        setVisible(true);
    }

    /**
     * Build and layout all Swing components.
     */
    private void initComponents() {
        JPanel main = new JPanel(new BorderLayout(10, 10));
        main.setBorder(new EmptyBorder(15, 15, 15, 15));

        lblTitle = new JLabel("[Screen Title]", SwingConstants.CENTER);
        lblTitle.setFont(new Font("Arial", Font.BOLD, 20));
        main.add(lblTitle, BorderLayout.NORTH);

        // Add your content panel here

        btnBack = new JButton("<- Back to Dashboard");
        btnBack.addActionListener(e -> goToDashboard());
        main.add(btnBack, BorderLayout.SOUTH);

        add(main);
    }

    /**
     * Load data from DB via your DAO.
     * Always use currentUser.getUserId() — never hardcode a user ID.
     */
    private void loadData() {
        // Example:
        // List<YourEntity> data = yourDAO.getByUserId(currentUser.getUserId());
        // populate UI components with data
    }

    /**
     * Navigate to another screen and close this one.
     */
    private void navigateTo(JFrame next) {
        next.setVisible(true);
        this.dispose();
    }

    /** Return to the main Dashboard. */
    private void goToDashboard() {
        navigateTo(new gectconnect.auth.ui.DashboardScreen());
    }
}
```

---

## 13. Integration Checklist Per Team

Go through this list before handing off your code to the final integration merge.

### All Teams (2–7)

- [ ] Root package is `gectconnect.<yourmodule>` — not `com.gect` or anything else
- [ ] Module has `model`, `dao`, `service`, `ui` sub-packages
- [ ] Using `DBConnection.getConnection()` — no custom JDBC connection anywhere
- [ ] Getting logged-in user from `SessionManager.getInstance().getCurrentUser()`
- [ ] NOT calling `setCurrentUser()` or `clearSession()` (Team 1 only)
- [ ] Main screen has a zero-argument constructor
- [ ] SQL file named `sql/0X_teamX_<module>_tables.sql`
- [ ] All your tables have FK pointing to `users.user_id`
- [ ] Tested with seed accounts from `03_seed_data.sql`
- [ ] "Back to Dashboard" button on every screen
- [ ] `.vscode/settings.json` has `"lib/**/*.jar"` in `referencedLibraries`
- [ ] Project compiles and launches cleanly with `F5` in VS Code
- [ ] No `System.out.println` used for user-visible messages (use `JOptionPane`)

### Team 2 (Profile)
- [ ] Updating `profile_pic_path` and `bio` in the `users` table directly
- [ ] NOT creating a separate profile table — `users` already has those columns
- [ ] Notified Team 1 when done so `getProfilePicPath()` can be used by others

### Team 3 (Contacts)
- [ ] `contacts` table has both `user_id` and `contact_user_id` as FKs to `users.user_id`
- [ ] Block list stored in your own table, not in `users`

### Team 4 (Individual Chat)
- [ ] `sender_id` and `receiver_id` both FK to `users.user_id`
- [ ] Message status stored as exact strings: `'SENT'`, `'DELIVERED'`, `'READ'`

### Team 5 (Group Chat)
- [ ] Group creator stored as `creator_id` FK to `users.user_id`
- [ ] Group members table has `user_id` FK to `users.user_id`

### Team 6 (Media & Announcements)
- [ ] Announcements store `posted_by` as FK to `users.user_id`
- [ ] Faculty-only screens guarded by `currentUser.getRole().equals("STAFF")`

### Team 7 (Notifications & Settings)
- [ ] Settings table has `user_id` FK to `users.user_id`
- [ ] Notification records have `user_id` FK to `users.user_id`
- [ ] Coordinated with Team 1 before writing to `last_seen` column in `users`

---

## 14. Common Mistakes to Avoid

### Database Mistakes

| Wrong | Correct |
|-------|---------|
| Create your own `users` or `user_info` table | Use Team 1's `users` table |
| Connect as `root` in your DAO | Use `gectconnect` via `DBConnection.getConnection()` |
| Use `Statement` with string concatenation | Always use `PreparedStatement` |
| Run your SQL before Team 1's SQL | Always run `02_create_tables.sql` first |
| Hardcode DB credentials in your DAO | Use `DBConnection.getConnection()` |

### Java Mistakes

| Wrong | Correct |
|-------|---------|
| Pass `User` through screen constructors | Fetch from `SessionManager` inside constructor |
| Re-query the DB to get the current user | Use `SessionManager.getInstance().getCurrentUser()` |
| `DriverManager.getConnection(...)` in your DAO | Use `DBConnection.getConnection()` |
| Package `com.gect.*` or `org.gect.*` | Use `gectconnect.*` |
| `System.out.println()` for errors | Use `JOptionPane.showMessageDialog()` |
| `System.exit(0)` to close a window | Use `this.dispose()` |

### Windows-Specific Mistakes

| Wrong | Correct |
|-------|---------|
| Project stored at `C:\Users\My Name\...` (space in path) | Move to `C:\Projects\GECTConnect` |
| Forget to add `.jar` to VS Code classpath | `"java.project.referencedLibraries": ["lib/**/*.jar"]` in settings.json |
| Edit Java files in Notepad | Always use VS Code with Extension Pack for Java |
| Use `\r\n` line endings (Windows default) | Add `"files.eol": "\n"` to VS Code settings |
| Paste MySQL password with `@` unquoted in PowerShell | Wrap in single quotes: `-p'gect@1234'` |

### Integration Mistakes

| Wrong | Correct |
|-------|---------|
| Assume user is always a Student | Check `user.getRole()` before casting |
| Assume `profile_pic_path` is never null | `if (user.getProfilePicPath() != null)` before use |
| Call `SessionManager.clearSession()` from your module | Only Team 1 calls this on logout |
| Hardcode `user_id = 1` for testing | Use `currentUser.getUserId()` at all times |

---

## 15. File Submission Structure

### Final Integration Zip (all teams together)

```
GECTConnect_Final/
├── .vscode/
│   ├── settings.json
│   └── launch.json
├── src/
│   └── gectconnect/
│       ├── model/              <- Team 1
│       ├── dao/                <- Team 1
│       ├── session/            <- Team 1
│       ├── util/               <- Team 1
│       ├── auth/               <- Team 1
│       │   ├── service/
│       │   └── ui/
│       ├── profile/            <- Team 2
│       ├── contacts/           <- Team 3
│       ├── chat/               <- Team 4
│       ├── groups/             <- Team 5
│       ├── media/              <- Team 6
│       └── notifications/      <- Team 7
├── sql/
│   ├── 01_create_database.sql
│   ├── 02_create_tables.sql
│   ├── 03_seed_data.sql
│   ├── 04_team2_profile_tables.sql
│   ├── 05_team3_contacts_tables.sql
│   ├── 06_team4_chat_tables.sql
│   ├── 07_team5_groups_tables.sql
│   ├── 08_team6_media_tables.sql
│   └── 09_team7_notifications_tables.sql
├── lib/
│   └── mysql-connector-j-8.x.x.jar
├── docs/
│   ├── ClassDiagram.txt
│   ├── DFD_Level0.txt
│   ├── DFD_Level1.txt
│   └── DatabaseSchema.txt
└── README.md
```

### Individual Team Submission (Google Classroom)

Each member uploads their own zip:
```
Team1_<RollNumber>_GECTConnect.zip
├── requirements/       <- Requirements document
├── design/             <- Class diagram, DFD, DB schema
├── src/                <- Your team's source files only
├── sql/                <- Your team's SQL file only
└── testlog/            <- Test log in standard format
```

---

## 16. Contact / Coordination

**Integration Coordinator:** Team 1

Before you build a workaround for anything in the shared infrastructure —
contact Team 1. If you need a method added to `UserDAO`, a field added to
`User`, a Dashboard button wired up for your screen, or any clarification
about shared components — ask first, build second.

### Day-by-Day Plan

| Days | What to do |
|------|-----------|
| **Day 1–2** | Read this README completely. Install JDK 17, VS Code + Extension Pack for Java, MySQL 8.0, MySQL Workbench. Set up `.vscode/settings.json` and `launch.json`. Run Team 1's SQL. Verify `users` table exists in Workbench before writing a single line of code. |
| **Day 3–5** | Build your DAO and service layer. Write and test all DB operations using the 4 seed accounts. Confirm reads and writes work correctly. |
| **Day 6–8** | Build your Swing screens using the template from Section 12. Wire navigation. Test that your module launches from `F5` in VS Code. |
| **Day 9–10** | Integration testing. Run all teams' SQL in order. Launch `LoginScreen`, log in as a seed user, navigate to your module from Dashboard, verify everything works end-to-end. |
| **Day 11–14** | Fix bugs. Write documentation. Prepare test log. Zip your submission folder and upload to Google Classroom. Every team member uploads individually. |

---

> **Last updated by Team 1.**
> This README is the authoritative source for all integration decisions.
> If this document conflicts with any other reference, this README wins
> for anything related to shared infrastructure, naming, and DB structure.
> For your module's own requirements, refer to the original project spec.
