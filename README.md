# covidprojectsubmissiontrial


import pandas as pd
import requests
import matplotlib.pyplot as plt
import seaborn as sns
from datetime import datetime

# API Configuration
COVID_API_URL = "https://disease.sh/v3/covid-19/countries"

def fetch_covid_data():
    """Fetches the latest COVID-19 data from the API."""
    try:
        response = requests.get(COVID_API_URL)
        response.raise_for_status()  # Check for HTTP errors
        data = response.json()
        return pd.DataFrame(data)
    except Exception as e:
        print(f"Error fetching data: {e}")
        return None

def preprocess_data(df):
    """Cleans and prepares the data for analysis."""
    if df is None:
        return None
    
    # Select relevant columns
    columns = ['country', 'cases', 'deaths', 'recovered', 'active', 'critical', 'tests', 'population']
    df = df[columns]
    
    # Handle missing values
    df.fillna(0, inplace=True)
    
    # Calculate additional metrics
    df['death_rate'] = (df['deaths'] / df['cases']) * 100
    df['recovery_rate'] = (df['recovered'] / df['cases']) * 100
    
    return df

def analyze_data(df):
    """Performs basic statistical analysis."""
    if df is None:
        return None
    
    print("\n📊 **COVID-19 Global Statistics** 📊")
    print("-----------------------------------")
    print(f"Total Cases Worldwide: {df['cases'].sum():,}")
    print(f"Total Deaths Worldwide: {df['deaths'].sum():,}")
    print(f"Global Death Rate: {df['death_rate'].mean():.2f}%")
    print("\nTop 5 Most Affected Countries:")
    print(df.sort_values('cases', ascending=False).head(5)[['country', 'cases', 'deaths']])

def visualize_data(df):
    """Generates key visualizations."""
    if df is None:
        return None
    
    plt.figure(figsize=(12, 6))
    
    # Top 10 countries by cases
    top_cases = df.sort_values('cases', ascending=False).head(10)
    sns.barplot(x='cases', y='country', data=top_cases, palette='viridis')
    plt.title("Top 10 Countries by Total COVID-19 Cases")
    plt.xlabel("Total Cases (Millions)")
    plt.ylabel("Country")
    plt.show()
    
    # Death rate vs. recovery rate
    plt.figure(figsize=(10, 6))
    sns.scatterplot(x='death_rate', y='recovery_rate', data=df, hue='country', legend=False)
    plt.title("Death Rate vs. Recovery Rate by Country")
    plt.xlabel("Death Rate (%)")
    plt.ylabel("Recovery Rate (%)")
    plt.show()

if __name__ == "__main__":
    print("🦠 COVID-19 Global Data Tracker 🦠")
    print("---------------------------------")
    
    # Fetch & process data
    raw_data = fetch_covid_data()
    clean_data = preprocess_data(raw_data)
    
    if clean_data is not None:
        analyze_data(clean_data)
        visualize_data(clean_data)
    else:
        print("❌ Failed to load data. Please check your internet connection or API status.")
