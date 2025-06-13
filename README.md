# Dockerization

## Day 1
### Dockerfile
Example Dockerfile for simple Python
```
FROM python:3.8-alpine

COPY . /python_app

WORKDIR /python_app

RUN pip install --no-cache-dir -r requirments.txt

ENV VERSION 1.0

EXPOSE 8081

CMD ["python", "/python_app/main.py"]
```
