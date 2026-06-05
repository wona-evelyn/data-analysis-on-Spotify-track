# Do Danceable Tracks Perform Better on Spotify?

**By Evelyn Na**

---

## Introduction

Spotify assigns every track a "danceability" score from 0 to 1, measuring how suitable it is for dancing based on tempo, rhythm stability, and beat strength. But does being more danceable actually make a track more popular? And does this relationship differ across genres like classical, hip-hop, metal, jazz, and pop?

This project uses a dataset of 114,000 Spotify tracks spanning 114 genres to investigate: **Do more danceable tracks tend to be more popular, and does this relationship vary across genres?**

This question matters because Spotify's recommendation algorithms and playlist curators rely heavily on audio features. Understanding which features correlate with popularity could reveal meaningful differences in how listeners across genres engage with music.

The dataset contains 114,000 rows, where each row represents one track. The relevant columns for our analysis are:

| Column | Description |
|---|---|
| `popularity` | Spotify popularity score (0–100) based on total plays and recency |
| `danceability` | How suitable the track is for dancing (0-1) |
| `energy` | Perceptual intensity and activity of the track (0-1) |
| `valence` | Musical positiveness - high = happy, low = sad/angry (0-1) |
| `loudness` | Overall loudness in decibels (dB) |
| `acousticness` | Confidence that the track is acoustic (0-1) |
| `track_genre` | Genre label assigned by Spotify |
| `explicit` | Whether the track contains explicit content |
| `release_date` | Release date (format varies: YYYY-MM-DD, YYYY-MM, or YYYY) |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

We performed the following cleaning steps before analysis:

1. Dropped the unnamed index column (`Unnamed: 0`) that was an artifact of the CSV export.
2. Filtered to 5 musically distinct genres: classical, hip-hop, metal, jazz, and pop - resulting in **4,795 tracks**. These genres were chosen because they represent very different sonic profiles, as confirmed by audio feature analysis.
3. Parsed `release_date` (mixed formats) to extract `release_year` and `decade`.
4. Merged with `artists.csv` on the primary artist name to obtain follower counts and artist-level popularity.
5. Dropped duplicate tracks by `track_id`.

After cleaning, `tempo` had 1,053 missing values. All other relevant columns including `danceability` and `popularity` were fully complete.

Here are the first few rows of the cleaned dataset:

| track_name | artists | track_genre | popularity | danceability | release_year |
|---|---|---|---|---|---|
| Zara Zara | Bombay Jayashri | classical | 58 | 0.64 | 2001.0 |
| Kajra Re | Shankar;Ehsaan;Loy | classical | 59 | 0.48 | 2005.0 |
| Zara Zara - Lofi | Bombay Jayashri;DJ Aftab | classical | 54 | 0.61 | 1984.0 |
| Vaseegara | Bombay Jayashri | classical | 68 | 0.69 | 1972.0 |
| Zara Zara - LoFi Chill | Bombay Jayashri;Swattrex | classical | 59 | 0.58 | 1987.0 |

### Univariate Analysis

We first looked at the distribution of danceability across our five genres. Hip-hop tracks cluster at high danceability values (0.6-0.9), while classical tracks are concentrated at lower values (0.2-0.5). This confirms that our genre choices are musically distinct and span a wide range of danceability.

<iframe
  src="assets/danceability_dist.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

### Bivariate Analysis

The scatter plot below shows the relationship between danceability and popularity for each genre, with trend lines. The relationship varies noticeably across genres - classical shows a slight positive trend, while hip-hop shows a slightly negative one. This suggests that genre moderates the danceability-popularity relationship, which motivates our hypothesis test.

<iframe
  src="assets/dance_vs_pop.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

### Interesting Aggregates

The table below shows mean audio features grouped by genre. Hip-hop has the highest danceability (0.74) but not the highest popularity. Pop has both high danceability (0.62) and the highest mean popularity (45.26). Classical and jazz have the lowest popularity scores despite their distinct audio profiles - suggesting danceability alone doesn't fully explain popularity.

