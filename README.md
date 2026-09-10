# How an AI Language Model Works

An interactive, animated website that explains how a large language model (LLM) is built and how it answers a prompt, from gathering internet text all the way to the answer on your screen. It's designed as a visual backdrop for a YouTube video: every step is a full-screen scene with a plain-language explanation on the left and an animation on the right.

Everything lives in one file, `how-llms-work.html`. No install, no build step, no server.

## Getting started

Open `how-llms-work.html` in Chrome, Edge or Firefox. Double-clicking the file is enough.

| Key | Action |
|---|---|
| `→` / `Space` | Next step |
| `←` | Previous step |
| `R` | Replay the current step's animation |
| `H` | Hide all controls (clean frame for recording) |
| `F` | Fullscreen |

You can also click any segment in the top progress bar, or any numbered node on the intro map, to jump straight to a step. Adding `#10` to the end of the URL opens step 10 directly, which is handy when re-recording one part.

### Recording tips

Press `F`, then `H` for a clean 16:9 frame; the arrow keys keep working. Steps 9 and 12 play a guided demo on their own, so you can just talk over them. Steps 8 and 10 are interactive: type your own prompt into the tokenizer, or click words and attention heads.

The fonts (Bricolage Grotesque, Figtree, JetBrains Mono) load from Google Fonts, so you need an internet connection for the intended look. Offline, the page falls back to system fonts.

## The journey, step by step

The site is organized into three stages and 13 steps, framed by an intro map and a recap.

**Collect the data**

1. **Crawl the web.** A crawler hops between linked pages and saves copies into an archive.
2. **Clean it up.** Documents pass through six filters: bad-site blocklist, text extraction from HTML, language filter, quality check, duplicate removal, and hiding personal info. Most documents don't make it.

**Train the model**

