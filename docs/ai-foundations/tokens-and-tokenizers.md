# Tokens & Tokenizers: How a Computer "Reads" a Sentence 

Here's a weird fact: when you read the sentence *"I love pizza"*, your brain doesn't see letters one by one — `I`, then space, then `l`, `o`, `v`, `e`... You just *see the words*. Your brain has already chopped the sentence into meaningful chunks without you noticing.

AI language models do something surprisingly similar. Before an AI can think about a sentence, it has to chop it into pieces called **tokens**. This blog is going to open up that machine and show you exactly how it works — with real, runnable code, a bit of history, and (fair warning) a few "wait, that's actually kind of genius" moments.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/91f1ce63-b591-4d3b-9253-b35e6c6529ea.png)

## 1\. Meet Our Star Sentence 🌟

We're going to use **one sentence** as our guinea pig for this discussion. Here it is:

> **"Although the young scientist was tired, she couldn't wait to unbelievably transform her old bicycle into a self-driving robot before the science fair started tomorrow."**

It's a good test subject because it has:

*   A long, "fancy" word (`unbelievably`)
    
*   A hyphenated word (`self-driving`)
    
*   A contraction (`couldn't`)
    
*   Punctuation (`,` and `.`)
    
*   Common everyday words (`the`, `was`, `her`)
    

By the end of this post, you'll know exactly how an AI would slice this sentence apart — and *why* it slices it that way.

**🤔 Quick question before we start:** if you had to guess, how many "pieces" do you think this sentence breaks into? Fewer than the number of words (25)? More? Keep your guess in your head — we'll check it later.

## 2\. What Even *Is* a Token?

A **token** is just a chunk of text that an AI model treats as one unit. It could be:

*   A whole word (`science`)
    
*   Part of a word (`token` + `ization`)
    
*   A single character (`x`)
    
*   Even a punctuation mark (`,`)
    

Think of tokens like **LEGO bricks**. You can't build a spaceship out of sand (too fine, too many grains, no shape) and you can't build it out of three giant foam blocks either (too chunky, not enough detail). LEGO bricks are the *sweet spot* — small enough to build anything, big enough that you're not fiddling with individual plastic molecules.

Tokens are the AI's LEGO bricks for language.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/490840b8-746d-4a22-8229-ca6f7f17035b.png)

## 3\. Why Isn't Every *Character* a Token?

This is a great question, and the answer comes down to **efficiency** and **meaning**.

Imagine reading a book where you had to process one letter at a time: `T`\-`h`\-`e`\- \-`c`\-`a`\-`t`\- \-`s`\-`a`\-`t`... It technically works, but it's painfully slow, and single letters don't carry much *meaning* on their own. What does `c` mean by itself? Nothing much. But `cat` means something immediately.

Here's the trade-off, spelled out:

| Splitting by... | Vocabulary size needed | Sequence length | Meaning per token |
| --- | --- | --- | --- |
| **Characters** | Tiny (~100 symbols) | Very long | Very low |
| **Words** | Huge (hundreds of thousands) | Short | Very high |
| **Subwords (tokens)** | Medium (30,000–100,000) | Medium | Medium-high ✅ |

Why does sequence *length* matter so much? Because of how transformer models (the tech behind ChatGPT, Claude, etc.) work internally — every token has to "look at" every other token to figure out context. If you double the number of tokens, the amount of computation can go up by roughly 4x (it's a quadratic relationship). So character-level tokenization would make AI models painfully slow and expensive to run.

And splitting by *whole words* has the opposite problem: English alone has hundreds of thousands of words, plus names, typos, slang, and words in other languages. A model would need a gigantic dictionary, and it would still panic every time it saw a word it had never seen before (like `rizzlorious` or a name like `Xiomara`).

**Subword tokens are the compromise**: common words stay whole (`the`, `science`), while rare or complex words get split into meaningful chunks (`tokenization` → `token` + `ization`).

**🤔 Think about it:** Why do you think a made-up word like `"blorptastic"` would still be understandable to a tokenizer, even though it's never seen that exact word before?

*(Answer: because it can break it into familiar pieces it HAS seen — like* `blorp` *+* `tastic` *— the same way you understand "fantastic" and can guess* `blorptastic` *follows the same pattern!)*

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/16623390-0486-4f4d-a35d-98e85d659fba.png)

