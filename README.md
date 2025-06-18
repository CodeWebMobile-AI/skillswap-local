```markdown
# SkillSwap Local

## 1. Project Title and Description

SkillSwap Local is a community-focused platform designed to connect individuals for reciprocal skill-sharing. It facilitates local learning and fosters resourcefulness by enabling users to offer their expertise in exchange for skills they wish to learn.

## 2. Problem Statement

Many people possess valuable skills they're willing to share, but lack a platform to connect with others seeking those skills locally. Traditional learning platforms are often expensive and impersonal, creating a barrier to affordable and accessible learning opportunities within communities.

## 3. Target Audience and Value Proposition

*   **Target Audience:** Community residents, hobbyists, retirees, and individuals seeking affordable learning opportunities or wanting to share their expertise.
*   **Value Proposition:**
    *   **Affordable Learning:** Access skills and knowledge without the high costs of traditional courses.
    *   **Community Building:** Connect with neighbors and build relationships through shared learning experiences.
    *   **Skill Development:** Learn new skills and enhance existing ones, while also sharing your own expertise.
    *   **Local Resourcefulness:** Tap into the wealth of knowledge and skills within your community.

## 4. Features

The initial features of SkillSwap Local include:

*   **Profile Creation:** Users can create detailed profiles showcasing their skills, interests, and experience.  Both offered and requested skills can be listed.
*   **Proximity-Based Search:**  Find other users within a specified radius who are offering or requesting specific skills.
*   **Secure Messaging System:**  Communicate directly with potential skill-swap partners to coordinate details and schedule sessions.
*   **Review and Rating System:**  Build trust and reputation by leaving reviews and ratings for other users after a skill-swap.
*   **Calendar Integration:**  Schedule skill-swap sessions using a built-in calendar to manage availability and appointments.
*   **Geolocation:** Enable location display on a map and facilitate proximity-based searching for relevant skill exchanges.

## 5. Tech Stack

SkillSwap Local utilizes a modern web development stack:

*   **Frontend:**
    *   **React:** A JavaScript library for building user interfaces.
    *   **Inertia.js:** Glues the React frontend to the Laravel backend, allowing for server-side routing and data fetching while maintaining a single-page application experience.
    *   **TypeScript:** A superset of JavaScript that adds static typing, improving code quality and maintainability.
*   **Backend:**
    *   **Laravel:** A PHP framework for building robust and scalable web applications.
*   **Database:**
    *   **PostgreSQL:** A powerful and reliable open-source relational database.
*   **Real-time Communication:**
    *   **Redis:**  An in-memory data store used as a message broker for real-time events.
    *   **laravel-echo-server:** A Node.js server for handling WebSocket connections and broadcasting events.
*   **Authentication:**
    *   **Laravel Sanctum:**  Provides a lightweight API authentication system using tokens.

## 6. Architecture Overview

SkillSwap Local employs a layered architecture for maintainability and scalability:

*   **Frontend (React/Inertia.js/TypeScript):**  Handles the user interface and user interactions.  Uses Inertia.js for seamless routing and data integration with the backend.
*   **Backend (Laravel):** Implements the application's business logic, data access, and API endpoints.
*   **Database (PostgreSQL):**  Stores all application data, including user profiles, skills, messages, and swap information. Geospatial extensions are used for location-based queries.
*   **Real-time Server (laravel-echo-server & Redis):**  Enables real-time communication using WebSockets, allowing for instant updates for messages and swap status changes.
*   **API Design:**  The API is designed using RESTful principles, providing a clear and consistent interface for the frontend to interact with the backend.
*   **API Gateway (Optional, for future scaling):**  Future implementation to manage authentication, rate limiting, and routing of API requests.

See `Architecture Details` for a detailed description of the application architecture.

## 7. Prerequisites

Before you begin, ensure you have the following installed:

*   **PHP:** Version 8.1 or higher
*   **Composer:**  PHP dependency manager
*   **Node.js:** Version 16 or higher
*   **npm:** Node Package Manager (usually installed with Node.js)
*   **PostgreSQL:**  With PostGIS extension enabled
*   **Redis:**  Data structure store
*   **laravel-echo-server:** (for real-time functionality)

## 8. Installation Steps

1.  **Clone the repository:**
    ```bash
    git clone <repository_url>
    cd skillswap-local
    ```

2.  **Install PHP dependencies:**
    ```bash
    composer install
    ```

3.  **Install JavaScript dependencies:**
    ```bash
    npm install
    ```

4.  **Copy the `.env.example` file to `.env` and configure your environment variables:**
    ```bash
    cp .env.example .env
    ```

5.  **Generate an application key:**
    ```bash
    php artisan key:generate
    ```

6.  **Configure your database in the `.env` file:**
    ```
    DB_CONNECTION=pgsql
    DB_HOST=127.0.0.1
    DB_PORT=5432
    DB_DATABASE=skillswap
    DB_USERNAME=your_username
    DB_PASSWORD=your_password
    ```

7.  **Run database migrations:**
    ```bash
    php artisan migrate
    ```

8.  **Seed the database (optional, for sample data):**
    ```bash
    php artisan db:seed
    ```

9.  **Compile the frontend assets:**
    ```bash
    npm run dev
    ```

10. **Start the development server:**
    ```bash
    php artisan serve
    ```

## 9. Environment Setup

Configure the following environment variables in your `.env` file:

*   `APP_NAME`: The name of your application (e.g., "SkillSwap Local")
*   `APP_URL`: The URL of your application (e.g., "http://localhost:8000")
*   `DB_CONNECTION`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`:  Database connection details for PostgreSQL.
*   `REDIS_HOST`, `REDIS_PORT`, `REDIS_PASSWORD`: Redis connection details.
*   `BROADCAST_DRIVER=redis`
*   `CACHE_DRIVER=redis`
*   `QUEUE_CONNECTION=redis`
*   `SESSION_DRIVER=redis`
*   `SANCTUM_STATEFUL_DOMAINS=localhost:3000,127.0.0.1:8000`

