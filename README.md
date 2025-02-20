# 🍽️ Yelp Restaurant Analysis 📊

## 🚀 Project Overview
This project analyzes Yelp restaurant data to understand the relationship between user engagement (reviews, tips, check-ins) and business success (ratings, review count). The goal is to provide insights that can help restaurant owners and stakeholders make data-driven decisions to enhance customer experience and boost ratings.

### 🎯 Research Objectives
1. **📈 Quantify the correlation between user engagement and review count/average rating**: Determine if higher engagement leads to increased reviews and better ratings.
2. **📝 Analyze the impact of sentiment on review count and average rating**: Investigate if positive sentiment in reviews influences star ratings and review volume.
3. **⏳ Time Trends in User Engagement**: Explore if consistent user engagement over time is a stronger predictor of long-term success than sporadic activity.

### 🧐 Hypothesis
- ✅ Higher user engagement correlates with higher review count and ratings.
- 😊 Positive sentiment in reviews and tips contributes to better ratings and more reviews.
- 📅 Consistent engagement over time positively impacts long-term business success.

## 🛠️ Skills & Technologies Used
| 💡 Skill | 📝 Description |
|---|---|
| 🐍 Python | Data analysis and processing |
| 🗃️ Pandas | Data manipulation and cleaning |
| 🛢️ SQL | Querying Yelp dataset |
| 📊 Matplotlib & Seaborn | Data visualization |
| 🧠 Sentiment Analysis | Understanding user sentiment in reviews |
| 🌍 Geolocation Analysis | Mapping restaurant locations |
| 🗺️ Folium | Interactive map visualizations |

## 🔍 Project Breakdown

### 1️⃣ Data Collection 📥
- The dataset is sourced from Yelp, containing information on restaurants, reviews, tips, and check-ins.
- SQL queries are used to extract relevant data from the Yelp database.
- The Yelp Open Dataset is a subset of Yelp data that is intended for educational use. It provides real-world data related to businesses including reviews, photos, check-ins, and attributes like hours, parking availability, and ambience.

()

### 2️⃣ Data Cleaning & Preprocessing 🧼
- Missing values and outliers are handled using Pandas.
- Date formats are adjusted for time-series analysis.


#### 🖥️ Code Snippet: Handling Missing Data (Example)
```python
business_df.dropna(inplace=True)  # Remove missing values
business_df["city"] = business_df["city"].str.title()  # Standardizing city names
```

#### 🖥️ Code Snippet: Handling Outliers
```python
def remove_outlier(df, col):
    q1 = df[col].quantile(0.25)
    q3 = df[col].quantile(0.75)
    IQR = q3 - q1
    lower_bound = q1 - 1.5*IQR
    upper_bound = q3 + 1.5*IQR

    df = df[(df[col] >= lower_bound) & (df[col] <= upper_bound)]
    
    return df
```

### 3️⃣ Exploratory Data Analysis (EDA) 🔍
- Summary statistics and visualizations are used to understand data distribution.
- Correlation analysis is performed between engagement metrics and business success.

#### 🖥️ Analysis and Findings
```sql
SELECT business_id, name, city, state, stars, review_count 
FROM business 
WHERE categories LIKE '%Restaurant%';
```
-  Filters only businesses that have "Restaurant" in their category.

()

#### 🖥️ Highest Rating and Review Count
Retrieves the **top 10 restaurants** with the highest review count and their average rating.
Similar query for Review Count as well

```sql
SELECT 
    name, 
    SUM(review_count) AS review_count, 
    AVG(stars) AS avg_rating 
FROM business 
WHERE business_id IN {tuple(business_id['business_id'])}  
GROUP BY name  
ORDER BY review_count DESC  
LIMIT 10;
```
() ()

#### 🖥️ Do restaurants with higher engagement tend to have higher ratings ?


