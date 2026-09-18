# How Large Language Models Actually Work: From Your Prompt to Every Word They Write

> **Original publication:** Hashnode  
> **Role in this repository:** AI/LLM foundations — the mental model behind the later RAG, agents, evaluation, and LLM infrastructure work.

*A complete, plain-English mental model — from tokens to attention to the exact moment ChatGPT picks its next word.*

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/b467051f-0911-4035-8b8a-21e9a8546789.png)

When you type a question into ChatGPT and an answer appears a few seconds later, it's easy to imagine a tiny digital brain reading your words, thinking about them, and then writing back. That mental picture feels natural — but it isn't quite what's happening.

Underneath the friendly chat window, a large language model (LLM) is doing something far more mechanical, and far more repetitive, than "thinking" in the way we do. And the strange part is that the core trick behind it is almost embarrassingly simple:

> **An LLM generates text by repeatedly predicting one word — or piece of a word — at a time.**

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/75696747-eda0-42db-a034-eb8beb0cf296.png)

That's it. That's the whole party trick.

Which immediately raises a fair question: how can "just guess the next word" write working code, explain black holes, or hold a conversation? The honest answer is that the *guessing* part is simple, but everything happening **between reading your prompt and making that guess** is not. That's where the real machinery lives, and that's what this article is going to unpack, piece by piece, until the whole system clicks into place.

To keep things concrete, we'll follow one sentence the entire way through the pipeline:

> **"The cat sat on the mat because it was tired."**

We picked this sentence on purpose — it looks simple, but it quietly contains almost every idea we need: word order, ambiguity, and a tricky little word ("it") whose meaning depends entirely on context. By the end, you'll be able to trace the full journey:

**Your Prompt → Tokens → Embeddings → Attention → Transformer Layers → Probabilities → Next Word → Repeat**

Let's begin the journey.

## 1. What Is an LLM Actually Trying to Do?

Imagine you handed a friend this sentence and asked them to finish it:

> "The sky is ___"

Most people would say "blue" without even thinking hard. Your brain isn't randomly guessing — it's drawing on a lifetime of seeing that phrase completed the same way, over and over, in books, conversations, and everyday life.

An LLM does something surprisingly similar, just at a scale no human could match. Formally, it's estimating:

```text
P(next word | everything that came before it)
```

In plain English: *"Given everything I've read so far, what's the most likely next piece of text?"*

Now stretch that idea across **hundreds of billions of examples** during training — books, articles, code, conversations, forums, manuals — and ask the model to get a little better at this guessing game every single time. To get good at "predict what comes next," the model is quietly forced to learn grammar, facts, reasoning patterns, coding syntax, and the relationships between ideas. Nobody explicitly taught it that Paris is in France or that `SELECT * FROM` starts a database query — it picked those patterns up purely by getting really, really good at next-word prediction across an enormous amount of text.

That's the punch line of this whole article: **a shockingly simple training goal, applied at a massive scale, produces behavior that looks a lot like understanding.** The rest of this piece is about *how* the model turns that simple goal into something that can actually read your sentence and respond intelligently.

## 2. A Tiny Sentence That Contains (Almost) Everything

Let's look closer at our example:

> **"The cat sat on the mat because it was tired."**

Quick question: who was tired — the cat, or the mat?

You answered instantly: *the cat*. Obviously. A mat can't get tired.

Now try this twist:

> **"The cat chased the mouse because it was frightened."**

This time, "it" could reasonably be the mouse *or* the cat — the sentence is genuinely a little ambiguous, and you probably paused for half a second before deciding.

Notice what just happened: the word **"it"** never changed. What changed was the *surrounding context*, and that context is what let you (and eventually the model) figure out what "it" really means. This single observation is one of the most important ideas in modern AI:

> **A word's useful meaning depends heavily on the words around it.**

Somehow, a language model needs a mechanism that lets every word "look around" at its neighbors before deciding what it means in this particular sentence. That mechanism is called **attention**, and we'll get there soon. But first, we need to back up — because before a model can even think about meaning, it has to solve a much more basic problem: it can't read English at all.

