import streamlit as st
import requests
import pandas as pd
import pandas_ta as ta

# Bybit API endpoint for OHLCV data
def fetch_bybit_data(symbol, interval, limit=200):
    url = f'https://api.bybit.com/v5/market/kline'
    params = {
        'category': 'linear',
        'symbol': symbol,
        'interval': interval,
        'limit': limit
    }
    response = requests.get(url, params=params)
    data = response.json()
    if data['ret_code'] != 0:
        st.error(f"Error fetching data for {symbol}: {data['ret_msg']}")
        return None
    
    kline_data = data['result']['list']
    df = pd.DataFrame(kline_data, columns=['timestamp', 'open', 'high', 'low', 'close', 'volume', 'turnover'])
    df['timestamp'] = pd.to_datetime(df['timestamp'], unit='s')
    df.set_index('timestamp', inplace=True)
    df = df.astype({'open': 'float', 'high': 'float', 'low': 'float', 'close': 'float', 'volume': 'float'})
    return df

# Fetch all available futures pairs
def fetch_all_pairs():
    url = 'https://api.bybit.com/v5/market/instruments-info'
    params = {'category': 'linear'}
    response = requests.get(url, params=params)
    data = response.json()
    if data['ret_code'] != 0:
        st.error(f"Error fetching pairs: {data['ret_msg']}")
        return []
    pairs = [item['symbol'] for item in data['result']['list']]
    return pairs

# Add 5-minute price change calculation
def calculate_price_change(df):
    df['5min_change'] = df['close'].pct_change() * 100
    return df

# Screener with filters
def screener(df, volume_threshold, price_change_threshold):
    df['RSI'] = ta.rsi(df['close'], length=14)
    high_volume = df[df['volume'] > volume_threshold]
    overbought = high_volume[high_volume['RSI'] > 70]
    oversold = high_volume[high_volume['RSI'] < 30]
    significant_change = df[abs(df['5min_change']) > price_change_threshold]
    return overbought, oversold, significant_change

# Streamlit App
def main():
    st.title("Crypto Futures Screener")
    st.sidebar.header("Settings")
    
    # User inputs for filters
    volume_threshold = st.sidebar.number_input("Volume Threshold", value=1000000, step=100000)
    price_change_threshold = st.sidebar.number_input("5-Minute Price Change (%) Threshold", value=1.0, step=0.1)
    
    st.sidebar.write("Click 'Run Screener' to process.")
    
    if st.sidebar.button("Run Screener"):
        st.write("Fetching data...")
        
        # Fetch pairs and process
        pairs = fetch_all_pairs()
        if not pairs:
            st.error("No pairs available!")
            return
        
        st.write(f"Processing {len(pairs)} pairs...")
        results = []
        
        for symbol in pairs[:10]:  # Limit to 10 pairs for demo
            st.write(f"Processing {symbol}...")
            df = fetch_bybit_data(symbol, '5')
            if df is not None:
                df = calculate_price_change(df)
                overbought, oversold, significant_change = screener(df, volume_threshold, price_change_threshold)
                results.append({
                    'symbol': symbol,
                    'overbought': overbought,
                    'oversold': oversold,
                    'significant_change': significant_change
                })
        
        # Display results
        for result in results:
            st.subheader(f"Results for {result['symbol']}")
            st.write("**Overbought:**")
            st.dataframe(result['overbought'])
            st.write("**Oversold:**")
            st.dataframe(result['oversold'])
            st.write("**Significant 5-Minute Price Changes:**")
            st.dataframe(result['significant_change'])

if __name__ == "__main__":
    main()
