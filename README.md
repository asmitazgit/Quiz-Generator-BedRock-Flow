**Quiz-Generator-BedROCK  Agent**

This project demonstrates how to build an AI workflow using the BedROCK. It leverages contextual information from documents stored in a vector database (Astra DB) and an LLM provider (NOVA) to dynamically generate multiple-choice quizzes.

**##Goal##**

The primary goal of this project is to:

Ingest domain-specific documents.

Use them as contextual knowledge for an LLM prompt.

Automatically generate a set of MCQs (Multiple Choice Questions) as a quiz.

Demonstrate an AI workflow where external knowledge retrieval, reasoning, and LLM inference are combined seamlessly.

**Solution Overview**

Document Ingestion: User-provided documents are uploaded and embedded into Astra Vector DB for efficient similarity search.

Query & Context Retrieval: At runtime, the system retrieves relevant document chunks based on the input query.

Prompt Assembly: The retrieved context is combined with the user query to form the final prompt.

LLM Inference: The prompt is sent to an LLM(NOVA Model) which is hosted via BedROCK Agent, which generates a structured set of MCQs.

Output Delivery: The generated quiz is returned in a human-readable format.

User → Bedrock Flow → Lambda Function → AstraDB → Bedrock Agent → Foundation Model → Quiz Output

### 🔹 How It Works
1. The **user** provides a topic, difficulty, and number of questions.
2. The **Bedrock Flow** triggers a **Lambda function** (via Action Group).
3. The Lambda queries **AstraDB** (using the DataStax REST API) to retrieve context or study material.
4. The retrieved text is sent back to the **Bedrock Agent**.
5. The Agent prompts a **Foundation Model** (NOVA Model or Claude or Tital depends on which LLM model u have selected) to generate a multiple-choice quiz.
6. The Flow outputs a structured JSON quiz result.

---

## 📂 Repository Structure
.
├── flows/
│ └── quiz-flow.json # Exported Bedrock Flow definition
├── lambda/
│ ├── handler.py # Lambda function querying AstraDB
│ ├── requirements.txt
│ └── template.yaml # (Optional) SAM template for Lambda deployment
├── README.md

---

## ⚙️ Setup Instructions

### 1. Prerequisites
- AWS account with **Amazon Bedrock** enabled.
- **AstraDB** account and keyspace created.
- AWS CLI or CloudShell installed.
- IAM permissions for:
  - `bedrock:*`
  - `lambda:*`
  - `iam:PassRole`

---

