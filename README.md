# PhishNet-Transformer (PhishGuard)

A machine learning, deep learning, and Transformer-based phishing URL detection project using the **PhiUSIIL Phishing URL Dataset**.

## Project Overview

**PhishNet-Transformer** aims to build a phishing URL detection pipeline by transforming the raw PhiUSIIL dataset into clean, balanced, and correctly labeled datasets for:

- Classic Machine Learning models
- Deep Learning models (character/sequence-based)
- DistilBERT Transformer-based classification

The goal is to determine whether a URL is **phishing** or **legitimate**.

Because the PhiUSIIL dataset is large (~235K URLs), it is well suited not just for classic feature-based ML but also for **deep learning models that learn directly from raw URL character sequences**, without relying on hand-engineered features.

## Dataset

This project uses the **PhiUSIIL Phishing URL Dataset**.

Dataset source:

https://www.kaggle.com/datasets/ndarvind/phiusiil-phishing-url-dataset

After downloading the dataset, place the CSV file in the same directory as the preprocessing notebook, or update `DATA_PATH` to point to the correct file.

## Label Convention

The raw PhiUSIIL dataset uses the following label convention:

| Raw Label | Meaning |
|---|---|
| `1` | Legitimate |
| `0` | Phishing |

For this project, the labels are reversed during preprocessing so that the project consistently uses:

| Project Label | Meaning |
|---|---|
| `1` | Phishing |
| `0` | Legitimate |

This conversion is performed during preprocessing and is used consistently throughout the project.

## Project Pipeline

```text
Raw PhiUSIIL Dataset
        │
        ▼
   Data Loading
        │
        ▼
   Data Cleaning
        │
        ▼
   Label Correction
        │
        ▼
 Duplicate / Invalid Data Handling
        │
        ▼
    Class Balancing
        │
        ▼
Train / Validation / Test Split
        │
        ├─────────────────┬─────────────────┐
        ▼                 ▼                 ▼
   Classic ML        Deep Learning      DistilBERT
    Pipeline           Pipeline          Pipeline
   (feature-based)   (char/sequence-    (Transformer)
                       based)
        │                 │                 │
        └────────┬────────┴────────┬────────┘
                 ▼                 ▼
              Model Evaluation
                 │
                 ▼
        Performance Comparison
```

## Dataset Preparation

The preprocessing stage converts the raw dataset into three standardized files:

```text
train.csv
val.csv
test.csv
```

These files are used by both the classic ML and DistilBERT pipelines so that both approaches are evaluated using the same data.

The preprocessing workflow includes:

1. Loading the raw PhiUSIIL CSV file
2. Inspecting the dataset
3. Selecting the required URL and label information
4. Handling missing or invalid records
5. Removing duplicate URLs
6. Correcting the original label convention
7. Handling class imbalance
8. Splitting the dataset into training, validation, and test sets
9. Saving the processed datasets as CSV files

## Final Dataset Structure

The processed datasets should contain the information required for model training, including:

```text
URL
label
```

The project uses:

```text
label = 1 → Phishing
label = 0 → Legitimate
```

## Train / Validation / Test Sets

### Training Set

Used to train the machine learning and Transformer models.

```text
train.csv
```

### Validation Set

Used during model development, validation, and hyperparameter tuning.

```text
val.csv
```

### Test Set

Used for the final evaluation of trained models.

```text
test.csv
```

The test set should remain isolated from model training and hyperparameter tuning to provide an unbiased evaluation.

## Models

The project uses two main approaches.

### 1. Classic Machine Learning

Traditional machine learning models can classify URLs using manually engineered URL features.

Possible models include:

- Logistic Regression
- Random Forest
- Support Vector Machine
- XGBoost
- LightGBM
- Other classification algorithms

Example URL features can include:

- URL length
- Number of dots
- Number of hyphens
- Number of special characters
- Number of digits
- Number of subdomains
- HTTPS usage
- IP address usage
- Suspicious keywords
- Domain-related characteristics