## 4\. Why Aren't Tokens a Fixed Length?

This trips a lot of people up. If tokens are "medium-sized chunks," why is `the` one token, but a longer word like `tokenization` gets split into two?

The answer: **tokenizers are trained on huge amounts of real text, and they learn to keep whatever chunks show up together *most often* as single tokens.**

This is the core idea behind an algorithm called **Byte Pair Encoding (BPE)** — one of the most popular tokenizer training methods. Here's the recipe:

1.  Start by treating every **character** as its own token.
    
2.  Look at your entire training text and find the **pair of tokens that appears next to each other most often**.
    
3.  **Merge** that pair into a single new token.
    
4.  Repeat steps 2-3 thousands of times.
    

Words (or word-parts) that show up constantly — like `the`, `ing`, `tion` — get merged early and often, so they end up as single, longer tokens. Rare stuff stays chopped into small pieces because the algorithm never saw a good reason to merge it.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/ee85704b-55da-4773-a4f9-e14cf019c4a6.png)

### Let's prove it with real code

Instead of just telling you this, let's actually **build a tiny BPE tokenizer from scratch** and train it, so you can see the merging happen for real.

```python
import re, collections

def get_word_freqs(corpus):
    # split into words/punctuation, and mark word endings with </w>
    words = re.findall(r"\w+|[^\w\s]", corpus.lower())
    freqs = collections.Counter(words)
    return {tuple(list(w) + ["</w>"]): f for w, f in freqs.items()}

def get_pair_stats(word_freqs):
    pairs = collections.Counter()
    for word, freq in word_freqs.items():
        for i in range(len(word) - 1):
            pairs[(word[i], word[i + 1])] += freq
    return pairs

def merge_pair(pair, word_freqs):
    new_word_freqs = {}
    bigram = re.escape(" ".join(pair))
    pattern = re.compile(r"(?<!\S)" + bigram + r"(?!\S)")
    for word, freq in word_freqs.items():
        w_str = pattern.sub("".join(pair), " ".join(word))
        new_word_freqs[tuple(w_str.split(" "))] = freq
    return new_word_freqs

# a tiny training corpus (real tokenizers use billions of words -- this is just a demo!)
corpus = """
Although the young scientist was tired, she couldn't wait to unbelievably
transform her old bicycle into a self-driving robot before the science
fair started tomorrow. The scientist loved science. Science is unbelievably
fun. The robot could bike unbelievably fast.
"""

word_freqs = get_word_freqs(corpus)
for i in range(40):                     # do 40 rounds of merging
    pairs = get_pair_stats(word_freqs)
    if not pairs or max(pairs.values()) < 2:
        break
    best_pair = max(pairs, key=pairs.get)
    word_freqs = merge_pair(best_pair, word_freqs)
```

I actually ran this. Here's what came out the other end — the final subword pieces for a few words in our corpus:

```plaintext
scientist       -> ['scientist</w>']                    (appeared 2x)
unbelievably    -> ['unbelievably</w>']                  (appeared 3x)
science         -> ['science</w>']                       (appeared 3x)
robot           -> ['ro', 'b', 'o', 't</w>']              (appeared 2x)
bicycle         -> ['b', 'i', 'c', 'y', 'c', 'l', 'e</w>']  (appeared 1x)
```

Look at that! `unbelievably` and `science` showed up 3 times in our tiny training text, so after just 40 merge rounds, they'd *already* collapsed into single, whole tokens. But `bicycle` showed up only **once**, so it never got the chance to merge — it's stuck as individual letters. The important caveat is that this tiny corpus is extremely small, so the tokenizer **overfits the examples in the training text**. A real tokenizer learns its vocabulary from a vastly larger and more diverse corpus, so its merges generalize much better.

**This is exactly why tokens have different lengths.** It's not random or based on how "complex" a word looks to a human — it's based purely on **how often chunks of text appeared together during training**. Real tokenizers (like the ones in GPT-4 or Claude) are trained on hundreds of billions of words, so common English words and even common phrases across many languages get their own single token, while rare words, typos, made-up words, or other languages get chopped into smaller pieces.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/6466da70-4eef-4a73-88ce-18bf0a51ed9b.png)

