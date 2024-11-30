from telegram import Update
from telegram.ext import Updater, CommandHandler, CallbackContext
from datetime import datetime

# FAQ funksiyasi
def faq(update: Update, context: CallbackContext) -> None:
    faq_text = (
        "📚 *Bot haqida ma'lumot:*\n\n"
        "Bu bot quyidagi vazifalarni bajaradi:\n"
        "1️⃣ /jadval - Bugungi kunning dars jadvalini ko‘rsatadi.\n"
        "2️⃣ /Fan <fani_nomi> - Kiritilgan fanning qaysi kunlarda borligini ko‘rsatadi.\n"
        "3️⃣ /kun - Bugungi kun bayrammi yoki oddiy kun ekanligini ko‘rsatadi va agar bayram bo‘lsa, tabriklaydi.\n"
        "4️⃣ /faq - Botning vazifalari va imkoniyatlari haqida ma'lumot beradi.\n\n"
        "💡 *Qo'shimcha ko'rsatma*: Har qanday buyruqni ishlatishda savollaringiz bo'lsa, shunchaki admin bilan bog'laning."
    )
    update.message.reply_text(faq_text, parse_mode="Markdown")


# Haftaning istalgan kuni uchun dars jadvalini so'rash
def specific_day_schedule(update: Update, context: CallbackContext) -> None:
    global week_schedule

    if context.args:  # Foydalanuvchi kun nomini yuborganmi?
        day_name = context.args[0].capitalize()  # Kun nomini katta harf bilan boshlaymiz
        if day_name in week_schedule:
            schedule = week_schedule[day_name] or f"{day_name} uchun dars jadvali kiritilmagan!"
            update.message.reply_text(f"{day_name} uchun dars jadvali:\n\n{schedule}")
        else:
            update.message.reply_text("Noto‘g‘ri kun nomi! Iltimos, 'Dushanba', 'Seshanba', ... kabi kun nomlarini yozing.")
    else:
        update.message.reply_text("Iltimos, kun nomini yuboring! Masalan:\n/jadval_kun Seshanba")

# Fan qaysi kunlarda borligini aniqlash
def subject_days(update: Update, context: CallbackContext) -> None:
    global week_schedule

    if context.args:  # Foydalanuvchi fan nomini yuborganmi?
        subject = " ".join(context.args)  # Fanning nomini olish
        days_with_subject = []

        # Har bir kun uchun jadvalni tekshirish
        for day, schedule in week_schedule.items():
            if subject in schedule:
                days_with_subject.append(day)

        if days_with_subject:
            days = ", ".join(days_with_subject)
            update.message.reply_text(f"{subject} fani quyidagi kunlarda bor:\n{days}")
        else:
            update.message.reply_text(f"{subject} fani haftalik jadvalda topilmadi.")
    else:
        update.message.reply_text("Iltimos, fanning nomini yuboring! Masalan:\n/Fan Matematika")

# Bayramlar ro‘yxati
holidays = {
    "01-01": "Yangi yil",
    "01-14": "Vatan himoyachilari kuni (O'zbekiston)",
    "02-14": "Valentin kuni",
    "03-08": "Xalqaro xotin-qizlar kuni",
    "03-21": "Navro'z bayrami (O'zbekiston)",
    "04-22": "Yer kuni",
    "05-01": "Xalqaro mehnat kuni",
    "05-09": "Xotira va qadrlash kuni (O'zbekiston)",
    "06-01": "Xalqaro bolalar kuni",
    "06-20": "Xalqaro musiqalar kuni",
    "07-30": "Do'stlik kuni",
    "08-31": "Mustaqillik bayrami arafasi (O'zbekiston)",
    "09-01": "Mustaqillik kuni (O'zbekiston)",
    "10-01": "O‘qituvchi va murabbiylar kuni (O'zbekiston)",
    "11-18": "O‘zbekiston Davlat bayrog‘i qabul qilingan kun",
    "12-08": "O‘zbekiston Konstitutsiyasi kuni",
    "12-31": "Yangi yil arafasi",
}

# Bugungi kun haqida ma'lumot beruvchi funksiya
def today_info(update: Update, context: CallbackContext) -> None:
    # Bugungi sanani olish
    today_date = datetime.now().strftime("%m-%d")  # Format: MM-DD

    # Bayramni tekshirish
    if today_date in holidays:
        holiday_name = holidays[today_date]
        message = (
            f"Bugun {holiday_name}! 🎉\n\n"
            f"Tabriklaymiz! Sizga ushbu {holiday_name} kuni sog‘lik, baxt, va omad tilaymiz! 😊"
        )
    else:
        message = "Bugun oddiy kun. Bayram emas. Ishlaringizga omad! 😊"

    update.message.reply_text(message)


