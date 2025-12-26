# 🐼 Ultimate Pandas Interview Cheat Sheet

## 1. Data Structures & Inspection
**Core Objects:**
*   **Series:** 1D array, homogeneous data, immutable size (unable to chnage).
*   **DataFrame:** 2D tabular data, mutable size, heterogeneous columns.

| Command | Description |
| :--- | :--- |
| `df.shape`, `df.columns` | Dimensions and column names. |
| `df.info()` | **Crucial.** Checks non-null counts and Dtypes (memory usage). |
| `df.describe()` | Summary statistics (mean, std, min, max) for numeric cols. |
| `df['col'].value_counts()` | Unique value counts (most useful for categorical analysis). |
| `df['col'].unique()` | Array of unique values. |
| `df['col'].nunique()` | Count of unique values. |

### Summary Table

| Method | Question it Answers | Returns | Handles NaN |
| :--- | :--- | :--- | :--- |
| **`.unique()`** | Returns an array of unique labels or values | **Array** (List) | Keeps it |
| **`.nunique()`** | Count number of unique values | **Integer** (Count) | Drops it |
| **`.value_counts()`**| Counts the frequency of each unique value | **Series** (Distribution)| Drops it |

### 💡 Interview Tip (The "NaN" Trap)
Interviewers often ask: *"How do I check unique values including Nulls?"*
*   `unique()` does it automatically.
*   `nunique(dropna=False)` forces it to count Nulls.
*   `value_counts(dropna=False)` forces it to list Null counts.

---

## 2. Selection & Filtering (The "Big 3")

### loc vs iloc (Classic Question)
| Feature | `loc` (Label Based) | `iloc` (Integer Based) |
| :--- | :--- | :--- |
| **Syntax** | `df.loc[row_label, col_label]` | `df.iloc[row_idx, col_idx]` |
| **Slicing** | Inclusive `['A':'C']` includes 'C' | Exclusive `[0:3]` excludes 3 |
| **Boolean** | Supports boolean masks | No boolean masks |

### Filtering Methods
```python
# 1. Boolean Indexing (Standard)
mask = (df['age'] > 25) & (df['dept'] == 'IT')
df[mask]

# 2. Query Method (SQL-like syntax - cleaner for complex logic)
df.query('age > 25 and dept == "IT"')

# 3. Is In (List filtering)
df[df['city'].isin(['NY', 'LA', 'SF'])]

# 4. String Accessor
df[df['email'].str.contains('@gmail.com')]
```

---

## 3. Data Cleaning & Transformation

### Handling Missing Data (`NaN`)
*   **Detection:** `df.isna().sum()` (Count per column).
*   **Removal:** `df.dropna(subset=['col1'], how='any')`.
*   **Imputation:** `df.fillna({'age': df.age.mean(), 'city': 'Unknown'})`.

### The `map`, `apply`, `applymap` Confusion
| Method | Target | Use Case | Example |
| :--- | :--- | :--- | :--- |
| **`map()`** | **Series** | Dictionary mapping or simple func | `df['sex'].map({'M': 0, 'F': 1})` |
| **`apply()`** | **Series/DF** | Complex logic on rows/cols | `df.apply(lambda row: row.a + row.b, axis=1)` |
| **`applymap()`** | **DataFrame** | Element-wise op on whole DF | `df.applymap(lambda x: len(str(x)))` |

### Replacing & Type Casting
```python
# Rename columns
df.rename(columns={'old_name': 'new_name'}, inplace=True)

# Change Data Type (Crucial for memory)
df['price'] = df['price'].astype('float32')
df['date'] = pd.to_datetime(df['date'])
```

---

## 4. Advanced Grouping & Aggregation
**The Concept:** Split $\rightarrow$ Apply $\rightarrow$ Combine.

