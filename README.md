# genai
import requests
import os

NEWS_API_KEY = os.getenv("NEWS_API_KEY")

def fetch_news(query, language='en'):
    url = f"https://newsapi.org/v2/everything?q={query}&language={language}&sortBy=publishedAt&apiKey={NEWS_API_KEY}"
    res = requests.get(url)
    articles = res.json().get('articles', [])
    return [f"{a['title']}: {a['description']}" for a in articles[:5]]

    
**#memory file**


from supabase import create_client
import requests
import os

SUPABASE_URL = os.getenv("SUPABASE_URL")
SUPABASE_API_KEY = os.getenv("SUPABASE_API_KEY")
supabase = create_client(SUPABASE_URL, SUPABASE_API_KEY)

def store_user_embedding(user_id, text, embedding):
    supabase.table("memory").insert({
        "user_id": user_id,
        "text": text,
        "embedding": embedding
    }).execute()

def retrieve_similar_context(user_id, query_embedding, top_k=3):
    
    data = supabase.table("memory").select("*").eq("user_id", user_id).execute().data
    return data[:top_k]  


**#main file**


from fastapi import FastAPI, Request
from agent.chatbot import process_query

app = FastAPI()

@app.post("/chat")
async def chat(req: Request):
    data = await req.json()
    user_id = data["user_id"]
    query = data["query"]
    result = process_query(user_id, query)
    return result


**#chatbot**

from agent.memory import store_user_embedding, retrieve_similar_context
from agent.tools import fetch_news

def process_query(user_id, query):
    # 1. Get real-time news
    news = fetch_news(query)

    # 2. Use Gemini API for embeddings & chat (mockup here)
    embedding = [0.1] * 768  # Replace with Gemini embeddings call
    store_user_embedding(user_id, query, embedding)

    past = retrieve_similar_context(user_id, embedding)
    return {
        "past_context": past,
        "latest_news": news
    }

    
#gemini file****


GEMINI_API_KEY=your_gemini_api_key
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_API_KEY=your_supabase_anon_key
NEWS_API_KEY=your_newsapi_key
    