**Configure `laravel-echo-server.json`:**

*  Update the `host`, `port`, `database` and `authHost` properties with your environment settings.

**Start `laravel-echo-server`:**
```bash
laravel-echo-server start
```

## 10. Development Workflow

1.  **Create a new branch for your feature or bug fix:**
    ```bash
    git checkout -b feature/your-feature-name
    ```

2.  **Make your changes and commit them with descriptive messages:**
    ```bash
    git add .
    git commit -m "Add: Implement feature X"
    ```

3.  **Push your branch to the remote repository:**
    ```bash
    git push origin feature/your-feature-name
    ```

4.  **Create a pull request to merge your branch into the `main` branch.**

## 11. Testing

SkillSwap Local utilizes a comprehensive testing strategy:

*   **Backend (Laravel):**
    *   **Unit Tests (Pest):**  Test individual units of code in isolation.
    *   **Feature Tests (Pest):** Test the application's features from an end-to-end perspective.
*   **Frontend (React):**
    *   **Unit Tests (Vitest):** Test individual components and functions in isolation.
    *   **Component Tests (React Testing Library):** Test how components render and behave.

**To run the backend tests:**

```bash
php artisan test
```

**To run the frontend tests:**

```bash
npm run test:unit
```

## 12. Deployment

SkillSwap Local can be deployed on various platforms, including:

*   **AWS:** Amazon Web Services
*   **DigitalOcean:** Cloud infrastructure provider
*   **Heroku:** Platform-as-a-Service

A typical deployment setup involves:

*   **Web Server (Nginx or Apache):** Serves the frontend assets and proxies requests to the PHP application.
*   **PHP Application Server (PHP-FPM):** Executes the Laravel application.
*   **Database Server (PostgreSQL):** Stores application data.
*   **Redis Server:** Acts as a message broker for real-time events.
*   **laravel-echo-server:** Runs the websocket server.

Refer to the `Deployment Plan` section in the Architecture Details for more detailed instructions on deploying and scaling the application.

## 13. Monetization Strategy

SkillSwap Local employs a freemium model with the following monetization strategies:

*   **Freemium Model:**  Limited monthly skill-swaps and basic profile features are available for free.  A paid subscription unlocks unlimited swaps and advanced profile customization.
*   **Verified Expert Badges:**  Offer paid "verified expert" badges to users who undergo background checks and skill assessments, providing added credibility and trust.
*   **Local Business Partnerships:**  Partner with local businesses to offer discounts or exclusive deals to SkillSwap users, generating revenue through affiliate commissions or sponsored content.

## 14. Contributing Guidelines

We welcome contributions to SkillSwap Local! Please follow these guidelines:

1.  **Fork the repository.**
2.  **Create a new branch for your changes.**
3.  **Write clear and concise commit messages.**
4.  **Include tests for your changes.**
5.  **Submit a pull request with a detailed description of your changes.**

## 15. Custom Sections (From Project Details)

*   **Design System:**  SkillSwap Local utilizes a consistent design system for a cohesive and user-friendly experience.  The design system includes:
    *   **Font:** Nunito Sans (`'Nunito Sans', sans-serif`)
    *   **Color Palette:**
        *   Primary: Teal (`#26A69A`)
        *   Secondary: Light Grey (`#ECEFF1`)
        *   Accent: Amber (`#FFB300`)
        *   Neutral Text: Dark Grey (`#212121`)
        *   Neutral Background: White (`#FFFFFF`)
        *   Neutral Border: Light Grey Border (`#CFD8DC`)
        *   Success: Green (`#4CAF50`)
        *   Warning: Orange (`#FF9800`)
        *   Danger: Red (`#F44336`)

*   **Feature Implementation Roadmap:** The `feature_implementation_roadmap` object contains a detailed plan for upcoming features, including database changes, impacted components, API endpoints, and suggested tests. See `Architecture Details` for more information.

*   **Database Schema:** The `database_schema` object describes the structure of the database, including table names, column names, data types, and relationships.  This schema is designed for clarity, efficiency, and data integrity. See `Architecture Details` for more information.

```
