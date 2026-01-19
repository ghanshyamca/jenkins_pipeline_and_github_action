# Jenkins CI/CD Pipeline

Complete CI/CD pipeline for automated testing, building, and deployment of Flask application using Jenkins.


---

## 🎯 Overview

This Jenkins pipeline automates the entire CI/CD process for the Flask application:

```
Git Push → Checkout → Setup → Build → Quality Check → Test → Security Scan → Deploy → Smoke Test → Notify
```

**Pipeline File**: [`Jenkinsfile`](Jenkinsfile)

**Jenkins URL**: `http://your-jenkins-server:8080`

---

## ✨ Features

### 🔄 Continuous Integration
- ✅ Automated source code checkout
- ✅ Python virtual environment setup
- ✅ Dependency installation
- ✅ Code quality checks (flake8)
- ✅ Unit testing with pytest
- ✅ Code coverage reporting (88%+)
- ✅ Security vulnerability scanning (Safety, Bandit)

### 🚀 Continuous Deployment
- ✅ Automated deployment to staging (on `staging` branch)
- ✅ Automated deployment to production (on version tags `v*`)
- ✅ SSH-based deployment to EC2 instances
- ✅ Smoke tests and health checks
- ✅ Service restart automation

### 📦 Build & Artifacts
- ✅ Build artifact creation (.tar.gz)
- ✅ Test results preservation (JUnit XML)
- ✅ Coverage reports (HTML)
- ✅ Security scan reports (JSON)
- ✅ Artifact archival and fingerprinting

### 📧 Notifications
- ✅ HTML email notifications on success
- ✅ HTML email notifications on failure
- ✅ Build status in email body
- ✅ Direct links to build console

---

## 🏗️ Pipeline Architecture

```
┌─────────────┐
│   Checkout  │ Git checkout
└──────┬──────┘
       │
┌──────▼──────┐
│Setup Environment│ Python venv
└──────┬──────┘
       │
┌──────▼──────┐
│    Build    │ Install dependencies
└──────┬──────┘
       │
┌──────▼──────┐
│Code Quality │ flake8 linting
└──────┬──────┘
       │
┌──────▼──────┐
│    Test     │ pytest + coverage
└──────┬──────┘
       │
┌──────▼──────┐
│Security Scan│ Safety + Bandit
└──────┬──────┘
       │
┌──────▼──────┐
│Build Artifact│ Create .tar.gz
└──────┬──────┘
       │
       ├─────────────────┬─────────────────┐
       │                 │                 │
┌──────▼──────┐   ┌─────▼─────┐   ┌──────▼──────┐
│Deploy Staging│   │Deploy Prod│   │   Skip      │
│(staging branch)│ │(tag v*)   │   │(other branches)│
└──────┬──────┘   └─────┬─────┘   └─────────────┘
       │                 │
┌──────▼──────┐   ┌─────▼─────┐
│Smoke Test   │   │Health Check│
└──────┬──────┘   └─────┬─────┘
       │                 │
       └────────┬────────┘
                │
         ┌──────▼──────┐
         │Email Notify │
         └─────────────┘
```

---

## 📋 Prerequisites

### Required Software

#### On Jenkins Server:
- [x] **Jenkins 2.400+** installed and running
- [x] **Java 11 or 17** (required by Jenkins)
- [x] **Git** (for source code checkout)
- [x] **Python 3.9+** (for building and testing)
- [x] **SSH client** (for deployment)

#### On Target Servers (Staging/Production):
- [x] **Ubuntu 22.04 LTS** (or compatible Linux)
- [x] **Python 3.9+** with pip and venv
- [x] **Nginx** (reverse proxy)
- [x] **systemd** (for service management)
- [x] Flask app configured as systemd service

### Required Jenkins Plugins

Install these plugins from **Manage Jenkins → Plugin Manager**:

1. **Pipeline** (Pipeline plugin)
2. **Git** (Git plugin)
3. **Credentials Binding** (Credentials Binding Plugin)
4. **Email Extension** (Email Extension Plugin)
5. **JUnit** (JUnit Plugin)
6. **HTML Publisher** (HTML Publisher plugin)
7. **Workspace Cleanup** (Workspace Cleanup Plugin)
8. **SSH Agent** (ssh agent)

### Infrastructure

- **Staging EC2 Instance**: t2.micro (free tier eligible)
- **Production EC2 Instance**: t2.small or larger
- **SSH Access**: Both servers accessible from Jenkins

---

## 🚀 Jenkins Setup

### Step 1: Install Jenkins

**On Ubuntu 22.04:**

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install Java 17
sudo apt install -y openjdk-17-jdk

# Add Jenkins repository
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# Install Jenkins
sudo apt update
sudo apt install -y jenkins

# Start Jenkins
sudo systemctl start jenkins
sudo systemctl enable jenkins

# Get initial admin password
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**Access Jenkins**: `http://your-server-ip:8080`

### Step 2: Initial Configuration

1. **Unlock Jenkins**: Enter the initial admin password
2. **Install Plugins**: Select "Install suggested plugins"
3. **Create Admin User**: Set username and password
4. **Configure Jenkins URL**: `http://your-jenkins-server:8080`

### Step 3: Install Required Plugins

1. Go to **Manage Jenkins** → **Plugin Manager**
2. Click **Available plugins** tab
3. Search and install:
   - Pipeline
   - Git
   - Credentials Binding
   - Email Extension Plugin
   - HTML Publisher
   - JUnit Plugin
   - SSH Agent

4. Click **Install without restart**
5. Restart Jenkins after installation

<img width="1154" height="764" alt="image" src="https://github.com/user-attachments/assets/77409c03-71f6-41b9-8e04-70236dc9f3fa" />

### Step 4: Configure System Tools

**Manage Jenkins → Tools**

#### Git Configuration:
- Name: `Default`
- Path to Git executable: `git` (or `/usr/bin/git`)

#### Python:
Jenkins will use system Python. Verify:
```bash
python3 --version  # Should be 3.9+
```

### Step 5: Configure Email

**Manage Jenkins → System → Extended E-mail Notification**

**For Gmail:**
- SMTP server: `smtp.gmail.com`
- SMTP port: `587`
- Use SSL: ✓
- Credentials: Add Gmail credentials (username + App Password)
- Default user e-mail suffix: `@gmail.com`

**Test configuration**: Click "Test configuration by sending test e-mail"

---

<img width="1212" height="768" alt="image" src="https://github.com/user-attachments/assets/83e15941-f102-43a2-8a97-78cbe0fae222" />



## ⚙️ Configuration

### Environment Variables (in Jenkinsfile)

```groovy
environment {
    PYTHON_VERSION = '3.9'              // Python version to use
    VIRTUAL_ENV = 'venv'                // Virtual environment name
    FLASK_APP = 'app.py'                // Flask application file
    
    // Deployment paths
    STAGING_DEPLOY_PATH = '/var/www/flask-app'
    PRODUCTION_DEPLOY_PATH = '/var/www/flask-app'
    
    // Email configuration
    EMAIL_RECIPIENTS = 'your-email@example.com'  // Update this!
}
```

### Branch-Based Deployment

```groovy
// Staging deployment - triggered by 'staging' branch
stage('Deploy to Staging') {
    when {
        branch 'staging'
    }
    // Deployment steps...
}

// Production deployment - triggered by tags matching v*.*.* pattern
stage('Deploy to Production') {
    when {
        tag pattern: "v\\d+\\.\\d+\\.\\d+", comparator: "REGEXP"
    }
    // Deployment steps...
}
```

---

## 🔐 Credentials Setup

### Add Credentials in Jenkins

**Manage Jenkins → Credentials → System → Global credentials → Add Credentials**

Add the following **10 credentials**:

#### 1. Staging Host
- **Kind**: Secret text
- **Scope**: Global
- **Secret**: `3.110.45.123` (your staging server IP)
- **ID**: `staging-host`
- **Description**: Staging Server IP/Hostname

#### 2. Staging User
- **Kind**: Secret text
- **Secret**: `ubuntu`
- **ID**: `staging-user`
- **Description**: Staging SSH Username

#### 3. Staging SSH Key
- **Kind**: Secret text
- **Secret**: [Paste entire private key content]
```
-----BEGIN RSA PRIVATE KEY-----
MIIEpAIBAAKCAQEA...
...
-----END RSA PRIVATE KEY-----
```
- **ID**: `staging-ssh-key`
- **Description**: Staging SSH Private Key

#### 4. Production Host
- **Kind**: Secret text
- **Secret**: `13.234.56.78` (your production server IP)
- **ID**: `production-host`
- **Description**: Production Server IP/Hostname

#### 5. Production User
- **Kind**: Secret text
- **Secret**: `ubuntu`
- **ID**: `production-user`
- **Description**: Production SSH Username

#### 6. Production SSH Key
- **Kind**: Secret text
- **Secret**: [Paste entire private key content]
- **ID**: `production-ssh-key`
- **Description**: Production SSH Private Key


<img width="1210" height="770" alt="image" src="https://github.com/user-attachments/assets/f80c4ba7-9cef-4925-a179-570b50657ce2" />

## setup ngrok in ubuntu

<img width="1083" height="473" alt="image" src="https://github.com/user-attachments/assets/23f33ae1-8529-4ef3-87a0-73dd5a0c6fd0" />

## Github webhook setup

<img width="1920" height="1399" alt="image" src="https://github.com/user-attachments/assets/8516001b-a830-45d4-9a6b-88ea95782598" />

## Github webhooks successfull

<img width="1920" height="6922" alt="image" src="https://github.com/user-attachments/assets/c70a76cb-57d4-4af6-bf32-836d121a52ea" />

## 🔧 Jenkins Pipeline Stages

### Stage 1: configure pipeline

<img width="1204" height="760" alt="image" src="https://github.com/user-attachments/assets/f9ea4772-3275-441f-9501-d5830dc26b68" />

<img width="1207" height="771" alt="image" src="https://github.com/user-attachments/assets/aaee7887-d3e3-4439-8465-952cdf99a53b" />

<img width="1194" height="773" alt="image" src="https://github.com/user-attachments/assets/a961a2e1-4860-4edc-a4ce-425044ce51b8" />

<img width="1209" height="777" alt="image" src="https://github.com/user-attachments/assets/d7ae7d0f-7ac2-48e5-b2c0-5e98b10d832b" />

### Stage 2: Push code staging branch

<img width="1920" height="1999" alt="image" src="https://github.com/user-attachments/assets/2b5fc711-6e92-44da-95ab-b62529728258" />

### Stage 3. Github Webhooks trigger

<img width="1920" height="6922" alt="image" src="https://github.com/user-attachments/assets/16317eb5-3d55-47fa-b6f2-d478c0a0accb" />

<img width="1206" height="765" alt="image" src="https://github.com/user-attachments/assets/cf5131d6-6507-47be-a1bf-b2eff9f53e58" />

### Stage 4. CI/CD logs

