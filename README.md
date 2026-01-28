# bot-revolution-energie

# bot.py
import logging
from telegram import (
    Update, ReplyKeyboardMarkup, KeyboardButton
)
from telegram.ext import (
    ApplicationBuilder,
    CommandHandler,
    MessageHandler,
    ContextTypes,
    filters
)

from database import init_db, save_request
from ai import ai_analyse

TOKEN = "7894156599:AAEIea1TO9U1V_pCcFpBBvrRI-rClTZcsXU"

logging.basicConfig(level=logging.INFO)

users = {}

# ================= START =================
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = [
        [KeyboardButton("📞 Contact", request_contact=True)],
        [KeyboardButton("📍 Adresse", request_location=True)]
    ]
    reply_markup = ReplyKeyboardMarkup(keyboard, resize_keyboard=True)

    await update.message.reply_text(
        "👋 Bonjour, je suis Manuel Bot, l’assistant de Revolution Energie.\n"
        "Veuillez utiliser le menu pour continuer.",
        reply_markup=reply_markup
    )

# ================= MESSAGES =================
async def messages(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    text = update.message.text.lower()

    if user.id not in users:
        users[user.id] = {}

    if "merci" in text:
        await update.message.reply_text(
            "🙏 Le plaisir est partagé.\n"
            "Je suis là pour vous aider 24h/24 et 7j/7.\n"
            "N’hésitez surtout pas à revenir."
        )
        return

    if text not in ["contact", "adresse"]:
        response = ai_analyse(text)
        users[user.id]["probleme"] = response

        await update.message.reply_text(
            "⚠️ Veuillez utiliser le menu des options pour pouvoir vous aider "
            "le plus rapidement possible.\n\n"
            "🤖 Moi je suis Manuel Bot, l’assistant de Revolution Energie."
        )
        return

# ================= CONTACT =================
async def contact_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    contact = update.message.contact.phone_number

    save_request(user.id, f"Contact: {contact}")

    await update.message.reply_text(
        "✅ Merci. Notre équipe vous contactera rapidement."
    )

# ================= LOCALISATION =================
async def location_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user = update.effective_user
    loc = update.message.location

    save_request(user.id, f"Location: {loc.latitude},{loc.longitude}")

    await update.message.reply_text(
        "📍 Merci. Notre équipe vous contactera rapidement."
    )

# ================= MAIN =================
def main():
    init_db()

    app = ApplicationBuilder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.CONTACT, contact_handler))
    app.add_handler(MessageHandler(filters.LOCATION, location_handler))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, messages))

    print("🤖 Manuel Bot est en ligne 24/7")
    app.run_polling()

if __name__ == "__main__":
    main()

# database.py
import sqlite3

def init_db():
    conn = sqlite3.connect("clients.db")
    cursor = conn.cursor()
    cursor.execute("""
        CREATE TABLE IF NOT EXISTS requests (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            user_id INTEGER,
            message TEXT
        )
    """)
    conn.commit()
    conn.close()

def save_request(user_id, message):
    conn = sqlite3.connect("clients.db")
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO requests (user_id, message) VALUES (?, ?)",
        (user_id, message)
    )
    conn.commit()
    conn.close()

# ai.py
def ai_analyse(text):
    return f"Analyse reçue: {text}"
