# LSTM-Boost: Input-Conditioned Confidence Gating for LSTMs

A PyTorch implementation exploring **LSTM-Boost**, a lightweight architectural extension of the standard LSTM cell that adds an input-conditioned confidence gate and a learned boost vector. This repository contains the implementation, ablation experiments, and a vanilla-LSTM baseline for comparison, all evaluated on a word-level language modelling task using the Tiny Shakespeare corpus.

> Independent research conducted as an undergraduate study.

## 📚 Overview

LSTM-Boost augments the standard LSTM hidden-state update with a residual term:

```
h_t ← h_t + confidence_t ⊙ boost_t
```

where `boost_t = tanh(W_boost · x_t)` is computed directly from the current input, and `confidence_t = σ(W_conf · [x_t, h_t, c_t])` is a per-dimension gate that decides how much of the boost to apply to each hidden dimension. The boost is additive, so the model degenerates to a standard LSTM when `W_boost` or `W_conf` approach zero.

## 🗂️ Repository Contents

### Notebooks

#### `nextword_rnn.ipynb`
A word-level next-token prediction notebook implementing both the vanilla LSTM baseline and LSTM-Boost on Shakespeare's text.

**Features:**
- Text preprocessing and word-level vocabulary building (~3,500 unique tokens)
- Sequence-to-sequence data preparation
- Vanilla LSTM and LSTM-Boost cell implementations (with vector confidence gate)
- GPU acceleration support
- Advanced text generation with:
  - Temperature-based sampling
  - Top-K sampling
  - Top-P (nucleus) sampling
- Interactive prompt-based text generation
- Sequential and shuffled 80/20 train/validation splits for generalisation analysis

**Model Architecture:**
```
Embedding (vocab_size → 100 dims)
    ↓
LSTM-Boost cell (100 → 100 hidden units)
    confidence gate: σ(W_conf · [x_t, h_t, c_t])
    boost vector:    tanh(W_boost · x_t)
    h_t ← h_t + confidence_t ⊙ boost_t
    ↓
Linear (100 → vocab_size)
```

**Training Details:**
- Loss Function: Cross-Entropy Loss
- Optimizer: Adam
- Gradient clipping: global norm 1.0
- Device: CUDA if available, else CPU
- Corpus: ~80,000-character subset of Tiny Shakespeare (~12,500 word-level training examples)

### Datasets

- **shakespeare.txt**: Shakespeare corpus subset used for training/evaluation
- **shakesphere.pkl**: Pre-trained model weights for quick inference

## 🚀 Getting Started

### Prerequisites

```bash
pip install torch torchvision
pip install rich  # For progress bars
pip install numpy
```

### Running the Notebook

1. Open `nextword_rnn.ipynb` in Jupyter Notebook or Google Colab
2. Ensure `shakespeare.txt` is in the same directory
3. Run cells sequentially:
   - Data loading and preprocessing
   - Model architecture definition (vanilla LSTM and LSTM-Boost)
   - Training loop (or load pre-trained model)
   - Validation (sequential and shuffled splits)
   - Interactive text generation

### Example Usage

```python
# After training/loading the model:
text = input('enter text: ')  # e.g., "to be or not"
generate_response(model, text, max_tokens=20, temperature=0.8, top_p=0.75)
```

## 🔧 Key Techniques

### 1. Embedding Layer
Converts word indices to dense vector representations (100-dimensional)

### 2. Standard LSTM Gates
Maintains hidden state and cell state:
- **Input Gate**: Controls information flow into the cell
- **Forget Gate**: Decides what to discard from previous state
- **Cell Gate**: Generates new candidate values
- **Output Gate**: Controls output based on cell state

### 3. LSTM-Boost Components
- **Boost vector**: `tanh(W_boost · x_t)`, computed directly from the current input embedding so it carries information complementary to the recurrent hidden state
- **Vector confidence gate**: `σ(W_conf · [x_t, h_t, c_t])`, a per-dimension gate observing input, hidden, and cell state, allowing the model to amplify or suppress the boost independently for each hidden dimension
- **Additive residual**: the boosted term is added to `h_t`, preserving standard LSTM dynamics as a special case

