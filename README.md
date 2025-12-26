import telebot
from telebot import types

TOKEN = "8464325964:AAHiJ02eRAXRZzRcHqs7LtqPGAAtSmX-dYA"
bot = telebot.TeleBot(TOKEN)

waiting = []
pairs = {}


def main_menu():
    kb = types.ReplyKeyboardMarkup(resize_keyboard=True)
    kb.add("🔍 Найти собеседника")
    kb.add("❌ Остановить чат")
    return kb


@bot.message_handler(commands=["start"])
def start(message):
    bot.send_message(
        message.chat.id,
        "👋 Добро пожаловать в анонимный чат!\n"
        "Нажми «Найти собеседника»",
        reply_markup=main_menu()
    )


@bot.message_handler(func=lambda m: m.text == "🔍 Найти собеседника")
def find_partner(message):
    user_id = message.chat.id

    if user_id in pairs:
        bot.send_message(user_id, "❗ Ты уже в чате")
        return

    if user_id in waiting:
        bot.send_message(user_id, "⏳ Мы уже ищем собеседника")
        return

    if waiting:
        partner = waiting.pop(0)
        pairs[user_id] = partner
        pairs[partner] = user_id

        bot.send_message(user_id, "✅ Собеседник найден!")
        bot.send_message(partner, "✅ Собеседник найден!")
    else:
        waiting.append(user_id)
        bot.send_message(user_id, "⏳ Ищем собеседника...")


@bot.message_handler(func=lambda m: m.text == "❌ Остановить чат")
def stop_chat(message):
    user_id = message.chat.id
    partner = pairs.pop(user_id, None)

    if partner:
        pairs.pop(partner, None)
        bot.send_message(partner, "❌ Собеседник вышел из чата")

    if user_id in waiting:
        waiting.remove(user_id)

    bot.send_message(user_id, "❌ Чат остановлен", reply_markup=main_menu())


@bot.message_handler(func=lambda m: m.chat.id in pairs)
def relay(message):
    partner = pairs.get(message.chat.id)
    if partner:
        bot.send_message(partner, message.text)


bot.infinity_polling()
