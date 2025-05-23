# Conversational Recommender System Project

## Project Overview

This project explores the development of a sophisticated conversational recommender system, focusing on leveraging historical newsletter data for product upselling and cross-selling. The primary context is e-commerce clients in South East Asia, dealing with multiple languages. The project demonstrates a pipeline from data extraction and synthetic data generation to user profiling, topic modeling, and recommender system evaluation, culminating in a prototype conversational agent.

The project is broken down into several Jupyter Notebooks, each tackling a specific part of a given AI research assignment.

## Project Structure

```
conversational-reco/
├── .env                    # Environment variables (API keys) - DO NOT COMMIT TO GIT
├── .gitignore              # Specifies intentionally untracked files
├── data/
│   ├── processed/          # Processed data (e.g., extracted metadata CSVs, synthetic data CSVs)
│   └── raw/                # Raw data (e.g., newsletter HTML examples)
├── fonts/                  # Directory for font files (e.g., for WordCloud)
│   └── NotoSansThai-Regular.ttf # Example font, user needs to add this
├── notebooks/              # Jupyter notebooks for each stage
│   ├── 01_newsletter_product_extraction.ipynb
│   ├── 02_synthetic_product_data_generation.ipynb
│   ├── 03_user_interest_profiling_and_topic_modeling.ipynb
│   ├── 04_recbole_offline_evaluation_v2.ipynb
│   └── 05_conversational_movie_recommendation.ipynb
├── recbole_data/           # Data used/generated by RecBole (e.g., ml-100k)
├── recbole_results/        # Evaluation results from RecBole experiments
├── recbole_saved_models/   # Saved RecBole model checkpoints
├── results/
│   └── figures/            # Saved visualizations (e.g., word clouds, embedding plots)
├── README.md               # This file
└── requirements.txt        # Python package dependencies
```

## Notebook Descriptions

1.  **`01_newsletter_product_extraction.ipynb`**:
    * **Purpose:** Extracts structured product metadata (product names, categories, prices, URLs) from sample HTML newsletter content (Konvy, Uniqlo, Gramedia) using the Gemini API.
    * **Output:** Stores the extracted metadata in a Pandas DataFrame and saves it to `data/processed/extracted_newsletter_product_metadata.csv`.

2.  **`02_synthetic_product_data_generation.ipynb`**:
    * **Purpose:** Generates a synthetic dataset of product information (including product descriptions) for the three stores, mimicking the structure of the extracted metadata. Uses the Gemini API with store-specific prompts.
    * **Output:** Saves the synthetic data to `data/processed/synthetic_newsletter_product_data.csv` (50 entries per store). Also generates SBERT embeddings for product descriptions and visualizes them in 2D using t-SNE, saving the plot to `results/figures/`.

3.  **`03_user_interest_profiling_and_topic_modeling.ipynb`**:
    * **Purpose:**
        * Simulates user interactions with the synthetic product data.
        * Uses the Gemini API to infer user profiles (interests, preferred categories, summary) from these simulated interactions.
        * Performs topic modeling (BERTopic) on the synthetic product descriptions to identify overarching themes.
        * Visualizes sample user profiles and discovered topics (e.g., using word clouds).
    * **Output:** Saves generated user profiles to `data/processed/synthetic_user_profiles.csv` and topic visualizations to `results/figures/`.

4.  **`04_recbole_offline_evaluation_v2.ipynb`**:
    * **Purpose:** Performs offline evaluation of several recommendation algorithms (Random, Pop, BPR, LightGCN, ItemKNN) on the MovieLens 100k dataset using the RecBole library.
    * **Metrics:** Includes accuracy (Precision@K, Recall@K, NDCG@K, MRR@K) and beyond-accuracy metrics (ItemCoverage@K, GiniIndex@K, ShannonEntropy@K, AveragePopularity@K, TailPercentage@K).
    * **Output:** Saves trained RecBole models to `recbole_saved_models/` and evaluation results to `recbole_results/`. **Crucially, this notebook also saves the processed MovieLens 100k dataset (with ID mappings) to `recbole_data/ml-100k/` when `save_dataset: True` is configured.**
    * **Important:** This notebook contains notes on potential manual patches required for the RecBole library to resolve specific runtime errors on CPU.

5.  **`05_conversational_movie_recommendation.ipynb`**:
    * **Purpose:** Demonstrates inference using a pre-trained RecBole model (e.g., LightGCN from notebook 04). Simulates a chatbot conversation using the Gemini API to provide personalized movie recommendations with explanations based on a user's liked movies.
    * **Dependencies:** Requires a successfully trained model and processed dataset from notebook 04, and RecBole's processed `ml-100k.item` file.

