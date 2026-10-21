# Machine Learning & AI Interview Questions

ML concepts, algorithms, deep learning, LLMs, and AI application design.

---

## 1. What's the difference between supervised and unsupervised learning?

<details>
<summary>View Answer</summary>

<h3>Supervised Learning: Labeled Data</h3>

<p><strong>Learn from labeled examples (input → output pairs).</strong></p>

<h4>Regression: Predict Continuous Values</h4>

<p>Example: Predict house price from features</p>

```
Input: [size, beds, location] → Output: $500,000
Input: [size, beds, location] → Output: $750,000
Model learns: price = 100*size + 50*beds + ...
```

<h4>Classification: Predict Categories</h4>

<p>Example: Spam detector</p>

```
Input: Email text → Output: [Spam, Not Spam]
Input: Medical scan → Output: [Cancer, Benign]
```

<h3>Unsupervised Learning: No Labels</h3>

<p><strong>Find patterns in unlabeled data.</strong></p>

<h4>Clustering: Group Similar Items</h4>

```
Input: Customer purchase histories
No labels
Model discovers: 3 customer segments
```

<h4>Dimensionality Reduction: Simplify Data</h4>

```
Input: 1000 features
Output: 50 most important features
Reduces noise, improves efficiency
```

<h3>Comparison Table</h3>

<table>
<tr>
<th>Property</th>
<th>Supervised</th>
<th>Unsupervised</th>
</tr>
<tr>
<td><strong>Data</strong></td>
<td>Labeled (x, y)</td>
<td>Unlabeled (x only)</td>
</tr>
<tr>
<td><strong>Goal</strong></td>
<td>Predict labels</td>
<td>Find patterns</td>
</tr>
<tr>
<td><strong>Evaluation</strong></td>
<td>Easy (compare to labels)</td>
<td>Hard (no ground truth)</td>
</tr>
<tr>
<td><strong>Effort</strong></td>
<td>Labeling is expensive</td>
<td>No labeling needed</td>
</tr>
<tr>
<td><strong>Examples</strong></td>
<td>Classification, Regression</td>
<td>Clustering, Dimension Reduction</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/ai-ml/fundamentals.md">Learn more: ML Fundamentals</a></p>

</details>

---

## 2. Explain overfitting and how to prevent it

<details>
<summary>View Answer</summary>

<h3>Overfitting: Memorizing Data</h3>

<p><strong>Model learns training data too well, including noise. Fails on new data.</strong></p>

```
Training data: High accuracy (99%)
Test data: Low accuracy (60%)
→ Overfitting: Model memorized training, doesn't generalize
```

<h3>Example</h3>

```
Training: Learning exact student names for exam answers
→ Works on training set (100%)
→ Fails on new students (0%)
vs
Learning the underlying concepts
→ Works on training (85%)
→ Works on new data (82%)
```

<h3>How to Detect Overfitting</h3>

```
Plot: Training vs Test Accuracy
         Training ─────────────
         Test        ──────
         Gap = overfitting
```

<h3>Prevention Strategies</h3>

<table>
<tr>
<th>Technique</th>
<th>How It Works</th>
<th>When to Use</th>
</tr>
<tr>
<td><strong>Regularization</strong></td>
<td>Penalize complex models (L1/L2)</td>
<td>Always (linear, neural nets)</td>
</tr>
<tr>
<td><strong>Dropout</strong></td>
<td>Randomly disable neurons during training</td>
<td>Deep neural networks</td>
</tr>
<tr>
<td><strong>Early Stopping</strong></td>
<td>Stop training when validation error increases</td>
<td>Neural networks</td>
</tr>
<tr>
<td><strong>Cross-Validation</strong></td>
<td>Train/test on multiple data splits</td>
<td>All algorithms</td>
</tr>
<tr>
<td><strong>More Data</strong></td>
<td>Larger training set reduces memorization</td>
<td>Always helps</td>
</tr>
<tr>
<td><strong>Simpler Model</strong></td>
<td>Fewer parameters, less capacity</td>
<td>When possible</td>
</tr>
</table>