## 3. Words Are Not What the Model Sees

Here's something that surprises a lot of people: an LLM never actually "sees" the sentence *"The cat sat on the mat."* the way you do. Neural networks don't understand raw text — they only understand numbers. So before anything else can happen, your sentence has to be chopped up and converted into a machine-friendly format. That first step is called **tokenization**.

A **tokenizer** breaks text into smaller chunks called **tokens**. A token might be:

* a whole common word (`"cat"`)
* a fragment of a longer or rarer word (`"tokeniz"` + `"ation"`)
* a punctuation mark (`"."`)
* a number
* a leading space, bundled with the word after it
* a special control symbol the model uses internally

For our sentence, a tokenizer might conceptually split it like this:

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/63e98463-3139-4d0f-bbe4-6444b55fb531.png)

Notice that spaces often get glued onto the *front* of the next word rather than floating on their own — that's a real quirk of how many modern tokenizers work. The key takeaway is simple but easy to forget:

> **A token is not the same thing as a word.**

Common words are often a single token. Rare, technical, or made-up words frequently get chopped into multiple smaller pieces. For example, a word like "tokenization" might become `"token"` + `"ization"` rather than staying whole, simply because the model's vocabulary doesn't have a dedicated slot for every possible word in every language — it's far more efficient to build long words out of smaller, reusable pieces, the same way you can build many different LEGO creations from the same box of bricks.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/89b3a1ee-2f4d-4393-b5f6-e55ee3c6b878.png)

## 4. Why Different Models Split Text Differently

Here's a natural follow-up question: does every AI model chop up text the same way? No — and this trips people up constantly.

Tokenization isn't a universal law of language; it's a *design choice* baked into each model. Different models are built with different:

* vocabulary sizes (some have 30,000 possible tokens, others have over 100,000)
* tokenizer algorithms (common approaches include Byte-Pair Encoding and SentencePiece)
* rules for handling spaces, punctuation, and capitalization
* special "reserved" tokens for things like the start or end of a message

Think of it like two shipping companies packing the exact same order. One company might box it up as three medium packages; another might use two larger ones. The *contents* are identical, but the *packaging* is different. In the same way, the same English sentence can turn into a different number of tokens, split at different points, depending on which model's tokenizer is doing the cutting.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/6c421ecf-bb6b-498a-b184-b4fa5644f1d8.png)

This has a practical consequence worth knowing: after tokenization, each token gets converted into a plain integer called a **token ID** — essentially, its address in that model's vocabulary list. A token ID like `481` means absolutely nothing on its own; it only makes sense *inside the specific vocabulary it belongs to*. Model A's token `481` and Model B's token `481` are very likely completely different pieces of text. It's a bit like how the same locker number means nothing until you know which building you're standing in.

> **Same sentence, different models → often a different number of tokens, and definitely different token IDs.**

## 5. From Numbers to Meaning: Embeddings

So the model now has a list of numbers — token IDs like `1523`, `481`, `9021`. But raw ID numbers are still useless on their own; `481` isn't "more meaningful" than `317` just because it's a bigger number. IDs are just addresses, like a mailbox number. What the model actually needs is something that captures *meaning*.

That's the job of an **embedding**.

Every token ID gets converted into a long list of decimal numbers — a **vector** — that has been learned during training. Something like:

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/4f553169-8288-47fc-bd74-d0b3e65c934d.png)

The best way to build intuition here is to imagine a giant map, except instead of two dimensions (north-south, east-west), this map has hundreds of dimensions. Every word in the model's vocabulary gets a "location" somewhere on that map, and — because of how the model was trained — **words that behave similarly in language end up located near each other.**

Picture a simplified, three-dimensional slice of that map:

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/43642a8c-4588-45ae-8d2f-ebaab84bdf53.png)

"Cat" and "dog" land close together because they show up in similar kinds of sentences — both get walked, fed, and petted. "tired" lands somewhere completely different, closer to "sat," because exercise related words tend to appear in exercise related contexts. Nobody manually programmed these rules; the model discovered them purely by noticing statistical patterns across enormous amounts of text.

