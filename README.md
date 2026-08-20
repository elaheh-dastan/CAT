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

## Make the LLM evaluator behave more like a deterministic test suite than a probabilistic judge

### Temperature + Seed are about reproducibility

- **Temperature** controls how much randomness is applied when sampling the next token from the model's probability distribution. At temp=0, decoding is effectively greedy: the model tends to select the highest probability token each step
- **Seed** initializes the pseudo-random number generator used during sampling. A **seed** is just the starting value for the pseudo-ransom number generator used when the model samples tokens. Suppose the model assigns

PASS = 0.7

FAIL = 0.3

If sampling is enabled the system generates a pseudo random number to decide which token to pick. For example:

random number = 0.21 → PASS

random number = 0.84 → FAIL

## Eval lock file
Suppose an engineer changes the agent prompt, but 950 out of your 1,000 test cases produce exactly the same candidate answers. You can reuse the cached/locked results for unchanged evaluations and judge only the changed ones

## Treating each data set item as a Bernoulli trial allows for flexible aggregation methods, including Bayesian updating or Monte Carlo sampling.

Suppose you evaluate 100 dataset items and get:

83 Pass

17 Fail

### Bayesian Updating
Bernoulli data works naturally with a Beta distribution. start with

p ~ Beta(1,1)

which says, "I don't know the true pass rate yet"

After observation

p | data ~ Beta(84, 18)

Now instead of saying pass rate is 83%. We can reason about the uncertainty around that number. And when you get another batch of evaluations, you don't need to start over. You update the existing Beta distribution with the new passes and failures. 

### Monte Carlo
Now we have a probability distribution, we can randomly sample from

```py
p_samples = beta.rvs(84, 18, size=100_000)
```
