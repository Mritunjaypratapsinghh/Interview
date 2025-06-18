# FastAPI Interview Questions and Answers

## Beginner Level

### 1. What is FastAPI? Why is it popular?

FastAPI is a modern Python web framework for building APIs, based on Starlette (for web handling) and Pydantic (for data validation). It is known for:

* High performance (comparable to Node.js and Go)
* Automatic OpenAPI docs
* Data validation using type hints

### 2. How do you define a simple GET and POST endpoint?

```python
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"message": "Hello World"}

@app.post("/submit")
def submit_data(data: dict):
    return {"received": data}
```

### 3. What is Pydantic and how is it used in FastAPI?

Pydantic is a data validation library used by FastAPI to validate request and response data based on Python type hints.

```python
from pydantic import BaseModel

class User(BaseModel):
    name: str
    age: int

@app.post("/user")
def create_user(user: User):
    return user
```

### 4. How do you return a JSON response in FastAPI?

FastAPI automatically serializes dictionaries to JSON:

```python
@app.get("/json")
def get_json():
    return {"message": "Hello JSON"}
```

### 5. Difference between @app.get() and @app.post()

* `@app.get()` is used to fetch/read data
* `@app.post()` is used to send/create data

### 6. How do you add path and query parameters?

```python
@app.get("/items/{item_id}")
def read_item(item_id: int, q: str = None):
    return {"item_id": item_id, "q": q}
```

### 7. How does FastAPI perform automatic validation?

* It uses Pydantic models and Python type hints to enforce types
* Returns 422 error for invalid input

### 8. Benefits of type hints in FastAPI

* Auto validation
* Editor autocomplete support
* Documentation generation

### 9. Serving static files

```python
from fastapi.staticfiles import StaticFiles

app.mount("/static", StaticFiles(directory="static"), name="static")
```

### 10. Adding middleware

```python
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class CustomMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        return response

app.add_middleware(CustomMiddleware)
```

## Intermediate Level

### 1. Handling form data and file uploads

```python
from fastapi import Form, File, UploadFile

@app.post("/form")
def handle_form(username: str = Form(...)):
    return {"username": username}

@app.post("/upload")
def upload_file(file: UploadFile = File(...)):
    return {"filename": file.filename}
```

### 2. Nested Pydantic validation

```python
class Address(BaseModel):
    city: str
    zip: str

class User(BaseModel):
    name: str
    address: Address
```

### 3. Dependency Injection with Depends()

```python
from fastapi import Header, Depends

def get_token(token: str = Header(...)):
    return token

@app.get("/secure")
def secure(token=Depends(get_token)):
    return {"token": token}
```

### 4. Environment variables and config

```python
from pydantic import BaseSettings

class Settings(BaseSettings):
    db_url: str

settings = Settings()
```

### 5. JWT Authentication

```python
from jose import jwt

SECRET_KEY = "secret"

def create_token(data: dict):
    return jwt.encode(data, SECRET_KEY, algorithm="HS256")

def verify_token(token: str):
    return jwt.decode(token, SECRET_KEY, algorithms=["HS256"])
```

### 6. Depends() vs normal parameters

* Normal parameters are used for query/path/body data
* `Depends()` is used for injecting dependencies (auth, DB, etc.)

### 7. Background tasks

```python
from fastapi import BackgroundTasks

def notify_user(email: str):
    print(f"Email sent to {email}")

@app.post("/send-notification")
def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(notify_user, email)
    return {"message": "Notification sent"}
```

### 8. API Versioning

```python
from fastapi import APIRouter

v1 = APIRouter(prefix="/v1")
v2 = APIRouter(prefix="/v2")

@app.get("/v1/items")
def v1_items(): ...

@app.get("/v2/items")
def v2_items(): ...
```

### 9. Exception handling

```python
from fastapi.responses import JSONResponse
from fastapi.exceptions import RequestValidationError

@app.exception_handler(RequestValidationError)
def validation_exception_handler(request, exc):
    return JSONResponse(status_code=400, content={"error": str(exc)})
```

### 10. Testing with TestClient

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_home():
    response = client.get("/")
    assert response.status_code == 200
