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

<p align="center">
<img src="https://imgur.com/m3QP2sh.png" height="80%" width="80%" alt="baseline"/>
</p>


<h4>Install Necessary Plugins in Jenkins</h4>
Eclipse Temurin Installer (Install without restart)
SonarQube Scanner (Install without restart)
NodeJs Plugin (Install without restart)

Configure Java and Node.js in Global Tool Configuration.
<p align="center">
<img src="https://imgur.com/obKULqS.png" height="80%" width="80%" alt="baseline"/>
</p>

<p align="center">
<img src="https://imgur.com/EqbFYuI.png" height="80%" width="80%" alt="baseline"/>
</p>

<h4>Configure SonarQube Server in Manage Jenkins</h4>
Create a token and add it to Jenkins. Go to Jenkins Dashboard → Manage Jenkins → Credentials

<p align="center">
<img src="https://imgur.com/ZfEVzIi.png" height="80%" width="80%" alt="baseline"/>
</p>

We will install a sonar scanner in the tools.

<p align="center">
<img src="https://imgur.com/zLRXN27.png" height="80%" width="80%" alt="baseline"/>
</p>


CI/CD Pipeline Configuration
<h4>Step 11: Configure CI/CD Pipeline in Jenkins</h4>

Created a CI/CD pipeline in Jenkins to automate the application deployment
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
        TMDB_API_KEY = 'API_KEY'
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

<h4>Install OWASP Dependency Check Plugins </h4>

- Go to "Dashboard" in your Jenkins web interface.
- Navigate to "Manage Jenkins" → "Manage Plugins."
- Click on the "Available" tab and search for "OWASP Dependency-Check."
- Check the checkbox for "OWASP Dependency-Check" and click on the "Install without restart" button.

  
Configure Dependency-Check Tool:

- After installing the Dependency-Check plugin, you need to configure the tool.
- Go to "Dashboard" → "Manage Jenkins" → "Global Tool Configuration."
- Find the section for "OWASP Dependency-Check."
- Add the tool's name, e.g., "DP-Check."
- Save your settings.

<p align="center">
<img src="https://imgur.com/BjkuWub.png" height="80%" width="80%" alt="baseline"/>
</p>

Click on Apply and Save here.

Now go configure → Pipeline and add this stage to your pipeline and build.

