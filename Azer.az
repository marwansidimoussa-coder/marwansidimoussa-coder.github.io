import requests
from datetime import datetime, timedelta
import json
import os

USERS_FILE = "users_database.json"

# ==========================================
# 📌 حط هنا عنوان محفظة الدوجي الخاصة بك بين القوسين
# ==========================================
MY_DOGECOIN_WALLET = "YOUR_DOGECOIN_WALLET_ADDRESS_HERE"

def get_doge_price():
    try:
        url = "https://api.coingecko.com/api/v3/simple/price"
        params = {'ids': 'dogecoin', 'vs_currencies': 'usd'}
        response = requests.get(url, params=params)
        if response.status_code == 200:
            return response.json()['dogecoin']['usd']
    except:
        pass
    return 0.20  # سعر تقريبي احتياطي

def load_users():
    if os.path.exists(USERS_FILE):
        with open(USERS_FILE, "r") as f:
            return json.load(f)
    return {}

def save_users(users):
    with open(USERS_FILE, "w") as f:
        json.dump(users, f, indent=4)

def process_doge_deposit(username, doge_amount):
    doge_price = get_doge_price()
    usd_value = doge_amount * doge_price
    
    effective_usd = (usd_value // 5) * 5
    
    if effective_usd < 5:
        return f"❌ الحد الأدنى للإيداع هو ما يعادل 5$ بالدوجي (حوالي {5/doge_price:.2f} DOGE)."
    
    months_to_add = int(effective_usd)
    
    users = load_users()
    now = datetime.utcnow()
    
    if username in users:
        current_expiry = datetime.fromisoformat(users[username]['expiry_date'])
        if current_expiry > now:
            new_expiry = current_expiry + timedelta(days=30 * months_to_add)
        else:
            new_expiry = now + timedelta(days=30 * months_to_add)
    else:
        new_expiry = now + timedelta(days=30 * months_to_add)
        
    users[username] = {
        "doge_paid": doge_amount,
        "usd_equivalent": effective_usd,
        "expiry_date": new_expiry.isoformat(),
        "status": "Active"
    }
    
    save_users(users)
    return f"✅ تم بنجاح! الزبون {username} أودع {doge_amount} DOGE (ما يعادل {effective_usd}$) وتم تفعيل اشتراكه لـ {months_to_add} أشهر."

def fetch_all_crypto():
    url = "https://api.coingecko.com/api/v3/coins/markets"
    params = {
        'vs_currency': 'usd',
        'order': 'market_cap_desc',
        'per_page': 100,
        'page': 1,
        'sparkline': 'false'
    }
    
    try:
        response = requests.get(url, params=params)
        if response.status_code == 200:
            return response.json()
    except Exception as e:
        print(f"Error fetching data: {e}")
    return []

def generate_markdown(data):
    doge_price = get_doge_price()
    min_doge_5usd = 5 / doge_price
    
    content = f"# 🚀 منصة تتبع جميع العملات الرقمية (الدفع بالدوجي 🐕)\n"
    content += f"⏰ آخر تحديث تلقائي: `{datetime.utcnow().strftime('%Y-%m-%d %H:%M:%S')} UTC`\n\n"
    content += f"### 💳 معلومات الإيداع والاشتراك\n"
    content += f"> * **فترة التجربة:** يومين مجاناً 🎁\n"
    content += f"> * **الحد الأدنى للإيداع:** 5$ (ما يعادل تقريباً `{min_doge_5usd:.2f} DOGE`)\n"
    content += f"> * **نظام الأشهر:** كل 5$ تمنح **5 أشهر** (وتتضاعف بـ 5$: 10$ = 10 أشهر، وهكذا).\n"
    content += f"> \n"
    content += f"> 🟡 **أرسل قيمة الدوجي إلى محفظتنا الرسمية التالية:**\n"
    content += f"> ```text\n"
    content += f"> {MY_DOGECOIN_WALLET}\n"
    content += f"> ```\n\n"
    content += f"---\n\n"
    content += f"| العملة | الرمز | الثمن الدقيق (USD) | التغير (24س) |\n"
    content += f"| :--- | :--- | :--- | :--- |\n"
    
    for coin in data:
        name = coin.get('name')
        symbol = coin.get('symbol').upper()
        price = coin.get('current_price')
        change = coin.get('price_change_percentage_24h', 0)
        
        if price is not None:
            if price < 0.0001:
                price_str = f"${price:.8f}"
            elif price < 1:
                price_str = f"${price:.4f}"
            else:
                price_str = f"${price:,.2f}"
        else:
            price_str = "N/A"
            
        change_str = f"{change:.2f}%" if change is not None else "0.00%"
        content += f"| {name} | **{symbol}** | {price_str} | {change_str} |\n"
        
    with open("README.md", "w", encoding="utf-8") as f:
        f.write(content)
    print("تم تحديث الواجهة ومحفظة الدوجي بنجاح!")

if __name__ == "__main__":
    crypto_data = fetch_all_crypto()
    if crypto_data:
        generate_markdown(crypto_data)
