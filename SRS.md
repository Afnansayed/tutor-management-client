 # Software Requirements Specification (SRS)
**Project:** Tutor Management System
**Version:** 1.0
**Date:** March 3, 2026

---

## 1. Introduction

### 1.1 Purpose
The purpose of this document is to define the software requirements for the "Tutor Management System". This system is a web-based educational platform designed to connect students with professional offline/online tutors. This SRS is intended to guide the development, testing, and project management teams in understanding the detailed features, functions, and constraints of the final product.

### 1.2 Scope
The Tutor Management System will provide a centralized marketplace where Tutors can create professional profiles, set their hourly rates, and manage their availability via a robust scheduling system. Students can browse diverse educational categories, find suitable instructors, book specialized sessions, and provide ratings/reviews. The platform will also include an Administrative portal for overseeing user activity, resolving disputes, and viewing high-level system analytics. The system will *not* process direct financial transactions (e.g., credit card processing) within this initial release, relying instead on tracking the total expected price of confirmed sessions.

### 1.3 Intended Audience and Use
This document is intended for:
*   **Developers:** To serve as a blueprint for implementing the backend APIs, database schemas, and frontend interfaces.
*   **Quality Assurance (QA) Testers:** To formulate precise test cases and validate that the system meets all mandatory requirements.
*   **Project Managers / Stakeholders:** To track development progress against the agreed-upon project boundaries and features.

### 1.4 Definitions, Acronyms, and Abbreviations
*   **SRS:** Software Requirements Specification
*   **API:** Application Programming Interface
*   **JWT:** JSON Web Token (used for secure session management)
*   **ORM:** Object-Relational Mapping (specifically Prisma)
*   **UUID:** Universally Unique Identifier
*   **CRUD:** Create, Read, Update, Delete

### 1.5 References
*   Business Requirements Document (BRD) - Internal
*   PostgreSQL 15 Documentation
*   Prisma ORM Reference Guide

---

## 2. Overall Description

### 2.1 Product Perspective
The Tutor Management System is a new, standalone web-based platform. It operates as a client-server architecture utilizing a modern Node.js/Express RESTful API backend, strictly coupled with a PostgreSQL relational database. It is intended to function independently of any legacy educational or payroll systems.

### 2.2 Product Functions
The major features of the system include:
*   **Identity & Access Control:** Secure user registration, authentication, and defined role-based authorizations (Student, Tutor, Admin).
*   **Profile & Category Ecosystem:** Creation and discovery of tutor profiles categorized by distinct subjects or skills.
*   **Availability Orchestration:** An engine allowing tutors to map out designated days and time slots.
*   **Transactional Bookings:** A reliable mechanism for a student to reserve a tutor's specific active timeslot.
*   **Quality Assurance (Reviews):** A feedback loop allowing students to rate their experiences and calculate global tutor aggregates.

### 2.3 User Classes and Characteristics
*   **Student:** The primary consumer. Typically possesses general web navigation skills. Requires a simple, intuitive interface to search, evaluate, and book tutors securely.
*   **Tutor:** The service provider. Requires slightly more technical familiarity to manage schedules, track incoming bookings, and curate their public-facing biography and rates.
*   **Administrator (Admin):** The platform operator. Possesses full trust and clearance. Responsible for data governance, moderation of reviews, ban/suspension of malicious accounts, and monitoring overall platform health.

### 2.4 Operating Environment
*   **Minimum Hardware (Server):** 1 vCPU, 1 GB RAM (Standard Cloud instance)
*   **Operating System (Server):** Linux (Ubuntu 22.04 LTS or compatible Node environment)
*   **Database Engine:** PostgreSQL (Version 15 or higher)
*   **Runtime Environment:** Node.js (Target LTS Version 20), Express.js framework
*   **Client Interface:** Any modern web browser (Chrome, Firefox, Safari, Edge) on desktop or mobile.

### 2.5 Design and Implementation Constraints
*   **Tech Stack Mandate:** Must be implemented using TypeScript, Express.js, and Prisma ORM.
*   **Authentication:** Must utilize the `better-auth` library for session and token management.
*   **Database Constraints:** Must enforce strict referential integrity (e.g., cascading deletes to ensure orphaned session tokens vanish when a user account is deleted).

### 2.6 Assumptions and Dependencies
*   **Dependency:** It is assumed that third-party video conferencing links (e.g., Zoom/Meet) for online sessions will be generated externally and pasted into the platform by the Tutor, rather than heavily integrated via internal API in Version 1.0.
*   **Assumption:** The hosting provider (e.g., Vercel) guarantees a minimum 99.9% uptime for the backend environment.

