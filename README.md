# udarmovie-bot
AIogram asosidagi Telegram bot. TMDB API orqali kinolar haqida tavsif, rasm va treylerlarni ko‘rsatadi.
import telebot
from telebot import types

TOKEN = "7965633823:AAHHwpgUe0fXiP80G4jb1bGj1INJTOcBw7Q"
ADMIN_ID = 5775346497
CHANNEL = "@Tarjima_kinoNI"

bot = telebot.TeleBot(TOKEN)
movies = {}

def check_sub(user_id):
    try:
        status = bot.get_chat_member(CHANNEL, user_id).status
        return status in ["member", "administrator", "creator"]
    except:
        return False

@bot.message_handler(commands=['start'])
def start(message):
    if not check_sub(message.from_user.id):
        btn = types.InlineKeyboardMarkup()
        btn.add(types.InlineKeyboardButton("📢 Kanalga obuna bo‘lish", url=f"https://t.me/{CHANNEL[1:]}"))
        bot.send_message(message.chat.id, "❗ Avval kanalga obuna bo‘ling", reply_markup=btn)
        return
    bot.send_message(message.chat.id, "🎬 Kino kodi yuboring\nAdmin: /add")

@bot.message_handler(commands=['add'])
def add_movie(message):
    if message.from_user.id != ADMIN_ID:
        return
    bot.send_message(message.chat.id, "Kino kodini yuboring:")
    bot.register_next_step_handler(message, get_code)

def get_code(message):
    code = message.text
    bot.send_message(message.chat.id, "Videoni yuboring:")
    bot.register_next_step_handler(message, get_video, code)

def get_video(message, code):
    if not message.video:
        bot.send_message(message.chat.id, "❌ Video yuboring")
        return
    movies[code] = message.video.file_id
    bot.send_message(message.chat.id, f"✅ Saqlandi\nKodi: {code}")

@bot.message_handler(content_types=['text'])
def send_movie(message):
    if message.text in movies:
        bot.send_video(message.chat.id, movies[message.text])
    else:
        bot.send_message(message.chat.id, "❌ Topilmadi")

bot.infinity_polling()
pyTelegramBotAPI
services:
  - type: worker
    name: udarmovie-bot
    env: python
    buildCommand: pip install -r requirements.txt
    main.py
render.yaml
    startCommand: python main.py
