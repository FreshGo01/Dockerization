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
