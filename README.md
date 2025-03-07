# 📈 Nifty vs. Bank Nifty: Financial Correlation Analysis

## 🔍 Project Overview
This project analyzes the relationship between **Nifty 50** and **Bank Nifty** using historical data from Yahoo Finance. The analysis includes:
- 📊 **Time Series Visualization**
- 📈 **Stationarity Test (ADF Test)**
- 🔥 **Correlation Analysis (Pearson Correlation & Heatmap)**
- 📉 **Rolling Correlation (30-Day Window)**
- 🤔 **Granger Causality Test**
- 🧠 **Key Insights & Market Interpretation**

## 📂 Dataset
We fetch historical **daily closing prices** (2015-2025) using the **Yahoo Finance API**.

## 🛠️ Steps & Code Implementation

### 1️⃣ Fetching Data from Yahoo Finance 📡
We retrieve historical data for **Nifty 50 (NSEI)** and **Bank Nifty (NSEBANK)**:
```python
import yfinance as yf

bank_nifty = yf.download("^NSEBANK", start="2015-01-01", end="2025-01-01", interval="1d")
nifty = yf.download("^NSEI", start="2015-01-01", end="2025-01-01", interval="1d")
```

### 2️⃣ Data Preprocessing & Merging 🧹
We combine the **closing prices** of both indices:
```python
import pandas as pd

df = pd.concat([nifty['Close'], bank_nifty['Close']], axis=1)
df.columns = ['nifty', 'bank_nifty']
df.dropna(inplace=True)  # Remove missing values
```

### 3️⃣ Time Series Visualization 📊
We plot **Nifty vs. Bank Nifty closing prices** over time:
```python
import matplotlib.pyplot as plt

plt.figure(figsize=(12,6))
plt.plot(df.index, df['nifty'], label='Nifty', alpha=0.7)
plt.plot(df.index, df['bank_nifty'], label='Bank Nifty', alpha=0.7)
plt.legend()
plt.title('Nifty vs Bank Nifty Closing Price')
plt.show()
```

📌 **Observation:** Both indices move together, indicating a strong relationship.
### **Why Use the ADF Test (Augmented Dickey-Fuller) for Nifty vs. Bank Nifty Analysis?**  

The **ADF Test (Augmented Dickey-Fuller Test)** is used to check if a **time series is stationary or not**. 
This is important in financial data analysis because:  

1. **Stationary Data is Required for Many Models**  
   - Financial time series data (e.g., Nifty 50, Bank Nifty) is often **non-stationary**.  
   - If the data is non-stationary, models like **linear regression, Granger causality, and ARIMA** may not work properly.  

2. **Removes Trends and Seasonality**  
   - If the ADF test shows that the data is non-stationary, we need to **differentiate** or transform the data before analysis.  
   - Stationarity ensures that statistical properties like **mean and variance remain constant over time**.  

---

### **📌 Why Not Use Other Tests Instead of ADF?**

| **Test** | **Why Not Use It?** |
|----------|---------------------|
| **KPSS Test** (Kwiatkowski–Phillips–Schmidt–Shin) | Tests **the opposite** of ADF (i.e., whether the series is stationary). Usually used **together** with ADF for confirmation. |
| **Phillips-Perron (PP) Test** | Similar to ADF but adjusts for heteroskedasticity. ADF is generally preferred for financial data. |
| **Ljung-Box Test** | Checks for autocorrelation, **not stationarity**. |
| **Zivot-Andrews Test** | Used if there is a **structural break** in the data (e.g., major economic events). |

---

### **📌 When to Use ADF Test?**
✅ Before applying **Granger Causality**, Regression, or ARIMA models.  
✅ To check if Nifty and Bank Nifty follow a trend.  
✅ If p-value > 0.05 → **Data is non-stationary** (must be differenced).  
✅ If p-value ≤ 0.05 → **Data is stationary** (no transformation needed).  

### 4️⃣ Stationarity Test 📏
We use the **Augmented Dickey-Fuller (ADF) test** to check stationarity:
```python
from statsmodels.tsa.stattools import adfuller

result = adfuller(df['nifty'])
print("Nifty ADF Statistic:", result[0], "P-value:", result[1])

result1 = adfuller(df['bank_nifty'])
print("Bank Nifty ADF Statistic:", result1[0], "P-value:", result1[1])
```
📌 **Interpretation:** If **p-value < 0.05**, the data is stationary.

