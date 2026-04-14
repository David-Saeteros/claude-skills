# NLP & Text Analysis

## Two layers in this reference
1. **General NLP** — corpus statistics, topic modelling, sentiment, embeddings: for any social scientist with text data
2. **Transformer & XAI layer** — BERT-family models, Integrated Gradients, personality inference: deep coverage for transformer-based pipelines

---

## Layer 1 — General NLP

### When this applies
- Survey open-text responses
- Social media data
- Archival text corpora
- Any dataset with a free-text variable

---

### Text Preprocessing Pipeline

```python
import re
import pandas as pd
import nltk
from nltk.corpus import stopwords
from nltk.stem import WordNetLemmatizer

nltk.download(['punkt', 'stopwords', 'wordnet'])

def preprocess(text):
    text = text.lower()
    text = re.sub(r'[^a-z\s]', '', text)
    tokens = nltk.word_tokenize(text)
    stop_words = set(stopwords.words('english'))
    tokens = [t for t in tokens if t not in stop_words]
    lemmatizer = WordNetLemmatizer()
    tokens = [lemmatizer.lemmatize(t) for t in tokens]
    return ' '.join(tokens)

df['text_clean'] = df['text'].apply(preprocess)
```

**R equivalent:**
```r
library(tidytext)
library(textstem)

df_tokens <- df %>%
  unnest_tokens(word, text) %>%
  anti_join(stop_words) %>%
  mutate(word = lemmatize_words(word))
```

---

### Corpus Statistics

```python
from sklearn.feature_extraction.text import CountVectorizer
from collections import Counter

# Word frequency
all_text = ' '.join(df['text_clean'])
word_freq = Counter(all_text.split()).most_common(30)

# Vocabulary size and lexical diversity
df['n_tokens'] = df['text_clean'].apply(lambda x: len(x.split()))
df['n_types'] = df['text_clean'].apply(lambda x: len(set(x.split())))
df['ttr'] = df['n_types'] / df['n_tokens']  # type-token ratio
```

---

### TF-IDF

Useful for identifying distinctive terms per document or group:
```python
from sklearn.feature_extraction.text import TfidfVectorizer

tfidf = TfidfVectorizer(max_features=500, ngram_range=(1,2))
tfidf_matrix = tfidf.fit_transform(df['text_clean'])
tfidf_df = pd.DataFrame(tfidf_matrix.toarray(), columns=tfidf.get_feature_names_out())
```

---

### Sentiment Analysis

**Lexicon-based (fast, interpretable):**
```python
# VADER — good for social media and short texts
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer

sid = SentimentIntensityAnalyzer()
df['sentiment'] = df['text'].apply(lambda x: sid.polarity_scores(x)['compound'])
# compound: -1 (most negative) to +1 (most positive)
```

```r
library(sentimentr)
df <- df %>%
  mutate(sentiment = sentiment_by(text)$ave_sentiment)
```

**Model-based (more accurate, especially for longer text):**
```python
from transformers import pipeline

sentiment_pipe = pipeline("sentiment-analysis", 
                           model="cardiffnlp/twitter-roberta-base-sentiment-latest")
df['sentiment_label'] = df['text'].apply(lambda x: sentiment_pipe(x[:512])[0]['label'])
```

---

### Topic Modelling

**LDA (interpretable, good baseline):**
```python
from sklearn.decomposition import LatentDirichletAllocation
from sklearn.feature_extraction.text import CountVectorizer

vectorizer = CountVectorizer(max_features=1000, min_df=5)
dtm = vectorizer.fit_transform(df['text_clean'])

lda = LatentDirichletAllocation(n_components=5, random_state=42)
lda.fit(dtm)

# Display top words per topic
feature_names = vectorizer.get_feature_names_out()
for i, topic in enumerate(lda.components_):
    top_words = [feature_names[j] for j in topic.argsort()[-10:]]
    print(f"Topic {i}: {', '.join(top_words)}")
```

**BERTopic (state-of-the-art, coherent topics):**
```python
from bertopic import BERTopic

topic_model = BERTopic(language="english", calculate_probabilities=True)
topics, probs = topic_model.fit_transform(df['text'].tolist())
topic_model.visualize_topics()
topic_model.get_topic_info()
```

**How many topics?** For LDA: use coherence scores. For BERTopic: let it decide, then merge near-duplicate topics.

---

### Word Embeddings for Social Science

```python
from sentence_transformers import SentenceTransformer

# Good general-purpose embeddings
model = SentenceTransformer('all-MiniLM-L6-v2')
embeddings = model.encode(df['text'].tolist(), show_progress_bar=True)

# Cluster documents
from sklearn.cluster import KMeans
kmeans = KMeans(n_clusters=5, random_state=42)
df['cluster'] = kmeans.fit_predict(embeddings)
```

---

## Layer 2 — Transformer & XAI Pipelines

### When this applies
- Personality inference from free text
- Fine-tuned BERT-family models
- Feature attribution / explainability (Integrated Gradients, SHAP)
- Text classification with psychological constructs

---

### Loading & Classifying with Fine-tuned Models

