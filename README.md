# LLM API Lab

This lab guides you through setting up a simple FastAPI application that uses Ollama to serve a local LLM (like Mistral). You'll build the app, secure it with an API key, and test it via curl/Postman.

## Lab Setup

### Step 1: Install Python and pip
Ensure Python 3.x and pip are installed:
```
python3 --version
pip3 --version
```
If not installed, download from [https://python.org](https://python.org).

### Step 2: Install Required Libraries

Create a `requirements.txt` file:
```
fastapi
uvicorn
ollama
python-dotenv
requests
```

Install with:
```
pip3 install -r requirements.txt
```

### Step 3: Set Up Ollama

1. Download Ollama: https://ollama.com/download
2. Pull a model:
```
ollama pull mistral
```
3. Test it:
```
ollama run mistral
```
Exit with `/bye`.

## Lab Steps

### Step 1: Create a Simple FastAPI Application

Create project folder:
```
mkdir llm-api-lab
cd llm-api-lab
```

Initialize git:
```
git init
```

Create `main.py`:
```python
from fastapi import FastAPI
import ollama

app = FastAPI()

@app.post("/generate")
def generate(prompt: str):
    response = ollama.chat(model="mistral", messages=[{"role": "user", "content": prompt}])
    return {"response": response["message"]["content"]}
```

Run app:
```
uvicorn main:app --reload
```

Commit:
```
git add main.py requirements.txt
git commit -m "Initial FastAPI app with /generate endpoint"
```

### Step 2: Test the Unsecured API

With curl:
```
curl -X POST "http://localhost:8000/generate?prompt=Hello%20World"
```

Or with Postman (POST to the same URL).

### Step 3: Secure the API with API Key

Update `main.py`:
```python
from fastapi import FastAPI, Depends, HTTPException, Header
import ollama
import os
from dotenv import load_dotenv

app = FastAPI()
load_dotenv()
API_KEYS = {os.getenv("API_KEY")}

def verify_api_key(x_api_key: str = Header(None)):
    if x_api_key not in API_KEYS:
        raise HTTPException(status_code=401, detail="Invalid API Key")
    return x_api_key

@app.post("/generate")
def generate(prompt: str, api_key: str = Depends(verify_api_key)):
    response = ollama.chat(model="mistral", messages=[{"role": "user", "content": prompt}])
    return {"response": response["message"]["content"]}
```

Create `.env`:
```
API_KEY=your_secret_key
```

Create `.gitignore`:
```
.env
```

Commit:
```
git add main.py .gitignore
git commit -m "Added API key authentication"
```

Restart:
```
uvicorn main:app --reload
```

### Step 4: Test Secured API

Correct key:
```
curl -X POST "http://localhost:8000/generate?prompt=Hello%20World" -H "x-api-key: your_secret_key"
```

Missing or wrong key:
Expect 401 Unauthorized.

Test with Postman by setting `x-api-key` in headers.

---
