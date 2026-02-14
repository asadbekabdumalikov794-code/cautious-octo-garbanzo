bot.py
from telegram import (
    Update, InlineKeyboardButton, InlineKeyboardMarkup
)
from telegram.ext import (
    Application, CommandHandler, MessageHandler,
    CallbackQueryHandler, ContextTypes, filters
)

API_TOKEN = "7981739807:AAHRcSmzdzg8-qqhHkhH71qNnfwQaHBsNKE"
KANAL = "@hasankino_kanal"
ADMIN_ID = 7464152329 # o'zingizni ID yozing

# Kino bazasi
movies = {}

# --- Obuna tekshirish ---
async def check_sub(user_id, context):
    member = await context.bot.get_chat_member(KANAL, user_id)
    return member.status in ["member", "administrator", "creator"]

# --- Start ---
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id

    if not await check_sub(user_id, context):
        keyboard = [
            [InlineKeyboardButton("📢 Kanalga obuna bo‘lish", url=f"https://t.me/{KANAL[1:]}")]
        ]
        await update.message.reply_text(
            "❗ Botdan foydalanish uchun kanalga obuna bo‘ling!",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )
        return

    keyboard = [
        [InlineKeyboardButton("🎬 Kino kod yuborish", callback_data="kino")],
    ]

    if user_id == ADMIN_ID:
        keyboard.append([InlineKeyboardButton("⚙️ Admin panel", callback_data="admin")])

    await update.message.reply_text(
        "Asosiy menyu:",
        reply_markup=InlineKeyboardMarkup(keyboard)
    )

# --- Tugma ishlashi ---
async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()

    if query.data == "kino":
        await query.message.reply_text("🎬 Kino kodini yuboring:")

    elif query.data == "admin":
        keyboard = [
            [InlineKeyboardButton("➕ Kino qo‘shish", callback_data="add_movie")]
        ]
        await query.message.reply_text(
            "⚙️ Admin panel",
            reply_markup=InlineKeyboardMarkup(keyboard)
        )

    elif query.data == "add_movie":
        await query.message.reply_text(
            "Format:\nKOD|NOMI|LINK\n\nMisol:\n101|Inception|https://t.me/file.mp4"
        )
        context.user_data["adding"] = True

# --- Kino qo‘shish (admin) ---
async def text_handler(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    text = update.message.text

    # Admin kino qo'shmoqda
    if context.user_data.get("adding") and user_id == ADMIN_ID:
        try:
            kod, nomi, link = text.split("|")
            movies[kod] = {"name": nomi, "link": link}
            await update.message.reply_text("✅ Kino qo‘shildi!")
            context.user_data["adding"] = False
        except:
            await update.message.reply_text("❌ Format noto‘g‘ri!")
        return

    # Oddiy foydalanuvchi kino kodi yozmoqda
    if text in movies:
        kino = movies[text]
        await update.message.reply_text(
            f"🎬 {kino['name']}\n🔗 Yuklab olish: {kino['link']}"
        )
    else:
        await update.message.reply_text("❌ Bunday kodli kino yo‘q!")

# --- Bot ishga tushishi ---
app = Application.builder().token(API_TOKEN).build()

app.add_handler(CommandHandler("start", start))
app.add_handler(CallbackQueryHandler(button))
app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, text_handler))

app.run_polling()
