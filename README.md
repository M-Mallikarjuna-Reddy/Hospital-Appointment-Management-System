# CarePoint — Hospital Appointment Management System

A Java desktop application for managing hospital patients, doctors, weekly clinic hours, and timed appointments. The interface is built with JavaFX; JDBC connects to MySQL.

## Problem Statement

Manual hospital appointment management can be time-consuming and may cause difficulties in checking doctor availability, managing appointments, and preventing double bookings.

## Solution

CarePoint provides a centralized system for patients, doctors, and administrators to manage hospital appointments. It allows patients to search doctors and book appointments, doctors to manage availability and visits, and administrators to manage patients, doctors, departments, and appointments.


## Features

- Role-based sign-in for administrators, doctors, and patients, with patient and doctor self-registration.
- Patient and doctor profiles can be edited from the account profile icon; password updates live in the same profile panel.
- Patient self-registration; patient records can also be maintained by an administrator.
- Users can change their own password after signing in.
- Admin dashboard with patient, doctor, appointment, today, upcoming, completed, and cancelled totals plus a status chart.
- Patient search and record management; doctor search by name, specialty, and department.
- Department management and doctor login creation.
- Weekly doctor availability with configurable slot lengths.
- Appointment booking, rescheduling, cancellation, and status updates.
- Available-time selection, future date/time validation, recurring availability checks, and duplicate-slot protection in both Java and MySQL.
- Appointment records retain the patient, doctor, visit date/time, reason, and visit outcome; doctor accounts can review their patient visit history.
- Passwords stored as salted PBKDF2 hashes. Original plain-text accounts are upgraded to a hash after their first successful login.
- Prepared statements, foreign keys, unique constraints, indexes, and friendly validation/error messages.

## Technologies

- Java 17+
- JavaFX 21 (Controls and Charts)
- MySQL 8.0.16 or newer
- JDBC with MySQL Connector/J 9.1
- Maven

## Requirements

Install a JDK 17 or newer, Maven 3.8 or newer, and MySQL 8.0.16 or newer. Maven resolves JavaFX and the JDBC driver from the project dependencies; no connector JAR needs to be copied into the repository.

## Database setup

For a new database, run `database/schema.sql` in MySQL Workbench or from a terminal:

```sh
mysql -u root -p < database/schema.sql
```

For a database created by the original Swing version, back it up first, then run `database/upgrade_existing_db.sql` once:

```sh
mysql -u root -p < database/upgrade_existing_db.sql
```

The upgrade script preserves existing patient, doctor, and appointment records. Older appointments receive a default time of 09:00. If one doctor already has more than one active appointment on the same date, edit those records to unique times before the final unique-slot index is created. Existing user passwords remain usable and are replaced with PBKDF2 hashes after successful login.

## Database connection

Database credentials are configured outside the normal sign-in UI. Copy `db.properties.example` to the ignored local file `db.properties` and enter your local values, or set the `HOSPITAL_DB_URL`, `HOSPITAL_DB_USER`, and `HOSPITAL_DB_PASSWORD` environment variables before launching the app. Environment variables take precedence over `db.properties`.

The defaults target `localhost:3306/hospital_db`, username `root`, and an empty password. If MySQL uses a password, configure it locally before launch. If the schema is missing, run `database/schema.sql` from the project directory.

PowerShell:

```powershell
$env:HOSPITAL_DB_URL = "jdbc:mysql://localhost:3306/hospital_db?serverTimezone=UTC&useSSL=false"
$env:HOSPITAL_DB_USER = "root"
$env:HOSPITAL_DB_PASSWORD = "your-local-mysql-password"
```

The application does not store database credentials in tracked source files. `db.properties` is a local plaintext settings file excluded by `.gitignore`; do not commit it or share its contents.

## Run the application

From the project directory:

```sh
mvn clean javafx:run
```

On the first launch against a fresh schema, the application creates a demo administrator if an account named `admin` does not already exist:

- Username: `admin`
- Password: `admin123`

Change or remove this demo account before using the application with real data. Patients can create their own accounts from the sign-in screen. Administrators create doctor profiles together with their login credentials.

## Project structure

```text
src/hospital/
  Main.java
  controller/       Role-aware JavaFX event controllers
  dao/              Prepared-statement JDBC data access
  db/               Environment-based JDBC connection
  model/            Patient, Doctor, Appointment, User, Department, Availability
  service/           Validation, scheduling rules, and dashboard operations
  ui/                JavaFX login, dashboards, management views, and CSS
  util/              PBKDF2 password hashing
database/
  schema.sql
  upgrade_existing_db.sql
```

## Database tables

- `users`: usernames, password hashes, roles, email, and active status.
- `patients`: patient demographics and optional link to a patient login.
- `doctors`: provider profile, optional linked login, department, fee, and active status.
- `departments`: searchable clinical departments.
- `doctor_availability`: recurring weekday windows and appointment slot length.
- `appointments`: patient, doctor, date, time, reason, status, and audit timestamps. A generated active-slot column and unique key prevent two non-cancelled visits for the same doctor/date/time.

## Manual verification checklist

After starting MySQL and launching the app, verify:

1. Sign in as `admin` with the initial demo password; try a wrong password.
2. Register a patient without an email field, sign in as that patient, and search for doctors by specialty.
3. As admin, add a doctor login, a department, and weekday availability.
4. Book an appointment inside an available future slot, then try booking that doctor at the same time again.
5. Reschedule and cancel an appointment; confirm a cancelled slot becomes available again.
6. Sign in as the doctor and update one of their scheduled visits to completed.
7. Try invalid email in patient/doctor management, plus invalid age, time, past date, duplicate department name, and unavailable time.
8. Stop MySQL and confirm the app displays a connection error rather than crashing.

## Verification performed

- Maven packaging passed with Java 17 source compatibility and JavaFX 21 dependencies.
- A temporary MySQL 8.0.44 instance passed integration checks for valid/invalid login, patient registration and CRUD/search, doctor create/update/search, department CRUD, availability and overlap rules, appointment booking, database-level duplicate prevention, patient ownership, rescheduling, cancellation, doctor/admin status changes, invalid dates/times, dashboard counts, and password changes.
- The original four-table schema upgrade passed against disposable legacy data: patient, doctor, and appointment records were preserved; old visits received 09:00; roles/status values were normalized; the legacy admin password was upgraded after login.
- With the temporary database stopped, JDBC returned a handled `SQLException`; the JavaFX login window rendered the database setup message without crashing.

These checks used temporary files under the ignored `target/` directory and did not touch a user database. The verification harness itself is not included in the project.

## Screenshots

Add screenshots of the JavaFX sign-in screen and role dashboards here after launching the application.

## Current limitations

- Database-backed workflows require a locally configured MySQL service; this repository does not include a database server or production credentials.
- The demo administrator password is intended only for first-run evaluation.
- Existing accounts from the original schema are assigned the `PATIENT` role by default, except the original `admin` account. Existing doctor records remain available, but migration does not automatically link them to login accounts.
- Automated database integration tests require a configured MySQL instance and are not bundled with this student project.
