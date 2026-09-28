# Chat with PDF (RAG)

Upload a PDF, ask questions about it, get answers grounded in the actual document — instead of the AI guessing or hallucinating.

This is a small backend project I built to understand how Retrieval-Augmented Generation (RAG) actually works under the hood, rather than just calling a library that does it for me.

## How it works

When you upload a PDF and ask a question, here's what happens behind the scenes:

1. The PDF's text is extracted
2. That text is split into chunks (by paragraph)
3. Each chunk is converted into a vector embedding (a numerical representation of its meaning) using Gemini
4. Those vectors are stored in Qdrant, a vector database
5. Your question is also converted into a vector
6. Qdrant finds the chunk whose meaning is closest to your question
7. That chunk + your question are sent to Gemini, which writes the final answer

So instead of dumping the whole PDF into the AI's context every time (slow, expensive, and limited by token size), only the relevant part of the document gets used.

## Stack

- Node.js + Express
- Multer (file uploads)
- pdf-parse (text extraction)
- Google Gemini API — embeddings + text generation
- Qdrant — vector database

## Running it locally

**1. Install dependencies**

```bash
npm install
```

**2. Set up your environment variables**

Copy `.env.example` to `.env` and fill in your own keys:

```
GEMINI_API_KEY=
QDRANT_URL=
QDRANT_API_KEY=
```

Gemini keys are free from [Google AI Studio](https://aistudio.google.com/). Qdrant offers a free cloud cluster at [cloud.qdrant.io](https://cloud.qdrant.io/).

**3. Start the server**

```bash
node index.js
```

**4. Create the vector collection (once)**

Visit `http://localhost:3000/create-collection` in your browser.

**5. Upload a PDF and ask something**

`POST` to `http://localhost:3000/upload` as `form-data`:

| key | type | value |
|---|---|---|
| pdf | File | your PDF |
| question | Text | your question |

You'll get a plain-text answer back, generated from the relevant part of your document.

## What I learned building this

- Why fixed-size text chunking breaks sentences mid-thought, and why splitting by paragraph works better
- What an embedding actually is, and why "similar meaning → similar vectors" is the whole trick behind semantic search
- Why keyword search fails when the question and the document use different words for the same idea
- Why production systems move similarity search into a dedicated vector database instead of comparing vectors manually in a loop

## Limitations (for now)

- Only handles one PDF at a time — uploading a new one overwrites the old vectors
- Retrieves just the single best-matching chunk, not multiple
- No conversation memory between questions

## Author

Aryan