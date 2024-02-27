pip install requests beautifulsoup4 openpyxl

import requests
from bs4 import BeautifulSoup
import openpyxl

# Mount Google Drive
from google.colab import drive
drive.mount('/content/drive')

def extract_article_text(url):
    try:
        response = requests.get(url)
        soup = BeautifulSoup(response.content, 'html.parser')
        article_title = soup.find('title').get_text().strip()
        article_text = ''
        for paragraph in soup.find_all('p'):
            article_text += paragraph.get_text().strip() + '\n'
        return article_title, article_text
    except Exception as e:
        print(f"Error extracting article from {url}: {e}")
        return None, None

def save_article_to_file(url_id, article_title, article_text):
    filename = f"/content/drive/MyDrive/{url_id}.txt"
    with open(filename, 'w', encoding='utf-8') as file:
        file.write(article_title + '\n\n')
        file.write(article_text)

def main():
    input_file = "/content/drive/MyDrive/Input.xlsx"
    wb = openpyxl.load_workbook(input_file)
    sheet = wb.active

    for row in sheet.iter_rows(min_row=2, max_row=sheet.max_row, values_only=True):
        url_id, url = row
        article_title, article_text = extract_article_text(url)
        if article_title and article_text:
            save_article_to_file(url_id, article_title, article_text)
            print(f"Article {url_id} extracted and saved.")
        else:
            print(f"Failed to extract article {url_id}.")

if __name__ == "__main__":
    main()

!pip install nltk textblob openpyxl

import nltk
from textblob import TextBlob


# Load NLTK resources
nltk.download('punkt')
nltk.download('averaged_perceptron_tagger')
nltk.download('wordnet')

def analyze_text(article_text):
    # Tokenize text into words
    words = nltk.word_tokenize(article_text)

    # Compute number of words
    num_words = len(words)

    # Compute number of sentences
    sentences = nltk.sent_tokenize(article_text)
    num_sentences = len(sentences)

    # Compute average word length
    avg_word_length = sum(len(word) for word in words) / num_words

    # Compute parts of speech tags
    pos_tags = nltk.pos_tag(words)

    # Compute number of nouns, verbs, adjectives, and adverbs
    num_nouns = len([word for word, pos in pos_tags if pos.startswith('N')])
    num_verbs = len([word for word, pos in pos_tags if pos.startswith('V')])
    num_adjectives = len([word for word, pos in pos_tags if pos.startswith('J')])
    num_adverbs = len([word for word, pos in pos_tags if pos.startswith('R')])

    # Compute polarity and subjectivity using TextBlob
    blob = TextBlob(article_text)
    polarity = blob.sentiment.polarity
    subjectivity = blob.sentiment.subjectivity

    return num_words, num_sentences, avg_word_length, num_nouns, num_verbs, num_adjectives, num_adverbs, polarity, subjectivity

def main():
    input_folder = "/content/drive/MyDrive/"
    output_file = "/content/drive/MyDrive/Output Data Structure.xlsx"

    wb = openpyxl.load_workbook(output_file)
    sheet = wb.active

    for i in range(1, 101):  # Assuming the files are named from blackassign0001.txt to blackassign0100.txt
        url_id = f"{i:04d}"  # Convert the number to a 4-digit string
        input_filename = f"blackassign{url_id}.txt"
        with open(input_folder + input_filename, 'r', encoding='utf-8') as file:
            article_text = file.read()

        num_words, num_sentences, avg_word_length, num_nouns, num_verbs, num_adjectives, num_adverbs, polarity, subjectivity = analyze_text(article_text)

        output_row = (num_words, num_sentences, avg_word_length, num_nouns, num_verbs, num_adjectives, num_adverbs, polarity, subjectivity)

        sheet.append(output_row)

    wb.save(output_file)

if __name__ == "__main__":
    main()

!pip install python-docx

from docx import Document

