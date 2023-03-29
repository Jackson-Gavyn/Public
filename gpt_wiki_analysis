import os
import requests
from bs4 import BeautifulSoup
import pandas as pd
import openai

# Set up the OpenAI API key from environment variable
api_key = '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'
openai.api_key = api_key

# Fetch and parse table data
url = 'https://starcitizen.tools/List_of_pledge_vehicles'
response = requests.get(url)

# Define the column options
column_options = {
    1: "Name",
    2: "Manufacturer",
    3: "Role",
    4: "Standalone cost",
    5: "Availability",
    6: "Status",
    7: "Loaner"
}


if response.status_code == 200:
    html_content = response.text
else:
    print("Failed to fetch the webpage.")

soup = BeautifulSoup(html_content, 'html.parser')
table = soup.find('table', class_='plain-table floatnone wikitable sortable')

headers = []
for th in table.find_all('th'):
    headers.append(th.text.strip())

table_data = []
for row in table.find_all('tr')[1:]:
    rowData = {}
    for i, cell in enumerate(row.find_all('td')):
        rowData[headers[i]] = cell.text.strip()
    table_data.append(rowData)

# Create a DataFrame with the table data
df = pd.DataFrame(table_data)


# Function to prompt GPT-3 with a question about which columns are relevant
def get_relevant_columns(question, column_options):
    prompt = f"Which columns are relevant for the following question: {question}\n\nColumn options: {column_options.values()}\n\n"

    response = openai.Completion.create(
        engine="text-davinci-002",
        prompt=prompt,
        max_tokens=100,
        n=1,
        stop=None,
        temperature=0.5,
    )

    # Convert the response to a list of relevant columns
    relevant_columns = []
    for choice in response.choices:
        choice_text = choice.text.strip()
        for num, column in column_options.items():
            if column in choice_text:
                relevant_columns.append(num)

    return relevant_columns


# Function to prompt GPT-3 with a question about the data
def ask_gpt(question, data, relevant_columns):
    # Filter the DataFrame to keep only the relevant columns
    relevant_data = data.iloc[:, relevant_columns]

    prompt = f"{question} based on the following data:\n\n{relevant_data.to_string(index=False)}\n\nAnswer:"

    response = openai.Completion.create(
        engine="text-davinci-002",
        prompt=prompt,
        max_tokens=100,
        n=1,
        stop=None,
        temperature=0.5,
    )

    return response.choices[0].text.strip()


# Get user's question
user_question = input("Please enter your question: ")

# Ask GPT-3 which columns are relevant
relevant_columns = get_relevant_columns(user_question, column_options)

# Print out the relevant columns
print("The relevant columns for your question are:")
for column in relevant_columns:
    print(column_options[column])

# Ask GPT-3 the user's question with the relevant columns only
answer = ask_gpt(user_question, df, relevant_columns)
print(answer)
print(df)
