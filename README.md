# Medo

import os
import logging
import requests
import time
import datetime
import random
import threading
from typing import Dict, List, Union
import telebot
from telebot.types import InlineKeyboardMarkup, InlineKeyboardButton, Message, ReplyKeyboardMarkup, KeyboardButton

# Set up logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Initialize bot
API_TOKEN = os.environ.get('TELEGRAM_BOT_TOKEN', '7844651676:AAECNyR45fy1evI8zME1EBE6NunUW1SKUxo')
bot = telebot.TeleBot(API_TOKEN)

# Constants
# قائمة المطورين
DEVELOPERS = ["@me_to77", "@dl_r7c"]

QURAN_SURAHS = [
    "الفاتحة", "البقرة", "آل عمران", "النساء", "المائدة", "الأنعام", "الأعراف", "الأنفال", "التوبة", "يونس",
    "هود", "يوسف", "الرعد", "إبراهيم", "الحجر", "النحل", "الإسراء", "الكهف", "مريم", "طه",
    "الأنبياء", "الحج", "المؤمنون", "النور", "الفرقان", "الشعراء", "النمل", "القصص", "العنكبوت", "الروم",
    "لقمان", "السجدة", "الأحزاب", "سبأ", "فاطر", "يس", "الصافات", "ص", "الزمر", "غافر",
    "فصلت", "الشورى", "الزخرف", "الدخان", "الجاثية", "الأحقاف", "محمد", "الفتح", "الحجرات", "ق",
    "الذاريات", "الطور", "النجم", "القمر", "الرحمن", "الواقعة", "الحديد", "المجادلة", "الحشر", "الممتحنة",
    "الصف", "الجمعة", "المنافقون", "التغابن", "الطلاق", "التحريم", "الملك", "القلم", "الحاقة", "المعارج",
    "نوح", "الجن", "المزمل", "المدثر", "القيامة", "الإنسان", "المرسلات", "النبأ", "النازعات", "عبس",
    "التكوير", "الانفطار", "المطففين", "الانشقاق", "البروج", "الطارق", "الأعلى", "الغاشية", "الفجر", "البلد",
    "الشمس", "الليل", "الضحى", "الشرح", "التين", "العلق", "القدر", "البينة", "الزلزلة", "العاديات",
    "القارعة", "التكاثر", "العصر", "الهمزة", "الفيل", "قريش", "الماعون", "الكوثر", "الكافرون", "النصر",
    "المسد", "الإخلاص", "الفلق", "الناس"
]

RECITERS = [
    "عبد الباسط عبد الصمد", "محمود خليل الحصري", "محمد صديق المنشاوي", "ماهر المعيقلي", "عبد الرحمن السديس",
    "سعد الغامدي", "مشاري راشد العفاسي", "أحمد العجمي", "محمد أيوب", "عبدالله بصفر",
    "ناصر القطامي", "فارس عباد", "ياسر الدوسري", "هاني الرفاعي", "محمود علي البنا",
    "أبو بكر الشاطري", "سعود الشريم", "علي جابر", "خالد القحطاني", "وديع اليمني",
    "إدريس أبكر", "خالد الجليل", "صلاح بدير", "أحمد الحذيفي", "عبدالمحسن القاسم",
    "عادل ريان", "زكي داغستاني", "محمد جبريل", "فارس عباد", "عبدالرشيد صوفي",
    "عبدالله خياط", "محمد عبدالكريم", "بندر بليلة", "ياسين الجزائري", "نبيل الرفاعي",
    "علي حجاج السويسي", "صابر عبدالحكم", "عبدالله المطرود", "محمد الطبلاوي", "إبراهيم الأخضر",
    "يوسف الدغيثر", "ماجد الزامل", "خليفة الطنيجي", "محمود الشيخ", "عبدالودود حنيف",
    "طارق عبدالغني دعوب", "سلمان العتيبي", "رعد الكردي", "عبدالهادي أحمد كناكري", "جمال شاكر",
    # 100 قارئ إضافي
    "أحمد الحواشي", "أحمد خضر الطرابلسي", "أحمد ديبان", "أحمد صابر", "أحمد نعينع",
    "أكرم العلاقمي", "أيمن سويد", "إبراهيم الأخضر", "إبراهيم الجبرين", "إبراهيم الدوسري",
    "توفيق الصايغ", "جمعان العصيمي", "حاتم فريد الواعر", "حسن صالح", "حسين آل الشيخ",
    "حمد الدغريري", "خالد المهنا", "خالد عبد الكافي", "خالد عبدالله القحطاني", "خليل الحصري",
    "خليفة الطنيجي", "رامي الدعيس", "رشيد بلعالية", "زهير حصوي", "سعد الغامدي",
    "سعيد بن صالح", "سعيد دحماني", "سهيل باهبري", "شعبان الصياد", "شيرزاد طاهر",
    "صابر عبدالحكم", "صالح الهبدان", "صلاح البدير", "صلاح الهاشم", "صلاح بو خاطر",
    "طارق عبد الحميد خضر", "عادل الكلباني", "عامر الكاظمي", "عبد الباري الثبيتي", "عبد البارئ محمد",
    "عبد الباسط عبد الصمد المرتل", "عبد الرحمن السديس", "عبد الرحمن العوسي", "عبد الرحمن الماجد", "عبد الرشيد صوفي",
    "عبد العزيز الأحمد", "عبد العزيز الزهراني", "عبد الله الخلف", "عبد الله المطرود", "عبد الله بصفر",
    "عبد الله خياط", "عبد الله عواد الجهني", "عبد المحسن الحارثي", "عبد المحسن العبيكان", "عبد المحسن القاسم",
    "عبد الهادي أحمد كناكري", "عبد الودود حنيف", "علي أبو الحسن", "علي الحذيفي", "علي جابر",
    "علي حجاج السويسي", "عماد زهير حافظ", "عمر القزابري", "فارس عباد", "فواز الكعبي",
    "ماجد الزامل", "ماهر المعيقلي", "مجتبى السادة", "محمد أبو سنينة", "محمد أيوب",
    "محمد الجبريل", "محمد الطبلاوي", "محمد اللحيدان", "محمد المحيسني", "محمد المنشاوي المرتل",
    "محمد حسان", "محمد رشاد الشريف", "محمد سحبل", "محمد شحاتة", "محمد صالح عالم",
    "محمد صديق المنشاوي", "محمد عبد الحكيم", "محمد عبد الكريم", "محمد عبدالله", "محمود الحصري المرتل",
    "محمود الرفاعي", "محمود الشيمي", "محمود الطبلاوي", "محمود خليل الحصري", "محمود علي البنا",
    "مشاري راشد العفاسي", "مصطفى إسماعيل", "مصطفى اللاهوني", "مصطفى رعد العزاوي", "معيض الحارثي",
    "منصور السالمي", "ناصر القطامي", "نبيل الرفاعي", "هاني الرفاعي", "وليد النائحي",
    "ياسر الدوسري", "ياسر القرشي", "ياسر المزروعي", "ياسر سلامة", "ياسين الجزائري"
]

ADHKAR_CATEGORIES = [
    "أذكار الصباح", "أذكار المساء", "أذكار بعد الصلاة", "أذكار النوم", "أذكار الاستيقاظ", 
    "أذكار دخول المنزل", "أذكار الخروج من المنزل", "أذكار الطعام", "أذكار المسجد", "أذكار الوضوء",
    "أذكار السفر", "أذكار اللباس", "أذكار المرض", "أذكار الاستخارة", "أذكار الهم والحزن"
]

# API endpoints for Quran data
QURAN_API_BASE = "https://api.alquran.cloud/v1"
AUDIO_API_BASE = "https://api.qurancdn.com/api/qdc/audio/reciters"
QURAN_CDN_BASE = "https://cdn.islamic.network/quran/audio"

# Cached data
surah_audio_cache = {}
reciter_audio_cache = {}
tafsir_cache = {}
adhkar_cache = {}

# قائمة المستخدمين المشتركين في التنبيهات
daily_subscribers = set()

# رمزيات جميلة للإضافة إلى الرسائل
BEAUTIFUL_SYMBOLS = ["✨", "🌙", "☪️", "📿", "🕋", "🕌", "🤲", "📖", "🌺", "🌹", "💫", "✅", "🌷"]

# قاموس معرفات القراء الرسمية للواجهة البرمجية
RECITER_API_MAPPING = {
    "Abdul_Basit_Mujawwad": ["عبد الباسط عبد الصمد", 0],
    "Abdurrahmaan_As-Sudais_192kbps": ["عبد الرحمن السديس", 4],
    "Abu_Bakr_Ash-Shaatree_128kbps": ["أبو بكر الشاطري", 16],
    "Ahmed_ibn_Ali_al-Ajamy_128kbps": ["أحمد العجمي", 7],
    "Alafasy_128kbps": ["مشاري راشد العفاسي", 6],
    "Hani_Rifai_192kbps": ["هاني الرفاعي", 13],
    "Husary_128kbps": ["محمود خليل الحصري", 1],
    "Maher_AlMuaiqly_64kbps": ["ماهر المعيقلي", 3],
    "Minshawy_Mujawwad_192kbps": ["محمد صديق المنشاوي", 2],
    "Mohammad_al_Tablaway_128kbps": ["محمد الطبلاوي", 38],
    "Saood_ash-Shuraym_128kbps": ["سعود الشريم", 16],
    "Yasser_Ad-Dussary_128kbps": ["ياسر الدوسري", 12]
}

