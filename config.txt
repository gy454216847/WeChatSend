import http.client
import json
import os
import random
import sys
import urllib
from datetime import date, datetime
from time import localtime, time

from requests import get, post
from zhdate import ZhDate

import cityinfo


def get_color():
    # 获取随机颜色
    def get_colors(n): return list(map(lambda i: "#" + "%06x" %
                                       random.randint(0, 0xFFFFFF), range(n)))
    color_list = get_colors(100)
    return random.choice(color_list)


def get_access_token():
    # appId
    app_id = config["app_id"]
    # appSecret
    app_secret = config["app_secret"]
    post_url = ("https://api.weixin.qq.com/cgi-bin/token?grant_type=client_credential&appid={}&secret={}"
                .format(app_id, app_secret))
    try:
        access_token = get(post_url).json()['access_token']
    except KeyError:
        print("获取access_token失败，请检查app_id和app_secret是否正确")
        os.system("pause")

        sys.exit(1)
    return access_token


def get_birthday(birthday, year, today):
    birthday_year = birthday.split("-")[0]
    # 判断是否为农历生日
    if birthday_year[0] == "r":
        r_mouth = int(birthday.split("-")[1])
        r_day = int(birthday.split("-")[2])
        # 今年生日
        birthday = ZhDate(year, r_mouth, r_day).to_datetime().date()
        year_date = birthday

    else:
        # 获取国历生日的今年对应月和日
        birthday_month = int(birthday.split("-")[1])
        birthday_day = int(birthday.split("-")[2])
        # 今年生日
        year_date = date(year, birthday_month, birthday_day)
    # 计算生日年份，如果还没过，按当年减，如果过了需要+1
    if today > year_date:
        if birthday_year[0] == "r":
            # 获取农历明年生日的月和日
            r_last_birthday = ZhDate(
                (year + 1), r_mouth, r_day).to_datetime().date()
            birth_date = date(
                (year + 1), r_last_birthday.month, r_last_birthday.day)
        else:
            birth_date = date((year + 1), birthday_month, birthday_day)
        birth_day = str(birth_date.__sub__(today)).split(" ")[0]
    elif today == year_date:
        birth_day = 0
    else:
        birth_date = year_date
        birth_day = str(birth_date.__sub__(today)).split(" ")[0]
    return birth_day


def get_anniversary(anniversaryday, year, today):
    # 获取纪念日年份
    anniversary_year = anniversaryday.split("-")[0]
    # 获取纪念日的今年对应月和日
    anniversary_month = int(anniversaryday.split("-")[1])
    anniversary_day = int(anniversaryday.split("-")[2])
    # 今年纪念日
    year_date = date(year, anniversary_month, anniversary_day)

    if today > year_date:
        anniversary_date = date((year+1), anniversary_month, anniversary_day)
        anniversary_day = str(anniversary_date.__sub__(today)).split(" ")[0]
    elif today == year_date:
        anniversary_day=0
    else:
        anniversary_date=year_date
        anniversary_day = str(anniversary_date.__sub__(today)).split(" ")[0]

    return anniversary_day


def get_weather(province, city):
    # 城市id
    try:
        city_id = cityinfo.cityInfo[province][city]["AREAID"]
    except KeyError:
        print("推送消息失败，请检查省份或城市是否正确")
        os.system("pause")
        sys.exit(1)
    # city_id = 101280101
    # 毫秒级时间戳
    t = (int(round(time() * 1000)))
    headers = {
        "Referer": "http://www.weather.com.cn/weather1d/{}.shtml".format(city_id),
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
                      'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'
    }
    url = "http://d1.weather.com.cn/dingzhi/{}.html?_={}".format(city_id, t)
    response = get(url, headers=headers)
    response.encoding = "utf-8"
    response_data = response.text.split(";")[0].split("=")[-1]
    response_json = eval(response_data)
    # print(response_json)
    weatherinfo = response_json["weatherinfo"]
    # 天气
    _weather = str(weatherinfo["weather"])
    if (_weather == "晴"):
        weather = "☀️"+_weather
    elif (_weather == "阴"):
        weather = "☁️"+_weather
    elif (_weather.find("云") != -1):
        weather = "☁️"+_weather
    elif (_weather.find("雨") != -1):
        weather = "⛈️"+_weather
    elif (_weather.find("雪") != -1):
        weather = "❄️"+_weather
    elif (_weather.find("雾") != -1 or _weather.find("️霾") != -1):
        weather = "🌫️"+_weather
        # 最高气温
    temp = weatherinfo["temp"]
    # 最低气温
    tempn = weatherinfo["tempn"]
    return weather, temp, tempn


# 词霸每日一句
def get_ciba():
    if (Whether_Eng != "否"):
        url = "http://open.iciba.com/dsapi/"
        headers = {
            'Content-Type': 'application/json',
            'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
            'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'
        }
        r = get(url, headers=headers)
        note_en = r.json()["content"]
        note_ch = r.json()["note"]
        return note_ch, note_en
    else:
        return "", ""

# 表情