| track_genre | mean_danceability | mean_popularity | mean_energy | mean_valence | track_count |
|---|---|---|---|---|---|
| classical | 0.38 | 13.52 | 0.20 | 0.38 | 933 |
| hip-hop | 0.74 | 38.08 | 0.68 | 0.55 | 991 |
| jazz | 0.51 | 13.62 | 0.35 | 0.49 | 989 |
| metal | 0.47 | 43.70 | 0.84 | 0.42 | 992 |
| pop | 0.62 | 45.26 | 0.60 | 0.51 | 890 |

---

## Assessment of Missingness

### NMAR Analysis

We believe the `tempo` column is likely **NMAR (Not Missing At Random)**. Spotify's tempo detection algorithm may fail on tracks with non-standard rhythms, silence, or very short duration - meaning the missingness of `tempo` is directly related to the track's own audio characteristics, which we cannot observe in the dataset. To make this MAR, we would need additional data such as raw audio quality indicators or a flag for whether the track has a detectable beat.

### Missingness Dependency

We tested whether `tempo` missingness depends on other columns using permutation tests (500 iterations, significance level = 0.05).

**Test 1: Does tempo missingness depend on `energy`?**

- **Null Hypothesis:** The distribution of `energy` is the same whether `tempo` is missing or not.
- **Alternative Hypothesis:** Tracks with missing `tempo` have lower mean energy.
- **Observed difference:** -0.185 | **P-value:** 0.0

We reject the null hypothesis. Tracks with missing tempo have significantly lower energy, which makes sense - Spotify's beat detection likely struggles with quiet, low-energy tracks like ambient classical music.

<iframe
  src="assets/missingness_energy.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

**Test 2: Does tempo missingness depend on `duration_ms`?**

- **Null Hypothesis:** The distribution of `duration_ms` is the same whether `tempo` is missing or not.
- **Alternative Hypothesis:** Track duration differs between tracks with and without missing tempo.
- **Observed difference:** -6719.83 | **P-value:** 0.136

We fail to reject the null hypothesis. Track length does not predict whether tempo is missing - the observed difference falls well within the range of random chance.

---

## Hypothesis Testing

**Null Hypothesis:** The mean popularity of high-danceability tracks (danceability ≥ 0.7) is the same as low-danceability tracks. Any observed difference is due to random chance.

**Alternative Hypothesis:** High-danceability tracks have higher mean popularity than low-danceability tracks.

**Test Statistic:** Difference in mean popularity (high − low danceability). We chose this because our hypothesis is directional and both groups are large enough for the mean to be a stable estimator.

**Significance Level:** 0.05

**Result:** The observed difference in mean popularity was **6.179**. Using a permutation test with 500 iterations, the p-value was **0.0**.

**Conclusion:** Since p-value < 0.05, we reject the null hypothesis. The data is consistent with high-danceability tracks tending to have higher mean popularity. However, since this is an observational study and not a randomized controlled trial, we cannot conclude that danceability directly causes higher popularity - other factors such as genre, artist popularity, or release year may also contribute.

<iframe
  src="assets/hypothesis_test.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

---

## Framing a Prediction Problem

From the EDA, we found that danceability and genre are both associated with popularity. This raises a natural prediction question: **can we predict whether a track will be popular based on its audio features?**

**Prediction Problem:** Predict whether a Spotify track is "popular" (popularity ≥ 70) - **binary classification**.

**Response Variable:** `popular` (1 = popular, 0 = not popular). We chose the threshold of 70 because it represents the top ~14% of tracks, aligning with what most listeners would consider genuinely popular.

**Evaluation Metric:** F1-score. The dataset is heavily imbalanced (86% not popular, 14% popular), so accuracy would be misleading - a model that always predicts "not popular" would achieve 86% accuracy. F1-score balances precision and recall for the minority class.