---

## 3. Specific Requirements

### 3.1 Functional Requirements

#### 3.1.1 Feature: User Registration
*   **Description:** The mechanism for a new user to create an account as either a Student or a Tutor.
*   **Priority:** High
*   **Stimulus/Response:** 
    *   *Stimulus:* A guest submits the registration form with valid Name, Email, Password, and desired Role.
    *   *Response:* The system creates a new user record, hashes the password, and provisions an active session.
*   **Functional Requirements:**
    *   **FR-1.1.1:** The system shall provide a registration interface requiring Name, Email, Password, and Role selection.
    *   **FR-1.1.2:** The system shall reject the registration and display an error if the provided email is already registered.
    *   **FR-1.1.3:** The system shall enforce a minimum password complexity (e.g., minimum 8 characters).
    *   **FR-1.1.4:** Upon successful registration, the system shall assign the default status of `ACTIVE`.

#### 3.1.2 Feature: User Login & Session Management
*   **Description:** Allows registered users to authenticate and receive an authorized session.
*   **Priority:** High
*   **Stimulus/Response:** 
    *   *Stimulus:* User submits login credentials. 
    *   *Response:* System validates credentials and issues a secure access token or session cookie.
*   **Functional Requirements:**
    *   **FR-1.2.1:** The system shall allow users to submit Email and Password credentials.
    *   **FR-1.2.2:** The system shall validate the provided credentials against the encrypted record in the database.
    *   **FR-1.2.3:** If credentials belong to an account mapped to a `BANNED` status, the system shall deny access and display a security notification.
    *   **FR-1.2.4:** Upon successful validation, the system shall automatically generate a session Token linked to a strict `expiresAt` timestamp.
    *   **FR-1.2.5:** The system shall permit the user to explicitly invoke a "Logout" action, which shall instantly invalidate their active session.

#### 3.1.3 Feature: Tutor Profile Management
*   **Description:** Enables tutors to advertise their personalized services.
*   **Priority:** High
*   **Stimulus/Response:**
    *   *Stimulus:* An authenticated Tutor inputs their bio and hourly rate.
    *   *Response:* The system saves the configuration mapped to their specific User ID.
*   **Functional Requirements:**
    *   **FR-2.1.1:** The system shall restrict the creation and modification of a `TutorProfile` exclusively to users with the `TUTOR` role.
    *   **FR-2.1.2:** The system shall securely store the tutor's textual "bio", "hourly_rate" (as a floating-point decimal), and an optional URL for a "profile_picture".
    *   **FR-2.1.3:** The system shall allow a Tutor to accurately map their profile to one or more predefined Educational `Categories`.

#### 3.1.4 Feature: Schedule Orchestration
*   **Description:** Tutors declare their recurring availability slots.
*   **Priority:** High
*   **Stimulus/Response:**
    *   *Stimulus:* A Tutor selects a Day of the Week, and inputs a Start and End time.
    *   *Response:* The system writes a schedule interval available for future bookings.
*   **Functional Requirements:**
    *   **FR-3.1.1:** The system shall dictate that Tutors must declare slots partitioned strictly by `DaysOfWeek` (SUNDAY through SATURDAY).
    *   **FR-3.1.2:** The system shall require specific textual or temporal inputs for `start_time` and `end_time` (e.g., "09:00" to "11:00").
    *   **FR-3.1.3:** The system shall expose a toggle allowing the Tutor to mark slots as globally `isActive` or inactive.
    *   **FR-3.1.4:** The system shall automatically flag an active slot as `isAvailable = false` once a student commits to a confirmed booking against it.

#### 3.1.5 Feature: Appointment Booking
*   **Description:** The process where a Student reserves an available slot of an active Tutor.
*   **Priority:** High
*   **Stimulus/Response:**
    *   *Stimulus:* A Student selects a Tutor's available schedule slot and confirms a booking date.
    *   *Response:* The system calculates the price, registers the transaction, and marks the slot as unavailable.
*   **Functional Requirements:**
    *   **FR-4.1.1:** The system shall permit only users with the `STUDENT` role to initiate a booking payload.
    *   **FR-4.1.2:** The system shall automatically compute the `total_price` of the booking by referencing the target Tutor's currently stored `hourly_rate`.
    *   **FR-4.1.3:** The system shall automatically generate and affix a universally unique `trakking_code` to the finalized booking.
    *   **FR-4.1.4:** The system shall initialize new bookings with a default status of `CONFIRMED`.
    *   **FR-4.1.5:** The system shall expose functionality allowing either the Tutor or the Student to shift the status to `CANCELLED`, or for the Tutor to later mark it as `COMPLETED`.

