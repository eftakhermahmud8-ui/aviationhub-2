# AGENTS.md — Aviation Hub v2

## 1. Project Goal

Aviation Hub v2 is a full-stack aviation information platform for aviation enthusiasts, spotters, and professionals.

The platform connects:

- Aircraft models
- Individual aircraft
- Airlines
- Airports
- Flights
- Live aircraft positions
- Aviation news
- Aviation photos
- User profiles
- Favorites
- User contributions

The goal is **not** to build a complicated Flightradar24 clone immediately.

Build a clean, understandable first version that can grow later.

---

# 2. Core Architecture

Use a simple monolithic architecture.

```text
                         USER
                          |
                          v
              +------------------------+
              |       FRONTEND         |
              |                        |
              | HTML                   |
              | CSS                    |
              | Vanilla JavaScript     |
              | Leaflet.js             |
              +-----------+------------+
                          |
                     HTTP / JSON
                          |
                          v
              +------------------------+
              |      FLASK BACKEND     |
              |                        |
              | Web routes             |
              | REST API               |
              | Authentication         |
              | Business logic         |
              +-----+-------------+----+
                    |             |
                    |             |
                    v             v
          +----------------+   +------------------+
          |  PostgreSQL    |   | External APIs    |
          |                |   |                  |
          | Aircraft       |   | ADSB.lol         |
          | Airports       |   | OpenSky          |
          | Airlines       |   | News APIs/RSS    |
          | Flights        |   | Photo sources    |
          | Users          |   |                  |
          | Photos         |   +------------------+
          | News           |
          +----------------+
```

## Important

Do NOT introduce these in the initial version:

- React
- Next.js
- Redis
- Docker
- Kubernetes
- Microservices
- WebSockets
- Celery
- Elasticsearch
- GraphQL

They may be added later if the project actually needs them.

---

# 3. Technology Stack

## Frontend

- HTML5
- CSS3
- Vanilla JavaScript
- Leaflet.js for maps

Do not use a frontend framework for v2 unless there is a concrete requirement.

## Backend

- Python
- Flask
- Flask-SQLAlchemy
- Flask-Migrate
- Flask-Login
- Werkzeug password hashing

## Database

- PostgreSQL

Use SQLAlchemy models instead of writing raw SQL throughout the application.

## External aviation data

Prefer free/open data sources.

Possible sources:

- ADSB.lol
- OpenSky
- OpenStreetMap
- Public airport/aircraft datasets
- RSS feeds for aviation news

Do not depend on paid APIs in the initial version.

## Deployment

The application should be designed for free-tier-first deployment.

Do not introduce infrastructure that requires payment for basic functionality.

---

# 4. Repository Structure

Use this structure:

```text
aviation-hub/
│
├── app/
│   ├── __init__.py
│   │
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── aircraft.py
│   │   ├── airline.py
│   │   ├── airport.py
│   │   ├── flight.py
│   │   ├── flight_position.py
│   │   ├── photo.py
│   │   ├── news.py
│   │   └── favorite.py
│   │
│   ├── routes/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── aircraft.py
│   │   ├── airlines.py
│   │   ├── airports.py
│   │   ├── flights.py
│   │   ├── live.py
│   │   ├── news.py
│   │   ├── gallery.py
│   │   ├── users.py
│   │   └── api.py
│   │
│   ├── services/
│   │   ├── aviation.py
│   │   ├── news.py
│   │   └── photos.py
│   │
│   ├── templates/
│   │   ├── base.html
│   │   ├── index.html
│   │   ├── aircraft/
│   │   ├── airports/
│   │   ├── airlines/
│   │   ├── flights/
│   │   ├── news/
│   │   ├── gallery/
│   │   ├── profile/
│   │   └── auth/
│   │
│   └── static/
│       ├── css/
│       │   ├── style.css
│       │   ├── home.css
│       │   ├── aircraft.css
│       │   ├── airports.css
│       │   ├── flights.css
│       │   ├── news.css
│       │   ├── gallery.css
│       │   └── profile.css
│       │
│       ├── js/
│       │   ├── main.js
│       │   ├── search.js
│       │   ├── aircraft.js
│       │   ├── airports.js
│       │   ├── flights.js
│       │   ├── live-map.js
│       │   └── gallery.js
│       │
│       └── images/
│
├── migrations/
│
├── tests/
│   ├── test_models.py
│   ├── test_api.py
│   └── test_routes.py
│
├── scripts/
│   └── seed_database.py
│
├── config.py
├── run.py
├── requirements.txt
├── .env.example
├── .gitignore
├── README.md
└── AGENTS.md
```

Keep the structure understandable. Do not create files just for the sake of abstraction.