<h3>Regularization Example</h3>

```
Loss = Error + λ * (model complexity)
       ↑         ↑
   accuracy    penalty

If λ high: Model stays simple
If λ low: Can overfit
```

<p>📚 <a href="/anandu-interview-playbook/ai-ml/fundamentals.md">Learn more: Model Validation</a></p>

</details>

---

## 3. What's the bias-variance tradeoff?

<details>
<summary>View Answer</summary>

<h3>Bias: Underfitting</h3>

<p><strong>Model too simple, misses true patterns.</strong></p>

```
Example: Fitting straight line to curved data
Result: Poor fit on both training and test
Prediction: Consistently wrong
```

<h3>Variance: Overfitting</h3>

<p><strong>Model too complex, learns noise.</strong></p>

```
Example: Fitting polynomial to noisy data
Result: Perfect fit on training, poor on test
Prediction: High variability with small data changes
```

<h3>The Tradeoff</h3>

```
Model Complexity →
Bias:   High → Low → Low
Variance: Low → Low → High
             Goldilocks zone
```

<h3>Total Error = Bias² + Variance + Irreducible Error</h3>

<table>
<tr>
<th>Scenario</th>
<th>Bias</th>
<th>Variance</th>
<th>Problem</th>
<th>Solution</th>
</tr>
<tr>
<td><strong>Simple Model</strong></td>
<td>High</td>
<td>Low</td>
<td>Underfitting</td>
<td>More complex</td>
</tr>
<tr>
<td><strong>Complex Model</strong></td>
<td>Low</td>
<td>High</td>
<td>Overfitting</td>
<td>Regularize</td>
</tr>
<tr>
<td><strong>Just Right</strong></td>
<td>Medium</td>
<td>Medium</td>
<td>Optimal</td>
<td>Good model</td>
</tr>
</table>

<h3>Visual Representation</h3>

```
Underfitting          Optimal            Overfitting
○ ○ ○ ○              ○ ╲               ╲ ○ ╱
○ ○ ● ○              ○ ╲ ●             ╲ ● ╱
○ ● ○ ○              ● ╲ ○             ╱ ○ ╲
(straight line)   (slight curve)    (wiggly line)
```

<p>📚 <a href="/anandu-interview-playbook/ai-ml/fundamentals.md">Learn more: Bias-Variance</a></p>

</details>

---

## 4. Explain how CNNs work and when to use them

<details>
<summary>View Answer</summary>

<h3>CNN: Convolutional Neural Network</h3>

<p><strong>Specialized neural network for image/spatial data.</strong></p>

<h3>Key Concept: Convolution</h3>

```
Filter (3x3):        Image:
[1 0 -1]            [5 2 8]
[2 0 -2]     *      [3 1 4]
[1 0 -1]            [7 9 2]
       ↓
Applies filter across image, detecting features
```

<h3>CNN Layers</h3>

<table>
<tr>
<th>Layer</th>
<th>Purpose</th>
<th>Output</th>
</tr>
<tr>
<td><strong>Input</strong></td>
<td>Image (e.g., 28×28×3)</td>
<td>Pixels</td>
</tr>
<tr>
<td><strong>Convolutional</strong></td>
<td>Detect features (edges, shapes)</td>
<td>Feature maps</td>
</tr>
<tr>
<td><strong>ReLU</strong></td>
<td>Activation (non-linearity)</td>
<td>Sparse activation</td>
</tr>
<tr>
<td><strong>Pooling</strong></td>
<td>Reduce dimensionality</td>
<td>Smaller feature maps</td>
</tr>
<tr>
<td><strong>Flatten</strong></td>
<td>Convert to 1D vector</td>
<td>Feature vector</td>
</tr>
<tr>
<td><strong>Dense</strong></td>
<td>Classification</td>
<td>Probabilities</td>
</tr>
</table>