def get_emoticon():
    emoticon_list = ["(￣▽￣)~*", "(～￣▽￣)～", "︿(￣︶￣)︿", "~(￣▽￣)~*", "(oﾟ▽ﾟ)o", "ヾ(✿ﾟ▽ﾟ)ノ", "٩(๑❛ᴗ❛๑)۶", "ヾ(◍°∇°◍)ﾉﾞ",
                     "ヾ(๑╹◡╹)ﾉ", "(๑´ㅂ`๑)", "(*´ﾟ∀ﾟ｀)ﾉ", "(´▽`)ﾉ", "ヾ(●´∀｀●)",
                     "(｡◕ˇ∀ˇ◕)", "(≖ᴗ≖)✧", "(◕ᴗ◕✿)", "(❁´◡`❁)*✲ﾟ*", "(๑¯∀¯๑)", "(*´・ｖ・)", "(づ｡◕ᴗᴗ◕｡)づ", "o(*￣▽￣*)o",
                     "(｀・ω・´)", "( • ̀ω•́ )✧", "ヾ(=･ω･=)o", "(￣３￣)a", "(灬°ω°灬)", "ヾ(•ω•`。)", "｡◕ᴗ◕｡"]
    return random.choice(emoticon_list)

# 彩虹屁


def caihongpi():
    if (caihongpi_API == "dfe766fa7f08ae4100cffd485f121ff6"):
        conn = http.client.HTTPSConnection('api.tianapi.com')  # 接口域名
        params = urllib.parse.urlencode({'key': caihongpi_API})
        headers = {'Content-type': 'application/x-www-form-urlencoded'}
        conn.request('POST', '/saylove/index', params, headers)
        res = conn.getresponse()
        data = res.read()
        data = json.loads(data)
        data = data["newslist"][0]["content"]
        if ("XXX" in data):
            data.replace("XXX", "小可爱")
        return data
    else:
        return ""

# 健康小提示API


def health():
    if (health_API == "dfe766fa7f08ae4100cffd485f121ff6"):
        conn = http.client.HTTPSConnection('api.tianapi.com')  # 接口域名
        params = urllib.parse.urlencode({'key': health_API})
        headers = {'Content-type': 'application/x-www-form-urlencoded'}
        conn.request('POST', '/healthtip/index', params, headers)
        res = conn.getresponse()
        data = res.read()
        data = json.loads(data)
        data = data["newslist"][0]["content"]
        return data
    else:
        return ""

# 星座运势


def lucky():
    if (lucky_API == "dfe766fa7f08ae4100cffd485f121ff6"):
        conn = http.client.HTTPSConnection('api.tianapi.com')  # 接口域名
        params = urllib.parse.urlencode({'key': lucky_API, 'astro': astro})
        headers = {'Content-type': 'application/x-www-form-urlencoded'}
        conn.request('POST', '/star/index', params, headers)
        res = conn.getresponse()
        data = res.read()
        data = json.loads(data)
        data = str(data["newslist"][0]["type"])+"："+str(data["newslist"][0]["content"])+"\n"+str(data["newslist"][1]["type"])+"："+str(data["newslist"][1]["content"]) + \
            "\n"+str(data["newslist"][2]["type"])+"："+str(data["newslist"][2]["content"]) + \
            "\n"+str(data["newslist"][7]["type"])+"：" + \
            str(data["newslist"][7]["content"])
        return data
    else:
        return ""

# 励志名言


def lizhi():
    if (lizhi_API != "替换掉我"):
        conn = http.client.HTTPSConnection('api.tianapi.com')  # 接口域名
        params = urllib.parse.urlencode({'key': lizhi_API})
        headers = {'Content-type': 'application/x-www-form-urlencoded'}
        conn.request('POST', '/lzmy/index', params, headers)
        res = conn.getresponse()
        data = res.read()
        data = json.loads(data)
        return data["newslist"][0]["saying"]
    else:
        return ""


# 下雨概率和建议
def tip():
    if (tianqi_API == "dfe766fa7f08ae4100cffd485f121ff6"):
        conn = http.client.HTTPSConnection('api.tianapi.com')  # 接口域名
        params = urllib.parse.urlencode({'key': tianqi_API, 'city': city})
        headers = {'Content-type': 'application/x-www-form-urlencoded'}
        # _city=urllib.parse.quote(city)
        url = "/tianqi/index"
        conn.request('POST', url, params, headers=headers)
        res = conn.getresponse()
        data = res.read()
        data = json.loads(data)
        pop = data["newslist"][0]["pcpn"]
        tips = data["newslist"][0]["tips"]
        wind = data["newslist"][0]["windsc"]
        humidity = data["newslist"][0]["humidity"]
        return pop, tips, wind, humidity
    else:
        return "", ""

# 推送信息


