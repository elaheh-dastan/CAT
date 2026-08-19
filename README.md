# CAT
my learning in Caterpillar 

I work on evaluation framework and these are the points in my job I had to to a bit of search 

## Pass/Fail or Score?
I have an AI assistant that has answered to a question, I also have the ground truth answer for that question. In my evaluation framework I'm making a call to an LLM and ask it to compare these two answers two define if the assistant answer is 
acceptable or not. In my prompt should I ask the LLM to return a score and then consider answers with scores above a threshold as Pass or should I ask the LLM to just return Pass or Fail?

When we ask an LLM to return a score from 0 to 1. The returned number should **not be interpreted as a true continuous probability or distance**. LLMs generate tokens and unless you explicitly calibrate the evaluator, distinctions such as 0.71 
and 0.76 usually don't have a well-defined statistical meaning. In practice, models often behave as if they have a small number of latent categories such as:
  ```
    wrong -> partially wrong -> mostly correct → essentially equivalent → identical
  ```
and then map those categories onto numbers. A 0.79 vs 0.81 difference may be essentially noise.

In short:
  **A prompted 0-1 score is generally not a meaningful continuous measurment unless you calibrate it. If your downstream metric is pass/fail, ask the model directly for pass/fail. Keep continuous or ordinal scores only if you actually need ranking**

## Embedding vs LLM call
We are comparing two texts, AI assistant answer and ground truth, why not using embeddings and cosine similarity instead of an LLM call which is more costly and slower?

Two text embeddings may be really close (which means semantically) but mean completely the opposite for example:
  1. I need an oil break
  2. I don't need an oil break


## Duck DB
It is an analytical SQL database, similar to SQLite but optimized for analytics rather than transactional workloads. It is especially good when your data is sotred as Parquet, CSV, JSON, or pandas, because you can query them directly without running a separate database server.
