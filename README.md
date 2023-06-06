# SETUP

1. `cp .env.example .env`
1. `OPENAI_API_KEY=` を設定
1. docker compose up
1. http://127.0.0.1:8000/qa にアクセス

app.py控え
```
from flask import Flask, request, render_template, session, redirect, url_for
import openai
from dotenv import load_dotenv
import os

load_dotenv()

app = Flask(__name__)
app.secret_key = os.getenv('SECRET_KEY', 'default-secret-key')  # add a secret key for the session

openai.api_key = os.getenv('OPENAI_API_KEY')

@app.route('/qa', methods=['GET', 'POST'])
def home():
   if 'chat_history' not in session:
       session['chat_history'] = []

   if request.method == 'POST':
       user_message = request.form.get('message')
       single_message = [{'role': 'user', 'content': user_message}]

       response = openai.ChatCompletion.create(
           model="gpt-3.5-turbo",
           messages=single_message
       )

       bot_message = response.choices[0].message['content']
       session['chat_history'].append((user_message, bot_message))

   return render_template('chat.html', chat_history=session['chat_history'])

@app.route('/clear', methods=['POST'])
def clear():
   session.pop('chat_history', None)
   return redirect(url_for('home'))

if __name__ == '__main__':
   app.run(host='0.0.0.0', port=8000)

```