Calculates the **total number of check-ins** for each business by counting date entries in the `checkin` table.
Since Checkins are in string
()
```sql
SELECT business_id,
       SUM(LENGTH(date) - LENGTH(REPLACE(date, ',', '')) + 1) AS checkin_count
FROM checkin
GROUP BY business_id;
```

- LENGTH(date) - LENGTH(REPLACE(date, ',', '')) → Counts the number of commas in the date string.
- +1 → Adds 1 since the last check-in date won’t have a trailing comma.
- SUM(...) → Computes the total check-ins per business.
- GROUP BY business_id → Aggregates data for each business.

Retrieves **average review count, check-ins, and tips** for businesses, grouped by rating.
```sql
SELECT total.avg_rating AS rating,
       AVG(total.review_count) AS avg_review_count,
       AVG(total.checkin_count) AS avg_checkin_count,
       AVG(total.tip_count) AS avg_tip_count
FROM (
    SELECT b.business_id,
           SUM(b.review_count) AS review_count,
           AVG(b.stars) AS avg_rating,
           SUM(LENGTH(cc.date) - LENGTH(REPLACE(cc.date, ',', '')) + 1) AS checkin_count,
           SUM(tip.tip_count) AS tip_count
    FROM business b
    LEFT JOIN checkin cc ON b.business_id = cc.business_id
    LEFT JOIN (
        SELECT business_id, COUNT(business_id) AS tip_count
        FROM tip
        GROUP BY business_id
    ) AS tip ON b.business_id = tip.business_id
    WHERE b.business_id IN {tuple(business_id['business_id'])}
    GROUP BY b.business_id
) AS total
GROUP BY total.avg_rating;
```
- AVG(total.review_count) → Average reviews per rating group.
- AVG(total.checkin_count) → Average check-ins per rating group.
- AVG(total.tip_count) → Average tips per rating group.
- LEFT JOIN checkin → Adds check-in data.
- LEFT JOIN tip → Aggregates tip counts per business.
- GROUP BY total.avg_rating → Groups by average rating.

()

#### 🖥️ Is there a correlation between the number of reviews, tips, and check-ins for a business?
Retrieves **average rating, review count, and sentiment metrics (useful, funny, cool)** for businesses.

```sql
SELECT b.business_id, 
       AVG(b.stars) AS avg_rating, 
       SUM(b.review_count) AS review_count,
       SUM(s.useful_count) AS useful_count,
       SUM(s.funny_count) AS funny_count,
       SUM(s.cool_count) AS cool_count
FROM (
    SELECT business_id,
           SUM(useful) AS useful_count,
           SUM(funny) AS funny_count,
           SUM(cool) AS cool_count
    FROM review
    GROUP BY business_id
) AS s
JOIN business AS b ON b.business_id = s.business_id
WHERE b.business_id IN {tuple(business_id['business_id'])}
GROUP BY b.business_id
ORDER BY review_count;
```
- AVG(b.stars) → Average rating per business.
- SUM(b.review_count) → Total reviews per business.
- SUM(useful_count, funny_count, cool_count) → Aggregates sentiment metrics.
- GROUP BY business_id → Groups results by business.
- ORDER BY review_count → Sorts businesses by review count.

()



#### 🖥️ How do the success metrics of restaurants vary across different states and cities ?
Retrieves the **top 10 cities** with the highest restaurant review count, along with average ratings and restaurant count.

```sql
SELECT 
    city, 
    state, 
    latitude, 
    longitude, 
    AVG(stars) AS avg_rating, 
    SUM(review_count) AS review_count, 
    COUNT(*) AS restaurant_count
FROM business
WHERE business_id IN {tuple(business_id['business_id'])}
GROUP BY city, state
ORDER BY review_count DESC
LIMIT 10;
```
- AVG(stars) → Average rating per city.
- SUM(review_count) → Total reviews across all restaurants in a city.
- COUNT(*) AS restaurant_count → Number of restaurants in the city.
- GROUP BY city, state → Aggregates data by city and state.
- ORDER BY review_count DESC → Ranks cities by review count.
- LIMIT 10 → Returns only the top 10 cities.