def send_message(to_user, access_token, city_name, weather, max_temperature, min_temperature, pipi, lizhi, pop, tips, wind, humidity, note_en, note_ch, health_tip, lucky):
    url = "https://api.weixin.qq.com/cgi-bin/message/template/send?access_token={}".format(
        access_token)
    week_list = ["星期日", "星期一", "星期二", "星期三", "星期四", "星期五", "星期六"]
    year = localtime().tm_year
    month = localtime().tm_mon
    day = localtime().tm_mday
    today = datetime.date(datetime(year=year, month=month, day=day))
    week = week_list[today.isoweekday() % 7]
    # 获取在一起的日子的日期格式
    love_year = int(config["love_date"].split("-")[0])
    love_month = int(config["love_date"].split("-")[1])
    love_day = int(config["love_date"].split("-")[2])
    love_date = date(love_year, love_month, love_day)
    # 获取在一起的日期差
    love_days = str(today.__sub__(love_date)).split(" ")[0]
    #获取结婚纪念日
    anniversaryday=get_anniversary(config["anniversary_date"],year,today)


    # 获取所有生日数据
    birthdays = {}
    for k, v in config.items():
        if k[0:5] == "birth":
            birthdays[k] = v
    data = {
        "touser": to_user,
        "template_id": config["template_id"],
        "url": "http://weixin.qq.com/download",
        "topcolor": "#FF0000",
        "data": {
            "date": {
                "value": "{} {}".format(today, week),
                "color": get_color()
            },
            "city": {
                "value": city_name,
                "color": get_color()
            },
            "weather": {
                "value": weather,
                "color": get_color()
            },
            "min_temperature": {
                "value": min_temperature,
                "color": get_color()
            },
            "max_temperature": {
                "value": max_temperature,
                "color": get_color()
            },
            "love_day": {
                "value": love_days,
                "color": get_color()
            },
            "note_en": {
                "value": note_en,
                "color": get_color()
            },
            "note_ch": {
                "value": note_ch,
                "color": get_color()
            },

            "pipi": {
                "value": pipi,
                "color": get_color()
            },

            "lucky": {
                "value": lucky,
                "color": get_color()
            },

            "lizhi": {
                "value": lizhi,
                "color": get_color()
            },

            "pop": {
                "value": pop,
                "color": get_color()
            },

            "health": {
                "value": health_tip,
                "color": get_color()
            },

            "tips": {
                "value": tips,
                "color": get_color()
            },
            "anniversary": {
                "value": anniversaryday,
                "color": get_color()
            },
            "emotion": {
                "value": emotion,
                "color": get_color()
            },
            "wind": {
                "value": wind,
                "color": get_color()
            },
            "humidity": {
                "value": humidity,
                "color": get_color()
            },
        }
    }
    for key, value in birthdays.items():
        # 获取距离下次生日的时间
        birth_day = get_birthday(value, year, today)
        # 将生日数据插入data
        data["data"][key] = {"value": birth_day, "color": get_color()}
    headers = {
        'Content-Type': 'application/json',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) '
                      'AppleWebKit/537.36 (KHTML, like Gecko) Chrome/103.0.0.0 Safari/537.36'
    }
    response = post(url, headers=headers, json=data).json()
    if response["errcode"] == 40037:
        print("推送消息失败，请检查模板id是否正确")
    elif response["errcode"] == 40036:
        print("推送消息失败，请检查模板id是否为空")
    elif response["errcode"] == 40003:
        print("推送消息失败，请检查微信号是否正确")
    elif response["errcode"] == 0:
        print("推送消息成功")
    else:
        print(response)


if __name__ == "__main__":
    try:
        with open(file="config.txt", encoding="utf-8") as f:
             config = eval(f.read())
    except FileNotFoundError:
        print("推送消息失败，请检查config.txt文件是否与程序位于同一路径")
        os.system("pause")
        sys.exit(1) 
    except SyntaxError:
        print("推送消息失败，请检查配置文件格式是否正确")
        os.system("pause")
        sys.exit(1)

    # 获取accessToken
    accessToken = get_access_token()
    # 接收的用户
    users = config["user"]
    # 传入省份和市获取天气信息
    province, city = config["province"], config["city"]
    weather, max_temperature, min_temperature = get_weather(province, city)
    # 获取彩虹屁API
    caihongpi_API = config["caihongpi_API"]
    # 获取励志古言API
    lizhi_API = config["lizhi_API"]
    # 获取天气预报API
    tianqi_API = config["tianqi_API"]
    # 是否启用词霸每日金句
    Whether_Eng = config["Whether_Eng"]
    # 获取健康小提示API
    health_API = config["health_API"]
    # 获取星座运势API
    lucky_API = config["lucky_API"]
    # 获取星座
    astro = config["astro"]
    # 获取词霸每日金句
    note_ch, note_en = get_ciba()
    # 彩虹屁
    pipi = caihongpi()
    # 健康小提示
    health_tip = health()
    # 下雨概率,建议,风力，空气质量，湿度
    pop, tips, wind, humidity = tip()
    # 励志名言
    lizhi = lizhi()
    # 星座运势
    lucky = lucky()
    # 表情
    emotion = get_emoticon()
    # 公众号推送消息
    for user in users:
        send_message(user, accessToken, city, weather, max_temperature, min_temperature,
                     pipi, lizhi, pop, tips, wind, humidity, note_en, note_ch, health_tip, lucky)
    import time
    time_duration = 3.5
    time.sleep(time_duration)
