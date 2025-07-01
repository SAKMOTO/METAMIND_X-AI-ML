# METAMIND_X-AI-ML
MetaMind X is creating an AI help bot for the MOSDAC portal using NLP and knowledge graphs. It extracts data from documents and web content to answer user queries instantly. Built with Python tools like spaCy and BeautifulSoup, it's fast, accurate, and easily adaptable to other portals.
##Step 1: Set Up Google Colab
Open Google Colab

Go to https://colab.research.google.com

Click "New Notebook" (bottom-right)

Rename Your Notebook

Click on "Untitled.ipynb" (top-left)

Rename to METAMIND_X


##Step 2: Install & Import Libraries
Run this in the first cell:

```
!pip install beautifulsoup4 requests chromadb sentence-transformers
```
```
import requests
from bs4 import BeautifulSoup
import json
from sentence_transformers import SentenceTransformer
import chromadb
```

What this does:

Installs tools for scraping (beautifulsoup4), AI (sentence-transformers), and database (chromadb).

Imports them for use in the notebook.

##Step 3: Scrape MOSDAC FAQs (or Use Sample Data)
Option A: Scrape Real FAQs
```
python
url = "https://www.mosdac.gov.in/faq"
response = requests.get(url)
```
Fetches the MOSDAC FAQ webpage.
```
python
soup = BeautifulSoup(response.text, 'html.parser')
```
Parses the HTML content for scraping.
```
python
faqs = []
for item in soup.select('.panel.panel-default'):
    question = item.find('a', class_='accordion-toggle').text.strip()
    answer = item.find('div', class_='panel-body').text.strip()
    faqs.append({"question": question, "answer": answer})
```
Extracts questions/answers using CSS selectors (adjust based on actual HTML structure).
Option B: Manual FAQs (Fallback)
```
python
faqs = [
    {"question": "How to download data?", "answer": "Register on MOSDAC portal..."},
    # Add more Q&A pairs
]
```
```
python
with open('faqs.json', 'w') as f:
    json.dump(faqs, f, indent=4)
```
##4. AI Knowledge Base Setup
```
python
model = SentenceTransformer('all-MiniLM-L6-v2')
```
Loads a pre-trained AI model for converting text to numerical embeddings.
```
python
client = chromadb.Client()
collection = client.create_collection("mosdac_faqs")
```
Creates an in-memory ChromaDB collection.
```
python
for i, faq in enumerate(faqs):
    embedding = model.encode(faq["question"]).tolist()
    collection.add(
        ids=[str(i)],
        embeddings=[embedding],
        documents=[faq["answer"]]
    )
```
Converts each question to an embedding and stores it with its answer in the database.
Saves FAQs to a JSON file for later use.
##5. Question-Answering Function
```
python
def ask_question(question):
    if not question.strip():
        return "Please enter a valid question."
```
Handles empty input.
```
python
    results = collection.query(
        query_texts=[question],
        n_results=1
    )
```
Searches the database for the most similar question to the user's input.
```
python
    return results['documents'][0][0] if results['documents'] else "Sorry, I couldn't find an answer."
```
Returns the matched answer or a default message.
##6. Testing
```
python
print(ask_question("How to download data?"))
```
Tests the bot with a sample question.
##7. GitHub Integration
```
python
from google.colab import drive
drive.mount('/content/drive')
```
Mounts Google Drive to save files.

```
python
!git clone https://github.com/yourusername/your-repo.git
!cp faqs.json /content/drive/your-repo/
!cp ISRO_MOSDAC_HelpBot.ipynb /content/drive/your-repo/
```
Clones your GitHub repo and copies project files.
```
python
%cd /content/drive/your-repo
!git add .
!git commit -m "Added bot files"
!git push origin main
```
Pushes changes to GitHub.