def extract_variables(docx_path):
    doc = Document(docx_path)
    variables = {}
    for paragraph in doc.paragraphs:
        # Split each paragraph by ':' to get the variable name and its value
        parts = paragraph.text.split(':')
        if len(parts) == 2:
            variable_name = parts[0].strip()
            variable_value = parts[1].strip()
            variables[variable_name] = variable_value
    return variables

def main():
    docx_path = "/content/drive/MyDrive/Text Analysis.docx"  # Adjust the path accordingly
    variables = extract_variables(docx_path)
    print("Extracted variables:")
    for variable_name, variable_value in variables.items():
        print(f"{variable_name}: {variable_value}")

if __name__ == "__main__":
    main()

def read_words_from_folder(file_path):
    words = set()
    with open(file_path, 'r') as file:
        for line in file:
            words.add(line.strip())
    return words

import os

def read_words_from_folder(file_path):
    words = set()
    encodings = ['utf-8', 'latin-1', 'ISO-8859-1']  # List of encodings to try
    for encoding in encodings:
        try:
            with open(file_path, 'r', encoding=encoding) as file:
                for line in file:
                    words.add(line.strip())
            return words
        except UnicodeDecodeError:
            continue
    print(f"Error: Unable to decode file '{file_path}' using any of the specified encodings.")
    return None

import os

def read_words_from_files(directory_path):
    words = set()
    encodings = ['utf-8', 'latin-1', 'ISO-8859-1']  # List of encodings to try
    for file_name in os.listdir(directory_path):
        file_path = os.path.join(directory_path, file_name)
        if os.path.isfile(file_path):
            for encoding in encodings:
                try:
                    with open(file_path, 'r', encoding=encoding) as file:
                        for line in file:
                            words.add(line.strip())
                    break  # Break out of encoding loop if successful
                except UnicodeDecodeError:
                    continue
    return words

stopwords_directory_path = '/content/drive/MyDrive/StopWords'
stopwords = read_words_from_files(stopwords_directory_path)

master_words = read_words_from_files('/content/drive/MyDrive/MasterDictionary')

import os
import pandas as pd
import nltk
from nltk.tokenize import word_tokenize, sent_tokenize
from nltk.corpus import stopwords
import string
import re

# Load NLTK resources
nltk.download('punkt')
nltk.download('stopwords')

# Function to read words from text files in a folder
def read_words_from_files(folder_path):
    words = set()
    for filename in os.listdir(folder_path):
        if filename.endswith(".txt"):
            with open(os.path.join(folder_path, filename), 'r', encoding='latin1') as file:
                for line in file:
                    words.add(line.strip())
    return words

# Function to calculate the number of syllables in a word
def count_syllables(word):
    word = word.lower()
    vowels = 'aeiouy'
    count = 0
    if word[0] in vowels:
        count += 1
    for index in range(1, len(word)):
        if word[index] in vowels and word[index - 1] not in vowels:
            count += 1
    if word.endswith('e'):
        count -= 1
    if word.endswith('le') and len(word) > 2 and word[-3] not in vowels:
        count += 1
    if count == 0:
        count += 1
    return count

