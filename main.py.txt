import os
import random
import string
from telegram import Update
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, filters, ContextTypes

# Tomar token de Telegram desde la variable de entorno
TOKEN = os.environ.get("TOKEN")

# Guardar usuarios y correos temporales
usuarios = {}       # chat_id -> correo temporal
mensajes_temp = {}  # correo -> lista de mensajes/OTP

def generar_correo():
    """Genera un correo temporal aleatorio"""
    nombre = ''.join(random.choices(string.ascii_lowercase + string.digits, k=6))
    return f"{nombre}@tempbot.com"

# Comando /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = update.message.chat_id
    correo = generar_correo()
    usuarios[chat_id] = correo
    mensajes_temp[correo] = []
    await update.message.reply_text(
        f"Tu correo temporal es: {correo}\nÚsalo donde necesites y cualquier mensaje/OTP que llegue será enviado aquí."
    )

# Recibir cualquier mensaje enviado al bot (simula OTP)
async def recibir_mensaje(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = update.message.chat_id
    texto = update.message.text

    if chat_id not in usuarios:
        await update.message.reply_text("Primero escribe /start para generar tu correo temporal.")
        return

    correo = usuarios[chat_id]
    mensajes_temp[correo].append(texto)
    await update.message.reply_text(
        f"Mensaje/OTP recibido en {correo} ✅\nMensajes en tu bandeja: {len(mensajes_temp[correo])}"
    )

# Comando /inbox para ver los mensajes recibidos
async def inbox(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = update.message.chat_id
    if chat_id not in usuarios:
        await update.message.reply_text("Primero escribe /start para generar tu correo temporal.")
        return

    correo = usuarios[chat_id]
    inbox_mensajes = mensajes_temp.get(correo, [])

    if not inbox_mensajes:
        await update.message.reply_text("Tu bandeja está vacía 📭")
        return

    texto = "\n\n".join(inbox_mensajes)
    await update.message.reply_text(f"📬 Mensajes en {correo}:\n\n{texto}")

# Iniciar el bot
if __name__ == "__main__":
    app = ApplicationBuilder().token(TOKEN).build()
    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("inbox", inbox))
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, recibir_mensaje))
    print("Bot de correos temporales iniciado...")
    app.run_polling()
