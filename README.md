Certainly! Here’s an updated version of the README file with details about the RAG (Retrieval-Augmented Generation) tool implementation:

---

# Automated Email Reply System

## Overview

This repository contains an automated email reply system designed to categorize, handle, and respond to incoming emails efficiently. Utilizing advanced language models and SQL database integration, this system ensures accurate and timely email management for film equipment rental services.

## Features

- **Email Categorization**: Automatically classifies emails into categories such as inquiries, reviews, and assistance requests.
- **Inquiry Handling**: Checks the availability of film equipment and provides pricing information from a SQL database.
- **Review Management**: Responds to both positive and negative reviews with appropriate actions.
- **Assistance Request Resolution**: Provides solutions or escalates assistance requests to customer service.
- **Email Drafting**: Generates professional email responses based on categorized information and agent feedback.

## Installation

### Required Libraries

Install the necessary libraries using the following commands:

```bash
pip -q install crewai langchain-groq duckduckgo-search
pip install -qU langchain-core langchain-community 'crewai[tools]'
pip install -qU composio_langchain sentence-transformers langchain_huggingface
```

### Setup SQL Database

Set up a SQLite database to store details about film equipment:

```python
import sqlite3

conn = sqlite3.connect('film_equipment.db')
cursor = conn.cursor()

cursor.execute('''
      CREATE TABLE equipment (
          id INTEGER PRIMARY KEY AUTOINCREMENT,
          item TEXT NOT NULL,
          category TEXT NOT NULL,
          price REAL,
          availability BOOLEAN
      )
''')

equipment_data = [
    ("Cannon XA70", "Camera", 2000.0, True),
    ("Sony A7 III", "Camera", 1800.0, False),
    ("ARRI Alexa", "Camera", 1500.0, True),
    # Add other equipment data here
]

cursor.executemany('INSERT INTO equipment (item, category, price, availability) VALUES (?, ?, ?, ?)', equipment_data)
conn.commit()
conn.close()
```

### Configuration

Set up necessary environment variables for Groq and Composio APIs:

```python
import os
from google.colab import userdata

os.environ["GROQ_API_KEY"] = userdata.get("GROQ_API_KEY")
os.environ["COMPOSIO_API_KEY"] = userdata.get("COMPOSIO_API_KEY")
```

## RAG Tool Implementation

### Overview

The Retrieval-Augmented Generation (RAG) tool enhances the system's capability by integrating external knowledge and providing accurate responses to assistance requests. It uses a combination of document retrieval and generation to find relevant information and draft appropriate replies.

### Setup RAG Tool

1. **PDF Search Tool Setup**:
   The RAG tool utilizes the `PDFSearchTool` to perform document-based retrieval. It searches through a PDF containing frequently asked questions about film equipment to find relevant information.

   ```python
   from crewai_tools import PDFSearchTool

   rag_tool = PDFSearchTool(
       pdf='Film_Equipment_FAQs.pdf',
       config=dict(
           llm=dict(
               provider="groq",
               config=dict(
                   model="llama3-70b-8192",
               ),
           ),
           embedder=dict(
               provider="huggingface",
               config=dict(
                   model="BAAI/bge-small-en-v1.5",
               ),
           ),
       )
   )
   ```

   - **PDF**: `Film_Equipment_FAQs.pdf` - Document containing FAQs about film equipment.
   - **LLM**: `llama3-70b-8192` - Large language model used for generation tasks.
   - **Embedder**: `BAAI/bge-small-en-v1.5` - Embedding model for document retrieval.

### Integration in Email System

The RAG tool is integrated with the Assistance Request Handling Agent to handle requests efficiently. It performs the following tasks:

- **Search for Solutions**: Uses the `rag_tool` to find solutions or relevant information from the provided FAQ document.
- **Escalate if Needed**: If no solution is found, the issue is escalated to customer service.

### Example Usage

Here's how the RAG tool is used within the `Assistance Request Handling Agent`:

```python
class EmailAgents:
    # Assistance Request Handling Agent
    def make_assistance_request_handling_agent(self):
        return Agent(
            role='Assistance Request Handling Agent',
            goal="""Take in an email from a human that has emailed our company email address with an assistance request. The category \
            'assistance_request' given by the categorizer agent, then:
            - If a suitable solution is found, provide it in the reply without referencing the search process or any internal documents.
            - If no solution is found, escalate the issue to customer service.""",
            backstory="""You are skilled at finding solutions to customer issues or escalating them to ensure resolution.""",
            llm=GROQ_LLM,
            verbose=True,
            allow_delegation=False,
            max_iter=10,
            memory=True,
            tools=[rag_tool],
        )
```

## Implementation

### Agents and Tasks

- **Categorizer Agent**: Categorizes incoming emails into inquiries, reviews, or assistance requests.
- **SQL Agent**: Handles SQL queries to check availability and pricing of equipment.
- **Review Handling Agent**: Manages responses to positive and negative reviews.
- **Assistance Request Handling Agent**: Uses the RAG tool to find solutions or escalates requests.
- **Draft Email Writer Agent**: Drafts final email responses based on categorized information and agent feedback.

### Usage Example

Define your email content and execute the system:

```python
email_content = """HI there, \n
I am interested in the Sony A7 III and would like to know if it is currently in stock.\n
Could you please provide me with the availability status and pricing details? \n
Thank you for your prompt attention to this matter.
Thanks,
George
"""

# Initialize Agents and Tasks
agents = EmailAgents()
tasks = EmailTasks()

categorizer_agent = agents.make_categorizer_agent()
sql_agent = agents.make_sql_agent()
review_handling_agent = agents.make_review_handling_agent()
assistance_request_handling_agent = agents.make_assistance_request_handling_agent()
draft_email_writer_agent = agents.make_draft_email_writer_agent()

categorize_email = tasks.categorize_email(email_content)
sql_task = tasks.sql_task(email_content)
review_handling_task = tasks.review_handling_task(email_content)
assistance_request_handling_task = tasks.assistance_request_handling_task(email_content)
draft_email = tasks.draft_email(email_content, categorize_email, sql_task, review_handling_task, assistance_request_handling_task)

# Create Crew and Execute Tasks
crew = Crew(
    agents=[categorizer_agent, sql_agent, review_handling_agent, assistance_request_handling_agent, draft_email_writer_agent],
    tasks=[categorize_email, sql_task, review_handling_task, assistance_request_handling_task, draft_email],
    verbose=2,
    full_output=True,
    share_crew=False,
)

results = crew.kickoff()
print(results)
```

## Contributing

Contributions are welcome! Please fork this repository and submit pull requests for any improvements or features. For issues, open an issue on GitHub.

Feel free to modify the details as needed for your project.
