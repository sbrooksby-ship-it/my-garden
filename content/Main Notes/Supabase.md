2026-08-26 14:56

Status:

Tags: [[DP13]][[D5]][[Database]] 


# Supabase

Pros: 25 a month, extra 10 per project. Progres but allows vibe coding. 
8 GB disk size(database size) per project, 0.125 per GB after.
250 Egress(if users download files) .09 per GB
100 GB file storage(user uploads) .03 per gb after
Instantly gets Auto-Generated rest and graphql API's
easy integration into coolify if cedric goes that way
Open source
RLS (if front end hacked backend still blocks it)

100 gb can be for the audio, n8n can get audio there automated or we can go google apps script/ python webhook

Supabase gives us a production-ready database and secure audio file storage for $25/month while cutting our development time in half through auto-generated APIs. It handles security at the database level to protect confidential call reviews, and because it's open-source, we retain full control over our data with the option to self-host on Coolify whenever we choose.

If DevOps (e.g., Cedric) prefers to host our stack internally on Coolify down the line, we can migrate seamlessly without rewriting a single line of code.

AI integration: Vector is free and self hosted in DB. Storage about 100,000 transcripts per .6gb goes towards the 8gb

Vector options to turn data into vector **Local Python (`sentence-transformers`)**Free**100% In-House** (Data never leaves server)Easy (3 lines of Python code)  

|**Model**|**Cost (per 1M Tokens)**|**Best Used For**|
|---|---|---|
|**`text-embedding-004`**|**$0.10**|Standard text embeddings and fast semantic search|
|**`gemini-embedding-001`**|**$0.15**|Standard Gemini API embedding endpoint|
|**`gemini-embedding-2`**|**$0.20**|Higher-dimensional / multimodal embeddings|

**Key Details for Your Call Evaluation Tool**

- **Free Tier Available:** Through Google AI Studio, you can generate embeddings for free up to generous rate limits (up to 1,500 requests per minute) for testing and development.
    
- **Practical Cost for Transcripts:** One million tokens is roughly 750,000 words. Converting a typical 2,500-word call transcript into vectors costs less than **0.0003**—meaning you can embed roughly **3,000 to 4,000 full transcripts for $1.00**. 

Costs 25 a month for data base. Then $2.50-3.00 for vector integration would be total of 27-28 a month.
### References
[[Dept 13 Tasks]]
[[What is Vector generating]]
