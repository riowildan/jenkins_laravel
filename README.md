# Laravel CI/CD Pipeline with Jenkins and GitHub Actions

## Overview

Project ini merupakan demonstrasi implementasi **CI/CD (Continuous Integration & Continuous Deployment)** menggunakan:

- **Jenkins Automation Server**
- **GitHub Actions**
- **Laravel Framework**
- **WSL (Ubuntu Environment)**

Pipeline ini digunakan untuk melakukan proses:

- Automated Build
- Automated Testing
- Continuous Integration
- Continuous Deployment Preparation

---

# CI/CD Architecture
<img width="1400" height="427" alt="1_Osk9sX6ZWkwS2uwdH7R9vQ" src="https://github.com/user-attachments/assets/9347fc63-07ad-4659-859b-eb6524cee429" />

```
Developer
   ↓
GitHub Repository
   ↓
GitHub Actions (CI)
   ↓
Jenkins Pipeline
   ↓
Build + Testing Laravel Application
```

---

# Concept Notes

1. **Jenkins Automation Server** digunakan untuk menjalankan proses **CI/CD** seperti build dan testing aplikasi secara otomatis.

2. **Git Pipeline** sebenarnya juga dapat melakukan CI/CD, namun untuk implementasi pipeline yang lebih kompleks biasanya memerlukan framework tambahan yang lebih advanced.

3. Repository seperti **GitHub atau GitLab** dapat digunakan sebagai **cloud storage untuk source code**, sedangkan proses automation dijalankan oleh Jenkins.

4. Resource besar atau proses build yang berat dapat ditempatkan di **Jenkins Server** agar repository hosting tetap ringan.

<img width="6840" height="3040" alt="jenkins-workflow-hd" src="https://github.com/user-attachments/assets/b1be9b88-530d-41e3-b6f7-80dcc1bf141e" />

<p align="center">
  <img width="899" height="453" alt="jenkins-workflow" src="https://github.com/user-attachments/assets/5f171bd3-96d1-477c-8086-ae637a4a4518" />
</p>


---

# Part 1 — Jenkins Installation (WSL)


## Masuk ke WSL

```bash
wsl
sudo su
cd ~
cd /home/aksal
```

---

## Install Java JDK

Jenkins membutuhkan Java agar dapat berjalan.

```bash
sudo apt update
sudo apt install openjdk-17-jdk -y
```

Cek instalasi Java:

```bash
java -version
```

---

## Tambahkan Jenkins Repository Key

```bash
sudo wget -O /etc/apt/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian-stable/jenkins.io-2026.key
```

---

## Tambahkan Jenkins Repository

```bash
echo "deb [signed-by=/etc/apt/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
```

---

## Install Jenkins

```bash
sudo apt install jenkins -y
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

---

## Cek apakah Jenkins berjalan

```bash
sudo lsof -i :8080
```

Jika Jenkins berjalan maka port **8080** akan aktif.

---

## Buka Jenkins Dashboard

```
http://localhost:8080
```

Ambil password awal Jenkins:

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Copy password tersebut ke halaman login Jenkins.

---

## Install Jenkins Plugins

Pilih:

<img width="1920" height="972" alt="Screenshot (299)" src="https://github.com/user-attachments/assets/fbce51ba-b977-4803-b991-4adad55cd555" />

```
Install Suggested Plugins

```

Setelah selesai, Jenkins akan masuk ke **Dashboard**.

---

## Test Jenkins Pipeline

Buat job pipeline sederhana:

```
New Item
→ Pipeline
```

Masukkan contoh script:

<img width="1920" height="971" alt="Screenshot (308)" src="https://github.com/user-attachments/assets/cbcae1b1-8640-48cf-bd1b-cebab931da76" />

```groovy
pipeline {
    agent any

    stages {
        stage('Hello') {
            steps {
                echo 'Hello World'
            }
        }
    }
}
```

Klik:

```
Build Now
```

Lihat hasilnya di:

```
Console Output
```

Jika berhasil akan muncul:

```
Finished: SUCCESS
```

---

# Part 2 — Laravel Project Setup

## Buat folder project

```bash
cd C:\Users\USERNAME\Documents
mkdir website
cd website
mkdir laravel_jenkins
```

---

## Install Laravel

```bash
composer create-project laravel/laravel laravel-ci
```

Masuk ke project:

```bash
cd laravel-ci
```

---

## Setup Environment

```bash
copy .env.example .env
php artisan key:generate
```

---

## Jalankan Laravel

```bash
php artisan serve
```

Buka di browser:

```
http://127.0.0.1:8000
```

---

# Part 3 — CI/CD using GitHub Actions

Push project ke GitHub terlebih dahulu.

Buat folder workflow:

```
.github/workflows
```

Buat file:

```
laravel.yml
```

---

## GitHub Actions Workflow

```yaml
name: Laravel CI

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  laravel-tests:

    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v4

      - name: Setup PHP
        uses: shivammathur/setup-php@v2
        with:
          php-version: 8.2

      - name: Install dependencies
        run: composer install --no-interaction --prefer-dist

      - name: Copy ENV
        run: cp .env.example .env

      - name: Generate key
        run: php artisan key:generate

      - name: Run tests
        run: php artisan test
