# Jenkins

## Overview
Jenkins is an open-source automation server that enables developers to build, test, and deploy applications through continuous integration and continuous deployment (CI/CD) pipelines.

## Architecture

### Core Components
- **Jenkins Master/Controller**: Manages builds, distributes work to agents
- **Jenkins Agents/Nodes**: Execute builds and tests
- **Jobs/Projects**: Configured tasks that Jenkins executes
- **Pipelines**: Code-as-configuration for build processes
- **Plugins**: Extensions that add functionality

## Installation

### Ubuntu/Debian
```bash
# Add Jenkins repository
wget -q -O - https://pkg.jenkins.io/debian/jenkins.io.key | sudo apt-key add -
echo "deb https://pkg.jenkins.io/debian-stable binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list

# Install Java (required dependency)
sudo apt update
sudo apt install openjdk-11-jdk

# Install Jenkins
sudo apt install jenkins

# Start Jenkins service
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Check status
sudo systemctl status jenkins
```

### Docker Installation
```bash
# Run Jenkins in Docker
docker run -d \
  --name jenkins \
  -p 8080:8080 \
  -p 50000:50000 \
  -v jenkins_home:/var/jenkins_home \
  jenkins/jenkins:lts

# Get initial admin password
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### CentOS/RHEL
```bash
# Add Jenkins repository
sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# Install Java and Jenkins
sudo yum install java-11-openjdk jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

## Initial Setup

### First-Time Configuration
1. **Access Jenkins**: http://localhost:8080
2. **Unlock Jenkins**: Use initial admin password from `/var/lib/jenkins/secrets/initialAdminPassword`
3. **Install plugins**: Choose "Install suggested plugins" or select specific ones
4. **Create admin user**: Set up the first administrator account
5. **Configure Jenkins URL**: Set the base URL for Jenkins

### Essential Plugins
```bash
# Install via Jenkins CLI or web interface
- Git plugin
- Pipeline plugin
- Docker plugin
- Build Timeout plugin
- Timestamper plugin
- Workspace Cleanup plugin
- Email Extension plugin
- Blue Ocean (modern UI)
```

## Job Types

### Freestyle Projects
```bash
# Basic build configuration
Source Code Management: Git repository URL
Build Triggers: Poll SCM, GitHub hook trigger
Build Environment: Delete workspace, timestamps
Build Steps: Execute shell, invoke Maven
Post-build Actions: Archive artifacts, email notifications
```

### Pipeline Jobs
```groovy
// Declarative Pipeline example
pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/user/repo.git'
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean compile'
            }
        }
        
        stage('Test') {
            steps {
                sh 'mvn test'
            }
            post {
                always {
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'mvn deploy'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}
```

## Pipeline as Code

### Jenkinsfile Examples

#### Simple Java Application
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.8.1-openjdk-11'
            args '-v /root/.m2:/root/.m2'
        }
    }
    
    environment {
        MAVEN_OPTS = '-Xmx1024m'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }
        
        stage('Test') {
            parallel {
                stage('Unit Tests') {
                    steps {
                        sh 'mvn test'
                    }
                    post {
                        always {
                            publishTestResults testResultsPattern: 'target/surefire-reports/TEST-*.xml'
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'mvn verify -Pfailsafe'
                    }
                }
            }
        }
        
        stage('Quality Gate') {
            steps {
                script {
                    def qg = waitForQualityGate()
                    if (qg.status != 'OK') {
                        error "Pipeline aborted due to quality gate failure: ${qg.status}"
                    }
                }
            }
        }
        
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                sh './deploy.sh'
            }
        }
    }
}
```

#### Docker Application
```groovy
pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'registry.example.com'
        IMAGE_NAME = 'my-app'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    stages {
        stage('Build Docker Image') {
            steps {
                script {
                    def image = docker.build("${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}")
                    image.push()
                    image.push('latest')
                }
            }
        }
        
        stage('Security Scan') {
            steps {
                sh "docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:latest ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
            }
        }
        
        stage('Deploy to Staging') {
            steps {
                sh """
                    docker run -d \
                        --name ${IMAGE_NAME}-staging \
                        -p 3001:3000 \
                        ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}
                """
            }
        }
        
        stage('Smoke Tests') {
            steps {
                sh 'curl -f http://localhost:3001/health'
            }
        }
        
        stage('Deploy to Production') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Deploy to production?', ok: 'Deploy'
                sh './deploy-production.sh'
            }
        }
    }
    
    post {
        always {
            sh 'docker container rm -f ${IMAGE_NAME}-staging || true'
        }
    }
}
```

## Build Configuration

### Source Code Management
```bash
# Git configuration
Repository URL: https://github.com/user/repo.git
Credentials: Username/password or SSH key
Branches: */main, */develop
Additional behaviors: Clean before checkout, Shallow clone
```

### Build Triggers
```bash
# Webhook trigger
GitHub hook trigger for GITScm polling

# Scheduled builds
H 2 * * * (daily at 2 AM)
H/15 * * * * (every 15 minutes)

# Poll SCM
H/5 * * * * (check for changes every 5 minutes)

# Upstream projects
Build after other projects are built
```

### Build Environment
```bash
# Environment variables
BUILD_NUMBER: Jenkins build number
JOB_NAME: Name of the job
WORKSPACE: Job workspace directory
JENKINS_HOME: Jenkins home directory

# Custom environment
MY_ENV=production
DATABASE_URL=jdbc:mysql://localhost:3306/app
```

## Agent Management

### Agent Configuration
```bash
# Add new agent
Manage Jenkins > Manage Nodes > New Node