This is why embeddings are so useful beyond just LLMs — they power search engines, recommendation systems, and "find similar items" features, all by measuring distance between points on this meaning-map.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/2a3093e4-732d-491f-9e30-7cbc026b335d.png)

> **🔍** Real embedding vectors for a modern LLM typically have somewhere between 768 and over 12,000 dimensions — nobody can visualize that directly, which is exactly why the 2D/3D "map" picture above is a simplification. No single dimension in that vector means one clean thing like "furriness" or "size" — meaning is spread out, or *distributed*, across the whole vector, and only becomes interpretable through combinations of many dimensions at once.

## 6. Why Word Order Matters

There's a catch, though. Embeddings capture what a word means in isolation, but not *where* it sits in the sentence. And word order changes everything:

> "The dog chased the cat."  
> "The cat chased the dog."

Exact same words. Exact same embeddings for each individual word. Completely different meaning. If the model only looked at *which* tokens are present, it would treat these two sentences as identical — which is obviously wrong.

So alongside each token's embedding, the model also needs to inject information about **position**: this is the 1st word, this is the 2nd word, and so on. Different architectures handle this in different technical ways — some use a separate "position" vector added directly onto the token's embedding, and many modern LLMs use a more elegant approach called **rotary positional embeddings (RoPE)**, which rotates the vector based on its position rather than simply adding numbers to it. The exact method varies model to model, but the underlying goal is always the same:

> **The model needs some built-in way to know not just *what* a token is, but *where* it sits in the sequence.**

Once each token carries both "what I am" and "where I am," the model has everything it needs to start the real work: figuring out how these tokens relate to each other.

## 7. The "Bank" Problem: Why Context Changes a Token's Meaning

Let's push the idea of context a little further with a classic example — the word **"bank."**

> Sentence 1: "I deposited money in the bank."  
> Sentence 2: "We sat on the bank of the river."

Same token. Same starting embedding. Totally different meaning depending on the sentence around it.

This exposes a real limitation: if the model just used the token's raw, generic embedding and never adjusted it, "bank" would mean the exact same thing in a finance sentence and a nature documentary — which is clearly wrong. What the model actually needs is a way to let each token's representation *shift* based on the specific sentence it's currently sitting inside.

That's precisely the problem attention was invented to solve, and it brings us to the real engine at the heart of every modern LLM.

## 8. What Is Attention?

Let's return to our earlier example, tweaked slightly for maximum ambiguity:

> **"The dog didn't chase the ball because it was tired."**

What does **"it"** refer to? Almost everyone reads this and instantly thinks: *the dog*. Balls don't get tired.

Somehow, your brain scanned the whole sentence, weighed each word's relevance to "it," and landed on "dog" as the most sensible match. That's essentially what **self-attention** does inside a Transformer — the "self" simply means the sentence is comparing its own words against each other, rather than looking at some outside source.

A useful mental question to ask for every single token is:

> **"Which other tokens in this sentence should I be paying attention to, in order to understand myself correctly?"**

Here's a helpful (if slightly whimsical) way to picture it: imagine every word in the sentence standing on a stage under its own spotlight.

```text
The   dog   didn't   chase   the   ball   because   it   was   tired
```

When it's "it"'s turn to be interpreted, its spotlight doesn't shine on every word equally. It shines *brightly* on "dog," a little on "ball," and barely at all on "because" or "didn't." Those different brightness levels are a rough analogy for **attention weights** — numbers that represent how much one token should factor in another token when building its contextual meaning.

```text
                    "it"
                     │
         ┌────────────┼────────────┐
         ▼            ▼            ▼
       "dog"        "ball"       "tired"
      (bright)      (dim)        (medium)
```

The model isn't literally shining lights, of course — but the intuition holds: **attention lets every token look across the whole sentence and pull in exactly the information it needs to understand itself correctly, right now, in this specific context.** This is exactly how the plain embedding for "it" gets transformed into a context-aware representation that effectively means *"the dog, specifically."*

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/79da7d7d-76f8-4aed-9561-b7349713afa6.png)

