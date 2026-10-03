# goddamn_avi


# ✈️ Final Aviation Website Architecture

```text
                         USER
                          │
                          ▼
              ┌──────────────────────┐
              │      FRONTEND         │
              │                      │
              │ HTML                 │
              │ CSS                  │
              │ JavaScript           │
              │ Leaflet              │
              └──────────┬───────────┘
                         │
                    HTTP / JSON
                         │
                         ▼
              ┌──────────────────────┐
              │     FLASK BACKEND    │
              │                      │
              │ REST API             │
              │ Business Logic       │
              │ Data Processing      │
              └───────┬───────┬──────┘
                      │       │
             ┌────────┘       └─────────┐
             ▼                          ▼
   ┌──────────────────┐       ┌──────────────────┐
   │   PostgreSQL     │       │ Aviation APIs    │
   │                  │       │                  │
   │ Aircraft         │       │ ADSB.lol         │
   │ Airlines         │       │ OpenSky          │
   │ Airports         │       │ Other sources    │
   │ Flights          │       │                  │
   │ Flight History   │       └──────────────────┘
   └──────────────────┘
```

## 🛠️ Technology Stack

| Part             | Technology                         |
| ---------------- | ---------------------------------- |
| Frontend         | **HTML + CSS + JavaScript**        |
| Map              | **Leaflet.js**                     |
| Backend          | **Python + Flask**                 |
| API              | **Flask REST API**                 |
| Database         | **PostgreSQL**                     |
| Database library | **SQLAlchemy**                     |
| Aviation data    | **ADSB.lol / OpenSky**             |
| Version control  | **Git + GitHub**                   |
| Deployment       | Free-tier services where practical |

No React/Next.js is necessary for the first version.

---

# 📁 Project Structure

I'd use a single repository:

```text
aviation-explorer/
│
├── app.py
├── config.py
├── requirements.txt
├── .env
├── .gitignore
├── README.md
│
├── models/
│   ├── __init__.py
│   ├── aircraft.py
│   ├── airline.py
│   ├── airport.py
│   └── flight.py
│
├── routes/
│   ├── __init__.py
│   ├── aircraft.py
│   ├── airlines.py
│   ├── airports.py
│   ├── flights.py
│   └── live.py
│
├── services/
│   ├── aviation_api.py
│   └── flight_service.py
│
├── templates/
│   ├── index.html
│   ├── aircraft.html
│   ├── aircraft_details.html
│   ├── airlines.html
│   ├── airline_details.html
│   ├── airports.html
│   ├── airport_details.html
│   ├── flights.html
│   ├── flight_details.html
│   └── live.html
│
├── static/
│   ├── css/
│   │   ├── style.css
│   │   ├── home.css
│   │   ├── aircraft.css
│   │   └── map.css
│   │
│   ├── js/
│   │   ├── main.js
│   │   ├── aircraft.js
│   │   ├── flights.js
│   │   ├── search.js
│   │   └── map.js
│   │
│   └── images/
│
└── database/
    ├── schema.sql
    └── seed.py
```

You can simplify the folder structure even further at the beginning; the important part is keeping **frontend, Flask logic, database models, and external API logic separated**.

---

# 🗄️ Database

PostgreSQL will contain your core aviation information.

### Aircraft models

```text
aircraft_models
────────────────────
id
icao_type
name
manufacturer
description
first_flight
cruise_speed
max_speed
range_km
max_altitude
capacity
```

Example:

```text
B789
Boeing 787-9
Boeing
...
```

### Individual aircraft

```text
aircraft
────────────────────
id
registration
icao24
aircraft_model_id
airline_id
serial_number
year
```

Example:

```text
N123AB
A1B2C3
Boeing 787-9
United Airlines
```

### Airlines

```text
airlines
────────────────────
id
name
icao_code
iata_code
country
callsign
```

### Airports

```text
airports
────────────────────
id
icao_code
iata_code
name
city
country
latitude
longitude
```

### Flights

```text
flights
────────────────────
id
callsign
aircraft_id
airline_id
origin_airport_id
destination_airport_id
departure_time
arrival_time
status
```

### Flight positions

For live/history data:

```text
flight_positions
────────────────────
id
flight_id
latitude
longitude
altitude
speed
heading
timestamp
```

---

# 🔗 Database Relationships

This is one of the most important parts of your project.

```text
Aircraft Model
      │
      │ 1 → many
      ▼
 Individual Aircraft
      │
      │
      ▼
    Flight
   /      \
  ▼        ▼
Origin    Destination
Airport    Airport

Flight ───────► Airline
```

For example:

```text
Boeing 787-9
      ↓
N123AB
      ↓
UAL123
      ↓
United Airlines
      ↓
Dhaka (DAC) ──────► London (LHR)
```

That's exactly the interconnected aviation explorer you're describing.

---

# 🌐 Flask API

The browser shouldn't directly communicate with ADSB/OpenSky.

Instead:

```text
Browser
   ↓
Flask
   ↓
Aviation API
```

This lets Flask process and normalize the data.

### Aircraft

```text
GET /api/aircraft
GET /api/aircraft/<id>
GET /api/aircraft/model/<icao_type>
```

### Airlines

```text
GET /api/airlines
GET /api/airlines/<icao>
```

### Airports

```text
GET /api/airports
GET /api/airports/<icao>
```

### Flights

```text
GET /api/flights
GET /api/flights/<callsign>
```

