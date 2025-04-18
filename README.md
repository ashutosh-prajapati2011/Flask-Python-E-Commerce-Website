# 🏍️ Flask E-Commerce Web Application

A simple and fully functional **E-Commerce web app** developed using **Python Flask** for backend and **Bootstrap, jQuery** for frontend. Products are stored in a **SQLite** database and support features like product listing, category-based filtering, and shopping cart.

> ✅ This repo was originally cloned from [haxamxam/Flask-Python-E-Commerce](https://github.com/haxamxam/Flask-Python-E-Commerce)

---

## ⚙️ Tech Stack

- **Frontend**: HTML, CSS, Bootstrap, jQuery
- **Backend**: Python Flask, Jinja2
- **Database**: SQLite
- **DevOps**: GitHub Actions, Docker, Nginx, EC2, Kubernetes
- **Monitoring**: Prometheus, Grafana

---

## ✨ Features

- Product catalog with filters (Shirts, Pants, Shoes, etc.)
- Add to cart functionality
- Category-based filtering
- Responsive UI using Bootstrap
- SQLite-backed dynamic data

---

## 🧱 Local Setup

```bash
# Clone the repo
git clone https://github.com/haxamxam/Flask-Python-E-Commerce.git
cd Flask-Python-E-Commerce

# Install dependencies
pip install -r requirements.txt

# Run the app
python wsgi.py
```

Visit: `http://localhost:5000`

---

## 🐳 Docker Setup

**Dockerfile**

```dockerfile
FROM python:3.9-slim
WORKDIR /app
COPY . .
RUN pip install --upgrade pip && pip install -r requirements.txt
EXPOSE 5000
CMD ["python", "wsgi.py"]
```

```bash
docker build -t flask-ecom .
docker run -p 5000:5000 flask-ecom
```

---

## 📦 Docker Compose Setup (Frontend + Backend)

```yaml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    restart: always
```

```bash
docker-compose up -d
```

---

## 🔄 GitHub Actions CI/CD Workflow

**.github/workflows/main.yml**

```yaml
name: CI/CD - Flask E-Commerce App

on:
  push:
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
          pip install -r requirements.txt
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
          ssh -o StrictHostKeyChecking=no -i key.pem ${{ secrets.EC2_USER }}@${{ secrets.EC2_HOST }} << 'EOF'
            cd ${{ secrets.APP_DIR }}
            git pull origin main
            pip install -r requirements.txt
            pkill -f wsgi.py || true
            nohup python3 wsgi.py > app.log 2>&1 &
          EOF
```

---

## ☁️ EC2 Setup Steps

```bash
# Login to EC2
ssh -i "key.pem" ubuntu@your-ec2-ip

# Pull the repo
git clone https://github.com/haxamxam/Flask-Python-E-Commerce.git
cd Flask-Python-E-Commerce

# Run the app
pip install -r requirements.txt
nohup python3 wsgi.py &
```

---

## ☘️ Kubernetes Deployment

**deployment.yaml**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: flask-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: flask-ecom
  template:
    metadata:
      labels:
        app: flask-ecom
    spec:
      containers:
        - name: flask-app
          image: your-dockerhub-username/flask-ecom
          ports:
            - containerPort: 5000
```

**service.yaml**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: flask-service
spec:
  selector:
    app: flask-ecom
  ports:
    - protocol: TCP
      port: 80
      targetPort: 5000
  type: LoadBalancer
```

---

## 📊 Monitoring with Prometheus + Grafana

### Step 1: Install Prometheus and Grafana

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack
```

### Step 2: Expose Grafana Dashboard

```bash
kubectl port-forward svc/monitoring-grafana 3000:80
```

Visit `http://localhost:3000`  
Login with `admin/prom-operator`

### Step 3: Add Metrics in Flask App

```bash
pip install prometheus_flask_exporter
```

```python
from prometheus_flask_exporter import PrometheusMetrics
metrics = PrometheusMetrics(app)
```

Now metrics are available at `/metrics`

### Step 4: ServiceMonitor for Prometheus

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: flask-monitor
  labels:
    release: monitoring
spec:
  selector:
    matchLabels:
      app: flask-ecom
  endpoints:
    - port: http
      path: /metrics
      interval: 15s
```

---

## 📃 License

This project is licensed under the MIT License.

---

Let me know if you want:
- Ingress + TLS (SSL)
- HPA for autoscaling
- Full k8s folder structure

