# RideBuddy V1 — Product Requirements

**Version:** 0.1
**Status:** Draft
**Product:** RideBuddy
**Purpose:** Student-focused ride-pooling platform

---

## 1. Product Vision

RideBuddy is a student-focused ride-pooling platform that connects students who need transportation with verified students who are already traveling in the same direction with available seats.

The platform focuses specifically on recurring student commutes such as:

* Home/locality → College
* College → Home/locality

RideBuddy is not intended to function as a general-purpose taxi or ride-hailing platform.

The initial system will be built as a production-oriented application and tested initially in controlled environments before any broader rollout.

---

## 2. User Types

RideBuddy has one primary user type: **Student**.

A verified student can act as both:

### Rider

A student looking for a ride.

### Driver

A student who owns/uses a verified vehicle and wants to offer available seats on a trip they are already making.

A user may switch between these roles without maintaining separate accounts.

---

## 3. Authentication & Account

Users must authenticate before using RideBuddy.

A user account contains:

* Name
* Email
* Phone number
* Profile information
* College
* Default commute preference
* Home locality
* Verification status
* Rating
* Ride history

Users must be associated with a registered college.

The system should distinguish between:

```text
Unverified Student
        ↓
College Verified
        ↓
Vehicle Verified (if posting rides)
```

---

# 4. Commute Preferences

After login, the user chooses their primary commute direction:

### Ride to College

```text
Current Location → College
```

### Ride to Home

```text
Current Location → Saved Home Locality
```

The selected option becomes the user's default commute mode.

The user must still be able to switch between the two modes later.

---

# 5. Ride to College

## 5.1 Starting Location

The user provides their current pickup location.

The application may allow the user to:

* Use their current location
* Select a location manually
* Select a nearby designated pickup point

Exact residential addresses should not be exposed to other users.

---

## 5.2 Destination

The destination is selected from RideBuddy's registered college list.

Users should not enter arbitrary destinations for the primary college commute flow.

Example:

```text
Select College

GRIET
CBIT
VNR VJIET
JNTUH
...
```

The list should contain only colleges supported by RideBuddy.

---

## 5.3 Search

The user selects:

* Current/pickup location
* College
* Date
* Preferred departure time/window

The user selects **Search Rides**.

RideBuddy returns compatible available rides.

---

# 6. Ride to Home

The user's home destination is represented by a **locality/area**, not an exact residential address.

Example:

```text
Home Locality:
Kukatpally
```

The system should not expose:

* House number
* Apartment number
* Exact home coordinates
* Other unnecessarily precise residential information

The user searches for rides using:

```text
Current Location → Home Locality
```

Where practical, the system may use public pickup/drop-off points within the locality.

---

# 7. Ride Results

Search results should contain relevant information about each available ride.

Example:

```text
Driver
Rating

Vehicle
Pickup area
Destination
Departure time
Available seats
Suggested contribution

[Request Ride]
```

Results should eventually be ranked according to factors such as:

* Route compatibility
* Pickup proximity
* Destination proximity
* Estimated detour
* Departure time
* Driver rating
* Contribution amount

The exact matching algorithm will be defined separately during system design.

---

# 8. Requesting a Ride

When a rider selects a ride:

```text
Available Ride
      ↓
Request Ride
      ↓
Ride Request Created
      ↓
Driver Notified
```

The ride is **not confirmed immediately**.

The driver must explicitly accept the request.

The rider should be able to see the request status.

Possible states include:

```text
REQUESTED
ACCEPTED
REJECTED
CANCELLED
```

---

# 9. Driver Approval

The driver receives a notification when a student requests a seat.

The driver can:

```text
Accept
Reject
```

If accepted:

```text
Ride Request
      ↓
Accepted
      ↓
Ride Confirmed
```

Both participants should receive confirmation.

The system must ensure that the number of confirmed passengers cannot exceed the available seats.

---

# 10. Active Ride & Live Tracking

After a ride has been confirmed and the driver starts the trip, the rider can view the driver's live location.

The rider sees:

* Driver's approximate live location
* Pickup point
* Destination
* Route/map
* Ride status

The driver should be able to start/end the active ride.

Live location must only be available to authorized participants of the active ride.

Exact location sharing should stop when the ride ends.

---

# 11. Posting a Ride

Users can access:

> **Post a Ride**

from the main application.

A user must satisfy the required verification conditions before posting.

The driver provides:

* Starting area/location
* Destination
* Date
* Departure time
* Available seats
* Suggested contribution
* Vehicle

Example:

```text
Post a Ride

From:
Miyapur

To:
GRIET

Date:
05 Oct

Departure:
8:00 AM

Available Seats:
2

Suggested Contribution:
₹40

[Post Ride]
```