<h3>Why CNNs for Images?</h3>

<ul>
<li><strong>✅ Spatial locality:</strong> Nearby pixels are related</li>
<li><strong>✅ Parameter sharing:</strong> Same filter detects feature anywhere</li>
<li><strong>✅ Hierarchical:</strong> Simple features → complex features</li>
<li><strong>✅ Translation invariant:</strong> Detects object anywhere in image</li>
</ul>

<h3>When to Use CNNs</h3>

<ul>
<li><strong>✅ Image classification:</strong> Cat/dog detector</li>
<li><strong>✅ Object detection:</strong> Find objects in images</li>
<li><strong>✅ Segmentation:</strong> Pixel-level classification</li>
<li><strong>✅ Face recognition:</strong> Identify faces</li>
<li><strong>❌ Not for:</strong> Tabular data, time series (use RNN/LSTM)</li>
</ul>

<p>📚 <a href="/anandu-interview-playbook/ai-ml/deep-learning.md">Learn more: Deep Learning & CNNs</a></p>

</details>

---

## 5. Explain the transformer architecture and attention mechanism

<details>
<summary>View Answer</summary>

<h3>Transformer: Revolution in NLP</h3>

<p><strong>Architecture based on attention, enables parallel processing.</strong></p>

<h3>Attention Mechanism: Focus on Relevant Info</h3>

<p><strong>Model learns which parts of input are important for each output.</strong></p>

```
Sentence: "The bank is on the river"
Word: "bank"
Attention: Focus on "river" (context clue)
           Less focus on "the"
```

<h3>Multi-Head Attention</h3>

```
Multiple attention heads looking at different aspects:
  Head 1: Subject-verb relationships
  Head 2: Adjective-noun relationships
  Head 3: Long-range dependencies
→ Combine all heads → Rich representation
```

<h3>Transformer Architecture</h3>

```
Input: "Hello world"
  ↓
Embedding (word vectors)
  ↓
Positional Encoding (position info)
  ↓
Encoder Stack:
  - Multi-head Attention
  - Feed-forward Network
  (repeated 6-12 times)
  ↓
Decoder Stack:
  - Multi-head Attention
  - Cross-attention to Encoder
  - Feed-forward Network
  ↓
Output: "Hola mundo"
```

<h3>Why Transformers are Powerful</h3>

<ul>
<li><strong>✅ Parallel:</strong> Process all words simultaneously (vs RNN sequential)</li>
<li><strong>✅ Long-range:</strong> Attention captures distant dependencies</li>
<li><strong>✅ Scalable:</strong> Train on massive datasets</li>
<li><strong>✅ Transfer learning:</strong> Pre-train, fine-tune for tasks</li>
</ul>

<h3>Transformer Variants</h3>

<table>
<tr>
<th>Model</th>
<th>Type</th>
<th>Use Case</th>
</tr>
<tr>
<td><strong>BERT</strong></td>
<td>Encoder-only</td>
<td>Understanding (classification, Q&A)</td>
</tr>
<tr>
<td><strong>GPT</strong></td>
<td>Decoder-only</td>
<td>Generation (text completion, chat)</td>
</tr>
<tr>
<td><strong>T5</strong></td>
<td>Encoder-Decoder</td>
<td>Seq2seq (translation, summarization)</td>
</tr>
</table>

<p>📚 <a href="/anandu-interview-playbook/ai-ml/llms-transformers.md">Learn more: Transformers & LLMs</a></p>

</details>

---

## 6. What's RAG and why is it useful?

<details>
<summary>View Answer</summary>

<h3>RAG: Retrieval-Augmented Generation</h3>

<p><strong>Combine information retrieval with language generation to ground LLM responses in facts.</strong></p>

<h3>Problem: LLM Hallucinations</h3>

<p><strong>LLMs generate plausible-sounding but false information.</strong></p>

