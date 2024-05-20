<h1>DevSecOps Project: Netflix Clone</h1>

<h2>Description</h2>
<b>The Powershell script in this repository sets up a DevSecOps pipeline for a Netflix clone application, including security scanning, CI/CD with Jenkins, monitoring using Prometheus and Grafana, and Kubernetes deployment.</b>

<h2>Languages and Utilities Used</h2>

- <b>PowerShell:</b> Scripting for automation
- <b>Jenkins:</b> Continuous Integration and Continuous Deployment (CI/CD)
- <b>Docker:</b> Containerization
- <b>Kubernetes:</b> Container Orchestration
- <b>Prometheus:</b> Monitoring and Alerting
- <b>Grafana:</b> Visualization and Dashboards
- <b>SonarQube:</b> Static Code Analysis
- <b>OWASP Dependency-Check:</b> Dependency Vulnerability Scanning
- <b>Trivy:</b> Container Image Vulnerability Scanning

![image](https://github.com/michaelmorley1/DevSecOps-Project/assets/39282112/541655e0-c6f6-46d5-9846-82b73fe6f4d6)


<h2>Steps Overview</h2>
1. Initial Setup and Deployment
2. Security Scanning (OWASP and Trivy)
3. Continuous Integration and Continuous Deployment (CI/CD) with Jenkins
4. Monitoring (Prometheus and Grafana)
5. Kubernetes Setup

<h2>Detailed Steps</h2>

<h3>Initial Setup and Deployment</h3>

<h4>Step 1: Launch EC2 Instance</h4>
- Added security group rules for all applications

<h4>Step 2: Clone Netflix Application Code</h4>
<pre><code>git clone https://github.com/N4si/DevSecOps-Project.git</code></pre>

<h4>Step 3: Install Docker</h4>
<pre><code>sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER
newgrp docker
sudo chmod 777 /var/run/docker.sock</code></pre>

<h4>Step 4: Create Dockerfile</h4>
<pre><code>FROM node:16.17.0-alpine as builder
WORKDIR /app
COPY ./package.json .
COPY ./yarn.lock .
RUN yarn install
COPY . .
ARG TMDB_V3_API_KEY
ENV VITE_APP_TMDB_V3_API_KEY=${TMDB_V3_API_KEY}
ENV VITE_APP_API_ENDPOINT_URL="https://api.themoviedb.org/3"
RUN yarn build

FROM nginx:stable-alpine
WORKDIR /usr/share/nginx/html
RUN rm -rf ./*
COPY --from=builder /app/dist .
EXPOSE 80
ENTRYPOINT ["nginx", "-g", "daemon off;"]</code></pre>

<h4>Step 5: Get the API Key</h4>
- Get the API key from [The Movie Database (TMDb)](https://www.themoviedb.org/)

<h4>Step 6: Build Docker Image</h4>
<pre><code>docker build --build-arg TMDB_V3_API_KEY=&lt;your_api_key&gt; -t netflix .</code></pre>

<h4>Step 7: Install SonarQube and Trivy</h4>
<pre><code># Install SonarQube
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

# Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy</code></pre>

<h3>Security Scanning (OWASP and Trivy)</h3>

<h4>Step 13: Configure CI/CD Pipeline in Jenkins</h4>
<pre><code>pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME=tool 'sonar-scanner'
    }
    stages {
        // ... Pipeline stages ...
    }
}</code></pre>
