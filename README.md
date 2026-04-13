<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">

</head>
<body>

<h1>🚀 Project 5 — Python App + Ingress (Real Routing)</h1>

<p>This is where Kubernetes starts to feel like <b>real production infrastructure</b> 🔥</p>

<p>Project 5 is a big jump — moving from <b>exposing services</b> to <b>real-world routing using Ingress</b>.</p>

<hr>

<h2>🎯 Goal (Real DevOps Context)</h2>
<ul>
  <li>Run a <b>Python Flask application</b></li>
  <li>Expose it using <b>Ingress (NOT NodePort)</b></li>
  <li>Access via a <b>custom domain</b></li>
</ul>

<p>👉 This is how production systems work (Nginx / AWS ALB / Traefik)</p>

<hr>

<h2>🧠 Architecture</h2>

<p><b>Flow:</b></p>
<pre>
User → Ingress → Service → Pod (Python App)
</pre>

<hr>

<h2>📁 Project Structure</h2>

<pre>
kubernetes-project5/
│
├── app/
│   ├── app.py
│   └── requirements.txt
│
├── Dockerfile
│
├── k8s/
│   ├── deployment.yaml
│   ├── service.yaml
│   └── ingress.yaml
│
└── README.md
</pre>

<hr>

<h2>🧩 Step 1 — Python Application</h2>

<h3>📄 app/app.py</h3>
<pre>
from flask import Flask

app = Flask(__name__)

@app.route("/")
def home():
    return "Hello from Kubernetes Ingress 🚀"

@app.route("/health")
def health():
    return "OK"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
</pre>

<h3>📄 requirements.txt</h3>
<pre>
flask
</pre>

<hr>

<h2>🐳 Step 2 — Dockerfile</h2>
<pre>
FROM python:3.9-slim

WORKDIR /app

COPY app/ .

RUN pip install -r requirements.txt

EXPOSE 5000

CMD ["python", "app.py"]
</pre>

<hr>

<h2>⚙️ Step 3 — Build Image in Minikube</h2>
<pre>
eval $(minikube docker-env)
docker build -t python-ingress-app:v1 .
</pre>

<hr>

<h2>🚀 Step 4 — Deployment</h2>

<h3>📄 k8s/deployment.yaml</h3>
<pre>
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-ingress-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: python-ingress
  template:
    metadata:
      labels:
        app: python-ingress
    spec:
      containers:
      - name: python-container
        image: python-ingress-app:v1
        ports:
        - containerPort: 5000
</pre>

<hr>

<h2>🌐 Step 5 — Service (ClusterIP)</h2>

<h3>📄 k8s/service.yaml</h3>
<pre>
apiVersion: v1
kind: Service
metadata:
  name: python-service
spec:
  type: ClusterIP
  selector:
    app: python-ingress
  ports:
    - port: 80
      targetPort: 5000
</pre>

<p><b>Note:</b> Not using NodePort. Ingress handles exposure.</p>

<hr>

<h2>🌍 Step 6 — Enable Ingress</h2>

<pre>
minikube addons enable ingress
kubectl get pods -n ingress-nginx
</pre>

<hr>

<h2>🌐 Step 7 — Ingress Configuration</h2>

<h3>📄 k8s/ingress.yaml</h3>
<pre>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: python-ingress
spec:
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: python-service
            port:
              number: 80
</pre>

<hr>

<h2>⚙️ Step 8 — Apply Resources</h2>

<pre>
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
kubectl apply -f k8s/ingress.yaml
</pre>

<hr>

<h2>🌐 Step 9 — Access Application</h2>

<h3>Get Minikube IP</h3>
<pre>
minikube ip
</pre>

<h3>Edit Hosts File</h3>

<p><b>Linux/Mac:</b></p>
<pre>sudo nano /etc/hosts</pre>

<p><b>Windows:</b></p>
<pre>C:\Windows\System32\drivers\etc\hosts</pre>

<p>Add:</p>
<pre>
192.168.49.2 myapp.local
</pre>

<hr>

<h2>🌐 Open in Browser</h2>
<pre>http://myapp.local</pre>

<p>You should see:</p>
<pre>Hello from Kubernetes Ingress 🚀</pre>

<hr>

<h2>🧪 Testing</h2>

<p><b>Health Check:</b></p>
<pre>http://myapp.local/health</pre>

<p><b>Scale Application:</b></p>
<pre>
kubectl scale deployment python-ingress-app --replicas=5
</pre>

<p>👉 Traffic will be load-balanced automatically.</p>

<hr>

<h2>🧠 What You Learned</h2>
<ul>
  <li>Ingress Controller (Nginx)</li>
  <li>Domain-based routing</li>
  <li>ClusterIP vs NodePort</li>
  <li>Real-world Kubernetes traffic flow</li>
</ul>

<hr>

<h2>❌ Common Mistakes</h2>
<ul>
  <li>Forgetting to enable ingress</li>
  <li>Wrong service name in ingress</li>
  <li>Using NodePort instead of ClusterIP</li>
  <li>Hosts file not updated</li>
</ul>

<hr>

<h2>🚀 Upgrade Ideas</h2>
<ul>
  <li>Add multiple routes (/api, /app)</li>
  <li>Add TLS (HTTPS)</li>
  <li>Use multiple services</li>
  <li>Add rate limiting</li>
</ul>

<hr>

<h2>🎯 Interview Question</h2>

<p><b>Why use Ingress instead of NodePort?</b></p>

<blockquote>
Ingress provides centralized routing, supports domains, SSL, and path-based routing.
NodePort is not production-friendly.
</blockquote>

<hr>

<h2>🧭 Your Task</h2>
<ol>
  <li>Push code to repository</li>
  <li>Build image in Minikube</li>
  <li>Deploy all resources</li>
  <li>Configure hosts file</li>
  <li>Test in browser</li>
</ol>

<hr>

<p>🔥 <b>This project simulates real production-grade Kubernetes routing.</b></p>

</body>
</html>
