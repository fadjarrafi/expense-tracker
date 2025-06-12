# Personal Expense Tracker - Initial Setup Guide

## Prerequisites

Before starting, ensure you have:
- Python 3.9+ installed
- Node.js 16+ and npm
- PostgreSQL 14+ running locally
- Git for version control
- Docker & Docker Compose (optional, for deployment)

## Step 1: Project Structure Setup

Create the main project directory and structure:

```bash
mkdir personal-expense-tracker
cd personal-expense-tracker

# Create main directories
mkdir backend frontend

# Initialize git repository
git init
```

## Step 2: Backend Setup (FastAPI)

### 2.1 Create Backend Structure

```bash
cd backend

# Create Python virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Create directory structure
mkdir -p app/{core,api/v1,models,schemas,services,utils}
mkdir -p alembic/versions
mkdir tests

# Create __init__.py files
touch app/__init__.py
touch app/core/__init__.py
touch app/api/__init__.py
touch app/api/v1/__init__.py
touch app/models/__init__.py
touch app/schemas/__init__.py
touch app/services/__init__.py
touch app/utils/__init__.py
touch tests/__init__.py
```

### 2.2 Create Requirements File

Create `requirements.txt`:

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
pydantic-settings==2.0.3
python-dotenv==1.0.0
pytest==7.4.3
pytest-asyncio==0.21.1
httpx==0.25.2
```

### 2.3 Install Dependencies

```bash
pip install -r requirements.txt
```

### 2.4 Create Core Configuration Files

**app/core/config.py**
```python
from pydantic_settings import BaseSettings
from typing import Optional

class Settings(BaseSettings):
    PROJECT_NAME: str = "Personal Expense Tracker"
    VERSION: str = "1.0.0"
    API_V1_STR: str = "/api/v1"
    
    # Database
    DATABASE_URL: str = "postgresql://username:password@localhost/expense_tracker"
    
    # Security
    SECRET_KEY: str = "your-secret-key-here-change-in-production"
    ALGORITHM: str = "HS256"
    ACCESS_TOKEN_EXPIRE_MINUTES: int = 30
    
    # CORS
    BACKEND_CORS_ORIGINS: list[str] = ["http://localhost:3000", "http://localhost:5173"]
    
    class Config:
        env_file = ".env"

settings = Settings()
```

**app/core/database.py**
```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from .config import settings

engine = create_engine(settings.DATABASE_URL)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

**app/core/security.py**
```python
from datetime import datetime, timedelta
from typing import Optional
from jose import JWTError, jwt
from passlib.context import CryptContext
from .config import settings

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, settings.SECRET_KEY, algorithm=settings.ALGORITHM)
    return encoded_jwt

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def verify_token(token: str):
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=[settings.ALGORITHM])
        return payload
    except JWTError:
        return None
```

### 2.5 Create Main Application File

**app/main.py**
```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from .core.config import settings

app = FastAPI(
    title=settings.PROJECT_NAME,
    version=settings.VERSION,
    openapi_url=f"{settings.API_V1_STR}/openapi.json"
)

# CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=settings.BACKEND_CORS_ORIGINS,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def root():
    return {"message": "Personal Expense Tracker API", "version": settings.VERSION}

@app.get("/health")
async def health_check():
    return {"status": "healthy"}
```

### 2.6 Create Environment File

Create `.env`:
```env
# Database
DATABASE_URL=postgresql://your_username:your_password@localhost/expense_tracker

# Security
SECRET_KEY=your-very-secret-key-change-this-in-production
ACCESS_TOKEN_EXPIRE_MINUTES=30

# Development
DEBUG=True
```

### 2.7 Setup Alembic

Initialize Alembic:
```bash
alembic init alembic
```

Edit `alembic.ini` to use your database URL:
```ini
sqlalchemy.url = postgresql://your_username:your_password@localhost/expense_tracker
```

Edit `alembic/env.py`:
```python
from logging.config import fileConfig
from sqlalchemy import engine_from_config, pool
from alembic import context
import sys
import os

# Add your project directory to Python path
sys.path.append(os.path.dirname(os.path.dirname(__file__)))

from app.core.database import Base
from app.core.config import settings

# this is the Alembic Config object
config = context.config

# Override sqlalchemy.url with our settings
config.set_main_option("sqlalchemy.url", settings.DATABASE_URL)

# Interpret the config file for Python logging
if config.config_file_name is not None:
    fileConfig(config.config_file_name)

target_metadata = Base.metadata

def run_migrations_offline() -> None:
    url = config.get_main_option("sqlalchemy.url")
    context.configure(
        url=url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )

    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online() -> None:
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection, target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

## Step 3: Frontend Setup (React)

### 3.1 Create React Application

```bash
cd ../frontend

# Create React app with Vite (recommended for better performance)
npm create vite@latest . -- --template react
npm install

# Install additional dependencies
npm install axios react-router-dom @tanstack/react-query
npm install -D tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

### 3.2 Setup Tailwind CSS v4.0

**For Tailwind CSS v4.0 (latest):**

Install Tailwind CSS v4.0:
```bash
npm install @tailwindcss/cli@next @tailwindcss/postcss@next
```

**Option A: Using PostCSS (Recommended)**

Add to your `postcss.config.js`:
```javascript
export default {
  plugins: {
    '@tailwindcss/postcss': {},
  },
}
```

**Option B: Using CLI**