The classic ML pipeline generally follows:

```text
URL
 │
 ▼
Feature Extraction
 │
 ▼
Feature Vector
 │
 ▼
ML Classifier
 │
 ▼
0 = Legitimate
1 = Phishing
```

### 2. Deep Learning (Character/Sequence-Based)

With a large dataset like PhiUSIIL, deep learning models can learn directly from the **raw character sequence of a URL**, rather than relying on manually engineered features. This lets the model pick up on subtle character-level patterns (e.g. unusual character n-grams, obfuscated domains, suspicious substrings) that hand-crafted features might miss.

Possible architectures include:

- **Character-level CNN** — 1D convolutions over character embeddings to detect local suspicious patterns (e.g. `paypa1`, `-secure-login-`)
- **LSTM / BiLSTM** — sequential model that captures long-range dependencies across the full URL string
- **CNN-LSTM hybrid** — convolutional layers for local feature extraction feeding into a recurrent layer for sequence context
- **GRU** — a lighter-weight recurrent alternative to LSTM for faster training on large datasets

The deep learning pipeline generally follows:

```text
URL
 │
 ▼
Character-level Tokenization
 │
 ▼
Padded Integer Sequence
 │
 ▼
Embedding Layer
 │
 ▼
CNN / LSTM / BiLSTM / GRU Layers
 │
 ▼
Dense + Sigmoid Output
 │
 ▼
0 = Legitimate
1 = Phishing
```

Key considerations for this pipeline:

- **Tokenization**: URLs are broken into individual characters (or n-grams) and mapped to integer indices using a character-level vocabulary built from the training set.
- **Sequence length**: URLs are padded/truncated to a fixed maximum length so they can be processed in batches.
- **Embedding layer**: learns a dense vector representation for each character, allowing the model to capture similarity between characters/substrings.
- **Regularization**: dropout and early stopping (monitored on `val.csv`) are used to prevent overfitting, since deep models can memorize URL patterns given a large enough dataset.
- **Class imbalance**: handled via class weighting or oversampling, consistent with the classic ML pipeline, so the deep model isn't biased toward the majority class.
- **Framework**: implemented using TensorFlow/Keras or PyTorch.

This deep learning approach sits between classic feature-based ML (fast, interpretable, but feature-limited) and DistilBERT (large, pretrained, language-oriented) — it is lightweight enough to train from scratch on the full dataset while still learning representations directly from raw URLs.

### 3. DistilBERT Transformer

**DistilBERT** is a smaller and faster version of BERT based on the Transformer architecture.

In this project, DistilBERT is fine-tuned to classify URLs directly as phishing or legitimate.

The pipeline is:

```text
URL
 │
 ▼
DistilBERT Tokenizer
 │
 ▼
Tokenized URL
 │
 ▼
DistilBERT
 │
 ▼
Classification Layer
 │
 ▼
0 = Legitimate
1 = Phishing
```

Unlike traditional ML, which depends heavily on manually engineered features, the Transformer model learns useful patterns from the URL representations during training.

> Note: DistilBERT was originally pretrained on natural language rather than specifically on URLs. It is used here as a practical Transformer-based classification approach for the project.

## Evaluation

The models should be evaluated using appropriate binary-classification metrics:

- Accuracy
- Precision
- Recall
- F1-score
- ROC-AUC
- Confusion Matrix

For phishing detection, **precision and recall are particularly important**.

A high phishing recall means the model identifies more malicious URLs, while high precision means fewer legitimate URLs are incorrectly classified as phishing.

### Model Comparison

A final comparison can be presented as:

| Model | Accuracy | Precision | Recall | F1-Score | ROC-AUC |
|---|---:|---:|---:|---:|---:|
| Classic ML | — | — | — | — | — |
| Deep Learning (CNN/LSTM) | — | — | — | — | — |
| DistilBERT | — | — | — | — | — |