# دالة للحصول على آية يومية عشوائية
def get_daily_verse():
    try:
        # اختيار سورة عشوائية
        surah_number = random.randint(1, 114)
        
        # الحصول على معلومات السورة
        response = requests.get(f"{QURAN_API_BASE}/surah/{surah_number}")
        surah_data = response.json()
        
        if surah_data["code"] == 200:
            # الحصول على عدد الآيات في السورة
            number_of_ayahs = surah_data["data"]["numberOfAyahs"]
            
            # اختيار آية عشوائية
            ayah_number = random.randint(1, number_of_ayahs)
            
            # الحصول على الآية
            ayah_response = requests.get(f"{QURAN_API_BASE}/ayah/{surah_number}:{ayah_number}/ar.asad")
            ayah_data = ayah_response.json()
            
            if ayah_data["code"] == 200:
                surah_name = QURAN_SURAHS[surah_number - 1]
                ayah_text = ayah_data["data"]["text"]
                
                # تنسيق النص
                daily_verse = f"🌟 الآية اليومية 🌟\n\n{ayah_text}\n\n📖 [{surah_name} : {ayah_number}]"
                return daily_verse
    except Exception as e:
        logging.error(f"Error getting daily verse: {e}")
    
    # في حالة حدوث خطأ، ارجع آية افتراضية
    return "﴿وَنُنَزِّلُ مِنَ الْقُرْآنِ مَا هُوَ شِفَاءٌ وَرَحْمَةٌ لِلْمُؤْمِنِينَ﴾ [الإسراء: 82]"

# دالة للحصول على ذكر يومي عشوائي
def get_daily_dhikr():
    try:
        # اختيار فئة عشوائية
        category_idx = random.randint(0, len(ADHKAR_CATEGORIES) - 1)
        category_name = ADHKAR_CATEGORIES[category_idx]
        
        # الحصول على أذكار هذه الفئة
        adhkar = get_adhkar_by_category(category_idx)
        
        # اختيار ذكر عشوائي
        dhikr = random.choice(adhkar)
        
        # تنسيق النص
        daily_dhikr = f"📿 الذكر اليومي 📿\n\nمن {category_name}:\n\n{dhikr['text']}"
        
        if "fadl" in dhikr and dhikr["fadl"]:
            daily_dhikr += f"\n\n✨ {dhikr['fadl']}"
        
        if "repeat" in dhikr and dhikr["repeat"] > 1:
            daily_dhikr += f"\n\n🔄 عدد التكرار: {dhikr['repeat']} مرات"
            
        return daily_dhikr
    except Exception as e:
        logging.error(f"Error getting daily dhikr: {e}")
    
    # في حالة حدوث خطأ، ارجع ذكر افتراضي
    return "📿 الذكر اليومي 📿\n\nسبحان الله وبحمده، سبحان الله العظيم\n\n✨ من قالها مائة مرة حُطت خطاياه وإن كانت مثل زبد البحر"

# Main menu keyboard
def get_main_keyboard():
    markup = InlineKeyboardMarkup()
    markup.row(
        InlineKeyboardButton("القرآن الكريم 📖", callback_data="quran"),
        InlineKeyboardButton("القراء 🎙️", callback_data="reciters")
    )
    markup.row(
        InlineKeyboardButton("الأذكار 📿", callback_data="adhkar"),
        InlineKeyboardButton("التفسير 📚", callback_data="tafsir")
    )
    markup.row(
        InlineKeyboardButton("آية اليوم 🌙", callback_data="daily_verse"),
        InlineKeyboardButton("ذكر اليوم ✨", callback_data="daily_dhikr")
    )
    markup.row(
        InlineKeyboardButton("البحث 🔍", callback_data="search"),
        InlineKeyboardButton("المساعدة ❓", callback_data="help")
    )
    markup.row(
        InlineKeyboardButton("الإشتراك في التنبيهات اليومية 🔔", callback_data="subscribe")
    )
    markup.row(
        InlineKeyboardButton("مطورين البوت 👨‍💻", callback_data="developers")
    )
    return markup

# Create pagination for long lists
def paginated_keyboard(items, callback_prefix, page=0, items_per_page=8):
    markup = InlineKeyboardMarkup()
    start_idx = page * items_per_page
    end_idx = min(start_idx + items_per_page, len(items))
    
    # Add item buttons
    for i in range(start_idx, end_idx):
        markup.add(InlineKeyboardButton(items[i], callback_data=f"{callback_prefix}_{i}"))
    
    # Add navigation buttons
    nav_buttons = []
    if page > 0:
        nav_buttons.append(InlineKeyboardButton("⬅️ السابق", callback_data=f"page_{callback_prefix}_{page-1}"))
    if end_idx < len(items):
        nav_buttons.append(InlineKeyboardButton("التالي ➡️", callback_data=f"page_{callback_prefix}_{page+1}"))
    
    if nav_buttons:
        markup.row(*nav_buttons)
    
    # Add back button
    markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
    return markup

# Start command handler
@bot.message_handler(commands=['start'])
def send_welcome(message):
    # إختيار رمز عشوائي لتجميل الرسالة
    symbol = random.choice(BEAUTIFUL_SYMBOLS)
    
    bot.send_message(
        message.chat.id,
        f"{symbol} بسم الله الرحمن الرحيم {symbol}\n\n"
        "أهلاً بك في بوت القرآن الكريم 🕋\n"
        "💫 يمكنك قراءة القرآن والاستماع إليه بأصوات القراء المفضلين لديك\n"
        "💫 الاطلاع على التفسير وقراءة الأذكار\n"
        "💫 الحصول على آية وذكر يومي\n"
        "💫 الاشتراك في خدمة التنبيهات اليومية\n\n"
        "اختر من القائمة أدناه:",
        reply_markup=get_main_keyboard()
    )

# Help command handler
@bot.message_handler(commands=['help'])
def send_help(message):
    bot.send_message(
        message.chat.id,
        "مرحباً بك في بوت القرآن الكريم 🕋\n\n"
        "الأوامر المتاحة:\n"
        "/start - بدء استخدام البوت\n"
        "/quran - تصفح سور القرآن الكريم\n"
        "/reciters - قائمة القراء\n"
        "/adhkar - الأذكار\n"
        "/tafsir - تفسير القرآن\n"
        "/search - البحث في القرآن\n\n"
        "للاستماع لسورة معينة، اختر السورة ثم اختر القارئ.\n"
        "للاطلاع على التفسير، اختر السورة من قائمة التفسير.",
        reply_markup=get_main_keyboard()
    )

# Main menu command
@bot.message_handler(commands=['menu'])
def show_main_menu(message):
    bot.send_message(
        message.chat.id,
        "القائمة الرئيسية:",
        reply_markup=get_main_keyboard()
    )

# Quran command
@bot.message_handler(commands=['quran'])
def show_quran_list(message):
    bot.send_message(
        message.chat.id,
        "اختر السورة:",
        reply_markup=paginated_keyboard(QURAN_SURAHS, "surah")
    )

# Reciters command
@bot.message_handler(commands=['reciters'])
def show_reciters_list(message):
    bot.send_message(
        message.chat.id,
        "اختر القارئ:",
        reply_markup=paginated_keyboard(RECITERS, "reciter")
    )

# Adhkar command
@bot.message_handler(commands=['adhkar'])
def show_adhkar_categories(message):
    bot.send_message(
        message.chat.id,
        "اختر نوع الذكر:",
        reply_markup=paginated_keyboard(ADHKAR_CATEGORIES, "adhkar_cat")
    )

# Tafsir command
@bot.message_handler(commands=['tafsir'])
def show_tafsir_surahs(message):
    bot.send_message(
        message.chat.id,
        "اختر السورة للتفسير:",
        reply_markup=paginated_keyboard(QURAN_SURAHS, "tafsir_surah")
    )

# Developers command
@bot.message_handler(commands=['developers'])
def show_developers(message):
    devs_text = "مطورين البوت:\n"
    for i, dev in enumerate(DEVELOPERS, 1):
        devs_text += f"{i}. {dev}\n"
    
    markup = InlineKeyboardMarkup()
    markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
    
    bot.send_message(
        message.chat.id,
        devs_text,
        reply_markup=markup
    )

# Daily verse command
@bot.message_handler(commands=['daily_verse'])
def show_daily_verse(message):
    verse = get_daily_verse()
    
    markup = InlineKeyboardMarkup()
    markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
    
    bot.send_message(
        message.chat.id,
        verse,
        reply_markup=markup
    )

# Daily dhikr command
@bot.message_handler(commands=['daily_dhikr'])
def show_daily_dhikr(message):
    dhikr = get_daily_dhikr()
    
    markup = InlineKeyboardMarkup()
    markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
    
    bot.send_message(
        message.chat.id,
        dhikr,
        reply_markup=markup
    )

# Subscribe command
@bot.message_handler(commands=['subscribe'])
def subscribe_to_daily(message):
    chat_id = message.chat.id
    
    if chat_id in daily_subscribers:
        text = "أنت مشترك بالفعل في خدمة التنبيهات اليومية ✅\nلإلغاء الاشتراك، استخدم الأمر /unsubscribe"
    else:
        daily_subscribers.add(chat_id)
        text = "تم اشتراكك بنجاح في خدمة التنبيهات اليومية ✅\nستصلك تنبيهات يومية بآية وذكر من القرآن الكريم"
    
    markup = InlineKeyboardMarkup()
    markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
    
    bot.send_message(
        chat_id,
        text,
        reply_markup=markup
    )

# Unsubscribe command
@bot.message_handler(commands=['unsubscribe'])
def unsubscribe_from_daily(message):
    chat_id = message.chat.id
    
    if chat_id in daily_subscribers:
        daily_subscribers.remove(chat_id)
        text = "تم إلغاء اشتراكك من خدمة التنبيهات اليومية ✅"
    else:
        text = "أنت غير مشترك في خدمة التنبيهات اليومية ❌\nللاشتراك، استخدم الأمر /subscribe"
    
    markup = InlineKeyboardMarkup()
    markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
    
    bot.send_message(
        chat_id,
        text,
        reply_markup=markup
    )

# Search command
@bot.message_handler(commands=['search'])
def search_prompt(message):
    msg = bot.send_message(
        message.chat.id,
        "أدخل كلمة أو جملة للبحث في القرآن الكريم:"
    )
    bot.register_next_step_handler(msg, search_quran)