If you prefer CLI approach:
```bash
npx tailwindcss init --esm --postcss
```

Configure `tailwind.config.js` for v4.0:
```javascript
/** @type {import('@tailwindcss/cli').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  // v4.0 uses CSS configuration instead of JS config for theme
}
```

**Add Tailwind to your CSS** - Replace `src/index.css`:
```css
/* Tailwind CSS v4.0 syntax */
@import "tailwindcss";

/* Custom styles can go here */
@layer components {
  .btn-primary {
    @apply bg-blue-500 hover:bg-blue-700 text-white font-bold py-2 px-4 rounded;
  }
}
```

**Alternative for Tailwind CSS v3.x (if you prefer stable version):**

If you want to stick with the stable v3.x:
```bash
npm install -D tailwindcss@latest postcss autoprefixer
npx tailwindcss init -p
```

Use the traditional configuration:
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

And traditional CSS:
```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

### 3.3 Create Directory Structure

```bash
mkdir -p src/{components/{common,auth,dashboard,expenses,categories,reports},hooks,context,services,utils}
```

### 3.4 Create Basic Configuration Files

**src/services/api.js**
```javascript
import axios from 'axios';

const API_BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:8000';

const api = axios.create({
  baseURL: `${API_BASE_URL}/api/v1`,
  headers: {
    'Content-Type': 'application/json',
  },
});

// Request interceptor for auth token
api.interceptors.request.use(
  (config) => {
    const token = localStorage.getItem('access_token');
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => {
    return Promise.reject(error);
  }
);

// Response interceptor for error handling
api.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      localStorage.removeItem('access_token');
      window.location.href = '/login';
    }
    return Promise.reject(error);
  }
);

export default api;
```

**src/context/AuthContext.jsx**
```javascript
import React, { createContext, useContext, useState, useEffect } from 'react';

const AuthContext = createContext();

export const useAuth = () => {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within an AuthProvider');
  }
  return context;
};

export const AuthProvider = ({ children }) => {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const token = localStorage.getItem('access_token');
    if (token) {
      // TODO: Validate token and set user
      setUser({ token });
    }
    setLoading(false);
  }, []);

  const login = (token, userData) => {
    localStorage.setItem('access_token', token);
    setUser({ token, ...userData });
  };

  const logout = () => {
    localStorage.removeItem('access_token');
    setUser(null);
  };

  const value = {
    user,
    login,
    logout,
    loading,
  };

  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
};
```

### 3.5 Create Environment File

Create `.env`:
```env
VITE_API_URL=http://localhost:8000
```

## Step 4: Database Setup

### 4.1 Create PostgreSQL Database

```bash
# Connect to PostgreSQL
psql -U postgres

# Create database and user
CREATE DATABASE expense_tracker;
CREATE USER expense_user WITH PASSWORD 'your_password';
GRANT ALL PRIVILEGES ON DATABASE expense_tracker TO expense_user;
\q
```

### 4.2 Update Database Configuration

Update your `.env` files with the correct database credentials:

**Backend .env:**
```env
DATABASE_URL=postgresql://expense_user:your_password@localhost/expense_tracker
```

## Step 5: Docker Setup (Optional)

### 5.1 Backend Dockerfile

Create `backend/Dockerfile`:
```dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

CMD ["uvicorn", "app.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

### 5.2 Frontend Dockerfile

Create `frontend/Dockerfile`:
```dockerfile
FROM node:18-alpine as builder

WORKDIR /app
COPY package*.json ./
RUN npm install

COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### 5.3 Docker Compose

Create `docker-compose.yml` in the root directory:
```yaml
version: '3.8'

services:
  db:
    image: postgres:15
    environment:
      POSTGRES_DB: expense_tracker
      POSTGRES_USER: expense_user
      POSTGRES_PASSWORD: your_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  backend:
    build: ./backend
    ports:
      - "8000:8000"
    depends_on:
      - db
    environment:
      DATABASE_URL: postgresql://expense_user:your_password@db/expense_tracker
    volumes:
      - ./backend:/app

  frontend:
    build: ./frontend
    ports:
      - "3000:80"
    depends_on:
      - backend

volumes:
  postgres_data:
```

## Step 6: Initial Testing

### 6.1 Test Backend

```bash
cd backend
source venv/bin/activate
uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

Visit `http://localhost:8000` and `http://localhost:8000/docs` for API documentation.

### 6.2 Test Frontend

```bash
cd frontend
npm run dev
```

Visit `http://localhost:5173` (or the port shown in terminal).

## Step 7: Git Configuration

Create `.gitignore` in the root directory:
```gitignore
# Python
__pycache__/
*.py[cod]
*$py.class
*.so
.Python
env/
venv/
ENV/
env.bak/
venv.bak/

# Node
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*
dist/
.DS_Store

# Environment variables
.env
.env.local
.env.production

# IDE
.vscode/
.idea/
*.swp
*.swo

# Database
*.db
*.sqlite3

# Logs
*.log
logs/

# Docker
docker-compose.override.yml
```

## Next Steps

1. **Database Models**: Create SQLAlchemy models for User, Category, Expense, Budget
2. **Authentication**: Implement user registration and login endpoints
3. **Core APIs**: Build CRUD operations for expenses and categories
4. **Frontend Components**: Create basic UI components for expense management
5. **Testing**: Add unit tests for core functionality

Your project structure is now ready for development! The next phase would be implementing the core models and authentication system.
