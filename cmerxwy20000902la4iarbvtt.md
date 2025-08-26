---
title: "Implementing Azure AI Vision with Python: OCR, Object Detection, Tagging & Captioning"
datePublished: Tue Aug 26 2025 02:41:54 GMT+0000 (Coordinated Universal Time)
cuid: cmerxwy20000902la4iarbvtt
slug: implementing-azure-ai-vision-with-python-ocr-object-detection-tagging-and-captioning
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1756175988971/df3d8f82-a095-4a97-9db4-5922db308e7d.png
tags: azure, openai, azure-ai-services, llms

---

# Introduction

Computer Vision is a branch of Artificial Intelligence that allows machines to **see, analyze, and understand** images and videos. Azure AI Vision provides pre-built models through simple APIs, enabling developers to integrate features like object detection, OCR, tagging, and caption generation into their applications.

In this blog, we’ll walk through how to implement Azure AI Vision using **Python**, showcasing its major features with practical code examples.

# Azure AI Vision & Use Cases

Instead of spending time training deep learning models from scratch, Azure AI Vision provides ready-to-use models that can:

* Detect objects in images
    
* Extract text using **OCR**
    
* Automatically generate tags
    
* Create human-like captions describing images
    

This makes it extremely useful for building real-world AI solutions quickly and at scale.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1756175452965/a7951067-716b-4783-8ba6-eae8e7331280.png align="center")

## How It Helps LLMs

When combined with **Large Language Models (LLMs)**, these visual capabilities make AI systems **multimodal**:

* LLMs can reason not just on text, but also on what’s inside an image.
    
* Example: Azure Vision extracts text from an invoice (OCR) → LLM interprets and summarizes it → business system updates automatically.
    

## Real-World Use Case

Imagine a **retail company** managing thousands of product images:

* Azure Vision **tags and captions** products for search and cataloging.
    
* OCR extracts text from product labels.
    
* LLMs use this extracted data to auto-generate product descriptions for e-commerce.
    

This saves time, improves accuracy, and makes systems more intelligent.

# Python Implementation

## 1- Analyzing an Image with Azure AI Vision in Python (Image Tagging)

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import os
import json
from dotenv import load_dotenv

load_dotenv("env.env")

endpoint="https://az-computer-vision1.cognitiveservices.azure.com/"
key=os.getenv("AI_VISION_KEY")

client = ImageAnalysisClient(endpoint=endpoint, credential=AzureKeyCredential(key))
with open("ai_vision_test.jpg", "rb") as image:
    image_details=image.read()

response=client.analyze(
    image_data=image_details,
    visual_features=[VisualFeatures.TAGS, VisualFeatures.CAPTION]

)

print(json.dumps(response.as_dict(), indent=4))
```

### Code Walkthrough

1. **Importing Required Libraries**
    
    * [`azure.ai.vision`](http://azure.ai.vision)`.imageanalysis` provides the `ImageAnalysisClient` to interact with the Azure AI Vision service.
        
    * `VisualFeatures` defines which features (Tags, Captions, OCR, etc.) we want to extract.
        
    * `AzureKeyCredential` securely passes our API key to the client.
        
    * `dotenv` is used to safely load credentials stored in an `.env` file.
        
2. **Setting Up Authentication**
    
    * The **endpoint** is the URL of your Azure AI Vision resource.
        
    * The **key** is stored in an environment file (`env.env`) for security, instead of hardcoding it into the script.
        
3. **Creating the Client**
    
    * The `ImageAnalysisClient` connects to Azure’s AI Vision service using the endpoint and API key.
        
4. **Reading the Image**
    
    * The image (`ai_vision_test.jpg`) is read in **binary format** because the API expects raw bytes of the image.
        
5. **Analyzing the Image**
    
    * We call the `analyze()` method and pass two visual features:
        
        * `VisualFeatures.TAGS` → Returns a list of keywords that describe the image.
            
        * `VisualFeatures.CAPTION` → Generates a human-readable caption summarizing the image.
            
6. **Printing Results**
    
    * The response is converted into a dictionary and printed in a nicely formatted **JSON output** using `json.dumps()`.
        
    * This makes it easy to see what Azure Vision has detected in the image.
        

## 2- Object Detection with Azure AI Vision in Python

Another powerful feature of Azure AI Vision is **object detection**. It not only identifies what objects are present in an image but also returns their **locations using bounding boxes**. This is especially useful in applications like surveillance, manufacturing defect detection, and retail product recognition.

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import os
import json
from dotenv import load_dotenv

load_dotenv("env.env")

endpoint="https://az-computer-vision1.cognitiveservices.azure.com/"
key=os.getenv("AI_VISION_KEY")

client = ImageAnalysisClient(endpoint=endpoint, credential=AzureKeyCredential(key))
with open("fruit_bucket.png", "rb") as image:
    image_details=image.read()

response=client.analyze(
    image_data=image_details,
    visual_features=[VisualFeatures.OBJECTS]    # detects objects in the image with bounding boxes

)

print(json.dumps(response.as_dict(), indent=4))
```

