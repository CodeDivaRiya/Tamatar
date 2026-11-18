## **Part 2 — Experiment 2: Clean and Preprocess a UCI Dataset**

**Aim:** To clean and preprocess the UCI Red Wine Quality dataset using Python.

Create a new Colab notebook named `Experiment_02_Wine_Quality.ipynb`.

Run these cells one by one.

### **Step 1: Import libraries**

```
import pandas as pdimport numpy as npimport matplotlib.pyplot as pltfrom sklearn.preprocessing import MinMaxScalerprint("Libraries imported successfully")
```

### **Step 2: Load the dataset**

```
url = "https://archive.ics.uci.edu/ml/machine-learning-databases/wine-quality/winequality-red.csv"df = pd.read_csv(url, sep=";")print("Dataset loaded successfully")print("Dataset Shape:", df.shape)display(df.head())
```

**Expected:** 1599 rows and 12 columns.

### **Step 3: Check missing values**

```
print("Missing values in each column:")print(df.isnull().sum())print("\nTotal missing values:", df.isnull().sum().sum())
```

The original dataset should have no missing values.

### **Step 4: Remove duplicate records**

```
print("Duplicate records before removal:", df.duplicated().sum())df = df.drop_duplicates()print("Duplicate records after removal:", df.duplicated().sum())print("Shape after removing duplicates:", df.shape)
```

**Expected:** 240 duplicate records removed; 1359 rows remain.

### **Step 5: Handle missing values**

Even though this dataset has no missing values, perform the median-filling step:

```
for column in df.select_dtypes(include=np.number).columns:    df[column] = df[column].fillna(df[column].median())print("Missing values after treatment:")print(df.isnull().sum().sum())
```

### **Step 6: Detect and treat outliers using IQR**

We will apply this to the input features, leaving `quality` unchanged because it is the target variable.

```
features = df.drop(columns=["quality"])Q1 = features.quantile(0.25)Q3 = features.quantile(0.75)IQR = Q3 - Q1lower_limit = Q1 - 1.5 * IQRupper_limit = Q3 + 1.5 * IQRfeatures = features.clip(lower=lower_limit, upper=upper_limit, axis=1)print("Outliers treated using IQR method")
```

### **Step 7: Normalize the features using Min-Max Scaling**

```
scaler = MinMaxScaler()scaled_features = scaler.fit_transform(features)scaled_df = pd.DataFrame(    scaled_features,    columns=features.columns)scaled_df["quality"] = df["quality"].valuesprint("Normalization completed successfully")display(scaled_df.head())
```

### **Step 8: Plot a histogram**

```
plt.figure(figsize=(7, 5))plt.hist(scaled_df["alcohol"], bins=20, edgecolor="black")plt.title("Distribution of Normalized Alcohol")plt.xlabel("Normalized Alcohol")plt.ylabel("Frequency")plt.show()
```

### **Step 9: Save the cleaned dataset**

```
scaled_df.to_csv("cleaned_wine_quality.csv", index=False)print("Cleaned dataset saved successfully")
```

To download the CSV to your laptop:

```
from google.colab import filesfiles.download("cleaned_wine_quality.csv")
```
Experiment 2 complete