LSTM-Boost shares conceptual similarities with attention mechanisms: the confidence gate (query: hidden/cell state) determines how much information from the current input (value: boost vector) contributes to the hidden representation, though it operates at a single timestep (O(1)) rather than attending over the full sequence history (O(T)).

### 4. Sequence Processing
- Takes the LSTM output at each time step
- Passes through linear layer for vocabulary-size predictions

### 5. Text Generation Strategies

**Temperature Sampling:**
- Lower temperature (< 1.0): More deterministic, conservative
- Higher temperature (> 1.0): More random, creative

**Top-K Sampling:**
- Only sample from top-K most likely tokens
- Prevents very unlikely tokens

**Top-P (Nucleus) Sampling:**
- Sample from smallest set of tokens with cumulative probability ≥ p
- Balances diversity and quality

## 📊 Results

### Parameter Count

| Model | Parameters |
|---|---|
| LSTM | 80,400 |
| LSTM-Boost | 80,400 + 40,000 = 120,400 |
| Increase | +49.8% |

### Training Loss

LSTM-Boost reaches a final training loss of **0.0418** vs. the vanilla LSTM's **0.0830** — a **49.6% reduction** — and converges faster. Generated text is qualitatively more coherent, with longer-range grammatical consistency, while the vanilla baseline degrades into repetitive bigram loops (e.g. "do do", "worthy worthy worthy").

### Validation (Generalisation)

Both models overfit severely on this small (~12,500 example) dataset under both sequential and shuffled 80/20 splits. The vanilla LSTM generalises marginally **better** than LSTM-Boost in this regime:

| Split | LSTM-Boost val. loss | Vanilla LSTM val. loss |
|---|---|---|
| Sequential | 13.7 | 12.6 |
| Shuffled | 12.7 | 11.8 |

LSTM-Boost's additional ~40,000 parameters increase effective model capacity, which accelerates training-loss reduction but, without regularisation, slightly worsens generalisation (7–8% higher validation loss than the baseline).

### Ablations

| Condition | Update Rule | Status |
|---|---|---|
| Vanilla LSTM | `h_t = o_t ⊙ tanh(c_t)` | Baseline |
| Boost (h-conditioned) | `boost = tanh(W_boost · h_t)` | Ablation — no improvement over baseline |
| Boost (1−confidence) | `h ← h + (1−conf) ⊙ boost` | Ablation — fails to converge |
| LSTM-Boost (ours) | `h ← h + conf ⊙ tanh(W_boost · x_t)` | Proposed |

Input-conditioning of the boost vector (rather than hidden-state conditioning) is essential to the training-loss improvement, and the proposed (non-inverted) confidence gate is required for stable convergence.

## ⚠️ Limitations

- **Overfitting regime**: both models overfit severely at this scale; the training-loss comparison reflects fitting speed/capacity, not generalisation
- **Single dataset/domain**: only evaluated on Shakespearean text; other domains (news, code, time series) untested
- **Small scale**: hidden/embedding size 100, ~12,500 training examples
- **Single baseline, single seed**: no comparison against GRU, LayerNorm-LSTM, or residual-LSTM variants, and no multi-seed statistics

## 🎯 Future Enhancements

- **Regularised LSTM-Boost variant** (dropout + weight decay) to test whether the extra capacity can be converted into a genuine generalisation improvement — the proposed next experiment
- Bidirectional LSTM (BiLSTM) and stacked LSTM-Boost layers
- Evaluation on larger/different datasets and modalities
- Beam search for text generation
- Evaluation metrics (perplexity, BLEU score)
- Multi-seed runs and additional baselines (GRU, LayerNorm-LSTM, residual-LSTM)

## 📝 Notes

- Model training takes significant time; pre-trained weights in `shakesphere.pkl` can be loaded for faster inference
- Adjust batch size and learning rate based on your GPU memory
- See the accompanying paper for full derivations, parameter-count analysis, and the proposed regularised variant (Section 6.4)

## 🔗 Resources

- [LSTM Papers](https://arxiv.org/abs/1506.02078) - Sequence to Sequence Learning
- [PyTorch LSTM Documentation](https://pytorch.org/docs/stable/generated/torch.nn.LSTM.html)
- [Understanding LSTMs](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)

---

**Status:** Independent research / educational repository  
**Framework:** PyTorch  
**Python Version:** 3.x  
**GPU Support:** Yes (CUDA)