**🤔 Quick check:** if we trained this same tiny tokenizer on *thousands* of books about bicycles instead, what do you think would happen to the word `bicycle`? (Yep — it'd likely become a single token too, once it showed up often enough!)

## 5\. A Little History: How Did We Get Here?

Tokenization didn't start out this clever. It evolved over decades of trial and error:

**🕰️ 1950s–1990s: Simple rule-based splitting** Early computer language tools just split text on spaces and punctuation. `"Don't stop!"` became `["Don't", "stop!"]`. Simple, but it made huge, clumsy vocabularies and completely broke on typos or new words.

**🕰️ 2015: Byte Pair Encoding arrives in NLP** BPE actually started life in the 1990s as a **data compression** algorithm — literally meant for zipping files! In 2015, researchers (Sennrich, Haddow, and Birch) realized it was *perfect* for splitting words into subword units for machine translation. This was the breakthrough that led to modern tokenizers.

**🕰️ 2018: WordPiece and Google's BERT** Google's BERT model used a cousin of BPE called **WordPiece**, which merges pairs based on *likelihood* (how much a merge improves the model's ability to predict text) rather than just raw frequency.

**🕰️ 2018: SentencePiece** Researchers at Google built **SentencePiece**, a tokenizer that treats the whole input — including spaces — as raw text, so it works well even for languages like Japanese or Thai that don't use spaces between words at all.

**🕰️ 2019: Byte-level BPE (GPT-2)** OpenAI's GPT-2 used a clever trick: instead of running BPE on *characters*, it runs on raw **bytes**. This means the tokenizer can represent literally *any* possible text — emojis, rare symbols, any language — without ever hitting an "unknown token" error. This approach became the standard for GPT-3, GPT-4, and beyond (OpenAI's modern version is a fast library called `tiktoken`).

**🕰️ Today** Modern AI tokenizers are trained on colossal, multilingual datasets, with carefully tuned vocabulary sizes (often 32,000–200,000 tokens), and use byte-level fallbacks so they never truly "get stuck" on unfamiliar text.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/29bace16-b67c-4ff4-8a79-44b317b10205.png)

## 6\. Meet the Tokenizer Family 👨‍👩‍👧‍👦

Different AI models use different tokenizer "flavors." Here's a friendly comparison:

| Tokenizer | Used by | Core idea | Fun fact |
| --- | --- | --- | --- |
| **BPE** (Byte Pair Encoding) | Original GPT, many others | Merge the most *frequent* adjacent pair repeatedly | Originally a 1994 file-compression trick! |
| **WordPiece** | BERT, DistilBERT | Merge pairs that most improve prediction *likelihood*, not just frequency | Marks word-continuations with `##`, e.g. `play` + `##ing` |
| **SentencePiece (Unigram)** | T5, ALBERT, many multilingual models | Starts with a big vocabulary and *removes* pieces that matter least | Treats spaces as a normal character (`▁`), so it never needs to "un-tokenize" awkwardly |
| **Byte-level BPE** | GPT-2, GPT-3, GPT-4 (`tiktoken`) | BPE, but on raw bytes instead of characters | Can tokenize *any* text on Earth — emojis, made-up symbols, any alphabet — with zero "unknown" errors |

**🤔 Question to chew on:** Why might a translation app that supports 100 languages prefer SentencePiece over a simple space-based tokenizer? *(Hint: not all languages use spaces between words — think about Japanese, Chinese, or Thai!)*

## 7\. Why Does the *Order* of Tokens Matter?

Here's something important: tokens aren't just a bag of ingredients you can shake up randomly. The **sequence** — the order — carries meaning.

Compare these two sentences, made of the *exact same words*:

*   `"The dog bit the man."`
    
*   `"The man bit the dog."`
    

Same 6 words. Same tokens, even. But very different — and very important — meanings! (One involves a very unlucky dog.)

AI models use the *position* of each token, plus a mechanism called **attention**, to figure out which words relate to which. The model literally tracks "who did what to whom" based on token order. Scramble the order, and you scramble the meaning — sometimes into nonsense, sometimes into something dangerously different.