stage('OWASP FS SCAN') {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

Install Docker Tools and Docker Plugins:

Go to "Dashboard" in your Jenkins web interface.
Navigate to "Manage Jenkins" → "Manage Plugins."
Click on the "Available" tab and search for "Docker."
Check the following Docker-related plugins:
Docker
Docker Commons
Docker Pipeline
Docker API
docker-build-step
Click on the "Install without restart" button to install these plugins.


Add DockerHub Credentials:

To securely handle DockerHub credentials in your Jenkins pipeline, follow these steps:
Go to "Dashboard" → "Manage Jenkins" → "Manage Credentials."
Click on "System" and then "Global credentials (unrestricted)."
Click on "Add Credentials" on the left side.
Choose "Secret text" as the kind of credentials.
Enter your DockerHub credentials (Username and Password) and give the credentials an ID (e.g., "docker").
Click "OK" to save your DockerHub credentials.



Monitoring Setup
# Prometheus and Grafana Setup Guide
<h4>Install Prometheus and Grafana</h4>
Set up Prometheus and Grafana to monitor your application.

<h4>Installing Prometheus</h4>

Created new t.2 medium EC2 instance for prometheus and grafana to run
<p align="center">
<img src="https://imgur.com/LkgJcr6.png" height="80%" width="80%" alt="baseline"/>
</p>
<pre><code>sudo useradd --system --no-create-home --shell /bin/false prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.47.1/prometheus-2.47.1.linux-amd64.tar.gz
</code></pre>

Extract Prometheus files, move them, and create directories:

<pre><code>tar -xvf prometheus-2.47.1.linux-amd64.tar.gz
sudo mv prometheus-2.47.1.linux-amd64/prometheus /usr/local/bin/
sudo mv prometheus-2.47.1.linux-amd64/promtool /usr/local/bin/
sudo mv prometheus-2.47.1.linux-amd64/consoles /etc/prometheus/
sudo mv prometheus-2.47.1.linux-amd64/console_libraries /etc/prometheus/
sudo mv prometheus-2.47.1.linux-amd64/prometheus.yml /etc/prometheus/
sudo mkdir /var/lib/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
</code></pre>

<h4>Prometheus Service Configuration</h4>

Create the `prometheus.service` file to define the Prometheus service:

<pre><code>[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/ \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
</code></pre>

<h4>Enabling and Starting Prometheus</h4>

Enable and start the Prometheus service:

<pre><code>sudo systemctl enable prometheus
sudo systemctl start prometheus
</code></pre>

<h4>Verifying Prometheus Status</h4>

Check the status of the Prometheus service:

<pre><code>sudo systemctl status prometheus
</code></pre>

<h4>Accessing Prometheus</h4>

Open your web browser and navigate to: http://<your-server-ip>:9090

Prometheus now running on port 9090
<p align="center">
<img src="https://imgur.com/JsmhZiw.png" height="80%" width="80%" alt="baseline"/>
</p>


<h4>Installing Node Exporter</h4>


Node Exporter is used to collect and expose system-level metrics such as CPU, memory, disk, and network usage for Prometheus to scrape and monitor.

<h4>Creating Node Exporter User and Downloading</h4>

Create a system user for Node Exporter and download the binaries:

<pre><code>sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz
</code></pre>

<h4>Extracting and Moving Node Exporter Files</h4>

Extract the tarball, move the binary, and clean up:

<pre><code>tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*
</code></pre>

<h4>Creating Node Exporter Service</h4>

Create a systemd unit file for Node Exporter:

<pre><code>sudo nano /etc/systemd/system/node_exporter.service
</code></pre>

Add the following content:

<pre><code>[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target
</code></pre>

<h4>Enabling and Starting Node Exporter</h4>

Enable and start the Node Exporter service:

<pre><code>sudo systemctl enable node_exporter
sudo systemctl start node_exporter
</code></pre>

<h4>Verifying Node Exporter Status</h4>

Check the status of Node Exporter:

<pre><code>sudo systemctl status node_exporter
</code></pre>

<h4>Prometheus Configuration</h4>

<h4>Configuring Prometheus to Scrape Metrics</h4>

Modify the `prometheus.yml` file to include Node Exporter and Jenkins scraping:

<pre><code>global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']
</code></pre>

<h4>Checking Configuration Validity</h4>

Check the validity of your Prometheus configuration:

<pre><code>promtool check config /etc/prometheus/prometheus.yml
</code></pre>

<h4>Reloading Prometheus Configuration</h4>

Reload Prometheus configuration without restarting:

<pre><code>curl -X POST http://localhost:9090/-/reload
</code></pre>

Node exporter metrics visible on port 9100
<p align="center">
<img src="https://imgur.com/aU2DgC5.png" height="80%" width="80%" alt="baseline"/>
</p>


<h4>Accessing Prometheus Targets</h4>

Open your web browser and navigate to: http://<your-prometheus-ip>:9090/targets

Node exporter and Jenkins target on Prometheus


<p align="center">
<img src="https://imgur.com/BFzJ9yy.png" height="80%" width="80%" alt="baseline"/>
</p>

<h4>Installing Grafana</h4>

<h4>Step 1: Install Dependencies</h4>

First, ensure that all necessary dependencies are installed:

<pre><code>sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common
</code></pre>

<h4>Step 2: Add the GPG Key</h4>

Add the GPG key for Grafana:

<pre><code>wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
</code></pre>

<h4>Step 3: Add Grafana Repository</h4>

Add the repository for Grafana stable releases:

<pre><code>echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
</code></pre>

<h4>Step 4: Update and Install Grafana</h4>

Update the package list and install Grafana:

<pre><code>sudo apt-get update
sudo apt-get -y install grafana
</code></pre>

<h4>Step 5: Enable and Start Grafana Service</h4>

To automatically start Grafana after a reboot, enable the service:

<pre><code>sudo systemctl enable grafana-server
</code></pre>

Then, start Grafana:

<pre><code>sudo systemctl start grafana-server
</code></pre>

<h4>Step 6: Check Grafana Status</h4>

Verify the status of the Grafana service to ensure it's running correctly:

<pre><code>sudo systemctl status grafana-server
</code></pre>

<h4>Step 7: Access Grafana Web Interface</h4>

Open a web browser and navigate to Grafana using your server's IP address. The default port for Grafana is 3000. For example: http://<your-server-ip>:3000

<p align="center">
<img src="https://imgur.com/JFfeiXm.png" height="80%" width="80%" alt="baseline"/>
</p>



You'll be prompted to log in to Grafana. The default username is "admin," and the default password is also "admin."

<h4>Step 8: Change the Default Password</h4>

When you log in for the first time, Grafana will prompt you to change the default password for security reasons. Follow the prompts to set a new password.

<h4>Step 9: Add Prometheus Data Source</h4>

To visualize metrics, you need to add a data source. Follow these steps:

1. Click on the gear icon (⚙️) in the left sidebar to open the "Configuration" menu.
2. Select "Data Sources."
3. Click on the "Add data source" button.
4. Choose "Prometheus" as the data source type.
5. In the "HTTP" section, set the "URL" to `http://localhost:9090` (assuming Prometheus is running on the same server).
6. Click the "Save & Test" button to ensure the data source is working.

<h4>Step 10: Import a Dashboard</h4>

To make it easier to view metrics, you can import a pre-configured dashboard. Follow these steps:

1. Click on the "+" (plus) icon in the left sidebar to open the "Create" menu.
2. Select "Dashboard."
3. Click on the "Import" dashboard option.
4. Enter the dashboard code you want to import (e.g., code 1860).
5. Click the "Load" button.
6. Select the data source you added (Prometheus) from the dropdown.
7. Click on the "Import" button.

You should now have a Grafana dashboard set up to visualize metrics from Prometheus.
<p align="center">
<img src="hhttps://imgur.com/3EPEyC3.png" height="80%" width="80%" alt="baseline"/>
</p>

Configure Prometheus Plugin Integration:
Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.
Installed plugin



<h







