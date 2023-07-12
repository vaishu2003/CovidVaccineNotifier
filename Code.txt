import requests
from pygame import mixer
from datetime import datetime, timedelta
import time
import pytz
IST = pytz.timezone('Asia/Kolkata')
raw_TS = datetime.now(IST)

age = 52
pincodes = ["502319"]
num_days = 2
tele_auth_token = "5801047052:AAHNgJrkS4D9oYeqG1S3aoaMc8iSTKO-Es0"
tel_group_id = "vaccine_alert1009"
print_flag = 'Y'

print("Starting search for Covid vaccine slots!")

actual = datetime.today()
list_format = [actual + timedelta(days=i) for i in range(num_days)]
actual_dates = [i.strftime("%d-%m-%Y") for i in list_format]
def send_msg_on_telegram(msg):
    telegram_api_url = f"https://api.telegram.org/bot{tele_auth_token}/sendMessage?chat_id=@{tel_group_id}&text={msg}"
    tel_resp = requests.get(telegram_api_url)
    if tel_resp.status_code == 200:
        print("Notification has been sent on Telegram")
    else:
        print("Could not send Message")
while True:
    counter = 0

    for pincode in pincodes:
        for given_date in actual_dates:

            URL = "https://cdn-api.co-vin.in/api/v2/appointment/sessions/public/calendarByPin?pincode={}&date={}".format(
                pincode, given_date)
            header = {
                'User-Agent': 'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/56.0.2924.76 Safari/537.36'}

            result = requests.get(URL, headers=header)

            if result.ok:
                response_json = result.json()
                if response_json["centers"]:
                    if (print_flag.lower() == 'y'):
                        for center in response_json["centers"]:
                            for session in center["sessions"]:
                                if (session["min_age_limit"] <= age and session["available_capacity"] > 0):
                                    print('Pincode: ' + pincode)
                                    print("Available on: {}".format(given_date))
                                    print("\t", center["name"])
                                    print("\t", center["block_name"])
                                    print("\t Price: ", center["fee_type"])
                                    print("\t Availablity : ", session["available_capacity"])
                                    msg = "Vaccine available at Pincode: "+pincode  , "Available on: " + given_date
                                    send_msg_on_telegram(msg)
                                    if (session["vaccine"] != ''):
                                        print("\t Vaccine type: ", session["vaccine"])
                                    print("\n")
                                    counter = counter + 1
            else:
                print("No Response!")

    if counter==0:
        print("No Vaccination slot available!")
        msg="No Vaccination slot available!"
        send_msg_on_telegram(msg)
    else:
        mixer.init()
        mixer.music.load('sound_dingdong.wav')
        mixer.music.play()
        print("Search Completed!")

    dt = datetime.now() + timedelta(minutes=3)

    while datetime.now() < dt:
        time.sleep(1)