# Function to calculate variables for each text file
def calculate_variables(article_text):
    # Clean the text
    stop_words = set(stopwords.words('english'))
    cleaned_text = [word.lower() for word in word_tokenize(article_text) if word.lower() not in stop_words and word.lower() not in string.punctuation]

    # Calculate variables
    positive_words = read_words_from_files('/content/drive/MyDrive/StopWords/')
    negative_words = read_words_from_files('/content/drive/MyDrive/StopWords/')
    positive_score = sum(1 for word in cleaned_text if word in positive_words)
    negative_score = sum(1 for word in cleaned_text if word in negative_words)
    polarity_score = (positive_score - negative_score) / ((positive_score + negative_score) + 0.000001)
    subjectivity_score = (positive_score + negative_score) / (len(cleaned_text) + 0.000001)

    sentences = sent_tokenize(article_text)
    words = word_tokenize(article_text)
    average_sentence_length = len(words) / len(sentences)
    complex_word_count = sum(1 for word in cleaned_text if count_syllables(word) > 2)
    percentage_complex_words = complex_word_count / len(cleaned_text)
    fog_index = 0.4 * (average_sentence_length + percentage_complex_words)
    average_words_per_sentence = len(words) / len(sentences)

    word_count = len(cleaned_text)

    syllable_per_word = sum(count_syllables(word) for word in cleaned_text) / len(cleaned_text)

    personal_pronouns = len(re.findall(r'\b(i|we|my|ours|us)\b', article_text.lower()))

    average_word_length = sum(len(word) for word in cleaned_text) / len(cleaned_text)

    return positive_score, negative_score, polarity_score, subjectivity_score, average_sentence_length, percentage_complex_words, fog_index, average_words_per_sentence, complex_word_count, word_count, syllable_per_word, personal_pronouns, average_word_length

# Function to calculate variables for each text file
def calculate_variables_for_file(file_path):
    with open(file_path, 'r', encoding='latin1') as file:
        article_text = file.read()
    return calculate_variables(article_text)

# Calculate variables for all text files
output_data = []
folder_path = '/content/drive/MyDrive/'
for filename in os.listdir(folder_path):
    if filename.endswith(".txt"):
        file_path = os.path.join(folder_path, filename)
        positive_score, negative_score, polarity_score, subjectivity_score, average_sentence_length, percentage_complex_words, fog_index, average_words_per_sentence, complex_word_count, word_count, syllable_per_word, personal_pronouns, average_word_length = calculate_variables_for_file(file_path)
        output_data.append([filename, positive_score, negative_score, polarity_score, subjectivity_score, average_sentence_length, percentage_complex_words, fog_index, average_words_per_sentence, complex_word_count, word_count, syllable_per_word, personal_pronouns, average_word_length])

# Create DataFrame from output data
columns = ["Filename", "Positive Score", "Negative Score", "Polarity Score", "Subjectivity Score", "Average Sentence Length", "Percentage of Complex Words", "Fog Index", "Average Number of Words Per Sentence", "Complex Word Count", "Word Count", "Syllable Per Word", "Personal Pronouns", "Average Word Length"]
output_df = pd.DataFrame(output_data, columns=columns)

# Load the updated DataFrame
output_excel_file = '/content/drive/MyDrive/Output Data Structure.xlsx'
output_df_existing = pd.read_excel(output_excel_file)

# Map the desired column names to their corresponding original column names
column_mapping = {
    "POSITIVE SCORE": "Positive Score",
    "NEGATIVE SCORE": "Negative Score",
    "POLARITY SCORE": "Polarity Score",
    "SUBJECTIVITY SCORE": "Subjectivity Score",
    "AVG SENTENCE LENGTH": "Average Sentence Length",
    "PERCENTAGE OF COMPLEX WORDS": "Percentage of Complex Words",
    "FOG INDEX": "Fog Index",
    "AVG NUMBER OF WORDS PER SENTENCE": "Average Number of Words Per Sentence",
    "COMPLEX WORD COUNT": "Complex Word Count",
    "WORD COUNT": "Word Count",
    "SYLLABLE PER WORD": "Syllable Per Word",
    "PERSONAL PRONOUNS": "Personal Pronouns",
    "AVG WORD LENGTH": "Average Word Length"
}

# Rename the columns in the updated DataFrame
output_df_existing = output_df_existing.rename(columns=column_mapping)

# Update the values of the desired columns
output_df_existing[output_df.columns[1:]] = output_df[output_df.columns[1:]]

# Save the updated DataFrame to the same Excel file
output_df_existing.to_excel(output_excel_file, index=False)
print("Updated output saved to:", output_excel_file)