## 9. Query, Key, and Value: The Engine Behind Attention

To make this spotlight idea actually computable, Transformers rely on three vectors with slightly intimidating names: **Query, Key, and Value** — almost always abbreviated **Q, K, V**.

The cleanest way to build intuition here is a library analogy.

Imagine you walk into a library and tell the librarian: *"I need books about outer space."* That request is your **Query** — a description of what you're looking for.

Every book on the shelves has a label or summary describing its contents — its subject, genre, and keywords. Those descriptions are the books' **Keys**. The librarian compares your Query against every book's Key to figure out which books are actually relevant to your request.

Once a book is judged relevant, what you actually walk away with is its *contents* — the real information inside. That's the **Value**.

```text
QUERY   → "What am I looking for?"
KEY     → "What do I represent, so others can decide if I'm relevant?"
VALUE   → "What information do I actually contribute once I'm picked?"
```

Every single token in the sentence generates all three of these vectors for itself. When the token "it" wants to figure out what it means, it builds a Query. Every other token in the sentence offers up its Key so "it" can check how relevant each one is. The tokens with the best-matching Keys — in our example, "dog" — contribute their Values most strongly to the final, updated representation of "it."

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/0e5b873e-9047-49c2-a9e1-b49c333f6e74.png)

## 10. Why Do We Need Three Separate Vectors?

A reasonable question at this point: why not just use one all-purpose vector instead of three? The answer is that Q, K, and V are doing three genuinely different jobs, and forcing one vector to do all three would blur them together and weaken the whole mechanism.

Think about it from "it"'s perspective one more time:

* As a **Query**, "it" needs to express *what kind of information it's searching for* (something that resolves an ambiguous pronoun).
* As a **Key**, "it" needs to describe *what kind of information it offers to other tokens* that might be searching for something it has.
* As a **Value**, "it" needs to carry *the actual content* it would contribute if some other token decided it was relevant.

These are three different questions with three different answers, so the Transformer learns three separate transformations — one that's good at producing useful Queries, one that's good at producing useful Keys, and one that's good at producing useful Values. Each token's base representation gets multiplied through all three, generating its own Query, Key, and Value simultaneously.

```text
Token representation
        │
        ├──────► Query   (what am I looking for?)
        ├──────► Key     (how can others find me?)
        └──────► Value   (what do I contribute?)
```

Just remember the short version: **Q asks, K matches, V provides.**

> **🔍** Mathematically, each of these is a matrix multiplication: `Q = X·Wq`, `K = X·Wk`, `V = X·Wv`, where `X` is the token's input vector and `Wq`, `Wk`, `Wv` are separate weight matrices the model learns during training. To score how relevant one token's Key is to another token's Query, the model takes their dot product and scales it down by dividing by the square root of the Key vector's dimension — written as `√d_k` — which keeps the numbers in a stable range so the next step (Softmax, which we'll meet shortly) doesn't become overly extreme. Put together, this is the famous scaled dot-product attention formula:

```text
Attention(Q, K, V) = softmax( Q·Kᵀ / √d_k ) · V
```

You don't need to memorize this to understand attention conceptually — but it's worth recognizing, because you'll see it in essentially every paper and diagram about Transformers.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/ac211239-3220-4dda-be77-872c003e7415.png)

## 11. What Is a Transformer, Exactly?

Attention is powerful, but it's only one ingredient. The full architecture that wraps attention into a complete, trainable system is called a **Transformer**, and it's the backbone of essentially every modern LLM — GPT, Claude, Gemini, Llama, and the rest.

A single, simplified Transformer block looks roughly like this:

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/cc68d886-6d0b-446a-b127-598c8ee96a70.png)

Two extra pieces are worth briefly demystifying, since they show up in every serious diagram of this architecture:

* **The "Add" step (residual connections):** the model adds the block's *output* back to its *original input*, rather than fully replacing it. This gives information a shortcut path through the network, which turns out to make very deep models dramatically easier to train.
* **Normalize:** this rescales the numbers after each step so they stay in a healthy, stable range instead of exploding or shrinking to nothing as they pass through dozens of layers.

The **Feed-Forward Network** is a smaller neural network applied to each token individually (not across tokens like attention). If attention is where tokens *gather information from each other*, the feed-forward layer is where each token *processes* that gathered information on its own.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/547d4b5a-ea23-428c-b8a7-6681eb8c2609.png)

## 12. What Happens Inside Multiple Transformer Layers?

Here's a crucial detail we've glossed over so far: a real LLM doesn't run this block just once — it stacks **dozens of these Transformer blocks** on top of each other, and the output of one layer becomes the input to the next.

This stacking is what allows the model to build up increasingly abstract understanding, layer by layer — a bit like how you first learn letters, then words, then sentences, then arguments, and eventually can analyze an entire essay. Roughly speaking:

```text
Early layers   → basic patterns: spelling, nearby-word relationships, simple grammar
Middle layers  → phrase structure, local context, resolving small ambiguities
Later layers   → long-range relationships, abstract meaning, task-relevant reasoning
```

By the time our token "it" has passed through all of these stacked layers, it isn't just carrying its original generic embedding anymore. It has been repeatedly re-shaped by attention and feed-forward processing at every layer, absorbing more and more relevant context each time, until its final vector effectively encodes something much closer to *"the tired dog, specifically, right here in this sentence."*

That final, richly contextual vector — for the very last token in the sequence — is what the model uses to make its actual prediction. Which brings us to the moment of truth.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/0cc0abae-550b-4d98-b54c-a0b843d569cc.png)

## 13. From Vectors to Words: Logits and Softmax

After all those layers, the model has produced one final vector representing "everything relevant, given the whole sentence so far." But that vector is just a list of numbers — it still isn't a word. We need one more conversion step to turn it into an actual prediction.

First, that final vector gets multiplied through one more learned matrix — called the **output projection** — which produces one raw score for *every single token in the model's vocabulary*. If the vocabulary has 50,000 possible tokens, the model now has 50,000 raw numbers, one per candidate word. These raw, unnormalized scores are called **logits**.

Logits on their own are a bit unruly — they can be any positive or negative number, and they don't add up to anything meaningful. So the model runs them through a function called **Softmax**, which converts that pile of raw scores into a clean **probability distribution**: every value becomes a number between 0 and 1, and the whole set adds up to exactly 100%.

For our sentence, imagine the model is about to predict the word after *"The cat is..."* The logits and resulting probabilities might look something like:

```text
Word          Logit        Softmax Probability
sleeping       4.8               42%
hungry         3.9               21%
cute           3.2               14%
outside        2.1                6%
...            ...               ...
```

> **🔍** Softmax works by exponentiating every logit (so all values become positive, and larger logits become disproportionately larger) and then dividing each one by the sum of all the exponentiated values. In three lines of Python:

```python
import numpy as np
def softmax(logits):
    exp = np.exp(logits - np.max(logits))  # subtract max for numerical stability
    return exp / exp.sum()
```

That `- np.max(logits)` trick doesn't change the final answer mathematically — it just prevents the numbers from overflowing during the actual computation.

Once we have this clean probability distribution, the model is finally ready to do something with it: pick a word.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/ee57ad2c-6f08-4d94-97bd-8663e942f29f.png)

## 14. Autoregressive Generation: Writing One Word at a Time

Here's where the whole system reveals its simplest, and arguably strangest, secret: the model doesn't write your entire answer at once. It writes **one token at a time**, and after producing each one, it feeds *its own output* back in as new input before predicting the next token. This looping pattern is called **autoregressive generation** — "auto" because it's using its own prior output, "regressive" because it's stepping backward over that growing sequence each time.

Let's trace it through our example, starting from just "The cat":

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/40b4c9e8-f70b-4a77-9f31-51eb358f2a08.png)