### Live

```text
GET /api/live
GET /api/live/<icao24>
```

### Search

```text
GET /api/search?q=boeing
```

---

# 🖥️ Frontend

Your frontend stays simple:

```text
HTML
  +
CSS
  +
JavaScript
```

JavaScript communicates with Flask:

```text
JavaScript
     │
     │ fetch()
     ▼
Flask API
     │
     ▼
JSON
     │
     ▼
JavaScript
     │
     ▼
Update HTML
```

For example:

```javascript
fetch("/api/live/N123AB")
    .then(response => response.json())
    .then(data => {
        document.querySelector("#altitude").textContent =
            data.altitude;
    });
```

---

# 🗺️ Live Map

Use **Leaflet.js**.

```text
                 Leaflet Map
                     │
          ┌──────────┼──────────┐
          ▼          ▼          ▼
        ✈️          ✈️         ✈️
      UAL123       BAW21      EK585
```

JavaScript periodically requests:

```text
/api/live
```

Flask gets/processes the current aircraft data and returns JSON.

The map updates the aircraft markers.

---

# 🏠 Main Pages

Your website can have:

### Home

```text
┌───────────────────────────────────┐
│ AVIATION EXPLORER                 │
│                                   │
│ Search aircraft, flights, airports│
│                                   │
├───────────────────────────────────┤
│ Popular Aircraft                  │
│ 737 │ A320 │ 777 │ 787 │ A350    │
├───────────────────────────────────┤
│ Live Flights                      │
│                                   │
│             MAP                   │
│                                   │
├───────────────────────────────────┤
│ Popular Airlines                  │
│ Popular Airports                  │
└───────────────────────────────────┘
```

### Aircraft

Browse aircraft models.

```text
Boeing
 ├── 737
 ├── 747
 ├── 777
 └── 787

Airbus
 ├── A320
 ├── A330
 ├── A350
 └── A380
```

### Aircraft model

```text
Boeing 787-9
────────────────────

Specifications
Operators
Photos
Aircraft count

Currently Flying
────────────────────

N123AB
N456CD
JA123A
...
```

### Individual aircraft

```text
N123AB

Boeing 787-9
United Airlines

Current Flight: UAL123

Altitude: 36,000 ft
Speed: 470 kt
Heading: 274°

Dhaka ───────────► London

             LIVE MAP
```

### Airline

```text
United Airlines

ICAO: UAL
IATA: UA
Country: United States

Fleet
Current flights
Airports
Routes
```

### Airport

```text
Hazrat Shahjalal International Airport

ICAO: VGHS
IATA: DAC
Dhaka, Bangladesh

Arrivals
Departures
Live flights
Airlines
Routes
```

### Flight

```text
UAL123

United Airlines
Boeing 787-9
N123AB

DAC ───────────► LHR

Altitude
Speed
Position
Route
Flight status

        LIVE MAP
```

---

# 🔄 How the whole system works

Suppose someone searches:

**N123AB**

```text
                USER
                  │
                  ▼
             Search box
                  │
                  ▼
           JavaScript
                  │
                  ▼
       GET /api/search?q=N123AB
                  │
                  ▼
               FLASK
                  │
          ┌───────┴────────┐
          ▼                ▼
      PostgreSQL       Aviation API
          │                │
          └───────┬────────┘
                  ▼
                JSON
                  │
                  ▼
             JavaScript
                  │
                  ▼
          Aircraft Page
```

Then the user clicks the current flight:

```text
N123AB
   ↓
UAL123
   ↓
DAC → LHR
   ↓
DAC Airport
   ↓
LHR Airport
```

Everything is interconnected.

---

# 🚀 Build it in stages

Even though the final architecture includes PostgreSQL, **don't build everything simultaneously**.

### Stage 1

```text
Flask
HTML
CSS
JavaScript
PostgreSQL
```

Build the basic pages and database.

### Stage 2

Add:

```text
Aircraft
Airlines
Airports
Flights
```

with database relationships.

### Stage 3

Connect:

```text
ADSB.lol / OpenSky
```

and bring in real flight data.

### Stage 4

Add:

```text
Leaflet
```

and build the live map.

### Stage 5

Connect everything:

```text
Aircraft
   ↓
Individual Aircraft
   ↓
Flight
   ↓
Airline
   ↓
Origin Airport
   ↓
Destination Airport
```

### Stage 6

Later, if the project becomes large, you can add things like:

```text
Redis
background workers
caching
WebSockets
React/Next.js
advanced flight history
authentication
notifications
```

**But none of those belong in your initial architecture.**

---

# ✅ Final stack

So I would lock your initial project to:

```text
┌─────────────────────────────────────────┐
│              AVIATION SITE              │
├─────────────────────────────────────────┤
│                                         │
│  Frontend                               │
│  ├── HTML                               │
│  ├── CSS                                │
│  ├── JavaScript                         │
│  └── Leaflet.js                         │
│                                         │
│              ↓ HTTP / JSON              │
│                                         │
│  Backend                                │
│  └── Python + Flask                     │
│                                         │
│          ↙                    ↘         │
│                                         │
│  PostgreSQL              Aviation APIs  │
│  ├── Aircraft            ├── ADSB.lol  │
│  ├── Airlines            └── OpenSky   │
│  ├── Airports                           │
│  ├── Flights                            │
│  └── Positions                           │
│                                         │
└─────────────────────────────────────────┘