#### 3.1.6 Feature: Review & Rating Module
*   **Description:** Students evaluate completed sessions.
*   **Priority:** Medium
*   **Stimulus/Response:**
    *   *Stimulus:* A Student submits a 1-5 rating and a text comment for a specific past booking.
    *   *Response:* The system saves the review and recalculates the Tutor's global rating average.
*   **Functional Requirements:**
    *   **FR-5.1.1:** The system shall rigidly enforce a one-to-one constraint, ensuring a Student can write only one distinct `Review` per unique `Booking ID`.
    *   **FR-5.1.2:** The system shall necessitate that the review contains a numerical `rating` variable.
    *   **FR-5.1.3:** Upon submission of a new review, the system shall seamlessly aggregate and update the target Tutor's `total_reviews` integer and `average_rating` float metric.

#### 3.1.7 Feature: Administrative Governance
*   **Description:** Privileged access to mediate the platform.
*   **Priority:** High
*   **Stimulus/Response:**
    *   *Stimulus:* An Admin accesses the management dashboard to modify categories or user states.
    *   *Response:* The system applies global updates to taxonomies or access laws.
*   **Functional Requirements:**
    *   **FR-6.1.1:** The system shall provide endpoints explicitly locked to the `ADMIN` role.
    *   **FR-6.1.2:** The system shall afford the Admin the capability to Create, Read, Update, and Delete system-wide Subject `Categories`.
    *   **FR-6.1.3:** The system shall afford the Admin the ability to forcefully manipulate a review's `isApproved` Boolean (e.g., removing abusive feedback from public view).

### 3.2 Non-Functional Requirements (Quality Attributes)

*   **Performance:** 
    *   The API shall parse, execute, and respond to synchronous database queries (e.g., fetching a Tutor's profile) in under 300 milliseconds at the 95th percentile under normal server load.
*   **Security:** 
    *   All passwords shall be irreversibly hashed utilizing industry-standard salt and cryptographic algorithms before insertion into the database.
    *   The API layer shall deny unauthenticated traffic from accessing any protected REST routes with an HTTP 401 Unauthorized status code.
*   **Reliability:** 
    *   The system shall ensure transactional safety during Bookings to prevent "double-booking" the same slot simultaneously (handled via ACID properties of PostgreSQL).
*   **Maintainability:** 
    *   The backend codebase shall be fully strictly typed utilizing TypeScript to mitigate runtime errors and encourage effortless developer onboarding.
*   **Usability:**
    *   The system API shall return consistent, predictably formatted JSON payloads for both success scenarios and localized Error handling across all routes.

### 3.3 External Interface Requirements

*   **User Interfaces:** 
    *   While the frontend is decoupled, the API shall return intuitive, human-readable error messages alongside HTTP response codes to ensure the UI can display robust diagnostic alerts to the end-user (e.g., "This schedule slot is no longer available").
*   **Software Interfaces:** 
    *   The system shall interface consistently with the PostgreSQL database over standard TCP/IP protocols mapped strictly by the Prisma ORM schema definitions.
*   **Communications Interfaces:** 
    *   All interactions between the Client (Browser/Mobile) and the System (Backend API) shall occur exclusively over HTTPS (encrypted TLS communication).
    *   Data payloads between client and server shall exclusively utilize `application/json` formatting.

---

## 4. Supporting Information

### 4.1 Traceability Summary
*All stated Functional Requirements (FR-x.y.z) trace directly back to the database schema constructs located in the `/prisma/schema/*.prisma` directory, dictating precisely how the relational data enforces the business logic.*

### 4.2 Analysis Models
*(Conceptual mapping for development context)*
*   **ERD Context:** `User` -> 1-to-Many -> `Bookings`. `User(Tutor)` -> 1-to-1 -> `TutorProfile`. `TutorProfile` -> 1-to-Many -> `TutorSchedule`. `Booking` -> 1-to-1 -> `Review`.

### 4.3 Approval and Sign-off
*This document requires formal approval from the Project Manager and Lead Architect before execution transitions to the final implementation and rigorous QA phases.*

*   __Approved By: Afnan Sayed Razin
*   __Date: 03.03.2026
*   __Signature: Afnan sayed