# 1 haftalik jadvalni saqlash uchun
week_schedule = {
    "Dushanba": "",
    "Seshanba": "",
    "Chorshanba": "",
    "Payshanba": "",
    "Juma": "",
    "Shanba": "",
    "Yakshanba": ""
}

# Haftaning kunlariga mos indeks
week_days = {
    1: "Dushanba",
    2: "Seshanba",
    3: "Chorshanba",
    4: "Payshanba",
    5: "Juma",
    6: "Shanba",
    7: "Yakshanba"
}

# Foydalanuvchiga /create bo‘yicha ko‘rsatma berish
def create_instruction(update: Update, context: CallbackContext) -> None:
    message = (
        "Dars jadvalni kiritish uchun quyidagi buyrug‘lardan foydalaning:\n\n"
        "/create1 - Dushanba uchun dars jadvali\n"
        "/create2 - Seshanba uchun dars jadvali\n"
        "/create3 - Chorshanba uchun dars jadvali\n"
        "/create4 - Payshanba uchun dars jadvali\n"
        "/create5 - Juma uchun dars jadvali\n"
        "/create6 - Shanba uchun dars jadvali\n"
        "/create7 - Yakshanba uchun dars jadvali\n\n"
        "Masalan: Matematika, Informatika, Ona-tili, Ingliz-tili"
    )
    update.message.reply_text(message)

# Foydalanuvchi dars jadvalini kiritish uchun ishlatiladigan funksiya
def create_schedule(update: Update, context: CallbackContext) -> None:
    global week_schedule
    
    # Buyruq orqali qaysi kunni tahrirlash kerakligini aniqlash
    command = update.message.text.split()[0]
    day_index = int(command.replace("/create", ""))
    day_name = week_days.get(day_index)
    
    # Agar foydalanuvchi noto‘g‘ri kun yuborgan bo‘lsa
    if not day_name:
        update.message.reply_text("Noto‘g‘ri buyruq! Faqat /create1 dan /create7 gacha buyrug‘larni ishlating.")
        return

    # Kiritilayotgan kun haqida ma'lumot berish
    if context.args:
        # Dars jadvalni yangilash
        schedule = " ".join(context.args)
        week_schedule[day_name] = schedule
        update.message.reply_text(f"{day_name} uchun dars jadvali muvaffaqiyatli kiritildi:\n{schedule}")
    else:
        # Agar foydalanuvchi dars jadvalini kiritmagan bo‘lsa
        update.message.reply_text(
            f"{day_name} uchun dars jadvalini kiritish uchun darslarni quyidagi formatda yuboring:\n\n"
            "Matematika, Informatika, Ona-tili, Ingliz-tili"
        )

# Bugungi jadvalni qaytarish
def today_schedule_command(update: Update, context: CallbackContext) -> None:
    today_index = datetime.now().weekday() + 1  # 1 - Dushanba, 7 - Yakshanba
    today_day = week_days.get(today_index)
    today_schedule = week_schedule.get(today_day, "Bugungi dars jadvali kiritilmagan!")
    update.message.reply_text(f"Bugungi dars jadvali ({today_day}):\n\n{today_schedule}")

def main():
    # @BotFather dan olingan TOKEN
    TOKEN = "7529857134:AAGHsmrECh76N1O2l7rD3_dvKZlp9zTPgr0"
    
    # Botni sozlash
    updater = Updater(TOKEN)
    dispatcher = updater.dispatcher

    # Buyruqlarni sozlash
    dispatcher.add_handler(CommandHandler("faq", faq))  # /faq qo‘shildi
    dispatcher.add_handler(CommandHandler("kun", today_info))  # /kun
    dispatcher.add_handler(CommandHandler("Fan", subject_days))  # /Fan
    dispatcher.add_handler(CommandHandler("jadval_kun", specific_day_schedule))  # /jadval_kun
    dispatcher.add_handler(CommandHandler("create", create_instruction))  # /create
    dispatcher.add_handler(CommandHandler("create1", create_schedule))  # /create1
    dispatcher.add_handler(CommandHandler("create2", create_schedule))  # /create2
    dispatcher.add_handler(CommandHandler("create3", create_schedule))  # /create3
    dispatcher.add_handler(CommandHandler("create4", create_schedule))  # /create4
    dispatcher.add_handler(CommandHandler("create5", create_schedule))  # /create5
    dispatcher.add_handler(CommandHandler("create6", create_schedule))  # /create6
    dispatcher.add_handler(CommandHandler("create7", create_schedule))  # /create7
    dispatcher.add_handler(CommandHandler("jadval", today_schedule_command))  # /jadval

    # Botni ishga tushirish
    updater.start_polling()
    updater.idle()

if __name__ == '__main__':
    main()
