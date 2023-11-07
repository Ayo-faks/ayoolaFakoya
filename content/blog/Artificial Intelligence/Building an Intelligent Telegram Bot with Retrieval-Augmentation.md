---
layout: Post
title: "Building an Intelligent Telegram Bot with Retrieval-Augmentation for Text Generation and Inference"
description: "Create an intelligent Telegram bot with retrieval augmentation using Transformers and Generative AI."
date: '2023-11-07'
featured: true
seo:
  title: "Building an Intelligent Telegram Bot with Retrieval-Augmentation for Text Generation and Inference"
  description: "Learn to create an intelligent Telegram bot with retrieval augmentation using Transformers and Generative AI."
  keywords:    
    - Retrieval augmentation
    - Text generation
    - Transformers
    - Generative AI
tags:
  - Retrieval augmentation
  - Text generation
  - Transformers
  - Generative AI
images:
  - src: /photos/telegramRag.jpeg
    alt: "Generative AI concept art"
---

# Building an Intelligent Telegram Bot with Retrieval-Augmentation and Large Language Models (LLMs)

## Introduction

In this tutorial, we'll guide you through the creation of a Telegram bot that leverages retrieval-augmentation techniques for text generation and answer inference. We'll emphasize the use of open-source libraries and the state-of-the-art open-source embedding model available at the time of writing this post. With a focus on practicality, we'll explain how to utilize NumPy to flatten vectors to enable us create memory . Let's begin.

### Step 1: Setting Up the Telegram Bot

Telegram bots are powerful tools for user interaction. To create your own bot, follow these steps:

1. **Open the Telegram App**: Ensure that you have the Telegram app installed.

2. **Find BotFather**: Search for "BotFather" within Telegram – this is the official bot for creating other bots.

3. **Start a Chat with BotFather**: Initiate a chat with BotFather and click the "START" button.

4. **Create a New Bot**: Send "/newbot" to BotFather to create a new bot. Follow the provided instructions to name and configure your bot. Ensure that your bot's username ends with "bot."

5. **Retrieve the Bot Token**: After successfully creating your bot, BotFather will send a message containing your bot's token. Save this token securely, as it is crucial for authenticating and communicating with your bot.

With your bot token in hand, let's proceed to the next steps.

### Step 2: Understanding the Libraries and the Embedding Model

In this tutorial, we'll harness the power of the following libraries and an exceptional open-source embedding model for text generation and answer inference:

#### Libraries Used:

- **Pandas**: A popular library for data manipulation and analysis, used for handling structured data.
- **NumPy**: A fundamental library for numerical operations, which we'll use to process and flatten vectors.
- **OpenAI**: A cutting-edge AI platform, which provides the means for sophisticated text generation and inference.
- **Transformers**: A popular library for NLP, which provides a high-level API for working with transformer models.

#### The Embedding Model:

We'll use the "thenlper/gte-base" model, known for its exceptional performance in natural language understanding tasks. This model excels at text embeddings, making it an excellent choice for our retrieval-augmented bot.

### Step 3: Preprocessing and Tokenization

Now, let's delve into the practical implementation of preprocessing and tokenization. These steps ensure that the text is cleaned and formatted for embedding and retrieval.

The provided Python code demonstrates the data preprocessing, tokenization, and chunking and embedding:

```python
import pandas as pd
import os
from dotenv import load_dotenv
from torch.nn.functional import normalize
from transformers import AutoTokenizer, AutoModel


# Load environment variables
load_dotenv()
DOMAIN = "developer.mozilla.org"

# Define a function to remove newlines and preprocess text
def remove_newlines(series):
    series = series.str.replace('\n', ' ')
    series = series.str.replace('\\n', ' ')
    series = series.str.replace('  ', ' ')
    series = series.str replace('  ', ' ')
    return series

# Create a list to store the text files
texts = []

# Get all the text files in the specified directory
for file in os.listdir("text/" + DOMAIN + "/"):
    with open("text/" + DOMAIN + "/" + file, "r", encoding="UTF-8") as f:
        text = f.read()
        filename = file[:-4].replace('_', '/')

        if filename.endswith(".txt") or 'users/fxa/login' in filename:
            continue

        texts.append((filename, text))

# Create a Pandas DataFrame from the collected text
df = pd.DataFrame(texts, columns=['fname', 'text'])

# Clean and preprocess the 'text' column
df['text'] = df.fname + ". " + remove_newlines(df.text)

# Save the DataFrame to a CSV file
df.to_csv('processed/scraped.csv')

# Load a tokenizer for text processing
tokenizer = AutoTokenizer.from_pretrained("thenlper/gte-base")

# Load the processed data from the CSV file
df = pd.read_csv('processed/scraped.csv', index_col=0)
df.columns = ['title', 'text']

# Split text into smaller chunks that fit within the maximum sequence length (e.g., 512 tokens)
max_sequence_length = 512
df['chunks'] = df.text.apply(lambda x: [x[i:i+max_sequence_length] for i in range(0, len(x), max_sequence_length])

# Initialize an empty list to store the embeddings
embeddings_list = []

# Initialize the model
model = AutoModel.from_pretrained("thenlper/gte-base")

# Process each batch of text and store the embeddings
batch_size = 1  # Adjust the batch size based on your resources
for index, row in df.iterrows():
    chunks = row['chunks']
    for i in range(0, len(chunks), batch_size):
        batch = chunks[i:i + batch_size]
        inputs = tokenizer(batch, return_tensors='pt', padding=True, truncation=True, max_length=max_sequence_length)
        outputs = model(**inputs)
        embeddings = outputs.last_hidden_state.mean(dim=1)  # You can use a pooling strategy here
        embeddings_list.append(embeddings)

# Concatenate the embeddings and convert to a tensor
embeddings_tensor = torch.cat(embeddings_list, dim=0)

# (Optionally) normalize embeddings
normalized_embeddings = normalize(embeddings_tensor, p=2, dim=1)

# Save the final DataFrame with text embeddings to a CSV file
df['embeddings'] = normalized_embeddings.tolist()
df.to_csv('processed/embeddings.csv')
```
We perform the following actions:

