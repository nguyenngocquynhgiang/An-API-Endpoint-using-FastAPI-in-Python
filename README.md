# **Deploy LLM Applications to API Endpoints Using FastAPI in Python**

LLMs like GPT, Claude, and LLaMA are revolutionizing chatbots, content creation, and many other applications. APIs act as essential bridges, allowing for seamless integration of complex language understanding and generation capabilities into projects.

This guide will help you build a simple Python application using OpenAI's GPT API and deploy it to a REST endpoint using the FastAPI framework.

---

## **1. Create a Simple FastAPI Web Server**

```python
from typing import Union
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def read_root():
    return {"Hello": "World"}

@app.get("/items/{item_id}")
def read_item(item_id: int, q: Union[str, None] = None):
    return {"item_id": item_id, "q": q}
```

This script creates a simple web server using FastAPI. The server responds to two types of web requests:
- Visiting `"/"` returns `{"Hello": "World"}`.
- Visiting `"/items/{item_id}"` displays the item ID and an optional query parameter `q`.

Run the server using:

```python
!uvicorn main:app --reload
```

---

## **2. Build an LLM-powered API**

We will build a REST API endpoint that uses OpenAI's GPT model to translate English text into French.

### **List Available OpenAI Models**

```python
from openai import OpenAI

client = OpenAI(api_key="your-api-key")
models = client.models.list()
print([model.id for model in models.data])
```

This prints all available OpenAI models that you can use.

### **Implement the Translation Function**

```python
from openai import OpenAI
import os

os.environ["OPENAI_API_KEY"] = "your-api-key"
client = OpenAI()

def translate_text(input_str):
    completion = client.chat.completions.create(
        model="gpt-4o",
        messages=[
            {"role": "system", "content": "You are an expert translator who translates text from English to French and only returns translated text."},
            {"role": "user", "content": input_str},
        ],
    )
    return completion.choices[0].message.content
```

### **Breakdown of the Code**

#### **Import Libraries**
- `from openai import OpenAI`: Import OpenAI SDK.
- `import os`: Load environment variables.

#### **Set API Key**
- Store the API key in `os.environ["OPENAI_API_KEY"]`.
- OpenAI SDK automatically reads it from the environment.

#### **Translation Function**
- Uses `model="gpt-4o"`.
- Sends an English text input and receives a French translation.
- Extracts and returns the translated text from the API response.

🔴 **Note**: Ensure you have an active OpenAI subscription; otherwise, the API may return an `insufficient_quota` error.

---

## **3. Deploy as a FastAPI Endpoint**

### **Why Use POST?**
- Translation modifies data, making `POST` more suitable than `GET`.
- `POST` supports sending large text inputs in the request body.

### **Create a FastAPI Endpoint for Translation**

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel

app = FastAPI()

class TranslationRequest(BaseModel):
    input_str: str

@app.post("/translate/")  # Define a POST endpoint
async def translate(request: TranslationRequest):
    try:
        translated_text = translate_text(request.input_str)
        return {"translated_text": translated_text}
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### **Code Explanation**

1. **Define `TranslationRequest` Pydantic model**:
   - Ensures request payloads contain a valid `input_str` field.

2. **Create `@app.post("/translate/")`**:
   - Handles POST requests to `/translate/`.
   - Calls `translate_text(request.input_str)`.

3. **Error Handling**:
   - If an error occurs, returns an HTTP `500 Internal Server Error`.

### **Example API Request**

```json
{
    "input_str": "Hello, how are you?"
}
```

### **Example API Response**

```json
{
    "translated_text": "Bonjour, comment ça va?"
}
```

---

## **4. Running the FastAPI Server**

To start the FastAPI application, run:

```bash
uvicorn main:app --reload
```

Then, visit `http://127.0.0.1:8000/docs` to test your API using the interactive Swagger UI.

---

## **5. Results and Deployment**

After running the server, you should see:

![FastAPI Translation API](https://www.datacamp.com/)

This confirms that your FastAPI-powered GPT translation API is up and running!

---

## **6. Summary**

✅ **Created a simple FastAPI web server**.
✅ **Integrated OpenAI's GPT API for translation**.
✅ **Built a REST API to expose translation functionality**.
✅ **Handled errors effectively**.
✅ **Deployed and tested the API**.

🚀 You can now extend this by deploying the API to **AWS, Google Cloud, or Heroku** for production use!