```python
# 1. Basic Groupby
df.groupby('dept')['salary'].mean()

# 2. Multiple Aggregations (agg) - VERY IMPORTANT
df.groupby('dept').agg({
    'salary': ['mean', 'max'],
    'age': 'min'
})

# 3. Transformation (Returns object of same size as input)
# Use case: Fill missing values with group mean
df['salary'] = df.groupby('dept')['salary'].transform(lambda x: x.fillna(x.mean()))

# 4. Filter Groups
# Use case: Keep only departments with > 5 employees
df.groupby('dept').filter(lambda x: len(x) > 5)
```

---

## 5. Reshaping & Pivoting (Missing in Basics)

### 1. Pivot Table (Excel style)
Calculates aggregations.
```python
# Rows: Date, Cols: City, Values: Sales
pt = df.pivot_table(index='date', columns='city', values='sales', aggfunc='sum')
```

### 2. Melt (Wide to Long)
Unpivots a DataFrame. Essential for visualization libraries (Seaborn/Plotly).
```python
# Before: [Date, Apple_Stock, Google_Stock]
# After:  [Date, Ticker, Price]
pd.melt(df, id_vars=['Date'], value_vars=['Apple', 'Google'], var_name='Ticker', value_name='Price')
```

### 3. Stack/Unstack
*   **Stack:** Moves columns to index (Wide $\to$ Long).
*   **Unstack:** Moves index to columns (Long $\to$ Wide).

### 4. Explode (Handling Lists)
If a cell contains `[A, B]`, explode creates two rows: one for A, one for B.
```python
df.explode('tags')
```

---

## 6. Merging & Combining (SQL logic)

```python
# 1. Merge (SQL JOIN)
# how: 'inner', 'left', 'right', 'outer', 'cross'
pd.merge(df1, df2, on='key_col', how='left')

# 2. Concat (Stacking)
# axis=0 (vertical stack), axis=1 (horizontal paste)
pd.concat([df1, df2], axis=0) 
```
*Interview Tip:* If merging on index, use `left_index=True, right_index=True`.

---

## 7. Time Series & Window Functions

```python
# 1. Resampling (Changing frequency)
# Calculate monthly average from daily data
df.set_index('date').resample('M')['sales'].mean()

# 2. Shift (Lagging)
# Use case: Calculate Day-over-Day growth
df['prev_day_sales'] = df['sales'].shift(1)
df['growth'] = (df['sales'] - df['prev_day_sales']) / df['prev_day_sales']

# 3. Rolling (Moving Window)
# Use case: 7-day Moving Average
df['sales'].rolling(window=7).mean()

# 4. Expanding (Cumulative)
# Use case: Running total
df['sales'].expanding().sum()
```

---

## 8. Performance & Optimization (The "Senior" Questions)

### Q: How do you handle large datasets in Pandas?
1.  **Chunking:** Read file in pieces using `read_csv(chunksize=1000)`.
2.  **Optimize Dtypes:**
    *   `int64` $\to$ `int32` or `int8`.
    *   `float64` $\to$ `float32`.
    *   **Objects (Strings)** $\to$ **Category** (if low cardinality).
3.  **Use Vectorization:** Avoid loops; use built-in pandas/numpy functions.

### The "Category" Dtype Trick
If a column has few unique values (e.g., "Low", "Med", "High") repeated millions of times, converting to Category saves massive memory.
```python
print(df['grade'].memory_usage()) 
df['grade'] = df['grade'].astype('category')
print(df['grade'].memory_usage()) # drastically lower
```

---

## 🧠 Trick Questions / Edge Cases

1.  **`View` vs `Copy` Warning:**
    *   *Problem:* `df[mask]['col'] = 5`. Pandas doesn't know if you want to modify the original DF or the slice.
    *   *Fix:* Use `.loc` $\to$ `df.loc[mask, 'col'] = 5`.
2.  **Vectorization vs. Loops:**
    *   Always prefer `df['a'] + df['b']` over `df.apply(...)` over `for row in df...`.
3.  **NaN equality:**
    *   `np.nan == np.nan` is **False**. Use `np.isnan()` or `pd.isna()` to check.