```python
sentence_a = "The dog bit the man."
sentence_b = "The man bit the dog."

tokens_a = sentence_a.lower().replace(".", "").split()
tokens_b = sentence_b.lower().replace(".", "").split()

print(set(tokens_a) == set(tokens_b))  # True -- same tokens!
print(tokens_a == tokens_b)            # False -- different order!
```

```plaintext
True
False
```

Same set of tokens. Totally different sentence. Order is doing a LOT of work.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/acb5382f-843b-4aea-89bd-38267f366d64.png)

## 8\. Context: The Same Token, Wildly Different Meanings

Here's one of the coolest (and trickiest) parts of language: the exact same word — the exact same *token* — can mean completely different things depending on what surrounds it.

Take the word **"bank"**:

*   *"I sat on the river* ***bank*** *to fish."* → the edge of a river 🏞️
    
*   *"I deposited money in the* ***bank****."* → a place that holds money 🏦
    

Or **"bat"**:

*   *"The* ***bat*** *flew out of the cave at night."* → a flying animal 🦇
    
*   *"She hit the ball with a* ***bat****."* → sports equipment 🏏
    

Here's the key insight: **the token itself doesn't carry the meaning — the surrounding tokens do.** When a tokenizer breaks `"bank"` into a token, it produces the *same* token ID whether it's near "river" or near "money." The tokenizer's job stops there — it doesn't try to guess meaning.

So how does the AI actually figure out *which* "bank" you mean? That's not the tokenizer's job at all — that's the *model's* job, using something called **contextual embeddings**. Here's the short version:

1.  The tokenizer turns `"bank"` into a token ID (say, `#4567`) — always the same ID, no matter the context.
    
2.  The model converts that ID into a big list of numbers (a "vector") representing a *starting guess* at meaning.
    
3.  As the model reads the surrounding words through its **attention** layers, it *adjusts* that vector based on nearby tokens like "river" or "money."
    
4.  By the end, the model has two *very different* internal representations for "bank," even though the token ID started out identical.
    

Think of the token ID like a person's *name badge* at a costume party — the badge says "Alex" the whole night, but depending on which room Alex walks into (the pool party room vs. the ghost story room), people react to "Alex" completely differently based on the surrounding vibe. The badge (token) doesn't change. The *context* changes how it's understood.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/fdc2a367-bf80-4888-9a91-4e20cfd45c68.png)

**One more to think about:** Can you come up with your own example of a word that changes meaning completely based on context? (Try: "light," "spring," "match," or "letter"!)

## 9\. Putting It All Together: Our Sentence, Fully Decoded

Let's return to our star sentence and summarize what a real, production-grade tokenizer (like the byte-level BPE used in modern GPT models) would generally do with it:

> "Although the young scientist was tired, she couldn't wait to unbelievably transform her old bicycle into a self-driving robot before the science fair started tomorrow."

*   **Common whole words** stay as single tokens: `the`, `was`, `her`, `into`, `before` — these show up constantly in English, so they earned their own dedicated token long ago.
    
