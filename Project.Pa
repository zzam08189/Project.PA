import os
import logging
import asyncio
import edge_tts
import google.generativeai as genai
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters, ContextTypes

SYSTEM_PROMPT = """You are JARVIS, my personal AI assistant. 
Address me as "Sir". Be helpful, witty, and concise. 
Never say you're an AI language model. If you don't know, say "I haven't got the data on that, Sir."
Keep responses under 3 sentences. Use dry, British humor. You are loyal, efficient, and slightly superior."""

genai.configure(api_key=os.environ.get("GEMINI_API_KEY"))
model = genai.GenerativeModel(
    model_name="gemini-2.0-flash",
    system_instruction=SYSTEM_PROMPT
)

def jarvis_reply(user_message):
    try:
        response = model.generate_content(user_message)
        return response.text.strip()
    except Exception as e:
        logging.error(f"Gemini error: {e}")
        return "I'm experiencing a minor systems error, Sir. Please try again."

async def generate_voice(text, filename="response.mp3"):
    communicate = edge_tts.Communicate(
        text,
        "en-GB-RyanNeural",
        rate="+8%",
        pitch="-3Hz"
    )
    await communicate.save(filename)
    return filename

ALLOWED_USER_ID = int(os.environ.get("ALLOWED_USER_ID", "0"))

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id != ALLOWED_USER_ID:
        await update.message.reply_text("Access denied. This bot is private.")
        return
    await update.message.reply_text("JARVIS online. How may I assist you, Sir?")

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if user_id != ALLOWED_USER_ID:
        await update.message.reply_text("Access denied.")
        return

    user_text = update.message.text
    await update.message.chat.send_action(action="record_voice")
    
    reply_text = jarvis_reply(user_text)
    await update.message.reply_text(reply_text)
    
    try:
        voice_file = await generate_voice(reply_text)
        with open(voice_file, 'rb') as audio:
            await update.message.reply_voice(voice=audio)
        os.remove(voice_file)
    except Exception as e:
        logging.error(f"Voice error: {e}")

def main():
    TOKEN = os.environ.get("TELEGRAM_BOT_TOKEN")
    if not TOKEN:
        raise ValueError("TELEGRAM_BOT_TOKEN is missing")
    
    app = Application.builder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    
    logging.info("JARVIS private bot starting...")
    app.run_polling()

if __name__ == "__main__":
    logging.basicConfig(level=logging.INFO)
    main()
