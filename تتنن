import os
import threading
from flask import Flask
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, ReplyKeyboardMarkup, Update
from telegram.ext import ApplicationBuilder, CommandHandler, CallbackQueryHandler, MessageHandler, filters, ContextTypes
import secrets

# تنظیمات
# پیشنهاد: TOKEN را در Render به عنوان متغیر محیطی تنظیم کنید
TOKEN = ("8342725594:AAG4RRLCNOx_9jo0unKajXO19dRKDWud5SU") 
CHANNEL_USERNAME = "@xzxdq"
OPTIONS = ["سنگ 🪨", "کاغذ 📄", "قیچی ✂️"]

# وب‌سرور کوچک برای زنده نگه داشتن Render
app = Flask(__name__)

@app.route('/')
def home():
    return "Bot is running!"

def run_web_server():
    port = int(os.environ.get("PORT", 8080))
    app.run(host="0.0.0.0", port=port)

# بقیه کدهای ربات
def get_start_keyboard():
    return ReplyKeyboardMarkup([['شروع بازی 🎮']], resize_keyboard=True)

async def check_membership(user_id, context):
    try:
        member = await context.bot.get_chat_member(chat_id=CHANNEL_USERNAME, user_id=user_id)
        if member.status in ['member', 'administrator', 'creator']:
            return True
        return False
    except Exception as e:
        print(f"خطا: {e}")
        return False

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    is_member = await check_membership(user_id, context)
    if not is_member:
        kb = InlineKeyboardMarkup([
            [InlineKeyboardButton("📢 عضویت در کانال", url=f"https://t.me/{CHANNEL_USERNAME.replace('@', '')}")],
            [InlineKeyboardButton("🔄 بررسی مجدد عضویت", callback_data='check_sub')]
        ])
        await update.message.reply_text(f"ابتدا در کانال {CHANNEL_USERNAME} عضو شوید:", reply_markup=kb)
        return
    await update.message.reply_text("سلام! برای شروع بازی روی دکمه پایین صفحه کلیک کن:", reply_markup=get_start_keyboard())

async def play_game(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_id = update.effective_user.id
    if not await check_membership(user_id, context):
        await update.message.reply_text(f"لطفاً اول در کانال {CHANNEL_USERNAME} عضو شوید!")
        return
    keyboard = [[InlineKeyboardButton("سنگ 🪨", callback_data='سنگ 🪨'),
                 InlineKeyboardButton("کاغذ 📄", callback_data='کاغذ 📄'),
                 InlineKeyboardButton("قیچی ✂️", callback_data='قیچی ✂️')]]
    await update.message.reply_text("از بین گزینه‌های زیر انتخاب کن:", reply_markup=InlineKeyboardMarkup(keyboard))

async def button_click(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    if query.data == 'check_sub':
        if await check_membership(update.effective_user.id, context):
            await query.edit_message_text("تأیید شد! حالا روی دکمه شروع بازی در پایین صفحه بزنید.")
        else:
            await query.answer("هنوز عضو نشده‌اید!", show_alert=True)
        return
    user_choice = query.data
    bot_choice = secrets.choice(OPTIONS)
    result = f"شما: {user_choice}\nمن: {bot_choice}\n"
    u, b = user_choice.split()[0], bot_choice.split()[0]
    if u == b: result += "مساوی شدیم! 😐"
    elif (u=="سنگ" and b=="قیچی") or (u=="کاغذ" and b=="سنگ") or (u=="قیچی" and b=="کاغذ"): result += "شما بردید! 🎉"
    else: result += "من بردم! 🤖"
    await context.bot.send_message(chat_id=query.message.chat_id, text=result, reply_markup=get_start_keyboard())

if __name__ == '__main__':
    # اجرای وب‌سرور در یک ترد جداگانه
    threading.Thread(target=run_web_server, daemon=True).start()
    
    # اجرای ربات
    application = ApplicationBuilder().token(TOKEN).build()
    application.add_handler(CommandHandler('start', start))
    application.add_handler(MessageHandler(filters.Text("شروع بازی 🎮"), play_game))
    application.add_handler(CallbackQueryHandler(button_click))
    print("ربات با وب‌سرور فعال شد...")
    application.run_polling()
