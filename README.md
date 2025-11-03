# LMS_INTRA

A comprehensive Learning Management System with microservices architecture, featuring Django REST backend, FastAPI authentication service, and Next.js frontend.

## 🏗️ Architecture

The system consists of three main components:

- **Astra-learn**: Django REST Framework backend for LMS functionality
- **GLOBAL-AUTH**: FastAPI-based authentication and authorization service with RBAC
- **LMS_FRONT**: Next.js frontend application

## 📋 Prerequisites

- Docker and Docker Compose
- Git

## 🚀 Quick Start

### 1. Clone the Repository

Clone this repository and fetch its submodules in one step:

```bash
git clone --recurse-submodules <this-repo-url>
cd LMS_INTRA
```

If you already cloned without submodules:

```bash
git submodule update --init --recursive
```


### 2. Initialize the Project

Use the convenient Docker management script:

```bash
# Full reset: removes containers, volumes, node_modules, migrations, and database, then rebuilds and seeds everything
./docker.sh init

# Or step by step:
./docker.sh build       # Build all containers
./docker.sh up          # Start all services
./docker.sh migrate     # Run database migrations
./docker.sh seed-all    # Create admin user + seed demo data (single script)
```

### 3. Access the Application

Once initialized, the services are available at:

| Service | URL | Description |
|---------|-----|-------------|
| **Frontend** | http://localhost:3000 | Next.js application |
| **Django API** | http://localhost:8000 | REST API endpoints |
| **Django Admin** | http://localhost:8000/admin | Admin panel |
| **GLOBAL-AUTH** | http://localhost:8001 | Authentication API |
| **GLOBAL-AUTH** | http://localhost:8001/api/docs/ | 
| **API Docs** | http://localhost:8000/api/docs/ Swagger documentation |

## 🔐 Default Credentials

All users have the same password for development: **`Password123`**

### Admin Account
- **Email**: `admin@um6p.ma`
- **Username**: `admin`
- **Role**: SuperUser
- **Access**: Django Admin, Full API access

### Other Accounts

| Username | Email | Role | Access |
|----------|-------|------|--------|
| instructor | instructor@um6p.ma | Staff | Course management |
| sally | sally@um6p.ma | Student | Enrolled in courses |
| sam | sam@um6p.ma | Student | Invited to courses |
| econ_instructor | econ@um6p.ma | Instructor | Created ECON101 |


## 📚 Seeded Demo Data

All demo data and admin user creation is handled by a single script: `Astra-learn/common/seed_all.py`.
After running `./docker.sh seed-all` or `./docker.sh init`, the database includes:

### Courses
1. **DJ101: Intro to Django** (2 weeks, 4 lessons)
   - Week 1: Getting Started
   - Week 2: Core Concepts
   - 2 enrolled students

2. **DS201: Data Science Foundations** (1 week, 2 lessons)
   - Week 1: Foundations
   - 1 enrolled student

3. **ECON101: Microeconomics** (14 weeks, 14 lessons)
   - Complete microeconomics curriculum
   - Topics: Supply/Demand, Consumer Behavior, Market Structures, etc.

### Additional Data
- **Assignments**: 2 assignments with submissions and grades
- **Quiz Links**: 1 quiz integration
- **Events**: 2 calendar events
- **Enrollments**: Students enrolled in courses

## 🛠️ Docker Management Script

The `docker.sh` script provides easy management of all services:

### Setup Commands
```bash
./docker.sh init              # Initialize project (build, migrate, seed)
./docker.sh build             # Build all Docker containers
./docker.sh up                # Start all containers
./docker.sh down              # Stop all containers
./docker.sh restart           # Restart all containers
./docker.sh clean             # Stop and remove volumes (deletes data)
```

### Database Commands
```bash
./docker.sh migrate           # Run Django migrations
./docker.sh makemigrations    # Create new migrations
./docker.sh createsuperuser   # Create Django superuser
./docker.sh seed-all          # Create admin + seed all demo data
./docker.sh db-summary        # Show database statistics
./docker.sh dbshell           # Open database shell
./docker.sh resetdb           # Reset database (deletes all data)
```

### Backend Commands
```bash
./docker.sh shell             # Open Django shell
./docker.sh bash              # Open bash in backend container
./docker.sh logs              # Show backend logs
./docker.sh test              # Run backend tests
```

### Global Auth Commands
```bash
./docker.sh auth-logs         # Show GLOBAL-AUTH logs
./docker.sh auth-bash         # Open bash in GLOBAL-AUTH container
./docker.sh create-user       # Create a test user in GLOBAL-AUTH
```

### Frontend Commands
```bash
./docker.sh front-logs        # Show frontend logs
./docker.sh front-bash        # Open bash in frontend container
```

### Utility Commands
```bash
./docker.sh ps                # Show running containers
./docker.sh status            # Show detailed status of all services
./docker.sh logs-all          # Show logs from all containers
./docker.sh help              # Show help message
```

## 🔑 Authentication System

The system uses **GLOBAL-AUTH** for centralized authentication with session-based auth:

### How It Works
1. **Frontend** sends login request to GLOBAL-AUTH (port 8001)
2. **GLOBAL-AUTH** validates credentials and creates session
3. Session cookie (`session_id`) is stored in Redis
4. **Astra-learn backend** validates session with GLOBAL-AUTH for API requests
5. **Django Admin** uses local database authentication

