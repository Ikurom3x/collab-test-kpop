# 🎤 K-pop Popularity Analysis with Pandas

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Ikurom3x/collab-test-kpop/blob/main/kpop_project.ipynb)

A school mini project that uses **Pandas** to find out which K-pop songs and groups are the most popular, based on YouTube views, Spotify streams and the official Korean music chart.

## Research question

Which K-pop song has the most YouTube views, and which song of each group has the most views?

## How to run

1. Click the **Open in Colab** button at the top of this page.
2. Choose **Runtime → Run all**.

The notebook downloads the data from Kaggle by itself. No login is needed.

To run it on your own computer instead:

```bash
pip install -r requirements.txt
jupyter notebook kpop_project.ipynb
```

## Datasets

| Dataset | What it contains |
|---|---|
| [Spotify and YouTube](https://www.kaggle.com/datasets/salvatorerastelli/spotify-and-youtube) | YouTube views, likes and Spotify streams for about 20,000 songs |
| [400+ K-Pop Groups](https://www.kaggle.com/datasets/nicolsalayoarias/kpop-groups-dataset) | Record label, debut date, gender and number of members |
| [Kpop song rankings](https://www.kaggle.com/datasets/romainfonta2/kpop-song-rankings) | Official weekly Korean chart (Circle Chart), top 200, 2010–2023 |

## What the notebook does

1. Loads the tables with `pd.read_csv`.
2. Filters the songs to 23 K-pop artists with `isin`.
3. Cleans missing values with `isna` and `dropna`.
4. Sorts and ranks songs with `sort_values`.
5. Finds the top song of each group with `groupby` and `idxmax`.
6. Joins the songs table to the groups table with `merge`.
7. Finds the most popular song of each year from the chart table.
8. Draws bar charts with `matplotlib`.
9. Runs an interactive menu where the user asks questions and gets answers.

## Key findings

- **Most viewed K-pop song:** PSY – *Gangnam Style*, with about 4.7 billion views.
- **Most viewed song by a group:** BLACKPINK – *DDU-DU DDU-DU*, with about 2 billion views.
- **Most streamed on Spotify:** BTS – *Dynamite*, with about 1.6 billion streams. The top song on YouTube isn't the top song on Spotify.
- **BLACKPINK vs. BTS:** BLACKPINK has more total YouTube views in this dataset (about 9.4 billion vs. 5.6 billion).
- **Record labels:** YG Entertainment's artists have the most total views.

## Limitations

- The YouTube and Spotify numbers were collected in early 2023, so newer songs and newer views aren't included.
- Each artist has only about 10 songs in the dataset.
- The Korean chart data ends in 2023, and some years have only one or two weeks of data.
- Some artist names in the chart table are written in Korean.

## Learning guide

[`LEARNING_GUIDE.md`](LEARNING_GUIDE.md) explains every step of the code from zero.

## Built with

Python · Pandas · Matplotlib · kagglehub · Google Colab
