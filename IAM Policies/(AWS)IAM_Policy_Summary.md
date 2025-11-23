# MercuryAI Project Policies

This project utilizes several key policies across the frontend and backend to manage **security**, **API interaction**, and **data handling**.  
The policies are based on the `server2.js`, `script.js`, and Nginx configuration files.

---

## 1. Security and Access Control Policy

Enforced primarily by the backend proxy server (`server2.js`).

| Policy Area | Implementation | Details |
|-------------|----------------|---------|
| **Cross-Origin Resource Sharing (CORS)** | `server2.js` | Only allows requests from explicitly approved frontend domains: GitHub Pages (`https://cdfkns.github.io/MercuryAI/`), Render (`https://mercuryai.onrender.com`), and local development addresses. All other origins are blocked. |
| **API Key Security** | `server2.js` | Gemini API key (`GEMINI_API_KEY`) stored as a secret environment variable. Never exposed to client-side. Server checks for key presence and returns error if missing. |
| **Payload Limit** | `server2.js` | JSON request body size limited to **50mb** to handle large inputs while protecting against excessive payloads. |

---

## 2. AI Interaction and Robustness Policy

Defines how the client (`script.js`) communicates with the external API.

| Policy Area | Implementation | Details |
|-------------|----------------|---------|
| **Model Selection** | `server2.js` | Uses `gemini-2.5-flash-preview-09-2025`, optimized for speed and efficiency. |
| **System Instruction** | `script.js` | Constructs a precise `systemPrompt` for every API call, including tone (Formal, Friendly, Technical). Ensures persona consistency. |
| **Retry Strategy** | `script.js` | Implements **exponential backoff** retry mechanism (up to 3 attempts) to handle temporary failures gracefully. |
| **Error Reporting** | `script.js` & `server2.js` | Explicitly checks for "API Key Missing" error and provides actionable configuration message to user. |

---

## 3. Data Persistence and UI Policy

Controls how user outputs are stored and presented (`script.js`).

| Policy Area | Implementation | Details |
|-------------|----------------|---------|
| **Data Storage** | `script.js` | AI-generated content saved to browser **`localStorage`** under key `mercury_ai_saved_outputs`. Non-persistent if cache is cleared. |
| **Saved Data** | `script.js` | Each saved item includes content, unique ID, save date/time, and tone used. |
| **User Interaction** | `script.js` | Default browser prompts (`alert`, `confirm`) are forbidden. Replaced with custom modal UI (`showCustomMessage`) for consistent UX. |

---

## 4. Deployment Policy (Elastic Beanstalk / Nginx)

Defines cloud deployment configuration using AWS Elastic Beanstalk and Nginx.

| Policy Area | Implementation | Details |
|-------------|----------------|---------|
| **Web Server Role** | `.platform/nginx/nginx.conf` | **Nginx** configured as public-facing web server, listening on port **80**. |
| **Application Port** | `.platform/nginx/nginx.conf` | Nginx reverse proxies traffic to Node.js Express app running internally on port **8080** (Elastic Beanstalk standard). |
| **Configuration Style** | `.platform/nginx/nginx.conf` | Monolithic `location /` block ensures all requests are proxied to Node.js server. |

---

## Summary

- **Backend (`server2.js`)** enforces strict CORS, API key security, and payload limits.  
- **Frontend (`script.js`)** ensures robust AI interaction with retry logic, persona consistency, and custom UI.  
- **Data handling** relies on localStorage with structured metadata.  
- **Deployment** uses Elastic Beanstalk with Nginx reverse proxy for streamlined hosting.