**Features at time of prediction:** All audio features (danceability, energy, valence, etc.) and metadata (genre, explicit, release_year) are properties of the track itself and are known at release time. We do not use `popularity` or any feature derived from it, avoiding data leakage.

---

## Baseline Model

Our baseline model is a **Random Forest Classifier** implemented in a single sklearn `Pipeline`.

**Features used:**
- `danceability` - quantitative
- `energy` - quantitative
- `track_genre` - nominal (OneHotEncoded, 5 categories)
- `explicit` - nominal (OneHotEncoded, 2 categories)

The quantitative features were passed through as-is. The nominal features were encoded with `OneHotEncoder`.

**Performance on test set (20% holdout):**

| | Precision | Recall | F1-score | Support |
|---|---|---|---|---|
| Not popular (0) | 0.88 | 0.95 | 0.92 | 820 |
| Popular (1) | 0.48 | 0.26 | **0.34** | 139 |

**Overall F1-score: 0.3364**

We do not consider this a good model. A recall of 0.26 for the popular class means the model misses 74% of actually popular tracks. This is expected for a simple baseline with limited features and no handling of class imbalance.

---

## Final Model

We improved upon the baseline by engineering new features and tuning hyperparameters.

**New features added and why:**
- `valence`: Happier-sounding tracks tend to be more commercially appealing and are playlisted more frequently, which drives up Spotify play counts.
- `loudness`: Louder, more punchy tracks tend to feel more engaging and are streamed more heavily in mainstream genres.
- `acousticness`: Acoustic tracks tend to be less mainstream, making this a useful negative signal for predicting high popularity.
- `release_year`: Spotify's popularity score is recency-biased - newer tracks accumulate plays faster, so release year captures this temporal effect directly.

We also applied `StandardScaler` to all numeric features and used `class_weight='balanced'` to force the model to pay more attention to the minority (popular) class.

**Hyperparameter tuning** used `GridSearchCV` with 5-fold cross-validation (scoring = F1):

| Hyperparameter | Values tested | Best value |
|---|---|---|
| `max_depth` | 5, 10, None | 10 |
| `n_estimators` | 100, 200 | 200 |
| `min_samples_split` | 2, 5 | 5 |

**Final Model Performance:**

| | Precision | Recall | F1-score | Support |
|---|---|---|---|---|
| Not popular (0) | 0.93 | 0.80 | 0.86 | 820 |
| Popular (1) | 0.36 | 0.67 | **0.47** | 139 |

**Overall F1-score: 0.4709** - an improvement of **+0.1345** over the baseline.

The most meaningful improvement is in recall for popular tracks: from 0.26 to 0.67, meaning the model now correctly identifies 67% of popular tracks instead of only 26%.

---

## Fairness Analysis

**Question:** Does our model perform worse for older tracks than newer tracks?

**Groups:**
- **Group X (old tracks):** released before 2000 - 504 tracks in test set
- **Group Y (new tracks):** released in 2000 or after - 455 tracks in test set

**Evaluation Metric:** F1-score

**Null Hypothesis:** Our model is fair. Its F1-score for old and new tracks is roughly the same; any difference is due to random chance.

**Alternative Hypothesis:** Our model performs better on new tracks than old tracks.

**Test Statistic:** Difference in F1-score (new − old) | **Significance Level:** 0.05

**Results:**
- F1 for new tracks: **0.5068**
- F1 for old tracks: **0.3636**
- Observed difference: **0.1431**
- P-value: **0.006**

**Conclusion:** Since p-value (0.006) < 0.05, we reject the null hypothesis. The model performs significantly better on newer tracks. This is likely because Spotify's popularity score is recency-biased - newer popular tracks have higher and more consistent popularity scores, making them easier for the model to identify. This suggests a fairness disparity by release year that would need to be addressed in a production model.

<iframe
  src="assets/fairness.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>