# Smart Study Dashboard

A dashboard where students can digitally raise a question for a teacher, and teachers see that queue **live**. It also includes a focus timer (Pomodoro style) with configurable breaks.

The project consists of two parts:

| Part | Folder | What it is |
|---|---|---|
| **Web app / backend** | `sample/` | Java web application (Struts 2 + Hibernate + MySQL) with a JSON API, WebSocket and the dashboard UI |
| **Student app** | `SSDStudenten/` | Java Swing tray app where a student logs in and raises a question |

## Features

- **Live queue**: students pick a teacher and subject and raise a question; the dashboard updates instantly via WebSocket (`/ws/wachtrij`)
- **Help button**: a teacher marks a question as handled, and the queue refreshes for everyone
- **Cooldown**: a student can raise a question for the same teacher only once per minute
- **Teacher filter** on the queue
- **Focus timer**: configurable focus and break durations, school breaks, and sound alerts when a session or break ends
- **Student overview** with the ability to delete students
- **Registration with email verification** (verification code sent via SMTP)
- **Themes**: light, dark, summer (fireflies) and winter (snow), plus daily quotes

## Tech stack

- Java 21, Maven (WAR packaging)
- Struts 2 (7.0.3) + JSON plugin
- Hibernate ORM 6.6 + MySQL (Connector/J 9.2)
- Jakarta Servlet 6 / Jakarta WebSocket 2.1 (requires Tomcat 10.1+ or similar)
- Jakarta Mail (Angus) for verification emails
- Vanilla JavaScript, JSP and CSS for the frontend
- Java Swing + `java.net.http` for the student app

## Getting started

### Prerequisites

- JDK 21 and Maven
- Tomcat 10.1+ (or another Jakarta EE 10 servlet container)
- A MySQL database

### 1. Configuration

Database and email credentials are **not** part of this repository and must be set up locally:

1. Update the connection details in `sample/src/main/resources/hibernate/hibernate.cfg.xml` (and `hibernate.properties`):
   ```xml
   <property name="hibernate.connection.url">jdbc:mysql://localhost:3306/your_database</property>
   <property name="hibernate.connection.username">your_user</property>
   <property name="hibernate.connection.password">your_password</property>
   ```
2. Set the SMTP details for `EmailService` (host, port, username and an app password).

> Tables are created automatically via `hibernate.hbm2ddl.auto = update`. Turn this off in production.

### 2. Build and run the web app

```bash
cd sample
mvn clean package
```

Copy `target/sample.war` into Tomcat's `webapps/` folder and start the server. The dashboard is then available at:

```
http://localhost:8080/sample/
```

### 3. Run the student app

The student app talks to `http://localhost:8080/sample` (see `BASE_URL` in `MijnVraagApp.java` and `LoginDialog.java`).

```bash
cd SSDStudenten
mvn clean package
```

Then run `trayApp.MijnTrayApp` (for example from your IDE). The app asks you to log in or register first and then appears as an icon in the system tray.

## API

All endpoints live under `/sample/api/`.

| Endpoint | Purpose |
|---|---|
| `login` | Student login |
| `register` | Register a student (sends a verification code) |
| `verifieer` | Verify an account with the code |
| `getAllStudents` | Fetch all students |
| `deleteStudent` | Delete a student |
| `getDocenten` | List of teachers |
| `getVakken` | List of subjects |
| `getWachtrij` | Fetch the active queue |
| `updateWachtrij` | Raise or resolve a question |
| `getSettings` / `saveSettings` | Read and save timer settings |

WebSocket: `ws://localhost:8080/sample/ws/wachtrij` sends `QUEUE_UPDATED` whenever the queue changes.
