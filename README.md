# CloudComputingITCS-6190-L3
Hands-on L3: Containers with Docker

## Project Overview

This project demonstrates how to containerize a Python Flask application using Docker and Docker Compose, including a Redis caching service and a PostgreSQL database container. The app counts and displays the number of times it has been accessed. 

**(Screenshots are uploaded below)**

## Execution Steps

1. **Install Docker Desktop on macOS**  
   Download and install the appropriate Docker Desktop for your chip type (Apple Silicon or Intel). Launch Docker.app and verify installation by running:
   > docker --version
   
2. **Pull PostgreSQL Image and Run Container**
   > docker pull postgres
   > docker run -d -p 5432:5432 --name postgres1 -e POSTGRES_PASSWORD=pass12345 postgres
  
3. **Access PostgreSQL Container**
   > docker exec -it postgres1 bash
   > psql -d postgres -U postgres

4. **Prepare Flask Application**
- Create `requirements.txt` with:
  ```
  flask
  redis
  ```
- Write `app.py`:
  ```
  import time
  import redis
  from flask import Flask

  app = Flask(__name__)
  cache = redis.Redis(host='redis', port=6379)

  def get_hit_count():
      retries = 5
      while True:
          try:
              return cache.incr('hits')
          except redis.exceptions.ConnectionError as exc:
              if retries == 0:
                  raise exc
              retries -= 1
              time.sleep(0.5)

  @app.route('/')
  def hello():
      count = get_hit_count()
      return f'Hello World! I have been seen {count} times.\n'
  ```

5. **Create Dockerfile**
   ```
   FROM python:3.7-alpine
   WORKDIR /code
   ENV FLASK_APP=app.py
   ENV FLASK_RUN_HOST=0.0.0.0
   RUN apk add --no-cache gcc musl-dev linux-headers
   COPY requirements.txt requirements.txt
   RUN pip install -r requirements.txt
   EXPOSE 5000
   COPY . .
   CMD ["flask", "run"]
   ```

6. **Define Docker Compose (`compose.yaml`)**
   ```
   version: "3.9"
   services:
     web:
       build: .
       ports: 
         - "8000:5000"
       depends_on: 
         - redis
      redis:
        image: "redis:alpine"
   ```
7. **Build and Run Application**
   > docker compose up
   
Access the app in a browser at [http://localhost:8000](http://localhost:8000)

## What I Learned

- **Containerization**: Learned how Docker packages an application and its dependencies into isolated containers for consistent environments.
- **Service Orchestration with Docker Compose**: Gained experience running multi-service applications where a web server depends on a cache service.
- **Error Handling**: Added retry logic in the Flask app to handle Redis connection errors robustly.
- **YAML Configuration**: Experienced the importance of proper YAML indentation and structure to avoid docker-compose errors.
- **Project Documentation**: Practiced writing clear, well-formatted Markdown README files with code blocks and execution instructions.
- **GitHub Usage**: Managed project files, error logs, and documentation within a GitHub repository for easy collaboration and submission.

<img width="1470" height="956" alt="Screenshot 2025-09-02 at 7 48 19 PM" src="https://github.com/user-attachments/assets/e56001a1-e972-4797-adbc-2a75c4f3a7d1" />


<img width="1470" height="956" alt="Screenshot 2025-09-02 at 7 49 41 PM" src="https://github.com/user-attachments/assets/e6e9319b-6397-45fb-ba21-dc41aac1f854" />