---

# 5. Database Design

PostgreSQL is the main persistent data store.

## Aircraft Models

Represents an aircraft type/model.

Example:

```text
Boeing 787-9
Airbus A350-900
Boeing 737-800
```

Fields:

```text
id
icao_type
manufacturer
model
variant
description
first_flight
cruise_speed
max_speed
range_km
max_altitude
passenger_capacity
created_at
updated_at
```

---

## Aircraft

Represents a physical aircraft.

Example:

```text
Registration: N123AB
ICAO24: A1B2C3
Model: Boeing 787-9
```

Fields:

```text
id
registration
icao24
serial_number
aircraft_model_id
airline_id
year_built
status
created_at
updated_at
```

Relationship:

```text
Aircraft Model 1 ---- N Aircraft
```

---

# 6. Airlines

Fields:

```text
id
name
icao_code
iata_code
callsign
country
country_code
logo_url
website
description
created_at
updated_at
```

Relationship:

```text
Airline 1 ---- N Aircraft
Airline 1 ---- N Flights
```

---

# 7. Airports

Fields:

```text
id
name
icao_code
iata_code
city
country
latitude
longitude
elevation
timezone
website
description
created_at
updated_at
```

Initially, do not attempt to model every runway/terminal as a separate complex system.

If needed later, add:

```text
runways
terminals
```

---

# 8. Flights

A flight represents a journey/service.

Fields:

```text
id
callsign
flight_number
aircraft_id
airline_id
origin_airport_id
destination_airport_id
status
scheduled_departure
scheduled_arrival
actual_departure
actual_arrival
created_at
updated_at
```

Relationships:

```text
Aircraft ---- N Flights
Airline ---- N Flights
Airport ---- N Flights (origin)
Airport ---- N Flights (destination)
```

---

# 9. Flight Positions

This table stores live or historical position data.

Fields:

```text
id
flight_id
aircraft_id
latitude
longitude
altitude
ground_speed
heading
vertical_rate
timestamp
```

Do not store every live position forever in the first version.

Use a sensible retention strategy later.

---

# 10. Photos

Photos belong to users and can optionally be associated with an aircraft or airport.

Fields:

```text
id
user_id
aircraft_id
airport_id
title
description
image_url
photographer
location
taken_at
created_at
status
```

Do not store large image binary data directly in PostgreSQL.

Store an image URL/path instead.

For v2, use a free-compatible image storage solution where practical.

---

# 11. News

News items should be normalized into the application's format.

Fields:

```text
id
title
summary
source
source_url
image_url
published_at
category
created_at
```

Do not copy entire articles.

Store metadata, summary, source, and link to the original article.

---

# 12. Users

Fields:

```text
id
username
email
password_hash
display_name
bio
avatar_url
created_at
updated_at
```

Never store plaintext passwords.

Use Werkzeug password hashing.

---

# 13. Favorites

Users can favorite aircraft and airports.

For the first version:

```text
favorites
----------------
id
user_id
aircraft_id
airport_id
created_at
```

A favorite should point to either an aircraft or airport.

Do not over-engineer this initially.

---

# 14. Main User Flow

The most important relationship in the application is:

```text
Aircraft Model
      |
      v
Individual Aircraft
      |
      v
Current Flight
      |
      +------------------+
      |                  |
      v                  v
Origin Airport     Destination Airport
      |
      v
Airline
```

Example:

```text
Boeing 787-9
      ↓
N123AB
      ↓
UAL123
      ↓
Dhaka (DAC) → London (LHR)
      ↓
United Airlines
```

Every page should link naturally to related entities.

---

# 15. Main Pages

## Home

Show:

- Search
- Featured aircraft
- Popular aircraft models
- Popular airports
- Popular airlines
- Current live flights
- Latest aviation news
- Recent gallery photos

---

## Aircraft

Aircraft model browsing.

Filters:

- Manufacturer
- Model
- Variant
- Airline
- Registration

Example:

```text
Aircraft
├── Boeing
│   ├── 737
│   ├── 747
│   ├── 777
│   └── 787
└── Airbus
    ├── A320
    ├── A330
    ├── A350
    └── A380
```

---

## Aircraft Details

Example:

```text
Boeing 787-9

Specifications
Operators
Photos
History

Currently Flying
- N123AB
- N456CD
- JA123A
```

---

## Individual Aircraft

Show:

- Registration
- ICAO24
- Aircraft model
- Airline
- Serial number
- Year
- Current flight
- Current position
- Altitude
- Speed
- Heading
- Origin
- Destination

---

## Airports

Show:

- Airport name
- IATA
- ICAO
- City
- Country
- Coordinates
- Elevation
- Photos
- Current arrivals
- Current departures
- Airlines
- Routes