1. **Load Libraries and Environment Variables:** We start by importing the necessary Python libraries and loading environment variables using `dotenv`.

2. **Text Preprocessing Function:** We define a function named `remove_newlines` to clean and preprocess text data. This function removes newlines and extra spaces from the text.

3. **Data Collection and Cleaning:** We collect text data from specified files located in the 'text/developer.mozilla.org/' directory. The collected data is cleaned and preprocessed to prepare it for further processing.

4. **Tokenization:** We utilize a chosen embedding model, 'thenlper/gte-base,' to tokenize the text data. Tokenization is a crucial step in natural language processing (NLP) that converts text into numerical representations that can be processed by machine learning models.

5. **Text Chunking:** The code splits the text into smaller chunks to ensure it fits within the maximum sequence length of the chosen model (e.g., 512 tokens per sequence). This is essential to handle longer documents.

6. **Embedding Generation:** We generate embeddings for the text data using the chosen model. The embeddings represent the semantic information of the text and can be used for various NLP tasks.

7. **Embedding Normalization:** Optionally, we normalize the embeddings using L2 normalization. Normalization ensures that the embeddings have a consistent scale and can be easily compared or used in downstream tasks.

**Step 4: Retrieval Augmentation and Answer Inference**

To enhance the intelligence of your bot, we implement retrieval augmentation and pass the returned documents as context to the Open-ai API to generate answers to the user's question. Here, the 'question.py' script plays a crucial role:

```python
import numpy as np
import pandas as pd
import openai
import os
from dotenv import load_dotenv
from openai.embeddings_utils import distances_from_embeddings

# Load environment variables
load_dotenv()
openai.api_key = os.environ['OPENAI_API_KEY']

df = pd.read_csv('processed/embeddings.csv', index_col=0)
df['embeddings'] = df['embeddings'].apply(eval).apply(np.array)

# Create a context for a question by finding the most similar context from the dataframe
def create_context(question, df, max_len=1800, size="ada"):
    # Get the embeddings for the question
    q_embeddings = openai.Embedding.create(input=question, engine='text-embedding-ada-002')['data'][0]['embedding']
    
    # Get the distances from the embeddings
    df['distances'] = distances_from_embeddings(q_embeddings, df['embeddings'].values, distance_metric='cosine')
    
    returns = []
    cur_len = 0
    
    # Sort by distance and add the text to the context until the context is too long
    for i, row in df.sort_values('distances', ascending=True).iterrows():
        cur_len += row['n_tokens'] + 4
        if cur_len > max_len:
            break
        returns.append(row["text"])
    
    return "\n\n###\n\n".join(returns)

# Answer a question based on the most similar context from the dataframe texts
def answer_question(df, model="gpt-3.5-turbo", question="What is the meaning of life?", max_len=1800, size="ada", debug=False, max_tokens=1500, stop_sequence=None):
    context = create_context(question, df, max_len=max_len, size=size)
    
    # If debug, print the raw model response
    if debug:
        print("Context:\n" + context)
        print("\n\n")
    
    try:
        response = openai.ChatCompletion.create(
            model=model,
            messages=[{
                "role": "user",
                "content": f"I want you to Answer the question based on the context below, if you can, and if the question can't be answered based on the context, say 'I don't know'.\".\n\nContext: {context}\n\n---\n\nQuestion: {question}\nSource:\nAnswer:",
            }],
            temperature=0.5,
            max_tokens=max_tokens,
            top_p=0.1,
            frequency_penalty=0,
            presence_penalty=0,
            stop=stop_sequence,
        )
        return response['choices'][0]['message']['content']
    except Exception as e:
        print(e)
        return ""
```
In this step, the script accomplishes the following tasks:

1. **Library Import and Environment Variables:** The script begins by importing the necessary Python libraries and loading any required environment variables.

2. **Loading Embeddings DataFrame:** It loads the DataFrame containing the text embeddings obtained during the preprocessing step. These embeddings represent the semantic information of the text data and will be used for further analysis.

3. **Context Generation Function:** The script defines a function named 'create_context.' This function's purpose is to generate context for a given question. Context is crucial for understanding the question in the context of the available information.

4. **Question Embedding:** Utilizing OpenAI's 'thenlper/gte-base' model, the script computes embeddings for the user's question. This step converts the question into a numerical representation that can be compared to the text data embeddings.

5. **Relevance Measurement:** The script calculates the relevance of each context in the DataFrame to the user's question. This is typically done by measuring cosine distances or similarity scores between the question embeddings and the context embeddings.

6. **Context Selection:** After determining the most relevant context(s), the script sorts and combines these contexts to form a coherent context for the question. This ensures that the question is understood in the context of the relevant information.

7. **Answer Inference Function:** The script defines the 'answer_question' function. This function uses the generated context to instruct the model to provide answers to the user's question. The model leverages the context to provide contextually relevant answers.

Overall, this script enhances the intelligence of your bot by retrieving relevant information and inferring accurate answers to user queries, making it a valuable component for delivering meaningful responses.

## Step 5: Integration with the Telegram Bot

In 'main.py,' we bring it all together by integrating the functions from 'question.py' into a Telegram bot.
``` python 
import os
import logging
from telegram import Update
from telegram.ext import filters, ApplicationBuilder, ContextTypes, CommandHandler, MessageHandler
import pandas as pd
import numpy as np
from questions import answer_question
from dotenv import load_dotenv
import openai

# Load environment variables
load_dotenv()
openai.api_key = os.environ['OPENAI_API_KEY']
tg_bot_token = os.environ['TG_BOT_TOKEN']

df = pd.read_csv('processed/embeddings.csv', index_col=0)
df['embeddings'] = df['embeddings'].apply(eval).apply(np.array)

messages = [{
    "role": "system",
    "content": "You are a helpful assistant that answers questions."
}]

logging.basicConfig(
    format='%(asctime)s - %(name)s - %(levelname)s - %(message)s',
    level=logging.INFO)

# Handle the /question command
async def question(update: Update, context: ContextTypes.DEFAULT_TYPE):
    answer = answer_question(df, question=update.message.text)
    await context.bot.send_message(chat_id=update.effective_chat.id, text=answer)

# Handle the /start command
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await context.bot.send_message(chat_id=update.effective_chat.id, text="I'm a bot, please talk to me!")

if __name__ == '__main__':
    application = ApplicationBuilder().token(tg_bot_token).build()

    start_handler = CommandHandler('start', start)
    question_handler = CommandHandler('question', question)

    application.add_handler(question_handler)
    application.add_handler(start_handler)

    application.run_polling()
``` 
This code sets up your Telegram bot with the following features:

1. **Library Import and Environment Variables:** The script begins by importing the necessary libraries and loading any required environment variables.

2. **Bot Initialization:** It initializes the bot with your Telegram bot token, enabling communication with the Telegram platform.

3. **Command Handlers:** The script defines handlers for two commands, "/start" and "/question." These commands allow users to interact with the bot and initiate the question-answering process.

4. **Chat Interaction:** An interactive chat handler is implemented to facilitate user interactions with the bot. Users can communicate with the bot to ask questions or provide input.

5. **Answering User Queries:** The 'answer_question' function is called to respond to user queries. This function leverages the context established in the previous steps to provide contextually relevant answers.

By running 'main.py,' you create a Telegram bot capable of answering user questions based on the provided context. Users have the option to type "/question" to ask questions within the established context or directly ask questions without the "/question" command, which will engage the model to provide answers.

## Conclusion

In this tutorial, we've walked you through the process of building a retrieval-augmented Telegram bot using Large Language Models. We've emphasized the importance of open-source libraries and the exceptional performance of the 'thenlper/gte-base' model in natural language understanding.

Our next project will explore the world of cutting-edge open-source AI libraries. This adventure will involve leveraging Langchain to orchestrate models and agents, harnessing Unstructured for data ingestion and transformation, diving into ChromaDB for efficient storage of embeddings, and so much more. Get ready for an exciting journey into the realm of modern AI tools and techniques!

For the complete code, you can download the repository from [here](https://github.com/Ayo-faks/RAG-Telegram-Bot).