### 2. Configure AstraDB
1. Log in to [AstraDB](https://astra.datastax.com/).
2. Create a **database** and note:
   - Database ID
   - Keyspace name
   - Application Token (with API access)
3. Create a collection or table for your knowledge base:
  
4.	Insert some learning content relevant to quiz topics.
________________________________________
3. Create Lambda Function
lambda/handler.py
import json
import requests
import os

ASTRA_DB_URL = os.environ["ASTRA_DB_URL"]
ASTRA_DB_TOKEN = os.environ["ASTRA_DB_TOKEN"]

def lambda_handler(event, context):
    topic = event.get("topic", "general knowledge")
    difficulty = event.get("difficulty", "medium")

    headers = {
        "x-cassandra-token": ASTRA_DB_TOKEN,
        "Content-Type": "application/json"
    }

    query = {"find": {"topic": topic}}
    response = requests.post(f"{ASTRA_DB_URL}/api/rest/v2/keyspaces/quiz_keyspace/quiz_knowledge/find", 
                             headers=headers, json=query)
    data = response.json()
    content = data["data"][0]["content"] if data["data"] else "No content found."

    return {
        "statusCode": 200,
        "body": json.dumps({
            "topic": topic,
            "difficulty": difficulty,
            "context": content
        })
    }
Lambda Configuration
Set environment variables:
•	ASTRA_DB_URL
•	ASTRA_DB_TOKEN
Deploy via AWS Console or SAM:
sam build && sam deploy
________________________________________
4. Create Bedrock Agent
1.	Go to Amazon Bedrock → Agents → Create Agent.
2.	Select a Foundation Model (e.g., Claude 3 Sonnet or Titan Text).
3.	Define the prompt template:
4.	You are a Quiz Generator Agent.
5.	Generate {question_count} multiple-choice questions from the provided context.
6.	Topic: {topic}
7.	Difficulty: {difficulty}
8.	Include four options per question and indicate the correct one.
9.	Add an Action Group → Connect the Lambda function created above.
o	Input schema:
o	{
o	  "topic": "string",
o	  "difficulty": "string"
o	}
10.	Publish a version and create an alias (e.g., quiz-agent-v1).
________________________________________
5. Import Bedrock Flow
1.	Go to Amazon Bedrock → Flows → Create Flow → Import from file.
2.	Upload flows/quiz-flow.json from this repo.
3.	Configure the Flow nodes:
o	Agent Node → select Quiz-Generator-Agent
o	Agent Alias → select quiz-agent-v1
4.	Save and test the flow.
________________________________________
🧾 Example Usage
Input:
Create a 5-question quiz on “Machine Learning” with medium difficulty.
Lambda Output (to Agent):
{
  "topic": "Machine Learning",
  "difficulty": "medium",
  "context": "Machine learning enables systems to learn from data..."
}
Final Quiz Output:
{
  "quiz": [
    {
      "question": "What does supervised learning use for training?",
      "options": ["Unlabeled data", "Labeled data", "No data", "Reinforcement signals"],
      "answer": "Labeled data"
    },
    {
      "question": "Which algorithm is used for classification?",
      "options": ["K-Means", "Linear Regression", "Decision Tree", "PCA"],
      "answer": "Decision Tree"
    }
  ]
}


**ACTUAL WORKFLOW REFERENCE**

 <img width="940" height="453" alt="image" src="https://github.com/user-attachments/assets/4fb12698-ff30-455f-9639-7d924b4ddb4f" />

 


 

 

 
 


 
 Here is a quiz on the role of Calcium:

1. What is the primary role of calcium in the body?
   a) Building bones and teeth
   b) Regulating muscle contractions
   c) Both a and b
   d) None of the above

2. Which of the following is a primary source of calcium?
   a) Meat
   b) Fruits
   c) Dairy products
   d) Vegetables

3. How does calcium help in blood clotting?
   a) By activating clotting factors
   b) By forming a clot
   c) By stopping bleeding
   d) By none of the above

4. What is the recommended daily intake of calcium for adults?
   a) 1000 mg
   b) 1200 mg
   c) 1500 mg
   d) 2000 mg

5. Which of the following is a sign of calcium deficiency?
   a) Brittle nails
   b) Muscle cramps
   c) Fatigue
   d) All of the above

6. What is the role of calcium in nerve function?
   a) Transmitting nerve impulses
   b) Regulating nerve activity
   c) Both a and b
   d) None of the above

7. How does calcium help in maintaining a healthy weight?
   a) By reducing appetite
   b) By increasing metabolism
   c) Both a and b
   d) None of the above

8. What is the role of calcium in bone health?
   a) Building and maintaining bones
   b) Preventing osteoporosis
   c) Both a and b
   d) None of the above

9. Which of the following is a risk factor for calcium deficiency?
   a) Age
   b) Gender
   c) Diet
   d) All of the above

10. What is the role of calcium in muscle function?
    a) Contracting and relaxing muscles
    b) Regulating muscle activity
    c) Both a and b
    d) None of the above

Answers: 1) c, 2) c, 3) a, 4) a, 5) d, 6) c, 7) c, 8) c, 9) d, 10) c
 

 
________________________________________
🔧 Export Flow JSON (for GitHub Versioning)
Run from AWS CloudShell:
aws bedrock-agent get-flow \
  --flow-identifier <FLOW_ID> \
  --region us-east-1 \
  > flows/quiz-flow.json
Commit to GitHub:
git add flows/quiz-flow.json
git commit -m "Export latest Bedrock Quiz Flow"
git push