---

## Airlines

Show:

- Airline information
- IATA/ICAO
- Country
- Fleet
- Current flights
- Airports served

---

## Flights

Show:

- Callsign
- Flight number
- Airline
- Aircraft
- Origin
- Destination
- Status
- Current position

---

## Live Map

Use Leaflet.js.

The first version should show:

- Aircraft markers
- Callsign
- Position
- Altitude
- Speed
- Heading

Clicking an aircraft opens a small information panel and links to its aircraft/flight page.

---

## News

Show:

- Latest aviation news
- Source
- Publication time
- Category
- Image
- Link to original article

News can initially come from RSS feeds or other free sources.

---

## Gallery

Show:

- Aircraft photos
- Airport photos
- Photographer
- Aircraft registration/model when available
- Airport when available

---

## Upload

Logged-in users can submit:

- Aircraft information
- Airport information
- Photos
- News tips

User submissions should not automatically overwrite trusted aviation database data.

Use a moderation/status field.

---

## Profile

Show:

- Username
- Avatar
- Bio
- Uploaded photos
- Contributions
- Favorites

---

# 16. API Design

The frontend should communicate with Flask through JSON APIs.

## Aircraft

```text
GET /api/aircraft
GET /api/aircraft/<id>
GET /api/aircraft/model/<icao_type>
```

## Individual aircraft

```text
GET /api/aircraft/registration/<registration>
GET /api/aircraft/icao24/<icao24>
```

## Airlines

```text
GET /api/airlines
GET /api/airlines/<id>
```

## Airports

```text
GET /api/airports
GET /api/airports/<id>
```

## Flights

```text
GET /api/flights
GET /api/flights/<id>
GET /api/flights/callsign/<callsign>
```

## Live

```text
GET /api/live
GET /api/live/<icao24>
```

## Search

```text
GET /api/search?q=<query>
```

Search should be able to find:

- Aircraft registration
- Aircraft model
- Airline
- Airport name
- IATA
- ICAO
- Callsign

---

# 17. External API Architecture

Never make the frontend directly depend on an external aviation API.

Use:

```text
Browser
   ↓
Flask
   ↓
Service layer
   ↓
External API
```

Example:

```text
JavaScript
    ↓
GET /api/live
    ↓
Flask
    ↓
aviation.py
    ↓
ADSB.lol / OpenSky
    ↓
Normalize data
    ↓
JSON response
    ↓
JavaScript
```

The service layer should convert different external API formats into one internal format.

This prevents the frontend from becoming dependent on a specific provider.

---

# 18. Live Data Strategy

Do not try to implement a perfect real-time tracking system initially.

Start with polling.

Example:

```text
Browser
   |
   | every 10-30 seconds
   v
GET /api/live
   |
   v
Flask
   |
   v
Aviation API
```

Later, WebSockets can be added if real-time performance requires them.

---

# 19. Frontend Rules

Use:

```text
HTML
CSS
Vanilla JavaScript
```

Keep JavaScript modular.

Example:

```text
static/js/
├── main.js
├── search.js
├── aircraft.js
├── airports.js
├── flights.js
├── live-map.js
└── gallery.js
```

Do not put the entire application into one giant `script.js`.

Use `fetch()` for API requests.

Avoid unnecessary frontend dependencies.

---

# 20. CSS Rules

Use normal CSS.

Create:

```text
style.css
```

for global styles and separate files only when a page becomes large.

Use CSS variables for the design system.

Example:

```css
:root {
    --bg: #0b1120;
    --surface: #111827;
    --text: #f8fafc;
    --muted: #94a3b8;
    --accent: #38bdf8;
}
```

The design should be:

- Modern
- Aviation-focused
- Responsive
- Dark/light friendly if practical
- Clean
- Fast
- Accessible

Do not add huge animation libraries.

---

# 21. Security

Minimum requirements:

- Hash passwords.
- Never commit `.env`.
- Validate uploaded files.
- Restrict image file types.
- Limit upload size.
- Validate user input.
- Use parameterized database queries through SQLAlchemy.
- Add CSRF protection for forms.
- Require authentication for uploads/favorites/profile changes.
- Do not expose API keys to JavaScript.
- Keep secrets server-side.

---

# 22. Environment Variables

Use `.env`.

Example:

```text
SECRET_KEY=
DATABASE_URL=

ADSB_API_URL=
OPENSKY_CLIENT_ID=
OPENSKY_CLIENT_SECRET=

NEWS_API_URL=
```

Only include credentials that are actually required.

Provide `.env.example` with empty values.

Never commit `.env`.

---

