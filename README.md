# Getting Started with Create React App

This project was bootstrapped with [Create React App](https://github.com/facebook/create-react-app).

## Available Scripts

In the project directory, you can run:

### `npm start`

Runs the app in the development mode.\
Open [http://localhost:3000](http://localhost:3000) to view it in your browser.

The page will reload when you make changes.\
You may also see any lint errors in the console.

### `npm test`

Launches the test runner in the interactive watch mode.\
See the section about [running tests](https://facebook.github.io/create-react-app/docs/running-tests) for more information.

### `npm run build`

Builds the app for production to the `build` folder.\
It correctly bundles React in production mode and optimizes the build for the best performance.

The build is minified and the filenames include the hashes.\
Your app is ready to be deployed!

See the section about [deployment](https://facebook.github.io/create-react-app/docs/deployment) for more information.

### `npm run eject`

**Note: this is a one-way operation. Once you `eject`, you can't go back!**

If you aren't satisfied with the build tool and configuration choices, you can `eject` at any time. This command will remove the single build dependency from your project.

Instead, it will copy all the configuration files and the transitive dependencies (webpack, Babel, ESLint, etc) right into your project so you have full control over them. All of the commands except `eject` will still work, but they will point to the copied scripts so you can tweak them. At this point you're on your own.

You don't have to ever use `eject`. The curated feature set is suitable for small and middle deployments, and you shouldn't feel obligated to use this feature. However we understand that this tool wouldn't be useful if you couldn't customize it when you are ready for it.

## Learn More

You can learn more in the [Create React App documentation](https://facebook.github.io/create-react-app/docs/getting-started).

To learn React, check out the [React documentation](https://reactjs.org/).

### Code Splitting

This section has moved here: [https://facebook.github.io/create-react-app/docs/code-splitting](https://facebook.github.io/create-react-app/docs/code-splitting)

### Analyzing the Bundle Size

This section has moved here: [https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size](https://facebook.github.io/create-react-app/docs/analyzing-the-bundle-size)

### Making a Progressive Web App

This section has moved here: [https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app](https://facebook.github.io/create-react-app/docs/making-a-progressive-web-app)

### Advanced Configuration

This section has moved here: [https://facebook.github.io/create-react-app/docs/advanced-configuration](https://facebook.github.io/create-react-app/docs/advanced-configuration)

### Deployment

This section has moved here: [https://facebook.github.io/create-react-app/docs/deployment](https://facebook.github.io/create-react-app/docs/deployment)

### `npm run build` fails to minify

This section has moved here: [https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify](https://facebook.github.io/create-react-app/docs/troubleshooting#npm-run-build-fails-to-minify)









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




cd Downloads

chmod 400 ollama-key.pem

ssh -i ollama-key.pem ubuntu@YOUR_EC2_IP

curl -fsSL https://ollama.com/install.sh | sh


 ollama --version

 ollama pull llama3.2:1b

 ollama pull qwen2.5:0.5b


sudo apt update

sudo apt install -y python3-pip python3-venv nginx

qwen2.5:0.5b

http://3.80.136.92:8000/api/health











REACT_APP_API_URL = http://98.82.135.92

const API_URL = process.env.REACT_APP_API_URL || 'http://localhost:8000';

npm start


allow_origins=["*"]  # Changed this



sudo nano /etc/systemd/system/ollama-chat-api.service

sudo systemctl daemon-reload
sudo systemctl start ollama-chat-api
sudo systemctl enable ollama-chat-api
sudo systemctl status ollama-chat-api


sudo systemctl daemon-reload
sudo systemctl restart ollama-chat-api
sudo systemctl status ollama-chat-api

npm run build
