# Personal Expense Tracker - FastAPI Implementation

A full-stack application for tracking personal expenses built with FastAPI, React, and PostgreSQL.

## Project Overview

This personal expense tracker application helps manage finances by tracking expenses, categorizing spending, setting budgets, and visualizing spending patterns.

### Updated Tech Stack

- **Backend**: FastAPI (Python 3.9+), PostgreSQL
- **Frontend**: React 18+
- **ORM**: SQLAlchemy with Alembic migrations
- **Authentication**: JWT with python-jose
- **Deployment**: Docker on VPS

## Architecture

### Backend Architecture

The FastAPI application follows a clean architecture pattern:

- **API Layer**: FastAPI routers and endpoints
- **Service Layer**: Business logic implementation
- **Repository Layer**: Data access using SQLAlchemy
- **Models Layer**: SQLAlchemy models and Pydantic schemas
- **Auth Layer**: JWT authentication middleware

### Database Design

Key tables in the PostgreSQL database (unchanged from original):

- **users**: User authentication and profile information
- **categories**: Expense categories (groceries, utilities, etc.)
- **expenses**: Individual expense records
- **budgets**: Monthly/weekly budget limits per category
- **recurring_expenses**: Subscriptions and recurring bills

### Frontend Architecture

The React application remains the same:

- **Component Structure**: Reusable UI components
- **State Management**: React Context for global state
- **Routing**: React Router for navigation
- **API Integration**: Axios for backend communication
- **Visualization**: Charts.js for expense analytics

## Features

### Core Features (Unchanged)

1. **Expense Tracking**
   - Add/edit/delete expenses
   - Categorize expenses
   - Search and filtering capabilities

2. **Budget Management**
   - Set monthly/weekly budgets per category
   - Visual indicators for budget health

3. **Reporting & Analytics**
   - Monthly spending breakdown
   - Category comparison
   - Spending trends over time

4. **User Experience**
   - Responsive design
   - Quick expense entry

## Development Plan

The project will be developed in phases:

1. **Initial Setup**
   - FastAPI project structure setup
   - SQLAlchemy models and database setup
   - JWT authentication implementation
   - Alembic migration setup

2. **Core Functionality**
   - Expense CRUD operations
   - Category management
   - Basic UI implementation
   - API documentation with FastAPI

3. **Advanced Features**
   - Budget tracking
   - Reporting and visualizations
   - Data import/export
   - Background tasks for recurring expenses

4. **Finalization**
   - Testing and bug fixes
   - Docker deployment setup
   - API documentation finalization

## Testing Strategy

Python-focused testing approach:

- **Backend**: pytest for unit and integration tests
- **API Testing**: FastAPI TestClient for endpoint testing
- **Frontend**: Manual testing of main user flows
- **Database**: pytest-postgresql for database testing

## Deployment Strategy

Simple deployment for personal VPS:

- Docker containers for backend and frontend
- Nginx as reverse proxy
- Uvicorn as ASGI server for FastAPI
- Basic backup strategy for PostgreSQL data
- Manual deployment process

## Project Structure

### Backend Directory Structure

```
backend/
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── core/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── security.py
│   │   └── database.py
│   ├── api/
│   │   ├── __init__.py
│   │   ├── deps.py
│   │   └── v1/
│   │       ├── __init__.py
│   │       ├── auth.py
│   │       ├── expenses.py
│   │       ├── categories.py
│   │       └── budgets.py
│   ├── models/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── expense.py
│   │   ├── category.py
│   │   └── budget.py
│   ├── schemas/
│   │   ├── __init__.py
│   │   ├── user.py
│   │   ├── expense.py
│   │   ├── category.py
│   │   └── budget.py
│   ├── services/
│   │   ├── __init__.py
│   │   ├── auth_service.py
│   │   ├── expense_service.py
│   │   └── budget_service.py
│   └── utils/
│       ├── __init__.py
│       └── helpers.py
├── alembic/
│   ├── versions/
│   ├── env.py
│   └── alembic.ini
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── test_auth.py
│   └── test_expenses.py
├── requirements.txt
├── Dockerfile
└── .env.example
```

### Frontend Directory Structure (Unchanged)

```
frontend/
├── src/
│   ├── components/
│   │   ├── common/
│   │   │   ├── Header.jsx
│   │   │   ├── Sidebar.jsx
│   │   │   └── Footer.jsx
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── expenses/
│   │   │   ├── ExpenseList.jsx
│   │   │   ├── ExpenseForm.jsx
│   │   │   └── ExpenseFilters.jsx
│   │   ├── categories/
│   │   └── reports/
│   ├── hooks/
│   ├── context/
│   ├── services/
│   │   ├── api.js
│   │   ├── authService.js
│   │   └── expenseService.js
│   ├── utils/
│   └── App.jsx
├── package.json
└── Dockerfile
```

## Key Dependencies

### Backend Requirements

```txt
fastapi==0.104.1
uvicorn[standard]==0.24.0
sqlalchemy==2.0.23
alembic==1.12.1
psycopg2-binary==2.9.9
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
python-multipart==0.0.6
pydantic==2.5.0
python-dotenv==1.0.0
pytest==7.4.3
pytest-asyncio==0.21.1
httpx==0.25.2
```

### FastAPI Advantages for This Project

1. **Automatic API Documentation**: Built-in Swagger/OpenAPI docs
2. **Type Safety**: Python type hints with Pydantic validation
3. **Performance**: Async support and high performance
4. **Developer Experience**: Easy to learn and maintain
5. **Modern Python**: Leverages latest Python features

## Getting Started

### Prerequisites

- Python 3.9+
- Node.js 16+
- PostgreSQL 14+
- Docker & Docker Compose

### Installation

1. Clone the repository
```bash
git clone https://github.com/yourusername/personal-expense-tracker.git
cd personal-expense-tracker
```

2. Set up the backend
```bash
cd backend
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
pip install -r requirements.txt
```

3. Set up environment variables
```bash
cp .env.example .env
# Edit .env with your database credentials
```

4. Run database migrations
```bash
alembic upgrade head
```

5. Set up the frontend
```bash
cd frontend
npm install
```

6. Run the application
```bash
# Backend (in backend directory)
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Frontend (in frontend directory)
npm run dev
```

### Development Commands

```bash
# Create new migration
alembic revision --autogenerate -m "description"

# Run tests
pytest

# Format code
black app/
isort app/

# Start FastAPI with auto-reload
uvicorn app.main:app --reload
```

### Deployment

1. Build Docker images
```bash
docker-compose build
```

2. Deploy containers
```bash
docker-compose up -d
```

3. Run migrations in production
```bash
docker-compose exec backend alembic upgrade head
```

## API Documentation

FastAPI automatically generates interactive API documentation:

- **Swagger UI**: `http://localhost:8000/docs`
- **ReDoc**: `http://localhost:8000/redoc`
- **OpenAPI JSON**: `http://localhost:8000/openapi.json`

## Key Implementation Notes

### Authentication Flow
1. User registers/logs in via `/auth/login`
2. Server returns JWT access token
3. Client includes token in `Authorization: Bearer <token>` header
4. FastAPI dependency injection validates token automatically

### Database Operations
- SQLAlchemy ORM for database operations
- Alembic for database migrations
- Connection pooling handled by SQLAlchemy
- Async database operations where beneficial

### Error Handling
- FastAPI exception handlers for consistent error responses
- Pydantic validation for request/response data
- Custom exception classes for business logic errors

## License

This project is licensed under the MIT License - see the LICENSE file for details.