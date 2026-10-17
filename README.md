<div align="center">

# INNsight Hotel Management

A JavaFX desktop app for managing hotel rooms, guests and reservations, with an embedded Botpress chatbot that books through a local REST API.

![Java](https://img.shields.io/badge/Java-17-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![JavaFX](https://img.shields.io/badge/JavaFX-21.0.2-1F6FB2?style=flat-square)
![SQLite](https://img.shields.io/badge/SQLite-3.45-003B57?style=flat-square&logo=sqlite&logoColor=white)
![Maven](https://img.shields.io/badge/Build-Maven-C71A36?style=flat-square&logo=apachemaven&logoColor=white)
![Botpress](https://img.shields.io/badge/Chatbot-Botpress-0F172A?style=flat-square)

</div>

## Overview

INNsight is a desktop application for running the front desk of a small hotel. Staff sign in to a JavaFX interface to manage rooms, guests and bookings, while a chatbot embedded in the app lets a guest book, modify or cancel a stay in plain language. The app starts a small REST API on port 8080 backed by a local SQLite database, and the Botpress workflows call that API through an ngrok tunnel. The code is split into UI, controllers, models and repositories, and class comments map each piece to a GRASP pattern.

## Key Features

- **Room management:** add, edit and delete rooms (Single, Double, Suite, Deluxe, Penthouse) with Available, Booked and Maintenance status, plus status filter tabs. A room with active reservations cannot be deleted.
- **Reservations:** card-style booking list, create a booking for an existing or new guest, change check-in and check-out dates, cancel a booking, and track Paid or Unpaid. Overlapping bookings on the same room are rejected, and check-out must be after check-in.
- **Guest directory:** add, edit and delete guests with a unique email per guest. A guest with an active reservation cannot be deleted.
- **Dashboard:** occupancy percentage, active bookings, available rooms, total guests, revenue from confirmed bookings, and the five most recent reservations.
- **Embedded AI chat:** a Botpress webchat loaded in a JavaFX WebView, with automatic retries (up to three) when the page fails to load and an option to open the chat in a browser.
- **REST API:** JSON endpoints for rooms, customers, reservations and dashboard stats, served by the JDK built-in HTTP server with Gson.
- **Layered design:** repository interfaces over SQLite, controllers per use case, and entities that validate themselves through a shared `BaseEntity`.
- **Themes:** four colour themes (Dark, Ocean, Sunset, Void) switched from the sidebar, with animated page and card transitions.

## Tech Stack

| Area | Technology |
|---|---|
| Language | Java 17 (source and target level in `pom.xml`) |
| Desktop UI | JavaFX 21.0.2 (controls, web) with CSS styling |
| REST API | JDK built-in HTTP server (`jdk.httpserver`) on port 8080, Gson 2.10.1 |
| Database | SQLite through sqlite-jdbc 3.45.1.0 |
| Chatbot | Botpress webchat embedded in a JavaFX WebView |
| Tunnel | ngrok, to expose the local API to Botpress |
| Build and packaging | Maven (javafx-maven-plugin 0.0.8, maven-dependency-plugin) and `jpackage` |

## Project Structure

```text
.
|-- pom.xml                          # Maven build: JavaFX, sqlite-jdbc, Gson
|-- INNsight.ico                     # Application icon
`-- src/main
    |-- java
    |   |-- module-info.java         # Java module descriptor
    |   `-- com/hotel/hotel_ai
    |       |-- Main.java            # Launcher entry point
    |       |-- ui/ChatbotUI.java    # JavaFX screens: login, dashboard, rooms, reservations, guests, chat
    |       |-- controller/          # Room, Reservation and Customer controllers, ApiServer, AIEngine
    |       |-- model/               # BaseEntity, Room, Reservation, Customer
    |       `-- repository/          # Repository interfaces, SQLite repositories, DatabaseManager
    `-- resources                    # styles.css and logo images
```

## Getting Started

### Prerequisites

- JDK 17 or newer (the project was packaged with JDK 21)
- Apache Maven
- Optional, for the chat features: ngrok and a Botpress bot (see Usage)

### Installation

```bash
git clone https://github.com/shahramshafiq/innsight-hotel-management.git
cd innsight-hotel-management
mvn clean package
```

### Run

```bash
mvn javafx:run
```

The app opens maximized with a logo splash, then a login card. On first launch it creates the database at `<your home folder>/INNsight/innsight.db`, seeds 8 sample rooms and 3 sample guests, and creates a default admin account (`admin` / `admin123`). The app reads no environment variables.

To build a Windows app image instead (run after `mvn clean package`, which also copies the dependencies into `target/libs`):

```bash
copy target\hotel_ai-1.0-SNAPSHOT.jar target\libs\
jpackage --input target\libs --main-jar hotel_ai-1.0-SNAPSHOT.jar --main-class com.hotel.hotel_ai.Main --name INNsight --app-version 1.0 --type app-image --dest INNsight_App
```

## Usage

The REST API starts together with the desktop app on `http://localhost:8080`.

| Method | Endpoint | Purpose |
|---|---|---|
| GET | `/api/rooms` | All rooms |
| GET | `/api/rooms/available` | Rooms with Available status |
| GET | `/api/customers` | All guests |
| POST | `/api/customers` | Add a guest (`fullName`, `email`, `phone`) |
| GET | `/api/reservations` | All reservations |
| POST | `/api/reservations` | Book a room |
| POST | `/api/reservations/cancel` | Cancel a booking (`reservationId`) |
| POST | `/api/reservations/modify` | Change dates (`reservationId`, `checkIn`, `checkOut`) |
| GET | `/api/dashboard` | Room, booking, guest and revenue totals |

Example requests (dates use `YYYY-MM-DD`, and `roomId` is the `id` field returned by `/api/rooms`):

```bash
curl http://localhost:8080/api/rooms/available

curl -X POST http://localhost:8080/api/reservations \
  -H "Content-Type: application/json" \
  -d '{"guestName":"Jane Doe","email":"jane@example.com","phone":"0300-0000000","roomId":1,"checkIn":"2026-12-01","checkOut":"2026-12-04","paymentStatus":"Unpaid"}'
```

To let the chatbot write to the local database, run `ngrok http 8080`, then use the forwarding URL as the API base URL inside your Botpress workflows (booking, cancellation and modification) and publish them. The URL changes every ngrok session.

## Limitations and Roadmap

- There is no automated test suite yet. Unit tests for the controllers and repositories would be the first addition.
- Security is demo-grade: the default admin password is stored in plain text in SQLite, and the REST API has no authentication and allows any origin, so it is meant for local use and short ngrok sessions only.
- The chat view loads a hosted Botpress webchat whose address is set in `ChatbotUI.java`, and the Botpress workflows are not part of this repository. Pointing it at your own bot means recreating those workflows. The `AIEngine` webhook client and its offline replies are not connected to the chat view.
- `ChatbotUI.java` holds every screen in one class of about 1,170 lines. Splitting it into one class per view is the natural next refactor.

## Author

**Shahram Shafiq** - [GitHub](https://github.com/shahramshafiq) | [LinkedIn](https://www.linkedin.com/in/shahramshafiq/) | Built together with Sara Nasir and Taha Jabbar
