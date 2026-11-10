import streamlit as st
import pandas as pd
import google.generativeai as genai

# --- Configuration ---
st.set_page_config(page_title="Veda eScoot X1 - PLM Agent", page_icon="🛵", layout="centered")

# --- UI Header ---
st.title("🛵 Veda eScoot X1 - PLM AI Agent")
st.markdown("Your intelligent assistant for Product Lifecycle Management (PLM) queries across design, manufacturing, quality, and service departments.")

# --- API Key Setup ---
api_key = st.secrets.get("GEMINI_API_KEY", "")
if not api_key:
    api_key = st.text_input("Enter Google Gemini API Key (Get a free key from Google AI Studio):", type="password")

if api_key:
    genai.configure(api_key=api_key)
    # Using Gemini 1.5 Flash (free & fast)
    model = genai.GenerativeModel('gemini-1.5-flash')

    # --- Load Dataset ---
    @st.cache_data
    def load_data():
        xls = pd.ExcelFile('dataset.xlsx')
        sheet_data = {}
        for sheet_name in xls.sheet_names:
            sheet_data[sheet_name] = pd.read_excel(xls, sheet_name=sheet_name).to_string()
        return sheet_data

    try:
        data = load_data()
    except FileNotFoundError:
        st.error("Error: 'dataset.xlsx' not found in the project folder.")
        st.stop()

    # --- Build Context for AI ---
    dataset_context = (
        "You are an expert PLM (Product Lifecycle Management) AI Agent for the electric scooter 'Veda eScoot X1'. "
        "Answer the user's question accurately based strictly on the dataset provided below. "
        "Do not invent or assume information outside the dataset.\n\n"
    )
    for sheet, content in data.items():
        dataset_context += f"--- Sheet: {sheet} ---\n{content}\n\n"

    # --- Sample Questions Sidebar / Quick Prompts ---
    st.sidebar.header("💡 Suggested Questions")
    sample_questions = [
        "What is the battery capacity of the electric scooter?",
        "What are the major components of the scooter?",
        "Which engineering changes are currently open?",
        "Which components are affected by engineering change ECR-001?",
        "What validation tests are required for ECR-001?",
        "What maintenance actions are recommended for the braking system?"
    ]
    
    selected_sample = st.sidebar.selectbox("Click to choose a sample question:", ["Select a question..."] + sample_questions)
    
    # --- Chat Interface ---
    if selected_sample != "Select a question...":
        question = st.text_input("Ask a question about the eScoot:", value=selected_sample)
    else:
        question = st.text_input("Ask a question about the eScoot:")
    
    if st.button("Submit Question", type="primary"):
        if question:
            with st.spinner("Analyzing PLM dataset..."):
                prompt = dataset_context + f"\nUser Question: {question}\nAnswer:"
                response = model.generate_content(prompt)
                
                st.markdown("### 📋 Answer:")
                st.success(response.text)
        else:
            st.warning("Please enter or select a question first.")
else:
    st.info("💡 Please enter your Gemini API key above to start querying the PLM agent.")
