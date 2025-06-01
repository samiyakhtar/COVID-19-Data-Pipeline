# COVID-19-Data-Pipeline

COVID-19 Data Pipeline: From API to Dashboard

This project demonstrates an end-to-end data engineering pipeline that:
- Extracts COVID-19 data from a public API
- Cleans and transforms the data
- Loads the data into a SQLite database
- Visualizes key metrics using Plotly

Tools: Python, Pandas, Requests, SQLite, Plotly

How to run:
1. Install dependencies:
   pip install pandas requests plotly
2. Run the script:
   python covid_pipeline.py
"""

import requests
import pandas as pd
import sqlite3
import datetime
import plotly.express as px

# ----------------------
# 1. Data Extraction
# ----------------------

def fetch_covid_data(country='united-states'):
    """
    Fetch COVID-19 data for a specific country using the COVID19API.
    """
    url = f"https://api.covid19api.com/dayone/country/{country}"
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        df = pd.json_normalize(data)
        print(f"Data fetched for {country}. Records: {len(df)}")
        return df
    else:
        print(f"Error: {response.status_code}")
        return None

# ----------------------
# 2. Data Transformation
# ----------------------

def clean_transform_data(df):
    """
    Select relevant columns, format dates, and handle missing values.
    """
    df = df[['Country', 'Date', 'Confirmed', 'Deaths', 'Recovered', 'Active']]
    df['Date'] = pd.to_datetime(df['Date'])
    df['Week'] = df['Date'].dt.strftime('%Y-%U')
    df = df.fillna(0)
    print("Data cleaned and transformed.")
    return df

# ----------------------
# 3. Data Loading (SQLite)
# ----------------------

def load_data_to_sqlite(df, db_name='covid_data.db'):
    """
    Store the cleaned data into a local SQLite database.
    """
    conn = sqlite3.connect(db_name)
    df.to_sql('covid_stats', conn, if_exists='replace', index=False)
    conn.close()
    print(f"Data loaded into {db_name} database.")

# ----------------------
# 4. Data Visualization
# ----------------------

def visualize_data(df):
    """
    Generate an interactive line chart of confirmed COVID-19 cases over time.
    """
    fig = px.line(
        df,
        x='Date',
        y='Confirmed',
        title=f"COVID-19 Confirmed Cases Over Time in {df['Country'].iloc[0]}",
        labels={'Confirmed': 'Confirmed Cases'}
    )
    fig.show()

# ----------------------
# 5. Main Pipeline Function
# ----------------------

def run_pipeline():
    """
    Run the entire pipeline: Extract, Transform, Load, and Visualize.
    """
    country = 'united-states'  # Change this to fetch data for another country
    raw_df = fetch_covid_data(country)
    if raw_df is not None:
        cleaned_df = clean_transform_data(raw_df)
        load_data_to_sqlite(cleaned_df)
        visualize_data(cleaned_df)

if __name__ == "__main__":
    run_pipeline()