```
Started by user admin
 > git rev-parse --resolve-git-dir /var/lib/jenkins/caches/git-640a1658cad41f253def4865ccabb7b3/.git # timeout=10
Setting origin to https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
Fetching origin...
Fetching upstream changes from origin
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git config --get remote.origin.url # timeout=10
 > git fetch --no-tags --force --progress -- origin +refs/heads/*:refs/remotes/origin/* # timeout=10
Seen branch in repository origin/feature/changes
Seen branch in repository origin/main
Seen branch in repository origin/revert-4-feature/changes
Seen branch in repository origin/staging
Seen 4 remote branches
Obtained Jenkinsfile from f725c1df22493fef0e38bc2f7f441b876bafd67a
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning with configured refspecs honoured and without tags
Cloning repository https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git init /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging # timeout=10
Fetching upstream changes from https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --no-tags --force --progress -- https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision f725c1df22493fef0e38bc2f7f441b876bafd67a (staging)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f f725c1df22493fef0e38bc2f7f441b876bafd67a # timeout=10
Commit message: "Merge pull request #6 from ghanshyamca/feature/changes"
 > git rev-list --no-walk f725c1df22493fef0e38bc2f7f441b876bafd67a # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withCredentials
Masking supported pattern matches of $PRODUCTION_HOST or $PRODUCTION_USER or $STAGING_USER or $STAGING_HOST
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Checkout)
[Pipeline] echo
Checking out source code...
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --no-tags --force --progress -- https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Checking out Revision f725c1df22493fef0e38bc2f7f441b876bafd67a (staging)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f f725c1df22493fef0e38bc2f7f441b876bafd67a # timeout=10
Commit message: "Merge pull request #6 from ghanshyamca/feature/changes"
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Setup Environment)
[Pipeline] echo
Setting up Python virtual environment...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ python3 -m venv venv
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install --upgrade pip
Requirement already satisfied: pip in ./venv/lib/python3.10/site-packages (22.0.2)
Collecting pip
  Using cached pip-25.3-py3-none-any.whl (1.8 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 22.0.2
    Uninstalling pip-22.0.2:
      Successfully uninstalled pip-22.0.2
Successfully installed pip-25.3
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
Installing dependencies...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install -r requirements.txt
Collecting Flask==3.0.0 (from -r requirements.txt (line 2))
  Using cached flask-3.0.0-py3-none-any.whl.metadata (3.6 kB)
Collecting Werkzeug==3.0.1 (from -r requirements.txt (line 3))
  Using cached werkzeug-3.0.1-py3-none-any.whl.metadata (4.1 kB)
Collecting pytest==7.4.3 (from -r requirements.txt (line 6))
  Using cached pytest-7.4.3-py3-none-any.whl.metadata (7.9 kB)
Collecting pytest-cov==4.1.0 (from -r requirements.txt (line 7))
  Using cached pytest_cov-4.1.0-py3-none-any.whl.metadata (26 kB)
Collecting gunicorn==21.2.0 (from -r requirements.txt (line 10))
  Using cached gunicorn-21.2.0-py3-none-any.whl.metadata (4.1 kB)
Collecting flake8==6.1.0 (from -r requirements.txt (line 13))
  Using cached flake8-6.1.0-py2.py3-none-any.whl.metadata (3.8 kB)
Collecting black==23.12.1 (from -r requirements.txt (line 14))
  Using cached black-23.12.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (68 kB)
Collecting python-dotenv==1.0.0 (from -r requirements.txt (line 17))
  Using cached python_dotenv-1.0.0-py3-none-any.whl.metadata (21 kB)
Collecting Jinja2>=3.1.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
Collecting itsdangerous>=2.1.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting click>=8.1.3 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached click-8.3.1-py3-none-any.whl.metadata (2.6 kB)
Collecting blinker>=1.6.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting MarkupSafe>=2.1.1 (from Werkzeug==3.0.1->-r requirements.txt (line 3))
  Using cached markupsafe-3.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.7 kB)
Collecting iniconfig (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached iniconfig-2.3.0-py3-none-any.whl.metadata (2.5 kB)
Collecting packaging (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached packaging-25.0-py3-none-any.whl.metadata (3.3 kB)
Collecting pluggy<2.0,>=0.12 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached pluggy-1.6.0-py3-none-any.whl.metadata (4.8 kB)
Collecting exceptiongroup>=1.0.0rc8 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached exceptiongroup-1.3.1-py3-none-any.whl.metadata (6.7 kB)
Collecting tomli>=1.0.0 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached tomli-2.4.0-py3-none-any.whl.metadata (10 kB)
Collecting coverage>=5.2.1 (from coverage[toml]>=5.2.1->pytest-cov==4.1.0->-r requirements.txt (line 7))
  Using cached coverage-7.13.1-cp310-cp310-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl.metadata (8.5 kB)
Collecting mccabe<0.8.0,>=0.7.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached mccabe-0.7.0-py2.py3-none-any.whl.metadata (5.0 kB)
Collecting pycodestyle<2.12.0,>=2.11.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached pycodestyle-2.11.1-py2.py3-none-any.whl.metadata (4.5 kB)
Collecting pyflakes<3.2.0,>=3.1.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached pyflakes-3.1.0-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting mypy-extensions>=0.4.3 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached mypy_extensions-1.1.0-py3-none-any.whl.metadata (1.1 kB)
Collecting pathspec>=0.9.0 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached pathspec-1.0.3-py3-none-any.whl.metadata (13 kB)
Collecting platformdirs>=2 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached platformdirs-4.5.1-py3-none-any.whl.metadata (12 kB)
Collecting typing-extensions>=4.0.1 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached typing_extensions-4.15.0-py3-none-any.whl.metadata (3.3 kB)
Using cached flask-3.0.0-py3-none-any.whl (99 kB)
Using cached werkzeug-3.0.1-py3-none-any.whl (226 kB)
Using cached pytest-7.4.3-py3-none-any.whl (325 kB)
Using cached pytest_cov-4.1.0-py3-none-any.whl (21 kB)
Using cached gunicorn-21.2.0-py3-none-any.whl (80 kB)
Using cached flake8-6.1.0-py2.py3-none-any.whl (58 kB)
Using cached black-23.12.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (1.7 MB)
Using cached python_dotenv-1.0.0-py3-none-any.whl (19 kB)
Using cached mccabe-0.7.0-py2.py3-none-any.whl (7.3 kB)
Using cached pluggy-1.6.0-py3-none-any.whl (20 kB)
Using cached pycodestyle-2.11.1-py2.py3-none-any.whl (31 kB)
Using cached pyflakes-3.1.0-py2.py3-none-any.whl (62 kB)
Using cached blinker-1.9.0-py3-none-any.whl (8.5 kB)
Using cached click-8.3.1-py3-none-any.whl (108 kB)
Using cached coverage-7.13.1-cp310-cp310-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl (247 kB)
Using cached exceptiongroup-1.3.1-py3-none-any.whl (16 kB)
Using cached itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Using cached jinja2-3.1.6-py3-none-any.whl (134 kB)
Using cached markupsafe-3.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (20 kB)
Using cached mypy_extensions-1.1.0-py3-none-any.whl (5.0 kB)
Using cached packaging-25.0-py3-none-any.whl (66 kB)
Using cached pathspec-1.0.3-py3-none-any.whl (55 kB)
Using cached platformdirs-4.5.1-py3-none-any.whl (18 kB)
Using cached tomli-2.4.0-py3-none-any.whl (14 kB)
Using cached typing_extensions-4.15.0-py3-none-any.whl (44 kB)
Using cached iniconfig-2.3.0-py3-none-any.whl (7.5 kB)
Installing collected packages: typing-extensions, tomli, python-dotenv, pyflakes, pycodestyle, pluggy, platformdirs, pathspec, packaging, mypy-extensions, mccabe, MarkupSafe, itsdangerous, iniconfig, coverage, click, blinker, Werkzeug, Jinja2, gunicorn, flake8, exceptiongroup, black, pytest, Flask, pytest-cov

Successfully installed Flask-3.0.0 Jinja2-3.1.6 MarkupSafe-3.0.3 Werkzeug-3.0.1 black-23.12.1 blinker-1.9.0 click-8.3.1 coverage-7.13.1 exceptiongroup-1.3.1 flake8-6.1.0 gunicorn-21.2.0 iniconfig-2.3.0 itsdangerous-2.2.0 mccabe-0.7.0 mypy-extensions-1.1.0 packaging-25.0 pathspec-1.0.3 platformdirs-4.5.1 pluggy-1.6.0 pycodestyle-2.11.1 pyflakes-3.1.0 pytest-7.4.3 pytest-cov-4.1.0 python-dotenv-1.0.0 tomli-2.4.0 typing-extensions-4.15.0
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Code Quality Check)
[Pipeline] echo
Running code quality checks...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ flake8 app.py --max-line-length=120 --exclude=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
Running unit tests...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pytest test_app.py -v --junitxml=test-results.xml --cov=app --cov-report=xml --cov-report=html
============================= test session starts ==============================
platform linux -- Python 3.10.12, pytest-7.4.3, pluggy-1.6.0 -- /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin/python3
cachedir: .pytest_cache
rootdir: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging
plugins: cov-4.1.0
collecting ... collected 11 items

test_app.py::test_home_page PASSED                                       [  9%]
test_app.py::test_health_check PASSED                                    [ 18%]
test_app.py::test_info_endpoint PASSED                                   [ 27%]
test_app.py::test_add_numbers_positive PASSED                            [ 36%]
test_app.py::test_add_numbers_negative PASSED                            [ 45%]
test_app.py::test_add_numbers_zero PASSED                                [ 54%]
test_app.py::test_add_numbers_large PASSED                               [ 63%]
test_app.py::test_invalid_endpoint PASSED                                [ 72%]
test_app.py::test_add_non_integer PASSED                                 [ 81%]
test_app.py::test_health_check_returns_json PASSED                       [ 90%]
test_app.py::test_info_returns_json PASSED                               [100%]

- generated xml file: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/test-results.xml -

---------- coverage: platform linux, python 3.10.12-final-0 ----------
Coverage HTML written to dir htmlcov
Coverage XML written to file coverage.xml

============================== 11 passed in 0.32s ==============================
[Pipeline] }
[Pipeline] // script
Post stage
[Pipeline] junit
Recording test results
[Checks API] No suitable checks publisher found.
[Pipeline] publishHTML
[htmlpublisher] Archiving HTML reports...
[htmlpublisher] Archiving at BUILD level /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/htmlcov to Coverage_20Report
[htmlpublisher] Copying recursive using current thread
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Security Scan)
[Pipeline] echo
Running security vulnerability scan...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install safety bandit
Collecting safety
  Using cached safety-3.7.0-py3-none-any.whl.metadata (11 kB)
Collecting bandit
  Using cached bandit-1.9.3-py3-none-any.whl.metadata (7.1 kB)
Collecting authlib>=1.2.0 (from safety)
  Using cached authlib-1.6.6-py2.py3-none-any.whl.metadata (9.8 kB)
Requirement already satisfied: click>=8.0.2 in ./venv/lib/python3.10/site-packages (from safety) (8.3.1)
Collecting dparse>=0.6.4 (from safety)
  Using cached dparse-0.6.4-py3-none-any.whl.metadata (5.5 kB)
Collecting filelock<4.0,>=3.16.1 (from safety)
  Using cached filelock-3.20.3-py3-none-any.whl.metadata (2.1 kB)
Collecting httpx (from safety)
  Using cached httpx-0.28.1-py3-none-any.whl.metadata (7.1 kB)
Requirement already satisfied: jinja2>=3.1.0 in ./venv/lib/python3.10/site-packages (from safety) (3.1.6)
Collecting marshmallow>=3.15.0 (from safety)
  Using cached marshmallow-4.2.0-py3-none-any.whl.metadata (7.4 kB)
Collecting nltk>=3.9 (from safety)
  Using cached nltk-3.9.2-py3-none-any.whl.metadata (3.2 kB)
Requirement already satisfied: packaging>=21.0 in ./venv/lib/python3.10/site-packages (from safety) (25.0)
Collecting pydantic>=2.6.0 (from safety)
  Using cached pydantic-2.12.5-py3-none-any.whl.metadata (90 kB)
Collecting requests (from safety)
  Using cached requests-2.32.5-py3-none-any.whl.metadata (4.9 kB)
Collecting ruamel-yaml>=0.17.21 (from safety)
  Using cached ruamel_yaml-0.19.1-py3-none-any.whl.metadata (16 kB)
Collecting safety-schemas==0.0.16 (from safety)
  Using cached safety_schemas-0.0.16-py3-none-any.whl.metadata (1.1 kB)
Collecting tenacity>=8.1.0 (from safety)
  Using cached tenacity-9.1.2-py3-none-any.whl.metadata (1.2 kB)
Requirement already satisfied: tomli in ./venv/lib/python3.10/site-packages (from safety) (2.4.0)
Collecting tomlkit (from safety)
  Using cached tomlkit-0.14.0-py3-none-any.whl.metadata (2.8 kB)
Collecting typer>=0.16.0 (from safety)
  Using cached typer-0.21.1-py3-none-any.whl.metadata (16 kB)
Requirement already satisfied: typing-extensions>=4.7.1 in ./venv/lib/python3.10/site-packages (from safety) (4.15.0)
Collecting PyYAML>=5.3.1 (from bandit)
  Using cached pyyaml-6.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.4 kB)
Collecting stevedore>=1.20.0 (from bandit)
  Using cached stevedore-5.6.0-py3-none-any.whl.metadata (2.3 kB)
Collecting rich (from bandit)
  Using cached rich-14.2.0-py3-none-any.whl.metadata (18 kB)
Collecting cryptography (from authlib>=1.2.0->safety)
  Using cached cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl.metadata (5.7 kB)
Requirement already satisfied: MarkupSafe>=2.0 in ./venv/lib/python3.10/site-packages (from jinja2>=3.1.0->safety) (3.0.3)
Collecting backports-datetime-fromisoformat (from marshmallow>=3.15.0->safety)
  Using cached backports_datetime_fromisoformat-2.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (6.3 kB)
Collecting joblib (from nltk>=3.9->safety)
  Using cached joblib-1.5.3-py3-none-any.whl.metadata (5.5 kB)
Collecting regex>=2021.8.3 (from nltk>=3.9->safety)
  Using cached regex-2026.1.15-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (40 kB)
Collecting tqdm (from nltk>=3.9->safety)
  Using cached tqdm-4.67.1-py3-none-any.whl.metadata (57 kB)
Collecting annotated-types>=0.6.0 (from pydantic>=2.6.0->safety)
  Using cached annotated_types-0.7.0-py3-none-any.whl.metadata (15 kB)
Collecting pydantic-core==2.41.5 (from pydantic>=2.6.0->safety)
  Using cached pydantic_core-2.41.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (7.3 kB)
Collecting typing-inspection>=0.4.2 (from pydantic>=2.6.0->safety)
  Using cached typing_inspection-0.4.2-py3-none-any.whl.metadata (2.6 kB)
Collecting shellingham>=1.3.0 (from typer>=0.16.0->safety)
  Using cached shellingham-1.5.4-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting markdown-it-py>=2.2.0 (from rich->bandit)
  Using cached markdown_it_py-4.0.0-py3-none-any.whl.metadata (7.3 kB)
Collecting pygments<3.0.0,>=2.13.0 (from rich->bandit)
  Using cached pygments-2.19.2-py3-none-any.whl.metadata (2.5 kB)
Collecting mdurl~=0.1 (from markdown-it-py>=2.2.0->rich->bandit)
  Using cached mdurl-0.1.2-py3-none-any.whl.metadata (1.6 kB)
Collecting cffi>=2.0.0 (from cryptography->authlib>=1.2.0->safety)
  Using cached cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (2.6 kB)
Collecting pycparser (from cffi>=2.0.0->cryptography->authlib>=1.2.0->safety)
  Using cached pycparser-2.23-py3-none-any.whl.metadata (993 bytes)
Collecting anyio (from httpx->safety)
  Using cached anyio-4.12.1-py3-none-any.whl.metadata (4.3 kB)
Collecting certifi (from httpx->safety)
  Using cached certifi-2026.1.4-py3-none-any.whl.metadata (2.5 kB)
Collecting httpcore==1.* (from httpx->safety)
  Using cached httpcore-1.0.9-py3-none-any.whl.metadata (21 kB)
Collecting idna (from httpx->safety)
  Using cached idna-3.11-py3-none-any.whl.metadata (8.4 kB)
Collecting h11>=0.16 (from httpcore==1.*->httpx->safety)
  Using cached h11-0.16.0-py3-none-any.whl.metadata (8.3 kB)
Requirement already satisfied: exceptiongroup>=1.0.2 in ./venv/lib/python3.10/site-packages (from anyio->httpx->safety) (1.3.1)
Collecting charset_normalizer<4,>=2 (from requests->safety)
  Using cached charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (37 kB)
Collecting urllib3<3,>=1.21.1 (from requests->safety)
  Using cached urllib3-2.6.3-py3-none-any.whl.metadata (6.9 kB)
Using cached safety-3.7.0-py3-none-any.whl (312 kB)
Using cached safety_schemas-0.0.16-py3-none-any.whl (39 kB)
Using cached filelock-3.20.3-py3-none-any.whl (16 kB)
Using cached bandit-1.9.3-py3-none-any.whl (134 kB)
Using cached authlib-1.6.6-py2.py3-none-any.whl (244 kB)
Using cached dparse-0.6.4-py3-none-any.whl (11 kB)
Using cached marshmallow-4.2.0-py3-none-any.whl (48 kB)
Using cached nltk-3.9.2-py3-none-any.whl (1.5 MB)
Using cached pydantic-2.12.5-py3-none-any.whl (463 kB)
Using cached pydantic_core-2.41.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.1 MB)
Using cached annotated_types-0.7.0-py3-none-any.whl (13 kB)
Using cached pyyaml-6.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (770 kB)
Using cached regex-2026.1.15-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (791 kB)
Using cached ruamel_yaml-0.19.1-py3-none-any.whl (118 kB)
Using cached stevedore-5.6.0-py3-none-any.whl (54 kB)
Using cached tenacity-9.1.2-py3-none-any.whl (28 kB)
Using cached typer-0.21.1-py3-none-any.whl (47 kB)
Using cached rich-14.2.0-py3-none-any.whl (243 kB)
Using cached pygments-2.19.2-py3-none-any.whl (1.2 MB)
Using cached markdown_it_py-4.0.0-py3-none-any.whl (87 kB)
Using cached mdurl-0.1.2-py3-none-any.whl (10.0 kB)
Using cached shellingham-1.5.4-py2.py3-none-any.whl (9.8 kB)
Using cached typing_inspection-0.4.2-py3-none-any.whl (14 kB)
Using cached backports_datetime_fromisoformat-2.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (52 kB)
Using cached cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl (4.5 MB)
Using cached cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (216 kB)
Using cached httpx-0.28.1-py3-none-any.whl (73 kB)
Using cached httpcore-1.0.9-py3-none-any.whl (78 kB)
Using cached h11-0.16.0-py3-none-any.whl (37 kB)
Using cached anyio-4.12.1-py3-none-any.whl (113 kB)
Using cached idna-3.11-py3-none-any.whl (71 kB)
Using cached certifi-2026.1.4-py3-none-any.whl (152 kB)
Using cached joblib-1.5.3-py3-none-any.whl (309 kB)
Using cached pycparser-2.23-py3-none-any.whl (118 kB)
Using cached requests-2.32.5-py3-none-any.whl (64 kB)
Using cached charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (153 kB)
Using cached urllib3-2.6.3-py3-none-any.whl (131 kB)
Using cached tomlkit-0.14.0-py3-none-any.whl (39 kB)
Using cached tqdm-4.67.1-py3-none-any.whl (78 kB)
Installing collected packages: urllib3, typing-inspection, tqdm, tomlkit, tenacity, stevedore, shellingham, ruamel-yaml, regex, PyYAML, pygments, pydantic-core, pycparser, mdurl, joblib, idna, h11, filelock, dparse, charset_normalizer, certifi, backports-datetime-fromisoformat, annotated-types, requests, pydantic, nltk, marshmallow, markdown-it-py, httpcore, cffi, anyio, safety-schemas, rich, httpx, cryptography, typer, bandit, authlib, safety

Successfully installed PyYAML-6.0.3 annotated-types-0.7.0 anyio-4.12.1 authlib-1.6.6 backports-datetime-fromisoformat-2.0.3 bandit-1.9.3 certifi-2026.1.4 cffi-2.0.0 charset_normalizer-3.4.4 cryptography-46.0.3 dparse-0.6.4 filelock-3.20.3 h11-0.16.0 httpcore-1.0.9 httpx-0.28.1 idna-3.11 joblib-1.5.3 markdown-it-py-4.0.0 marshmallow-4.2.0 mdurl-0.1.2 nltk-3.9.2 pycparser-2.23 pydantic-2.12.5 pydantic-core-2.41.5 pygments-2.19.2 regex-2026.1.15 requests-2.32.5 rich-14.2.0 ruamel-yaml-0.19.1 safety-3.7.0 safety-schemas-0.0.16 shellingham-1.5.4 stevedore-5.6.0 tenacity-9.1.2 tomlkit-0.14.0 tqdm-4.67.1 typer-0.21.1 typing-inspection-0.4.2 urllib3-2.6.3
+ echo Running Safety check...
Running Safety check...
+ safety check --json
+ true
+ echo Running Bandit security scan...
Running Bandit security scan...
+ bandit -r app.py -f json -o bandit-report.json
[main]	INFO	profile include tests: None
[main]	INFO	profile exclude tests: None
[main]	INFO	cli include tests: None
[main]	INFO	cli exclude tests: None
[json]	INFO	JSON output written to file: bandit-report.json
+ true
+ echo Security scans completed!
Security scans completed!
[Pipeline] }
[Pipeline] // script
Post stage
[Pipeline] archiveArtifacts
Archiving artifacts
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build Artifact)
[Pipeline] echo
Creating deployment artifact...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ tar -czf flask-app-11.tar.gz --exclude=venv --exclude=*.pyc --exclude=__pycache__ app.py requirements.txt
[Pipeline] }
[Pipeline] // script
[Pipeline] archiveArtifacts
Archiving artifacts
Recording fingerprints
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to Staging)
[Pipeline] echo
Deploying to staging environment...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sshagent
[ssh-agent] Using credentials ****
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXWjDRQL/agent.5888
SSH_AGENT_PID=5891
Running ssh-add (command line suppressed)
Identity added: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging@tmp/private_key_17421059893353317528.key (/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging@tmp/private_key_17421059893353317528.key)
[ssh-agent] Started.
[Pipeline] {
[Pipeline] sh
+ echo Deploying to staging server: ****
Deploying to staging server: ****
+ echo Copying files to staging server...
Copying files to staging server...
+ scp -o StrictHostKeyChecking=no flask-app-11.tar.gz ****@****:/var/www/flask-app
+ echo Extracting and installing on staging server...
Extracting and installing on staging server...
+ ssh -o StrictHostKeyChecking=no ****@**** cd /var/www/flask-app && tar -xzf flask-app-11.tar.gz
+ ssh -o StrictHostKeyChecking=no ****@**** cd /var/www/flask-app && python3 -m venv venv && venv/bin/pip install -r requirements.txt
Requirement already satisfied: Flask==3.0.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 2)) (3.0.0)
Requirement already satisfied: Werkzeug==3.0.1 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 3)) (3.0.1)
Requirement already satisfied: pytest==7.4.3 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 6)) (7.4.3)
Requirement already satisfied: pytest-cov==4.1.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 7)) (4.1.0)
Requirement already satisfied: gunicorn==21.2.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 10)) (21.2.0)
Requirement already satisfied: flake8==6.1.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 13)) (6.1.0)
Requirement already satisfied: black==23.12.1 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 14)) (23.12.1)
Requirement already satisfied: python-dotenv==1.0.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 17)) (1.0.0)
Requirement already satisfied: Jinja2>=3.1.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (3.1.6)
Requirement already satisfied: itsdangerous>=2.1.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (2.2.0)
Requirement already satisfied: click>=8.1.3 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (8.3.1)
Requirement already satisfied: blinker>=1.6.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (1.9.0)
Requirement already satisfied: MarkupSafe>=2.1.1 in ./venv/lib/python3.12/site-packages (from Werkzeug==3.0.1->-r requirements.txt (line 3)) (3.0.3)
Requirement already satisfied: iniconfig in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (2.3.0)
Requirement already satisfied: packaging in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (25.0)
Requirement already satisfied: pluggy<2.0,>=0.12 in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (1.6.0)
Requirement already satisfied: coverage>=5.2.1 in ./venv/lib/python3.12/site-packages (from coverage[toml]>=5.2.1->pytest-cov==4.1.0->-r requirements.txt (line 7)) (7.13.1)
Requirement already satisfied: mccabe<0.8.0,>=0.7.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (0.7.0)
Requirement already satisfied: pycodestyle<2.12.0,>=2.11.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (2.11.1)
Requirement already satisfied: pyflakes<3.2.0,>=3.1.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (3.1.0)
Requirement already satisfied: mypy-extensions>=0.4.3 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (1.1.0)
Requirement already satisfied: pathspec>=0.9.0 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (1.0.3)
Requirement already satisfied: platformdirs>=2 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (4.5.1)
+ echo Restarting Flask service on staging...
Restarting Flask service on staging...
+ ssh -o StrictHostKeyChecking=no ****@**** sudo systemctl restart flask-app
Warning: The unit file, source configuration file or drop-ins of flask-app.service changed on disk. Run 'systemctl daemon-reload' to reload units.
+ echo Deployment to staging completed successfully!
Deployment to staging completed successfully!
[Pipeline] }
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 5891 killed;
[ssh-agent] Stopped.
[Pipeline] // sshagent
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to Production)
Stage "Deploy to Production" skipped due to when conditional
[Pipeline] getContext
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Smoke Test - Staging)
[Pipeline] echo
Running smoke tests on staging...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ echo Waiting for service to start...
Waiting for service to start...
+ sleep 5
+ echo Testing health endpoint...
Testing health endpoint...
+ curl -f http://****/api/health
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    69  100    69    0     0    101      0 --:--:-- --:--:-- --:--:--   101
{"message":"Application is running successfully","status":"healthy"}
+ echo Testing info endpoint...
Testing info endpoint...
+ curl -f http://****/api/info
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   165  100   165    0     0    280      0 --:--:-- --:--:-- --:--:--   280
{"description":"A simple Flask application demonstrating CI/CD with Jenkins and GitHub Actions","environment":"staging","name":"Flask CI/CD Demo","version":"1.0.0"}
+ echo All smoke tests passed!
All smoke tests passed!
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Health Check - Production)
Stage "Health Check - Production" skipped due to when conditional
[Pipeline] getContext
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Cleaning up workspace...
[Pipeline] cleanWs
[WS-CLEANUP] Deleting project workspace...
[WS-CLEANUP] Deferred wipeout is used...
[WS-CLEANUP] done
[Pipeline] echo
Pipeline executed successfully!
[Pipeline] emailext
Sending email to: ghanshyamjobi+jenkins@gmail.com
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS

```