### 5️⃣ Correlation Analysis 🔥
We compute **Pearson correlation** and visualize it using a **heatmap**:
```python
import seaborn as sns

sns.heatmap(df.corr(), annot=True, cmap="coolwarm")
plt.title("Correlation Matrix")
plt.show()
```
📌 **Insight:** If correlation is **0.95**, Bank Nifty and Nifty 50 are highly interdependent.

### **Why Use Pearson Correlation for Nifty & Bank Nifty?**  

The **Pearson correlation coefficient** measures the **linear relationship** between two time series (**Nifty 50 & Bank Nifty**). In financial markets, we use **Pearson correlation** because:  

1. **Measures Linear Dependency**  
   - It shows how **strongly Nifty 50 and Bank Nifty move together** in a straight-line fashion.  
   - Values range from **-1 to 1**:  
     - **1** → Perfect positive correlation (both move in the same direction).  
     - **-1** → Perfect negative correlation (one goes up, the other goes down).  
     - **0** → No linear relationship.  

2. **Financial Data Often Has a Linear Relationship**  
   - Stock indices (like **Nifty & Bank Nifty**) generally move in a **linear fashion**, making Pearson ideal.  
   - Bank Nifty is a subset of Nifty 50 (heavily weighted by banking stocks), so their **movements tend to be linearly correlated**.  

---

### **Why Not Use Other Correlation Methods?**

| **Correlation Type** | **Why Not Use It?** |
|----------------------|---------------------|
| **Spearman Correlation** | Measures **rank correlation** (not actual price movements). Useful if data has **non-linear** relationships, which is **not the case** for Nifty-Bank Nifty. |
| **Kendall’s Tau** | Best for **ordinal (ranked) data**, not continuous financial data. |
| **Distance Correlation** | Captures **non-linear relationships**, but financial indices mostly follow linear trends. |
| **Dynamic Time Warping (DTW)** | Good for time-series similarity, but **not necessary** for simple correlation analysis. |

---

### **📌 When is Pearson NOT the Best Choice?**
- If the **relationship is non-linear**, then **Spearman** might be better.  
- If analyzing **rank-based trends** (e.g., top-performing stocks), **Kendall's Tau** is useful.  
- If comparing **returns instead of prices**, we might use **log returns correlation** instead. 

### 6️⃣ Rolling Correlation 📉
We compute a **30-day rolling correlation** to observe changes over time:
```python
rolling_corr = df['nifty'].rolling(window=30).corr(df['bank_nifty'])
rolling_corr.plot(title="Rolling Correlation (30 Days)")
plt.show()
```
📌 **Market Insight:** Variations in rolling correlation indicate **economic shifts, regulatory changes, or market movements.**

### 7️⃣ Granger Causality Test 🤔
We check if one index can **predict** the other:
```python
from statsmodels.tsa.stattools import grangercausalitytests

grangercausalitytests(df[['nifty', 'bank_nifty']], maxlag=5)
grangercausalitytests(df[['bank_nifty', 'nifty']], maxlag=5)
```
📌 **Conclusion:** If **p-value > 0.05**, we fail to reject the null hypothesis, meaning **Nifty does NOT Granger-cause Bank Nifty.**

## 📌 Key Takeaways
✅ **Strong Correlation:** A high correlation (~0.95) confirms that both indices move in sync.
✅ **Rolling Correlation Insights:** Market shifts can be observed through changing correlation trends.
✅ **No Predictive Relationship:** **Nifty does NOT cause Bank Nifty and vice versa**, indicating that both are influenced by external market factors.

## 📷 Visualizations
### 1️⃣ **Nifty vs. Bank Nifty Closing Prices**
![Nifty vs Bank Nifty](nifty_vs_bank_nifty.png)

### 2️⃣ **Correlation Matrix**
![Correlation Matrix](correlation_matrix.png)

### 3️⃣ **Rolling Correlation (30 Days)**
![Rolling Correlation](rolling_correlation.png)

---

## 🚀 Future Scope
- Implement **Machine Learning models** to predict index movements 📊.
- Extend analysis to **other market indices** (e.g., Sensex, Midcap, etc.) 📈.
- Explore **macro-economic indicators** affecting these indices 🏦.

## 📢 Let's Connect!
If you found this project useful, feel free to ⭐ the repository and connect with me on **[LinkedIn](#)**! 😊

