# Email-automation-task
Here's a template for a README file for your GitHub repository:

---

# Automated Email Reply System

## Overview

This project implements an automated email reply system designed to categorize incoming emails, handle specific requests, and draft appropriate responses. The system uses a set of specialized agents and tasks to ensure efficient and accurate email management.

## Features

- **Email Categorization**: Classify emails into categories such as inquiries, reviews, and assistance requests.
- **Inquiry Handling**: Retrieve and provide information about items from a database based on customer inquiries.
- **Review Management**: Respond to both positive and negative reviews with appropriate actions.
- **Assistance Request Resolution**: Provide solutions to assistance requests or escalate them if needed.
- **Email Drafting**: Generate well-crafted email responses based on categorized information and agent responses.

## Setup

### Prerequisites

- Python 3.7 or later
- Required Python libraries: `GROQ_LLM`, `sql_tools`, `rag_tool`

### Installation

1. **Clone the Repository**
   ```bash
   git clone https://github.com/yourusername/automated-email-reply-system.git
   cd automated-email-reply-system
   ```

2. **Install Dependencies**
   ```bash
   pip install -r requirements.txt
   ```

3. **Configure the Database**
   - Ensure you have a SQLite database file named `film_equipment.db` in the project directory.

### Configuration

- **Agents**: Defined for categorizing emails, handling SQL queries, managing reviews, resolving assistance requests, and drafting emails.
- **Tasks**: Set up for categorizing emails, querying the database, handling reviews, managing assistance requests, and drafting final replies.

## Usage

1. **Define Your Email Content**: Prepare the email content you want to process.
2. **Run the Agents and Tasks**: Use the provided agents and tasks to categorize, handle, and draft responses to emails.

   ```python

    agents = EmailAgents()
    tasks = EmailTasks()
    
    ## Agents
    categorizer_agent = agents.make_categorizer_agent()
    sql_agent = agents.make_sql_agent()
    review_handling_agent = agents.make_review_handling_agent()
    assistance_request_handling_agent = agents.make_assistance_request_handling_agent()
    draft_email_writer_agent = agents.make_draft_email_writer_agent()
    
    ## Tasks
    categorize_email = tasks.categorize_email(email_content)
    sql_task = tasks.sql_task(email_content)
    review_handling_task = tasks.review_handling_task(email_content)
    assistance_request_handling_task = tasks.assistance_request_handling_task(email_content)
    draft_email = tasks.draft_email(email_content, categorize_email, sql_task, review_handling_task, assistance_request_handling_task)

    crew = Crew(
        agents=[categorizer_agent, sql_agent, review_handling_agent, assistance_request_handling_agent, draft_email_writer_agent],
        tasks=[categorize_email, sql_task, review_handling_task, assistance_request_handling_task, draft_email],
        verbose=2,
        # process=Process.sequential,
        full_output=True,
        share_crew=False,
    )

   results = crew.kickoff()
   print(results)
   ```

## Contributing

Feel free to fork this repository and submit pull requests. For any issues or feature requests, please open an issue in the GitHub repository.


Feel free to adjust the specifics as needed for your project.