At every single step, the entire process we just walked through — tokenize, embed, add position info, run through every Transformer layer, compute logits, apply Softmax — happens again from scratch, using the *updated* sequence as input. The model has no separate "memory" of having done this before; it simply re-reads the whole conversation (now one token longer) and predicts what comes next, over and over, until it decides to stop.

This is genuinely the entire generation engine behind ChatGPT, Claude, and every other modern LLM: a tight loop of *predict one token → append it → repeat*, dressed up to feel like a flowing conversation.

## 15. Why ChatGPT's Answer Appears Word by Word

This autoregressive loop also explains something you've probably noticed but maybe never questioned: why does ChatGPT's response appear gradually, word by word, instead of showing up all at once?

The honest answer is: because that's *literally how it's being produced*. The model isn't hiding a finished paragraph and slowly revealing it for dramatic effect — each word genuinely doesn't exist yet until its specific generation step completes. As soon as one token is predicted, the interface can display it to you immediately and simultaneously start computing the next one. This is called **streaming**, and it exists for a very practical reason: waiting for a 400-word answer to fully finish generating before showing you *anything* would feel painfully slow. Streaming lets you start reading within a second or two, while the rest of the response is still being computed behind the scenes, token by token, exactly as we described above.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/94e4a430-860b-466b-832a-bb355c0e439f.png)

## 16. Choosing the Next Word: Greedy Decoding vs. Sampling

We now have a probability distribution over every possible next token — but the model still has to actually *pick one*. This decision-making step is called **decoding**, and there's more than one strategy for it.

**Greedy decoding** is the simplest possible approach: always pick the single highest-probability token, every single time, no exceptions. Using our earlier example, greedy decoding would always choose "sleeping" (42%) over every other option, because it's ranked first. This produces very consistent, repeatable output — but it can also feel flat, repetitive, or overly predictable across longer passages, since the model never takes even a slightly less obvious path.

**Sampling**, on the other hand, treats those probabilities as genuine odds rather than a strict ranking. Instead of always taking the top choice, the model essentially rolls a weighted die: "sleeping" gets picked about 42% of the time, "hungry" about 21% of the time, "cute" about 14% of the time, and so on. Most of the time, this still lands on a very sensible word — but occasionally it introduces variety, which is part of why asking the same question twice can produce two differently worded (but both perfectly reasonable) answers.

Neither approach is objectively "better" — they're suited to different needs. Tasks like code generation or factual lookups often lean toward more deterministic decoding, while creative writing tends to benefit from a bit of sampling-driven variety.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/6263176f-2121-4486-a707-07a093424119.png)

## 17. Temperature: The Creativity Dial

Sampling has one more important control knob attached to it: **temperature**. You may have seen this setting in an API playground or developer tool, often ranging from something like 0 to 2.

Temperature adjusts how "sharp" or "flat" the probability distribution becomes *before* sampling happens. Here's the intuition:

* **Low temperature** (closer to 0) sharpens the distribution — the already-likely word becomes even more dominant, and the model behaves almost like greedy decoding: focused, predictable, conservative.
* **High temperature** (above 1) flattens the distribution — the gap between the top choice and the runner-ups shrinks, so less-obvious words get a genuinely fair shot at being picked, and output becomes noticeably more varied and, sometimes, more surprising or creative.

A subtlety worth being precise about, because it's commonly misunderstood: **temperature doesn't change what the model "knows."** The underlying probabilities the model computed from its training remain exactly the same — temperature only reshapes how those probabilities get used during the final sampling step. It's a dial on *randomness and expressive range*, not a dial on intelligence, accuracy, or knowledge.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/6d520edc-e91d-4d40-8bc1-5d971ef0a3f3.png)

## 18. Model vs. Decoding Strategy: Two Different Jobs

It's worth explicitly separating two ideas that people often blur together: the **model** and the **decoding strategy** are two completely different components doing two completely different jobs.