---

# 12. College Verification

A student must verify their association with a college before becoming an eligible RideBuddy driver.

The verification process may involve:

* College ID
* College email
* Other approved verification mechanisms

The exact verification mechanism will be decided during system design.

Verification status should be visible internally to the system and represented appropriately in the user profile.

---

# 13. Vehicle Verification

Users posting rides must verify their vehicle.

Vehicle information may include:

* Vehicle type
* Registration number
* Make/model
* Required vehicle documentation

The system should prevent an unverified vehicle from being used for public ride posting.

A user may eventually have more than one verified vehicle, but V1 can initially support one active vehicle per user.

---

# 14. Ride Lifecycle

A ride should have a clearly defined lifecycle.

Initial conceptual state:

```text
POSTED
   ↓
REQUESTED
   ↓
ACCEPTED
   ↓
ACTIVE
   ↓
COMPLETED
```

Alternative paths:

```text
REQUESTED → REJECTED
REQUESTED → CANCELLED
ACCEPTED  → CANCELLED
```

The complete state machine and transition rules will be formally defined during backend architecture.

---

# 15. Notifications

RideBuddy should notify users about important events.

Examples:

### Rider

* Ride request submitted
* Driver accepted request
* Driver rejected request
* Driver cancelled ride
* Ride starting
* Ride completed

### Driver

* New ride request
* Rider cancelled
* Ride-related updates

The initial notification implementation can begin with in-app notifications.

Push/email notifications can be added later if required.

---

# 16. Ratings

After completing a ride, participants can rate each other.

The system should support:

* Rating
* Optional review/feedback

Ratings should contribute to a user's RideBuddy reputation.

Only completed rides should be eligible for ratings.

---

# 17. Privacy & Safety

Privacy is a core requirement.

RideBuddy should avoid exposing unnecessary personal information.

### Location privacy

Exact home addresses should never be publicly exposed.

### Live location

Live location should only be visible to authorized participants of an active ride.

### Identity

Users should be verified students.

### Vehicle

Drivers must use verified vehicles.

### Abuse prevention

The system should eventually support mechanisms for:

* Reporting users
* Reporting rides
* Blocking users
* Account suspension
* Suspicious activity detection

The detailed security model will be designed separately.

---

# 18. Core V1 Screens

The initial application should contain approximately:

```text
Authentication
├── Login
└── Registration

Main Application
├── Home / Dashboard
├── Ride to College
├── Ride to Home
├── Search Results
├── Ride Details
├── My Rides
├── Active Ride / Live Tracking
├── Post a Ride
├── My Posted Rides
├── Notifications
├── Profile
└── Verification
```

The exact frontend information architecture will be defined during frontend design.

---

# 19. Out of Scope for Initial V1

The following should not be required for the first working version:

* General public ride-hailing
* Arbitrary destinations
* Professional/commercial drivers
* Dynamic surge pricing
* Complex payment processing
* Wallet system
* Multiple cities
* Multi-college expansion infrastructure
* Advanced recommendation/ML systems
* Microservices architecture
* Complex loyalty/reward systems

These can be considered only after the core ride-pooling workflow is stable.

---

# 20. V1 Success Criteria

RideBuddy V1 is considered functionally successful when a verified student can:

1. Create an account and log in.
2. Select a default commute direction.
3. Select a college for a college commute.
4. Set/select a privacy-preserving pickup location.
5. Search for available rides.
6. View compatible rides and their details.
7. Request a ride.
8. Receive confirmation when the driver accepts.
9. View the driver's authorized live location during an active ride.
10. Complete the ride.
11. Rate the ride participant.

A verified driver must be able to:

1. Verify their college identity.
2. Verify their vehicle.
3. Create a ride.
4. Specify available seats.
5. Specify a suggested contribution.
6. Receive ride requests.
7. Accept/reject requests.
8. Start and complete a ride.
9. Share live location only during the authorized ride.

---

# 21. Initial Technology Direction

The current agreed technology direction is:

```text
Frontend
Next.js + TypeScript
        ↓
Vercel

Backend
Python + FastAPI
        ↓
Google Cloud Run

Database
PostgreSQL + PostGIS
        ↓
Supabase
```

Additional infrastructure such as Redis, background workers, object storage, notification providers and mapping services will be introduced only when justified by system requirements.

---

# 22. Product Principle

RideBuddy should be developed according to:

> **Production-oriented engineering with controlled initial scope.**

The system should be designed with proper security, maintainability, testing, observability and scalability in mind, while avoiding unnecessary complexity before the underlying requirement exists.

Initial deployment and testing will be controlled and limited rather than immediately opened to a large user base.
