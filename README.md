# Data-1  
import pandas as pd
import yfinance as yf
import matplotlib.pyplot as plt
import datetime

# Define a function to get historical stock data
def get_stock_data(symbol, start_date, end_date):
    stock_data = yf.download(symbol, start=start_date, end=end_date)
    return stock_data

# Define a function to calculate the Relative Strength Index (RSI)
def calculate_rsi(data, window=36):
    delta = data['Close'].diff()
    gain = delta.where(delta > 0, 0)
    loss = -delta.where(delta < 0, 0)
    avg_gain = gain.rolling(window=window).mean()
    avg_loss = loss.rolling(window=window).mean()
    rs = avg_gain / avg_loss
    rsi = 100 - (100 / (1 + rs))
    return rsi

# Define a function to generate trading signals based on RSI
def generate_signals(data, buy_threshold=30, sell_threshold=70):
    data['RSI'] = calculate_rsi(data)
    data['Buy_Signal'] = 0
    data['Sell_Signal'] = 0
    # Buy signal when RSI is below the threshold (oversold)
    data.loc[data['RSI'] < buy_threshold, 'Buy_Signal'] = 1
    # Sell signal when RSI is above the threshold (overbought)
    data.loc[data['RSI'] > sell_threshold, 'Sell_Signal'] = -1
    return data

# Define a function to plot stock price and date
def plot_stock_price(data):
    plt.figure(figsize=(10, 6))
    plt.plot(data.index, data['Close'], color='purple', label='Stock Price')
    plt.xlabel('Date')
    plt.ylabel('Price')
    plt.title('Stock Price Over Time')
    plt.legend()
    plt.show()

# Example usage
if __name__ == "__main__":
    stock_symbol = 'PFE'
    start_date = '2010-01-01'
    end_date = '2024-10-26'
    stock_data = get_stock_data(stock_symbol, start_date, end_date)
    
    # Generate signals
    stock_data = generate_signals(stock_data)

    # Print sell signals where RSI > 70
    sell_signals = stock_data[stock_data['RSI'] > 70]
    print("Sell Signals (RSI > 70):")
    print(sell_signals[['RSI', 'Sell_Signal']])
    
    # Plot stock price, RSI buy signals and RSI sell signals on the same graph
    plt.figure(figsize=(12, 6))
    plt.plot(stock_data.index, stock_data['Close'], color='purple', label='Stock Price')  # Plot stock price
    plt.scatter(stock_data.index[stock_data['Buy_Signal'] == 1], stock_data['Close'][stock_data['Buy_Signal'] == 1], color='green', marker='^', label='Buy Signal')  # Plot buy signals on price line
    plt.scatter(stock_data.index[stock_data['Sell_Signal'] == -1], stock_data['Close'][stock_data['Sell_Signal'] == -1], color='red', marker='v', label='Sell Signal')  # Plot sell signals on price line
    plt.xlabel('Date')
    plt.ylabel('Price')
    plt.title(f'{stock_symbol} Stock Price and Buy/Sell Signals')
    plt.legend()
    plt.show()

    # Plot stock price and date
    plot_stock_price(stock_data)
    