**Calculating Success Metrics**
 - success_score=avg_rating×log(review_count+1)
```python
def calculate_success_metric(df):
    return df['avg_rating'] * np.log(df['review_count'] + 1)

####
city_df['success_score'] = calculate_success_metric(city_df)
####
```

***Visualization***
### 📌 Folium Map: Visualizing Restaurant Success  

Creates an **interactive map** with circle markers and a heatmap to show restaurant success scores by city.

```python
m = folium.Map(
    location=[city_df['latitude'].mean(), city_df['longitude'].mean()],
    zoom_start=5,
    tiles="cartodbpositron"
)
```

Centers the map at the average city location.
Uses a clean tile style for better visibility.

```python
color_scale = branca.colormap.LinearColormap(
    colors=['green', 'yellow', '#E54F29'],
    vmin=city_df['success_score'].min(),
    vmax=city_df['success_score'].max(),
    caption="Success Score"
)
```

Defines a color gradient for success scores:
🟢 Low | 🟡 Medium | 🔴 High

```python
for index, row in city_df.iterrows():
    folium.CircleMarker(
        location=[row['latitude'], row['longitude']],
        radius=7,
        color=color_scale(row['success_score']),
        fill=True,
        fill_color=color_scale(row['success_score']),
        fill_opacity=0.8,
        popup=folium.Popup(f"<b>Success Score:</b> {row['success_score']:.2f}", max_width=200),
        weight=1
    ).add_to(m)

```

Circle markers represent cities, with color based on success score.
Popup displays the score for each location.

```python
heat_data = city_df[['latitude', 'longitude', 'success_score']].values.tolist()
HeatMap(heat_data, radius=15, blur=10).add_to(m)
Adds a heatmap layer to visualize density of successful restaurants.
```


()

#### 🖥️ Are there any patterns in user engagement over time for successful businesses compared to less successful ones?

```sql
SELECT review.month_year, review.review_count, tip.tip_count 
FROM (
    SELECT strftime('%m-%Y', date) AS month_year, COUNT(*) AS review_count
    FROM review
    WHERE business_id IN {tuple(business_id['business_id'])} AND stars >= 3.5
    GROUP BY month_year
    ORDER BY month_year
) AS review
JOIN (
    SELECT strftime('%m-%Y', tip.date) AS month_year, COUNT(*) AS tip_count
    FROM tip
    JOIN business AS b ON tip.business_id = b.business_id
    WHERE tip.business_id IN {tuple(business_id['business_id'])} AND b.stars >= 3.5
    GROUP BY month_year
    ORDER BY month_year
) AS tip
ON review.month_year = tip.month_year;
```

- Extracts month & year (strftime('%m-%Y', date)) for reviews & tips.
- ounts reviews (COUNT(*) AS review_count) per month for high-rated businesses.
- ounts tips (COUNT(*) AS tip_count) per month for the same businesses.
- oins review & tip data on the month-year field.
-  similar query runs for low-rated (<3.5 stars) restaurants to compare trends.
### 📊 Insights:
- Identifies seasonal patterns in user engagement.
- Compares high-rated vs. low-rated restaurants to see if success affects engagement trends.
()

#### 🖥️ Trend and Seasonality Analysis
()

#### 🖥️ How does the sentiment of reviews and tips (useful, funny,cool) correlate with the success metrics of restaurants?

```sql
SELECT b.business_id, 
       AVG(b.stars) AS avg_rating, 
       SUM(b.review_count) AS review_count,
       SUM(s.useful_count) AS useful_count,
       SUM(s.funny_count) AS funny_count,
       SUM(s.cool_count) AS cool_count
FROM (
    SELECT business_id,
           SUM(useful) AS useful_count,
           SUM(funny) AS funny_count,
           SUM(cool) AS cool_count
    FROM review
    GROUP BY business_id
) AS s
JOIN business AS b ON b.business_id = s.business_id
WHERE b.business_id IN {tuple(business_id['business_id'])}
GROUP BY b.business_id
ORDER BY review_count;
```

