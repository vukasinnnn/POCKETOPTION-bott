import logging
import time
from pocketoptionapi.stable_api import PocketOption

# Postavimo debug mode
logging.basicConfig(level=logging.DEBUG, format='%(asctime)s %(message)s')

# Podaci za prijavu
ssid = r"""42["auth",{"session":"a:4:{s:10:\"session_id\";s:32:\"123123123\";s:10:\"ip_address\";s:12:\"2.111.11.5\";s:10:\"user_agent\";s:104:\"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.77 Safari/537.36\";s:13:\"last_activity\";i:123232;}1232321213","isDemo":0,"uid":"123232132"}]"""

def get_signals(account, asset, period=60, offset=120):
    _time = int(time.time())
    candles = account.get_candle(asset, _time, offset, period)
    return candles['data']

def analyze_signals(candles):
    if not candles:
        return None

    last_candle = candles[-1]
    if last_candle['close'] > last_candle['open']:
        return 'call'
    elif last_candle['close'] < last_candle['open']:
        return 'put'
    else:
        return None

def trade(account, asset, direction, amount=1, duration=60):
    buy_info = account.buy(asset, amount, direction, duration)
    return buy_info

def main():
    account = PocketOption(ssid)
    check_connect, message = account.connect()
    
    if check_connect:
        account.change_balance("PRACTICE")  # Možete promeniti u "REAL"
        asset = "EURUSD"
        amount = 1
        duration = 60  # sekunde
        
        print("Početni balans: ", account.get_balance())
        
        while True:
            candles = get_signals(account, asset)
            decision = analyze_signals(candles)
            
            if decision:
                print(f"Trgovina: {decision}")
                trade_info = trade(account, asset, decision, amount, duration)
                print("----Trgovina----")
                print("Rezultat: ", account.check_win(trade_info["id"]))
                print("----Trgovina----")
                print("Trenutni balans: ", account.get_balance())
            
            # Pauza između trgovina
            time.sleep(300)  # 5 minuta
        
        account.close()
    else:
        print("Neuspešna konekcija: ", message)

if __name__ == '__main__':
    main()