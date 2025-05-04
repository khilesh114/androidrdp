# androidrdp
# 🚀 Android RDP via GitHub Actions + Docker + ngrok

Remotely access Android via GitHub Actions using [budtmo/docker-android](https://github.com/budtmo/docker-android) and ngrok tunnel.

---

## 📦 Step-by-step Guide

### 1️⃣ Create GitHub Repository

Create a new repository with name: **android-rdp**

---

### 2️⃣ Get ngrok Account & Auth Token

1. Go to [https://ngrok.com](https://ngrok.com) and sign up / login.
2. Visit the **Auth Token** section in your ngrok dashboard.
3. Copy your **Auth Token**.

---

### 3️⃣ Add ngrok Token to GitHub Secrets

1. Go to your GitHub repository: **Settings > Secrets and variables > Actions > New repository secret**
2. Add a new secret:
   - Name: `NGROK_TOKEN`
   - Value: *(paste your ngrok auth token)*

---

### 4️⃣ Add Workflow File

Create a file at: `.github/workflows/android-rdp.yml`

```yaml
name: Android Emulator with Docker and ngrok

on:
  push:
    branches:
      - main
  pull_request:
    branches:
      - main

jobs:
  build:
    runs-on: ubuntu-latest

    services:
      docker:
        image: docker:19.03.12
        options: --privileged
        ports:
          - 6080:6080

    steps:
      - name: Checkout repository
        uses: actions/checkout@v2

      - name: Set up Docker
        uses: docker/setup-buildx-action@v2

      - name: Run Android Emulator in Docker
        run: |
          docker run -d \
            -p 6080:6080 \
            -e EMULATOR_DEVICE="Samsung Galaxy S10" \
            -e WEB_VNC=true \
            --device /dev/kvm \
            --name android-container \
            budtmo/docker-android:emulator_11.0

      - name: Install ngrok v3
        run: |
          curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
          echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
          sudo apt update && sudo apt install ngrok

      - name: Authenticate ngrok with your token
        run: |
          ngrok config add-authtoken ${{ secrets.NGROK_AUTH_TOKEN }}

      - name: Start ngrok to forward port 6080
        run: |
          nohup ngrok http 6080 > ngrok.log &

      - name: Wait for Emulator to start
        run: |
          sleep 60

      - name: Get ngrok public URL
        run: |
          sleep 10
          curl -s http://localhost:4040/api/tunnels | jq -r .tunnels[0].public_url
          sleep 3600

      - name: Run tests (optional)
        run: |
          echo "Running Android tests..."
