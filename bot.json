import logging
from telegram import Update
from telegram.ext import Application, CommandHandler, ContextTypes
import requests
from datetime import datetime

# Biến lưu job
jobs = {}

# Link ảnh mặc định
IMAGE_URL = "https://hoanghamobile.com/tin-tuc/wp-content/uploads/2024/05/anh-conan-ngau.jpg"

# Hàm lấy thông tin thời tiết từ wttr.in (free)
def get_weather(city: str):
    url = f"https://wttr.in/{city}?format=j1"
    try:
        data = requests.get(url).json()
        current = data["current_condition"][0]

        temp = current["temp_C"]
        feels = current["FeelsLikeC"]
        humidity = current["humidity"]
        desc = current["weatherDesc"][0]["value"]
        wind = current["windspeedKmph"]
        pressure = current["pressure"]
        visibility = current["visibility"]

        now = datetime.now().strftime("%d/%m %H:%M")

        return (
            f"🌍 Thời tiết tại **{city.capitalize()}** (cập nhật: {now}):\n"
            f"🌡 Nhiệt độ: {temp}°C (cảm giác: {feels}°C)\n"
            f"💧 Độ ẩm: {humidity}%\n"
            f"🌤 Trạng thái: {desc}\n"
            f"💨 Gió: {wind} km/h\n"
            f"🔽 Áp suất: {pressure} hPa\n"
            f"👁 Tầm nhìn: {visibility} km"
        )
    except Exception as e:
        return f"❌ Lỗi khi lấy thời tiết: {e}"

# /start
async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("✅ Bot đã sẵn sàng! Hãy dùng lệnh /thoitiet <địa điểm>")

# Job cập nhật mỗi 1h
async def weather_job(context: ContextTypes.DEFAULT_TYPE):
    job = context.job
    city = job.data["city"]
    chat_id = job.data["chat_id"]
    await context.bot.send_photo(
        chat_id=chat_id,
        photo=IMAGE_URL,
        caption=get_weather(city),
        parse_mode="Markdown"
    )

# /thoitiet
async def thoitiet(update: Update, context: ContextTypes.DEFAULT_TYPE):
    if not context.args:
        await update.message.reply_text("⚠️ Vui lòng nhập địa điểm!\nVí dụ: `/thoitiet Hanoi`", parse_mode="Markdown")
        return

    city = " ".join(context.args)
    text = get_weather(city)
    chat_id = update.message.chat_id

    # Gửi ảnh từ link + thời tiết
    await update.message.reply_photo(photo=IMAGE_URL,
                                     caption=text,
                                     parse_mode="Markdown")

    # Nếu có job cũ thì hủy
    if chat_id in jobs:
        jobs[chat_id].schedule_removal()

    # Tạo job mới
    job = context.job_queue.run_repeating(
        weather_job, interval=3600, first=3600,
        data={"city": city, "chat_id": chat_id}
    )
    jobs[chat_id] = job
    await update.message.reply_text("🔄 Sẽ tự động cập nhật thời tiết mỗi 1h!")

# /stop
async def stop(update: Update, context: ContextTypes.DEFAULT_TYPE):
    chat_id = update.message.chat_id
    if chat_id in jobs:
        jobs[chat_id].schedule_removal()
        del jobs[chat_id]
        await update.message.reply_text("🛑 Đã dừng cập nhật thời tiết.")
    else:
        await update.message.reply_text("⚠️ Không có tiến trình cập nhật nào đang chạy.")

# Main
def main():
    logging.basicConfig(level=logging.INFO)
    TOKEN = "YOUR_TELEGRAM_BOT_TOKEN"  # thay token bot của bạn
    app = Application.builder().token(TOKEN).build()

    app.add_handler(CommandHandler("start", start))
    app.add_handler(CommandHandler("thoitiet", thoitiet))
    app.add_handler(CommandHandler("stop", stop))

    app.run_polling()

if __name__ == "__main__":
    main()
