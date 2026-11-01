# singlepageapp
my info
personal info

added one more line
# LAB LAPTOP – EXECUTION PROCEDURE

## STEP 1: Install and start software

Install:
1. Java JDK 21
2. Git
3. Docker Desktop
4. Jenkins
5. Maven (if needed; project includes `mvnw.cmd`)

Open Docker Desktop → Settings → Kubernetes → Enable Kubernetes.

Wait until Kubernetes is running.

Open CMD and execute:

```bat
java -version
git --version
docker --version
kubectl get nodes
```

The Kubernetes node must show `Ready`.

## STEP 2: Download the project

Open CMD:

```bat
git clone https://github.com/cpmanasareddy/springboot-registration.git
```

```bat
cd springboot-registration
```

Verify:

```bat
dir
```

Check that these files exist:

```text
pom.xml
mvnw.cmd
Dockerfile
k8s\deployment.yaml
k8s\service.yaml
k8s\pvc.yaml
```

## STEP 3: Find Docker and kubectl paths

Run:

```bat
where docker
where kubectl
```

Copy the executable paths.

Find your Windows username:

```bat
echo %USERNAME%
```

Find Kubernetes config:

```text
C:\Users\YOUR_USERNAME\.kube\config
```

Replace `YOUR_USERNAME` in the Jenkins scripts below.

## STEP 4: Open Jenkins

Open:

```text
http://localhost:8080
```

If Jenkins is configured on port 8081, use:

```text
http://localhost:8081
```

Install these plugins through Manage Jenkins → Plugins:

- Git
- Pipeline
- Stage View

## STEP 5: Create Job1

1. Click New Item.
2. Name: `Job1`
3. Select Freestyle project.
4. Click OK.
5. Under Source Code Management → Git, enter:

```text
https://github.com/cpmanasareddy/springboot-registration.git
```

Branch:

```text
*/main
```

6. Under Build Steps → Add build step → Execute Windows batch command, paste:

```bat
echo Source code pulled successfully from GitHub
echo Job1 completed successfully
```

7. Save.

## STEP 6: Create Job2

1. New Item → `Job2` → Freestyle project.
2. Configure Git SCM using the same repository URL and `*/main`.
3. Add Execute Windows batch command.
4. Paste the following script after replacing the paths:

```bat
@echo off

set KUBECONFIG=C:\Users\YOUR_USERNAME\.kube\config
set DOCKER=C:\Users\YOUR_USERNAME\AppData\Local\Programs\DockerDesktop\resources\bin\docker.exe
set KUBECTL=C:\Users\YOUR_USERNAME\AppData\Local\Programs\DockerDesktop\resources\bin\kubectl.exe

echo STEP 1: Maven build

call mvnw.cmd clean package -DskipTests
if errorlevel 1 exit /b 1

echo STEP 2: Docker image build

"%DOCKER%" build -t exp12:latest .
if errorlevel 1 exit /b 1

echo STEP 3: Kubernetes connection

"%KUBECTL%" get nodes
if errorlevel 1 exit /b 1

echo STEP 4: Persistent storage

"%KUBECTL%" apply -f k8s\pvc.yaml
if errorlevel 1 exit /b 1

echo STEP 5: Deployment

"%KUBECTL%" apply -f k8s\deployment.yaml
if errorlevel 1 exit /b 1

echo STEP 6: Service

"%KUBECTL%" apply -f k8s\service.yaml
if errorlevel 1 exit /b 1

echo STEP 7: Rollout status

"%KUBECTL%" rollout status deployment/experiment12-deployment --timeout=120s
if errorlevel 1 exit /b 1

"%KUBECTL%" get pods
"%KUBECTL%" get services

echo KUBERNETES DEPLOYMENT SUCCESSFUL
```

5. Save.

## STEP 7: Create Job3

1. New Item → `Job3` → Freestyle project.
2. Add Execute Windows batch command.
3. Paste:

```bat
@echo off

set KUBECTL=C:\Users\YOUR_USERNAME\AppData\Local\Programs\DockerDesktop\resources\bin\kubectl.exe

echo STEP 1: Checking deployment

"%KUBECTL%" get deployment experiment12-deployment
if errorlevel 1 exit /b 1

echo STEP 2: Testing application

powershell -NoProfile -ExecutionPolicy Bypass -Command "try { $r=Invoke-WebRequest -Uri 'http://localhost:8083/registrations' -UseBasicParsing -TimeoutSec 15; if ($r.StatusCode -eq 200) { Write-Host 'APPLICATION TEST PASSED'; exit 0 } else { exit 1 } } catch { Write-Host $_; exit 1 }"

if errorlevel 1 exit /b 1

echo JOB3 TEST COMPLETED SUCCESSFULLY
```