```

## Advanced Level

### 1. Async I/O and concurrency

* Use `async def` for I/O-bound functions
* Uses Python's asyncio event loop

### 2. Database integration

```python
from sqlalchemy.orm import Session

def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### 3. Role-based access control (RBAC)

* Check roles in dependency
* Add role attribute to user model

### 4. Scaling FastAPI

* Run with Uvicorn + Gunicorn: `gunicorn -w 4 -k uvicorn.workers.UvicornWorker main:app`
* Add Docker, Nginx, Celery, etc.

### 5. WebSocket support

```python
@app.websocket("/ws")
async def websocket_endpoint(websocket: WebSocket):
    await websocket.accept()
    await websocket.send_text("Hello")
```

### 6. Optimize Pydantic performance

* Use `Config.orm_mode = True`
* Use `json_encoders` to speed up encoding

### 7. Sub-applications

```python
from fastapi import FastAPI

sub_app = FastAPI()

@sub_app.get("/")
def sub_home(): return {"msg": "sub"}

app.mount("/sub", sub_app)
```

### 8. Limitations of FastAPI

* Complex async DB setup
* Not best for CPU-bound tasks
* Cold starts in serverless can be slow

### 9. Custom OpenAPI docs

```python
from fastapi.openapi.utils import get_openapi

def custom_openapi():
    if app.openapi_schema:
        return app.openapi_schema
    openapi_schema = get_openapi(
        title="Custom API",
        version="2.5.0",
        description="My custom docs",
        routes=app.routes,
    )
    app.openapi_schema = openapi_schema
    return app.openapi_schema

app.openapi = custom_openapi
```

### 10. Reusable services with Depends()

```python
# db.py
def get_db():
    ...

# services.py
def get_current_user(db=Depends(get_db)):
    ...
```

# Bonus: System Design with FastAPI

## How would you structure a scalable FastAPI application?

### Recommended Folder Structure

```
app/
├── main.py              # Entry point
├── api/                 # Routers
│   ├── v1/
│   │   ├── routes/
│   │   │   ├── user.py
│   │   │   └── item.py
│   │   └── api.py
├── core/                # Configuration, startup events
│   ├── config.py
│   └── security.py
├── models/              # SQLAlchemy ORM models
│   ├── user.py
│   └── item.py
├── schemas/             # Pydantic models
│   ├── user.py
│   └── item.py
├── services/            # Business logic
│   ├── user_service.py
│   └── item_service.py
├── db/                  # Database session management
│   ├── session.py
│   └── base.py
└── utils/               # Helpers/utilities
```

### Benefits

* Separation of concerns (routes, logic, DB, validation)
* Scalable for versioning, microservices, testing
* Easier onboarding and collaboration

---

## How would you deploy a FastAPI app on AWS / GCP / Heroku?

### Common Steps

1. **Dockerize the app**

```dockerfile
# Dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```

2. **Deployment Targets**

* **Heroku**: Use `heroku container:push web -a app-name`
* **GCP**: Use Google Cloud Run or App Engine
* **AWS**:

  * Elastic Beanstalk (easiest)
  * ECS with Fargate (scalable)
  * Lambda (serverless with Mangum wrapper)

3. **CI/CD**: GitHub Actions or GitLab CI for build, test, and deploy pipelines.

---

## How do you implement logging and monitoring in FastAPI?

### Logging

```python
import logging

logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s'
)
logger = logging.getLogger(__name__)

@app.get("/test")
def test():
    logger.info("Test endpoint hit")
    return {"message": "logged"}
```

### Monitoring

* **Prometheus + Grafana**:

  * Use `prometheus_fastapi_instrumentator` for metrics

```python
from prometheus_fastapi_instrumentator import Instrumentator

instrumentator = Instrumentator()
instrumentator.instrument(app).expose(app)
```

* **Sentry** for error tracking:

```python
import sentry_sdk
from sentry_sdk.integrations.asgi import SentryAsgiMiddleware

sentry_sdk.init(dsn="<your_sentry_dsn>")
app.add_middleware(SentryAsgiMiddleware)
```

* **Uvicorn logs**: Use `--log-level info` in production

---

This setup ensures modularity, observability, and cloud readiness for large-scale FastAPI applications.