# 23. Development Rules

## Keep it simple

Prefer:

```text
simple working code
```

over:

```text
complex abstraction
```

Do not create a service, class, utility, or abstraction unless it solves a real problem.

## Build incrementally

Implement in this order:

### Phase 1

```text
Flask
PostgreSQL
HTML
CSS
JavaScript
```

Create:

- Home
- Aircraft
- Airports
- Airlines
- Flights

Use seed/sample data.

### Phase 2

Create database relationships.

```text
Aircraft Model
    ↓
Aircraft
    ↓
Flight
    ↓
Airports
```

### Phase 3

Add real aviation data.

```text
ADSB.lol / OpenSky
```

### Phase 4

Add Leaflet live map.

### Phase 5

Add authentication.

### Phase 6

Add gallery and user uploads.

### Phase 7

Add news aggregation.

### Phase 8

Add favorites and profiles.

---

# 24. Do Not Overbuild

The first working version does NOT need:

- AI
- recommendation systems
- machine learning
- advanced analytics
- real-time WebSockets
- Redis
- background job systems
- microservices
- Kubernetes
- multiple databases

If a future feature genuinely requires one of these, introduce it at that point.

---

# 25. Git Rules

Use clear commits.

Examples:

```text
feat: add aircraft database model
feat: add aircraft API
feat: add aircraft details page
feat: add airport pages
feat: add live aircraft map
fix: handle missing aircraft data
style: improve aircraft cards
docs: update setup instructions
```

Do not make giant commits containing unrelated changes.

---

# 26. Testing

At minimum, test:

- Database models
- API responses
- Search
- Aircraft lookup
- Airport lookup
- Authentication
- Upload validation

Use `pytest`.

Do not wait until the entire project is finished before testing.

---

# 27. Performance

Do not optimize prematurely.

Initially:

```text
Browser
   ↓
Flask
   ↓
PostgreSQL
```

is enough.

Add caching only after a real performance problem appears.

For live aviation data, avoid unnecessarily requesting external APIs on every user action.

---

# 28. Free-First Requirement

The project should prioritize:

1. Open-source software
2. Free APIs
3. Free database tiers
4. Free hosting tiers
5. Static assets/CDNs with free usage
6. User-contributed content

Avoid paid services unless there is no reasonable free alternative.

Never design the core application around a paid-only API.

---

# 29. Important Data Principle

Separate trusted aviation data from user-generated content.

For example:

```text
Trusted database
    |
    ├── Boeing 787-9
    ├── Dhaka Airport
    └── United Airlines

User content
    |
    ├── Photos
    ├── Descriptions
    └── Tips
```

A user-uploaded photo should not automatically modify the official aircraft specifications.

---

# 30. Coding Agent Instructions

When working on this repository:

1. Read `AGENTS.md` before making changes.
2. Inspect the existing project before creating files.
3. Do not rewrite working code unnecessarily.
4. Keep the architecture simple.
5. Follow the existing database relationships.
6. Use Flask for backend logic.
7. Use HTML/CSS/Vanilla JavaScript for frontend.
8. Use PostgreSQL through SQLAlchemy.
9. Keep external API calls inside the service layer.
10. Never expose secrets to the frontend.
11. Test changes before considering them complete.
12. Update documentation when architecture changes.
13. Do not introduce new dependencies without a reason.
14. Prefer free/open-source solutions.
15. Do not implement future features unless explicitly requested.

---

# 31. Definition of Done

A feature is considered complete when:

- The backend works.
- The database relationship is correct.
- The API returns the required data.
- The frontend displays the data correctly.
- Errors are handled.
- The page works on mobile.
- Basic security requirements are respected.
- Relevant tests pass.
- Documentation is updated if necessary.

---

# 32. Final Architecture

The intended v2 architecture is:

```text
                         AVIATION HUB
                              |
          +-------------------+-------------------+
          |                                       |
          v                                       v
      FRONTEND                                FLASK
          |                                       |
   HTML/CSS/JS                              REST API
   Leaflet.js                                    |
          |                         +------------+------------+
          |                         |                         |
          +-----------------------> |                     PostgreSQL
                                    |
                                    v
                              Service Layer
                                    |
                      +-------------+-------------+
                      |             |             |
                      v             v             v
                   ADSB.lol      OpenSky      News/RSS
```

The most important entity chain is:

```text
Aircraft Model
      ↓
Individual Aircraft
      ↓
Flight
      ↓
Origin Airport
      ↓
Destination Airport

Aircraft ───────────► Airline
Flight ──────────────► Airline
Airport ─────────────► Airline / Flights
```

This architecture should remain the baseline unless a future requirement clearly justifies changing it.