4. Save.

## STEP 8: Start application port forwarding

Open a separate CMD window:

```bat
kubectl port-forward service/experiment12-service 8083:8080
```

Keep this window open.

Test in browser:

```text
http://localhost:8083/registrations
```

It should display:

```json
[]
```

## STEP 9: Create Job4

1. New Item → `Job4` → Freestyle project.
2. Under Build Triggers, select Build periodically.
3. Enter:

```text
H/1 * * * *
```

4. Under Build Steps → Execute Windows batch command, paste:

```bat
@echo off

set KUBECTL=C:\Users\YOUR_USERNAME\AppData\Local\Programs\DockerDesktop\resources\bin\kubectl.exe

echo STEP 1: Checking deployment

"%KUBECTL%" rollout status deployment/experiment12-deployment --timeout=30s
if errorlevel 1 exit /b 1

echo STEP 2: Checking pods

"%KUBECTL%" get pods -l app=experiment12

"%KUBECTL%" wait --for=condition=Ready pod -l app=experiment12 --timeout=30s
if errorlevel 1 exit /b 1

echo APPLICATION IS HEALTHY
```

5. Save.

## STEP 10: Create Pipeline for Stage View

1. Click New Item.
2. Name:

```text
Experiment9-Pipeline
```

3. Select Pipeline.
4. Click OK.
5. Scroll to Pipeline.
6. Definition → Pipeline script.
7. Paste:

```groovy
pipeline {
    agent any

    stages {
        stage('Job1 - GitHub Checkout') {
            steps {
                build job: 'Job1', wait: true
            }
        }

        stage('Job2 - Kubernetes Deployment') {
            steps {
                build job: 'Job2', wait: true
            }
        }

        stage('Job3 - Application Testing') {
            steps {
                build job: 'Job3', wait: true
            }
        }

        stage('Job4 - Application Monitoring') {
            steps {
                build job: 'Job4', wait: true
            }
        }
    }
}
```

8. Click Save.

## STEP 11: Run the complete project

1. Open `Experiment9-Pipeline`.
2. Click Build Now.
3. Wait for completion.
4. Open Stage View.

Expected result:

| Stage | Expected status |
|---|---|
| Job1 – GitHub Checkout | Green |
| Job2 – Kubernetes Deployment | Green |
| Job3 – Application Testing | Green |
| Job4 – Application Monitoring | Green |

## IMPORTANT BEFORE RUNNING

For this Pipeline-based execution:

- Do not configure Job2 to trigger automatically after Job1.
- Do not configure Job3 to trigger automatically after Job2.
- Do not configure Job2 as a post-build action in Job4.
- The Pipeline itself runs all four jobs in order.

Keep the port-forward CMD window open until the Pipeline finishes.

**If any job fails:** Open that job → latest build → Console Output and check the last error.
## Before Configuring Jenkins Jobs – Replace Laptop-Specific Paths

### Step 1: Find your laptop username and tool paths

Open Command Prompt (CMD) and execute:

```bat
echo %USERNAME%
where docker
where kubectl
```

Note the output. Do not copy paths from another laptop.

### Step 2: Find the Kubernetes configuration file

Usually, it is located at:

```text
C:\Users\<YOUR_USERNAME>\.kube\config
```

Confirm that this file exists on your laptop.

### Step 3: Replace paths in Jenkins jobs

Open Jenkins → Select the job → Configure → Build Steps → Execute Windows batch command.

Replace the paths as follows:

| Job | Paths to replace |
|---|---|
| Job1 | No replacement required |
| Job2 | KUBECONFIG, DOCKER and KUBECTL |
| Job3 | KUBECONFIG and KUBECTL |
| Job4 | KUBECONFIG and KUBECTL |
| Pipeline job | No replacement required |

Use the actual paths found on your laptop.

Example (if your username is `Student`):

**KUBECONFIG**
```bat
set KUBECONFIG=C:\Users\Student\.kube\config
```

**DOCKER**
```bat
set DOCKER=C:\Users\Student\AppData\Local\Programs\DockerDesktop\resources\bin\docker.exe
```

**KUBECTL**
```bat
set KUBECTL=C:\Users\Student\AppData\Local\Programs\DockerDesktop\resources\bin\kubectl.exe
```

If `where docker` or `where kubectl` returns a different location, use that actual location instead.

### Step 4: Save the changes

After replacing the paths in each job:

1. Click Save.
2. Repeat for the remaining jobs.
3. Verify Docker Desktop Kubernetes is enabled.
4. Run `kubectl get nodes` and confirm the node status is `Ready`.
5. Continue with the pipeline execution steps.

**Important:** Do not replace the paths in the Pipeline script. Only replace laptop-specific paths in the Windows batch scripts.