def search_quran(message):
    query = message.text
    try:
        response = requests.get(f"{QURAN_API_BASE}/search/{query}/all/ar")
        data = response.json()
        
        if data["code"] == 200 and data["data"]["count"] > 0:
            results = data["data"]["matches"][:10]  # Limit to 10 results
            result_text = "نتائج البحث:\n\n"
            
            for i, result in enumerate(results, 1):
                surah_name = QURAN_SURAHS[result["surah"]["number"] - 1]
                result_text += f"{i}. {surah_name} ({result['surah']['number']}:{result['numberInSurah']})\n"
                result_text += f"   {result['text']}\n\n"
                
            bot.send_message(message.chat.id, result_text, reply_markup=get_main_keyboard())
        else:
            bot.send_message(
                message.chat.id,
                "لم يتم العثور على نتائج. حاول استخدام كلمات أخرى.",
                reply_markup=get_main_keyboard()
            )
    except Exception as e:
        logger.error(f"Search error: {e}")
        bot.send_message(
            message.chat.id,
            "حدث خطأ أثناء البحث. يرجى المحاولة مرة أخرى لاحقاً.",
            reply_markup=get_main_keyboard()
        )

# Callback query handler
@bot.callback_query_handler(func=lambda call: True)
def handle_query(call):
    try:
        if call.data == "main_menu":
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text="القائمة الرئيسية:",
                reply_markup=get_main_keyboard()
            )
            
        elif call.data == "quran":
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text="اختر السورة:",
                reply_markup=paginated_keyboard(QURAN_SURAHS, "surah")
            )
            
        elif call.data == "reciters":
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text="اختر القارئ:",
                reply_markup=paginated_keyboard(RECITERS, "reciter")
            )
            
        elif call.data == "adhkar":
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text="اختر نوع الذكر:",
                reply_markup=paginated_keyboard(ADHKAR_CATEGORIES, "adhkar_cat")
            )
            
        elif call.data == "tafsir":
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text="اختر السورة للتفسير:",
                reply_markup=paginated_keyboard(QURAN_SURAHS, "tafsir_surah")
            )
            
        elif call.data == "search":
            bot.answer_callback_query(call.id)
            msg = bot.send_message(
                call.message.chat.id,
                "أدخل كلمة أو جملة للبحث في القرآن الكريم:"
            )
            bot.register_next_step_handler(msg, search_quran)
            
        elif call.data == "help":
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text="مرحباً بك في بوت القرآن الكريم 🕋\n\n"
                    "الأوامر المتاحة:\n"
                    "/start - بدء استخدام البوت\n"
                    "/quran - تصفح سور القرآن الكريم\n"
                    "/reciters - قائمة القراء\n"
                    "/adhkar - الأذكار\n"
                    "/tafsir - تفسير القرآن\n"
                    "/search - البحث في القرآن\n"
                    "/developers - مطورين البوت\n\n"
                    "للاستماع لسورة معينة، اختر السورة ثم اختر القارئ.\n"
                    "للاطلاع على التفسير، اختر السورة من قائمة التفسير ثم اختر نوع التفسير المطلوب:\n"
                    "- التفسير الميسر: تفسير مبسط وميسر للقرآن الكريم\n"
                    "- تفسير ابن كثير: من أشهر التفاسير المعتمدة على الأثر\n"
                    "- تفسير السعدي: تفسير ميسر يجمع بين الأثر والمعنى\n"
                    "- تفسير الطبري: أحد أقدم وأشمل التفاسير\n"
                    "- تفسير القرطبي: تفسير شامل للأحكام الفقهية\n"
                    "- تفسير البغوي: تفسير معتمد على الأحاديث والآثار",
                reply_markup=get_main_keyboard()
            )
            
        elif call.data == "developers":
            devs_text = "مطورين البوت:\n"
            for i, dev in enumerate(DEVELOPERS, 1):
                devs_text += f"{i}. {dev}\n"
                
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=devs_text,
                reply_markup=InlineKeyboardMarkup().add(
                    InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu")
                )
            )
            
        # Handle daily verse request
        elif call.data == "daily_verse":
            verse = get_daily_verse()
            
            markup = InlineKeyboardMarkup()
            markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
            
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=verse,
                reply_markup=markup
            )
            
        # Handle daily dhikr request
        elif call.data == "daily_dhikr":
            dhikr = get_daily_dhikr()
            
            markup = InlineKeyboardMarkup()
            markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
            
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=dhikr,
                reply_markup=markup
            )
            
        # Handle subscription
        elif call.data == "subscribe":
            chat_id = call.message.chat.id
            
            if chat_id in daily_subscribers:
                text = "أنت مشترك بالفعل في خدمة التنبيهات اليومية ✅\nلإلغاء الاشتراك، استخدم الأمر /unsubscribe"
            else:
                daily_subscribers.add(chat_id)
                text = "تم اشتراكك بنجاح في خدمة التنبيهات اليومية ✅\nستصلك تنبيهات يومية بآية وذكر من القرآن الكريم"
            
            markup = InlineKeyboardMarkup()
            markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
            
            bot.edit_message_text(
                chat_id=chat_id,
                message_id=call.message.message_id,
                text=text,
                reply_markup=markup
            )
            
        # Handle pagination
        elif call.data.startswith("page_"):
            _, prefix, page = call.data.split("_")
            page = int(page)
            
            if prefix == "surah":
                text = "اختر السورة:"
                markup = paginated_keyboard(QURAN_SURAHS, "surah", page)
            elif prefix == "reciter":
                text = "اختر القارئ:"
                markup = paginated_keyboard(RECITERS, "reciter", page)
            elif prefix == "adhkar_cat":
                text = "اختر نوع الذكر:"
                markup = paginated_keyboard(ADHKAR_CATEGORIES, "adhkar_cat", page)
            elif prefix == "tafsir_surah":
                text = "اختر السورة للتفسير:"
                markup = paginated_keyboard(QURAN_SURAHS, "tafsir_surah", page)
            else:
                text = "القائمة:"
                markup = get_main_keyboard()
                
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=text,
                reply_markup=markup
            )
            
        # Handle surah selection
        elif call.data.startswith("surah_"):
            surah_idx = int(call.data.split("_")[1])
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            
            # Create keyboard for this surah
            markup = InlineKeyboardMarkup()
            markup.row(
                InlineKeyboardButton("قراءة السورة 📖", callback_data=f"read_surah_{surah_idx}"),
                InlineKeyboardButton("الاستماع للسورة 🎧", callback_data=f"listen_surah_{surah_idx}")
            )
            markup.row(
                InlineKeyboardButton("تفسير السورة 📚", callback_data=f"tafsir_surah_{surah_idx}")
            )
            markup.row(
                InlineKeyboardButton("العودة لقائمة السور 📋", callback_data="quran"),
                InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
            )
            
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=f"سورة {surah_name} - اختر العملية:",
                reply_markup=markup
            )
            
        # Handle reading surah
        elif call.data.startswith("read_surah_"):
            surah_idx = int(call.data.split("_")[2])
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            
            try:
                # Get surah text
                response = requests.get(f"{QURAN_API_BASE}/surah/{surah_number}/ar.asad")
                data = response.json()
                
                if data["code"] == 200:
                    verses = data["data"]["ayahs"]
                    
                    # Special handling for Al-Fatiha and other surahs
                    if surah_number != 1 and surah_number != 9:  # Not Al-Fatiha or At-Tawbah
                        bismillah = "بِسْمِ اللَّهِ الرَّحْمَٰنِ الرَّحِيمِ"
                    else:
                        bismillah = ""
                    
                    # Prepare text chunks (Telegram has message size limits)
                    text_chunks = []
                    current_chunk = f"سورة {surah_name}\n\n{bismillah}\n\n"
                    
                    for verse in verses:
                        verse_text = f"{verse['numberInSurah']}. {verse['text']}\n"
                        
                        # Check if adding this verse would exceed message limit
                        if len(current_chunk) + len(verse_text) > 4000:
                            text_chunks.append(current_chunk)
                            current_chunk = verse_text
                        else:
                            current_chunk += verse_text
                    
                    if current_chunk:
                        text_chunks.append(current_chunk)
                    
                    # Send the first chunk with the keyboard
                    markup = InlineKeyboardMarkup()
                    markup.row(
                        InlineKeyboardButton("العودة للسورة 📋", callback_data=f"surah_{surah_idx}"),
                        InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
                    )
                    
                    bot.edit_message_text(
                        chat_id=call.message.chat.id,
                        message_id=call.message.message_id,
                        text=text_chunks[0],
                        reply_markup=markup
                    )
                    
                    # Send the rest of the chunks
                    for chunk in text_chunks[1:]:
                        bot.send_message(call.message.chat.id, chunk)
                        
                else:
                    bot.answer_callback_query(call.id, "حدث خطأ في جلب النص، يرجى المحاولة مرة أخرى.")
            except Exception as e:
                logger.error(f"Error reading surah: {e}")
                bot.answer_callback_query(call.id, "حدث خطأ، يرجى المحاولة مرة أخرى.")
                
        # Handle listening to surah
        elif call.data.startswith("listen_surah_"):
            surah_idx = int(call.data.split("_")[2])
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            
            # Show reciter selection for this surah
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=f"اختر القارئ لسورة {surah_name}:",
                reply_markup=paginated_keyboard(RECITERS, f"play_surah_{surah_idx}")
            )
            
        # Handle playing surah by specific reciter
        elif call.data.startswith("play_surah_"):
            parts = call.data.split("_")
            surah_idx = int(parts[2])
            reciter_idx = int(parts[3])
            
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            reciter_name = RECITERS[reciter_idx]
            
            bot.answer_callback_query(call.id, f"جاري تحميل سورة {surah_name} بصوت {reciter_name}...")
            
            try:
                # استخدام خريطة القراء المحسنة
                reciter_name = RECITERS[reciter_idx]
                
                # البحث عن المعرف في القاموس المحسن
                reciter_id = None
                for api_id, (name, idx) in RECITER_API_MAPPING.items():
                    if idx == reciter_idx:
                        reciter_id = api_id
                        break
                
                # إذا لم يتم العثور على القارئ، استخدم الافتراضي
                if not reciter_id:
                    reciter_id = "Alafasy_128kbps" if reciter_idx != 0 else "Abdul_Basit_Mujawwad"
                
                # الحصول على عنوان الصوت باستخدام واجهة برمجة التطبيقات المحسنة
                audio_url = f"{QURAN_CDN_BASE}/{reciter_id}/{surah_number:03d}.mp3"
                
                # Send audio
                markup = InlineKeyboardMarkup()
                markup.row(
                    InlineKeyboardButton("العودة للسورة 📋", callback_data=f"surah_{surah_idx}"),
                    InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
                )
                
                bot.send_audio(
                    call.message.chat.id,
                    audio_url,
                    caption=f"سورة {surah_name} - بصوت {reciter_name}",
                    reply_markup=markup
                )
                
                # Edit original message to show successful loading
                bot.edit_message_text(
                    chat_id=call.message.chat.id,
                    message_id=call.message.message_id,
                    text=f"تم تحميل سورة {surah_name} بصوت {reciter_name}",
                    reply_markup=markup
                )
                
            except Exception as e:
                logger.error(f"Error sending audio: {e}")
                bot.edit_message_text(
                    chat_id=call.message.chat.id,
                    message_id=call.message.message_id,
                    text=f"حدث خطأ أثناء تحميل الملف الصوتي. يرجى المحاولة مرة أخرى.",
                    reply_markup=get_main_keyboard()
                )
                
        # Handle reciter selection
        elif call.data.startswith("reciter_"):
            reciter_idx = int(call.data.split("_")[1])
            reciter_name = RECITERS[reciter_idx]
            
            # Create keyboard for surah selection for this reciter
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=f"اختر السورة للاستماع بصوت {reciter_name}:",
                reply_markup=paginated_keyboard(QURAN_SURAHS, f"reciter_surah_{reciter_idx}")
            )
            
        # Handle reciter + surah selection
        elif call.data.startswith("reciter_surah_"):
            parts = call.data.split("_")
            reciter_idx = int(parts[2])
            surah_idx = int(parts[3])
            
            reciter_name = RECITERS[reciter_idx]
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            
            # Same logic as play_surah_ above
            bot.answer_callback_query(call.id, f"جاري تحميل سورة {surah_name} بصوت {reciter_name}...")
            
            try:
                # For simplicity, we'll use a mapping of reciters to their IDs in the API
                reciter_api_ids = {
                    0: "abdul_basit_murattal",  # عبد الباسط عبد الصمد
                    1: "mahmoud_khalil_al-husary",  # محمود خليل الحصري
                    2: "mohammed_siddiq_al-minshawi",  # محمد صديق المنشاوي
                    3: "maher_al-muaiqly",  # ماهر المعيقلي
                    4: "abdul_rahman_al-sudais",  # عبد الرحمن السديس
                    # ... add more mappings for other reciters
                }
                
                # Fallback to a default reciter if not found
                reciter_id = reciter_api_ids.get(reciter_idx, "abdul_basit_murattal")
                
                # Get audio URL - using a public API
                audio_url = f"https://download.quranicaudio.com/quran/{reciter_id}/{surah_number:03d}.mp3"
                
                # Send audio
                markup = InlineKeyboardMarkup()
                markup.row(
                    InlineKeyboardButton("اختيار سورة أخرى 📋", callback_data=f"reciter_{reciter_idx}"),
                    InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
                )
                
                bot.send_audio(
                    call.message.chat.id,
                    audio_url,
                    caption=f"سورة {surah_name} - بصوت {reciter_name}",
                    reply_markup=markup
                )
                
                # Edit original message to show successful loading
                bot.edit_message_text(
                    chat_id=call.message.chat.id,
                    message_id=call.message.message_id,
                    text=f"تم تحميل سورة {surah_name} بصوت {reciter_name}",
                    reply_markup=markup
                )
                
            except Exception as e:
                logger.error(f"Error sending audio: {e}")
                bot.edit_message_text(
                    chat_id=call.message.chat.id,
                    message_id=call.message.message_id,
                    text=f"حدث خطأ أثناء تحميل الملف الصوتي. يرجى المحاولة مرة أخرى.",
                    reply_markup=get_main_keyboard()
                )
                
        # Handle tafsir surah selection
        elif call.data.startswith("tafsir_surah_"):
            surah_idx = int(call.data.split("_")[2])
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            
            # إنشاء لوحة مفاتيح لاختيار نوع التفسير
            markup = InlineKeyboardMarkup()
            markup.row(
                InlineKeyboardButton("تفسير ميسر 📚", callback_data=f"tafsir_muyassar_{surah_idx}"),
                InlineKeyboardButton("تفسير ابن كثير 📚", callback_data=f"tafsir_ibn_kathir_{surah_idx}")
            )
            markup.row(
                InlineKeyboardButton("تفسير السعدي 📚", callback_data=f"tafsir_saadi_{surah_idx}"),
                InlineKeyboardButton("تفسير الطبري 📚", callback_data=f"tafsir_tabari_{surah_idx}")
            )
            markup.row(
                InlineKeyboardButton("تفسير القرطبي 📚", callback_data=f"tafsir_qurtubi_{surah_idx}"),
                InlineKeyboardButton("تفسير البغوي 📚", callback_data=f"tafsir_baghawi_{surah_idx}")
            )
            markup.row(
                InlineKeyboardButton("العودة لقائمة السور 📋", callback_data="tafsir"),
                InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
            )
            
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=f"اختر نوع التفسير لسورة {surah_name}:",
                reply_markup=markup
            )
            
        # Handle different tafsir types
        elif any(call.data.startswith(prefix) for prefix in ["tafsir_muyassar_", "tafsir_ibn_kathir_", "tafsir_saadi_", "tafsir_tabari_", "tafsir_qurtubi_", "tafsir_baghawi_"]):
            parts = call.data.split("_")
            tafsir_type = parts[1]
            surah_idx = int(parts[2])
            surah_name = QURAN_SURAHS[surah_idx]
            surah_number = surah_idx + 1
            
            # تعيين اسم التفسير
            tafsir_names = {
                "muyassar": "التفسير الميسر",
                "ibn_kathir": "تفسير ابن كثير",
                "saadi": "تفسير السعدي",
                "tabari": "تفسير الطبري",
                "qurtubi": "تفسير القرطبي",
                "baghawi": "تفسير البغوي"
            }
            
            tafsir_name = tafsir_names.get(tafsir_type, "التفسير")
            
            try:
                # استخدام التفاسير المحلية المخزنة
                tafsir_text = ""
                if str(surah_number) in TAFSIR_DATA and tafsir_type in TAFSIR_DATA[str(surah_number)]:
                    tafsir_text = TAFSIR_DATA[str(surah_number)][tafsir_type]
                
                if tafsir_text:
                    # تحضير نص التفسير
                    text = f"{tafsir_name} - سورة {surah_name}\n\n{tafsir_text}"
                    
                    # إنشاء لوحة مفاتيح للعودة
                    markup = InlineKeyboardMarkup()
                    markup.row(
                        InlineKeyboardButton("العودة لاختيار نوع التفسير 📚", callback_data=f"tafsir_surah_{surah_idx}"),
                        InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
                    )
                    
                    # إرسال التفسير
                    bot.edit_message_text(
                        chat_id=call.message.chat.id,
                        message_id=call.message.message_id,
                        text=text,
                        reply_markup=markup
                    )
                else:
                    # محاولة الحصول على التفسير من API إذا لم يكن متوفراً محلياً
                    tafsir_api_ids = {
                        "muyassar": 169,     # التفسير الميسر
                        "ibn_kathir": 141,   # تفسير ابن كثير
                        "saadi": 162,        # تفسير السعدي
                        "tabari": 164,       # تفسير الطبري
                        "qurtubi": 151,      # تفسير القرطبي
                        "baghawi": 136       # تفسير البغوي
                    }
                    
                    tafsir_id = tafsir_api_ids.get(tafsir_type, 169)  # استخدم التفسير الميسر كافتراضي
                    
                    # جلب التفسير من API
                    response = requests.get(f"https://api.quran.com/api/v4/quran/tafsirs/{tafsir_id}?chapter_number={surah_number}")
                    data = response.json()
                    
                    if data and "tafsirs" in data:
                        tafsirs = data["tafsirs"]
                        
                        # Prepare text chunks (Telegram has message size limits)
                        text_chunks = []
                        current_chunk = f"{tafsir_name} - سورة {surah_name}\n\n"
                        
                        for tafsir in tafsirs:
                            verse_number = tafsir.get("verse_number", "")
                            text = tafsir.get("text", "")
                            
                            if text:
                                tafsir_text = f"الآية {verse_number}:\n{text}\n\n"
                                
                                # Check if adding this tafsir would exceed message limit
                                if len(current_chunk) + len(tafsir_text) > 4000:
                                    text_chunks.append(current_chunk)
                                    current_chunk = tafsir_text
                                else:
                                    current_chunk += tafsir_text
                        
                        if current_chunk:
                            text_chunks.append(current_chunk)
                        
                        # إنشاء لوحة مفاتيح للعودة
                        markup = InlineKeyboardMarkup()
                        markup.row(
                            InlineKeyboardButton("العودة لاختيار نوع التفسير 📚", callback_data=f"tafsir_surah_{surah_idx}"),
                            InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
                        )
                        
                        # Send the first chunk with the keyboard
                        bot.edit_message_text(
                            chat_id=call.message.chat.id,
                            message_id=call.message.message_id,
                            text=text_chunks[0],
                            reply_markup=markup
                        )
                        
                        # Send the rest of the chunks
                        for chunk in text_chunks[1:]:
                            bot.send_message(call.message.chat.id, chunk)
                    else:
                        fallback_text = f"{tafsir_name} غير متوفر حالياً لسورة {surah_name}. يرجى المحاولة بنوع تفسير آخر."
                        markup = InlineKeyboardMarkup()
                        markup.row(
                            InlineKeyboardButton("العودة لاختيار نوع التفسير 📚", callback_data=f"tafsir_surah_{surah_idx}"),
                            InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
                        )
                        
                        bot.edit_message_text(
                            chat_id=call.message.chat.id,
                            message_id=call.message.message_id,
                            text=fallback_text,
                            reply_markup=markup
                        )
            except Exception as e:
                logger.error(f"Error getting tafsir: {e}")
                bot.edit_message_text(
                    chat_id=call.message.chat.id,
                    message_id=call.message.message_id,
                    text=f"حدث خطأ أثناء جلب التفسير. يرجى المحاولة مرة أخرى.",
                    reply_markup=get_main_keyboard()
                )
                
        # Handle adhkar category selection
        elif call.data.startswith("adhkar_cat_"):
            category_idx = int(call.data.split("_")[2])
            category_name = ADHKAR_CATEGORIES[category_idx]
            
            # In a real implementation, these would be fetched from a database or API
            # For now, we'll use placeholder data
            adhkar = get_adhkar_by_category(category_idx)
            
            # Prepare text
            text = f"أذكار: {category_name}\n\n"
            for i, dhikr in enumerate(adhkar, 1):
                text += f"{i}. {dhikr['text']}\n"
                if "fadl" in dhikr and dhikr["fadl"]:
                    text += f"الفضل: {dhikr['fadl']}\n"
                if "repeat" in dhikr and dhikr["repeat"] > 1:
                    text += f"التكرار: {dhikr['repeat']} مرات\n"
                text += "\n"
                
            # Create keyboard
            markup = InlineKeyboardMarkup()
            markup.row(
                InlineKeyboardButton("العودة لأنواع الأذكار 📋", callback_data="adhkar"),
                InlineKeyboardButton("القائمة الرئيسية 🏠", callback_data="main_menu")
            )
            
            # Send message
            bot.edit_message_text(
                chat_id=call.message.chat.id,
                message_id=call.message.message_id,
                text=text,
                reply_markup=markup
            )
    except Exception as e:
        logger.error(f"Callback error: {e}")
        bot.answer_callback_query(call.id, "حدث خطأ، يرجى المحاولة مرة أخرى.")