```
User: "What's the release date of Avatar 3?"
LLM (without RAG): "Avatar 3 was released on December 15, 2024"
   ❌ Might be wrong, outdated, or made up
```

<h3>RAG Solution</h3>

```
1. Retrieve relevant documents from knowledge base
2. Pass documents + query to LLM
3. LLM generates answer grounded in retrieved facts
```

<h3>RAG Workflow</h3>

```
User Query: "What's Avatar 3 release date?"
  ↓
Retriever: Search knowledge base
  ↓ (finds relevant documents)
Relevant docs: "Avatar: The Way of Water released Dec 16, 2022"
  ↓
LLM: Generate answer using retrieved docs
  ↓
Output: Grounded response with sources
```

<h3>Components</h3>

<table>
<tr>
<th>Component</th>
<th>Purpose</th>
<th>Technology</th>
</tr>
<tr>
<td><strong>Vector Embeddings</strong></td>
<td>Convert text to vectors</td>
<td>OpenAI embeddings, Hugging Face</td>
</tr>
<tr>
<td><strong>Vector DB</strong></td>
<td>Store/search embeddings</td>
<td>Pinecone, Weaviate, Milvus</td>
</tr>
<tr>
<td><strong>Retriever</strong></td>
<td>Find relevant documents</td>
<td>Semantic search</td>
</tr>
<tr>
<td><strong>LLM</strong></td>
<td>Generate answer</td>
<td>GPT-4, Llama, Claude</td>
</tr>
</table>

<h3>Benefits</h3>

