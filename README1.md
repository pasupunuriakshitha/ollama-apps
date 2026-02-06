# Ollama Chat Application - Deployment Guide

A full-stack chat application using Ollama AI models with FastAPI backend and React frontend, deployed on AWS.


## Create instance
Create New EC2 Instance - Complete Settings
Step 1: Launch Instance

Go to EC2 Dashboard
Click "Launch Instance"


Step 2: Configure Everything
Name:
ollama-chat-server
Application and OS Images:

Quick Start: Ubuntu
AMI: Ubuntu Server 22.04 LTS (HVM), SSD Volume Type
Architecture: 64-bit (x86)

Instance type:

t3.medium (2 vCPU, 4 GB Memory)

Key pair:

Select your existing key OR create new one
If creating new: Name it ollama-key, Type: RSA, Format: .pem
Download and save the .pem file

Network settings:

Click "Edit"
VPC: Keep default
Subnet: Keep default (No preference)
Auto-assign public IP: Enable
Firewall (security groups): Create security group
Security group name: ollama-sg-new
Description: Security group for Ollama Chat

Security group rules - Add these 4 rules:
Rule 1:

Type: SSH
Port: 22
Source type: My IP

Rule 2:

Type: HTTP
Port: 80
Source type: Anywhere (0.0.0.0/0)

Rule 3:

Type: HTTPS
Port: 443
Source type: Anywhere (0.0.0.0/0)

Rule 4: ⚠️ IMPORTANT - This is the missing one!

Type: Custom TCP
Port range: 8000
Source type: Anywhere (0.0.0.0/0)
Description: Ollama API

Configure storage:

Size: 30 GiB
Volume type: gp3

Advanced details:

Leave everything as default


Step 3: Launch
Click "Launch instance"
Wait for it to say "Successfully initiated launch"
Click "View all instances"
Get the Public IP and save it!

## 🏗️ Architecture
```
User Browser
     ↓
S3 (Frontend - React)
     ↓
EC2 (Backend - FastAPI + Ollama)
     ↓
Ollama Models (qwen2.5:0.5b)
```

## 📋 Prerequisites

- AWS Account
- GitHub Account
- Local: Node.js, npm, Git
- EC2 Instance (t3.medium, Ubuntu 22.04)

## 🚀 Backend Deployment (EC2)

### Step 1: Connect to EC2
```bash
ssh -i your-key.pem ubuntu@YOUR_EC2_IP
```

### Step 2: Update System
```bash
sudo apt update
```

### Step 3: Install Ollama
```bash
curl -fsSL https://ollama.com/install.sh | sh
```

### Step 4: Pull Model
```bash
ollama pull qwen2.5:0.5b
```

Test (optional):
```bash
ollama run qwen2.5:0.5b "Hello"
# Press Ctrl+D to exit
```

### Step 5: Install Dependencies
```bash
sudo apt install -y python3-pip python3-venv nginx
```

### Step 6: Clone Repository
```bash
cd ~
git clone https://github.com/YOUR_USERNAME/ollama-app.git
cd ollama-app/backend
```

### Step 7: Setup Python Environment
```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

### Step 8: Create Systemd Service
```bash
sudo nano /etc/systemd/system/ollama-chat-api.service
```

**Paste:**
```ini
[Unit]
Description=Ollama Chat API
After=network.target ollama.service

[Service]
Type=simple
User=ubuntu
WorkingDirectory=/home/ubuntu/ollama-app/backend
Environment="PATH=/home/ubuntu/ollama-app/backend/venv/bin"
ExecStart=/home/ubuntu/ollama-app/backend/venv/bin/python -m uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
```

**Save:** Ctrl+X, Y, Enter

### Step 9: Start Service
```bash
sudo systemctl daemon-reload
sudo systemctl start ollama-chat-api
sudo systemctl enable ollama-chat-api
sudo systemctl status ollama-chat-api
```

### Step 10: Configure Nginx
```bash
sudo nano /etc/nginx/sites-available/ollama-chat-api
```

**Paste (replace YOUR_EC2_IP):**
```nginx
server {
    listen 80;
    server_name YOUR_EC2_IP;

    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_read_timeout 300s;
    }
}
```

**Save:** Ctrl+X, Y, Enter

### Step 11: Enable Nginx
```bash
sudo ln -s /etc/nginx/sites-available/ollama-chat-api /etc/nginx/sites-enabled/
sudo rm /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
```

### Step 12: Test Backend

Open in browser:
```
http://YOUR_EC2_IP:8000/api/health
```

✅ Should show JSON response!

---

## 💻 Frontend Deployment (S3)

### Step 1: Update API URL

**On local computer**, create `frontend/.env`:
```
REACT_APP_API_URL=http://YOUR_EC2_IP:8000
```

### Step 2: Install Dependencies
```bash
cd frontend
npm install
```

### Step 3: Test Locally (Optional)
```bash
npm start
```

Open `http://localhost:3000` - should connect to EC2 backend.

### Step 4: Build Production
```bash
npm run build
```

### Step 5: Create S3 Bucket

