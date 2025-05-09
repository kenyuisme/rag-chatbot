# rag-chatbot

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

To run, please include a .env file with the following variables:
QDRANT_URL
QDRANT_API_KEY
HUGGINGFACE_TOKEN

Please ensure that the "data" folder is in the same directory as the "chunking and upserting.ipynb" notebook.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

This repository is was created for Aparavi's New Hire Challenge for the role of Quality Assurance Engineer – AI Product Line
Specifically, the challenge is named Agentic RAG Chatbot Challenge with Crew AI, Chonkie, and DeepEval 
I'm not sure if the details of the challenge is confidential, so I will not be sharing them. The gist of it is to create an RAG chatbot and evaluate it using programmatic frameworks.

This file will document my approach, including the contents of the three notebooks and the reasoning behind my choices.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

chunking and upserting.ipynb:
chunking the text documents and upserting them in qdrant

ragchatbot.ipynb:
chatbot to query the qdrant database

chatbot eval.ipynb:
evaluating the chatbot based on 5 criterias

data:
3 documents were fed to the chatbot. One of which is my CV, the other two are essays that I wrote a few months ago. The three documents have very little in common.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

1. Chunking and Upserting

As per standard practice, documents of unstructured texts must be processed before they can be queried by an RAG chatbot. In other words, they must be sliced into smaller segments, known as chunking, followed by vectorization, which translates the text into numbers which the algorithm can use to compute the similarity of a sentence to another sentence.

To achieve this, I used Chonkie for chunking the documents due to its ease of use. I'm not sure if Qdrant provides its own vectorization (similar to what Weaviate has), but I chose to provide my own embedding/vectorization instead to have more control over the process flow.

I chose all-MiniLM-L6-v2 for the embedding as it is a relatively light and free model, as well as being very popular and commonly used. It is a model I have experience with, so I chose to stick with it.

I used Semantic Chunking as the documents were not structured in any predictable manner (e.g. every paragraph is a different profile, or each document focus on one topic). Instead, the two essays were written in natural prose and covering a variety of topics, albeit related topics. Semantic Chunking helps to ensure that the chunks are sliced sensibly, maintaining much of the context.

I originally intended to use Weaviate, but I struggled greatly in making weaviate store the semantic chunk "text" payload properly. I switched over to Qdrant as it is capable of multi-vector collections, allowing me to easily store the semantic chunks (which were lists of vectors).

I wanted to create a named vector collection, but it was too difficult for me to work with, so I eventually went with a more basic approach (which I named standard collection)

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

2. RAG chatbot

After our data is chunked, vectorized, and stored in Qdrant, the next step would be to query it using the same embedding as before, as well as a language model to tokenize the text and generate an output.

At this point, I started to struggle with hardware limitations, which greatly affected my choice of language models. I originally wanted to use Mistral or Llama, but my machine would crash either due to lack of GPU VRAM or CPU RAM. Phi-2 was able to run without issues, providing a result which I determined to be "good enough" for this proof of concept. Notably, the answers which it gave to my queries were relatively shallow, but that is to be expected given the smaller model.

The prompt I used (You are a helpful assistant. Answer the question using only the context provided) is quite generic. I kept it that way due to how different each of the documents in the data provided was. If the documents were more focused on a specific area (e.g. if they were all related to the hotel industry), a more tailored prompt could be used.

The reasoning for only retrieving 5 relevant chunks is also due to hardware limitations. In this case, I believe 5 chunks is plentiful, given the small size of the data provided to the model.

---------------------------------------------------------------------------------------------------------------------------------------------------------------------

3. Chatbot Evaluation

Lastly, we must evaluate our chatbot to see if it is performing well. To accomplish this, we must write a few example queries, manually choose the relevant context from the data provided to the query, and manually write out what the answer to the query should be, based on the chosen context. We can then compare the chatbot's chosen context and generated answer to that of a humans.

Likely the easiest way to do this comparison would be to simply a few people who were not involved in the project to judge between the two outputs without telling them one was machine generated. In this challenge, the instructions were to evaluate it using a framework such as Deep Eval (which I used).

Unfortunately, my machine could not run the chatbot eval notebook. I tried Llama and Mistral, which both crashed during initialization due to insufficient resources. I tried Qwen, which did not support a feature (the "schema" argument) needed by Deep Eval. In the end, I returned back to using Llama and followed the documentation as best I could. Fingers crossed that there are no bugs in the code!

The default model used by Deep Eval is OpenAI, which I shyed away from as I wanted to stick to free models.

As far as hyper parameters are concerned, I mostly stuck to common values popularly used. 

The metrics chosen are all based on the challenge given. Without being able to view the metrics myself, it is difficult to evaluate the model. With that said, we should aim for a model with relatively high precision, as an RAG chatbot should not given untrue answers, even if it comes at the cost of a slightly lower recall. It is more important that the chatbot does not provide incorrect information, rather than failing to answer a query which it should have the data to answer.

Similarly, the faithfulness metric is of the utmost important to avoid hallucination.
