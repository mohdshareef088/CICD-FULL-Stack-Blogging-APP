##📘 **FullStack Blogging App – CI/CD**


This full‑stack, Java-based blogging application uses Jenkins, Docker, Terraform, and Kubernetes for DevOps automation.
It allows users to create, edit, delete, and publish blog posts with authentication and role‑based access. 
The project is containerized with Docker, orchestrated by Kubernetes, and deployed via Jenkins CI/CD pipelines.

---
## **Architecture Diagram**

<img width="2556" height="1722" alt="image" src="https://github.com/user-attachments/assets/770b3330-2a9e-4d6f-94aa-45dbf84a6d1f" />

---
## 🏗️**Architecture Overview**
```
yaml
│prometheus.yml
│grafana_dashboard.json
kubernetes
│deployment.yml
│service.yml
│role.yaml
│rolebinding.yaml
│serviceaccount.yaml
terraform
│main.tf
│variables.tf
│outputs.tf
src
│app.js
│Dockerfile
README.md
```
---
## 🛠️**Prerequisites and tools used**
- **ec2 - Ubuntu 24.04 LTS 
- **Java 21 (OpenJDK)
- **Jenkins - CI/CD Continuous Integration and Continuous Deployment)
- **SonarQube - Docker container (Static code analysis for quality and security checks)
- **Nexus-Artifactory - Docker container (Artifact repository manager)
- **Docker: Containerization of the application.
- **Kubernetes (EKS): Orchestration of containerized applications.
- **Monitoring & Observability: Grafana, Prometheus (application performance), Blackbox for availability and uptime.
- **Custom ports: 2000-11000 required for Jenkins jobs & monitoring tools
---

## 🌐**Executions Steps**

1. **GitHub Integration**:
    - Developer pushes the code to a GitHub repo that has the source code for the application and the Terraform code that will build the EKS cluster.
    - Code is pushed to the Jenkins pipeline to perform compilation of the source code and check syntax-based errors in the pipeline,and goes to test cases to test the code. 
2. **Jenkins Manage**:
    - Setting up required servers like Jenkins, SonarQube, Nexus, and monitoring tools
    - Jenkins management to Configure and add tools, tokens, URLs, and credentials (Git, Sonar, Maven, Docker) to (systems/tools) to trigger continuous integration
3. **Trivy & Sonarqube Code Scan**:
   - Code is pushed to Trivy for vulnerabilities.
   - Code analysis & code quality check is done using SonarQube.
4. **Maven Build & Test**:
   - Maven is used to build the code and compile it  into an executable .jar file
6. **Docker & Nexus**:
   - The Nexus repo is used to store artifacts.
   - Jenkins builds a Docker image for the application.
   - The image is tagged and scanned again by Trivy to check and pushed to Docker Hub.
8. **Kubernetes Deployment**:
   - The Docker image is pushed to the Kubernetes cluster (add to secret text) to get access to the Docker Hub image, and the app is deployed to an EKS cluster.
   - Terraform is used to build EKS; Kubernetes handles scaling and service management.
   - Kubernetes provides the ALB URL to get access to the custom URL GoDaddy provides to map the domain 
9. **Monitoring**:
   - To monitor the web application metrics, the Blackbox Exporter is used and forwarded to Prometheus.
   - Grafana UI visualizes application uptime, health, and other critical metrics.

---
## 🚀**Application Build & Deploy to Dockerhub**

    pipeline {
    agent any
	
    tools {
        jdk 'jdk21'
        maven 'maven3'
    }

    environment {
        SCANNER_HOME = tool 'sonar-server'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/mohdshareef088/FullStack-Blogging-App.git'
            }
        }
   
        stage('Compile') {
            steps {
                sh "mvn compile"
            }
        }

        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh "trivy fs --format table -o fs.html ."
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar-server') {
                    sh '''
                        $SCANNER_HOME/bin/sonar-scanner \
                        -Dsonar.projectName=Blogging-app \
                        -Dsonar.projectKey=Blogging-app \
                        -Dsonar.java.binaries=target
                    '''
                }
            }
        }
        stage('Publish Artifacts') {
            steps {
                withMaven(globalMavenSettingsConfig: 'maven-settings',jdk: 'jdk21', maven: 'maven3',traceability: true) {
                    sh "mvn deploy"
                }
            }
        }
       stage('docker build') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                sh "docker build -t mohdshareef088/blogging-app:latest ."
            }
         }
       }
     }
     stage('Trivy image Scan') {
    steps {
        sh "trivy image --format table -o image.html mohdshareef088/blogging-app:latest"
      }
    }
    stage('docker push') {
    steps {
        script {
            withDockerRegistry(credentialsId: 'docker-cred', url: 'https://index.docker.io/v1/') {
                sh "docker push mohdshareef088/blogging-app:latest"
            }
           }
         }
       }
      }
     }

##🧪 **Screenshots**
<img width="1705" height="816" alt="image" src="https://github.com/user-attachments/assets/67a0a0f3-86dc-47b2-848f-42859812c38b" />
<img width="1388" height="450" alt="image" src="https://github.com/user-attachments/assets/be6245ba-4635-49f2-8512-01fb05c6e048" />
<img width="1339" height="461" alt="image" src="https://github.com/user-attachments/assets/94408e39-8f27-4772-bc72-5ad634e646a4" />
<img width="1873" height="801" alt="image" src="https://github.com/user-attachments/assets/aa24ccef-a6c5-4811-940f-acdad820d598" />
<img width="1859" height="636" alt="image" src="https://github.com/user-attachments/assets/24726cb4-cfce-450d-a534-b31d6e141966" />


---
## 🚀 **kubernetes deploy**
## Install
- **eksctl -- create eks cluster and update the config file 
- **kubectl -- create namespace webapps 
- **Svc -- create to assign a service nodeport
- **Role -- creation to assign to the service account in Jenkins
- **Bind -- the role to the service account to perform the deployment to update it or delete it
- **webapps secret -- for pulling the Docker image from my private registry and to be accessed  and utilized by Kubernetes to make pods of this image
- **pods -- to get the load balancer url to access the application


<img width="1881" height="802" alt="image" src="https://github.com/user-attachments/assets/98626aa9-e73d-4a06-a088-942a13340970" />

---

##🧪 **Application Screenshots**
---
- Homepage  Dashboard 
<img width="1908" height="970" alt="image" src="https://github.com/user-attachments/assets/4ee9c676-97d4-4996-9b7c-5b04a17bed29" />


- registration page
<img width="464" height="561" alt="image" src="https://github.com/user-attachments/assets/5de57ad0-c471-49f7-86eb-f541575de4d4" />


- Create blog page 
<img width="1888" height="787" alt="image" src="https://github.com/user-attachments/assets/4a6943d4-aa89-4dbb-901a-089d2227e2a9" />

---

##🏁 **Observibility**
## Install
- **install -- prometheus,blackbox_exporter, Grafana accesss throught separate ports 
- **configure  -- add connection url to garafana datasources & import balckbox id to grafana dashboard
---


##🧪 **Screenshots**
---
<img width="1858" height="589" alt="image" src="https://github.com/user-attachments/assets/85d20049-93b6-4f2b-9ab7-4a536910b0a9" />


<img width="1888" height="938" alt="image" src="https://github.com/user-attachments/assets/328c9a73-f0f5-40ed-b136-068b0de8c096" />





