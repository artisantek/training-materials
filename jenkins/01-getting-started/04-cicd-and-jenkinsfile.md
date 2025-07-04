# CI/CD and Jenkinsfile Guide

Understanding Continuous Integration, Continuous Delivery, Continuous Deployment, and how Jenkinsfile enables Pipeline as Code.

<div align="center">
  <img src="../images/cicd2.png" alt="CI/CD Pipeline Overview" width="800" />
</div>

## What is CI/CD?

CI/CD stands for **Continuous Integration**, **Continuous Delivery**, and **Continuous Deployment** - a set of practices that enable development teams to deliver code changes more frequently and reliably.

### Continuous Integration (CI)

**What**: Automatically integrate code changes from multiple developers into a shared repository  
**Why**: Detect integration issues early, maintain code quality, reduce merge conflicts

**Key Practices**:
- Developers commit code frequently (multiple times per day)
- Automated builds trigger on every commit
- Automated tests run to validate changes
- Quick feedback on build/test failures

**Benefits**:
- **Early bug detection** - Issues found within minutes of commit
- **Reduced integration problems** - Small, frequent changes easier to debug
- **Improved code quality** - Automated testing and quality checks
- **Faster development** - Developers get immediate feedback

### Continuous Delivery (CD)

**What**: Automatically prepare code changes for release to production  
**Why**: Ensure code is always in a deployable state, reduce deployment risk

**Key Practices**:
- All code changes pass through automated pipeline
- Deployments to staging/pre-production environments
- Manual approval gates for production deployment
- Automated rollback capabilities

**Benefits**:
- **Reduced deployment risk** - Thoroughly tested changes
- **Faster time to market** - Ready to deploy at any time
- **Improved reliability** - Consistent deployment process
- **Better collaboration** - Shared responsibility for releases

### Continuous Deployment

**What**: Automatically deploy every code change that passes tests to production  
**Why**: Eliminate manual deployment bottlenecks, deliver value faster

**Key Practices**:
- Fully automated deployment pipeline
- Comprehensive automated testing
- Monitoring and alerting in production
- Feature flags for controlled rollouts

**Benefits**:
- **Fastest delivery** - Changes reach users immediately
- **Reduced human error** - No manual deployment steps
- **Continuous feedback** - Real user feedback on changes
- **Higher productivity** - Developers focus on coding, not deployments

## CI/CD Pipeline Stages

### 1. Source Control
- Code committed to version control (Git)
- Triggers pipeline execution
- Branch-based workflows

### 2. Build
- Compile source code
- Package applications
- Generate artifacts

### 3. Test
- Unit tests
- Integration tests
- Security scans
- Code quality checks

### 4. Deploy to Staging
- Deploy to test environment
- Run additional tests
- Performance testing

### 5. Deploy to Production
- Manual approval (Continuous Delivery)
- Automatic deployment (Continuous Deployment)
- Monitoring and rollback if needed

## What is Jenkinsfile?

**Jenkinsfile** is a text file that defines a Jenkins Pipeline using code. It enables **Pipeline as Code** - treating your CI/CD pipeline definition as code that can be versioned, reviewed, and maintained alongside your application code.

### Types of Jenkinsfile

#### Declarative Pipeline
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'make build'
            }
        }
        stage('Test') {
            steps {
                sh 'make test'
            }
        }
    }
}
```

#### Scripted Pipeline
```groovy
node {
    stage('Build') {
        sh 'make build'
    }
    stage('Test') {
        sh 'make test'
    }
}
```

## Advantages of Jenkinsfile

### 1. Version Control
**What**: Pipeline definition stored in source control  
**Benefits**: 
- Track changes to pipeline over time
- Review pipeline changes like code changes
- Rollback pipeline changes if needed
- Branch-specific pipeline configurations

### 2. Pipeline as Code
**What**: Define pipelines using code instead of UI  
**Benefits**:
- Reproducible pipelines across environments
- No manual pipeline configuration
- Easy to backup and restore
- Consistent pipeline behavior

### 3. Collaboration
**What**: Multiple developers can work on pipeline  
**Benefits**:
- Code reviews for pipeline changes
- Shared ownership of CI/CD process
- Knowledge sharing across team
- Reduced single points of failure

### 4. Reusability
**What**: Share pipeline components across projects  
**Benefits**:
- Shared libraries for common tasks
- Standardized deployment processes
- Reduced duplication of pipeline code
- Consistent practices across teams

## How to Use Jenkinsfile

### 1. Create Jenkinsfile
Create a file named `Jenkinsfile` in your repository root:

```groovy
pipeline {
    agent any
    
    tools {
        maven 'Maven-3.8'
        jdk 'JDK-11'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
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
        }
        
        stage('Package') {
            steps {
                sh 'mvn package'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'echo "Deploying application..."'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
```

### 2. Configure Jenkins Job
1. Create new **Multibranch Pipeline** job
2. Configure source code repository
3. Jenkins automatically discovers Jenkinsfile
4. Pipeline runs based on Jenkinsfile definition

### 3. Key Jenkinsfile Sections

#### Agent
Specifies where pipeline runs:
```groovy
agent any                    // Run on any available agent
agent { label 'linux' }      // Run on agents with 'linux' label
agent none                   // No global agent, specify per stage
```

#### Tools
Define tools to use:
```groovy
tools {
    maven 'Maven-3.8'
    jdk 'JDK-11'
    nodejs 'Node-16'
}
```

#### Environment
Set environment variables:
```groovy
environment {
    APP_NAME = 'myapp'
    VERSION = '1.0.0'
}
```

#### Stages
Define pipeline steps:
```groovy
stages {
    stage('Build') {
        steps {
            sh 'make build'
        }
    }
}
```

#### Post Actions
Actions after pipeline completion:
```groovy
post {
    always {
        cleanWs()
    }
    success {
        emailext to: 'team@company.com',
                 subject: 'Build Success'
    }
    failure {
        emailext to: 'team@company.com',
                 subject: 'Build Failed'
    }
}
```

## Summary

**CI/CD** enables teams to deliver software faster and more reliably through automation:
- **Continuous Integration** - Frequent code integration with automated testing
- **Continuous Delivery** - Always ready to deploy with manual approval
- **Continuous Deployment** - Fully automated deployment to production

**Jenkinsfile** brings Pipeline as Code benefits:
- **Version controlled** pipelines alongside application code
- **Collaborative** development of CI/CD processes
- **Reusable** and **scalable** pipeline definitions
- **Flexible** programming capabilities for complex workflows

Start with simple Jenkinsfile, iterate based on team needs, and gradually build sophisticated CI/CD pipelines that accelerate your development process. 