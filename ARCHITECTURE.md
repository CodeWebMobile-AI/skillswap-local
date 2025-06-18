# skillswap-local Architecture

## Overview
To connect individuals within a community for reciprocal skill-sharing, fostering local learning and resourcefulness.

### Problem Statement
Many people possess valuable skills they're willing to share, but lack a platform to connect with others seeking those skills locally. Traditional learning platforms are often expensive and impersonal.

### Target Audience
Community residents, hobbyists, retirees, and individuals seeking affordable learning opportunities or wanting to share their expertise.

### Core Entities
Users, Skills (Offered), Skills (Requested), Locations (Proximity search), Reviews, Messages, Swaps (Proposed/Confirmed/Completed)

## Design System

### Typography
- **Font**: Nunito Sans
- **Font Stack**: `'Nunito Sans', sans-serif`
- **Google Fonts**: https://fonts.google.com/specimen/Nunito+Sans
- **Rationale**: Nunito Sans is a well-balanced, highly readable sans-serif font suitable for both body text and headers, enhancing the user experience across the application. Its modern and clean look complements the SkillSwap Local's theme. [Accessibility Best Practices - Font Size & Readability](https://www.w3.org/WAI/WCAG21/Understanding/resize-text)

### Color Palette
A complementary color palette built around a primary teal, conveying trust and community. Secondary colors provide contrast and visual hierarchy, ensuring accessibility and a professional aesthetic.  [Understanding Color Contrast - WebAIM](https://webaim.org/articles/contrast/)

- **Teal**: `#26A69A` - Main brand color, used for headers and primary actions.
- **Light Grey**: `#ECEFF1` - Supporting color for secondary elements like card backgrounds.
- **Amber**: `#FFB300` - Used for call-to-action buttons and highlights to draw attention.
- **Dark Grey**: `#212121` - Primary text color for high contrast and readability.
- **White**: `#FFFFFF` - Main background color for content areas to maximize readability.
- **Light Grey Border**: `#CFD8DC` - For card borders, dividers, and form inputs to provide subtle separation.
- **Green**: `#4CAF50` - For success messages and confirmation to visually reinforce positive actions.
- **Orange**: `#FF9800` - For warnings and non-critical alerts to alert users to potential issues.
- **Red**: `#F44336` - For error messages and destructive actions to clearly indicate critical errors.

## System Architecture

### 1. Core Components & Rationale
The SkillSwap Local application adopts a layered architecture, comprising:

*   **Frontend (React/Inertia.js/TypeScript):** Responsible for the user interface and interactions. Inertia.js allows us to use React components with Laravel routing, reducing complexity. TypeScript provides static typing for improved code quality and maintainability. [Inertia.js Documentation](https://inertiajs.com/)
*   **Backend (Laravel):** Handles business logic, data access, and API endpoints. Laravel's robust features and elegant syntax provide a solid foundation.
*   **Database (PostgreSQL):** Stores application data. PostgreSQL is chosen for its reliability, scalability, and support for advanced data types like JSON and geospatial data.
*   **Real-time Server (laravel-echo-server & Redis):** Facilitates real-time communication using WebSockets. Redis acts as a message broker for broadcasting events to connected clients. [Laravel Echo Documentation](https://laravel.com/docs/10.x/broadcasting)
*   **API Gateway (Optional, for future scaling):** In the future, a dedicated API Gateway (e.g., Kong or Tyk) can be introduced to manage authentication, rate limiting, and routing of API requests, improving security and scalability. [Kong API Gateway Documentation](https://konghq.com/)

### 2. Database Schema Design
The database schema is designed to support the core entities and their relationships:

*   **users:** `id`, `name`, `email`, `password`, `location (POINT)`, `latitude`, `longitude`, `profile_image`, `bio`, `created_at`, `updated_at` (Uses Laravel's built-in `users` migration. `location` is a `POINT` type for geospatial queries.)
*   **skills:** `id`, `name`, `created_at`, `updated_at` (A simple table for managing skills taxonomy.)
*   **user_skills:** `id`, `user_id (FK)`, `skill_id (FK)`, `type ENUM('offered', 'requested')`, `description`, `created_at`, `updated_at` (Connects users and skills, specifying whether they offer or request the skill. Includes a description to provide context.)
*   **reviews:** `id`, `reviewer_id (FK to users)`, `reviewee_id (FK to users)`, `rating (INT)`, `comment`, `created_at`, `updated_at` (Stores reviews between users.)
*   **messages:** `id`, `sender_id (FK to users)`, `receiver_id (FK to users)`, `content`, `created_at`, `updated_at`, `read_at (nullable timestamp)` (Stores messages between users. `read_at` tracks message read status.)
*   **swaps:** `id`, `offerer_id (FK to users)`, `requester_id (FK to users)`, `offered_skill_id (FK to user_skills)`, `requested_skill_id (FK to user_skills)`, `status ENUM('proposed', 'confirmed', 'completed', 'cancelled')`, `start_time (timestamp)`, `end_time (timestamp)`, `location_id (FK to locations)`, `description`, `created_at`, `updated_at` (Represents a skill-swap event. `status` tracks the swap's lifecycle.)
*   **locations:** `id`, `name`, `address`, `latitude`, `longitude`, `created_at`, `updated_at` (Stores locations that swaps may occur at.)
*   **password_reset_tokens:** `email`, `token`, `created_at` (standard Laravel password reset table)

Rationale: This schema prioritizes clarity and efficiency. Foreign keys enforce relationships and ensure data integrity. ENUMs are used for predefined states (e.g., `swap.status`) to improve data consistency. Geospatial types are used for location-based queries. [PostgreSQL Geospatial Data Types](https://postgis.net/docs/)


### 3. API Design & Key Endpoints
The API follows a RESTful architecture with versioning (e.g., `/api/v1/*`). Authentication is handled using Laravel Sanctum, which provides lightweight token-based authentication. [Laravel Sanctum Documentation](https://laravel.com/docs/10.x/sanctum)

Key Endpoints:

*   `/api/v1/auth/register` (POST): Registers a new user.
*   `/api/v1/auth/login` (POST): Logs in an existing user, returns API token.
*   `/api/v1/auth/logout` (POST): Logs out the current user, invalidates API token.
*   `/api/v1/users` (GET): Lists users (with optional filters for proximity, skills).
*   `/api/v1/users/{id}` (GET): Retrieves a specific user's profile.
*   `/api/v1/users/{id}` (PUT/PATCH): Updates a user's profile (requires authentication).
*   `/api/v1/users/{id}/skills` (GET): Retrieves skills offered and requested by a user.
*   `/api/v1/skills` (GET): Lists all available skills.
*   `/api/v1/user-skills` (POST): Creates a new user skill (requires authentication).
*   `/api/v1/user-skills/{id}` (PUT/PATCH): Updates a user skill (requires authentication).
*   `/api/v1/user-skills/{id}` (DELETE): Deletes a user skill (requires authentication).
*   `/api/v1/reviews` (POST): Creates a new review (requires authentication).
*   `/api/v1/reviews/{id}` (GET): Retrieves a specific review.
*   `/api/v1/messages` (GET): Lists messages for the authenticated user.
*   `/api/v1/messages` (POST): Sends a new message (requires authentication).
*   `/api/v1/messages/{id}` (GET): Retrieves a specific message.
*   `/api/v1/swaps` (GET): Lists swaps for the authenticated user (as offerer or requester).
*   `/api/v1/swaps` (POST): Creates a new swap proposal (requires authentication).
*   `/api/v1/swaps/{id}` (GET): Retrieves a specific swap.
*   `/api/v1/swaps/{id}` (PUT/PATCH): Updates a swap's status (requires authentication).
*   `/api/v1/locations` (GET): lists nearby locations
*   `/api/v1/locations` (POST): Creates a new location

Rationale: This API design follows RESTful principles, using standard HTTP methods and resource naming conventions. Versioning allows for future API changes without breaking existing clients. Laravel Sanctum ensures secure authentication. Rate limiting will be implemented at the API Gateway level (if implemented) to prevent abuse.

### 4. Frontend Structure (TypeScript)
The frontend is built using React with Inertia.js and TypeScript, structured as follows:

*   **`src/`:** Root directory for all source code.
    *   **`Components/`:** Reusable UI components (e.g., Button, Input, Card, SkillCard).
    *   **`Layouts/`:** Application layouts (e.g., Authenticated, Guest).
    *   **`Pages/`:** Page-level components, each corresponding to a route (e.g., Pages/Profile/Edit.tsx, Pages/Skills/Index.tsx).
    *   **`Hooks/`:** Custom React hooks for managing state and side effects (e.g., useGeolocation, useDebounce).
    *   **`Types/`:** TypeScript type definitions for data models (e.g., User, Skill, Swap). These types mirror the backend data models.
    *   **`Services/`:** API service functions for making requests to the backend (e.g., UserService, SkillService).
    *   **`Utils/`:** Utility functions (e.g., date formatting, string manipulation).
    *   **`context/`:** Context providers for managing global state (e.g., AuthContext).
    *   **`App.tsx`:** Main application entry point.

*   **`resources/js/`:** contains frontend logic, including `app.tsx`
*   **`resources/ts/`:** contains type definitions
*   **`resources/views/`:** Blade templates used by Inertia.js (primarily a root template).

Rationale: This structure promotes code organization, reusability, and maintainability. TypeScript enhances code quality and helps catch errors early. Using Inertia.js enables leveraging React components within a Laravel application seamlessly. [React Component Structure - Official Documentation](https://react.dev/learn/thinking-in-react)

### 5. Real-time & Events Architecture
Real-time functionality (e.g., new messages, swap status updates) is implemented using WebSockets with `laravel-echo-server` and Redis:

*   **Backend (Laravel):** Laravel events are used to trigger real-time updates. These events are broadcast using Redis. [Laravel Broadcasting Documentation](https://laravel.com/docs/10.x/broadcasting)
*   **Redis:** Redis acts as a message broker, publishing events to subscribed clients.
*   **`laravel-echo-server`:** A Node.js server that handles WebSocket connections and broadcasts events received from Redis to connected clients.
*   **Frontend (React):** The frontend uses `laravel-echo` (a JavaScript library) to subscribe to channels and listen for events. When an event is received, the frontend updates the UI accordingly. [laravel-echo Documentation](https://github.com/tlaverdure/laravel-echo)

Channels:

*   **Private Channels:** Used for user-specific events (e.g., new messages) - `users.{user_id}`
*   **Presence Channels:** Could be used to track online users for chat feature - `chat`

Rationale: This architecture provides a scalable and reliable way to implement real-time features. Private channels ensure that only authorized users can receive certain events. Laravel Echo simplifies the process of subscribing to channels and handling events on the frontend. [Pub/Sub Pattern - Microsoft Documentation](https://learn.microsoft.com/en-us/azure/architecture/patterns/pub-sub)

### 6. Authentication & Authorization Flow
Authentication and authorization are handled using Laravel Sanctum:

1.  **Registration:** User submits registration form. Backend validates data and creates a new user record. An API token is generated and returned to the frontend.
2.  **Login:** User submits login form. Backend authenticates credentials and returns an API token.
3.  **Subsequent Requests:** The frontend includes the API token in the `Authorization` header (using the `Bearer` scheme) for all subsequent requests.
4.  **Authentication Middleware:** Laravel Sanctum's authentication middleware verifies the token on each request. If the token is valid, the user is authenticated; otherwise, a 401 Unauthorized error is returned.
5.  **Authorization:** Access control is implemented using Laravel's built-in authorization features (gates and policies). These policies determine whether a user is authorized to perform specific actions on specific resources.  [Laravel Authorization Documentation](https://laravel.com/docs/10.x/authorization)

Rationale: Laravel Sanctum provides a simple and secure way to implement token-based authentication for APIs. Authorization policies ensure that users can only access resources they are authorized to access, protecting sensitive data and functionality.  [OWASP Authentication Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html)

### 7. Deployment & Scalability Plan
The application can be deployed on various platforms (e.g., AWS, DigitalOcean, Heroku). A typical deployment setup involves:

*   **Web Server (Nginx or Apache):** Serves the frontend assets and proxies requests to the PHP application.
*   **PHP Application Server (PHP-FPM):** Executes the Laravel application.
*   **Database Server (PostgreSQL):** Stores application data.
*   **Redis Server:** Acts as a message broker for real-time events.
*   **laravel-echo-server:** Runs the websocket server.

Scalability:

*   **Horizontal Scaling:** The web server and PHP application server can be scaled horizontally by adding more instances behind a load balancer. [Load Balancing - Nginx Documentation](https://docs.nginx.com/nginx/admin-guide/load-balancer/)
*   **Database Scaling:** PostgreSQL can be scaled using replication and clustering.  [PostgreSQL Replication Documentation](https://www.postgresql.org/docs/current/different-replication-solutions.html)
*   **Redis Scaling:** Redis can be scaled using clustering. [Redis Clustering Documentation](https://redis.io/docs/concepts/clustering/)
*   **Caching:** Implement caching at various levels (e.g., browser caching, server-side caching using Redis) to improve performance. [Laravel Caching Documentation](https://laravel.com/docs/10.x/cache)
*   **Queue System:** Use Laravel's queue system to offload long-running tasks (e.g., sending emails, processing images) to background workers. [Laravel Queues Documentation](https://laravel.com/docs/10.x/queues)

Rationale: This deployment plan provides a scalable and resilient infrastructure. Horizontal scaling allows the application to handle increased traffic. Caching and queue systems improve performance and responsiveness. Using a load balancer enables distributing traffic across multiple servers. [Scalability Best Practices - Microsoft Azure](https://learn.microsoft.com/en-us/azure/architecture/best-practices/scalability)

### 8. Testing Strategy
A comprehensive testing strategy is crucial for ensuring the quality and reliability of the application:

*   **Backend (Laravel):**
    *   **Unit Tests (Pest):** Test individual units of code (e.g., models, services, controllers) in isolation. [Pest PHP Documentation](https://pestphp.com/)
    *   **Feature Tests (Pest):** Test the application's features from an end-to-end perspective, simulating user interactions.
*   **Frontend (React):**
    *   **Unit Tests (Vitest):** Test individual components and functions in isolation. [Vitest Documentation](https://vitest.dev/)
    *   **Component Tests (React Testing Library):** Test how components render and behave in response to user interactions. [React Testing Library Documentation](https://testing-library.com/docs/react-testing-library/intro/)
    *   **End-to-End Tests (Cypress or Playwright):** Test the entire application workflow from the user's perspective. (Optional, for increased confidence.) [Cypress Documentation](https://www.cypress.io/)

*   **Test-Driven Development (TDD):** Encourage developers to write tests before writing code.
*   **Continuous Integration (CI):** Integrate testing into the CI/CD pipeline to automatically run tests on every code change.  [Continuous Integration - Martin Fowler](https://martinfowler.com/articles/continuousIntegration.html)

Rationale: This testing strategy covers all aspects of the application, from individual units of code to end-to-end workflows. TDD helps ensure that code is testable and that tests are written proactively. CI/CD automates the testing process and provides continuous feedback on code quality.

### 9. Security & Hardening Plan
Security is a top priority. The following measures will be implemented to address OWASP Top 10 vulnerabilities:

*   **Input Validation & Sanitization:** Validate and sanitize all user inputs to prevent injection attacks (e.g., SQL injection, XSS). [OWASP Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
*   **Output Encoding:** Encode all output to prevent XSS attacks.
*   **Authentication & Authorization:** Implement strong authentication and authorization mechanisms to protect sensitive data and functionality. Use Laravel Sanctum.
*   **Password Management:** Use bcrypt hashing for passwords. Enforce strong password policies.
*   **Session Management:** Use secure session management techniques to prevent session hijacking.
*   **Error Handling:** Implement proper error handling to prevent information leakage.
*   **Regular Security Audits:** Conduct regular security audits to identify and address vulnerabilities.
*   **Dependency Management:** Keep all dependencies up-to-date to patch security vulnerabilities. Use tools like `composer audit` and `npm audit` to identify vulnerable dependencies. [OWASP Dependency Check](https://owasp.org/www-project-dependency-check/)
*   **Rate Limiting:** Implement rate limiting to prevent brute-force attacks and denial-of-service attacks. Implemented at the API gateway level (if used).
*   **CSRF Protection:** Use Laravel's built-in CSRF protection to prevent cross-site request forgery attacks. [OWASP CSRF Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
*   **Security Headers:** Set appropriate security headers (e.g., Content-Security-Policy, X-Frame-Options) to prevent various attacks.  [OWASP Secure Headers Project](https://owasp.org/www-project-secure-headers/)
*   **Regular Security Scans:** Use tools like Snyk to perform automated security scans of the application and its dependencies. [Snyk](https://snyk.io/)

Rationale: This security hardening plan addresses the most common web application vulnerabilities. Regular security audits and dependency updates help ensure that the application remains secure over time. [OWASP Top 10](https://owasp.org/top10/)

### 10. Logging & Observability
Comprehensive logging and monitoring are essential for identifying and resolving issues in production.

*   **Logging:**
    *   **Application Logs:** Log important events and errors using Laravel's built-in logging facilities. Configure the `stderr` channel to output logs to standard error, which is easily collected by most monitoring tools.  [Laravel Logging Documentation](https://laravel.com/docs/10.x/logging)
    *   **Access Logs:** Configure the web server (Nginx or Apache) to log all incoming requests.
*   **Monitoring:**
    *   **Application Performance Monitoring (APM):** Use an APM tool (e.g., New Relic, DataDog, Sentry) to monitor the performance of the application. These tools provide insights into response times, error rates, and resource usage. [New Relic Documentation](https://newrelic.com/)
    *   **System Monitoring:** Monitor the health and performance of the servers using a system monitoring tool (e.g., Prometheus, Grafana). [Prometheus Documentation](https://prometheus.io/)
    *   **Alerting:** Configure alerts to notify developers when critical errors occur or when performance degrades.  [PagerDuty Documentation](https://www.pagerduty.com/)

*   **Centralized Logging:** Aggregate logs from all servers into a central logging system (e.g., ELK stack) for easier analysis. [ELK Stack](https://www.elastic.co/elk-stack/)

Rationale: Comprehensive logging and monitoring provide the visibility needed to identify and resolve issues quickly. APM tools provide detailed insights into application performance. System monitoring tools track the health of the servers. Centralized logging enables efficient log analysis and troubleshooting. [Observability - Honeycomb](https://www.honeycomb.io/)

## Feature Implementation Roadmap

### 1. Profile creation with skill listings (both offered and requested).
Users can create profiles with personal information and list skills they offer and skills they request.

**Database Changes:**
- Add 'profile_image', 'bio', 'latitude', 'longitude' columns to the 'users' table.
- Create 'user_skills' table with columns: id, user_id (FK), skill_id (FK), type ENUM('offered', 'requested'), description, created_at, updated_at.

**Backend Components:**
- UserService: Methods for creating, updating, and retrieving user profiles and skills.
- UserController: Endpoints for managing user profiles and skills.
- SkillService: Methods for creating and listing all available skills.
- SkillController: Endpoints for managing available skills.
- UserSkillController: Endpoints for managing user skills (add, update, delete).

**Frontend Components:**
- Pages/Profile/Create.tsx: Form for creating a new user profile.
- Pages/Profile/Edit.tsx: Form for editing an existing user profile.
- Components/SkillList.tsx: Component for displaying a list of skills (offered or requested).
- Components/SkillForm.tsx: Component for adding/editing user skills.
- Types/User.ts: Typescript Type definitions for user data.
- Types/Skill.ts: Typescript Type definitions for skill data.
- Types/UserSkill.ts: Typescript Type definitions for user skill data.

**API Endpoints:**
- GET /api/v1/users/{id}
- PUT /api/v1/users/{id}
- GET /api/v1/skills
- POST /api/v1/user-skills
- PUT /api/v1/user-skills/{id}
- DELETE /api/v1/user-skills/{id}

**Testing Requirements:**
- Unit tests for UserService and SkillService to ensure data is handled correctly.
- Feature tests for UserController to ensure profile creation and updates are working as expected.
- Component tests for SkillList.tsx and SkillForm.tsx to ensure proper rendering and user interaction.


### 2. Proximity-based search for users offering or requesting specific skills.
Users can search for other users within a specified radius who offer or request specific skills.

**Database Changes:**
- Add a geospatial index on the 'location' column in the 'users' table for efficient proximity searches.
- Ensure that location data (latitude and longitude) is accurately stored in the 'users' table.

**Backend Components:**
- UserService: Implement a method to search for users within a radius based on skill and location.
- UserController: Add a 'search' endpoint with query parameters for skill, latitude, longitude, and radius.
- SkillService: Method to retrieve and sort skills by relevance.
- LocationService: Class to handle getting the actual longitude and latitude.

**Frontend Components:**
- Pages/Search/Index.tsx: Search form with skill selection, location input, and radius selection.
- Components/UserCard.tsx: Component for displaying search results (user profiles with relevant information).
- Hooks/useGeolocation.ts: Custom hook to obtain the user's current location.
- Types/UserSearchResult.ts: Typescript Type definitions for user search results.

**API Endpoints:**
- GET /api/v1/users/search?skill={skill_id}&latitude={latitude}&longitude={longitude}&radius={radius}

**Testing Requirements:**
- Unit tests for UserService's search method to ensure accurate results based on location and skill.
- Feature tests for the /api/v1/users/search endpoint to verify correct parameter handling and response.
- Component tests for UserCard.tsx to ensure proper data display.
- Integration tests for the entire search workflow.


### 3. Secure messaging system for coordinating skill-swaps.
Users can send and receive messages to coordinate skill-swap details.

**Database Changes:**
- Create 'messages' table with columns: id, sender_id (FK to users), receiver_id (FK to users), content, created_at, updated_at, read_at (nullable timestamp).

**Backend Components:**
- MessageService: Methods for sending, retrieving, and marking messages as read.
- MessageController: Endpoints for managing messages.
- Events/NewMessage.php: Laravel event to broadcast new messages in real-time.

**Frontend Components:**
- Pages/Messages/Index.tsx: Displays a list of conversations.
- Pages/Messages/Show.tsx: Displays a conversation and allows sending new messages.
- Components/Message.tsx: Component for displaying a single message.
- Types/Message.ts: Typescript Type definitions for message data.

**API Endpoints:**
- GET /api/v1/messages
- POST /api/v1/messages
- GET /api/v1/messages/{id}

**Real-time Events:**
- A 'NewMessage' event broadcast on a private channel: 'users.{receiver_id}'.

**Testing Requirements:**
- Unit tests for MessageService to ensure messages are sent and retrieved correctly.
- Feature tests for MessageController to verify message sending and retrieval endpoints.
- Component tests for Message.tsx to ensure proper display of messages.
- Integration tests to verify real-time message broadcasting.


### 4. Review and rating system to build trust and reputation.
Users can leave reviews and ratings for other users after a skill-swap has been completed.

**Database Changes:**
- Create 'reviews' table with columns: id, reviewer_id (FK to users), reviewee_id (FK to users), rating (INT), comment, created_at, updated_at.

**Backend Components:**
- ReviewService: Methods for creating, retrieving, and calculating average ratings.
- ReviewController: Endpoints for managing reviews.
- UserService: Method to retrieve a user's average rating and review count.

**Frontend Components:**
- Components/ReviewForm.tsx: Form for submitting a review.
- Components/ReviewList.tsx: Component for displaying a list of reviews.
- Components/RatingDisplay.tsx: Component for displaying a user's average rating.
- Types/Review.ts: Typescript Type definitions for review data.

**API Endpoints:**
- POST /api/v1/reviews
- GET /api/v1/reviews/{id}

**Testing Requirements:**
- Unit tests for ReviewService to ensure reviews are created and ratings are calculated correctly.
- Feature tests for ReviewController to verify review creation and retrieval endpoints.
- Component tests for ReviewForm.tsx and ReviewList.tsx to ensure proper display of reviews.
- Integration tests to ensure reviews are correctly associated with users and swaps.


### 5. Calendar integration for scheduling skill-swap sessions.
Users can schedule skill-swap sessions using a calendar integration, suggesting dates and times.

**Database Changes:**
- Add 'start_time' and 'end_time' columns (timestamp) to the 'swaps' table.
- Update existing 'swaps' table to include location_id for identifying the location where the swap occurs

**Backend Components:**
- SwapService: Methods for managing swap scheduling and time availability.
- SwapController: Endpoints for managing swap scheduling.
- CalendarService: Optional integration with external calendar services (e.g., Google Calendar).

**Frontend Components:**
- Components/Calendar.tsx: Calendar component for selecting dates and times.
- Pages/Swaps/Create.tsx: Form for creating a new swap proposal with calendar integration.
- Pages/Swaps/Edit.tsx: Form for editing an existing swap proposal with calendar integration.
- Types/Swap.ts: Typescript Type definitions for swap data.

**API Endpoints:**
- POST /api/v1/swaps
- PUT /api/v1/swaps/{id}

**Testing Requirements:**
- Unit tests for SwapService to ensure swap scheduling is working correctly.
- Feature tests for SwapController to verify swap creation and update endpoints.
- Component tests for Calendar.tsx to ensure proper date and time selection.
- Integration tests to ensure swap schedules are correctly stored and retrieved.


### 6. Geolocation for user location display and proximity searching.
The system uses geolocation to display user locations on a map and enable proximity-based searching.

**Database Changes:**
- Add 'latitude' and 'longitude' columns to the 'users' table (if not already present).
- Add 'location' column (POINT) to the 'users' table for geospatial queries.

**Backend Components:**
- UserService: Methods for retrieving users with geolocation data and performing proximity searches.
- UserController: Endpoints for retrieving user locations.
- GeoLocationService: Optional service to make google maps api calls

**Frontend Components:**
- Components/Map.tsx: Map component for displaying user locations.
- Components/LocationPicker.tsx: Component for selecting a location on the map.
- Hooks/useGeolocation.ts: Custom hook to obtain the user's current location.

**API Endpoints:**
- GET /api/v1/users/{id}/location
- GET /api/v1/users/search?latitude={latitude}&longitude={longitude}&radius={radius}

**Testing Requirements:**
- Unit tests for UserService to ensure geolocation data is handled correctly.
- Feature tests for UserController to verify location retrieval and proximity search endpoints.
- Component tests for Map.tsx to ensure user locations are displayed correctly.
- Integration tests to verify the entire geolocation workflow.


## Metadata
- **Generated**: 2025-06-18T11:19:41.034184+00:00
- **Project Type**: Laravel React Starter Kit
- **Architecture Version**: 1.0.0

---
*This architecture document is maintained by the CodeWebMobile AI system and should be the source of truth for all development decisions.*
