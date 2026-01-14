import FinanceDataReader as fdr
import yfinance as yf
import pandas as pd
import requests
import os
import time

# [보안 설정] 깃허브 금고(Secrets)에서 꺼내오기
TELEGRAM_TOKEN = os.environ.get('TELEGRAM_TOKEN')
CHAT_ID = os.environ.get('CHAT_ID')

def send_telegram_message(message):
    if not TELEGRAM_TOKEN or not CHAT_ID:
        print("토큰이나 ID가 설정되지 않았습니다.")
        return
    url = f"https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage"
    data = {"chat_id": CHAT_ID, "text": message}
    try:
        requests.post(url, data=data)
    except Exception as e:
        print(f"전송 실패: {e}")

def get_top_tickers(limit=50): # 요청하신대로 50개로 늘렸습니다!
    print(f"KOSPI 상위 {limit}개 종목 리스트를 불러옵니다...")
    df_krx = fdr.StockListing('KOSPI')
    top_stocks = df_krx.sort_values(by='Marcap', ascending=False).head(limit)
    return dict(zip(top_stocks['Code'], top_stocks['Name']))

def scanner():
    send_telegram_message(f"🌤️ [모닝 브리핑] KOSPI 상위 50위 종목 스캔 시작!")
    
    tickers_map = get_top_tickers(50)
    found_count = 0
    
    for code, name in tickers_map.items():
        try:
            ticker = f"{code}.KS"
            # 데이터 다운로드 (속도를 위해 progress=False)
            data = yf.download(ticker, period='4mo', progress=False)
            if isinstance(data.columns, pd.MultiIndex):
                data.columns = data.columns.droplevel(1)
            
            if len(data) < 60: continue

            # 일목균형표 계산
            high_9 = data['High'].rolling(window=9).max()
            low_9 = data['Low'].rolling(window=9).min()
            tenkan = (high_9 + low_9) / 2
            
            high_26 = data['High'].rolling(window=26).max()
            low_26 = data['Low'].rolling(window=26).min()
            kijun = (high_26 + low_26) / 2
            
            # 오늘과 어제 데이터 비교
            today = data.iloc[-1]
            yesterday = data.iloc[-2]
            
            # 매수 신호 (골든크로스)
            if (yesterday['Tenkan'] <= yesterday['Kijun']) and (today['Tenkan'] > today['Kijun']):
                msg = f"💎 [매수 포착] {name} ({code})\n"
                msg += f"현재가: {today['Close']:,.0f}원\n"
                msg += f"신호: 전환선 골든크로스 발생!"
                send_telegram_message(msg)
                found_count += 1
                time.sleep(0.5) # 메시지 너무 빨리 보내면 차단될 수 있어서 쉼표
            
        except Exception:
            continue
            
    send_telegram_message(f"✅ 스캔 완료! 총 {found_count}개의 매수 기회를 발견했습니다.")

if __name__ == "__main__":
    scanner()
