from aiogram import Bot, Dispatcher, types
from aiogram.types import InlineKeyboardMarkup, InlineKeyboardButton
from aiogram.utils import executor

TOKEN = "7441165567:AAG9mVCtWpe05XvfUNgHfjsiH9qEWcoMHtE"
bot = Bot(token=TOKEN, parse_mode="HTML")
dp = Dispatcher(bot)

# UC narxlari va valyuta kurslari
UC_PRICES = {
    325: 39000,
    660: 75000,
    1800: 195000,
    3850: 410000,
    8100: 830000,
}

USD_RATE = 12300  # 1 USD = 12300 UZS
RUB_RATE = 140    # 1 RUB = 140 UZS

# Start komandasi
@dp.message_handler(commands=['start'])
async def send_welcome(message: types.Message):
    markup = InlineKeyboardMarkup()
    markup.add(InlineKeyboardButton("🔹 Buy", callback_data="buy"))
    markup.add(InlineKeyboardButton("🌍 Language", callback_data="language"))
    markup.add(InlineKeyboardButton("⚙ Settings", callback_data="settings"))
    markup.add(InlineKeyboardButton("ℹ Help", callback_data="help"))

    await message.answer("👋 Assalomu alaykum!\n🎮 PUBG UC sotib olish uchun <b>PUBG ID</b> va <b>UC miqdorini</b> kiriting.", reply_markup=markup)

# UC sotib olish jarayoni
@dp.callback_query_handler(lambda c: c.data == "buy")
async def enter_pubg_id(callback_query: types.CallbackQuery):
    await bot.send_message(callback_query.from_user.id, "🆔 Iltimos, PUBG ID kiriting:")
    await callback_query.answer()

@dp.message_handler(lambda message: message.text.isdigit() and len(message.text) in [8, 9, 10])
async def pubg_id_received(message: types.Message):
    markup = InlineKeyboardMarkup()
    for uc, price in UC_PRICES.items():
        price_usd = round(price / USD_RATE, 2)
        price_rub = round(price / RUB_RATE, 2)
        btn_text = f"{uc} UC - {price} UZS (~{price_usd}$ / ~{price_rub}₽)"
        markup.add(InlineKeyboardButton(btn_text, callback_data=f"uc_{uc}"))

    await message.answer("✅ <b>PUBG ID qabul qilindi!</b> Endi UC miqdorini tanlang:", reply_markup=markup)

@dp.callback_query_handler(lambda c: c.data.startswith("uc_"))
async def uc_selected(callback_query: types.CallbackQuery):
    uc_amount = int(callback_query.data.split("_")[1])
    price = UC_PRICES[uc_amount]
    price_usd = round(price / USD_RATE, 2)
    price_rub = round(price / RUB_RATE, 2)

    markup = InlineKeyboardMarkup()
    markup.add(InlineKeyboardButton("💳 To‘lov qildim", callback_data="payment_done"))
    markup.add(InlineKeyboardButton("❌ Bekor qilish", callback_data="cancel"))

    await bot.send_message(
        callback_query.from_user.id,
        f"💰 <b>To‘lov tafsilotlari:</b>\n\n"
        f"🛒 {uc_amount} UC narxi: {price} UZS (~{price_usd}$ / ~{price_rub}₽)\n"
        f"💳 To‘lov kartasi: <code>9860160637918600</code>\n\n"
        f"To‘lovni amalga oshirib, \"To‘lov qildim\" tugmasini bosing.",
        reply_markup=markup
    )
    await callback_query.answer()

@dp.callback_query_handler(lambda c: c.data == "payment_done")
async def payment_done(callback_query: types.CallbackQuery):
    markup = InlineKeyboardMarkup()
    markup.add(InlineKeyboardButton("📷 Chekni yuborish", callback_data="send_receipt"))

    await bot.send_message(callback_query.from_user.id, "📸 Iltimos, to‘lov chekini yuboring:", reply_markup=markup)
    await callback_query.answer()

@dp.message_handler(content_types=['photo'])
async def receive_receipt(message: types.Message):
    admin_id = 7909176880  # Admin ID
    await bot.send_message(
        admin_id,
        f"📢 <b>Yangi to‘lov tasdiqlashga keldi!</b>\n"
        f"👤 Foydalanuvchi: {message.from_user.full_name}\n"
        f"🆔 Telegram ID: {message.from_user.id}\n\n"
        f"⏳ Tasdiqlash kutilmoqda..."
    )
    await bot.forward_message(admin_id, message.chat.id, message.message_id)

    markup = InlineKeyboardMarkup()
    markup.add(InlineKeyboardButton("✅ Tasdiqlash", callback_data="confirm_payment"))
    markup.add(InlineKeyboardButton("❌ Rad etish", callback_data="reject_payment"))

    await bot.send_message(admin_id, "🔍 To‘lovni tasdiqlang:", reply_markup=markup)
    await message.answer("✅ Chek qabul qilindi! Tasdiqlash kutilmoqda.")

@dp.callback_query_handler(lambda c: c.data == "confirm_payment")
async def confirm_payment(callback_query: types.CallbackQuery):
    await bot.send_message(callback_query.from_user.id, "✅ To‘lov tasdiqlandi! UC 10 daqiqada hisobingizga tushadi.")
    await callback_query.answer()

@dp.callback_query_handler(lambda c: c.data == "reject_payment")
async def reject_payment(callback_query: types.CallbackQuery):
    await bot.send_message(callback_query.from_user.id, "❌ To‘lov rad etildi. Iltimos, qayta urinib ko‘ring.")
    await callback_query.answer()

if __name__ == '__main__':
    executor.start_polling(dp, skip_updates=True)
