# Reka AI Social Manager — Spring Boot backend

This is the "real backend" version of your content calendar: a Java (Spring Boot) server
with a proper database, instead of the browser-only storage from the first version.
The same black-themed calendar frontend is included and served directly by the server.

## What changed vs. the browser-only version
- Clients and posts are stored in a real database (H2 file DB by default — easy to swap for Postgres/MySQL)
- A server-side scheduler (`AlarmScheduler`) checks every 60 seconds for content that's due,
  and can send an **email reminder** — this works even if nobody has the calendar open in a browser
- The frontend now talks to REST endpoints instead of `window.storage`, so any number of people
  hitting the same server URL see the same live data

## Requirements
- Java 17 or newer
- Maven 3.9+ (or use your IDE's built-in Maven)

## Run it
```bash
cd reka-backend
mvn spring-boot:run
```
Then open **http://localhost:8080** in a browser — that's the calendar.

Or build a runnable jar:
```bash
mvn clean package
java -jar target/reka-social-manager.jar
```

## Database
By default this uses H2, a file-based database that lives in `./data/rekadb.mv.db` next to
wherever you run the app — no separate database server needed, and data survives restarts.

You can browse it at **http://localhost:8080/h2-console**
(JDBC URL: `jdbc:h2:file:./data/rekadb;AUTO_SERVER=TRUE`, user `sa`, blank password).

### Moving to Postgres/MySQL later
Swap the 4 datasource lines in `src/main/resources/application.properties` for your real
connection details, and add the matching JDBC driver dependency to `pom.xml`
(`org.postgresql:postgresql` or `mysql:mysql-connector-j`). No Java code changes needed —
Spring Data JPA handles the rest.

## Email alarms (optional)
By default the scheduler only logs to the console when content is due. To get real emails:

1. In `application.properties`, set:
   ```
   reka.alarm.mail-enabled=true
   reka.alarm.agency-email=you@youragency.com
   spring.mail.username=your-smtp-username
   spring.mail.password=your-smtp-app-password
   ```
2. If a customer has a `notifyEmail` set on their profile, reminders for their content go to
   them directly; otherwise they fall back to `reka.alarm.agency-email`.
3. The example config uses Gmail SMTP — for Gmail you'll need an
   [App Password](https://myaccount.google.com/apppasswords), not your regular password.

Note: true browser push notifications (the kind that pop up on a phone's lock screen without
any email) need a separate piece — a service worker + a push provider (e.g. Firebase Cloud
Messaging or web push/VAPID keys). Email is the realistic "works with zero extra
infrastructure" option; ask if you want push added later.

## API endpoints (for reference)
| Method | Path                          | Purpose                        |
|--------|-------------------------------|---------------------------------|
| GET    | `/api/clients`                | List all customers              |
| POST   | `/api/clients`                | Create a customer               |
| PUT    | `/api/clients/{id}`           | Update a customer                |
| DELETE | `/api/clients/{id}`           | Delete a customer (and their posts) |
| GET    | `/api/clients/{id}/posts`     | List a customer's content        |
| POST   | `/api/clients/{id}/posts`     | Create content for a customer    |
| PUT    | `/api/posts/{id}`             | Update a piece of content        |
| DELETE | `/api/posts/{id}`             | Delete a piece of content        |
| GET    | `/api/alarms/today`           | Today's due content, across all customers |

## Project layout
```
src/main/java/com/reka/socialmanager/
  model/          Client, Post, AlarmView (JPA entities + DTO)
  repository/     Spring Data JPA repositories
  controller/     REST controllers
  service/        AlarmScheduler (the @Scheduled job that checks for due content)
src/main/resources/
  application.properties   database + mail config
  static/index.html        the calendar frontend, served at "/"
```
