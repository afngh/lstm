# LSTM (Long Short-Term Memory) Learning Repository

A PyTorch implementation for understanding Long Short-Term Memory (LSTM) networks in depth. This repository contains practical examples demonstrating LSTM architecture and applications.

## 📚 Overview

This repository is focused on learning and understanding LSTM networks through hands-on implementations. LSTMs are a type of recurrent neural network (RNN) capable of learning long-term dependencies in sequential data.

## 🗂️ Repository Contents

### Notebooks

#### `nextword_rnn.ipynb`
A comprehensive LSTM implementation for **next word prediction** using Shakespeare's text corpus.

**Features:**
- Text preprocessing and vocabulary building
- Sequence-to-sequence data preparation
- LSTM model architecture with embedding layer
- GPU acceleration support
- Advanced text generation with:
  - Temperature-based sampling
  - Top-K sampling
  - Top-P (nucleus) sampling
- Interactive prompt-based text generation

**Model Architecture:**
```
Embedding (vocab_size → 64 dims)
    ↓
LSTM (64 → 128 hidden units)
    ↓
Linear (128 → vocab_size)
```

**Training Details:**
- Loss Function: Cross-Entropy Loss
- Optimizer: Adam (lr=0.01)
- Epochs: 35
- Batch Size: 64
- Device: CUDA if available, else CPU

### Datasets

- **shakespeare.txt**: Complete Shakespeare corpus (5.4 MB)
- **shakesphere.pkl**: Pre-trained model weights (23 MB) for quick inference

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
   - Model architecture definition
   - Training loop (or load pre-trained model)
   - Interactive text generation

### Example Usage

```python
# After training/loading the model:
text = input('enter text: ')  # e.g., "to be or not"
generate_response(model, text, max_tokens=20, temperature=0.8, top_p=0.75)
```

## 🔧 Key Techniques

### 1. Embedding Layer
Converts word indices to dense vector representations (64-dimensional)

### 2. LSTM Cell
Maintains hidden state and cell state:
- **Input Gate**: Controls information flow into the cell
- **Forget Gate**: Decides what to discard from previous state
- **Cell Gate**: Generates new candidate values
- **Output Gate**: Controls output based on cell state

### 3. Sequence Processing
- Takes the last LSTM output (final time step)
- Passes through linear layer for vocabulary-size predictions

### 4. Text Generation Strategies

**Temperature Sampling:**
- Lower temperature (< 1.0): More deterministic, conservative
- Higher temperature (> 1.0): More random, creative

**Top-K Sampling:**
- Only sample from top-K most likely tokens
- Prevents very unlikely tokens

**Top-P (Nucleus) Sampling:**
- Sample from smallest set of tokens with cumulative probability ≥ p
- Balances diversity and quality

## 📊 Model Performance

The model learns to predict the next word in Shakespeare's writing style after training for 35 epochs on the corpus.

## 💡 Learning Objectives

- Understand LSTM architecture and gates
- Implement sequence-to-sequence learning
- Work with text data preprocessing
- Use PyTorch's built-in LSTM module
- Implement different sampling strategies for text generation
- Leverage GPU acceleration for training

## 🎯 Future Enhancements

- Bidirectional LSTM (BiLSTM)
- Stacked LSTM layers
- Attention mechanism
- Beam search for text generation
- Evaluation metrics (perplexity, BLEU score)

## 📝 Notes

- Model training takes significant time; pre-trained weights in `shakesphere.pkl` can be loaded for faster inference
- Adjust batch size and learning rate based on your GPU memory
- Text generation quality improves with more training epochs

## 🔗 Resources

- [LSTM Papers](https://arxiv.org/abs/1506.02078) - Sequence to Sequence Learning
- [PyTorch LSTM Documentation](https://pytorch.org/docs/stable/generated/torch.nn.LSTM.html)
- [Understanding LSTMs](https://colah.github.io/posts/2015-08-Understanding-LSTMs/)

---

**Status:** Educational repository for personal learning  
**Framework:** PyTorch  
**Python Version:** 3.x  
**GPU Support:** Yes (CUDA)