```

Atau bisa dibuat melalui menu:

```
GitHub → Actions → Setup Workflow
```

---

# Part 4 — CI/CD using Jenkins

## Install Required Jenkins Plugins

Masuk ke:

<img width="1920" height="988" alt="Screenshot (315)" src="https://github.com/user-attachments/assets/974c3232-0782-42fa-a0c8-6892e2b5d511" />


```
Manage Jenkins → Plugins
```

Install plugin berikut:

- Git plugin
- GitHub Integration
- Credentials Binding
- Pipeline
- Pipeline: Stage View

---

```
Install Tools

```
- php
- composer
- 
---

```bash
sudo apt install php-cli php-json php-zip wget unzip curl -y
sudo php composer-setup.php --install-dir=/usr/local/bin --filename=composer
```

---

Tes run di pipeline:

```bash
pipeline {
    agent any

    stages {

        stage('Check Tools') {
            steps {
                sh 'git --version'
                sh 'php -v'
                sh 'composer -V'
            }
        }

    }
}
```

---

## Install PHP & Composer on Jenkins (WSL)

```bash
sudo apt install php php-cli php-mbstring php-xml php-curl unzip -y
sudo apt install composer -y
```

---

# Jenkins Pipeline Script

Buat file di root project:

```
Jenkinsfile
```

Isi pipeline:

```groovy
pipeline {
    agent any

    stages {

        stage('Clone Repository') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'composer install --no-interaction'
            }
        }

        stage('Setup Laravel') {
            steps {
                sh 'cp .env.example .env'
                sh 'php artisan key:generate'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'php artisan test'
            }
        }

    }
}
```

Push file ini ke GitHub.

---

# Jenkins Pipeline Configuration

Masuk Jenkins:

```
New Item → Pipeline
```

Nama:

```
laravel-ci
```

---

## Pipeline Configuration

```
Definition → Pipeline script from SCM
```

SCM:

```
Git
```

Repository:

```
https://github.com/Aqshalikhsan/jenkins_laravel.git
```

Branch:

```
*/main
```

---

# GitHub Credential Setup

Jika Jenkins belum terhubung ke GitHub:

Masuk ke:

```
Manage Jenkins → Credentials
```

Tambah credential:

<img width="1920" height="980" alt="Screenshot (324)" src="https://github.com/user-attachments/assets/4dbc15b2-43c9-4995-be22-56101b9125e3" />

```
Kind: Username with password
Username: GitHub username
Password: GitHub Personal Access Token
```

---

## Generate GitHub Token

```
GitHub → Settings
→ Developer Settings
→ Personal Access Token
→ Generate Token (Classic)
```

Centang permission:

```
repo
workflow
```

Copy token dan masukkan ke Jenkins.

<img width="1920" height="974" alt="Screenshot (325)" src="https://github.com/user-attachments/assets/d63fc974-5e68-46d8-aba7-f15d1f152faa" />

---

# Run Jenkins Pipeline

Klik:

```
Build Now
```

Pipeline akan menjalankan:

```
Clone Repository
↓
Composer Install
↓
Laravel Setup
↓
Run Tests
```

Jika berhasil:

```
Finished: SUCCESS
```

---

# Final CI/CD Workflow

```
Developer Push
↓
GitHub Repository
↓
GitHub Actions CI
↓
Jenkins Pipeline
↓
Build + Test Laravel Application
```

<img width="1920" height="780" alt="Screenshot (327)" src="https://github.com/user-attachments/assets/1f4cb15c-33b4-4b28-9ea2-229ebd570983" />

---

# Author

**Aqshal Ikhsan**

DevOps Learning Project  
Laravel CI/CD with Jenkins and GitHub Actions
