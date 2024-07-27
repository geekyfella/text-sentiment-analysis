# Text Sentiment Analysis

This script performs sentiment analysis on textual content obtained from either a web page (HTML) or a PDF document. It processes the text, analyzes the sentiment, and generates a word cloud visualization.

## Table of Contents

- [Dependencies](#dependencies)
- [Script Breakdown](#script-breakdown)
  - [1. Import Libraries](#1-import-libraries)
  - [2. Fetch Articles from a Website](#2-fetch-articles-from-a-website)
  - [3. Preprocess Text Data](#3-preprocess-text-data)
  - [4. Analyze Sentiment of Text](#4-analyze-sentiment-of-text)
  - [5. Summarize Results](#5-summarize-results)
  - [6. Fetch and Process PDF](#6-fetch-and-process-pdf)
  - [7. Main Function](#7-main-function)
- [Example Usage](#example-usage)
- [Notes](#notes)

## Dependencies

Before running the script, ensure you have the following Python packages installed:

- `requests`
- `beautifulsoup4`
- `nltk`
- `vaderSentiment`
- `wordcloud`
- `matplotlib`
- `PyPDF2`

You can install these packages using pip:

```bash
pip install requests beautifulsoup4 nltk vaderSentiment wordcloud matplotlib PyPDF2
```

## Script Breakdown

### 1. Import Libraries

The script begins by importing the necessary libraries for web scraping, text processing, sentiment analysis, and PDF parsing:

```python
import requests
from bs4 import BeautifulSoup
import re
import nltk
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize
from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer
from wordcloud import WordCloud
import matplotlib.pyplot as plt
import PyPDF2

# Downloading necessary Natural Language Toolkit (NLTK) data
nltk.download('punkt')
nltk.download('stopwords')
```

### 2. Fetch Articles from a Website

The `fetch_articles` function fetches the content of a webpage and extracts text from specified HTML tags:

```python
def fetch_articles(url):
    headers = {'User-Agent': 'Mozilla/5.0'}
    response = requests.get(url, headers=headers)
    soup = BeautifulSoup(response.content, 'html.parser')
    
    # Print the first 1000 characters of the HTML content to inspect
    print(soup.prettify()[:1000])
    
    # Find all the divs with class 'c-article-section__content' or divs with the content to be analyzed
    content_divs = soup.find_all('div', {'class': 'c-article-section__content'})
    
    data = []
    for div in content_divs:
        paragraphs = div.find_all('p')
        for paragraph in paragraphs:
            content = paragraph.text.strip()
            data.append({'content': content})
    
    if not data:
        print("No articles found.")
    return data
```

### 3. Preprocess Text Data

The `preprocess_text` function cleans and tokenizes the text:

```python
def preprocess_text(text):
    text = re.sub(r'\s+', ' ', text)  # Remove extra spaces
    text = re.sub(r'\d', '', text)  # Remove digits
    text = text.lower()  # Convert to lowercase
    words = word_tokenize(text)  # Tokenize
    stop_words = set(stopwords.words('english'))
    words = [word for word in words if word.isalpha() and word not in stop_words]  # Remove stopwords and non-alphabetic words
    return ' '.join(words)
```

### 4. Analyze Sentiment of Text

The `analyze_sentiment` function uses the VADER sentiment analyzer to score the sentiment of the text:

```python
analyzer = SentimentIntensityAnalyzer()
def analyze_sentiment(text):
    scores = analyzer.polarity_scores(text)
    return scores
```

### 5. Summarize Results

The `summarize_results` function summarizes the sentiment analysis results:

```python
def summarize_results(articles):
    positive = sum(1 for article in articles if article['sentiment']['compound'] > 0)
    neutral = sum(1 for article in articles if article['sentiment']['compound'] == 0)
    negative = sum(1 for article in articles if article['sentiment']['compound'] < 0)
    total = len(articles)

    print(f"Total articles analyzed: {total}")
    print(f"Positive articles: {positive} ({positive / total * 100:.2f}%)")
    print(f"Neutral articles: {neutral} ({neutral / total * 100:.2f}%)")
    print(f"Negative articles: {negative} ({negative / total * 100:.2f}%)")
```

### 6. Fetch and Process PDF

The `fetch_pdf_content` function extracts text from a PDF file:

```python
def fetch_pdf_content(pdf_path):
    text = ""
    with open(pdf_path, 'rb') as file:
        reader = PyPDF2.PdfReader(file)
        for page in reader.pages:
            text += page.extract_text()
    return text
```

### 7. Main Function

The `main` function orchestrates the entire process, handling both HTML and PDF inputs:

```python
def main(input_source, is_pdf=False):
    articles = []
    if is_pdf:
        # Process PDF file
        pdf_text = fetch_pdf_content(input_source)
        articles.append({'content': pdf_text})
    else:
        # Process HTML URL
        articles = fetch_articles(input_source)

    # Check if articles are fetched correctly
    if not articles:
        print("No articles fetched. Please check the input source or structure.")
    else:
        # Preprocess article contents
        for article in articles:
            article['content'] = preprocess_text(article['content'])
            print(f"Preprocessed content: {article['content']}")  # Debug statement

        # Analyze sentiment of each article
        for article in articles:
            article['sentiment'] = analyze_sentiment(article['content'])
            print(f"Sentiment scores: {article['sentiment']}")  # Debug statement

        # Combine all article contents
        all_text = ' '.join([article['content'] for article in articles])
        print(f"All combined text: {all_text}")  # Debug statement

        # Generate word cloud if there is any text
        if all_text:
            wordcloud = WordCloud(width=800, height=400, background_color='white').generate(all_text)

            # Display word cloud
            plt.figure(figsize=(10, 5))
            plt.imshow(wordcloud, interpolation='bilinear')
            plt.axis('off')
            plt.show()
        else:
            print("No text available for word cloud generation.")

        # Summarize results
        summarize_results(articles)
```

## Example Usage

To use the script, set the `input_source` to either a URL (as `url`) or a PDF (as `pdf_path`) and set the `is_pdf` flag accordingly for analyzing URL/PDF file respectively:

```python
# URL to fetch
url = 'https://link.springer.com/article/10.1007/s42398-018-0013-3'
# PDF file path 
pdf_path = 'C:/Users/ojasv/text-sentiment-analysis/src/Eco-design_processes_in_the_automotive_industry.pdf'

# Set input source and set is_pdf flag accordingly
input_source = url  # Change to pdf_path for PDF analysis
is_pdf = False  # Change to True for PDF analysis

main(input_source, is_pdf)
```

## Notes

- Ensure that the PDF file path is correct when analyzing a PDF.
- Use Forward slash or Oblique (" / ") insted of reverse slash ( " \ " ) doing so will remove the unicode error.
- The script prints debug statements to help track the processing stages.
- The `main` function handles both URL and PDF inputs, making it versatile for different text sources.