* The **model** (all those stacked Transformer layers) is entirely responsible for computing *how likely each possible next word is*, given everything so far. This is fixed once training finishes — the same prompt, sent to the same model, always produces the exact same underlying probability distribution.
* The **decoding strategy** (greedy, sampling, temperature, and a few more advanced techniques like top-k or nucleus sampling) is a separate, external process that decides *how those probabilities actually get converted into a chosen word*. This can be adjusted freely at the moment of generation, without retraining or changing the model at all.

A helpful analogy: the model is like a weather forecaster who calculates "70% chance of rain, 30% chance of sun." That calculation doesn't change. But *what you decide to do* with that forecast — always assume the most likely outcome, or occasionally gamble on the less likely one — is a completely separate decision layered on top. Same idea here: one AI model can produce wildly different-feeling output depending purely on which decoding strategy and temperature setting sit on top of it.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/9f95e50d-b5fe-4970-aa4b-53a0e930c93a.png)

## 19. Putting It All Together: The Full Journey

Let's zoom back out and walk the entire pipeline one final time, start to finish, using our sentence as the guide.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/f39f4ffa-c0cf-49c2-acd6-c5986ab9a307.png)

Here's a quick reference table tying every term back to its job:

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/e938769f-76fb-4fc8-b01f-68b9b34dff1f.png)

## Final Takeaway

If you remember only one paragraph from this entire piece, make it this one:

> **A large language model converts your text into tokens, turns those tokens into meaning-carrying vectors, repeatedly refines those vectors using self-attention and feed-forward processing across many stacked Transformer layers, converts the final result into a probability distribution over every possible next word, selects one word using a decoding strategy, and then repeats the entire process — one word at a time — until the response is complete.**

That's the machine. The experience feels like a conversation. Underneath, it's an extraordinarily large number of very fast, very repetitive numerical calculations, run one token at a time, at a scale no human brain performs consciously.

The genuinely surprising part isn't that an LLM predicts the next word — that's almost too simple to be impressive on its own. The surprising part is that a big enough neural network, trained on a big enough slice of human writing, can turn *"just predict the next word"* into something that can explain physics, debug code, and hold a coherent conversation with you.

And now, the next time someone says *"the AI generated an answer,"* you have the real mental model running quietly underneath that sentence:

**Prompt → Tokens → Embeddings → Attention → Transformer Layers → Probabilities → Next Word → Repeat.**

## References & Further Reading

**Core architecture**

1. Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). *Attention Is All You Need*. Advances in Neural Information Processing Systems 30 (NeurIPS 2017). [arxiv.org/abs/1706.03762](https://arxiv.org/abs/1706.03762)
2. Alammar, J. (2018). *The Illustrated Transformer* [Blog post]. [jalammar.github.io/illustrated-transformer](https://jalammar.github.io/illustrated-transformer/)
3. Alammar, J. (2019). *The Illustrated GPT-2 (Visualizing Transformer Language Models)* [Blog post]. [jalammar.github.io/illustrated-gpt2](https://jalammar.github.io/illustrated-gpt2/)

**Tokenization**

4. Sennrich, R., Haddow, A., & Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units* (the original Byte-Pair Encoding paper). [arxiv.org/abs/1508.07909](https://arxiv.org/abs/1508.07909)
5. Kudo, T., & Richardson, J. (2018). *SentencePiece: A Simple and Language Independent Subword Tokenizer and Detokenizer for Neural Text Processing*. [arxiv.org/abs/1808.06226](https://arxiv.org/abs/1808.06226)

**Positional encoding**

6. Su, J., Lu, Y., Pan, S., Wen, B., & Liu, Y. (2021). *RoFormer: Enhanced Transformer with Rotary Position Embedding*. [arxiv.org/abs/2104.09864](https://arxiv.org/abs/2104.09864)

**Scale and emergent behavior**

7. Brown, T. B., et al. (2020). *Language Models are Few-Shot Learners* (the GPT-3 paper). [arxiv.org/abs/2005.14165](https://arxiv.org/abs/2005.14165)

### Credits & Tools Used

* **Outline and editorial review:** Claude (Anthropic)
* **Illustrations:** generated with ChatGPT (OpenAI)
