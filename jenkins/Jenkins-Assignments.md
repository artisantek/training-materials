# Jenkins Assignments

Hands-on assignments to practice Jenkins concepts and build real-world CI/CD skills.

## ✅ 1. Freestyle Job: Generate WAR and Deploy to Tomcat

**Objective:**  
Introduce students to Jenkins Freestyle jobs and basic CI/CD concepts.

**Tasks:**  
- Create a Freestyle job that:
  - Checks out code from Git
  - Runs `mvn package` to generate a WAR file
  - Copies the WAR file to Tomcat server's `webapps/` folder using `ssh/scp`

**Learning Outcomes:**  
- Jenkins Freestyle job setup  
- WAR file creation  
- Manual deployment to Tomcat

**Skills Practiced:**
- Git integration
- Maven build process
- SSH/SCP deployment
- Basic CI/CD workflow

---

## ✅ 2. Run a Pipeline Job (Basic CI Pipeline)

**Objective:**  
Learn to use Jenkins Pipelines instead of Freestyle jobs.

**Tasks:**  
- Create a `Jenkinsfile` with these stages:
  - Checkout
  - Build (`mvn clean package`)
  - Run Tests (`unit tests`)
  - Deploy step

**Learning Outcomes:**  
- Declarative pipeline syntax  
- Stage-wise CI logic  
- Understanding pipeline flow vs Freestyle

**Skills Practiced:**
- Pipeline as Code
- Jenkinsfile creation
- Stage-based workflow
- Test automation integration

---

## ✅ 3. Integrate SonarQube Analysis in Jenkins Pipeline

**Objective:**  
Add static code analysis to ensure code quality.

**Tasks:**  
- Set up SonarQube and connect it to Jenkins  
- Add a Sonar scan stage in the pipeline  
- Push results to SonarQube server and review issues

**Learning Outcomes:**  
- Jenkins integration with SonarQube  
- Use of `withSonarQubeEnv` and scanner commands  
- Improving code quality early in the pipeline

**Skills Practiced:**
- Static code analysis
- SonarQube configuration
- Code quality metrics

---

## ✅ 4. CI/CD with JFrog Artifactory and Multi-Agent Setup

**Objective:**  
Use Jenkins with JFrog to separate build and deployment processes.

**Tasks:**  
- On **Agent A**:
  - Checkout code, run tests, build WAR, push to Artifactory
- On **Agent B** (Tomcat server):
  - Download WAR from Artifactory  
  - Deploy to `webapps/`

**Learning Outcomes:**  
- JFrog Artifactory integration  
- Multi-agent pipelines in Jenkins  
- Real-world CI → artifact → CD flow

**Skills Practiced:**
- Artifact management
- Distributed builds
- Agent configuration
- Deployment automation

---

## ✅ 5. Manual Approval Step Before Deployment

**Objective:**  
Introduce gated deployment using manual review.

**Tasks:**  
- After Sonar and build, pause pipeline using `input` step  
- Deploy `jar` only after manual confirmation

**Learning Outcomes:**  
- Safe deployment practices  
- Human-in-the-loop CI/CD  
- Use of `input`, `timeout`, and `post` in pipelines

**Skills Practiced:**
- Approval workflows
- Risk management
- Pipeline control flow
- Production deployment safety

---

## ✅ 6. Multibranch Pipeline Setup

**Objective:**  
Enable Jenkins to automatically detect and run pipelines for different branches.

**Tasks:**  
- Create a Git repo with multiple branches (e.g., `main`, `dev`)  
- Configure a **Multibranch Pipeline Job** in Jenkins  
- Use one shared `Jenkinsfile`, but control behavior with:
  - `when { branch 'main' }`  
  - Or environment-based conditionals
- Main branch does full CI/CD, dev branch do only test/scan

**Learning Outcomes:**  
- Dynamic pipeline behavior based on branch  
- Branch-wise CI/CD automation  
- Scalable Jenkins configuration for real projects

**Skills Practiced:**
- Branch-based workflows
- Conditional pipeline logic
- Git flow integration
- Environment-specific deployments

---

## Assignment Progression

### Beginner Level
- **Assignment 1**: Basic Freestyle job concepts
- **Assignment 2**: Introduction to Pipeline as Code

### Intermediate Level  
- **Assignment 3**: Quality integration with SonarQube
- **Assignment 5**: Approval workflows and safety

### Advanced Level
- **Assignment 4**: Multi-agent and artifact management
- **Assignment 6**: Complex branching strategies


## Additional Challenges

For students who complete the core assignments:

### Bonus Assignment A: Security Integration
- Add Snyk or OWASP dependency check to pipeline
- Generate security reports

### Bonus Assignment B: Notification System
- Set up email notifications for build status
- Integrate with Slack or Teams