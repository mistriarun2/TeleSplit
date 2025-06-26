# TeleSplit
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes
import re
import os

def clean_phone_number(raw):
    number = re.sub(r'\D', '', raw)
    if not number.startswith('+' or '00'):
        number = '+' + number
    else:
        number = '+' + number.lstrip('0')
    return number

def is_valid(number):
    digits_only = number.replace('+', '')
    return 8 <= len(digits_only) <= 15

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("Send phone numbers and I will clean and generate t.me links!")

async def handle_numbers(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    raw_numbers = re.split(r'[,\s]+', text)
    cleaned = set()
    results = []
    for raw in raw_numbers:
        if raw.strip() == '':
            continue
        number = clean_phone_number(raw)
        if is_valid(number) and number not in cleaned:
            cleaned.add(number)
            results.append(f"{number} → t.me/{number}")
    if results:
        await update.message.reply_text('\n'.join(results))
    else:
        await update.message.reply_text('❌ No valid numbers found.')

def main():
    token = os.getenv("BOT_TOKEN")
    app = ApplicationBuilder().token(token).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_numbers))
    app.run_polling()

if __name__ == "__main__":
    main()