*   **The contraction** `couldn't` depends on the tokenizer. The GPT-2/3 and GPT-3.5/4 tokenizers split it into `couldn` + `'t`, keeping the apostrophe ending as a reusable piece. GPT-4o's tokenizer keeps `couldn't` as a single token.
    
*   **The hyphenated word** `self-driving` also varies. GPT-2/3 splits it into `self` + `-` + `driving`; GPT-3.5/4 and GPT-4o keep `-driving` together as `self` + `-driving`.
    
*   **The long word** `unbelievably` is a good reminder that tokenizers follow frequency, not grammar. GPT-3.5/4 splits it into `unbelie` + `vably` — not the prefix, root and suffix a linguist would pick — while GPT-2/3 and GPT-4o each hold it as a single token. The pieces are whatever was common in that tokenizer's training text.
    

Counting with OpenAI's public tokenizers, the sentence comes out at **30 tokens** with GPT-2/3 and GPT-3.5/4, and **28** with GPT-4o. That is a little more than the 25 whitespace-separated words, but nowhere close to the 167 individual characters. That "medium zone" is exactly the sweet spot we talked about back in Section 3.

**Remember your guess from the very beginning?** How close were you?

## 10\. Quick Recap

*   **Tokens** are the chunks of text an AI reads — bigger than characters, smaller (usually) than whole sentences, often smaller than whole words.
    
*   We don't use single **characters** as tokens because it makes sequences too long and each piece carries too little meaning.
    
*   Tokens aren't a **fixed length** because tokenizers are trained to merge whatever chunks appear together *most frequently* in real text — common stuff becomes one token, rare stuff stays chopped up.
    
*   There are several tokenizer **families** — BPE, WordPiece, SentencePiece/Unigram, and byte-level BPE — each with a slightly different merging strategy and different strengths (like handling every language on Earth).
    
*   **Token order** matters enormously — the same tokens in a different sequence can mean something totally different.
    
*   The **same token can mean different things** depending on context — the tokenizer just hands over a token ID, and it's the model's attention mechanism that figures out *which* meaning fits, based on the surrounding words.
    

You now know more about tokenization than most people who use AI chatbots every single day.

![](https://cdn.hashnode.com/uploads/covers/653aa283805ce8301a2a5d7d/70616037-1f4c-4034-b345-739f42a6e5e0.png)

## 11\. References & Further Reading

Everything in this post is based on real research and real tools. If you want to go deeper (or just want to see where a grown-up got their facts), here's the trail:

**The original idea (data compression, not language!)**

*   Gage, P. (1994). *A New Algorithm for Data Compression.* The C Users Journal, 12(2), 23–38. — The very first description of Byte Pair Encoding, invented to shrink files, not to help AI read.
    

**Bringing subwords into language models**

*   Schuster, M., & Nakajima, K. (2012). *Japanese and Korean Voice Search.* IEEE ICASSP 2012. — Introduced WordPiece, built to handle languages that don't use spaces between words.
    
*   Sennrich, R., Haddow, B., & Birch, A. (2016). *Neural Machine Translation of Rare Words with Subword Units.* Proceedings of ACL 2016, pp. 1715–1725. https://aclanthology.org/P16-1162/ — The paper that adapted BPE for modern NLP and kicked off the subword-tokenizer era.
    
*   Kudo, T. (2018). *Subword Regularization: Improving Neural Network Translation Models with Multiple Subword Candidates.* Proceedings of ACL 2018. — Introduced the Unigram Language Model tokenization algorithm.
    
*   Kudo, T., & Richardson, J. (2018). *SentencePiece: A Simple and Language Independent Subword Tokenizer and Detokenizer for Neural Text Processing.* Proceedings of EMNLP 2018: System Demonstrations, pp. 66–71. https://aclanthology.org/D18-2012/
    

**The models that made tokenizers famous**

*   Vaswani, A., Shazeer, N., Parmar, N., Uszkoreit, J., Jones, L., Gomez, A. N., Kaiser, Ł., & Polosukhin, I. (2017). *Attention Is All You Need.* Advances in Neural Information Processing Systems 30. https://arxiv.org/abs/1706.03762 — Introduced the Transformer architecture built around self-attention, including the scaled dot-product attention mechanism used in the model.
    
*   Devlin, J., Chang, M.-W., Lee, K., & Toutanova, K. (2019). *BERT: Pre-training of Deep Bidirectional Transformers for Language Understanding.* Proceedings of NAACL-HLT 2019. https://arxiv.org/abs/1810.04805 — A major model built on WordPiece tokenization.
    
*   Radford, A., Wu, J., Child, R., Luan, D., Amodei, D., & Sutskever, I. (2019). *Language Models are Unsupervised Multitask Learners.* OpenAI. https://cdn.openai.com/better-language-models/language\_models\_are\_unsupervised\_multitask\_learners.pdf — GPT-2, the model that introduced byte-level BPE with a 50,257-token vocabulary.
    

**Tools you can actually play with**

*   OpenAI. *tiktoken* (GitHub repository). https://github.com/openai/tiktoken — The fast byte-level BPE tokenizer library used for GPT-3.5, GPT-4, and newer OpenAI models.
    
*   OpenAI Tokenizer (interactive web tool). https://platform.openai.com/tokenizer — Paste in any sentence and watch it get tokenized live, with each token highlighted in a different color.
    

**A note on the code in this post:** the mini-BPE trainer in Section 4 is an original, simplified implementation written for this post, based on the algorithm described in Gage (1994) and Sennrich et al. (2016) — it's not copied from any library, and its output was generated by actually running it.