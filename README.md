# Jenkins CI Setup for Java Project with GitHub Webhook Integration (On Localhost)

This guide walks you through setting up Jenkins on Ubuntu (via WSL or native), configuring it to work with a GitHub repository using PAT credentials, exposing it publicly with ngrok, and triggering CI builds using webhooks. ✅

---

## 💻 0. Setup Linux Environment with WSL (Windows Users)

### Install WSL and Ubuntu 22.04

```bash
wsl --install -d Ubuntu-22.04
```

👉 Restart your PC if prompted.

### Open Ubuntu (from Start Menu or Terminal)

Update all packages:

```bash
sudo apt update && sudo apt upgrade -y
```

You're now ready to follow the rest of the Jenkins setup below.

---

## 1. Install Jenkins as a System Service

### Prerequisites

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install fontconfig openjdk-17-jdk wget gnupg curl unzip -y
```

### Add Jenkins Repo and Key

```bash
curl -fsSL https://pkg.jenkins.io/debian/jenkins.io-2023.key | sudo tee /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
```

### Install Jenkins

```bash
sudo apt update
sudo apt install jenkins -y
```

### Start & Enable Jenkins

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

### Access Jenkins

Visit: `http://localhost:8080`

Initial admin password:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

---

## 2. Install Required Plugins (via UI)

Navigate to **Manage Jenkins → Plugins** and install:

* GitHub Integration Plugin
* Git Plugin
* Pipeline
* Pipeline: GitHub Groovy Libraries
* GitHub Branch Source
* Timestamper
* Credentials Binding Plugin

---

## 3. Create a Simple Java Project with Jenkinsfile

### GitHub Repo Structure

```
java-project-ci/
├── Jenkinsfile
└── src/
    └── Main.java
```

### Jenkinsfile Example

```groovy
pipeline {
    agent any

    environment {
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${env.PATH}"
    }

    stages {
        stage('Compile') {
            steps {
                sh 'javac src/Main.java'
                echo '✅ Compilation done.'
            }
        }
        stage('Run') {
            steps {
                sh 'java -cp src Main'
            }
        }
    }
    post {
        success {
            echo '✅ Build succeeded.'
        }
        failure {
            echo '❌ Build failed.'
        }
    }
}
```

### Example Main.java

```java
public class Main {
    public static void main(String[] args) {
        System.out.println("Hello from Jenkins CI!");
    }
}
```

Push this to GitHub repo (e.g., `https://github.com/<username>/java-project-ci`)

---

## 4. Add GitHub PAT in Jenkins Credentials

1. Generate a GitHub [Personal Access Token (classic)](https://github.com/settings/tokens)

   * Scopes: `repo`, `admin:repo_hook`

2. Go to Jenkins → **Manage Jenkins → Credentials → Global → Add Credentials**

   * Kind: `Username with password`
   * Username: your GitHub username
   * Password: your PAT token
   * ID: `github-pat` (or custom)

---

## 5. Create Jenkins Pipeline Job (From SCM)

1. Jenkins Dashboard → **New Item → Pipeline**

2. Name: `java-ci-job`

3. Configure:

   * **Pipeline → Definition**: Pipeline script from SCM
   * **SCM**: Git
   * **Repository URL**: `https://github.com/<username>/java-project-ci.git`
   * **Credentials**: Select the GitHub PAT you added
   * **Branch**: `main`
   * **Script Path**: `Jenkinsfile`

4. **Build Triggers**: Check ✅ `GitHub hook trigger for GITScm polling`

---

## 6. Expose Jenkins using ngrok

### Install ngrok

```bash
wget https://bin.equinox.io/c/bNyj1mQVY4c/ngrok-stable-linux-amd64.zip
unzip ngrok-stable-linux-amd64.zip
sudo mv ngrok /usr/local/bin/
```

### Authenticate ngrok

```bash
ngrok config add-authtoken <your_token>
```

### Create Config File

Create `~/.config/ngrok/ngrok.yml`:

```yaml
version: "2"
authtoken: <your_token>
tunnels:
  jenkins:
    addr: 8080
    proto: http
```

### Create Systemd Service

```bash
sudo nano /etc/systemd/system/ngrok.service
```

Paste:

```ini
[Unit]
Description=ngrok tunnel for Jenkins
After=network.target

[Service]
ExecStart=/usr/local/bin/ngrok start --config /root/.config/ngrok/ngrok.yml --all
Restart=on-failure
User=root

[Install]
WantedBy=multi-user.target
```

Then run:

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable ngrok
sudo systemctl start ngrok
```

### Get Public URL

```bash
curl http://localhost:4040/api/tunnels
```

Use the `https://xyz.ngrok.io` URL in the next step.

---

## 7. GitHub Webhook Setup

1. Go to your GitHub repo → **Settings → Webhooks → Add webhook**

2. Fill:

   * **Payload URL**:

     ```
     ```

[https://xyz.ngrok.io/github-webhook/](https://xyz.ngrok.io/github-webhook/)

```
   - **Content type**: `application/json`
   - **Events**: `Just the push event`

3. Save webhook

4. Test by making a commit → push → Jenkins job should trigger

---

## ✅ Verification
- GitHub → Settings → Webhooks → View recent deliveries (should be 200 OK)
- Jenkins → Console Output should show build running and `Hello from Jenkins CI!`

---

## 🎉 You Did It!
You now have a fully functional Jenkins CI pipeline that:
- Compiles & runs Java code
- Authenticates to GitHub using PAT
- Is triggered by GitHub webhooks
- Runs locally but is exposed publicly via ngrok

Let me know if you'd like to extend this with testing, deployment, or Docker builds!

```
