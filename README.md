# 🎬 CineMatch — Content-Based Movie Recommendation Engine

CineMatch is a content-based movie recommender system that suggests the top 5 most similar movies to any film a user picks. It uses NLP feature engineering on movie metadata (overview, genres, keywords, cast, and director) combined with **CountVectorizer** and **cosine similarity** to compute similarity scores across ~4,800 movies from the TMDB 5000 dataset. The recommendations are served through a custom-themed, dark-mode **Streamlit** web app with live poster fetching via the OMDb API.

---

## 📸 Screenshots

| Dashboard | Recommendations |
|---|---|
| ![Dashboard](Screenshots/Dashboard.png) | ![Recommendations](Screenshots/Recommendations.png) |

---

## ✨ Features

- **Content-based filtering** — recommends movies based on plot overview, genres, keywords, top 3 cast members, and director rather than user ratings/history.
- **Vectorized similarity search** — Bag-of-Words vectorization (5,000 max features, English stop-words removed) + cosine similarity for fast nearest-neighbor lookup.
- **Live poster fetching** — pulls movie posters dynamically from the OMDb API at request time.
- **Custom Streamlit UI** — dark, cinematic theme with a hero header, styled movie cards, and a responsive 5-column results grid.
- **Cached data loading** — uses `st.cache_data` so the ~4,800 x 4,800 similarity matrix loads once per session.

---

## 🧠 How It Works

1. **Data merge** — `tmdb_5000_movies.csv` and `tmdb_5000_credits.csv` are merged on `title`.
2. **Feature selection** — Keeps `movie_id`, `title`, `overview`, `genres`, `keywords`, `cast`, `crew`.
3. **Feature engineering**:
   - `genres` / `keywords` → extracted from JSON-like strings via `ast.literal_eval`.
   - `cast` → top 3 billed actors only.
   - `crew` → director's name only.
   - All multi-word names are collapsed (e.g. `Science Fiction` → `ScienceFiction`) so the vectorizer treats them as single tokens.
4. **Tag creation** — overview + genres + keywords + cast + crew are combined into a single `tags` string per movie.
5. **Vectorization** — `CountVectorizer(max_features=5000, stop_words='english')` converts tags into numerical vectors.
6. **Similarity computation** — `cosine_similarity` computes a similarity score between every pair of movies.
7. **Recommendation** — for a selected movie, the 5 highest-scoring (non-self) movies are returned.

---

## 🗂️ Project Structure

```
filmlens-recommender/
│
├── Screenshots/
│   ├── Dashboard.png
│   └── Recommendations.png
│
├── movie_recommendation_train.py   # Data preprocessing + model training script
├── movie_recommender_streamlit.py  # Streamlit web application
├── secrets.toml.example            # Template for your local OMDb API key (copy → .streamlit/secrets.toml)
├── requirements.txt
├── .gitignore
└── README.md
```

> **Note:** `tmdb_5000_movies.csv`, `tmdb_5000_credits.csv`, `movie_dict.pkl`, and `similarity.pkl` are **not included** in this repository due to GitHub file-size limits. See setup instructions below to generate them locally.
>
> **Security note:** No API key is stored in this repo. `movie_recommender_streamlit.py` loads the OMDb key from Streamlit secrets or an environment variable at runtime — see step 5 below.

---

## ⚙️ Setup & Installation

### 1. Clone the repository
```bash
git clone https://github.com/Chowdri-Furkhan07/filmlens-recommender.git
cd filmlens-recommender
```

### 2. Install dependencies
```bash
pip install -r requirements.txt
```

### 3. Download the dataset
This project uses the **TMDB 5000 Movie Dataset** from Kaggle:
🔗 https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata

Download `tmdb_5000_movies.csv` and `tmdb_5000_credits.csv` and place them in the project's root folder.

### 4. Generate the model files
Run the training script to preprocess the data and build the similarity matrix:
```bash
python movie_recommendation_train.py
```
This creates `movie_dict.pkl` and `similarity.pkl` locally (both are `.gitignore`-excluded since they're too large for GitHub).

### 5. Set up your OMDb API key
Posters are fetched live from the [OMDb API](https://www.omdbapi.com/apikey.aspx) (free tier available). The app never hardcodes a key — it reads from Streamlit secrets first, then falls back to an environment variable, so it's safe to keep this repo public.

**Local development:**
```bash
mkdir -p .streamlit
cp secrets.toml.example .streamlit/secrets.toml
# then edit .streamlit/secrets.toml and paste in your real key
```

**Or, using an environment variable instead:**
```bash
export OMDB_API_KEY="your_omdb_api_key_here"      # macOS/Linux
setx OMDB_API_KEY "your_omdb_api_key_here"        # Windows
```

**Deploying to Streamlit Community Cloud:** add `OMDB_API_KEY` under your app's **Settings → Secrets** in the same TOML format as `secrets.toml.example`.

### 6. Launch the app
```bash
streamlit run movie_recommender_streamlit.py
```

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Language | Python |
| Data Processing | Pandas, NumPy |
| ML / NLP | Scikit-learn (CountVectorizer, Cosine Similarity) |
| Web App | Streamlit |
| External API | OMDb API (poster fetching) |
| Data Source | TMDB 5000 Movie Dataset (Kaggle) |

---

## 📈 Future Improvements

- Switch from CountVectorizer to **TF-IDF** or **sentence embeddings** for richer semantic similarity.
- Add a **hybrid recommender** (content-based + collaborative filtering) once user rating data is available.
- Host the similarity matrix in a lightweight vector database (e.g., FAISS) to avoid large `.pkl` files entirely.
- Deploy on Streamlit Community Cloud with the dataset regenerated on first run.

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 👤 Author

**Chowdri Furkhan**

🔗 [LinkedIn](https://www.linkedin.com/in/chowdri-furkhan/) · [GitHub](https://github.com/Chowdri-Furkhan07)
