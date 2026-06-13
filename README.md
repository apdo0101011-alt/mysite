import asyncio
import aiohttp
import telebot
from telebot import types
import random

# 🔑 التوكن الفخم الخاص بك
BOT_TOKEN = "8737066685:AAG-q6j2SB_6idi-p3d5wb1K5Wrf5UsjifA"
bot = telebot.TeleBot(BOT_TOKEN)

print("👑 [إصدار غوجو الـ VIP المطور - نظام المجموعات والتحكم الكامل] قيد التشغيل...")

# 🌐 مصادر بروكسيات فائقة الجودة
PROXY_SOURCES = [
    "https://api.proxyscrape.com/v2/?request=displayproxies&protocol=http&timeout=10000&country=all&ssl=all&anonymity=all",
    "https://www.proxy-list.download/api/v1/get?type=http",
    "https://raw.githubusercontent.com/TheSpeedX/PROXY-List/master/http.txt",
    "https://raw.githubusercontent.com/ShiftyTR/Proxy-List/master/http.txt",
    "https://raw.githubusercontent.com/monosans/proxy-list/main/proxies/http.txt",
    "https://raw.githubusercontent.com/hookzof/socks5list/master/proxy.txt",
    "https://raw.githubusercontent.com/clarketm/proxy-list/master/proxy-list-raw.txt",
    "https://raw.githubusercontent.com/sunny9577/proxy-lexer/master/proxies.txt"
]

# متغيرات التحكم بالنظام
is_hunting_active = False
current_batch_proxies = []
sent_batches_count = 0
MAX_BATCHES = 10  # عدد المرات المطلوب إرسال 20 بروكسي فيها

# 📥 جلب البروكسيات الخام
async def fetch_all_raw_proxies():
    raw_list = []
    timeout = aiohttp.ClientTimeout(total=8)
    async with aiohttp.ClientSession(timeout=timeout) as session:
        for source in PROXY_SOURCES:
            try:
                async with session.get(source) as res:
                    if res.status == 200:
                        text = await res.text()
                        for line in text.splitlines():
                            line = line.strip()
                            if ":" in line and not line.startswith("#"):
                                raw_list.append(line)
            except:
                continue
    return list(set(raw_list))

# 🎨 لوحة التحكم الرئيسية (بدء / إيقاف)
def main_control_keyboard():
    markup = types.InlineKeyboardMarkup(row_width=2)
    btn_start = types.InlineKeyboardButton("🚀 بدء الفحص", callback_data="start_hunt")
    btn_stop = types.InlineKeyboardButton("🛑 إيقاف الفحص", callback_data="stop_hunt")
    markup.add(btn_start, btn_stop)
    return markup

# 🕒 فحص جودة البروكسي الفردي وتجميعه
async def check_single_proxy(session, semaphore, proxy, chat_id):
    global current_batch_proxies, sent_batches_count, is_hunting_active
    
    if not is_hunting_active or sent_batches_count >= MAX_BATCHES:
        return

    async with semaphore:
        proxy_url = f"http://{proxy}"
        
        # مواقع فحص ذكية وعالية الجودة تضمن استقرار البروكسي
        test_urls = ["https://www.google.com", "https://www.cloudflare.com", "https://www.httpbin.org/ip"]
        target_url = random.choice(test_urls)

        try:
            async with session.get(target_url, proxy=proxy_url, timeout=3.0) as response:
                if response.status == 200:
                    fake_users = ["tickets", "proxyon", "shaga3", "bahr", "premium", "turbo"]
                    user = random.choice(fake_users)
                    password = f"vip{random.randint(1000, 9999)}"
                    formatted_proxy = f"{proxy}:{user}:{password}"
                    
                    if formatted_proxy not in current_batch_proxies:
                        current_batch_proxies.append(formatted_proxy)
                        print(f"✅ تم صيد لايف عالي الجودة: {proxy} | المجموع الحالي: {len(current_batch_proxies)}/20")
                        
                        # أول ما يوصل لـ 20 بروكسي بالضبط يتم إرسالهم فوراً
                        if len(current_batch_proxies) == 20:
                            sent_batches_count += 1
                            raw_output = "\n".join(current_batch_proxies)
                            
                            msg_text = (
                                f"🔥 **GOJO PREMIUM BATCH [{sent_batches_count}/{MAX_BATCHES}]** 👽\n"
                                f"━━━━━━━━━━━━━━━━━━\n"
                                f"✅ **تم صيد 20 بروكسي لايف فائق الجودة بنجاح!**\n"
                                f"━━━━━━━━━━━━━━━━━━\n"
                                f"👇 _اضغط ضغطة واحدة داخل المربع لنسخ الـ 20 فوراً:_\n\n"
                                f"```\n{raw_output}\n```\n"
                                f"━━━━━━━━━━━━━━━━━━\n"
                                f"🔄 جاري الانتقال تلقائياً لفحص الـ 20 القادمين..."
                            )
                            
                            bot.send_message(chat_id, msg_text, parse_mode="Markdown")
                            
                            # تفريغ القائمة فوراً للبدء في فحص الـ 20 التانين كاش ونظيف
                            current_batch_proxies = []
                            
                            # إذا وصلنا لـ 10 مجموعات يتم إيقاف الفحص تلقائياً لحماية الموبايل
                            if sent_batches_count >= MAX_BATCHES:
                                is_hunting_active = False
                                finish_text = (
                                    f"🏁 **اكتملت المهمة بنجاح يا قلب أخوك!** 🪐\n"
                                    f"━━━━━━━━━━━━━━━━━━\n"
                                    f"📊 تم إرسال `{MAX_BATCHES}` مجموعات كاملة (إجمالي 200 بروكسي لايف).\n"
                                    f"🛑 المحرك توقف الآن تلقائياً للحفاظ على موارد الهاتف ونظافة الاتصال."
                                )
                                bot.send_message(chat_id, finish_text, parse_mode="Markdown", reply_markup=main_control_keyboard())
                                print("🏁 انتهت المهمة وتم إرسال الـ 10 مجموعات بنجاح.")
        except:
            pass

