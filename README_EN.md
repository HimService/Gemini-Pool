# Gemini-Pool - Gemini API Key Pool

[繁體中文](README.md) | [English](README_EN.md)

<blockquote style="color: red;">
  <p><strong>Warning!</strong><br>
  Gemini-Pool version v0.17a contains critical vulnerabilities. It is strongly recommended to upgrade to Gemini-Pool v0.59a to fix the known issues.</p>
</blockquote>


## Overview

Gemini-Pool is a Google Gemini API key pool project developed with the FastAPI framework. It aims to help developers manage and use multiple Gemini API keys more effectively, providing features like automatic load balancing, key rotation, failover retry, and a monitoring dashboard for request logs.

### Features
- **Key Pool Management**: Centrally manage all your keys.
- **Automatic Rotation & Load Balancing**: Rotates among available keys to distribute request pressure evenly.
- **Failover**: Automatically switches to the next available key when a key fails, ensuring service availability.
- **Periodic Health Checks**: Automatically checks inactive keys periodically and adds them back to the active pool upon recovery.
- **Detailed Dashboard**: Provides a web interface for real-time monitoring of key status, request traffic, and detailed logs.
- **Dynamic Configuration**: Manage keys and settings dynamically through the dashboard, with most changes not requiring a service restart.
- **Access Control**: Configure client API keys allowed to access this proxy service, with fine-grained traffic management.
- **Log Management**: Detailed request logging.

## Workflow

The core workflow of this project is as follows:

1.  **Request Interception**: Your application sends API requests to `Gemini-Pool` instead of directly to Google.
2.  **Key Selection**: `Gemini-Pool` selects a currently available and low-load API key from its managed pool.
3.  **Request Forwarding**: The original request is forwarded to the Google Gemini API endpoint using the selected key.
4.  **Successful Response**: If the request is successful, `Gemini-Pool` returns the response from Google to your application.
5.  **Failure Handling**: If a request fails (e.g., due to key quota exhaustion), the system automatically marks the key as temporarily unavailable and retries the request with the **next** available key. This process is transparent to your application.
6.  **Logging & Monitoring**: Detailed information for all requests is logged and can be monitored in real-time on the admin dashboard.

<blockquote style="color: red;">
  <p><strong>Warning!</strong><br>
  We cannot guarantee the advanced security and stability of Gemini-Pool. It is recommended to use it for testing and experimental purposes only.</p>
</blockquote>

## 🛠️ Installation

### Prerequisites
- Python 3.8 or higher.
- One or more valid Google Gemini API keys.

### Steps
1. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

2. **Set Up Environment Variables**
   - In the `gemini_key_pooler` directory, copy `.env.example` to `.env` (create it manually if it doesn't exist).
   - Edit the `.env` file to fill in the necessary information. See the "Configuration" section below for details.

3. **Start the Server**
   ```bash
   python -m gemini_key_pooler.main
   ```
   - The console will display `Uvicorn running on http://0.0.0.0:8000`
   - Access `http://localhost:8000` in your browser to go to the login page.

## Usage

### Login to the Admin Panel
- **Access URL**: `http://localhost:8000`
- **Login**: Use the `ADMIN_USERNAME` and `ADMIN_PASSWORD` you set in your `.env` file. After successful login, you will be redirected to the `/status` dashboard.

### API Proxy Usage
You need to change your application to make requests to this proxy service endpoint instead of the official Google Gemini API.

- **Proxy Endpoint**: `http://localhost:8000/v1beta/models/{model_id}:generateContent`
- **Authentication**: Include one of the keys you set in `ALLOWED_API_KEYS` in the `Authorization` header of your request.

### cURL Example
```bash
# Assuming "CLIENT_KEY_1" is in your ALLOWED_API_KEYS
curl -X POST http://localhost:8000/v1beta/models/gemini-1.5-flash:generateContent \
-H "Authorization: Bearer CLIENT_KEY_1" \
-H "Content-Type: application/json" \
-d '{
  "contents": [{
    "parts":[{
      "text": "Hi, :D"
    }]
  }]
}'
```

## ⚙️ Configuration

All configurations for this project are managed via the `.env` file.

[View Configuration](https://himservice-docs.himserver.com/#Gemini-Pool/set-env)
Please note: The website currently only supports Traditional Chinese.

## API Endpoints

This project provides two types of API endpoints: Admin and Proxy.

[View Documentation](https://himservice-docs.himserver.com/#Gemini-Pool/api)
Please note: The website currently only supports Traditional Chinese.

## Reporting Issues

## Development & Contribution

## License
Please read the `LICENSE` file.