### Stage 4. Email received

<img width="1167" height="444" alt="image" src="https://github.com/user-attachments/assets/cedc0005-a15e-4c91-a74a-c436510094bc" />


## 🔧 Jenkins Multibranch Pipeline Stages

### Stage 1: configure pipeline

<img width="1209" height="775" alt="image" src="https://github.com/user-attachments/assets/58a9388e-4dec-4bf6-a68b-f7f424f7fe58" />

<img width="1204" height="767" alt="image" src="https://github.com/user-attachments/assets/f7d0b86c-3970-436b-85be-38f8ddb0b668" />

<img width="1210" height="765" alt="image" src="https://github.com/user-attachments/assets/c3667224-7f2d-40ce-920a-08fb93f92f3a" />

<img width="1203" height="767" alt="image" src="https://github.com/user-attachments/assets/84c7f310-7bf0-492f-9a4a-8412e75416c4" />

### Stage 2: Push code staging branch

```
git add app.py
git commit -m "push code to staging"
git push origin staging
```
### Stage 2: Push code production tag branch

```
git tag -a v1.3.0 -m "Release version 1.3.0"
git push origin v1.3.0
```
### Stage 3. CI/CD logs for staging

```
Started by user admin
 > git rev-parse --resolve-git-dir /var/lib/jenkins/caches/git-640a1658cad41f253def4865ccabb7b3/.git # timeout=10
Setting origin to https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
Fetching origin...
Fetching upstream changes from origin
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git config --get remote.origin.url # timeout=10
 > git fetch --no-tags --force --progress -- origin +refs/heads/*:refs/remotes/origin/* # timeout=10
Seen branch in repository origin/feature/changes
Seen branch in repository origin/main
Seen branch in repository origin/revert-4-feature/changes
Seen branch in repository origin/staging
Seen 4 remote branches
Obtained Jenkinsfile from f725c1df22493fef0e38bc2f7f441b876bafd67a
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning with configured refspecs honoured and without tags
Cloning repository https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git init /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging # timeout=10
Fetching upstream changes from https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --no-tags --force --progress -- https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision f725c1df22493fef0e38bc2f7f441b876bafd67a (staging)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f f725c1df22493fef0e38bc2f7f441b876bafd67a # timeout=10
Commit message: "Merge pull request #6 from ghanshyamca/feature/changes"
 > git rev-list --no-walk f725c1df22493fef0e38bc2f7f441b876bafd67a # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withCredentials
Masking supported pattern matches of $PRODUCTION_HOST or $PRODUCTION_USER or $STAGING_USER or $STAGING_HOST
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Checkout)
[Pipeline] echo
Checking out source code...
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --no-tags --force --progress -- https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Checking out Revision f725c1df22493fef0e38bc2f7f441b876bafd67a (staging)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f f725c1df22493fef0e38bc2f7f441b876bafd67a # timeout=10
Commit message: "Merge pull request #6 from ghanshyamca/feature/changes"
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Setup Environment)
[Pipeline] echo
Setting up Python virtual environment...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ python3 -m venv venv
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install --upgrade pip
Requirement already satisfied: pip in ./venv/lib/python3.10/site-packages (22.0.2)
Collecting pip
  Using cached pip-25.3-py3-none-any.whl (1.8 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 22.0.2
    Uninstalling pip-22.0.2:
      Successfully uninstalled pip-22.0.2
Successfully installed pip-25.3
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
Installing dependencies...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install -r requirements.txt
Collecting Flask==3.0.0 (from -r requirements.txt (line 2))
  Using cached flask-3.0.0-py3-none-any.whl.metadata (3.6 kB)
Collecting Werkzeug==3.0.1 (from -r requirements.txt (line 3))
  Using cached werkzeug-3.0.1-py3-none-any.whl.metadata (4.1 kB)
Collecting pytest==7.4.3 (from -r requirements.txt (line 6))
  Using cached pytest-7.4.3-py3-none-any.whl.metadata (7.9 kB)
Collecting pytest-cov==4.1.0 (from -r requirements.txt (line 7))
  Using cached pytest_cov-4.1.0-py3-none-any.whl.metadata (26 kB)
Collecting gunicorn==21.2.0 (from -r requirements.txt (line 10))
  Using cached gunicorn-21.2.0-py3-none-any.whl.metadata (4.1 kB)
Collecting flake8==6.1.0 (from -r requirements.txt (line 13))
  Using cached flake8-6.1.0-py2.py3-none-any.whl.metadata (3.8 kB)
Collecting black==23.12.1 (from -r requirements.txt (line 14))
  Using cached black-23.12.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (68 kB)
Collecting python-dotenv==1.0.0 (from -r requirements.txt (line 17))
  Using cached python_dotenv-1.0.0-py3-none-any.whl.metadata (21 kB)
Collecting Jinja2>=3.1.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
Collecting itsdangerous>=2.1.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting click>=8.1.3 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached click-8.3.1-py3-none-any.whl.metadata (2.6 kB)
Collecting blinker>=1.6.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting MarkupSafe>=2.1.1 (from Werkzeug==3.0.1->-r requirements.txt (line 3))
  Using cached markupsafe-3.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.7 kB)
Collecting iniconfig (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached iniconfig-2.3.0-py3-none-any.whl.metadata (2.5 kB)
Collecting packaging (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached packaging-25.0-py3-none-any.whl.metadata (3.3 kB)
Collecting pluggy<2.0,>=0.12 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached pluggy-1.6.0-py3-none-any.whl.metadata (4.8 kB)
Collecting exceptiongroup>=1.0.0rc8 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached exceptiongroup-1.3.1-py3-none-any.whl.metadata (6.7 kB)
Collecting tomli>=1.0.0 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached tomli-2.4.0-py3-none-any.whl.metadata (10 kB)
Collecting coverage>=5.2.1 (from coverage[toml]>=5.2.1->pytest-cov==4.1.0->-r requirements.txt (line 7))
  Using cached coverage-7.13.1-cp310-cp310-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl.metadata (8.5 kB)
Collecting mccabe<0.8.0,>=0.7.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached mccabe-0.7.0-py2.py3-none-any.whl.metadata (5.0 kB)
Collecting pycodestyle<2.12.0,>=2.11.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached pycodestyle-2.11.1-py2.py3-none-any.whl.metadata (4.5 kB)
Collecting pyflakes<3.2.0,>=3.1.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached pyflakes-3.1.0-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting mypy-extensions>=0.4.3 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached mypy_extensions-1.1.0-py3-none-any.whl.metadata (1.1 kB)
Collecting pathspec>=0.9.0 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached pathspec-1.0.3-py3-none-any.whl.metadata (13 kB)
Collecting platformdirs>=2 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached platformdirs-4.5.1-py3-none-any.whl.metadata (12 kB)
Collecting typing-extensions>=4.0.1 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached typing_extensions-4.15.0-py3-none-any.whl.metadata (3.3 kB)
Using cached flask-3.0.0-py3-none-any.whl (99 kB)
Using cached werkzeug-3.0.1-py3-none-any.whl (226 kB)
Using cached pytest-7.4.3-py3-none-any.whl (325 kB)
Using cached pytest_cov-4.1.0-py3-none-any.whl (21 kB)
Using cached gunicorn-21.2.0-py3-none-any.whl (80 kB)
Using cached flake8-6.1.0-py2.py3-none-any.whl (58 kB)
Using cached black-23.12.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (1.7 MB)
Using cached python_dotenv-1.0.0-py3-none-any.whl (19 kB)
Using cached mccabe-0.7.0-py2.py3-none-any.whl (7.3 kB)
Using cached pluggy-1.6.0-py3-none-any.whl (20 kB)
Using cached pycodestyle-2.11.1-py2.py3-none-any.whl (31 kB)
Using cached pyflakes-3.1.0-py2.py3-none-any.whl (62 kB)
Using cached blinker-1.9.0-py3-none-any.whl (8.5 kB)
Using cached click-8.3.1-py3-none-any.whl (108 kB)
Using cached coverage-7.13.1-cp310-cp310-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl (247 kB)
Using cached exceptiongroup-1.3.1-py3-none-any.whl (16 kB)
Using cached itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Using cached jinja2-3.1.6-py3-none-any.whl (134 kB)
Using cached markupsafe-3.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (20 kB)
Using cached mypy_extensions-1.1.0-py3-none-any.whl (5.0 kB)
Using cached packaging-25.0-py3-none-any.whl (66 kB)
Using cached pathspec-1.0.3-py3-none-any.whl (55 kB)
Using cached platformdirs-4.5.1-py3-none-any.whl (18 kB)
Using cached tomli-2.4.0-py3-none-any.whl (14 kB)
Using cached typing_extensions-4.15.0-py3-none-any.whl (44 kB)
Using cached iniconfig-2.3.0-py3-none-any.whl (7.5 kB)
Installing collected packages: typing-extensions, tomli, python-dotenv, pyflakes, pycodestyle, pluggy, platformdirs, pathspec, packaging, mypy-extensions, mccabe, MarkupSafe, itsdangerous, iniconfig, coverage, click, blinker, Werkzeug, Jinja2, gunicorn, flake8, exceptiongroup, black, pytest, Flask, pytest-cov

Successfully installed Flask-3.0.0 Jinja2-3.1.6 MarkupSafe-3.0.3 Werkzeug-3.0.1 black-23.12.1 blinker-1.9.0 click-8.3.1 coverage-7.13.1 exceptiongroup-1.3.1 flake8-6.1.0 gunicorn-21.2.0 iniconfig-2.3.0 itsdangerous-2.2.0 mccabe-0.7.0 mypy-extensions-1.1.0 packaging-25.0 pathspec-1.0.3 platformdirs-4.5.1 pluggy-1.6.0 pycodestyle-2.11.1 pyflakes-3.1.0 pytest-7.4.3 pytest-cov-4.1.0 python-dotenv-1.0.0 tomli-2.4.0 typing-extensions-4.15.0
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Code Quality Check)
[Pipeline] echo
Running code quality checks...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ flake8 app.py --max-line-length=120 --exclude=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
Running unit tests...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pytest test_app.py -v --junitxml=test-results.xml --cov=app --cov-report=xml --cov-report=html
============================= test session starts ==============================
platform linux -- Python 3.10.12, pytest-7.4.3, pluggy-1.6.0 -- /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin/python3
cachedir: .pytest_cache
rootdir: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging
plugins: cov-4.1.0
collecting ... collected 11 items

test_app.py::test_home_page PASSED                                       [  9%]
test_app.py::test_health_check PASSED                                    [ 18%]
test_app.py::test_info_endpoint PASSED                                   [ 27%]
test_app.py::test_add_numbers_positive PASSED                            [ 36%]
test_app.py::test_add_numbers_negative PASSED                            [ 45%]
test_app.py::test_add_numbers_zero PASSED                                [ 54%]
test_app.py::test_add_numbers_large PASSED                               [ 63%]
test_app.py::test_invalid_endpoint PASSED                                [ 72%]
test_app.py::test_add_non_integer PASSED                                 [ 81%]
test_app.py::test_health_check_returns_json PASSED                       [ 90%]
test_app.py::test_info_returns_json PASSED                               [100%]

- generated xml file: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/test-results.xml -

---------- coverage: platform linux, python 3.10.12-final-0 ----------
Coverage HTML written to dir htmlcov
Coverage XML written to file coverage.xml

============================== 11 passed in 0.32s ==============================
[Pipeline] }
[Pipeline] // script
Post stage
[Pipeline] junit
Recording test results
[Checks API] No suitable checks publisher found.
[Pipeline] publishHTML
[htmlpublisher] Archiving HTML reports...
[htmlpublisher] Archiving at BUILD level /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/htmlcov to Coverage_20Report
[htmlpublisher] Copying recursive using current thread
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Security Scan)
[Pipeline] echo
Running security vulnerability scan...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install safety bandit
Collecting safety
  Using cached safety-3.7.0-py3-none-any.whl.metadata (11 kB)
Collecting bandit
  Using cached bandit-1.9.3-py3-none-any.whl.metadata (7.1 kB)
Collecting authlib>=1.2.0 (from safety)
  Using cached authlib-1.6.6-py2.py3-none-any.whl.metadata (9.8 kB)
Requirement already satisfied: click>=8.0.2 in ./venv/lib/python3.10/site-packages (from safety) (8.3.1)
Collecting dparse>=0.6.4 (from safety)
  Using cached dparse-0.6.4-py3-none-any.whl.metadata (5.5 kB)
Collecting filelock<4.0,>=3.16.1 (from safety)
  Using cached filelock-3.20.3-py3-none-any.whl.metadata (2.1 kB)
Collecting httpx (from safety)
  Using cached httpx-0.28.1-py3-none-any.whl.metadata (7.1 kB)
Requirement already satisfied: jinja2>=3.1.0 in ./venv/lib/python3.10/site-packages (from safety) (3.1.6)
Collecting marshmallow>=3.15.0 (from safety)
  Using cached marshmallow-4.2.0-py3-none-any.whl.metadata (7.4 kB)
Collecting nltk>=3.9 (from safety)
  Using cached nltk-3.9.2-py3-none-any.whl.metadata (3.2 kB)
Requirement already satisfied: packaging>=21.0 in ./venv/lib/python3.10/site-packages (from safety) (25.0)
Collecting pydantic>=2.6.0 (from safety)
  Using cached pydantic-2.12.5-py3-none-any.whl.metadata (90 kB)
Collecting requests (from safety)
  Using cached requests-2.32.5-py3-none-any.whl.metadata (4.9 kB)
Collecting ruamel-yaml>=0.17.21 (from safety)
  Using cached ruamel_yaml-0.19.1-py3-none-any.whl.metadata (16 kB)
Collecting safety-schemas==0.0.16 (from safety)
  Using cached safety_schemas-0.0.16-py3-none-any.whl.metadata (1.1 kB)
Collecting tenacity>=8.1.0 (from safety)
  Using cached tenacity-9.1.2-py3-none-any.whl.metadata (1.2 kB)
Requirement already satisfied: tomli in ./venv/lib/python3.10/site-packages (from safety) (2.4.0)
Collecting tomlkit (from safety)
  Using cached tomlkit-0.14.0-py3-none-any.whl.metadata (2.8 kB)
Collecting typer>=0.16.0 (from safety)
  Using cached typer-0.21.1-py3-none-any.whl.metadata (16 kB)
Requirement already satisfied: typing-extensions>=4.7.1 in ./venv/lib/python3.10/site-packages (from safety) (4.15.0)
Collecting PyYAML>=5.3.1 (from bandit)
  Using cached pyyaml-6.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.4 kB)
Collecting stevedore>=1.20.0 (from bandit)
  Using cached stevedore-5.6.0-py3-none-any.whl.metadata (2.3 kB)
Collecting rich (from bandit)
  Using cached rich-14.2.0-py3-none-any.whl.metadata (18 kB)
Collecting cryptography (from authlib>=1.2.0->safety)
  Using cached cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl.metadata (5.7 kB)
Requirement already satisfied: MarkupSafe>=2.0 in ./venv/lib/python3.10/site-packages (from jinja2>=3.1.0->safety) (3.0.3)
Collecting backports-datetime-fromisoformat (from marshmallow>=3.15.0->safety)
  Using cached backports_datetime_fromisoformat-2.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (6.3 kB)
Collecting joblib (from nltk>=3.9->safety)
  Using cached joblib-1.5.3-py3-none-any.whl.metadata (5.5 kB)
Collecting regex>=2021.8.3 (from nltk>=3.9->safety)
  Using cached regex-2026.1.15-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (40 kB)
Collecting tqdm (from nltk>=3.9->safety)
  Using cached tqdm-4.67.1-py3-none-any.whl.metadata (57 kB)
Collecting annotated-types>=0.6.0 (from pydantic>=2.6.0->safety)
  Using cached annotated_types-0.7.0-py3-none-any.whl.metadata (15 kB)
Collecting pydantic-core==2.41.5 (from pydantic>=2.6.0->safety)
  Using cached pydantic_core-2.41.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (7.3 kB)
Collecting typing-inspection>=0.4.2 (from pydantic>=2.6.0->safety)
  Using cached typing_inspection-0.4.2-py3-none-any.whl.metadata (2.6 kB)
Collecting shellingham>=1.3.0 (from typer>=0.16.0->safety)
  Using cached shellingham-1.5.4-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting markdown-it-py>=2.2.0 (from rich->bandit)
  Using cached markdown_it_py-4.0.0-py3-none-any.whl.metadata (7.3 kB)
Collecting pygments<3.0.0,>=2.13.0 (from rich->bandit)
  Using cached pygments-2.19.2-py3-none-any.whl.metadata (2.5 kB)
Collecting mdurl~=0.1 (from markdown-it-py>=2.2.0->rich->bandit)
  Using cached mdurl-0.1.2-py3-none-any.whl.metadata (1.6 kB)
Collecting cffi>=2.0.0 (from cryptography->authlib>=1.2.0->safety)
  Using cached cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (2.6 kB)
Collecting pycparser (from cffi>=2.0.0->cryptography->authlib>=1.2.0->safety)
  Using cached pycparser-2.23-py3-none-any.whl.metadata (993 bytes)
Collecting anyio (from httpx->safety)
  Using cached anyio-4.12.1-py3-none-any.whl.metadata (4.3 kB)
Collecting certifi (from httpx->safety)
  Using cached certifi-2026.1.4-py3-none-any.whl.metadata (2.5 kB)
Collecting httpcore==1.* (from httpx->safety)
  Using cached httpcore-1.0.9-py3-none-any.whl.metadata (21 kB)
Collecting idna (from httpx->safety)
  Using cached idna-3.11-py3-none-any.whl.metadata (8.4 kB)
Collecting h11>=0.16 (from httpcore==1.*->httpx->safety)
  Using cached h11-0.16.0-py3-none-any.whl.metadata (8.3 kB)
Requirement already satisfied: exceptiongroup>=1.0.2 in ./venv/lib/python3.10/site-packages (from anyio->httpx->safety) (1.3.1)
Collecting charset_normalizer<4,>=2 (from requests->safety)
  Using cached charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (37 kB)
Collecting urllib3<3,>=1.21.1 (from requests->safety)
  Using cached urllib3-2.6.3-py3-none-any.whl.metadata (6.9 kB)
Using cached safety-3.7.0-py3-none-any.whl (312 kB)
Using cached safety_schemas-0.0.16-py3-none-any.whl (39 kB)
Using cached filelock-3.20.3-py3-none-any.whl (16 kB)
Using cached bandit-1.9.3-py3-none-any.whl (134 kB)
Using cached authlib-1.6.6-py2.py3-none-any.whl (244 kB)
Using cached dparse-0.6.4-py3-none-any.whl (11 kB)
Using cached marshmallow-4.2.0-py3-none-any.whl (48 kB)
Using cached nltk-3.9.2-py3-none-any.whl (1.5 MB)
Using cached pydantic-2.12.5-py3-none-any.whl (463 kB)
Using cached pydantic_core-2.41.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.1 MB)
Using cached annotated_types-0.7.0-py3-none-any.whl (13 kB)
Using cached pyyaml-6.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (770 kB)
Using cached regex-2026.1.15-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (791 kB)
Using cached ruamel_yaml-0.19.1-py3-none-any.whl (118 kB)
Using cached stevedore-5.6.0-py3-none-any.whl (54 kB)
Using cached tenacity-9.1.2-py3-none-any.whl (28 kB)
Using cached typer-0.21.1-py3-none-any.whl (47 kB)
Using cached rich-14.2.0-py3-none-any.whl (243 kB)
Using cached pygments-2.19.2-py3-none-any.whl (1.2 MB)
Using cached markdown_it_py-4.0.0-py3-none-any.whl (87 kB)
Using cached mdurl-0.1.2-py3-none-any.whl (10.0 kB)
Using cached shellingham-1.5.4-py2.py3-none-any.whl (9.8 kB)
Using cached typing_inspection-0.4.2-py3-none-any.whl (14 kB)
Using cached backports_datetime_fromisoformat-2.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (52 kB)
Using cached cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl (4.5 MB)
Using cached cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (216 kB)
Using cached httpx-0.28.1-py3-none-any.whl (73 kB)
Using cached httpcore-1.0.9-py3-none-any.whl (78 kB)
Using cached h11-0.16.0-py3-none-any.whl (37 kB)
Using cached anyio-4.12.1-py3-none-any.whl (113 kB)
Using cached idna-3.11-py3-none-any.whl (71 kB)
Using cached certifi-2026.1.4-py3-none-any.whl (152 kB)
Using cached joblib-1.5.3-py3-none-any.whl (309 kB)
Using cached pycparser-2.23-py3-none-any.whl (118 kB)
Using cached requests-2.32.5-py3-none-any.whl (64 kB)
Using cached charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (153 kB)
Using cached urllib3-2.6.3-py3-none-any.whl (131 kB)
Using cached tomlkit-0.14.0-py3-none-any.whl (39 kB)
Using cached tqdm-4.67.1-py3-none-any.whl (78 kB)
Installing collected packages: urllib3, typing-inspection, tqdm, tomlkit, tenacity, stevedore, shellingham, ruamel-yaml, regex, PyYAML, pygments, pydantic-core, pycparser, mdurl, joblib, idna, h11, filelock, dparse, charset_normalizer, certifi, backports-datetime-fromisoformat, annotated-types, requests, pydantic, nltk, marshmallow, markdown-it-py, httpcore, cffi, anyio, safety-schemas, rich, httpx, cryptography, typer, bandit, authlib, safety

Successfully installed PyYAML-6.0.3 annotated-types-0.7.0 anyio-4.12.1 authlib-1.6.6 backports-datetime-fromisoformat-2.0.3 bandit-1.9.3 certifi-2026.1.4 cffi-2.0.0 charset_normalizer-3.4.4 cryptography-46.0.3 dparse-0.6.4 filelock-3.20.3 h11-0.16.0 httpcore-1.0.9 httpx-0.28.1 idna-3.11 joblib-1.5.3 markdown-it-py-4.0.0 marshmallow-4.2.0 mdurl-0.1.2 nltk-3.9.2 pycparser-2.23 pydantic-2.12.5 pydantic-core-2.41.5 pygments-2.19.2 regex-2026.1.15 requests-2.32.5 rich-14.2.0 ruamel-yaml-0.19.1 safety-3.7.0 safety-schemas-0.0.16 shellingham-1.5.4 stevedore-5.6.0 tenacity-9.1.2 tomlkit-0.14.0 tqdm-4.67.1 typer-0.21.1 typing-inspection-0.4.2 urllib3-2.6.3
+ echo Running Safety check...
Running Safety check...
+ safety check --json
+ true
+ echo Running Bandit security scan...
Running Bandit security scan...
+ bandit -r app.py -f json -o bandit-report.json
[main]	INFO	profile include tests: None
[main]	INFO	profile exclude tests: None
[main]	INFO	cli include tests: None
[main]	INFO	cli exclude tests: None
[json]	INFO	JSON output written to file: bandit-report.json
+ true
+ echo Security scans completed!
Security scans completed!
[Pipeline] }
[Pipeline] // script
Post stage
[Pipeline] archiveArtifacts
Archiving artifacts
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build Artifact)
[Pipeline] echo
Creating deployment artifact...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ tar -czf flask-app-11.tar.gz --exclude=venv --exclude=*.pyc --exclude=__pycache__ app.py requirements.txt
[Pipeline] }
[Pipeline] // script
[Pipeline] archiveArtifacts
Archiving artifacts
Recording fingerprints
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to Staging)
[Pipeline] echo
Deploying to staging environment...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sshagent
[ssh-agent] Using credentials ****
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXWjDRQL/agent.5888
SSH_AGENT_PID=5891
Running ssh-add (command line suppressed)
Identity added: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging@tmp/private_key_17421059893353317528.key (/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_staging@tmp/private_key_17421059893353317528.key)
[ssh-agent] Started.
[Pipeline] {
[Pipeline] sh
+ echo Deploying to staging server: ****
Deploying to staging server: ****
+ echo Copying files to staging server...
Copying files to staging server...
+ scp -o StrictHostKeyChecking=no flask-app-11.tar.gz ****@****:/var/www/flask-app
+ echo Extracting and installing on staging server...
Extracting and installing on staging server...
+ ssh -o StrictHostKeyChecking=no ****@**** cd /var/www/flask-app && tar -xzf flask-app-11.tar.gz
+ ssh -o StrictHostKeyChecking=no ****@**** cd /var/www/flask-app && python3 -m venv venv && venv/bin/pip install -r requirements.txt
Requirement already satisfied: Flask==3.0.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 2)) (3.0.0)
Requirement already satisfied: Werkzeug==3.0.1 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 3)) (3.0.1)
Requirement already satisfied: pytest==7.4.3 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 6)) (7.4.3)
Requirement already satisfied: pytest-cov==4.1.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 7)) (4.1.0)
Requirement already satisfied: gunicorn==21.2.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 10)) (21.2.0)
Requirement already satisfied: flake8==6.1.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 13)) (6.1.0)
Requirement already satisfied: black==23.12.1 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 14)) (23.12.1)
Requirement already satisfied: python-dotenv==1.0.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 17)) (1.0.0)
Requirement already satisfied: Jinja2>=3.1.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (3.1.6)
Requirement already satisfied: itsdangerous>=2.1.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (2.2.0)
Requirement already satisfied: click>=8.1.3 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (8.3.1)
Requirement already satisfied: blinker>=1.6.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (1.9.0)
Requirement already satisfied: MarkupSafe>=2.1.1 in ./venv/lib/python3.12/site-packages (from Werkzeug==3.0.1->-r requirements.txt (line 3)) (3.0.3)
Requirement already satisfied: iniconfig in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (2.3.0)
Requirement already satisfied: packaging in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (25.0)
Requirement already satisfied: pluggy<2.0,>=0.12 in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (1.6.0)
Requirement already satisfied: coverage>=5.2.1 in ./venv/lib/python3.12/site-packages (from coverage[toml]>=5.2.1->pytest-cov==4.1.0->-r requirements.txt (line 7)) (7.13.1)
Requirement already satisfied: mccabe<0.8.0,>=0.7.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (0.7.0)
Requirement already satisfied: pycodestyle<2.12.0,>=2.11.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (2.11.1)
Requirement already satisfied: pyflakes<3.2.0,>=3.1.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (3.1.0)
Requirement already satisfied: mypy-extensions>=0.4.3 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (1.1.0)
Requirement already satisfied: pathspec>=0.9.0 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (1.0.3)
Requirement already satisfied: platformdirs>=2 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (4.5.1)
+ echo Restarting Flask service on staging...
Restarting Flask service on staging...
+ ssh -o StrictHostKeyChecking=no ****@**** sudo systemctl restart flask-app
Warning: The unit file, source configuration file or drop-ins of flask-app.service changed on disk. Run 'systemctl daemon-reload' to reload units.
+ echo Deployment to staging completed successfully!
Deployment to staging completed successfully!
[Pipeline] }
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 5891 killed;
[ssh-agent] Stopped.
[Pipeline] // sshagent
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to Production)
Stage "Deploy to Production" skipped due to when conditional
[Pipeline] getContext
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Smoke Test - Staging)
[Pipeline] echo
Running smoke tests on staging...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ echo Waiting for service to start...
Waiting for service to start...
+ sleep 5
+ echo Testing health endpoint...
Testing health endpoint...
+ curl -f http://****/api/health
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    69  100    69    0     0    101      0 --:--:-- --:--:-- --:--:--   101
{"message":"Application is running successfully","status":"healthy"}
+ echo Testing info endpoint...
Testing info endpoint...
+ curl -f http://****/api/info
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   165  100   165    0     0    280      0 --:--:-- --:--:-- --:--:--   280
{"description":"A simple Flask application demonstrating CI/CD with Jenkins and GitHub Actions","environment":"staging","name":"Flask CI/CD Demo","version":"1.0.0"}
+ echo All smoke tests passed!
All smoke tests passed!
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Health Check - Production)
Stage "Health Check - Production" skipped due to when conditional
[Pipeline] getContext
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Cleaning up workspace...
[Pipeline] cleanWs
[WS-CLEANUP] Deleting project workspace...
[WS-CLEANUP] Deferred wipeout is used...
[WS-CLEANUP] done
[Pipeline] echo
Pipeline executed successfully!
[Pipeline] emailext
Sending email to: ghanshyamjobi+jenkins@gmail.com
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS

```