@bot.message_handler(commands=['start'])
def send_welcome(message):
    welcome_text = (
        f"👑 **لوحة تحكم غوجو الذكية (إصدار المجموعات اللامتناهية)** 👑\n\n"
        f"المطور الرئيسي: `Bahr` 🛸\n"
        f"━━━━━━━━━━━━━━━━━━\n"
        f"⚙️ **نظام العمل الحالي:**\n"
        f"• عند الصيد، كل **20 بروكسي لايف** هيتبعتوا في رسالة لوحدهم.\n"
        f"• النظام هيفحص لحد **10 مجموعات** (200 بروكسي) ويقف لوحده تلقائياً.\n\n"
        f"👇 استخدم الأزرار تحت للتحكم الفوري والمباشر بالصيد:"
    )
    bot.reply_to(message, welcome_text, parse_mode="Markdown", reply_markup=main_control_keyboard())

# 🚀 تفعيل أمر بدء الفحص من الزر
@bot.callback_query_handler(func=lambda call: call.data == "start_hunt")
def callback_start(call):
    global is_hunting_active, current_batch_proxies, sent_batches_count
    
    if is_hunting_active:
        bot.answer_callback_query(call.id, text="⚠️ الصيد شغال بالفعل في الخلفية يسطا!", show_alert=True)
        return

    is_hunting_active = True
    current_batch_proxies = []
    sent_batches_count = 0  # تصفير عداد المجموعات عند البدء الجديد

    bot.edit_message_text(
        text="🚀 **تم إطلاق المحرك بنجاح!**\n\nجاري سحب وفحص البروكسيات بأعلى جودة وتجميع أول مجموعة (20 بروكسي)... تابع Pydroid والرسائل القادمة.",
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        parse_mode="Markdown",
        reply_markup=main_control_keyboard()
    )

    # تشغيل الحلقة في الخلفية بدون حظر لـ البوت
    loop = asyncio.new_event_loop()
    asyncio.set_event_loop(loop)

    async def infinite_hunting():
        semaphore = asyncio.Semaphore(50) # حد أمان للاتصالات لتجنب الـ 429 وحظر التيليجرام
        
        while is_hunting_active and sent_batches_count < MAX_BATCHES:
            timeout = aiohttp.ClientTimeout(total=5)
            connector = aiohttp.TCPConnector(limit=70, ssl=False)
            
            async with aiohttp.ClientSession(connector=connector, timeout=timeout) as session:
                print(f"\n🔄 [جاري السحب] جاري تجهيز لستة جديدة للبحث عن البروكسيات اللايف...")
                raw_proxies = await fetch_all_raw_proxies()
                
                if raw_proxies and is_hunting_active:
                    random.shuffle(raw_proxies)
                    tasks = []
                    for p in raw_proxies[:1000]: # سقف الفحص في الدورة عشان جودة المعالج
                        if not is_hunting_active or sent_batches_count >= MAX_BATCHES:
                            break
                        tasks.append(check_single_proxy(session, semaphore, p, call.message.chat.id))
                    
                    await asyncio.gather(*tasks)
            
            await asyncio.sleep(3) # راحة قصيرة لتنظيف الذاكرة الكاش للموبايل

    # تشغيل حلقة الصيد
    import threading
    def run_async():
        loop.run_until_complete(infinite_hunting())
    
    threading.Thread(target=run_async).start()

# 🛑 تفعيل أمر إيقاف الفحص من الزر
@bot.callback_query_handler(func=lambda call: call.data == "stop_hunt")
def callback_stop(call):
    global is_hunting_active
    
    if not is_hunting_active:
        bot.answer_callback_query(call.id, text="⚠️ الصيد متوقف بالفعل!", show_alert=True)
        return

    is_hunting_active = False # كسر الحلقة فوراً وإيقاف الفحص
    
    bot.edit_message_text(
        text="🛑 **تم إيقاف الفحص فورا بنجاح!**\n\nالمحرك الآن في وضع السكون الجاهز. يمكنك إعادة التشغيل في أي وقت بالضغط على زر البدء.",
        chat_id=call.message.chat.id,
        message_id=call.message.message_id,
        parse_mode="Markdown",
        reply_markup=main_control_keyboard()
    )
    bot.answer_callback_query(call.id, text="🛑 تم إيقاف المحرك بنجاح", show_alert=False)

bot.infinity_polling()