1. Go to AWS S3 Console
2. Click "Create bucket"
3. Bucket name: `ollama-chat-frontend-yourname`
4. Region: `us-east-1`
5. **UNCHECK** "Block all public access" ❌
6. Check acknowledgment ✅
7. Click "Create bucket"

### Step 6: Upload Files

1. Click on bucket name
2. Click "Upload"
3. Drag ALL files from `frontend/build/` folder:
   - `index.html`
   - `static/` folder
   - All other files
4. Click "Upload"
5. Wait for completion

### Step 7: Enable Website Hosting

1. Go to "Properties" tab
2. Scroll to "Static website hosting"
3. Click "Edit"
4. Select "Enable"
5. Hosting type: "Host a static website"
6. Index document: `index.html`
7. Error document: `index.html`
8. Click "Save changes"

### Step 8: Add Bucket Policy

1. Go to "Permissions" tab
2. Scroll to "Bucket policy"
3. Click "Edit"
4. Paste (replace YOUR_BUCKET_NAME):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::YOUR_BUCKET_NAME/*"
    }
  ]
}
```

5. Click "Save changes"

### Step 9: Get Website URL

Go to "Properties" → "Static website hosting" → Copy endpoint URL

Example: `http://your-bucket.s3-website-us-east-1.amazonaws.com`

### Step 10: Test

Open the S3 URL in browser - Your app is live! 🎉

---

## 🔧 Useful Commands

### Backend (EC2)

**View logs:**
```bash
sudo journalctl -u ollama-chat-api -f
```

**Restart service:**
```bash
sudo systemctl restart ollama-chat-api
```

**Check status:**
```bash
sudo systemctl status ollama-chat-api
```

**Update code:**
```bash
cd ~/ollama-app/backend
git pull
sudo systemctl restart ollama-chat-api
```

**Pull new Ollama model:**
```bash
ollama pull llama3.2
```

**List models:**
```bash
ollama list
```

### Frontend (Local)

**Update and redeploy:**
```bash
cd frontend
npm run build
# Then upload build/ folder to S3 again
```

**Using AWS CLI (faster):**
```bash
aws s3 sync build/ s3://your-bucket-name/ --delete
```

---

## 🔒 Security Group Configuration

**EC2 Instance Security Group:**

| Type | Port | Source | Description |
|------|------|--------|-------------|
| SSH | 22 | Your IP | SSH access |
| HTTP | 80 | 0.0.0.0/0 | HTTP |
| HTTPS | 443 | 0.0.0.0/0 | HTTPS |
| Custom TCP | 8000 | 0.0.0.0/0 | API (temporary) |

**Note:** Remove port 8000 after Nginx is working (use port 80 only)

---

## 💰 Cost Estimate

| Service | Specs | Cost/Month |
|---------|-------|------------|
| EC2 t3.medium | 2 vCPU, 4GB RAM | ~$30 |
| EBS Storage | 30GB | ~$3 |
| S3 | 1GB | ~$0.02 |
| Data Transfer | ~10GB | ~$1 |
| **Total** | | **~$34/month** |

---

## 🐛 Troubleshooting

### Backend not working
```bash
# Check service status
sudo systemctl status ollama-chat-api

# View recent logs
sudo journalctl -u ollama-chat-api -n 50

# Restart service
sudo systemctl restart ollama-chat-api
```

### Frontend shows "Disconnected"

1. Check browser console (F12 → Console)
2. Verify `.env` file has correct EC2 IP with `:8000`
3. Check CORS in `backend/main.py` - should have `allow_origins=["*"]`
4. Restart backend after CORS change

### Out of Memory

- Upgrade to t3.large (8GB RAM)
- Use smaller model: `ollama pull qwen2.5:0.5b`

---

## 📚 Project Structure
```
ollama-app/
├── backend/
│   ├── main.py              # FastAPI application
│   ├── requirements.txt     # Python dependencies
│   └── venv/               # Virtual environment
├── frontend/
│   ├── src/
│   │   └── App.js          # React main component
│   ├── public/
│   ├── build/              # Production build
│   ├── .env                # Environment variables
│   └── package.json
└── README.md
```

---

## 🎯 Tech Stack

**Backend:**
- FastAPI (Python web framework)
- Ollama (LLM runtime)
- Uvicorn (ASGI server)
- Nginx (Reverse proxy)
- Systemd (Service management)

**Frontend:**
- React
- React Markdown
- Syntax Highlighter

**Infrastructure:**
- AWS EC2 (Backend hosting)
- AWS S3 (Frontend hosting)
- Ubuntu 22.04 LTS

---

## 📝 Notes

- Model used: `qwen2.5:0.5b` (works on 4GB RAM)
- For better performance, use t3.large or GPU instance
- Backend URL: `http://YOUR_EC2_IP:8000`
- Frontend URL: `http://your-bucket.s3-website-region.amazonaws.com`

---

## 🤝 Contributing

Feel free to submit issues and pull requests!

---

## 📄 License

MIT License

---

## 🙏 Acknowledgments

- Ollama for the AI runtime
- FastAPI for the web framework
- AWS for infrastructure

---

**Happy Chatting with Ollama! 🦙**