```python
from transformers import AutoTokenizer, AutoModelForSequenceClassification
import torch

model_name = "your-model-name-or-path"
tokenizer = AutoTokenizer.from_pretrained(model_name)
model = AutoModelForSequenceClassification.from_pretrained(model_name)
model.eval()

def classify(text):
    inputs = tokenizer(text, return_tensors="pt", 
                       truncation=True, max_length=512, padding=True)
    with torch.no_grad():
        outputs = model(**inputs)
    probs = torch.softmax(outputs.logits, dim=-1)
    return probs.numpy()

df['predictions'] = df['text'].apply(classify)
```

---

### Integrated Gradients (XAI)

IG attributes a model's prediction to input token contributions. Core formula:
*IG_i = (x_i - x'_i) × ∫ ∂F/∂x_i along the path from baseline x' to input x*

```python
from captum.attr import LayerIntegratedGradients
import torch

def compute_ig(text, target_class=1, n_steps=50):
    inputs = tokenizer(text, return_tensors="pt", truncation=True, max_length=512)
    input_ids = inputs['input_ids']
    
    # Baseline: [PAD] tokens
    baseline_ids = torch.zeros_like(input_ids)
    
    lig = LayerIntegratedGradients(
        lambda ids: model(ids).logits[:, target_class],
        model.bert.embeddings
    )
    
    attributions, delta = lig.attribute(
        input_ids,
        baseline_ids,
        n_steps=n_steps,
        return_convergence_delta=True
    )
    
    # Sum over embedding dimension, get per-token attribution
    token_attrs = attributions.sum(dim=-1).squeeze(0)
    tokens = tokenizer.convert_ids_to_tokens(input_ids[0])
    
    return list(zip(tokens, token_attrs.tolist())), delta.item()

attrs, convergence = compute_ig(text_sample)
```

**Convergence delta** should be close to 0 (ideally < 0.05) — flag if not.

---

### Visualising Token Attributions

```python
def visualise_ig(token_attr_pairs, title="Token Attributions"):
    tokens = [t for t, _ in token_attr_pairs if t not in ['[CLS]','[SEP]','[PAD]']]
    attrs  = [a for t, a in token_attr_pairs if t not in ['[CLS]','[SEP]','[PAD]']]
    
    import matplotlib.pyplot as plt
    import matplotlib.colors as mcolors
    
    fig, ax = plt.subplots(figsize=(max(10, len(tokens)*0.5), 2))
    norm = mcolors.TwoSlopeNorm(vmin=min(attrs), vcenter=0, vmax=max(attrs))
    cmap = plt.cm.RdYlGn
    
    for i, (tok, attr) in enumerate(zip(tokens, attrs)):
        ax.text(i, 0, tok, ha='center', va='center', fontsize=9,
                bbox=dict(boxstyle='round,pad=0.3', 
                          facecolor=cmap(norm(attr)), alpha=0.8))
    
    ax.set_xlim(-0.5, len(tokens) - 0.5)
    ax.axis('off')
    ax.set_title(title)
    plt.tight_layout()
    plt.show()
```

---

### Aggregating Attributions for Psycholinguistic Analysis

```python
import numpy as np

def aggregate_attributions_by_word(token_attr_pairs, tokenizer_type='wordpiece'):
    """Merge subword tokens and their attributions back to whole words."""
    words, word_attrs = [], []
    current_word, current_attrs = '', []
    
    for token, attr in token_attr_pairs:
        if token in ['[CLS]', '[SEP]', '[PAD]']:
            continue
        if token.startswith('##'):  # WordPiece subword
            current_word += token[2:]
            current_attrs.append(attr)
        else:
            if current_word:
                words.append(current_word)
                word_attrs.append(np.mean(current_attrs))
            current_word = token
            current_attrs = [attr]
    
    if current_word:
        words.append(current_word)
        word_attrs.append(np.mean(current_attrs))
    
    return pd.DataFrame({'word': words, 'attribution': word_attrs})
```

---

### Personality Inference Pipeline (Big Five)

```python
# Pattern for OCEAN inference from text
traits = ['Openness', 'Conscientiousness', 'Extraversion', 'Agreeableness', 'Neuroticism']

results = []
for _, row in df.iterrows():
    text = row['text']
    scores = {}
    for i, trait in enumerate(traits):
        probs = classify(text)
        scores[trait] = probs[0][i]  # adjust indexing to your model's output
    results.append(scores)

df_traits = pd.DataFrame(results)
df = pd.concat([df, df_traits], axis=1)
```

---

### Model Evaluation for Classification

```python
from sklearn.metrics import classification_report, confusion_matrix
import seaborn as sns

print(classification_report(df['true_label'], df['pred_label']))

cm = confusion_matrix(df['true_label'], df['pred_label'])
sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')
```

For regression outputs (continuous personality scores):
```python
from sklearn.metrics import mean_absolute_error, r2_score
print(f"MAE: {mean_absolute_error(y_true, y_pred):.3f}")
print(f"R²: {r2_score(y_true, y_pred):.3f}")
```

---

### Adaptive Depth Note

For highly specialised topics (e.g. custom training loops, LoRA fine-tuning, multi-label personality classification with ordinal regression heads), this skill will scaffold the approach and flag where specialist resources or experimentation are needed. The Hugging Face documentation and the `captum` library docs are the primary references for IG implementation details.
