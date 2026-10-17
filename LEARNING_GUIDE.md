# Learning Guide – Explained from Zero

This guide walks through the notebook step by step. Open `kpop_project.ipynb` next to it and read each part alongside the matching cell.

---

## Part 1: Basic concepts

**Colab** – a Google website that runs Python code in the browser, with nothing to install. A notebook is made of **cells**:
- **Text cell** – explanations.
- **Code cell** – code you run with ▶️ or Shift+Enter.

Cells run in order. If cell 5 uses a variable created in cell 3, cell 3 has to run first.

**Kaggle** – a website where people share datasets for free.

**CSV file** – a table saved as text. Each line is a row, and the values are separated by commas:

```
Artist,Track,Views
BTS,Dynamite,1640946000
BLACKPINK,DDU-DU DDU-DU,2012855000
```

**Pandas** – a Python library for working with tables. A table in Pandas is called a **DataFrame**. Think of it as an Excel sheet inside your code.

---

## Part 2: The code, step by step

### Step 1: Importing libraries and downloading the data

```python
import kagglehub
import pandas as pd
import matplotlib.pyplot as plt
```

`import` loads a library so we can use it.
- `kagglehub` downloads data from Kaggle.
- `pandas` handles tables. `as pd` gives it a short nickname, so we write `pd` instead of `pandas`.
- `matplotlib.pyplot` draws charts. Its nickname is `plt`.

```python
pd.options.display.float_format = "{:,.0f}".format
```

This only changes how numbers are shown: `2,012,854,627` instead of `2.01e+09`.

```python
path_groups = kagglehub.dataset_download("nicolsalayoarias/kpop-groups-dataset")
```

Downloads the dataset and saves the folder it went to in the **variable** `path_groups`. A variable is a name that holds a value, like a labeled box.

---

### Step 2: Loading the table

```python
songs = pd.read_csv(path_spotify_youtube + "/Spotify_Youtube.csv")
```

`read_csv` reads the file and turns it into a DataFrame. The `+` joins the folder name and the file name into the full path.

```python
print("Rows and columns:", songs.shape)
songs.head()
```

- `shape` gives the size of the table, here `(20718, 28)`: 20,718 rows and 28 columns.
- `head()` shows the first 5 rows. It's how you take a quick look at the data.

---

### Step 3: Choosing columns

```python
songs = songs[["Artist", "Track", "Album", "Views", "Likes", "Stream"]]
```

Inside the brackets is a **list** of column names, which is why there are double brackets `[[ ]]`. The result is a table with only these 6 columns. We save it back into `songs`, replacing the old table.

**Remember:**
- `songs["Views"]` with single brackets returns **one column**.
- `songs[["Views", "Track"]]` with double brackets returns **a table** with several columns.

---

### Step 4: Filtering – K-pop only

```python
kpop_artists = ["BTS", "BLACKPINK", "TWICE", ...]
```

A **list** in Python is written in square brackets, with values separated by commas.

```python
kpop = songs[songs["Artist"].isin(kpop_artists)]
```

This is the most important line in this step. Read it from the inside out:
1. `songs["Artist"]` takes the artist column.
2. `.isin(kpop_artists)` checks each row: is the artist in the list? It returns True or False.
3. `songs[...]` keeps only the rows that got True.

This is called **filtering**. Another example of the same idea:

```python
songs[songs["Views"] > 1000000000]   # only songs with more than a billion views
```

```python
kpop["Artist"].nunique()
```

`nunique` counts how many **different** values a column has. The result is 23 artists.

---

### Step 5: Cleaning the data

```python
kpop.isna().sum()
```

- `isna()` checks each cell: is it empty? (NaN = Not a Number, a missing value.)
- `sum()` counts the empty cells in each column.

```python
kpop = kpop.dropna(subset=["Views"])
```

`dropna` removes rows with missing values. `subset=["Views"]` means only remove rows where the view count is missing. A song without views doesn't help answer our question.

---

### Step 6: The 10 most viewed songs

```python
top10 = kpop.sort_values("Views", ascending=False).head(10)
```

- `sort_values("Views")` sorts by the views column.
- `ascending=False` sorts from largest to smallest. `True` sorts from smallest to largest.
- `head(10)` takes the first 10 rows.

You can chain actions with dots. Each action runs on the result of the one before it.

---

### Step 7: The chart

```python
labels = top10["Artist"] + " - " + top10["Track"]
```

Joins the artist and song name into one label, for example `"BTS - Dynamite"`.

```python
plt.figure(figsize=(10, 5))                                    # chart size
plt.barh(labels, top10["Views"] / 1_000_000, color="hotpink")  # horizontal bar chart
plt.gca().invert_yaxis()                                       # number 1 at the top
plt.xlabel("Views (millions)")                                 # axis title
plt.title("Top 10 K-pop songs on YouTube")                     # chart title
plt.show()                                                     # show the chart
```

- `barh` draws horizontal bars (h = horizontal), so long song names are easy to read.
- We divide by 1,000,000 to show the views in millions.

---

### Step 8: groupby – the most viewed song of each group

This is the most important concept in the project.

**`groupby` splits the table into groups, then runs a calculation on each group separately.**

```python
best_rows = kpop.groupby("Artist")["Views"].idxmax()
```

1. `groupby("Artist")` makes one group per artist: all of BTS's songs, all of TWICE's songs, and so on.
2. `["Views"]` focuses on the views column.
3. `idxmax()` returns, for each group, the **row number** of the song with the most views.

```python
best_song_per_artist = kpop.loc[best_rows]
```

`loc` pulls rows by their row number. This gives us the full rows of the winning songs.

The next step uses the same idea with a different calculation:

```python
kpop.groupby("Artist")["Views"].sum()
```