***Data Cleaning: Removing Outliers***
```python


def remove_outlier(df, col):
    q1 = df[col].quantile(0.25)
    q3 = df[col].quantile(0.75)
    IQR = q3 - q1
    lower_bound = q1 - 1.5*IQR
    upper_bound = q3 + 1.5*IQR

    df = df[(df[col] >= lower_bound) & (df[col] <= upper_bound)]
    
    return df

# sentiment_df = ABOVE QUERY
sentiment_df = remove_outlier(sentiment_df,'review_count')
sentiment_df = remove_outlier(sentiment_df,'useful_count')
sentiment_df = remove_outlier(sentiment_df,'funny_count')
sentiment_df = remove_outlier(sentiment_df,'cool_count')
```

- AVG(b.stars) → Computes the average rating per business.
- SUM(b.review_count) → Calculates the total number of reviews.
- SUM(useful_count, funny_count, cool_count) → Aggregates sentiment metrics.
- GROUP BY business_id → Groups data per business.
- ORDER BY review_count → Sorts businesses by number of reviews.
### 📊 Insights:
-Examines whether higher engagement (useful, funny, cool votes) correlates with higher ratings.
- Helps in understanding how customer sentiment impacts restaurant success.

#### 🖥️ Is there any difference in engagement of elite users and non elite users ?
### 📌 SQL Query: Elite vs. Non-Elite User Engagement  
This query categorizes users as **Elite** or **Not Elite** and analyzes their engagement by **total reviews**.

```sql
SELECT
    elite,
    COUNT(*) AS num_users,
    SUM(review_count) AS total_review_count
FROM (
    SELECT
        CASE
            WHEN elite = '' THEN 'Not Elite'
            ELSE 'Elite'
        END AS elite,
        u.review_count
    FROM user u
) AS user_elite
GROUP BY elite;
```
- CASE WHEN elite = '' THEN 'Not Elite' ELSE 'Elite' →
- Users with an empty elite field are labeled as "Not Elite".
- Others are labeled as "Elite".
- COUNT(*) AS num_users → Counts the number of users in each group.
- SUM(review_count) AS total_review_count → Calculates the total reviews written by each group.
- GROUP BY elite → Groups data by "Elite" and "Not Elite".
### 📊 Insights:
- Shows whether Elite users contribute more reviews than Non-Elite users.
- Helps in understanding the impact of Yelp’s Elite program on user engagement.

()

#### 🖥️ Busiest Hour

()

#### 🖥️ Recommendation

- Utilizing insights from the analysis of various metrics such as user engagement, sentiment of reviews, peak hours, and the impact of elite users, businesses can make informed decisions to drive success.
- Collaborating with elite users and leveraging their influence can amplify promotional efforts, increase brand awareness, and drive customer acquisition.
- Businesses can adjust their operating hours or introduce special promotions to capitalize on the increased demand during peak hours.
- Less successful businesses may need to focus on strategies to enhance user engagement over time, such as improving service quality, responding to customer feedback.
- Cities with high success scores presents opportunities for restaurant chains to expand or invest further


## 📌 Results & Insights
- 📊 Restaurants with higher user engagement tend to have better ratings and more reviews.
- 😊 Positive sentiment in reviews significantly impacts star ratings.
- ⏳ Consistent engagement over time is a stronger predictor of long-term success than occasional bursts of activity.

## 🏆 Conclusion
This project demonstrates the power of data-driven insights in the restaurant industry. By understanding customer engagement patterns, businesses can strategize to improve customer satisfaction and boost their Yelp ratings. 🍽️