# Helper function to get adhkar by category
def get_adhkar_by_category(category_idx):
    # جميع الأذكار من مصادر موثوقة مثل كتب الأذكار المعتمدة
    adhkar_samples = {
        0: [  # أذكار الصباح
            {"text": "أَصْبَحْنَا وَأَصْبَحَ الْمُلْكُ لِلَّهِ، وَالْحَمْدُ لِلَّهِ، لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، لَهُ الْمُلْكُ وَلَهُ الْحَمْدُ وَهُوَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ", "repeat": 1, "fadl": "من قالها حين يصبح أدى شكر يومه"},
            {"text": "اللَّهُمَّ أَنْتَ رَبِّي لاَ إِلَهَ إِلاَّ أَنْتَ، خَلَقْتَنِي وَأَنَا عَبْدُكَ، وَأَنَا عَلَى عَهْدِكَ وَوَعْدِكَ مَا اسْتَطَعْتُ، أَعُوذُ بِكَ مِنْ شَرِّ مَا صَنَعْتُ، أَبُوءُ لَكَ بِنِعْمَتِكَ عَلَيَّ، وَأَبُوءُ بِذَنْبِي فَاغْفِرْ لِي فَإِنَّهُ لاَ يَغْفِرُ الذُّنُوبَ إِلاَّ أَنْتَ", "repeat": 1, "fadl": "من قالها موقنا بها حين يمسي فمات من ليلته دخل الجنة وكذلك حين يصبح"},
            {"text": "اللَّهُمَّ إِنِّي أَسْأَلُكَ الْعَفْوَ وَالْعَافِيَةَ فِي الدُّنْيَا وَالآخِرَةِ، اللَّهُمَّ إِنِّي أَسْأَلُكَ الْعَفْوَ وَالْعَافِيَةَ فِي دِينِي وَدُنْيَايَ وَأَهْلِي وَمَالِي، اللَّهُمَّ اسْتُرْ عَوْرَاتِي، وَآمِنْ رَوْعَاتِي، اللَّهُمَّ احْفَظْنِي مِنْ بَيْنِ يَدَيَّ، وَمِنْ خَلْفِي، وَعَنْ يَمِينِي، وَعَنْ شِمَالِي، وَمِنْ فَوْقِي، وَأَعُوذُ بِعَظَمَتِكَ أَنْ أُغْتَالَ مِنْ تَحْتِي", "repeat": 1, "fadl": "من الأدعية المستحبة في الصباح والمساء"},
            {"text": "اللَّهُمَّ فَاطِرَ السَّمَوَاتِ وَالأَرْضِ، عَالِمَ الغَيْبِ وَالشَّهَادَةِ، رَبَّ كُلِّ شَيْءٍ وَمَلِيكَهُ، أَشْهَدُ أَنْ لاَ إِلَهَ إِلاَّ أَنْتَ، أَعُوذُ بِكَ مِنْ شَرِّ نَفْسِي، وَمِنْ شَرِّ الشَّيْطَانِ وَشِرْكِهِ، وَأَنْ أَقْتَرِفَ عَلَى نَفْسِي سُوءًا، أَوْ أَجُرَّهُ إِلَى مُسْلِمٍ", "repeat": 1, "fadl": "من أذكار الصباح والمساء الواردة في السنة"},
            {"text": "بِسْمِ اللهِ الَّذِي لاَ يَضُرُّ مَعَ اسْمِهِ شَيْءٌ فِي الأَرْضِ وَلاَ فِي السَّمَاءِ وَهُوَ السَّمِيعُ الْعَلِيمُ", "repeat": 3, "fadl": "من قالها ثلاثاً إذا أصبح وثلاثاً إذا أمسى لم يضره شيء"},
            {"text": "رَضِيتُ بِاللهِ رَبًّا، وَبِالإِسْلاَمِ دِينًا، وَبِمُحَمَّدٍ صَلَّى اللهُ عَلَيْهِ وَسَلَّمَ نَبِيًّا", "repeat": 3, "fadl": "من قالها حين يصبح وحين يمسي كان حقا على الله أن يرضيه يوم القيامة"},
            {"text": "اللَّهُمَّ إِنِّي أَصْبَحْتُ أُشْهِدُكَ، وَأُشْهِدُ حَمَلَةَ عَرْشِكَ، وَمَلاَئِكَتَكَ، وَجَمِيعَ خَلْقِكَ، أَنَّكَ أَنْتَ اللهُ لاَ إِلَهَ إِلاَّ أَنْتَ وَحْدَكَ لاَ شَرِيكَ لَكَ، وَأَنَّ مُحَمَّدًا عَبْدُكَ وَرَسُولُكَ", "repeat": 4, "fadl": "من قالها أعتقه الله من النار"},
            {"text": "اللَّهُمَّ مَا أَصْبَحَ بِي مِنْ نِعْمَةٍ أَوْ بِأَحَدٍ مِنْ خَلْقِكَ فَمِنْكَ وَحْدَكَ لاَ شَرِيكَ لَكَ، فَلَكَ الْحَمْدُ وَلَكَ الشُّكْرُ", "repeat": 1, "fadl": "من قالها حين يصبح فقد أدى شكر يومه، ومن قالها حين يمسي فقد أدى شكر ليلته"},
            {"text": "حَسْبِيَ اللهُ لاَ إِلَهَ إِلاَّ هُوَ عَلَيْهِ تَوَكَّلْتُ وَهُوَ رَبُّ الْعَرْشِ الْعَظِيمِ", "repeat": 7, "fadl": "من قالها حين يصبح وحين يمسى سبع مرات كفاه الله ما أهمه من أمر الدنيا والآخرة"},
            {"text": "أَعُوذُ بِكَلِمَاتِ اللهِ التَّامَّاتِ مِنْ شَرِّ مَا خَلَقَ", "repeat": 3, "fadl": "من قالها حين يمسي ثلاث مرات لم تضره حمة (سم) تلك الليلة"},
            {"text": "اللَّهُمَّ عَافِنِي فِي بَدَنِي، اللَّهُمَّ عَافِنِي فِي سَمْعِي، اللَّهُمَّ عَافِنِي فِي بَصَرِي، لاَ إِلَهَ إِلاَّ أَنْتَ", "repeat": 3, "fadl": "من الأدعية المأثورة"}
        ],
        1: [  # أذكار المساء
            {"text": "أَمْسَيْنَا وَأَمْسَى الْمُلْكُ لِلَّهِ، وَالْحَمْدُ لِلَّهِ، لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، لَهُ الْمُلْكُ وَلَهُ الْحَمْدُ وَهُوَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ", "repeat": 1, "fadl": "من قالها حين يمسي أدى شكر ليلته"},
            {"text": "اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ شَرِّ مَا صَنَعْتُ، وَمِنْ شَرِّ مَا لَمْ أَصْنَعْ", "repeat": 1, "fadl": "من أذكار المساء المأثورة"},
            {"text": "اللَّهُمَّ بِكَ أَمْسَيْنَا، وَبِكَ أَصْبَحْنَا، وَبِكَ نَحْيَا، وَبِكَ نَمُوتُ، وَإِلَيْكَ الْمَصِيرُ", "repeat": 1, "fadl": "من أذكار المساء التي كان يقولها النبي صلى الله عليه وسلم"},
            {"text": "اللَّهُمَّ إِنِّي أَمْسَيْتُ أُشْهِدُكَ، وَأُشْهِدُ حَمَلَةَ عَرْشِكَ، وَمَلاَئِكَتَكَ، وَجَمِيعَ خَلْقِكَ، أَنَّكَ أَنْتَ اللهُ لاَ إِلَهَ إِلاَّ أَنْتَ وَحْدَكَ لاَ شَرِيكَ لَكَ، وَأَنَّ مُحَمَّدًا عَبْدُكَ وَرَسُولُكَ", "repeat": 4, "fadl": "من قالها أعتقه الله من النار"},
            {"text": "اللَّهُمَّ مَا أَمْسَى بِي مِنْ نِعْمَةٍ أَوْ بِأَحَدٍ مِنْ خَلْقِكَ فَمِنْكَ وَحْدَكَ لاَ شَرِيكَ لَكَ، فَلَكَ الْحَمْدُ وَلَكَ الشُّكْرُ", "repeat": 1, "fadl": "من قالها حين يمسي فقد أدى شكر ليلته"},
            {"text": "اللَّهُمَّ قِنِي عَذَابَكَ يَوْمَ تَبْعَثُ عِبَادَكَ", "repeat": 3, "fadl": "كان النبي صلى الله عليه وسلم يقولها عند النوم"},
            {"text": "بِاسْمِكَ اللَّهُمَّ أَمُوتُ وَأَحْيَا", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا أراد أن ينام قال ذلك"},
            {"text": "سُبْحَانَ اللهِ وَبِحَمْدِهِ عَدَدَ خَلْقِهِ، وَرِضَا نَفْسِهِ، وَزِنَةَ عَرْشِهِ، وَمِدَادَ كَلِمَاتِهِ", "repeat": 3, "fadl": "من قالها حين يمسي ثلاث مرات غفرت ذنوبه وإن كانت مثل زبد البحر"},
            {"text": "اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنَ الْهَمِّ وَالْحَزَنِ، وَالْعَجْزِ وَالْكَسَلِ، وَالْبُخْلِ وَالْجُبْنِ، وَضَلَعِ الدَّيْنِ، وَغَلَبَةِ الرِّجَالِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يتعوذ بهذه الكلمات"},
            {"text": "أَسْتَغْفِرُ اللهَ الَّذِي لاَ إِلَهَ إِلاَّ هُوَ، الْحَيَّ الْقَيُّومَ، وَأَتُوبُ إِلَيْهِ", "repeat": 3, "fadl": "من قالها غفر له وإن كان قد فر من الزحف"}
        ],
        2: [  # أذكار بعد الصلاة
            {"text": "أَسْتَغْفِرُ اللهَ (ثلاثاً) اللَّهُمَّ أَنْتَ السَّلاَمُ، وَمِنْكَ السَّلاَمُ، تَبَارَكْتَ يَا ذَا الْجَلاَلِ وَالإِكْرَامِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا انصرف من صلاته استغفر ثلاثاً وقال ذلك"},
            {"text": "لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، لَهُ الْمُلْكُ وَلَهُ الْحَمْدُ وَهُوَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ، اللَّهُمَّ لاَ مَانِعَ لِمَا أَعْطَيْتَ، وَلاَ مُعْطِيَ لِمَا مَنَعْتَ، وَلاَ يَنْفَعُ ذَا الْجَدِّ مِنْكَ الْجَدُّ", "repeat": 1, "fadl": "ذكر بعد التسليم من الصلاة"},
            {"text": "لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، لَهُ الْمُلْكُ، وَلَهُ الْحَمْدُ، وَهُوَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ، لاَ حَوْلَ وَلاَ قُوَّةَ إِلاَّ بِاللهِ، لاَ إِلَهَ إِلاَّ اللهُ، وَلاَ نَعْبُدُ إِلاَّ إِيَّاهُ، لَهُ النِّعْمَةُ وَلَهُ الْفَضْلُ وَلَهُ الثَّنَاءُ الْحَسَنُ، لاَ إِلَهَ إِلاَّ اللهُ مُخْلِصِينَ لَهُ الدِّينَ وَلَوْ كَرِهَ الْكَافِرُونَ", "repeat": 1, "fadl": "من المعقبات بعد الصلاة"},
            {"text": "سُبْحَانَ اللهِ", "repeat": 33, "fadl": "من سبح الله دبر كل صلاة ثلاثاً وثلاثين، وحمد الله ثلاثاً وثلاثين، وكبر الله ثلاثاً وثلاثين، وقال تمام المائة: لا إله إلا الله وحده لا شريك له، له الملك، وله الحمد، وهو على كل شيء قدير غفرت خطاياه وإن كانت مثل زبد البحر"},
            {"text": "الْحَمْدُ لِلَّهِ", "repeat": 33, "fadl": "من التسبيح بعد الصلاة"},
            {"text": "اللهُ أَكْبَرُ", "repeat": 33, "fadl": "من التسبيح بعد الصلاة"},
            {"text": "لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، لَهُ الْمُلْكُ وَلَهُ الْحَمْدُ وَهُوَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ", "repeat": 1, "fadl": "تقال بعد التسبيح لتمام المائة"},
            {"text": "قُلْ هُوَ اللهُ أَحَدٌ", "repeat": 1, "fadl": "من قرأ بعد كل صلاة آية الكرسي وقل هو الله أحد والمعوذتين، لم يمنعه من دخول الجنة إلا أن يموت"},
            {"text": "قُلْ أَعُوذُ بِرَبِّ الْفَلَقِ", "repeat": 1, "fadl": "من المعوذات"},
            {"text": "قُلْ أَعُوذُ بِرَبِّ النَّاسِ", "repeat": 1, "fadl": "من المعوذات"},
            {"text": "اللَّهُمَّ إِنِّي أَسْأَلُكَ عِلْمًا نَافِعًا، وَرِزْقًا طَيِّبًا، وَعَمَلًا مُتَقَبَّلًا", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يقوله عقب صلاة الفجر"}
        ],
        3: [  # أذكار النوم
            {"text": "بِاسْمِكَ اللَّهُمَّ أَمُوتُ وَأَحْيَا", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا أراد أن ينام قال ذلك"},
            {"text": "الْحَمْدُ لِلَّهِ الَّذِي أَطْعَمَنَا وَسَقَانَا، وَكَفَانَا، وَآوَانَا، فَكَمْ مِمَّنْ لا كَافِيَ لَهُ وَلا مُؤْوِيَ", "repeat": 1, "fadl": "من أذكار النوم المأثورة"},
            {"text": "اللَّهُمَّ قِنِي عَذَابَكَ يَوْمَ تَبْعَثُ عِبَادَكَ", "repeat": 3, "fadl": "كان النبي صلى الله عليه وسلم يقولها عند النوم"},
            {"text": "اللَّهُمَّ بِاسْمِكَ أَمُوتُ وَأَحْيَا", "repeat": 1, "fadl": "من أذكار النوم"},
            {"text": "سُبْحَانَ اللهِ", "repeat": 33, "fadl": "كان النبي صلى الله عليه وسلم يعلم فاطمة وعلي أن يسبحا ثلاثاً وثلاثين ويحمدا ثلاثاً وثلاثين ويكبرا أربعاً وثلاثين إذا أخذا مضاجعهما"},
            {"text": "الْحَمْدُ لِلَّهِ", "repeat": 33, "fadl": "من التسبيح عند النوم"},
            {"text": "اللهُ أَكْبَرُ", "repeat": 34, "fadl": "من التسبيح عند النوم"},
            {"text": "اللَّهُمَّ أَسْلَمْتُ نَفْسِي إِلَيْكَ، وَفَوَّضْتُ أَمْرِي إِلَيْكَ، وَوَجَّهْتُ وَجْهِي إِلَيْكَ، وَأَلْجَأْتُ ظَهْرِي إِلَيْكَ، رَغْبَةً وَرَهْبَةً إِلَيْكَ، لاَ مَلْجَأَ وَلاَ مَنْجَا مِنْكَ إِلاَّ إِلَيْكَ، آمَنْتُ بِكِتَابِكَ الَّذِي أَنْزَلْتَ، وَبِنَبِيِّكَ الَّذِي أَرْسَلْتَ", "repeat": 1, "fadl": "من قالها ثم مات من ليلته مات على الفطرة"},
            {"text": "اللَّهُمَّ خَلَقْتَ نَفْسِي وَأَنْتَ تَوَفَّاهَا، لَكَ مَمَاتُهَا وَمَحْيَاهَا، إِنْ أَحْيَيْتَهَا فَاحْفَظْهَا، وَإِنْ أَمَتَّهَا فَاغْفِرْ لَهَا، اللَّهُمَّ إِنِّي أَسْأَلُكَ الْعَافِيَةَ", "repeat": 1, "fadl": "من الأدعية عند النوم"},
            {"text": "بِاسْمِكَ رَبِّي وَضَعْتُ جَنْبِي، وَبِكَ أَرْفَعُهُ، فَإِنْ أَمْسَكْتَ نَفْسِي فَارْحَمْهَا، وَإِنْ أَرْسَلْتَهَا فَاحْفَظْهَا، بِمَا تَحْفَظُ بِهِ عِبَادَكَ الصَّالِحِينَ", "repeat": 1, "fadl": "من أذكار النوم المأثورة"}
        ],
        4: [  # أذكار الاستيقاظ
            {"text": "الْحَمْدُ للهِ الَّذِي أَحْيَانَا بَعْدَ مَا أَمَاتَنَا وَإِلَيْهِ النُّشُورُ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا استيقظ من منامه قال ذلك"},
            {"text": "لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، لَهُ الْمُلْكُ وَلَهُ الْحَمْدُ، وَهُوَ عَلَى كُلِّ شَيْءٍ قَدِيرٌ، سُبْحَانَ اللهِ، وَالْحَمْدُ للهِ، وَلاَ إِلَهَ إِلاَّ اللهُ، وَاللهُ أَكْبَرُ، وَلاَ حَوْلَ وَلاَ قُوَّةَ إِلاَّ بِاللهِ الْعَلِيِّ الْعَظِيمِ، رَبِّ اغْفِرْ لِي", "repeat": 1, "fadl": "من قالها غفر له، فإن دعا استجيب له، فإن توضأ وصلى قبلت صلاته"},
            {"text": "الْحَمْدُ لِلَّهِ الَّذِي عَافَانِي فِي جَسَدِي، وَرَدَّ عَلَيَّ رُوحِي، وَأَذِنَ لِي بِذِكْرِهِ", "repeat": 1, "fadl": "من أذكار الاستيقاظ من النوم"},
            {"text": "إِنَّ فِي خَلْقِ السَّمَوَاتِ وَالأَرْضِ وَاخْتِلافِ اللَّيْلِ وَالنَّهَارِ لآيَاتٍ لِأُولِي الألْبَابِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يقرأ العشر آيات من آخر سورة آل عمران إذا استيقظ من الليل"}
        ],
        5: [  # أذكار دخول المنزل
            {"text": "بِسْمِ اللهِ وَلَجْنَا، وَبِسْمِ اللهِ خَرَجْنَا، وَعَلَى رَبِّنَا تَوَكَّلْنَا", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا دخل بيته يقول ذلك"},
            {"text": "اللَّهُمَّ إِنِّي أَسْأَلُكَ خَيْرَ الْمَوْلِجِ وَخَيْرَ الْمَخْرَجِ، بِسْمِ اللهِ وَلَجْنَا، وَبِسْمِ اللهِ خَرَجْنَا، وَعَلَى اللهِ رَبِّنَا تَوَكَّلْنَا", "repeat": 1, "fadl": "من الأذكار المأثورة عند دخول المنزل"},
            {"text": "بِسْمِ اللهِ، اللَّهُمَّ إِنِّي أَسْأَلُكَ خَيْرَ الْمَوْلِجِ وَخَيْرَ الْمَخْرَجِ، بِسْمِ اللهِ وَلَجْنَا، وَبِسْمِ اللهِ خَرَجْنَا، وَعَلَى اللهِ رَبِّنَا تَوَكَّلْنَا، ثُمَّ لِيُسَلِّمْ عَلَى أَهْلِهِ", "repeat": 1, "fadl": "يستحب أن يذكر الله عند دخوله وخروجه"}
        ],
        6: [  # أذكار الخروج من المنزل
            {"text": "بِسْمِ اللهِ، تَوَكَّلْتُ عَلَى اللهِ، وَلاَ حَوْلَ وَلاَ قُوَّةَ إِلاَّ بِاللهِ", "repeat": 1, "fadl": "من قالها يعني إذا خرج من بيته قيل له: كفيت ووقيت وهديت وتنحى عنه الشيطان"},
            {"text": "اللَّهُمَّ إِنِّي أَعُوذُ بِكَ أَنْ أَضِلَّ أَوْ أُضَلَّ، أَوْ أَزِلَّ أَوْ أُزَلَّ، أَوْ أَظْلِمَ أَوْ أُظْلَمَ، أَوْ أَجْهَلَ أَوْ يُجْهَلَ عَلَيَّ", "repeat": 1, "fadl": "من أذكار الخروج من المنزل"},
            {"text": "بِسْمِ اللهِ، آمَنْتُ بِاللهِ، اعْتَصَمْتُ بِاللهِ، تَوَكَّلْتُ عَلَى اللهِ، وَلا حَوْلَ وَلا قُوَّةَ إِلاَّ بِاللهِ", "repeat": 1, "fadl": "من أذكار الخروج من المنزل"}
        ],
        7: [  # أذكار الطعام
            {"text": "بِسْمِ اللهِ", "repeat": 1, "fadl": "إذا أكل أحدكم فليذكر اسم الله تعالى في أوله"},
            {"text": "بِسْمِ اللهِ فِي أَوَّلِهِ وَآخِرِهِ", "repeat": 1, "fadl": "من نسي أن يذكر الله في أول طعامه فليقل حين يذكر"},
            {"text": "اللَّهُمَّ بَارِكْ لَنَا فِيهِ وَأَطْعِمْنَا خَيْرًا مِنْهُ", "repeat": 1, "fadl": "من أذكار الطعام"},
            {"text": "الْحَمْدُ لِلَّهِ الَّذِي أَطْعَمَنِي هَذَا، وَرَزَقَنِيهِ، مِنْ غَيْرِ حَوْلٍ مِنِّي وَلاَ قُوَّةٍ", "repeat": 1, "fadl": "من قالها بعد الأكل غفر له ما تقدم من ذنبه"},
            {"text": "الْحَمْدُ لِلَّهِ حَمْدًا كَثِيرًا طَيِّبًا مُبَارَكًا فِيهِ، غَيْرَ مَكْفِيٍّ وَلاَ مُوَدَّعٍ، وَلاَ مُسْتَغْنًى عَنْهُ رَبَّنَا", "repeat": 1, "fadl": "من أذكار ما بعد الطعام"},
            {"text": "اللَّهُمَّ أَطْعَمْتَ، وَأَسْقَيْتَ، وَأَغْنَيْتَ، وَأَقْنَيْتَ، وَهَدَيْتَ، وَاجْتَبَيْتَ، فَلَكَ الْحَمْدُ عَلَى مَا أَعْطَيْتَ", "repeat": 1, "fadl": "من أذكار بعد الطعام"}
        ],
        8: [  # أذكار المسجد
            {"text": "اللَّهُمَّ افْتَحْ لِي أَبْوَابَ رَحْمَتِكَ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا دخل المسجد قال ذلك"},
            {"text": "أَعُوذُ بِاللهِ الْعَظِيمِ، وَبِوَجْهِهِ الْكَرِيمِ، وَسُلْطَانِهِ الْقَدِيمِ، مِنَ الشَّيْطَانِ الرَّجِيمِ", "repeat": 1, "fadl": "من قالها قال الشيطان: حفظ مني سائر اليوم"},
            {"text": "بِسْمِ اللهِ، وَالصَّلاَةُ وَالسَّلاَمُ عَلَى رَسُولِ اللهِ، اللَّهُمَّ افْتَحْ لِي أَبْوَابَ رَحْمَتِكَ", "repeat": 1, "fadl": "من أذكار دخول المسجد"},
            {"text": "اللَّهُمَّ إِنِّي أَسْأَلُكَ مِنْ فَضْلِكَ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا خرج من المسجد قال ذلك"},
            {"text": "بِسْمِ اللهِ وَالصَّلاةُ وَالسَّلامُ عَلَى رَسُولِ اللهِ، اللَّهُمَّ إِنِّي أَسْأَلُكَ مِنْ فَضْلِكَ، اللَّهُمَّ اعْصِمْنِي مِنَ الشَّيْطَانِ الرَّجِيمِ", "repeat": 1, "fadl": "من أذكار الخروج من المسجد"}
        ],
        9: [  # أذكار الوضوء
            {"text": "بِسْمِ اللهِ", "repeat": 1, "fadl": "من أذكار الوضوء"},
            {"text": "أَشْهَدُ أَنْ لاَ إِلَهَ إِلاَّ اللهُ وَحْدَهُ لاَ شَرِيكَ لَهُ، وَأَشْهَدُ أَنَّ مُحَمَّدًا عَبْدُهُ وَرَسُولُهُ", "repeat": 1, "fadl": "من قالها بعد الوضوء فتحت له أبواب الجنة الثمانية، يدخل من أيها شاء"},
            {"text": "اللَّهُمَّ اجْعَلْنِي مِنَ التَّوَّابِينَ، وَاجْعَلْنِي مِنَ الْمُتَطَهِّرِينَ", "repeat": 1, "fadl": "من أذكار الوضوء"},
            {"text": "سُبْحَانَكَ اللَّهُمَّ وَبِحَمْدِكَ، أَشْهَدُ أَنْ لاَ إِلَهَ إِلاَّ أَنْتَ، أَسْتَغْفِرُكَ وَأَتُوبُ إِلَيْكَ", "repeat": 1, "fadl": "من قالها بعد الوضوء كتبت في رق ثم جعلت في طابع فلم تكسر إلى يوم القيامة"}
        ],
        10: [  # أذكار السفر
            {"text": "اللَّهُ أَكْبَرُ، اللَّهُ أَكْبَرُ، اللَّهُ أَكْبَرُ، سُبْحَانَ الَّذِي سَخَّرَ لَنَا هَذَا وَمَا كُنَّا لَهُ مُقْرِنِينَ، وَإِنَّا إِلَى رَبِّنَا لَمُنْقَلِبُونَ، اللَّهُمَّ إِنَّا نَسْأَلُكَ فِي سَفَرِنَا هَذَا الْبِرَّ وَالتَّقْوَى، وَمِنَ الْعَمَلِ مَا تَرْضَى، اللَّهُمَّ هَوِّنْ عَلَيْنَا سَفَرَنَا هَذَا وَاطْوِ عَنَّا بُعْدَهُ، اللَّهُمَّ أَنْتَ الصَّاحِبُ فِي السَّفَرِ، وَالْخَلِيفَةُ فِي الأَهْلِ، اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ وَعْثَاءِ السَّفَرِ، وَكَآبَةِ الْمَنْظَرِ، وَسُوءِ الْمُنْقَلَبِ فِي الْمَالِ وَالأَهْلِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا سافر قال ذلك"},
            {"text": "آيِبُونَ، تَائِبُونَ، عَابِدُونَ، لِرَبِّنَا حَامِدُونَ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم إذا قفل من سفر قال ذلك"},
            {"text": "اللَّهُمَّ إِنِّي أَعُوذُ بِكَ مِنْ وَعْثَاءِ السَّفَرِ، وَكَآبَةِ الْمُنْقَلَبِ، وَالْحَوْرِ بَعْدَ الْكَوْرِ، وَدَعْوَةِ الْمَظْلُومِ، وَسُوءِ الْمَنْظَرِ فِي الأَهْلِ وَالْمَالِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يتعوذ من هذه الكلمات"}
        ],
        11: [  # أذكار اللباس
            {"text": "الْحَمْدُ لِلَّهِ الَّذِي كَسَانِي هَذَا (الثَّوْبَ) وَرَزَقَنِيهِ مِنْ غَيْرِ حَوْلٍ مِنِّي وَلاَ قُوَّةٍ", "repeat": 1, "fadl": "من قالها غفر له ما تقدم من ذنبه وما تأخر"},
            {"text": "بِسْمِ اللهِ", "repeat": 1, "fadl": "من أذكار لبس الثوب الجديد"},
            {"text": "اللَّهُمَّ لَكَ الْحَمْدُ أَنْتَ كَسَوْتَنِيهِ، أَسْأَلُكَ مِنْ خَيْرِهِ وَخَيْرِ مَا صُنِعَ لَهُ، وَأَعُوذُ بِكَ مِنْ شَرِّهِ وَشَرِّ مَا صُنِعَ لَهُ", "repeat": 1, "fadl": "من أذكار لبس الثوب الجديد"}
        ],
        12: [  # أذكار المرض
            {"text": "اللَّهُمَّ رَبَّ النَّاسِ، أَذْهِبِ الْبَاسَ، اشْفِ أَنْتَ الشَّافِي، لاَ شِفَاءَ إِلاَّ شِفَاؤُكَ، شِفَاءً لاَ يُغَادِرُ سَقَمًا", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يمسح بيده اليمنى على المريض ويقول ذلك"},
            {"text": "اللَّهُمَّ اشْفِهِ اللَّهُمَّ اشْفِهِ", "repeat": 7, "fadl": "من قالها سبع مرات عند مريض لم يحضر أجله عوفي"},
            {"text": "أَسْأَلُ اللهَ الْعَظِيمَ، رَبَّ الْعَرْشِ الْعَظِيمِ، أَنْ يَشْفِيَكَ", "repeat": 7, "fadl": "من قالها للمريض سبع مرات عافاه الله من ذلك المرض"},
            {"text": "بِسْمِ اللهِ أَرْقِيكَ، مِنْ كُلِّ شَيْءٍ يُؤْذِيكَ، مِنْ شَرِّ كُلِّ نَفْسٍ أَوْ عَيْنِ حَاسِدٍ، اللهُ يَشْفِيكَ، بِسْمِ اللهِ أَرْقِيكَ", "repeat": 3, "fadl": "من الرقية الشرعية الثابتة"}
        ],
        13: [  # أذكار الاستخارة
            {"text": "اللَّهُمَّ إِنِّي أَسْتَخِيرُكَ بِعِلْمِكَ، وَأَسْتَقْدِرُكَ بِقُدْرَتِكَ، وَأَسْأَلُكَ مِنْ فَضْلِكَ الْعَظِيمِ، فَإِنَّكَ تَقْدِرُ وَلاَ أَقْدِرُ، وَتَعْلَمُ وَلاَ أَعْلَمُ، وَأَنْتَ عَلاَّمُ الْغُيُوبِ، اللَّهُمَّ إِنْ كُنْتَ تَعْلَمُ أَنَّ هَذَا الأَمْرَ (هنا تسمي حاجتك) خَيْرٌ لِي فِي دِينِي وَمَعَاشِي وَعَاقِبَةِ أَمْرِي، فَاقْدُرْهُ لِي وَيَسِّرْهُ لِي، ثُمَّ بَارِكْ لِي فِيهِ، وَإِنْ كُنْتَ تَعْلَمُ أَنَّ هَذَا الأَمْرَ شَرٌّ لِي فِي دِينِي وَمَعَاشِي وَعَاقِبَةِ أَمْرِي، فَاصْرِفْهُ عَنِّي وَاصْرِفْنِي عَنْهُ، وَاقْدُرْ لِي الْخَيْرَ حَيْثُ كَانَ، ثُمَّ أَرْضِنِي بِهِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يعلم أصحابه الاستخارة في الأمور كلها كما يعلمهم السورة من القرآن"}
        ],
        14: [  # أذكار الهم والحزن
            {"text": "اللَّهُمَّ إِنِّي عَبْدُكَ، ابْنُ عَبْدِكَ، ابْنُ أَمَتِكَ، نَاصِيَتِي بِيَدِكَ، مَاضٍ فِيَّ حُكْمُكَ، عَدْلٌ فِيَّ قَضَاؤُكَ، أَسْأَلُكَ بِكُلِّ اسْمٍ هُوَ لَكَ، سَمَّيْتَ بِهِ نَفْسَكَ، أَوْ أَنْزَلْتَهُ فِي كِتَابِكَ، أَوْ عَلَّمْتَهُ أَحَدًا مِنْ خَلْقِكَ، أَوِ اسْتَأْثَرْتَ بِهِ فِي عِلْمِ الْغَيْبِ عِنْدَكَ، أَنْ تَجْعَلَ الْقُرْآنَ رَبِيعَ قَلْبِي، وَنُورَ صَدْرِي، وَجَلاَءَ حُزْنِي، وَذَهَابَ هَمِّي", "repeat": 1, "fadl": "ما قاله أحد مهموم إلا فرج الله همه وأبدله مكان حزنه فرحاً"},
            {"text": "لاَ إِلَهَ إِلاَّ اللهُ الْعَظِيمُ الْحَلِيمُ، لاَ إِلَهَ إِلاَّ اللهُ رَبُّ الْعَرْشِ الْعَظِيمِ، لاَ إِلَهَ إِلاَّ اللهُ رَبُّ السَّمَوَاتِ وَرَبُّ الأَرْضِ وَرَبُّ الْعَرْشِ الْكَرِيمِ", "repeat": 1, "fadl": "كان النبي صلى الله عليه وسلم يقولها عند الكرب"},
            {"text": "اللَّهُمَّ رَحْمَتَكَ أَرْجُو، فَلاَ تَكِلْنِي إِلَى نَفْسِي طَرْفَةَ عَيْنٍ، وَأَصْلِحْ لِي شَأْنِي كُلَّهُ، لاَ إِلَهَ إِلاَّ أَنْتَ", "repeat": 1, "fadl": "دعاء الكرب"},
            {"text": "لاَ إِلَهَ إِلاَّ أَنْتَ سُبْحَانَكَ إِنِّي كُنْتُ مِنَ الظَّالِمِينَ", "repeat": 1, "fadl": "دعاء ذي النون، ما دعا به مكروب إلا فرج الله عنه"},
            {"text": "اللهُ اللهُ رَبِّي لاَ أُشْرِكُ بِهِ شَيْئًا", "repeat": 7, "fadl": "كان النبي صلى الله عليه وسلم يقولها عند الفزع"}
        ]
    }
    
    # Return default if category not found
    return adhkar_samples.get(category_idx, [{"text": "الذكر غير متوفر حاليًا", "repeat": 1}])

# استيراد بيانات التفسير
from tafsir_data import TAFSIR_DATA

# تعريف قائمة الأوامر للقائمة الجانبية
def set_bot_commands():
    commands = [
        telebot.types.BotCommand("/start", "بدء استخدام البوت"),
        telebot.types.BotCommand("/menu", "القائمة الرئيسية"),
        telebot.types.BotCommand("/quran", "القرآن الكريم"),
        telebot.types.BotCommand("/reciters", "القراء"),
        telebot.types.BotCommand("/adhkar", "الأذكار"),
        telebot.types.BotCommand("/tafsir", "التفسير"),
        telebot.types.BotCommand("/search", "البحث في القرآن"),
        telebot.types.BotCommand("/developers", "مطورين البوت"),
        telebot.types.BotCommand("/help", "المساعدة")
    ]
    bot.set_my_commands(commands)
    logger.info("Bot commands menu set successfully")

# وظيفة إرسال التنبيهات اليومية
def send_daily_notifications():
    while True:
        try:
            # الحصول على الوقت الحالي
            now = datetime.datetime.now()
            
            # تحقق مما إذا كان الوقت هو 5 صباحًا
            if now.hour == 5 and now.minute == 0:
                # الحصول على الآية والذكر اليومي
                verse = get_daily_verse()
                dhikr = get_daily_dhikr()
                
                # إنشاء لوحة مفاتيح للعودة
                markup = InlineKeyboardMarkup()
                markup.row(InlineKeyboardButton("العودة للقائمة الرئيسية 🏠", callback_data="main_menu"))
                
                # إرسال التنبيهات لجميع المشتركين
                for chat_id in daily_subscribers:
                    try:
                        # إرسال الآية اليومية
                        bot.send_message(chat_id, verse, reply_markup=markup)
                        # انتظار قليلاً لتجنب قيود الإرسال
                        time.sleep(1)
                        
                        # إرسال الذكر اليومي
                        bot.send_message(chat_id, dhikr, reply_markup=markup)
                    except Exception as send_error:
                        logger.error(f"Error sending notification to {chat_id}: {send_error}")
                
                # انتظار لمدة 55 دقيقة لتجنب الإرسال المتكرر
                time.sleep(55 * 60)
            
            # انتظار دقيقة واحدة قبل التحقق مرة أخرى
            time.sleep(60)
        except Exception as e:
            logger.error(f"Error in daily notification thread: {e}")
            # انتظار قبل المحاولة مرة أخرى
            time.sleep(60)

if __name__ == "__main__":
    logger.info("Starting the Quran bot...")
    logger.info(f"Bot username: {bot.get_me().username}")
    logger.info(f"Bot is running with token: {API_TOKEN[:5]}...{API_TOKEN[-5:]}")
    
    # إعداد قائمة الأوامر الجانبية
    set_bot_commands()
    
    # بدء خيط التنبيهات اليومية
    notification_thread = threading.Thread(target=send_daily_notifications)
    notification_thread.daemon = True
    notification_thread.start()
    logger.info("Daily notification thread started")
    
    # استمرار تشغيل البوت حتى مع وجود أخطاء
    while True:
        try:
            bot.polling(none_stop=True, interval=0, timeout=20)
        except Exception as e:
            logger.error(f"Bot polling error: {e}")
            time.sleep(5)