Here each group's views are **added up**, giving the total views of each group.

Common calculations after `groupby`:

| Function | What it calculates |
|---|---|
| `sum()` | Total |
| `mean()` | Average |
| `max()` / `min()` | Largest / smallest value |
| `size()` | How many rows are in each group |
| `idxmax()` | Row number of the largest value |

---

### Step 9: merge – joining two tables

Each table has different information:
- **kpop:** group, song, views.
- **groups:** group, record label, debut date.

```python
kpop_with_info = kpop.merge(groups, left_on="Artist", right_on="Name", how="left")
```

- `merge` joins the tables using a shared column – the group name.
- `left_on="Artist"`: in the left table (kpop), the group name is in the `Artist` column.
- `right_on="Name"`: in the right table (groups), the group name is in the `Name` column.
- `how="left"`: keep **all** songs from the left table, even if they have no match in the other table.

Solo artists like PSY have no match, because the second table only lists groups. So they get `NaN`.

```python
kpop_with_info = kpop_with_info.drop(columns="Name")
```

`drop` removes the `Name` column, because it's a duplicate of `Artist`.

---

### Step 10: The third table – most popular song of each year

In the chart table, each row says: "In year X, in week Y, song Z was at position N."

```python
number_one = rankings[rankings["rank"] == 1]
```

Filtering: keep only the rows for position 1. Notice the `==` (two equal signs), which means "is it equal to?". A single `=` means "save into a variable".

```python
weeks_at_top = (
    number_one.groupby(["year", "song_title", "artist"])
    .size()
    .reset_index(name="weeks_at_1")
    .sort_values(["year", "weeks_at_1"], ascending=[True, False])
)
```

1. `groupby` on 3 columns: year, song and artist.
2. `size()` counts the rows in each group, which is how many weeks the song was at number 1.
3. `reset_index(name=...)` turns the result back into a regular table and names the new column.
4. `sort_values` sorts by year from oldest to newest, and within each year by weeks from most to least.

```python
weeks_at_top.groupby("year").head(1)
```

Takes the first row of each year, which is that year's top song.

---

### Step 11: Functions

A **function** is a piece of code with a name, which you can run again and again with different values.

```python
def most_popular_in_year(year):
    result = weeks_at_top[weeks_at_top["year"] == year]
    if result.empty:
        print("❌ No data for", year)
        return
    best = result.iloc[0]
    print(f"🏆 Most popular song of {year}: {best['song_title']}")
```

- `def` defines a function. `year` is a **parameter**, the value the function receives.
- `if result.empty:` checks whether the filter found nothing. If so, it prints a message and `return` exits the function.
- `iloc[0]` takes the first row by position.
- `f"...{year}..."` is an **f-string**: anything inside `{ }` is replaced by the variable's value.

Running it: `most_popular_in_year(2020)`

**The `find_artist` function** lets the user type a group name in upper or lower case:

```python
for artist in kpop["Artist"].unique():
    if artist.lower() == name.strip().lower():
        return artist
return None
```

- `for` is a **loop** that goes over every artist in the table.
- `lower()` turns text into lower case, so "blackpink" and "BLACKPINK" count as the same.
- `strip()` removes extra spaces at the start and end.
- If the artist isn't found, the function returns `None`, which means "nothing".

**The `korean_names` dictionary:**

```python
korean_names = {"BTS": "방탄소년단", ...}
```

A **dictionary** holds key–value pairs. In the Korean chart, BTS's name is written in Korean, so the dictionary translates the name before searching.

---

### Step 12: The menu

```python
def menu():
    while True:
        print("1 - Which song of a group got the most views?")
        ...
        choice = input("Choose a number: ").strip()

        if choice == "1":
            most_viewed_song(input("Group name: "))
        elif choice == "2":
            ...
        elif choice == "0":
            break
        else:
            print("❌ That option doesn't exist")
```

- `while True:` is an endless loop, so the menu keeps coming back.
- `input()` stops and waits for the user to type something. The input always arrives as **text**.
- `if / elif / else` choose what to do based on the number picked.
- `break` exits the loop, which ends the program.

For the year question, the code first checks `year.isdigit()`, meaning the input is a number. Then `int(year)` turns the text `"2020"` into the number `2020`.

---

## Part 3: Cheat sheet

| Command | What it does |
|---|---|
| `pd.read_csv("file.csv")` | Load a file into a table |
| `df.head()` | Show the first 5 rows |
| `df.shape` | Table size (rows, columns) |
| `df["col"]` | One column |
| `df[["a", "b"]]` | Several columns |
| `df[df["col"] > 5]` | Filter rows by a condition |
| `df["col"].isin(list)` | Is the value in the list? |
| `df.isna().sum()` | Count missing values |
| `df.dropna()` | Remove rows with missing values |
| `df.sort_values("col", ascending=False)` | Sort from largest to smallest |
| `df.groupby("col")["x"].sum()` | Total per group |
| `df.merge(other, ...)` | Join two tables |
| `df["col"].nunique()` | Count different values |
| `df.iloc[0]` | The first row |

---

## Part 4: Questions you might be asked

1. **Why these datasets?** They include YouTube views, Spotify streams and an official chart, which is exactly what's needed to answer the research question.
2. **What's the difference between `sum` and `mean` after `groupby`?** `sum` gives a group's total views. `mean` gives the average per song.
3. **Why is there NaN after the merge?** Solo artists have no row in the groups table, and `how="left"` keeps them anyway.
4. **What are the data's limitations?** The views were collected in early 2023, each artist has only about 10 songs, and some years are missing chart weeks.

---

## Tip

Don't just read – change something small and see what happens. For example, change `head(10)` to `head(3)`, `ascending=False` to `True`, or `sum()` to `mean()`. That's the fastest way to understand what each command does.
