---
title: I Built a Contract RAG App in a Week and It Won an Award
date: 2025-03-06
category: projects
tags:
  - AI
  - RAG
  - .NET
  - vector search
  - legal technology
---

During Kinaxis's annual corporate hackathon, our team had one week to solve a real problem: years of contracts were difficult to search, metadata was inconsistent, and answering a simple risk or compliance question could turn into a large manual review.

I built [FluKsAI](https://ironcladapp.com/resources/articles/winners-legal-ai-innovators-award-2025), an internal web application that lets people ask natural-language questions across the contract repository instead of opening documents one at a time.

The award entry lists Paul Mason, Director of IT Application Excellence, and me, then an Intern Developer in IT Application Excellence. I was the sole developer: I designed and implemented the complete application end to end.

FluKsAI later took [second place in Ironclad's 2025 Legal AI Innovators Award](https://ironcladapp.com/resources/articles/winners-legal-ai-innovators-award-2025). Not bad for something that started as a one-week hackathon build.

## The actual problem

The immediate legal use case involved reviewing approximately 1,500 active contracts for risk-related questions. The broader repository was much larger, roughly 40,000 contract records, with old documents, uneven metadata, and information that was technically stored but not practically searchable.

Keyword search is not enough when the question is something like:

```text
Which active agreements contain language related to this risk,
and what obligations or exceptions should we review?
```

The wording in a contract rarely matches the wording in the question. The system needed semantic retrieval, document context, and a model that could turn the retrieved text into a useful response.

## The pipeline

FluKsAI was a self-hosted .NET web application backed by SQL for contract records and application data. A Linux server handled document processing and stored chunked embeddings in a vector database.

The basic flow was:

```text
legacy contracts
    -> ingest and organize
    -> extract and chunk text
    -> generate embeddings
    -> store vectors and document references
    -> retrieve relevant chunks for a question
    -> send grounded context to the language model
    -> return an answer for review
```

That is retrieval-augmented generation, or RAG: use vector search to find the contract sections that are semantically close to the question, then give those sections to the model as context instead of asking it to answer from general training data.

The system used the OpenAI API for natural-language queries and was designed to work with custom and fine-tuned model workflows as the use cases evolved. The useful part was not bolting a chat box onto a database. It was building the ingestion, organization, chunking, retrieval, and model pipeline needed to make answers come from the contract corpus.

## Keeping it internal

Contracts are sensitive, so this could not be a public chatbot with a convenient upload button. The application was self-hosted, ran in a closed environment, and integrated with Okta SSO. SQL kept the structured repository while the vector database stored the embedding index used for semantic search.

The model API was one component in a larger controlled system. Authentication, data ingestion, access boundaries, retrieval quality, and repeatable processing mattered just as much as prompt wording.

## What I liked about it

FluKsAI turned an annoying data problem into a useful workflow. Legal users could search across contracts for audit, compliance, regulatory, business, and risk questions without manually opening every possible document.

The hardest work in practical AI is often everything around the model. Old files need to be found. Text needs to be extracted. Chunks need enough context to remain meaningful. Metadata needs to survive ingestion. Retrieval needs to return useful evidence rather than vaguely related paragraphs. Then the application still needs authentication, storage, deployment, and an interface normal people can use.

The award article says the project helped show a bunch of lawyers that AI is their friend. I will take that as a successful deployment metric.

Reference: [Meet the Winners of the 2025 Legal AI Innovators Award](https://ironcladapp.com/resources/articles/winners-legal-ai-innovators-award-2025), Ironclad, March 6, 2025.