3. **Build a vocabulary.** Byte-pair encoding (BPE) repeatedly merges the most frequent pair of symbols into a new token.
4. **Pretraining.** The model guesses the next token; wrong guesses nudge the weights. You can watch the correct answer climb to the top and the loss fall.
5. **The base model.** After pretraining, the model continues text instead of answering. Asked a question, it writes more questions.
6. **Fine-tuning.** Example conversations with special role tokens teach the model to write the assistant's part.
7. **Learn from feedback.** Answers are compared, a reward model learns to score them, and the model practices to score higher. See [Where the feedback comes from](#where-the-feedback-in-step-7-comes-from) below.

**Answer your prompt**

8. **Your prompt becomes tokens.** A live tokenizer shows the chat format, the tokens and their ID numbers.
9. **Tokens become vectors.** Embedding lookup plus position vectors, then a meaning map where similar words cluster.
10. **Attention.** "The animal didn't cross the street because it was too tired": watch "it" attend to "animal". Includes three switchable heads and a heatmap showing that tokens can't look ahead.
11. **Through the layers.** A vector climbs through 12 attention + MLP blocks while the best guess sharpens.
12. **Pick the next token.** Softmax probabilities with temperature and top-p sliders, plus a roulette-style pick.
13. **Repeat until done.** Each chosen token is fed back in, one pass at a time, until an end token appears.

## Where the feedback in step 7 comes from

The site says "people compare two answers and pick the better one". In practice, that feedback comes from three sources.

### 1. Paid human raters

AI companies hire contractors, directly or through data-labeling firms, to compare model answers. The raters follow detailed written guidelines from the company that define what "better" means: more helpful, more honest, safer.

The best-documented example is OpenAI's InstructGPT paper (2022), which popularized the method. OpenAI hired a team of about 40 contractors through Upwork and Scale AI and chose them with a screening test. The test measured whether they were sensitive to different groups' preferences and good at spotting harmful outputs. The reward model was trained on about 33,000 prompts. Later models used far more data: Meta's Llama 2 used around 1.4 million comparisons.

So "people's taste" really means the company's standards, applied by trained raters. The InstructGPT authors say this openly: they wrote the labeling instructions and answered raters' questions about edge cases themselves, so the model was aligned to the researchers' preferences. For specialized topics like medicine, law or code, companies increasingly hire domain experts instead of generalists.

### 2. AI feedback

Human rating is slow and expensive, so another AI model can be the judge instead. In Anthropic's Constitutional AI method, a model compares two answers against a written list of principles (the "constitution"). A preference model is trained on these AI judgments and then used as the reward. This is called RL from AI Feedback (RLAIF). In the original paper, the only human input on harmlessness came through the principles themselves.

### 3. Automatic checkers

For tasks with a verifiable answer, no opinion is needed. A math answer is either correct or not, and code either passes its tests or fails. Rewarding checkable results trains reasoning skills directly and is increasingly important for math and programming.

### The mechanism

In all three cases the mechanism is the same. The feedback trains a reward model (or supplies a score directly), and the assistant is then optimized with reinforcement learning to produce answers that score higher. Humans don't rate every answer during training. The reward model stands in for them at scale.

## How numbers work, and why an LLM can do math

### Numbers are just tokens

An LLM has no built-in idea of numbers. "12345" is cut into tokens like any other text: one chunk, several chunks, or single digits, depending on the tokenizer. Try the "12345 + 67890" button in step 8. Each number token gets an embedding vector like any word, and nobody programs in what numbers mean.

### Why it learns arithmetic anyway

The training text is full of sums, prices, dates, tables and homework. To predict the token after "36 + 59 =", the model has to get good at producing "95". Over trillions of prediction steps, the network builds internal circuits that actually compute answers instead of only memorizing them.

### What researchers found inside

**Two paths at once.** Anthropic traced Claude's internal activity while it solved 36 + 59. It doesn't use the school method of carrying the one. Two paths run in parallel: one estimates the rough size of the answer (about 92), and the other works out the exact last digit (6 + 9 ends in 5). Combined, they give 95.

**It explains a method it didn't use.** When asked how it got the answer, Claude describes the standard carry-the-1 algorithm. The likely reason: it learned to *explain* math by imitating explanations written by people, but learned to *do* math on its own, developing strategies it can't see into.

**Numbers on a clock.** Other studies found that models store numbers using repeating, wave-like patterns called Fourier features, somewhat like hands on a clock. In a paper by Zhou et al., MLP layers mainly estimate how big the answer is. Attention layers mainly handle digit-level details, such as whether the result is even or odd. Kantamneni and Tegmark showed that mid-sized models place numbers on a helix and add them by rotating it, a method they call the "Clock" algorithm.

**Researchers still disagree** on how clean these mechanisms are. Some work suggests models rely on a "bag of heuristics", many small rules of thumb, rather than one neat algorithm.

### Why models still make math mistakes

There is no exact calculator inside the model, only learned, approximate circuits. They work well for small, common calculations but can slip on long multiplications or unusual numbers. Modern assistants compensate in three ways:

- **Working step by step** in writing (chain of thought), breaking a hard problem into small, easy ones.
- **Training with automatic checkers**, which reward correct final answers (see feedback source 3 above).
- **Calling a tool.** They write and run code or use a calculator when an exact result matters.

## Technical notes

- **One file.** HTML, CSS and JavaScript are all in `how-llms-work.html`. The only external request is Google Fonts.
- **Real, tiny tokenizer.** Step 8 uses genuine byte-pair encoding with about 1,400 merge rules, trained on a small English word list. Common words come out as single tokens and rare words split into pieces, just like real tokenizers. The vocabulary is much smaller than a production one (50,000 to 200,000 tokens), so IDs and splits won't match GPT or Claude exactly.
- **Illustrative numbers.** The filter percentages in step 2, the probabilities in step 4, the attention weights in step 10 and the layer-by-layer guesses in step 11 are hand-made to show the concept, not measured from a real model. The on-screen text labels the layer panel as an illustration.
- **Sandboxed previews.** In embedded previews (for example inside a chat app), URL hash links and fullscreen may be blocked by the browser. The page catches these errors and keeps working; open the file directly for the full experience.

## Sources

**Data**
- Penedo et al., [The FineWeb Datasets](https://arxiv.org/abs/2406.17557) (2024): 15 trillion tokens from 96 Common Crawl snapshots, and the extraction, filtering and deduplication pipeline.
- [RefinedWeb dataset overview](https://www.emergentmind.com/topics/refinedweb-dataset): deduplication removing 45 to 75% of raw web data.

**Model internals**
- Georgia Tech Polo Club, [Transformer Explainer](https://poloclub.github.io/transformer-explainer/): embeddings, masked self-attention, MLP layers, softmax and temperature.

**Feedback**
- Ouyang et al., [Training language models to follow instructions with human feedback](https://arxiv.org/abs/2203.02155) (InstructGPT, 2022).
- Bai et al., [Constitutional AI: Harmlessness from AI Feedback](https://arxiv.org/abs/2212.08073) (Anthropic, 2022).

**Math**
- Anthropic, [Tracing the thoughts of a large language model](https://www.anthropic.com/research/tracing-thoughts-language-model) (2025).
- Zhou et al., [Pre-trained Large Language Models Use Fourier Features to Compute Addition](https://arxiv.org/abs/2406.03445) (2024).
- Kantamneni and Tegmark, [Language Models Use Trigonometry to Do Addition](https://www.lesswrong.com/posts/E7z89FKLsHk5DkmDL/language-models-use-trigonometry-to-do-addition-1) (2025).
