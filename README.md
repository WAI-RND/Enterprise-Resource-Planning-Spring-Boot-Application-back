# Enterprise Resource Planning System

A full-stack Enterprise Resource Planning (ERP) system built with **Spring Boot** (backend) and **React** (frontend) to support business operations including HR records, sales document workflows, task/calendar scheduling, real-time collaboration, and security log monitoring.

## Overview

Built for Union Concept, this system combines real-time chat, task and calendar management, an HR employee module, a sales quote-to-invoice pipeline, and a log-based security monitoring layer into a single Spring Boot + React application, deployed locally via Docker Compose.

## Project Status

This is a broad, actively-developed prototype rather than a finished production ERP. The chat, calendar/task, HR, and sales modules are functionally substantial. Authorization hardening across modules, automated testing, and database migrations are still in progress — see [Known Limitations](#known-limitations) below before relying on this for real business or production data.

## Key Features

### Account & User Management
- User registration with administrator approval before an account becomes active
- JWT-based authentication
- Role-based access: `USER` and `ADMIN`
- Password reset via a six-digit emailed verification code
- Profile management and account settings
- Admin dashboard for reviewing pending accounts and managing users

### Calendar & Scheduling
- Interactive calendar with event creation, editing, and deletion
- Tasks and events share a unified model, with start/end times and completion status
- Global (all-user) and user-assigned events
- Status tracking: pending, in progress, completed, cancelled

### Real-Time Chat System
- Instant messaging between users with participant-based conversations
- Text and file messages, message history, typing indicators, and read status
- Conversation archiving and search
- Delivered over WebSocket/STOMP

### Task Management
- Create and assign tasks to employees
- Track task status, priority, and due dates
- Task detail view and an all-tasks overview dashboard

### Employee Management (HR)
- Employee profile creation and editing (contact info, position, salary, hire/birth date, address)
- HR task assignment and tracking per employee
- Employee status tracking

### Sales Module
- Client and product management
- Quote creation and lifecycle tracking (draft, sent, accepted, rejected, expired, converted to order)
- Order lifecycle tracking, with quote-to-order conversion
- Invoice generation, lifecycle tracking, and email delivery
- Client-side PDF generation for invoices
- Sales summary and product-sales reporting

### Security Log Monitoring (Experimental)
- Backend log scanner that flags patterns associated with brute-force attempts, path traversal, SQL injection, cross-site scripting, and similar activity
- Alert listing, summary statistics, and top-attacking-IP views
- Frontend security dashboard for browsing alerts and reports

This module is an early, pattern-matching log scanner rather than a full security-operations platform — see Known Limitations.

## Technology Stack

### Frontend
- **React 19** - UI framework
- **Vite** - Build tool and dev server
- **React Router** - Navigation and routing
- **Axios** - HTTP client for API calls
- **Tailwind CSS**, **Material UI**, **Ant Design** - styling and components
- **React Hook Form** + **Yup** - form handling and validation
- **TanStack Query** / **TanStack Table** - data fetching and tables
- **STOMP.js** + **SockJS** - WebSocket real-time communication
- **Chart.js** / **Recharts** - charts and reporting visuals
- **jsPDF** + **html2canvas** - client-side PDF generation
- **Google reCAPTCHA v2** - bot-protection widget (frontend only, see Known Limitations)

### Backend
- **Java 21**
- **Spring Boot 3.4** - application framework
- **Spring Security** - authentication and authorization
- **Spring Data JPA** / **Hibernate** - database access
- **Spring WebSocket** (STOMP/SockJS) - real-time messaging
- **Spring Mail** - email delivery
- **JJWT** - JWT issuing and validation
- **BCrypt** - password hashing
- **Maven** - build and dependency management
- **PostgreSQL 16** - database

### Infrastructure
- **Docker Compose** - local multi-service deployment (PostgreSQL, backend, Nginx-served frontend)
- **Nginx** - static frontend hosting and API/WebSocket proxying
- **JWT** - stateless, token-based session handling
- **CORS** - cross-origin configuration
- **Environment-based configuration** - `.properties` files per profile (dev, docker, prod)

## Project Structure

```
Enterprise-Resource-Planning-Spring-Boot-Application/
├── Backend/
│   ├── pom.xml
│   ├── Dockerfile
│   └── src/main/
│       ├── java/com/secureops/
│       │   ├── config/
│       │   ├── controller/
│       │   ├── dto/
│       │   ├── entity/
│       │   ├── exception/
│       │   ├── repository/
│       │   ├── service/
│       │   ├── util/
│       │   ├── hr/          # Employee & HR-task module
│       │   └── sales/       # Clients, products, quotes, orders, invoices
│       └── resources/
│           ├── application.properties
│           ├── application-dev.properties
│           ├── application-docker.properties
│           └── application-prod.properties
├── Frontend/
│   ├── package.json
│   ├── vite.config.js
│   ├── Dockerfile
│   ├── nginx.conf
│   └── src/
│       ├── Components/
│       ├── pages/
│       ├── context/
│       ├── services/
│       ├── Router/
│       ├── modules/
│       ├── hooks/
│       └── utils/
├── docker-compose.yml
├── README.md
├── PROJECT_DESCRIPTION.md
└── LICENSE.txt
```

## Getting Started

### Prerequisites
- **Docker Desktop** (for the Quick Start path)
- **Node.js** (v18+) and **Java JDK 21** + **Maven** (for manual/non-Docker setup)
- **PostgreSQL** (only needed for manual/non-Docker setup)
- **Git**

### Quick Start (Docker Compose)

1. **Clone the repository**
   ```bash
   git clone https://github.com/yassernamez03/Enterprise-Resource-Planning-Spring-Boot-Application.git
   cd Enterprise-Resource-Planning-Spring-Boot-Application
   ```

2. **Set your own environment values**

   Before starting, set your own values for the database credentials, JWT signing secret, mail credentials, and the initial admin account — either in a `.env` file referenced by `docker-compose.yml` or as exported environment variables. Don't rely on any built-in fallback values beyond a throwaway local test.

3. **Start the stack**
   ```bash
   docker-compose up -d --build
   ```
   This brings up PostgreSQL, the Spring Boot backend, and the Nginx-served frontend together. Check `docker-compose.yml` for the exposed frontend port on your machine.

### Manual Setup (without Docker)

**Frontend**
```bash
cd Frontend
npm install
```
Create a `.env` file:
```env
VITE_BACKEND_URL=https://localhost:8443/api
VITE_API_URL=https://localhost:8443/api
VITE_RECAPTCHA_SITE_KEY=your_recaptcha_site_key
```
Generate local SSL certificates (one-time setup, for HTTPS in local dev):
```bash
# Install mkcert
choco install mkcert  # Windows
brew install mkcert   # macOS

mkcert -install
mkdir cert && cd cert
mkcert localhost 127.0.0.1 ::1
```
Then:
```bash
npm run dev
```
Access at `https://localhost:3000`.

**Backend**

Edit `src/main/resources/application.properties`:
```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/erp_db
spring.datasource.username=your_username
spring.datasource.password=your_password
server.port=8443
```
Then:
```bash
mvn clean install
mvn spring-boot:run
```
API available at `https://localhost:8443/api`.

## API Documentation

### Authentication
- `POST /api/auth/login` - User login
- `POST /api/auth/register` - User registration
- `POST /api/auth/forgot-password` - Request password reset
- `POST /api/auth/verify-reset` - Verify reset code

### User Management
- `GET /api/users` - Get all users (Admin)
- `GET /api/users/{id}` - Get user by ID
- `PUT /api/users/{id}` - Update user
- `DELETE /api/users/{id}` - Delete user (Admin)

### Task Management
- `GET /api/tasks` - Get all tasks
- `POST /api/tasks` - Create new task
- `PUT /api/tasks/{id}` - Update task
- `DELETE /api/tasks/{id}` - Delete task

### Sales
- `GET /api/sales/orders` - Get sales orders
- `POST /api/sales/orders` - Create order
- `GET /api/sales/invoices` - Get invoices
- `POST /api/sales/invoices` - Generate invoice
- `GET /api/sales/reports` - Get sales reports

### Calendar
- `GET /api/calendar/events` - Get all events
- `POST /api/calendar/events` - Create event
- `PUT /api/calendar/events/{id}` - Update event
- `DELETE /api/calendar/events/{id}` - Delete event

## Security Features

- **Authentication**: JWT-based, stateless session handling
- **Roles**: `USER` and `ADMIN`, with administrator approval required before a new account is activated
- **Password Security**: Bcrypt hashing
- **Transport**: HTTPS/SSL supported for local development via mkcert; configure TLS termination explicitly for any shared or production deployment
- **CORS**: Configurable cross-origin policy
- **Input Validation**: Bean Validation on core request DTOs (authentication, several sales endpoints)

Refresh-token rotation, consistent role-based authorization across every module, and enforced server-side reCAPTCHA validation are not yet in place — see Known Limitations.

## User Interface

- Responsive design across desktop, tablet, and mobile
- Dark/Light mode (planned)
- Real-time updates via WebSocket
- Loading states, toast notifications, and user-friendly error handling

## Database Schema

Main entities:

- **Users**: accounts and authentication
- **Tasks / Events**: calendar and task scheduling
- **Messages**: chat messages and conversations
- **Employees**: HR employee records
- **Clients, Products, Quotes, Orders, Invoices**: sales pipeline
- **Security Logs**: activity and security-scan log entries

## Deployment

### Docker Compose (recommended)
The repository includes `Dockerfile`s for both the frontend and backend plus a root `docker-compose.yml` that runs PostgreSQL, the backend, and an Nginx-served frontend together. This is the primary supported deployment path — see Quick Start above.

### Frontend only (Vercel/Netlify)
```bash
cd Frontend
npm run build
# Deploy the dist/ folder
```

### Backend only (Heroku/AWS/Azure)
```bash
cd Backend
mvn clean package
# Deploy target/*.jar
```

## Known Limitations

- Authorization is currently strongest around authentication and admin-only routes; consistent role-based checks across the HR and sales modules are still being built out.
- No refresh-token flow yet — sessions rely on a single JWT until it expires.
- reCAPTCHA has a frontend widget in place, but server-side verification isn't wired up yet.
- No automated test suite (backend or frontend) yet.
- No versioned database migrations yet — schema changes currently rely on Hibernate's auto-update.
- The security dashboard currently displays sample data in several panels while backend integration is completed.
- Database driver support is PostgreSQL only (not MySQL, despite older docs implying otherwise).
- Inventory, general-ledger accounting, and payroll are out of scope for the current sales/HR modules.

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

**Important**: Never commit sensitive information (API keys, passwords, certificates).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE.txt) file for details.

## Author

**Yasser Namez**
- GitHub: [@yassernamez03](https://github.com/yassernamez03)

## Acknowledgments

- Built for Union Concept
- Spring Boot and React communities
- Open source contributors

## Support

For support and questions:
- Create an issue on GitHub
- Contact the development team
