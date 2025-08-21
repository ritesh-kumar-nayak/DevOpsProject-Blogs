---
title: "Exploring the Role of Retrieval-Augmented Generation (RAG) in Modern AI"
datePublished: Thu Aug 21 2025 14:48:13 GMT+0000 (Coordinated Universal Time)
cuid: cmelinqig000202jt371rdk28
slug: exploring-the-role-of-retrieval-augmented-generation-rag-in-modern-ai
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1755787059235/90a9f99d-8afc-4b3b-b1d6-5a9ab3e2481e.png
tags: ai, ai-tools, ai-agents, azure-openai, azure-ai-102

---

# What is RAG?

Retrieval-Augmented Generation (RAG) pairs a Large Language Model (LLM) with external knowledge sources—such as databases, blob/object stores, file shares, and document repositories—to ground its responses in real data. Rather than relying solely on its pretraining, the LLM first retrieves relevant context from these connected sources and then generates an answer that blends that up-to-date evidence with its language capabilities. This approach improves factual accuracy, reduces hallucinations, and lets applications incorporate private or domain-specific knowledge without retraining the model.

# Sources for Retrieval Augmented Generation in Azure

In Azure, you can connect multiple enterprise data sources to power RAG with current, organization-specific knowledge. Common sources include:

* Azure Blob Storage: documents, PDFs, text files, and other unstructured content
    
* SharePoint: internal company docs, policies, and knowledge-base articles
    
* File shares: content from network drives and on-prem servers • Databases: SQL databases, Azure Cosmos DB, and more
    
* Web and APIs: company intranet, public sites, and custom REST endpoints
    
* OneDrive: Office documents such as Word, Excel, and PowerPoint
    

These sources can be ingested and indexed in Azure AI Search, which then serves as the retrieval layer for Retrieval-Augmented Generation—grounding your LLM with authoritative, up-to-date enterprise data.

# Internal flow of RAG with Azure

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1755785870720/e79dc655-344a-469d-96d3-b34ae99ac9c9.png align="center")

1. User query: Example: “What is our company’s travel reimbursement policy?”
    
2. Route to search: The LLM forwards the query to Azure AI Search (formerly Cognitive Search) rather than guessing from pretraining alone.
    
3. Retrieve from enterprise index: Azure AI Search scans its index built from enterprise sources (Blob Storage, SharePoint, file shares, databases, websites/APIs, OneDrive).
    
4. Vector and semantic retrieval: It fetches the most relevant chunks using vector search and semantic ranking (beyond simple keyword matching).
    
5. Ground the model: The retrieved passages are passed back to the LLM as context (grounding data).
    
6. Generate the answer: The LLM composes a final, natural-language response that cites and synthesizes the retrieved context with its reasoning.
    

Result: Answers are accurate, current, and aligned with company policy—while minimizing hallucinations and avoiding retraining.

# RAG implementation in Python with Azure services

## What you’ll build

A Retrieval-Augmented Generation (RAG) app where an LLM (GPT‑4) answers questions grounded in your enterprise content indexed by Azure AI Search and stored in Azure Blob Storage.

## Resources required on Azure

1. LLM: GPT‑4 deployment in Azure AI Foundry (Azure OpenAI)
    
2. Retrieval: Azure AI Search with vector search enabled
    
3. Content store: Azure Blob Storage (one container with your documents: PDFs, DOCX, TXT, etc.)
    
4. App runtime: Python (requests/openai/azure-openai SDKs) and a .env file for secrets
    

## High-level architecture

1. Documents live in Azure Blob Storage.
    
2. Azure AI Search ingests and indexes them (including embeddings for vector search).
    
3. Your Python app sends a user query to GPT‑4 and passes RAG config that points to the AI Search index.
    
4. Azure AI Search returns the most relevant chunks (vector + semantic retrieval).
    
5. GPT‑4 generates an answer grounded in those chunks.
    

## Setup checklist

* Create a Blob Storage account and container; upload sample docs.
    
* Provision Azure AI Search; enable vector search; create index (schema with content, metadata, vector fields).
    
* Connect the blob container to AI Search (data source + indexer) so content is ingested and chunked.
    
* In Azure AI Foundry, deploy a GPT‑4 model and note the endpoint, API version, and key.
    
* Capture keys/URLs in env.env: AZURE\_OPENAI\_API\_KEY, AZURE\_SEARCH\_API\_KEY, Azure OpenAI endpoint, Azure AI Search endpoint, Index name
    
* In Python, load env.env, initialize the Azure OpenAI client, and attach the AI Search data source via extra\_body (as shown in the snippet).
    

# Code Snippet

```python
import os
import json
from openai import AzureOpenAI
from dotenv import load_dotenv # Load environment variables from .env file

load_dotenv("env.env")  # Load environment variables from the specified .env file

subscription_key = os.getenv("AZURE_OPENAI_API_KEY")
client = AzureOpenAI(
    api_version="2024-12-01-preview",
    azure_endpoint="https://ai-foundary-demo.cognitiveservices.azure.com/",
    api_key=subscription_key,
)

rag_parameters = {
    "data_sources": [
        {
            "type": "azure_search",
            "parameters": {
                "endpoint": "https://aisearch20251999.search.windows.net",
                "index_name": "index",
                "authentication": {
                    "type": "api_key",
                    "key": os.getenv("AZURE_SEARCH_API_KEY"),
                }

            }
        }
    ],

}

response = client.chat.completions.create(
    messages=[
        {
            "role": "system",
            "content": "You are a helpful assistant that helps student learn Python Basics."
        },
        {"role": "user",
         "content": "What is Python?"
         
         }
    ],
    temperature=0.7,
    top_p=0.9,
    model="gpt-4.1",
    extra_body=rag_parameters
)

model_dump_response=response.model_dump()

print(response.choices[0].message.content)  # Print the response in a readable format
```