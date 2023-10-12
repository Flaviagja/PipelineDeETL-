import pandas as pd
import requests
import json
import re
from bardapi import Bard
import os

sdw2023_api_url = 'https://sdw-2023-prd.up.railway.app'

# abertura do arquivo csv
df = pd.read_csv('/content/SDW2023.CSV.csv')
user_ids = df['UserID'].tolist()
print(user_ids)


# Consultar banco de dados
def get_user(id):
    response = requests.get(f'{sdw2023_api_url}/users/{id}')
    return response.json() if response.status_code == 200 else None


users = [user for id in user_ids if (user := get_user(id)) is not None]
print(json.dumps(users, indent=2, ensure_ascii=False))
# |
# |-> (ensure_ascii=False) indicativo para a biblioteca json não remover caractere não=ASCII

# area de acesso ao Bard
os.environ['_BARD_API_KEY'] = 'bwiLgA2DFfyJDE2uPF1ZfXqSedFRFpG-XZtPhfUy25f4rIJZEfCM6S6RCaEEUw3cvvl-Jg.'


def gen_ia_news(user):
    input_text = f"Você é um especialista em marketing Bancário. Gere para {user['name']} um email sobre a importância dos investimentos com no máximo 100 caracteres"

    return Bard().get_answer(input_text)['content']


for user in users:
    news = gen_ia_news(user)
    news_format = re.sub(r'[\n\r]', '', news)
    news_format = re.sub(r'\s+', ' ', news_format)
    user.setdefault('news', []).append({
        "description": news_format
    })

print(json.dumps(users, indent=2, ensure_ascii=False))


def update_user(user):
    response = requests.put(f"{sdw2023_api_url}/users/{user['id']}", json=user)
    return True if response.status_code == 200 else False


for user in users:
    success = update_user(user)
    print(f"User {user['name']} update? {success}")




