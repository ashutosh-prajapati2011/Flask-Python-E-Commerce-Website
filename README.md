# Python Flask E-Commerce Website

A simple full-stack e-commerce website built using Flask (Python), Jinja2, SQLite, jQuery, Bootstrap, and Docker. It features product listings, shopping cart functionality, and category-based filtering. The database (SQLite) is used to store clothing item data, and the UI is styled using Bootstrap's responsive cards.

---

## 🔄 Project Fork and Setup

This project was cloned from the original GitHub repository:
[https://github.com/haxamxam/Flask-Python-E-Commerce](https://github.com/haxamxam/Flask-Python-E-Commerce)

---

## 🛠 Tech Stack

- **Frontend:** HTML, CSS, Bootstrap, jQuery
- **Backend:** Python Flask, Jinja2
- **Database:** SQLite
- **DevOps:** Docker, GitHub Actions, Nginx, AWS EC2, Kubernetes

---

## 💻 Local Installation

```bash
pip install -r requirements.txt
python wsgi.py
```

Visit: `http://localhost:5000`

---

## 🐳 Docker Setup

### Dockerfile
```Dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . /app
RUN pip install -r requirements.txt
CMD ["python", "wsgi.py"]
```

### Docker Commands
```bash
docker build -t flask-ecommerce .
docker run -d -p 5000:5000 flask-ecommerce
```

---

## 🐙 GitHub Actions CI/CD Workflow

### `.github/workflows/deploy.yml`
```yaml
name: CI/CD - Flask E-Commerce App

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-python@v4
        with:
          python-version: "3.9"
      - run: |
          python -m pip install --upgrade pip
          pip install -r requirements.txt
      - run: |
          python wsgi.py &
          sleep 5
          curl --fail http://localhost:5000 || exit 1

  deploy:
    needs: build-test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - run: |
          echo "${{ secrets.EC2_KEY }}" > key.pem
          chmod 600 key.pem
      - run: |
          ssh -o StrictHostKeyChecking=no -i key.pem ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }} << 'EOF'
            cd ${{ secrets.APP_DIR }}
            git pull origin main
            pip install -r requirements.txt
            pkill -f wsgi.py || true
            nohup python3 wsgi.py > app.log 2>&1 &
          EOF
```

---

## ☸ Kubernetes Deployment

### 1. Docker Image Push (example)
```bash
docker build -t your-dockerhub-username/flask-ecommerce .
docker push your-dockerhub-username/flask-ecommerce
```

### 2. `deployment.yaml`
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-ecommerce
spec:
  replicas: 1
  selector:
    matchLabels:
      app: flask-ecommerce
  template:
    metadata:
      labels:
        app: flask-ecommerce
    spec:
      containers:
      - name: flask-ecommerce
        image: your-dockerhub-username/flask-ecommerce
        ports:
        - containerPort: 5000
```

### 3. `service.yaml`
```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  type: LoadBalancer
  selector:
    app: flask-ecommerce
  ports:
    - port: 80
      targetPort: 5000
```

---

## 🔍 Features

- Product listing and filtering
- Shopping cart
- SQLite database
- Responsive UI with Bootstrap
- CI/CD with GitHub Actions
- EC2 deployment using SSH
- Dockerized application
- Kubernetes-ready

---

## 📄 License
[MIT](https://choosealicense.com/licenses/mit/)