The values should be populated after training and evaluation.

## Recommended Project Structure

```text
PhishNet-Transformer/
│
├── data/
│   ├── raw/
│   │   └── phiusiil.csv
│   │
│   └── processed/
│       ├── train.csv
│       ├── val.csv
│       └── test.csv
│
├── notebooks/
│   ├── data_preprocessing.ipynb
│   ├── classic_ml.ipynb
│   ├── deep_learning.ipynb
│   └── distilbert.ipynb
│
├── models/
│   ├── classic_ml/
│   ├── deep_learning/
│   └── distilbert/
│
├── requirements.txt
├── README.md
└── .gitignore
```

## Installation

Create a Python environment and install the required dependencies.

```bash
pip install pandas numpy scikit-learn matplotlib seaborn
pip install tensorflow          # or: pip install torch
pip install transformers datasets
```

Additional libraries may be installed depending on the selected ML algorithms, deep learning framework, and feature engineering methods.

## Usage

### Step 1 — Download the Dataset

Download the PhiUSIIL dataset from Kaggle and place the CSV file in the project directory.

### Step 2 — Prepare the Dataset

Run the preprocessing notebook.

The notebook should generate:

```text
train.csv
val.csv
test.csv
```

### Step 3 — Train Classic ML Models

Load `train.csv`, extract URL features, train the selected machine learning models, and validate them using `val.csv`.

### Step 4 — Train Deep Learning Models

Load the same processed datasets, build a character-level vocabulary from `train.csv`, tokenize and pad the URLs, then train a CNN/LSTM/BiLSTM model, using `val.csv` for validation and early stopping.

### Step 5 — Train DistilBERT

Load the same processed datasets, tokenize the URLs using the DistilBERT tokenizer, and fine-tune DistilBERT using the training and validation sets.

### Step 6 — Final Evaluation

Evaluate the final models (Classic ML, Deep Learning, DistilBERT) using `test.csv` and compare their performance.

## Important Considerations

### Data Leakage

The test set must not be used during model training or hyperparameter tuning.

Preprocessing and feature engineering should also be designed carefully to avoid allowing information from the test set to influence the training process.

### Duplicate URLs

Duplicate URLs should be handled before splitting the dataset. Otherwise, identical URLs could appear in multiple subsets and artificially increase model performance.

### Class Balance

Phishing datasets may contain an imbalance between phishing and legitimate URLs. The preprocessing pipeline should handle class distribution appropriately.

### Label Consistency

The most important label convention throughout this project is:

```text
1 = Phishing
0 = Legitimate
```

All training, validation, testing, evaluation, and visualization code should follow this convention.

## Security and Ethics

This project is intended for **cybersecurity research, education, and defensive applications**.

Phishing URLs are untrusted data and may point to malicious websites. URLs from the dataset should not be opened or visited directly.

When experimenting with phishing datasets, use appropriate security precautions and isolated environments where necessary.

## Future Improvements

Potential future improvements include:

- Advanced URL feature engineering
- URL-specific Transformer models
- Attention-based deep learning architectures (e.g. Transformer encoder trained from scratch on characters)
- Ensemble learning
- Hyperparameter optimization
- Explainable AI
- SHAP-based feature analysis
- Adversarial URL testing
- Real-time URL classification
- REST API deployment
- Browser extension integration
- Model quantization for faster inference

## Acknowledgements

This project uses the **PhiUSIIL Phishing URL Dataset** for phishing URL classification research.

Dataset:

https://www.kaggle.com/datasets/ndarvind/phiusiil-phishing-url-dataset

## License

This project is intended for educational and research purposes.

The PhiUSIIL dataset is subject to its original license and terms of use.

---

**PhishNet-Transformer — Phishing URL Detection using Classic Machine Learning, Deep Learning, and DistilBERT.**