### Stage 3. CI/CD logs for Production tag push

```
Started by user admin
 > git rev-parse --resolve-git-dir /var/lib/jenkins/caches/git-640a1658cad41f253def4865ccabb7b3/.git # timeout=10
Setting origin to https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
Fetching origin...
Fetching upstream changes from origin
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git config --get remote.origin.url # timeout=10
 > git fetch --tags --force --progress -- origin +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/tags/v1.3.0^{commit} # timeout=10
Obtained Jenkinsfile from ef95c8ea7a2af9b1d16bcabe17e78bf7e3c648e8
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
Cloning the remote Git repository
Cloning with configured refspecs honoured and with tags
Cloning repository https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git init /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0 # timeout=10
Fetching upstream changes from https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --tags --force --progress -- https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
Avoid second fetch
Checking out Revision ef95c8ea7a2af9b1d16bcabe17e78bf7e3c648e8 (v1.3.0)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f ef95c8ea7a2af9b1d16bcabe17e78bf7e3c648e8 # timeout=10
Commit message: "Merge pull request #7 from ghanshyamca/staging"
 > git rev-list --no-walk ef95c8ea7a2af9b1d16bcabe17e78bf7e3c648e8 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withCredentials
Masking supported pattern matches of $PRODUCTION_HOST or $PRODUCTION_USER or $STAGING_USER or $STAGING_HOST
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Checkout)
[Pipeline] echo
Checking out source code...
[Pipeline] checkout
Selected Git installation does not exist. Using Default
The recommended git tool is: NONE
No credentials specified
 > git rev-parse --resolve-git-dir /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/.git # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git # timeout=10
Fetching with tags
Fetching upstream changes from https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git
 > git --version # timeout=10
 > git --version # 'git version 2.34.1'
 > git fetch --tags --force --progress -- https://github.com/ghanshyamca/jenkins_pipeline_and_github_action.git +refs/heads/*:refs/remotes/origin/* # timeout=10
Checking out Revision ef95c8ea7a2af9b1d16bcabe17e78bf7e3c648e8 (v1.3.0)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f ef95c8ea7a2af9b1d16bcabe17e78bf7e3c648e8 # timeout=10
Commit message: "Merge pull request #7 from ghanshyamca/staging"
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Setup Environment)
[Pipeline] echo
Setting up Python virtual environment...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ python3 -m venv venv
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install --upgrade pip
Requirement already satisfied: pip in ./venv/lib/python3.10/site-packages (22.0.2)
Collecting pip
  Using cached pip-25.3-py3-none-any.whl (1.8 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 22.0.2
    Uninstalling pip-22.0.2:
      Successfully uninstalled pip-22.0.2
Successfully installed pip-25.3
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build)
[Pipeline] echo
Installing dependencies...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install -r requirements.txt
Collecting Flask==3.0.0 (from -r requirements.txt (line 2))
  Using cached flask-3.0.0-py3-none-any.whl.metadata (3.6 kB)
Collecting Werkzeug==3.0.1 (from -r requirements.txt (line 3))
  Using cached werkzeug-3.0.1-py3-none-any.whl.metadata (4.1 kB)
Collecting pytest==7.4.3 (from -r requirements.txt (line 6))
  Using cached pytest-7.4.3-py3-none-any.whl.metadata (7.9 kB)
Collecting pytest-cov==4.1.0 (from -r requirements.txt (line 7))
  Using cached pytest_cov-4.1.0-py3-none-any.whl.metadata (26 kB)
Collecting gunicorn==21.2.0 (from -r requirements.txt (line 10))
  Using cached gunicorn-21.2.0-py3-none-any.whl.metadata (4.1 kB)
Collecting flake8==6.1.0 (from -r requirements.txt (line 13))
  Using cached flake8-6.1.0-py2.py3-none-any.whl.metadata (3.8 kB)
Collecting black==23.12.1 (from -r requirements.txt (line 14))
  Using cached black-23.12.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (68 kB)
Collecting python-dotenv==1.0.0 (from -r requirements.txt (line 17))
  Using cached python_dotenv-1.0.0-py3-none-any.whl.metadata (21 kB)
Collecting Jinja2>=3.1.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached jinja2-3.1.6-py3-none-any.whl.metadata (2.9 kB)
Collecting itsdangerous>=2.1.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached itsdangerous-2.2.0-py3-none-any.whl.metadata (1.9 kB)
Collecting click>=8.1.3 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached click-8.3.1-py3-none-any.whl.metadata (2.6 kB)
Collecting blinker>=1.6.2 (from Flask==3.0.0->-r requirements.txt (line 2))
  Using cached blinker-1.9.0-py3-none-any.whl.metadata (1.6 kB)
Collecting MarkupSafe>=2.1.1 (from Werkzeug==3.0.1->-r requirements.txt (line 3))
  Using cached markupsafe-3.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.7 kB)
Collecting iniconfig (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached iniconfig-2.3.0-py3-none-any.whl.metadata (2.5 kB)
Collecting packaging (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached packaging-25.0-py3-none-any.whl.metadata (3.3 kB)
Collecting pluggy<2.0,>=0.12 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached pluggy-1.6.0-py3-none-any.whl.metadata (4.8 kB)
Collecting exceptiongroup>=1.0.0rc8 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached exceptiongroup-1.3.1-py3-none-any.whl.metadata (6.7 kB)
Collecting tomli>=1.0.0 (from pytest==7.4.3->-r requirements.txt (line 6))
  Using cached tomli-2.4.0-py3-none-any.whl.metadata (10 kB)
Collecting coverage>=5.2.1 (from coverage[toml]>=5.2.1->pytest-cov==4.1.0->-r requirements.txt (line 7))
  Using cached coverage-7.13.1-cp310-cp310-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl.metadata (8.5 kB)
Collecting mccabe<0.8.0,>=0.7.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached mccabe-0.7.0-py2.py3-none-any.whl.metadata (5.0 kB)
Collecting pycodestyle<2.12.0,>=2.11.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached pycodestyle-2.11.1-py2.py3-none-any.whl.metadata (4.5 kB)
Collecting pyflakes<3.2.0,>=3.1.0 (from flake8==6.1.0->-r requirements.txt (line 13))
  Using cached pyflakes-3.1.0-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting mypy-extensions>=0.4.3 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached mypy_extensions-1.1.0-py3-none-any.whl.metadata (1.1 kB)
Collecting pathspec>=0.9.0 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached pathspec-1.0.3-py3-none-any.whl.metadata (13 kB)
Collecting platformdirs>=2 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached platformdirs-4.5.1-py3-none-any.whl.metadata (12 kB)
Collecting typing-extensions>=4.0.1 (from black==23.12.1->-r requirements.txt (line 14))
  Using cached typing_extensions-4.15.0-py3-none-any.whl.metadata (3.3 kB)
Using cached flask-3.0.0-py3-none-any.whl (99 kB)
Using cached werkzeug-3.0.1-py3-none-any.whl (226 kB)
Using cached pytest-7.4.3-py3-none-any.whl (325 kB)
Using cached pytest_cov-4.1.0-py3-none-any.whl (21 kB)
Using cached gunicorn-21.2.0-py3-none-any.whl (80 kB)
Using cached flake8-6.1.0-py2.py3-none-any.whl (58 kB)
Using cached black-23.12.1-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (1.7 MB)
Using cached python_dotenv-1.0.0-py3-none-any.whl (19 kB)
Using cached mccabe-0.7.0-py2.py3-none-any.whl (7.3 kB)
Using cached pluggy-1.6.0-py3-none-any.whl (20 kB)
Using cached pycodestyle-2.11.1-py2.py3-none-any.whl (31 kB)
Using cached pyflakes-3.1.0-py2.py3-none-any.whl (62 kB)
Using cached blinker-1.9.0-py3-none-any.whl (8.5 kB)
Using cached click-8.3.1-py3-none-any.whl (108 kB)
Using cached coverage-7.13.1-cp310-cp310-manylinux1_x86_64.manylinux_2_28_x86_64.manylinux_2_5_x86_64.whl (247 kB)
Using cached exceptiongroup-1.3.1-py3-none-any.whl (16 kB)
Using cached itsdangerous-2.2.0-py3-none-any.whl (16 kB)
Using cached jinja2-3.1.6-py3-none-any.whl (134 kB)
Using cached markupsafe-3.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (20 kB)
Using cached mypy_extensions-1.1.0-py3-none-any.whl (5.0 kB)
Using cached packaging-25.0-py3-none-any.whl (66 kB)
Using cached pathspec-1.0.3-py3-none-any.whl (55 kB)
Using cached platformdirs-4.5.1-py3-none-any.whl (18 kB)
Using cached tomli-2.4.0-py3-none-any.whl (14 kB)
Using cached typing_extensions-4.15.0-py3-none-any.whl (44 kB)
Using cached iniconfig-2.3.0-py3-none-any.whl (7.5 kB)
Installing collected packages: typing-extensions, tomli, python-dotenv, pyflakes, pycodestyle, pluggy, platformdirs, pathspec, packaging, mypy-extensions, mccabe, MarkupSafe, itsdangerous, iniconfig, coverage, click, blinker, Werkzeug, Jinja2, gunicorn, flake8, exceptiongroup, black, pytest, Flask, pytest-cov

Successfully installed Flask-3.0.0 Jinja2-3.1.6 MarkupSafe-3.0.3 Werkzeug-3.0.1 black-23.12.1 blinker-1.9.0 click-8.3.1 coverage-7.13.1 exceptiongroup-1.3.1 flake8-6.1.0 gunicorn-21.2.0 iniconfig-2.3.0 itsdangerous-2.2.0 mccabe-0.7.0 mypy-extensions-1.1.0 packaging-25.0 pathspec-1.0.3 platformdirs-4.5.1 pluggy-1.6.0 pycodestyle-2.11.1 pyflakes-3.1.0 pytest-7.4.3 pytest-cov-4.1.0 python-dotenv-1.0.0 tomli-2.4.0 typing-extensions-4.15.0
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Code Quality Check)
[Pipeline] echo
Running code quality checks...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ flake8 app.py --max-line-length=120 --exclude=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Test)
[Pipeline] echo
Running unit tests...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pytest test_app.py -v --junitxml=test-results.xml --cov=app --cov-report=xml --cov-report=html
============================= test session starts ==============================
platform linux -- Python 3.10.12, pytest-7.4.3, pluggy-1.6.0 -- /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv/bin/python3
cachedir: .pytest_cache
rootdir: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0
plugins: cov-4.1.0
collecting ... collected 11 items

test_app.py::test_home_page PASSED                                       [  9%]
test_app.py::test_health_check PASSED                                    [ 18%]
test_app.py::test_info_endpoint PASSED                                   [ 27%]
test_app.py::test_add_numbers_positive PASSED                            [ 36%]
test_app.py::test_add_numbers_negative PASSED                            [ 45%]
test_app.py::test_add_numbers_zero PASSED                                [ 54%]
test_app.py::test_add_numbers_large PASSED                               [ 63%]
test_app.py::test_invalid_endpoint PASSED                                [ 72%]
test_app.py::test_add_non_integer PASSED                                 [ 81%]
test_app.py::test_health_check_returns_json PASSED                       [ 90%]
test_app.py::test_info_returns_json PASSED                               [100%]

- generated xml file: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/test-results.xml -

---------- coverage: platform linux, python 3.10.12-final-0 ----------
Coverage HTML written to dir htmlcov
Coverage XML written to file coverage.xml

============================== 11 passed in 1.14s ==============================
[Pipeline] }
[Pipeline] // script
Post stage
[Pipeline] junit
Recording test results
[Checks API] No suitable checks publisher found.
[Pipeline] publishHTML
[htmlpublisher] Archiving HTML reports...
[htmlpublisher] Archiving at BUILD level /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/htmlcov to Coverage_20Report
[htmlpublisher] Copying recursive using current thread
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Security Scan)
[Pipeline] echo
Running security vulnerability scan...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ . venv/bin/activate
+ deactivate nondestructive
+ [ -n  ]
+ [ -n  ]
+ [ -n  -o -n  ]
+ [ -n  ]
+ unset VIRTUAL_ENV
+ unset VIRTUAL_ENV_PROMPT
+ [ ! nondestructive = nondestructive ]
+ VIRTUAL_ENV=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv
+ export VIRTUAL_ENV
+ _OLD_VIRTUAL_PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ PATH=/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0/venv/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin
+ export PATH
+ [ -n  ]
+ [ -z  ]
+ _OLD_VIRTUAL_PS1=$ 
+ PS1=(venv) $ 
+ export PS1
+ VIRTUAL_ENV_PROMPT=(venv) 
+ export VIRTUAL_ENV_PROMPT
+ [ -n  -o -n  ]
+ pip install safety bandit
Collecting safety
  Using cached safety-3.7.0-py3-none-any.whl.metadata (11 kB)
Collecting bandit
  Using cached bandit-1.9.3-py3-none-any.whl.metadata (7.1 kB)
Collecting authlib>=1.2.0 (from safety)
  Using cached authlib-1.6.6-py2.py3-none-any.whl.metadata (9.8 kB)
Requirement already satisfied: click>=8.0.2 in ./venv/lib/python3.10/site-packages (from safety) (8.3.1)
Collecting dparse>=0.6.4 (from safety)
  Using cached dparse-0.6.4-py3-none-any.whl.metadata (5.5 kB)
Collecting filelock<4.0,>=3.16.1 (from safety)
  Using cached filelock-3.20.3-py3-none-any.whl.metadata (2.1 kB)
Collecting httpx (from safety)
  Using cached httpx-0.28.1-py3-none-any.whl.metadata (7.1 kB)
Requirement already satisfied: jinja2>=3.1.0 in ./venv/lib/python3.10/site-packages (from safety) (3.1.6)
Collecting marshmallow>=3.15.0 (from safety)
  Using cached marshmallow-4.2.0-py3-none-any.whl.metadata (7.4 kB)
Collecting nltk>=3.9 (from safety)
  Using cached nltk-3.9.2-py3-none-any.whl.metadata (3.2 kB)
Requirement already satisfied: packaging>=21.0 in ./venv/lib/python3.10/site-packages (from safety) (25.0)
Collecting pydantic>=2.6.0 (from safety)
  Using cached pydantic-2.12.5-py3-none-any.whl.metadata (90 kB)
Collecting requests (from safety)
  Using cached requests-2.32.5-py3-none-any.whl.metadata (4.9 kB)
Collecting ruamel-yaml>=0.17.21 (from safety)
  Using cached ruamel_yaml-0.19.1-py3-none-any.whl.metadata (16 kB)
Collecting safety-schemas==0.0.16 (from safety)
  Using cached safety_schemas-0.0.16-py3-none-any.whl.metadata (1.1 kB)
Collecting tenacity>=8.1.0 (from safety)
  Using cached tenacity-9.1.2-py3-none-any.whl.metadata (1.2 kB)
Requirement already satisfied: tomli in ./venv/lib/python3.10/site-packages (from safety) (2.4.0)
Collecting tomlkit (from safety)
  Using cached tomlkit-0.14.0-py3-none-any.whl.metadata (2.8 kB)
Collecting typer>=0.16.0 (from safety)
  Using cached typer-0.21.1-py3-none-any.whl.metadata (16 kB)
Requirement already satisfied: typing-extensions>=4.7.1 in ./venv/lib/python3.10/site-packages (from safety) (4.15.0)
Collecting PyYAML>=5.3.1 (from bandit)
  Using cached pyyaml-6.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (2.4 kB)
Collecting stevedore>=1.20.0 (from bandit)
  Using cached stevedore-5.6.0-py3-none-any.whl.metadata (2.3 kB)
Collecting rich (from bandit)
  Using cached rich-14.2.0-py3-none-any.whl.metadata (18 kB)
Collecting cryptography (from authlib>=1.2.0->safety)
  Using cached cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl.metadata (5.7 kB)
Requirement already satisfied: MarkupSafe>=2.0 in ./venv/lib/python3.10/site-packages (from jinja2>=3.1.0->safety) (3.0.3)
Collecting backports-datetime-fromisoformat (from marshmallow>=3.15.0->safety)
  Using cached backports_datetime_fromisoformat-2.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (6.3 kB)
Collecting joblib (from nltk>=3.9->safety)
  Using cached joblib-1.5.3-py3-none-any.whl.metadata (5.5 kB)
Collecting regex>=2021.8.3 (from nltk>=3.9->safety)
  Using cached regex-2026.1.15-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (40 kB)
Collecting tqdm (from nltk>=3.9->safety)
  Using cached tqdm-4.67.1-py3-none-any.whl.metadata (57 kB)
Collecting annotated-types>=0.6.0 (from pydantic>=2.6.0->safety)
  Using cached annotated_types-0.7.0-py3-none-any.whl.metadata (15 kB)
Collecting pydantic-core==2.41.5 (from pydantic>=2.6.0->safety)
  Using cached pydantic_core-2.41.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl.metadata (7.3 kB)
Collecting typing-inspection>=0.4.2 (from pydantic>=2.6.0->safety)
  Using cached typing_inspection-0.4.2-py3-none-any.whl.metadata (2.6 kB)
Collecting shellingham>=1.3.0 (from typer>=0.16.0->safety)
  Using cached shellingham-1.5.4-py2.py3-none-any.whl.metadata (3.5 kB)
Collecting markdown-it-py>=2.2.0 (from rich->bandit)
  Using cached markdown_it_py-4.0.0-py3-none-any.whl.metadata (7.3 kB)
Collecting pygments<3.0.0,>=2.13.0 (from rich->bandit)
  Using cached pygments-2.19.2-py3-none-any.whl.metadata (2.5 kB)
Collecting mdurl~=0.1 (from markdown-it-py>=2.2.0->rich->bandit)
  Using cached mdurl-0.1.2-py3-none-any.whl.metadata (1.6 kB)
Collecting cffi>=2.0.0 (from cryptography->authlib>=1.2.0->safety)
  Using cached cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl.metadata (2.6 kB)
Collecting pycparser (from cffi>=2.0.0->cryptography->authlib>=1.2.0->safety)
  Using cached pycparser-2.23-py3-none-any.whl.metadata (993 bytes)
Collecting anyio (from httpx->safety)
  Using cached anyio-4.12.1-py3-none-any.whl.metadata (4.3 kB)
Collecting certifi (from httpx->safety)
  Using cached certifi-2026.1.4-py3-none-any.whl.metadata (2.5 kB)
Collecting httpcore==1.* (from httpx->safety)
  Using cached httpcore-1.0.9-py3-none-any.whl.metadata (21 kB)
Collecting idna (from httpx->safety)
  Using cached idna-3.11-py3-none-any.whl.metadata (8.4 kB)
Collecting h11>=0.16 (from httpcore==1.*->httpx->safety)
  Using cached h11-0.16.0-py3-none-any.whl.metadata (8.3 kB)
Requirement already satisfied: exceptiongroup>=1.0.2 in ./venv/lib/python3.10/site-packages (from anyio->httpx->safety) (1.3.1)
Collecting charset_normalizer<4,>=2 (from requests->safety)
  Using cached charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl.metadata (37 kB)
Collecting urllib3<3,>=1.21.1 (from requests->safety)
  Using cached urllib3-2.6.3-py3-none-any.whl.metadata (6.9 kB)
Using cached safety-3.7.0-py3-none-any.whl (312 kB)
Using cached safety_schemas-0.0.16-py3-none-any.whl (39 kB)
Using cached filelock-3.20.3-py3-none-any.whl (16 kB)
Using cached bandit-1.9.3-py3-none-any.whl (134 kB)
Using cached authlib-1.6.6-py2.py3-none-any.whl (244 kB)
Using cached dparse-0.6.4-py3-none-any.whl (11 kB)
Using cached marshmallow-4.2.0-py3-none-any.whl (48 kB)
Using cached nltk-3.9.2-py3-none-any.whl (1.5 MB)
Using cached pydantic-2.12.5-py3-none-any.whl (463 kB)
Using cached pydantic_core-2.41.5-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (2.1 MB)
Using cached annotated_types-0.7.0-py3-none-any.whl (13 kB)
Using cached pyyaml-6.0.3-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (770 kB)
Using cached regex-2026.1.15-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (791 kB)
Using cached ruamel_yaml-0.19.1-py3-none-any.whl (118 kB)
Using cached stevedore-5.6.0-py3-none-any.whl (54 kB)
Using cached tenacity-9.1.2-py3-none-any.whl (28 kB)
Using cached typer-0.21.1-py3-none-any.whl (47 kB)
Using cached rich-14.2.0-py3-none-any.whl (243 kB)
Using cached pygments-2.19.2-py3-none-any.whl (1.2 MB)
Using cached markdown_it_py-4.0.0-py3-none-any.whl (87 kB)
Using cached mdurl-0.1.2-py3-none-any.whl (10.0 kB)
Using cached shellingham-1.5.4-py2.py3-none-any.whl (9.8 kB)
Using cached typing_inspection-0.4.2-py3-none-any.whl (14 kB)
Using cached backports_datetime_fromisoformat-2.0.3-cp310-cp310-manylinux_2_17_x86_64.manylinux2014_x86_64.whl (52 kB)
Using cached cryptography-46.0.3-cp38-abi3-manylinux_2_34_x86_64.whl (4.5 MB)
Using cached cffi-2.0.0-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.whl (216 kB)
Using cached httpx-0.28.1-py3-none-any.whl (73 kB)
Using cached httpcore-1.0.9-py3-none-any.whl (78 kB)
Using cached h11-0.16.0-py3-none-any.whl (37 kB)
Using cached anyio-4.12.1-py3-none-any.whl (113 kB)
Using cached idna-3.11-py3-none-any.whl (71 kB)
Using cached certifi-2026.1.4-py3-none-any.whl (152 kB)
Using cached joblib-1.5.3-py3-none-any.whl (309 kB)
Using cached pycparser-2.23-py3-none-any.whl (118 kB)
Using cached requests-2.32.5-py3-none-any.whl (64 kB)
Using cached charset_normalizer-3.4.4-cp310-cp310-manylinux2014_x86_64.manylinux_2_17_x86_64.manylinux_2_28_x86_64.whl (153 kB)
Using cached urllib3-2.6.3-py3-none-any.whl (131 kB)
Using cached tomlkit-0.14.0-py3-none-any.whl (39 kB)
Using cached tqdm-4.67.1-py3-none-any.whl (78 kB)
Installing collected packages: urllib3, typing-inspection, tqdm, tomlkit, tenacity, stevedore, shellingham, ruamel-yaml, regex, PyYAML, pygments, pydantic-core, pycparser, mdurl, joblib, idna, h11, filelock, dparse, charset_normalizer, certifi, backports-datetime-fromisoformat, annotated-types, requests, pydantic, nltk, marshmallow, markdown-it-py, httpcore, cffi, anyio, safety-schemas, rich, httpx, cryptography, typer, bandit, authlib, safety

Successfully installed PyYAML-6.0.3 annotated-types-0.7.0 anyio-4.12.1 authlib-1.6.6 backports-datetime-fromisoformat-2.0.3 bandit-1.9.3 certifi-2026.1.4 cffi-2.0.0 charset_normalizer-3.4.4 cryptography-46.0.3 dparse-0.6.4 filelock-3.20.3 h11-0.16.0 httpcore-1.0.9 httpx-0.28.1 idna-3.11 joblib-1.5.3 markdown-it-py-4.0.0 marshmallow-4.2.0 mdurl-0.1.2 nltk-3.9.2 pycparser-2.23 pydantic-2.12.5 pydantic-core-2.41.5 pygments-2.19.2 regex-2026.1.15 requests-2.32.5 rich-14.2.0 ruamel-yaml-0.19.1 safety-3.7.0 safety-schemas-0.0.16 shellingham-1.5.4 stevedore-5.6.0 tenacity-9.1.2 tomlkit-0.14.0 tqdm-4.67.1 typer-0.21.1 typing-inspection-0.4.2 urllib3-2.6.3
+ echo Running Safety check...
Running Safety check...
+ safety check --json
+ true
+ echo Running Bandit security scan...
Running Bandit security scan...
+ bandit -r app.py -f json -o bandit-report.json
[main]	INFO	profile include tests: None
[main]	INFO	profile exclude tests: None
[main]	INFO	cli include tests: None
[main]	INFO	cli exclude tests: None
[json]	INFO	JSON output written to file: bandit-report.json
+ true
+ echo Security scans completed!
Security scans completed!
[Pipeline] }
[Pipeline] // script
Post stage
[Pipeline] archiveArtifacts
Archiving artifacts
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Build Artifact)
[Pipeline] echo
Creating deployment artifact...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ tar -czf flask-app-2.tar.gz --exclude=venv --exclude=*.pyc --exclude=__pycache__ app.py requirements.txt
[Pipeline] }
[Pipeline] // script
[Pipeline] archiveArtifacts
Archiving artifacts
Recording fingerprints
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to Staging)
Stage "Deploy to Staging" skipped due to when conditional
[Pipeline] getContext
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Deploy to Production)
[Pipeline] echo
Deploying to production environment...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sshagent
[ssh-agent] Using credentials ****
$ ssh-agent
SSH_AUTH_SOCK=/tmp/ssh-XXXXXXJVXSKa/agent.10942
SSH_AGENT_PID=10945
Running ssh-add (command line suppressed)
Identity added: /var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0@tmp/private_key_5946313140748503531.key (/var/lib/jenkins/workspace/Flask-CI-CD-Multibranch_v1.3.0@tmp/private_key_5946313140748503531.key)
[ssh-agent] Started.
[Pipeline] {
[Pipeline] sh
+ echo Deploying to production server: ****
Deploying to production server: ****
+ echo Copying files to production server...
Copying files to production server...
+ scp -o StrictHostKeyChecking=no flask-app-2.tar.gz ****@****:/var/www/flask-app
Warning: Permanently added **** (ED25519) to the list of known hosts.
+ echo Extracting and installing on production server...
Extracting and installing on production server...
+ ssh -o StrictHostKeyChecking=no ****@**** cd /var/www/flask-app && tar -xzf flask-app-2.tar.gz
+ ssh -o StrictHostKeyChecking=no ****@**** cd /var/www/flask-app && python3 -m venv venv && venv/bin/pip install -r requirements.txt
Requirement already satisfied: Flask==3.0.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 2)) (3.0.0)
Requirement already satisfied: Werkzeug==3.0.1 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 3)) (3.0.1)
Requirement already satisfied: pytest==7.4.3 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 6)) (7.4.3)
Requirement already satisfied: pytest-cov==4.1.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 7)) (4.1.0)
Requirement already satisfied: gunicorn==21.2.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 10)) (21.2.0)
Requirement already satisfied: flake8==6.1.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 13)) (6.1.0)
Requirement already satisfied: black==23.12.1 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 14)) (23.12.1)
Requirement already satisfied: python-dotenv==1.0.0 in ./venv/lib/python3.12/site-packages (from -r requirements.txt (line 17)) (1.0.0)
Requirement already satisfied: Jinja2>=3.1.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (3.1.6)
Requirement already satisfied: itsdangerous>=2.1.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (2.2.0)
Requirement already satisfied: click>=8.1.3 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (8.3.1)
Requirement already satisfied: blinker>=1.6.2 in ./venv/lib/python3.12/site-packages (from Flask==3.0.0->-r requirements.txt (line 2)) (1.9.0)
Requirement already satisfied: MarkupSafe>=2.1.1 in ./venv/lib/python3.12/site-packages (from Werkzeug==3.0.1->-r requirements.txt (line 3)) (3.0.3)
Requirement already satisfied: iniconfig in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (2.3.0)
Requirement already satisfied: packaging in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (25.0)
Requirement already satisfied: pluggy<2.0,>=0.12 in ./venv/lib/python3.12/site-packages (from pytest==7.4.3->-r requirements.txt (line 6)) (1.6.0)
Requirement already satisfied: coverage>=5.2.1 in ./venv/lib/python3.12/site-packages (from coverage[toml]>=5.2.1->pytest-cov==4.1.0->-r requirements.txt (line 7)) (7.13.1)
Requirement already satisfied: mccabe<0.8.0,>=0.7.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (0.7.0)
Requirement already satisfied: pycodestyle<2.12.0,>=2.11.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (2.11.1)
Requirement already satisfied: pyflakes<3.2.0,>=3.1.0 in ./venv/lib/python3.12/site-packages (from flake8==6.1.0->-r requirements.txt (line 13)) (3.1.0)
Requirement already satisfied: mypy-extensions>=0.4.3 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (1.1.0)
Requirement already satisfied: pathspec>=0.9.0 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (1.0.3)
Requirement already satisfied: platformdirs>=2 in ./venv/lib/python3.12/site-packages (from black==23.12.1->-r requirements.txt (line 14)) (4.5.1)
+ echo Restarting Flask service on production...
Restarting Flask service on production...
+ ssh -o StrictHostKeyChecking=no ****@**** sudo systemctl restart flask-app
+ echo Deployment to production completed successfully!
Deployment to production completed successfully!
[Pipeline] }
$ ssh-agent -k
unset SSH_AUTH_SOCK;
unset SSH_AGENT_PID;
echo Agent pid 10945 killed;
[ssh-agent] Stopped.
[Pipeline] // sshagent
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Smoke Test - Staging)
Stage "Smoke Test - Staging" skipped due to when conditional
[Pipeline] getContext
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Health Check - Production)
[Pipeline] echo
Running health checks on production...
[Pipeline] script
[Pipeline] {
[Pipeline] isUnix
[Pipeline] sh
+ echo Waiting for service to start...
Waiting for service to start...
+ sleep 5
+ echo Testing health endpoint...
Testing health endpoint...
+ curl -f http://****/api/health
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100    69  100    69    0     0     43      0  0:00:01  0:00:01 --:--:--    44
{"message":"Application is running successfully","status":"healthy"}
+ echo Testing info endpoint...
Testing info endpoint...
+ curl -f http://****/api/info
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed

  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   168  100   168    0     0    269      0 --:--:-- --:--:-- --:--:--   269
{"description":"A simple Flask application demonstrating CI/CD with Jenkins and GitHub Actions","environment":"production","name":"Flask CI/CD Demo","version":"1.0.0"}
+ echo All health checks passed!
All health checks passed!
[Pipeline] }
[Pipeline] // script
[Pipeline] }
[Pipeline] // stage
[Pipeline] stage
[Pipeline] { (Declarative: Post Actions)
[Pipeline] echo
Cleaning up workspace...
[Pipeline] cleanWs
[WS-CLEANUP] Deleting project workspace...
[WS-CLEANUP] Deferred wipeout is used...
[WS-CLEANUP] done
[Pipeline] echo
Pipeline executed successfully!
[Pipeline] emailext
Sending email to: ghanshyamjobi+jenkins@gmail.com
[Pipeline] }
[Pipeline] // stage
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withCredentials
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline
Finished: SUCCESS

```

### Stage 4. Email received for Staging

<img width="1203" height="442" alt="image" src="https://github.com/user-attachments/assets/64a8173d-429a-4c0c-9bf6-4432135c0f7e" />


### Stage 5. Email received for Production tag version

<img width="1049" height="446" alt="image" src="https://github.com/user-attachments/assets/a02e4ea3-4752-4626-93e4-1bcb90801718" />

## EC2 instances of Staging
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/53a6804b-cb42-4652-997d-e86bf57aefb0" />

## EC2 instances of production

<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/0fccc4ca-25ac-4406-be36-456fb5271e89" />
