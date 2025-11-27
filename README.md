Requirments-
OPENROUTER_API_KEY="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"
LANGSMITH_API_KEY
LANGSMITH_PROJECT="openrouter-chatbot"
LANGCHAIN_TRACING_V2="true"

agent-
import os
from dotenv import load_dotenv
from openai import OpenAI

load_dotenv()

client = OpenAI(
    base_url="https://openrouter.ai/api/v1",
    api_key=os.getenv("OPENROUTER_API_KEY"),
)

def process_user_message(message: str):
    response = client.chat.completions.create(
        model="openai/gpt-4o-mini",
        extra_headers={
            "HTTP-Referer": "https://example.com",
            "X-Title": "OpenRouter Chatbot",
        },
        messages=[
            {"role": "user", "content": message}
        ]
    )
    return response.choices[0].message.content

graphs - 
from langgraph.graph import StateGraph, END
from agent import process_user_message

# State definition (optional but recommended)
class ChatState(dict):
    message: str
    reply: str | None

def input_node(state: ChatState):
    return {"message": state["message"]}

def ai_node(state: ChatState):
    reply = process_user_message(state["message"])
    return {"reply": reply}

# Build graph using StateGraph
workflow_graph = StateGraph(ChatState)

workflow_graph.add_node("input", input_node)
workflow_graph.add_node("ai", ai_node)

workflow_graph.add_edge("input", "ai")
workflow_graph.add_edge("ai", END)

workflow_graph.set_entry_point("input")

workflow = workflow_graph.compile()

langsmith setup - 
import os
from dotenv import load_dotenv

def setup_langsmith():
    load_dotenv()

    os.environ["LANGCHAIN_TRACING_V2"] = "true"
    os.environ["LANGSMITH_API_KEY"] = os.getenv("LANGSMITH_API_KEY")
    os.environ["LANGSMITH_PROJECT"] = os.getenv("LANGSMITH_PROJECT")

main.py - 
from fastapi import FastAPI
from graph import workflow
from langsmith_setup import setup_langsmith
import uvicorn

setup_langsmith()

app = FastAPI()

@app.get("/")
def home():
    return {"message": "OpenRouter Chatbot with LangGraph + LangSmith is running"}

@app.post("/chat")
def chat(data: dict):
    user_msg = data.get("message", "")
    result = workflow.invoke({"message": user_msg})
    return {"reply": result["reply"]}

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)


Requirments - 
openai
python-dotenv
langchain
langsmith
langgraph
fastapi
uvicorn

ro.py - 
import streamlit as st
from graph import workflow
from langsmith_setup import setup_langsmith

# Initialize LangSmith
setup_langsmith()

st.set_page_config(
    page_title="OpenRouter Chatbot",
    page_icon="🤖",
    layout="wide"
)

st.title("💬 OpenRouter Chatbot (LangGraph + LangSmith)")

# Create chat history
if "history" not in st.session_state:
    st.session_state["history"] = []

# Chat display
for chat in st.session_state["history"]:
    with st.chat_message(chat["role"]):
        st.write(chat["content"])

# Chat input
user_input = st.chat_input("Type your message...")

if user_input:
    # Show user's message
    st.session_state["history"].append({"role": "user", "content": user_input})
    with st.chat_message("user"):
        st.write(user_input)

    # Process using LangGraph workflow
    result = workflow.invoke({"message": user_input})
    reply = result["reply"]

    # Show bot message
    st.session_state["history"].append({"role": "assistant", "content": reply})
    with st.chat_message("assistant"):
        st.write(reply)
