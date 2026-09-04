# RaceDay – API Endpoint Plan

This document lists every API endpoint the RaceDay system will expose, planned before any code is written in Part 2. It covers Authentication, User Profile, Events, Categories, Event Enrolments, and Results, as required by the POE brief.

Roles: **None** = public/no login required · **Any** = any logged-in user · **Organiser** / **Participant** = restricted to that role.

## Authentication

| HTTP Method | Route | Description | Role Required | Request Body | Expected Response |
|---|---|---|---|---|---|
| POST | /api/auth/register | Creates a new user account as either a Participant or an Organiser. | None (public) | `{ fullName, email, password, phoneNumber, role }` | 201 Created – returns new user id and role. 400 Bad Request – validation failed. 409 Conflict – email already registered. |
| POST | /api/auth/login | Authenticates a user and issues a JWT access token. | None (public) | `{ email, password }` | 200 OK – returns JWT token, userId, role. 401 Unauthorized – invalid credentials. |

## User Profile

| HTTP Method | Route | Description | Role Required | Request Body | Expected Response |
|---|---|---|---|---|---|
| GET | /api/users/{id} | Retrieves a single user's profile details. | Any (own profile) or Organiser | None | 200 OK – user profile object. 404 Not Found – user does not exist. |
| PUT | /api/users/{id} | Updates the logged-in user's own profile information. | Any (own profile only) | `{ fullName, phoneNumber }` | 200 OK – updated profile. 400 Bad Request – validation failed. 403 Forbidden – attempting to edit another user's profile. |

## Events

| HTTP Method | Route | Description | Role Required | Request Body | Expected Response |
|---|---|---|---|---|---|
| GET | /api/events | Lists all upcoming events so Participants can browse them. | None (public) | None | 200 OK – array of event summaries. |
| GET | /api/events/{id} | Retrieves full details for a single event, including its categories. | None (public) | None | 200 OK – event details with nested categories. 404 Not Found – event does not exist. |
| POST | /api/events | Creates a new event. | Organiser | `{ eventName, description, eventDate, location, routeInfo }` | 201 Created – new event id. 400 Bad Request – validation failed. |
| PUT | /api/events/{id} | Updates an existing event owned by the logged-in Organiser. | Organiser (owner only) | `{ eventName, description, eventDate, location, routeInfo }` | 200 OK – updated event. 403 Forbidden – not the event owner. 404 Not Found. |
| DELETE | /api/events/{id} | Deletes an event owned by the logged-in Organiser. | Organiser (owner only) | None | 200 OK – confirmation. 403 Forbidden – not the event owner. 404 Not Found. |

## Categories

| HTTP Method | Route | Description | Role Required | Request Body | Expected Response |
|---|---|---|---|---|---|
| GET | /api/events/{eventId}/categories | Lists all categories (e.g. 5km, 10km) for a specific event. | None (public) | None | 200 OK – array of categories. 404 Not Found – event does not exist. |
| POST | /api/events/{eventId}/categories | Adds a new category to an event. | Organiser (owner only) | `{ categoryName, distanceKm, maxParticipants, entryFee }` | 201 Created – new category id. 403 Forbidden – not the event owner. |
| PUT | /api/categories/{id} | Updates an existing category. | Organiser (owner only) | `{ categoryName, distanceKm, maxParticipants, entryFee }` | 200 OK – updated category. 403 Forbidden. 404 Not Found. |
| DELETE | /api/categories/{id} | Removes a category from an event. | Organiser (owner only) | None | 200 OK – confirmation. 403 Forbidden. 404 Not Found. |

## Event Enrolments

| HTTP Method | Route | Description | Role Required | Request Body | Expected Response |
|---|---|---|---|---|---|
| POST | /api/enrolments | Enrols the logged-in Participant into a chosen category. | Participant | `{ categoryId }` | 201 Created – enrolment record with race number. 400 Bad Request – category full or invalid. 409 Conflict – already enrolled in this category. |
| GET | /api/enrolments/mine | Lists the logged-in Participant's own enrolments. | Participant | None | 200 OK – array of the participant's enrolments. |
| GET | /api/events/{eventId}/enrolments | Lists all enrolments for a specific event (for the Organiser managing it). | Organiser (owner only) | None | 200 OK – array of enrolments with participant details. 403 Forbidden – not the event owner. |
| DELETE | /api/enrolments/{id} | Cancels a Participant's own enrolment. | Participant (owner only) | None | 200 OK – confirmation. 403 Forbidden. 404 Not Found. |

## Results

| HTTP Method | Route | Description | Role Required | Request Body | Expected Response |
|---|---|---|---|---|---|
| POST | /api/results | Captures a finishing result for a participant's enrolment. | Organiser | `{ enrolmentId, finishTime, overallPosition, categoryPosition, completionStatus }` | 201 Created – new result record. 400 Bad Request – validation failed. 404 Not Found – enrolment does not exist. 409 Conflict – result already captured for this enrolment. |
| GET | /api/results/mine | Retrieves the logged-in Participant's personal performance history. | Participant | None | 200 OK – array of the participant's past results. |
| GET | /api/events/{eventId}/results | Retrieves all results for an event (leaderboard view). | None (public) | None | 200 OK – array of results ordered by position. 404 Not Found – event does not exist. |
| PUT | /api/results/{id} | Corrects a previously captured result. | Organiser | `{ finishTime, overallPosition, categoryPosition, completionStatus }` | 200 OK – updated result. 403 Forbidden. 404 Not Found. |

---
*Plan produced for Part 1 of the RaceDay POE (PROG6212). The implemented API in Part 2 will match this plan; any deviations will be explained in the README.*