# Agent connection methods
- Launch via SSH
- Launch via Java Web Start
- Launch via Windows service
- Docker containers as agents
```

### Docker Agents
```groovy
pipeline {
    agent {
        docker {
            image 'node:16-alpine'
            args '-v /tmp:/tmp'
        }
    }
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
                sh 'npm run build'
            }
        }
    }
}
```

### Label-based Assignment
```groovy
pipeline {
    agent {
        label 'linux && docker'
    }
    // Job runs on agents with both 'linux' and 'docker' labels
}
```

## Security

### User Management
```bash
# Security realms
- Jenkins own database
- LDAP
- Active Directory
- GitHub OAuth

# Authorization strategies
- Matrix-based security
- Project-based Matrix Authorization
- Role-based Authorization
```

### Credentials Management
```bash
# Credential types
- Username/password
- SSH private key
- Secret text
- Certificate
- Docker credentials

# Using credentials in pipeline
withCredentials([usernamePassword(credentialsId: 'github-creds', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
    sh 'git push https://${USERNAME}:${PASSWORD}@github.com/user/repo.git'
}
```

### Script Security
```groovy
// Approved script signatures
method java.lang.String toLowerCase
method java.util.Collection size
staticMethod java.lang.System currentTimeMillis

// In-process script approval
@NonCPS
def processData(data) {
    return data.collect { it.toLowerCase() }
}
```

## Monitoring and Maintenance

### System Information
```bash
# Jenkins system info
Manage Jenkins > System Information

# Available plugins
Manage Jenkins > Manage Plugins

# System log
Manage Jenkins > System Log

# Script console
Manage Jenkins > Script Console
```

### Backup Strategy
```bash
# Backup Jenkins home
sudo tar -czf jenkins-backup-$(date +%Y%m%d).tar.gz /var/lib/jenkins/

# Exclude unnecessary files
--exclude='workspace/*' \
--exclude='jobs/*/builds/*/log' \
--exclude='jobs/*/builds/*/changelog.xml'

# Automated backup script
#!/bin/bash
BACKUP_DIR="/backups/jenkins"
JENKINS_HOME="/var/lib/jenkins"
DATE=$(date +%Y%m%d_%H%M%S)

sudo systemctl stop jenkins
sudo tar -czf "$BACKUP_DIR/jenkins-$DATE.tar.gz" "$JENKINS_HOME"
sudo systemctl start jenkins

# Keep only last 7 backups
find "$BACKUP_DIR" -name "jenkins-*.tar.gz" -mtime +7 -delete
```

### Performance Tuning
```bash
# JVM options in /etc/default/jenkins
JAVA_ARGS="-Xmx2048m -XX:+UseConcMarkSweepGC"

# Increase number of executors
Manage Jenkins > Configure System > # of executors

# Clean up old builds
Build Discarder: Keep builds for 30 days, max 50 builds
```

## Integrations

### Version Control
```groovy
// GitHub integration
stage('Checkout') {
    steps {
        checkout([$class: 'GitSCM',
            branches: [[name: '*/main']],
            userRemoteConfigs: [[url: 'https://github.com/user/repo.git']]
        ])
    }
}

// Bitbucket integration
// GitLab integration
```

### Testing Frameworks
```groovy
// JUnit integration
post {
    always {
        junit 'target/surefire-reports/*.xml'
    }
}

// TestNG integration
step([$class: 'Publisher', reportFilenamePattern: 'test-output/testng-results.xml'])

// Coverage reports
publishHTML([
    allowMissing: false,
    alwaysLinkToLastBuild: true,
    keepAll: true,
    reportDir: 'coverage',
    reportFiles: 'index.html',
    reportName: 'Coverage Report'
])
```

### Deployment Tools
```groovy
// Kubernetes deployment
stage('Deploy to K8s') {
    steps {
        kubernetesDeploy(
            configs: 'k8s-deployment.yaml',
            kubeconfigId: 'k8s-config'
        )
    }
}

// Ansible integration
stage('Deploy with Ansible') {
    steps {
        ansiblePlaybook(
            playbook: 'deploy.yml',
            inventory: 'inventory/production'
        )
    }
}
```

## Best Practices

### Pipeline Design
1. **Keep pipelines simple** and readable
2. **Use shared libraries** for common functionality
3. **Implement proper error handling**
4. **Use parallel stages** for independent tasks
5. **Clean workspace** after builds

### Security Best Practices
1. **Regular updates** of Jenkins and plugins
2. **Use least privilege** access control
3. **Secure credentials** management
4. **Enable CSRF protection**
5. **Use HTTPS** for Jenkins access

### Performance Optimization
1. **Distribute builds** across multiple agents
2. **Use appropriate workspace** cleanup
3. **Optimize build tools** (Maven, Gradle settings)
4. **Monitor resource usage**
5. **Archive only necessary** artifacts

## Troubleshooting

### Common Issues
```bash
# Build stuck in queue
- Check agent availability
- Verify agent labels match job requirements
- Check system resources

# Out of disk space
find /var/lib/jenkins -type f -name "*.log" -size +100M
du -sh /var/lib/jenkins/workspace/*

# Plugin conflicts
- Check Jenkins logs: /var/log/jenkins/jenkins.log
- Disable conflicting plugins
- Update to compatible versions

# Permission issues
sudo chown -R jenkins:jenkins /var/lib/jenkins/
```

### Log Analysis
```bash
# Jenkins system log
tail -f /var/log/jenkins/jenkins.log

# Build console output
Available in job build history

# Agent connection logs
Available in agent configuration page
```

## Related Tools
- [[Docker]] - Container integration
- [[Kubernetes]] - Orchestration platform
- [[Ansible]] - Configuration management
- [[SonarQube]] - Code quality analysis
- [[Nexus]] - Artifact repository

## Related Topics
- [[CI/CD pipelines]]
- [[DevOps]]
- [[Build automation]]
- [[Continuous integration]]
- [[Infrastructure as Code]]
