<p align="center">
  <img src="caterpillar.svg" width="160" alt="Caterpillar" />
</p>

<h1 align="center">CAT</h1>

My learning log from building an LLM evaluation framework at work — notes on the
things I had to dig into along the way.

## Pass/Fail or Score?
I have an AI assistant that has answered to a question, I also have the ground truth answer for that question. In my evaluation framework I'm making a call to an LLM and ask it to compare these two answers to define if the assistant answer is 
acceptable or not. In my prompt should I ask the LLM to return a score and then consider answers with scores above a threshold as Pass or should I ask the LLM to just return Pass or Fail?

When we ask an LLM to return a score from 0 to 1. The returned number should **not be interpreted as a true continuous probability or distance**. LLMs generate tokens and unless you explicitly calibrate the evaluator, distinctions such as 0.71 
and 0.76 usually don't have a well-defined statistical meaning. In practice, models often behave as if they have a small number of latent categories such as:
  ```
    wrong -> partially wrong -> mostly correct → essentially equivalent → identical
  ```
and then map those categories onto numbers. A 0.79 vs 0.81 difference may be essentially noise.

In short:
  **A prompted 0-1 score is generally not a meaningful continuous measurement unless you calibrate it. If your downstream metric is pass/fail, ask the model directly for pass/fail. Keep continuous or ordinal scores only if you actually need ranking**

## Embedding vs LLM call
We are comparing two texts, AI assistant answer and ground truth, why not using embeddings and cosine similarity instead of an LLM call which is more costly and slower?

Two text embeddings may be really close (which means semantically) but mean completely the opposite for example:
  1. I need an oil break
  2. I don't need an oil break


## Duck DB
It is an analytical SQL database, similar to SQLite but optimized for analytics rather than transactional workloads. It is especially good when your data is stored as Parquet, CSV, JSON, or pandas, because you can query them directly without running a separate database server.

## Make the LLM evaluator behave more like a deterministic test suite than a probabilistic judge

### Temperature + Seed are about reproducibility

- **Temperature** controls how much randomness is applied when sampling the next token from the model's probability distribution. At temp=0, decoding is effectively greedy: the model tends to select the highest probability token each step
- **Seed** initializes the pseudo-random number generator used during sampling. A **seed** is just the starting value for the pseudo-random number generator used when the model samples tokens. Suppose the model assigns

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
Bernoulli data works naturally with a Beta distribution. Start with

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

Then we can answer to questions like "What's the probability that our true pass rate is above 80%?"

Or compare two prompt versions:

Prompt A: Beta(84, 18)
Prompt B: Beta(91, 11)


```py
a = beta.rvs(84, 18, size=100_000)
b = beta.rvs(91, 11, size=100_000)

probability_b_better = (b > a).mean()
```

you might get 0.94 meaning: There is approximately a 94% posterior probability that Prompt B has a higher pass rate than Prompt A.

## BLEU
**Bilingual Evaluation Understudy** is a metric for comparing a generated text with one or more reference texts. It was originally designed for machine translation.

The main idea is: **how many short word sequences in the generated answer also appear in the reference?**

Suppose:

- Reference: the cat is sitting on the mat
- Generated: the cat is on the mat

BLEU looks at n-grams:

- 1-grams: the, cat, is
- 2-grams: the cat, cat is, is on
- 3-grams: triples
- 4-grams: sequences of four words

  it calculates:

  p_n = matching n-grams / generated n-grams

Then BLEU combines these using a geometric mean, typically BLEU-4:  

  BLEU = BP * exp(1/4 sigma_(n=1,4)(log(p_n)))

### Geometric Mean
The important difference is geometric mean **punishes low values more strongly**

A normal arithmetic mean would be:

  (0.8 + 0.6 + 0.4 + 0.01)/4 = 0.4525

The geometric mean is:

  (0.8 * 0.6 * 0.4 * 0.01) to power of 1/4 = 0.21

This is useful for BLEU because we don't want a translation to get a good score just because individual words match. **You need to perform reasonably well on all n-gram levels; one very bad level should significantly hurt the final score.**

### Brevity Penalty
Without it a model could generate something extremely short like: "the cat" and get high precision because every word matches. So BLEU penalizes outputs that are much shorter than the reference

### Caveat
Two sentences may be semantically equivalent, but share very few n-grams and vice versa so if you want to know 
whether the sentences have the same meaning, BLEU is usually not a good primary metric