### Code Walkthrough

1. **Importing Libraries**  
    We bring in the same core classes (`ImageAnalysisClient`, `VisualFeatures`, and `AzureKeyCredential`) used earlier for tagging and captions.
    
2. **Authentication**
    
    * The `endpoint` points to your Azure AI Vision resource.
        
    * The `key` is securely loaded from an `.env` file.
        
3. **Image Input**
    
    * We load the image `fruit_bucket.png` in binary format so it can be sent to the API.
        
4. **Object Detection Request**
    
    * `VisualFeatures.OBJECTS` tells Azure Vision to detect all visible objects in the image.
        
    * The API responds with a list of objects, their confidence scores, and bounding box coordinates.
        
5. **Readable Output**
    
    * The [`response.as`](http://response.as)`_dict()` method returns the structured result as a dictionary.
        
    * `json.dumps()` formats it neatly so we can see exactly what was detected.
        

## 3- Extracting Text from Images with Azure AI Vision (OCR)

Azure AI Vision provides powerful **OCR (Optical Character Recognition)** capabilities. With this feature, we can detect both **printed and handwritten text** from images and documents. This is especially useful in scenarios like digitizing scanned documents, extracting text from receipts, or reading quotes from images.

```python
from azure.ai.vision.imageanalysis import ImageAnalysisClient
from azure.ai.vision.imageanalysis.models import VisualFeatures
from azure.core.credentials import AzureKeyCredential
import os
import json
from dotenv import load_dotenv

load_dotenv("env.env")

endpoint="https://az-computer-vision1.cognitiveservices.azure.com/"
key=os.getenv("AI_VISION_KEY")

client = ImageAnalysisClient(endpoint=endpoint, credential=AzureKeyCredential(key))
with open("quote.jpg", "rb") as image:
    image_details=image.read()

response=client.analyze(
    image_data=image_details,
    visual_features=[VisualFeatures.READ]    # detects text in the image using Optical Character Recognition (OCR)

)

for line in response.read.blocks[0].lines:
    print(line. Text)
```

### Code Walkthrough

1. **Authentication**
    
    * The endpoint and key are loaded from the `.env` file to keep secrets secure.
        
    * `ImageAnalysisClient` is initialized to interact with Azure AI Vision.
        
2. **Reading the Image**
    
    * The file `quote.jpg` is loaded in binary mode before sending it to the API.
        
3. **Performing OCR**
    
    * The [`VisualFeatures.READ`](http://VisualFeatures.READ) option tells Azure Vision to run OCR.
        
    * The response contains detected text blocks, lines, and even word-level details if needed.
        
4. **Extracting Results**
    
    * We loop through [`response.read`](http://response.read)`.blocks[0].lines` and print each line of detected text.