### Role-Based Access Control (RBAC)

The system supports 5 user roles:
- **SuperUser**: Full system access
- **Admin**: Administrative privileges
- **Professor**: Course management and grading
- **Staff**: Limited administrative access
- **Student**: Course enrollment and content access

### API Authentication

DRF views use `GlobalAuthBackend` which validates sessions with GLOBAL-AUTH:

```python
from common.permissions import IsAuthenticated, IsProfessor, IsStudent

class MyCourseView(APIView):
    authentication_classes = [GlobalAuthBackend]
    permission_classes = [IsAuthenticated, IsProfessor]
```

### Django Admin Authentication

Django admin uses `EmailBackend` which allows login with email or username:

```python
# settings.py
AUTHENTICATION_BACKENDS = [
    'common.authentication.EmailBackend',  # Supports email and username
]
```

## 🗄️ Database Information

### View Database Summary
```bash
./docker.sh db-summary
```

Output example:
```
📊 DATABASE STATISTICS
👥 Users:        6
📚 Courses:      3
📅 Weeks:        17
📝 Lessons:      20
📋 Assignments:  2
📤 Submissions:  2
🎯 Grades:       2
🧪 Quiz Links:   1
📆 Events:       2
```

### Database Structure

The system uses PostgreSQL with the following main models:

#### Accounts App
- User (extends Django User)
- CourseRole (Professor, TA assignments)

#### Courses App
- Course (main course entity)
- Week (course weeks/sections)
- Lesson (individual lessons)
- Enrollment (student enrollments)
- LessonCompletion (progress tracking)

#### Assessments App
- Assignment
- Submission
- Rubric
- AssessmentHistory

#### Grades App
- GradeItem
- Grade

#### Quiz Integration App
- QuizLink (external quiz integration)
- QuizAttemptMap

#### Timetable App
- Event (calendar events)

## 🧪 Testing

Run backend tests:
```bash
./docker.sh test
```

## 📝 Development Workflow

### Making Database Changes

1. Modify models in Django apps
2. Create migrations:
   ```bash
   ./docker.sh makemigrations
   ```
3. Apply migrations:
   ```bash
   ./docker.sh migrate
   ```


### Adding or Modifying Demo Data

To add or change demo data, edit the unified seeding script:
- `Astra-learn/common/seed_all.py`

Then run:
```bash
./docker.sh seed-all
```


### Resetting Everything

To start from a truly clean state (removes containers, volumes, node_modules, migrations, and database, then rebuilds and reseeds):
```bash
./docker.sh init           # Full reset and initialize (recommended)
```
Or, to just remove containers and volumes:
```bash
./docker.sh clean          # Remove all containers and volumes (data only)
```
## 📝 Notes

- All demo data and admin user creation is now handled by a single file: `Astra-learn/common/seed_all.py`.
- The `init` command is the recommended way to fully reset and reinitialize your development environment.
- If you encounter issues or want a truly clean state, always use `./docker.sh init`.

## 🔧 Configuration

### Environment Variables

Key environment variables are configured in `docker-compose.yml`:

#### Astra-learn Backend
- `GLOBAL_AUTH_URL`: http://fastapi-app:80 (internal Docker network)
- `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASSWORD`: PostgreSQL settings

#### GLOBAL-AUTH
- `DATABASE_URL`: PostgreSQL connection string
- `REDIS_URL`: Redis connection for sessions
- `SECRET_KEY`: JWT/session encryption key

### Ports

| Service | Internal Port | External Port |
|---------|---------------|---------------|
| Frontend | 3000 | 3000 |
| Astra-learn | 8000 | 8000 |
| GLOBAL-AUTH | 80 | 8001 |
| PostgreSQL (Astra) | 5432 | 5432 |
| PostgreSQL (Auth) | 5432 | 5433 |
| Redis | 6379 | 6379 |

## 🔄 Updating Submodules

To pull the latest commits from each submodule:

```bash
git pull --recurse-submodules
git submodule update --init --recursive
```

To update submodules to their latest tracked branch:

```bash
git submodule update --remote --merge --recursive
```

## 📖 Additional Documentation

For more detailed information, see:
- **Astra-learn**: `Astra-learn/README.md`
- **GLOBAL-AUTH**: `GLOBAL-AUTH/AUTHENTICATION_GUIDE.md`
- **Frontend**: `LMS_FRONT/README.md`

## 🐛 Troubleshooting

### Containers won't start
```bash
./docker.sh logs-all    # Check logs
./docker.sh clean       # Clean everything
./docker.sh init        # Rebuild
```

### Database issues
```bash
./docker.sh resetdb     # Reset database
./docker.sh migrate     # Run migrations
./docker.sh seed-all    # Reseed data
```

### Authentication issues
```bash
./docker.sh auth-logs   # Check GLOBAL-AUTH logs
```

### Can't login to Django admin
- Make sure you're using `admin@um6p.ma` or username `admin`
- Password is `Password123`
- If still failing, run `./docker.sh seed-all` to reset credentials

## 🤝 Contributing

1. Create a feature branch
2. Make your changes
3. Test thoroughly using `./docker.sh test`
4. Submit a pull request
