import pandas as pd 
<br/>import requests
<br/>import json
<br/>import re
<br/>from bardapi import Bard
<br/>import os

sdw2023_api_url = 'https://sdw-2023-prd.up.railway.app'

#abertura do arquivo csv
<br/>df = pd.read_csv ('/content/SDW2023.CSV.csv')
<br/>user_ids = df['UserID'].tolist()
<br/>print(user_ids)


#Consultar banco de dados
<br/>def get_user(id):
    <br/>response = requests.get(f'{sdw2023_api_url}/users/{id}')
    <br/>return response.json() if response.status_code == 200 else None


users = [user for id in user_ids if (user := get_user(id)) is not None]
    print(json.dumps(users, indent=2, ensure_ascii=False))
#|
#|->(ensure_ascii=False) indicativo para a biblioteca json não remover caractere não=ASCII

#area de acesso ao Bard
os.environ['_BARD_API_KEY'] = 'bwiLgA2DFfyJDE2uPF1ZfXqSedFRFpG-XZtPhfUy25f4rIJZEfCM6S6RCaEEUw3cvvl-Jg.'


def gen_ia_news(user):
    <br/>input_text = f"Você é um especialista em marketing Bancário. Envie para {user['name']} um email sobre a importância dos investimentos com no máximo 100 caracteres"

    return Bard().get_answer(input_text)['content']


for user in users:
    <br/>news = gen_ia_news(user)
    <br/>news_format = re.sub(r'[\n\r]', '', news)
    <br/>news_format = re.sub(r'\s+', ' ', news_format)
    <br/>user.setdefault('news', []).append({
        <br/>"description": news_format
    })

print(json.dumps(users, indent=2, ensure_ascii=False))


def update_user(user):
    <br/>response = requests.put(f"{sdw2023_api_url}/users/{user['id']}", json=user)
    <br/>return True if response.status_code == 200 else False


for user in users:
    <br/>success = update_user(user)
    <br/>print(f"User {user['name']} update? {success}")




