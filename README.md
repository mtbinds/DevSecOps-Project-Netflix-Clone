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

-Launch an AWS T2 Large Instance using the Ubuntu image.
-Configure HTTP and HTTPS settings in the security group.

<p align="center">
<img src="https://imgur.com/OVWgoVf.png" height="80%" width="80%" alt="baseline"/>
</p>


<h4>Step 2: Clone  Application Code</h4>
- Update all the packages
- Clone application code repository onto the EC2 instance

<pre><code>git clone https://github.com/N4si/DevSecOps-Project.git</code></pre>

<h4>Step 3: Install Docker</h4>
- Set up Docker on the EC2 instance 
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
- Open a web browser and navigate to TMDB (The Movie Database) website.
- Click on "Login" and create an account.
- Once logged in, go to your profile and select "Settings."
- Click on "API" from the left-side panel.
- Create a new API key by clicking "Create" and accepting the terms and conditions.
- Provide the required basic details and click "Submit."
- You will receive your TMDB API key.

<p align="center">
<img src="https://imgur.com/AZJrgl8.png" height="80%" width="80%" alt="baseline"/>
</p>

<h4>Step 6: Build Docker Image</h4>
<pre><code>docker build --build-arg TMDB_V3_API_KEY=&lt;your_api_key&gt; -t netflix .</code></pre>

<h4>Step 7: Install SonarQube and Trivy</h4>

- Install SonarQube and Trivy on the EC2 instance to scan for vulnerabilities.

<pre><code># Install SonarQube
docker run -d --name sonar -p 9000:9000 sonarqube:lts-community

To access:

publicIP:9000 (by default username & password is admin)
<p align="center">
<img src="https://imgur.com/p1ZIl15.png" height="80%" width="80%" alt="baseline"/>
</p>

- Integrate SonarQube with your CI/CD pipeline.
- Configure SonarQube to analyze code for quality and security issues.


# Install Trivy
sudo apt-get install wget apt-transport-https gnupg lsb-release
wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | sudo apt-key add -
echo deb https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main | sudo tee -a /etc/apt/sources.list.d/trivy.list
sudo apt-get update
sudo apt-get install trivy</code></pre>

<p align="center">
<img src="https://imgur.com/j72Oe0n.png" height="80%" width="80%" alt="baseline"/>
</p>

Jenkins Configuration
<h4>Install Jenkins</h4>

-Install Jenkins on the EC2 instance to automate deployment
<pre><code>sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] https://pkg.jenkins.io/debian-stable binary/ | sudo tee /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
</code></pre>

- Access Jenkins in a web browser using the public IP of your EC2 instance.
  publicIp:8080

  



publicIp:8080
<h4>Step 9: Install Necessary Plugins in Jenkins</h4>
Eclipse Temurin Installer (Install without restart)
SonarQube Scanner (Install without restart)
NodeJs Plugin (Install without restart)
Email Extension Plugin
Configure Java and Node.js in Global Tool Configuration.

<h4>Step 10: Configure SonarQube Server in Manage Jenkins</h4>
Create a token and add it to Jenkins.
CI/CD Pipeline Configuration
<h4>Step 11: Configure CI/CD Pipeline in Jenkins</h4>
<pre><code>pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        DOCKER_CREDENTIALS_ID = 'docker'
        SONAR_CREDENTIALS_ID = 'Sonar-token'
        TMDB_API_KEY = 'e3226ad6b25bbeecc43179071aee4afa'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from Git') {
            steps {
                git branch: 'main', url: 'https://github.com/N4si/DevSecOps-Project.git'
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Netflix \
                        -Dsonar.projectKey=Netflix'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: SONAR_CREDENTIALS_ID
                }
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . > trivyfs.txt'
            }
        }
        stage('Docker Build & Push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: DOCKER_CREDENTIALS_ID, toolName: 'docker') {
                        sh 'docker build --build-arg TMDB_V3_API_KEY=${TMDB_API_KEY} -t netflix .'
                        sh 'docker tag netflix morlo66/netflix:latest'
                        sh 'docker push morlo66/netflix:latest'
                    }
                }
            }
        }
        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image nasi101/netflix:latest > trivyimage.txt'
            }
        }
        stage('Deploy to Container') {
            steps {
                sh 'docker run -d --name netflix -p 8081:80 nasi101/netflix:latest'
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withKubeConfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'k8s', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                        sh 'kubectl apply -f Kubernetes/deployment.yml'
                        sh 'kubectl apply -f Kubernetes/service.yml'
                    }
                }
            }
        }
    }
}
</code></pre>
Monitoring and Notification Setup
<h4>Step 12: Install Prometheus</h4>
<pre><code>sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
cd prometheus-2.47.1.linux-amd64/
sudo mkdir -p /data /etc/prometheus
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles/ console_libraries/ /etc/prometheus/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

sudo nano /etc/systemd/system/prometheus.service
</code></pre>
Add the following content to the prometheus.service file:

<pre><code>[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
</code></pre>
Enable and start Prometheus:

<pre><
