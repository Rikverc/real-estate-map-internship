# real-estate-map-internship

A production-grade Android application built as a **proof of concept** to explore scalable, map-based real-estate discovery.  
The goal was to validate architecture, data flow, and performance strategies for handling **location-driven listings**, pagination, and image loading at scale.

The project focuses on **clean architecture**, **state-driven UI**, and **efficient data loading**, rather than UI polish.

---

## Tech Stack

**Language & Platform**
- Kotlin
- Android

**UI**
- Jetpack Compose
- Google Maps Compose

**Architecture**
- MVVM
- Clean Architecture (domain / data / presentation separation)

**Concurrency & State**
- Kotlin Coroutines
- Flow / StateFlow

**Networking & Data**
- Retrofit
- Gson
- Local storage (test database)

**Dependency Injection**
- Hilt

---

## Project Overview

The application allows users to explore real-estate listings **based on map visibility**, rather than traditional text search.

Key idea:
> **The visible map bounds drive which locations and listings are loaded.**

This enables:
- Efficient querying
- Progressive loading
- Clear separation between *locations* and *listings*

### Important distinction

- **Location ≠ Listing**
  - *Location* represents a geographical area (e.g. municipality, city, region)
  - *Listing* represents an individual real-estate ad within a location

This distinction is central to the app’s data model and performance optimizations.

---

## User Flow

1. **Map initialization**
   - Map centers on the user’s current location
   - Default example: Belgrade

2. **Explore area**
   - User taps **“Explore area”**
   - The system reads current map bounds
   - Only locations fully or partially visible are considered

3. **Markers appear**
   - Markers represent **locations**, not listings
   - Marker density adapts to zoom level

4. **Marker selection**
   - Tapping a marker opens a bottom sheet
   - Sheet displays all listings for that location

5. **Listing selection**
   - Navigates to a detail screen
   - Displays full listing data and image

6. **Pagination & scaling**
   - Locations are loaded in batches
   - “Load more” fetches additional pages without reloading existing data

---

## Data Pipeline

CSV (locations seed)
↓
Local storage (test DB)
↓
User input (map bounds)
↓
Filtered location IDs
↓
Remote API (listings)
↓
Image API (per listing)
↓
UI (Compose)


Key characteristics:
- Locations are filtered **before** API calls, by utilizing complex SQL queries to the local DB
- Images are loaded asynchronously and paired with listings
- Domain layer never depends on UI or framework code

---

## Architecture Overview

The project follows **Clean Architecture** principles:

presentation/
└── ViewModel (state, user actions)

domain/
├── use_cases/
└── models/

data/
├── remote/
├── local/
└── dto/


### Responsibilities

**Presentation**
- Compose UI
- ViewModels expose `StateFlow`
- No business logic

**Domain**
- Use cases encapsulate application logic
- Pure Kotlin
- Framework-agnostic

**Data**
- Retrofit APIs
- DTO → domain model mapping
- Image decoding

---

## Image Loading Strategy

- Images are fetched **after listings**, not inline with listing requests
- Each listing is paired with its image into a `ListingWithImage` domain model
- Image loading:
  - Uses coroutines
  - Falls back to a placeholder when unavailable
- UI never handles raw byte arrays or decoding

This avoids:
- Blocking UI threads
- Coupling UI to networking
- Inconsistent list/image alignment

---

## Pagination & Performance Optimizations

### Batched loading
- Locations and listings are loaded in **controlled batches**
- Prevents overwhelming the API and UI

### Bounds-aware queries
- Only locations within visible bounds are queried
- Entire regions are loaded only when fully visible

### Example optimization
- A municipality marker appears **only when the entire area is visible**
- Zooming in hides the marker and prevents unnecessary queries

---

## State Management

- All UI state is driven by `StateFlow`
- No mutable state in composables
- ViewModel is the single source of truth

Handled states include:
- Marker visibility
- Listing pagination
- Loading states
- Selected listing

---

## Known Challenges & Learnings

- Synchronizing paginated data with asynchronous image loading
- Avoiding index mismatches between listings and images
- Handling large datasets without blocking UI
- Gson field naming pitfalls with mixed naming conventions
- Designing domain models that survive refactors

These challenges heavily influenced:
- Domain model design
- Use case boundaries
- ViewModel responsibilities

---

## Status

This project is an **experimental proof of concept**, not a production app.

Its primary value lies in:
- Architecture decisions
- Data flow design
- Performance strategies
- Real-world Compose + Maps integration

---

## Disclaimer

This repository does **not** contain any code, and as such does not contain proprietary backend logic or production APIs.  
Some implementation details are simplified or anonymized to respect usage rights.

