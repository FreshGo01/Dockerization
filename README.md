# Dockerization

## Day 2
### Example Dockerfile for simple Python
- Dockerfile
```
FROM python:3.8-alpine

COPY . /python_app

WORKDIR /python_app

RUN pip install --no-cache-dir -r requirments.txt

ENV VERSION 1.0

EXPOSE 8081

CMD ["python", "/python_app/main.py"]
```

### Docker ignore for Python 
- .dockerignore
```
# Python bytecode files
__pycache__
*.pyc
*.pyo
*.pyd

# Virtual environments
venv/
.venv/
env/
*.env

# IDE/Editor files
.idea/
.vscode/
*.sublime-project
*.sublime-workspace

# System files
.DS_Store
Thumbs.db

# Log files
*.log

# Cache files
*.cache

# Test and coverage files
nosetests.xml
coverage.xml
*.cover
*.hypothesis/
*.tox/
*.nox/
*.coverage

# Python build and distribution files
build/
dist/
*.egg-info/

# Jupyter Notebook checkpoints
.ipynb_checkpoints/

# Docker specific
docker-compose.override.yml

```

### Dockerfile for Lab Day2
```
FROM ghcr.io/astral-sh/uv:python3.12-bookworm

WORKDIR /app

COPY pyproject.toml /app/
COPY uv.lock /app/

# Install the application dependencies.
RUN uv sync --frozen --no-cache

# Copy the application into the container.
COPY . /app

ENV PATH="/app/.venv/bin:$PATH"
# Run the application.
CMD ["fastapi", "run", "main.py"]
```

### docker-compose.yml for docker compose lab
```
version: '3.8' # กำหนด version ของ docker compose เราจะใช้

services:
  backend:
    build: .  # Build from Dockerfile ที่อยู่ใน directory ปัจจุบัน
    container_name: fastapi-backend  # ตั้งชื่อให้กับ container
    ports:
      - "8000:8000"  # แมปพอร์ต 8000 ของ container ไปยังพอร์ต 8000 ของเครื่อง host
    environment:
      - APP_ENV=production  # ตัวอย่างการตั้งค่า environment variable

  frontend:
    image: poonnachitdevops/workshop007-frontend:1.0  # ดึง image จาก Docker Hub
    container_name: frontend-app  # ตั้งชื่อให้กับ container
    ports:
      - "3000:80"  # แมปพอร์ต 3000 ของ container ไปยังพอร์ต 3000 ของเครื่อง host
    depends_on:
      - backend  # รัน frontend หลังจาก backend รันเสร็จ
```

### main.py edited fix CORS for docker compose lab
```
from fastapi import FastAPI
from pydantic import BaseModel
from fastapi.middleware.cors import CORSMiddleware


app = FastAPI()


origins = [
    "http://localhost",
    "http://localhost:3000",
]
app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

@app.get("/")
async def root():
    return {"message": "Hello World"}

class Item(BaseModel):
    item_id: int
    name: str
    description: str | None = None
    price: float

class ItemDto(BaseModel):
    name: str
    description: str | None = None
    price: float


items: dict[int, Item] = {}
id = 0

@app.post("/items/", response_model=Item)
def create_item(item: ItemDto):
    global id
    id += 1

    item_with_id = Item(item_id=id, **item.model_dump())
    items[id] = item_with_id
    return Item(item_id=id, **item.model_dump())


@app.get("/items/", response_model=list[Item])
def read_items():
    return list(items.values())

@app.get("/items/{item_id}", response_model=Item | dict)
def read_item(item_id: int):
    if item_id not in items:
        return {"error": "Item not found"}
    item = items[item_id]
    return item

@app.put("/items/{item_id}", response_model=Item | dict)
def update_item(item_id: int, item: ItemDto):
    if item_id not in items:
        return {"error": "Item not found"}
    updated_item = Item(item_id=item_id, **item.model_dump())
    items[item_id] = updated_item
    return updated_item

@app.delete("/items/{item_id}", response_model=dict)
def delete_item(item_id: int):
    if item_id not in items:
        return {"error": "Item not found"}
    del items[item_id]
    return {"message": "Item deleted successfully"}
```