<ul>
<li><strong>✅ Factual accuracy:</strong> Grounds in retrieved documents</li>
<li><strong>✅ Updated knowledge:</strong> Can index new documents</li>
<li><strong>✅ Sources:</strong> Can cite where answer came from</li>
<li><strong>✅ Cost:</strong> Smaller LLM sufficient (don't need largest models)</li>
</ul>

<h3>Example: Document QA</h3>

```
Upload: Company employee handbook
User: "What's our leave policy?"
RAG: Retrieves relevant section
     Generates answer from policy
     Cites policy document
```

<p>📚 <a href="/anandu-interview-playbook/ai-ml/rag-generative.md">Learn more: RAG & Generative AI</a></p>

</details>

---

## 7. What's the difference between precision and recall?

<details>
<summary>View Answer</summary>

<h3>Classification Confusion Matrix</h3>

<table>
<tr>
<th></th>
<th>Predicted Positive</th>
<th>Predicted Negative</th>
</tr>
<tr>
<td><strong>Actual Positive</strong></td>
<td>TP (True Positive)</td>
<td>FN (False Negative)</td>
</tr>
<tr>
<td><strong>Actual Negative</strong></td>
<td>FP (False Positive)</td>
<td>TN (True Negative)</td>
</tr>
</table>

<h3>Precision: False Positive Rate</h3>

<p><strong>Of positive predictions, how many are correct?</strong></p>

```
Precision = TP / (TP + FP)

Email spam detector:
  "How many emails we flagged as spam were actually spam?"
  High precision = fewer false spam alerts (better UX)
```

<h3>Recall: False Negative Rate</h3>

<p><strong>Of actual positives, how many did we find?</strong></p>

```
Recall = TP / (TP + FN)

Email spam detector:
  "Of all spam emails, how many did we catch?"
  High recall = fewer spam in inbox (better filtering)
```

<h3>Tradeoff: Precision vs Recall</h3>

```
Strict model (high threshold)
→ High precision, low recall
→ Few false positives, many false negatives

Lenient model (low threshold)
→ Low precision, high recall
→ Many false positives, few false negatives
```

<h3>When to Optimize</h3>

<table>
<tr>
<th>Scenario</th>
<th>Optimize For</th>
<th>Why</th>
</tr>
<tr>
<td><strong>Medical diagnosis</strong></td>
<td>Recall</td>
<td>Don't miss patients with disease</td>
</tr>
<tr>
<td><strong>Spam detection</strong></td>
<td>Precision</td>
<td>Don't block legitimate emails</td>
</tr>
<tr>
<td><strong>Document search</strong></td>
<td>Precision</td>
<td>Return relevant results</td>
</tr>
<tr>
<td><strong>Fraud detection</strong></td>
<td>Recall</td>
<td>Catch all fraud</td>
</tr>
</table>

<h3>F1 Score: Balance Both</h3>

```
F1 = 2 × (Precision × Recall) / (Precision + Recall)
     ↑
   Harmonic mean, penalizes imbalance
```

<p>📚 <a href="/anandu-interview-playbook/ai-ml/fundamentals.md">Learn more: Model Evaluation</a></p>

</details>

---

## 8. How do you detect and handle model drift?

<details>
<summary>View Answer</summary>

<h3>Model Drift: Performance Degradation</h3>

<p><strong>Model performance decreases over time on new data.</strong></p>

```
Deployment day: Accuracy 95%
Month 1: Accuracy 94%
Month 3: Accuracy 88%
Year 1: Accuracy 72% ← Model drift!
```

<h3>Types of Drift</h3>

<h4>Data Drift (Covariate Shift)</h4>

<p><strong>Input data distribution changed.</strong></p>

```
Example: Fraud detection
  Training: 2020 transaction patterns
  2024: Different payment methods, new fraud tactics
  Model sees unexpected patterns
```

<h4>Concept Drift (Label Drift)</h4>

<p><strong>Relationship between input and output changed.</strong></p>

```
Example: Stock prediction
  2020: Certain features predicted price well
  2024: Market dynamics changed, relationships different
```

<h3>Detection Methods</h3>

<table>
<tr>
<th>Method</th>
<th>How It Works</th>
<th>Example</th>
</tr>
<tr>
<td><strong>Performance monitoring</strong></td>
<td>Track accuracy/metrics over time</td>
<td>Alert if accuracy drops 5%</td>
</tr>
<tr>
<td><strong>Statistical tests</strong></td>
<td>Compare data distributions (Kolmogorov-Smirnov)</td>
<td>Detect if data shifted significantly</td>
</tr>
<tr>
<td><strong>Prediction confidence</strong></td>
<td>Monitor prediction uncertainty</td>
<td>If confidence drops, data might have drifted</td>
</tr>
<tr>
<td><strong>Ground truth comparison</strong></td>
<td>Compare predictions to actual labels</td>
<td>Manual labeling on sample, measure error</td>
</tr>
</table>

<h3>Handling Drift</h3>

<ul>
<li><strong>1. Retrain:</strong> Periodically retrain on new data</li>
<li><strong>2. Continuous learning:</strong> Update model incrementally</li>
<li><strong>3. Ensemble:</strong> Multiple models for robustness</li>
<li><strong>4. Monitoring:</strong> Set up alerts for performance drop</li>
<li><strong>5. Feature engineering:</strong> Add new relevant features</li>
</ul>

<h3>Retraining Strategy</h3>

```
Weekly:
  - Collect new labeled data
  - Measure performance
  - If accuracy drops > 2%: Retrain
  - A/B test new model
  - Deploy if better
```

<h3>Example: Email Spam Detector</h3>

```
Month 0: 95% accuracy
Month 3: New spam tactics emerge, 90% accuracy
Action: Retrain on recent emails with new patterns
Result: 94% accuracy restored
```

<p>📚 <a href="/anandu-interview-playbook/ai-ml/fundamentals.md">Learn more: Model Monitoring</a></p>

</details>

---

**See also:** [ML Fundamentals](../../ai-ml/fundamentals.md), [Algorithms](../../ai-ml/algorithms.md), [Deep Learning](../../ai-ml/deep-learning.md), [Transformers & LLMs](../../ai-ml/llms-transformers.md), [RAG & Generative AI](../../ai-ml/rag-generative.md), [Interview Preparation](../index.md)