## Setup Instructions

1.  **Clone the Repository (if applicable) or Create Project Directory:**
    Set up the project directory structure as outlined above.

2.  **Create a Virtual Environment (Recommended):**
    It's highly recommended to use a virtual environment (e.g., Conda or venv) to manage dependencies and avoid conflicts.
    ```bash
    # Using Conda
    conda create -n conversational_reco python=3.10 -y
    conda activate conversational_reco
    ```

3.  **Install Dependencies:**
    Install the required Python packages using the `requirements.txt` file:
    ```bash
    pip install -r requirements.txt
    ```
    *Note: `requirements.txt` includes `numpy==1.23.5` due to compatibility issues with RecBole/Ray. Ensure this version is used in the environment for notebook 04.*

4.  **Create `.env` File:**
    In the project root directory, create a file named `.env` and add your API keys:
    ```env
    GEMINI_API_KEY="YOUR_GEMINI_API_KEY_HERE"
    # OPENAI_API_KEY="YOUR_OPENAI_API_KEY_HERE" # If OpenAI is used in any custom extensions
    ```
    Replace `"YOUR_GEMINI_API_KEY_HERE"` with your actual Gemini API key. **Do not commit this file to version control.**

5.  **Download Font for Word Clouds (for Notebook 02 & 03):**
    * For word clouds to display multilingual characters correctly (especially Thai and Indonesian), a suitable font is needed.
    * Download a font like "Noto Sans Thai" or a general "Noto Sans" (which has broad Unicode coverage) from Google Fonts: [https://fonts.google.com/noto](https://fonts.google.com/noto)
    * Create a `fonts/` directory in your project root if it doesn't exist.
    * Place the downloaded `.ttf` font file (e.g., `NotoSansThai-Regular.ttf` or `NotoSans-Regular.ttf`) into the `fonts/` directory.
    * Notebook `02` and `03` (in CELL 4) will attempt to use these fonts. Update the `FONT_PATH_USER_DEFINED` variable in those notebooks if your font file has a different name or location.

6.  **RecBole Manual Patches (for Notebook 04):**
    As detailed in `04_recbole_offline_evaluation_v2.ipynb` (CELL 1), you may need to manually patch files within your installed RecBole library to resolve specific runtime errors when running on CPU. These include:
    * Patching `recbole/evaluator/collector.py` for `AttributeError: 'int' object has no attribute 'cpu'`.
    * Patching `recbole/model/general_recommender/lightgcn.py` for `AttributeError: 'dok_matrix' object has no attribute '_update'`.
    Refer to the detailed instructions within notebook 04 before running it.

7.  **MovieLens 100k Data (for Notebooks 04 & 05):**
    * Notebook `04` is configured to automatically download the MovieLens 100k dataset and save its processed version (including `ml-100k.item`, `.inter`, `.user` files and ID mappings) into the `recbole_data/ml-100k/` directory due to the `'save_dataset': True` setting.
    * Notebook `05` relies on RecBole's processed `ml-100k.item` file being present in `recbole_data/ml-100k/` for movie titles and genres. Ensure notebook 04 has run successfully to create these files.

## Running the Notebooks

1.  Ensure your virtual environment is activated.
2.  Start Jupyter Lab or Jupyter Notebook from your project root directory:
    ```bash
    jupyter lab
    # or
    # jupyter notebook
    ```
3.  Navigate to the `notebooks/` directory in the Jupyter interface.
4.  Open and run the notebooks sequentially (01 through 05), as later notebooks often depend on the outputs of earlier ones.
5.  Pay close attention to any specific instructions or prerequisites mentioned at the beginning of each notebook, especially for notebook 04 regarding RecBole patches and dataset generation.

## Key Technologies Used

* Python 3.10+
* Jupyter Notebooks
* Pandas: Data manipulation and analysis.
* Google Generative AI SDK (Gemini API): For LLM-based data extraction, synthetic data generation, and conversational agent simulation.
* Sentence-Transformers (SBERT): For generating text embeddings.
* BERTopic: For topic modeling.
* Scikit-learn: For t-SNE (dimensionality reduction).
* Matplotlib & Seaborn: For plotting and visualizations.
* WordCloud: For generating word cloud visualizations.
* RecBole: For training and evaluating recommendation models.
* NumPy, SciPy: Core scientific computing libraries.
* python-dotenv: For managing environment variables.

This README should help you get started with the project!
