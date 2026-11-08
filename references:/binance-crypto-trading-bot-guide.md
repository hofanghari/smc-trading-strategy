# Hướng Dẫn Kỹ Thuật Và Thiết Kế AI Agent Giao Dịch Tiền Mã Hóa

Tài liệu này tổng hợp tri thức kỹ thuật, các trường phái thuật toán giao dịch định lượng, kiến trúc hệ thống, quy trình vận hành Paper Trading và danh mục 10 kho mã nguồn mở tham khảo trên GitHub dành cho AI Agent lập trình bot giao dịch trên sàn Binance Futures.

---

## 1. Tổng Quan Trí Tuệ Kỹ Thuật Và Các Chiến Lược Giao Dịch Cốt Lõi

Phân tích và tổng hợp 10 dự án mã nguồn mở cho thấy các hệ thống giao dịch tự động tiền mã hóa hiện nay được phát triển dựa trên 5 nhóm chiến lược cốt lõi:

### 1.1. Chiến Lược Giao Dịch Hai Chiều Đồng Thời (Hedge Mode Crossover)
Dựa trên cơ chế Binance Futures Hedge Mode (cho phép giữ đồng thời vị thế mua Long và vị thế bán Short trên cùng một tài sản), thuật toán phát tín hiệu bằng đường trung bình động lũy thừa (EMA Crossover). Khi đường EMA nhanh cắt đường EMA chậm, bot kích hoạt đồng thời hai vị thế ngược chiều với các mức Chốt lời (Take Profit) và Cắt lỗ (Stop Loss) đối xứng. Khi một vị thế dính Stop Loss, vị thế còn lại sẽ tự động được điều chỉnh điểm Stop Loss về mức hòa vốn nhằm tối ưu hóa xác suất thắng tổng thể. Quản lý phí giao dịch (Taker Fees) và bộ lọc tín hiệu nhiễu là hai điều kiện tiên quyết để chiến lược đạt lợi nhuận thực tế.

### 1.2. Chiến Lược Tâm Lý Thị Trường (Long/Short Ratio Sentiment)
Chiến lược khai thác chỉ số tâm lý *Top Trader Long/Short Ratio (Positions)* do Binance Futures API cung cấp thời gian thực. Khi tỷ lệ số lượng tài khoản giữ vị thế Long so với Short của các nhà giao dịch hàng đầu vượt quá ngưỡng 1.0, hệ thống tự động mở lệnh Long; ngược lại khi tỷ lệ giảm xuống dưới 1.0, lệnh Short sẽ được kích hoạt. Mô hình này thường kết hợp với cổng thông báo Telegram Bot để cập nhật liên tục biến động chỉ số và trạng thái tài khoản.

### 1.3. Mô Hình Học Máy Chuỗi Thời Gian (LSTM & Random Forest)
Các hệ thống học máy tiên tiến ứng dụng mạng nơ-ron hồi quy Long Short-Term Memory (LSTM) kết hợp cùng thuật toán cây quyết định Random Forest hoặc Linear Regression để dự báo xu hướng giá. Dữ liệu đầu vào là chuỗi nến lịch sử (OHLCV) kết hợp với tập hợp chỉ báo kỹ thuật (RSI, MACD, Moving Averages, VWAP, Bollinger Bands). Mô hình được huấn luyện định kỳ, theo dõi chỉ số đánh giá sai số (như RMSE) và tự động đưa ra quyết định mua/bán dựa trên xác suất dự báo.

### 1.4. Chiến Lược Tạo Lập Thị Trường Kháng Xu Hướng & Lưới Tự Điều Chỉnh (Passivbot & Trailing Grid)
Phương pháp Market Maker không cố gắng dự báo hướng đi tương lai của giá. Thuật toán liên tục rải các dải giá giới hạn (Price Bands) để cung cấp thanh khoản, sử dụng bộ tối ưu hóa tiến hóa (Evolutionary Optimizer) để tìm bộ tham số tối ưu qua backtest. Đặc biệt, hệ thống tích hợp quy trình gỡ kẹt lệnh (Unstucking Procedure) bằng cách tự động cắt từng phần lỗ nhỏ khi thị trường biến động mạnh một chiều để bảo vệ an toàn cho tài khoản.

### 1.5. Kinh Doanh Chênh Lệch Giá Trung Lập Thị Trường (Market-Neutral Arbitrage & Long-Short Beta)
Chiến lược Arbitrage khai thác sự chênh lệch giá (price spread) giữa hai tài sản có độ tương quan cao (như các hợp đồng perpetual bọc vàng PAXG và XAUT) trên cùng một sàn giao dịch, hoặc chênh lệch giá giữa các sàn giao dịch khác nhau. Hệ thống ưu tiên sử dụng lệnh giới hạn (Maker Order) để vừa giảm trượt giá vừa nhận hoàn phí giao dịch. Trong quản trị danh mục, chiến lược Long-Short Beta xây dựng danh mục trung lập thị trường (Beta-Neutral) bằng cách mua các tài sản Beta thấp và short các tài sản Beta cao để tối ưu hóa chỉ số Sharpe.

---

## 2. Kiến Trúc Mã Nguồn & Engine Mô Phỏng Paper Trading

Để AI Agent phát triển một Bot giao dịch hoàn chỉnh, mã nguồn chương trình cần được cấu trúc theo các module độc lập:

### 2.1. Phân Chia Module Mã Nguồn
- **`config.py`**: Lưu trữ các thông số kết nối API, cấu hình `PAPER_TRADING = True/False`, cặp tiền giao dịch, mức đòn bẩy và tỷ lệ rủi ro.
- **`exchange_client.py`**: Khởi tạo kết nối qua thư viện `ccxt` hoặc `python-binance`, đặt chế độ đòn bẩy (*Crossed/Isolated*) và kích hoạt Hedge Mode (`set_position_mode(dualSidePosition=True)`).
- **`strategy.py`**: Tiếp nhận dữ liệu thị trường thời gian thực, tính toán chỉ báo kỹ thuật hoặc gọi mô hình học máy để phát tín hiệu giao dịch.
- **`paper_trader.py`**: Engine mô phỏng vị thế ảo. Ghi nhận giá vào (Entry), tính toán PnL chưa thực hiện (Unrealized PnL), PnL đã thực hiện (Realized PnL), trừ phí giao dịch giả định (0.04% Taker fee) và cập nhật số dư tài khoản ảo.
- **`risk_manager.py`**: Kiểm soát quy mô vị thế (% vốn/lệnh), tự động đặt mức Take Profit / Stop Loss và theo dõi tỷ lệ sụt giảm tài sản tối đa (Max Drawdown).
- **`logger_notifier.py`**: Ghi nhật ký lịch sử giao dịch dưới dạng JSON/CSV và gửi thông báo trạng thái qua Telegram Bot.

### 2.2. Quy Trình Vận Hành Paper Trading
1. Khởi tạo tài khoản ảo với số dư mặc định (ví dụ: 10,000 USDT mô phỏng).
2. Kết nối WebSocket hoặc REST API để lấy giá thị trường thời gian thực từ Binance Futures.
3. Khi tín hiệu từ `strategy.py` được kích hoạt, `paper_trader.py` khởi tạo vị thế ảo và ghi log vào hệ thống.
4. Mỗi chu kỳ cập nhật giá (Tick/Kline close), cập nhật PnL và kiểm tra điều kiện dính TP/SL để đóng vị thế mô phỏng.

---

## 3. Các Thư Viện Python Khuyên Dùng

| Nhóm Chức Năng | Thư Viện Khuyên Dùng |
| :--- | :--- |
| **Kết nối API & Lấy Dữ liệu** | `ccxt`, `python-binance`, `requests`, `websockets` |
| **Phân Tích & Chỉ Báo** | `pandas`, `numpy`, `ta-lib`, `pandas-ta` |
| **Học Máy & Dự Báo** | `scikit-learn`, `torch` (PyTorch), `tensorflow`, `statsmodels` |
| **Tối Ưu & Lưu Trữ** | `scipy`, `docker`, `sqlite3` |

---

## 4. Mẫu Prompt Chuẩn Hóa Gửi AI Agent Lập Trình Bot

Dưới đây là mẫu Prompt có cấu trúc chuẩn để giao nhiệm vụ lập trình trực tiếp cho AI Agent:

```markdown
Bạn là một Kỹ sư Giao dịch Định lượng (Quantitative Trading Engineer). 
Hãy viết cho tôi một ứng dụng Python hoàn chỉnh chạy Paper Trading Bot trên Binance Futures dựa trên cấu trúc module như sau:

1. Yêu cầu Cấu trúc & Kết nối:
   - Sử dụng thư viện `ccxt` để kết nối Binance Futures API.
   - Hỗ trợ biến cấu hình `PAPER_TRADING = True` (mô phỏng với 10,000 USDT ảo).
   - Thiết lập chế độ vị thế song song Hedge Mode.

2. Chiến lược Giao dịch (Hybrid EMA & RSI Strategy):
   - Sử dụng dữ liệu nến 15 phút cặp BTC/USDT.
   - Mở vị thế Long khi EMA(9) cắt lên EMA(21) và RSI(14) > 50.
   - Mở vị thế Short khi EMA(9) cắt xuống EMA(21) và RSI(14) < 50.
   - Thiết lập đòn bẩy 10x, tỷ lệ quản lý vốn 2% tài khoản ảo cho mỗi lệnh.
   - Tự động đặt mức Take Profit tại 1.5% và Stop Loss tại 0.8%.

3. Engine Mô phỏng (Paper Trading Engine):
   - Ghi nhật ký các lệnh vào/ra vào tệp JSON `trade_history.json`.
   - Tính toán chính xác Realized PnL, trừ phí giao dịch mô phỏng 0.04%.
   - Ngừng giao dịch nếu Max Drawdown đạt 10% tổng tài sản ảo.

4. Yêu cầu Mã nguồn:
   - Mã nguồn được phân chia thành các tệp: `config.py`, `strategy.py`, `paper_trader.py`, `main.py`.
   - Cung cấp hướng dẫn chi tiết cách cài đặt thư viện và khởi chạy bot.
```

---

## 5. Thư Mục Mã Nguồn Mở Tham Khảo (GitHub Repositories Directory)

Danh mục 10 kho mã nguồn mở dưới đây cung cấp tài liệu chi tiết và mã nguồn thực thi cho từng chiến lược thuật toán để AI Agent truy cập tham khảo:

1. **Passivbot**
   - **Đường dẫn Repository**: GitHub (`enarjord/passivbot`)
   - **Mô tả kỹ thuật**: Bot tạo lập thị trường tự động trên hợp đồng vĩnh cửu Binance, Bybit, OKX... Áp dụng thuật toán tối ưu hóa tiến hóa và quy trình tự động gỡ kẹt lệnh.

2. **Binance Futures Hedge Mode Bot**
   - **Đường dẫn Repository**: GitHub (`gokhannsahin/binance_hedge_bot`)
   - **Mô tả kỹ thuật**: Thuật toán giao dịch Binance Futures chạy trên cơ chế Hedge Mode, mở song song vị thế Long và Short dựa trên điểm cắt EMA.

3. **Top Trader Long/Short Ratio Bot**
   - **Đường dẫn Repository**: GitHub (`Crypto trading bot for Futures on Binance`)
   - **Mô tả kỹ thuật**: Bot tự động giao dịch Futures theo tỷ lệ vị thế Long/Short của nhà giao dịch top đầu trên Binance kèm thông báo Telegram.

4. **CryptoBoat AI Trading Bot**
   - **Đường dẫn Repository**: GitHub (`Solrikk/Cripto-Boats`)
   - **Mô tả kỹ thuật**: Hệ thống giao dịch Futures kết hợp trí tuệ nhân tạo với mô hình LSTM và Random Forest trên dữ liệu nến thực tế.

5. **LSTM Automated Trading Bot**
   - **Đường dẫn Repository**: GitHub (`valerio591/LSTM-Automated-Trading-Bot`)
   - **Mô tả kỹ thuật**: Bot tự động sử dụng mô hình học máy LSTM để dự báo biến động giá và kích hoạt lệnh giao dịch hợp đồng vĩnh cửu.

6. **Bitcoin Trading Bot Dissertation**
   - **Đường dẫn Repository**: GitHub (`MohamadKanso/Dissertation`)
   - **Mô tả kỹ thuật**: Dự án nghiên cứu dự báo giá Bitcoin với LSTM và Linear Regression, đánh giá chỉ số RMSE và thực thi backtesting qua Binance API.

7. **GoldArb Price-Spread Arbitrage**
   - **Đường dẫn Repository**: GitHub (`Patrick-code-Bot/GoldArb`)
   - **Mô tả kỹ thuật**: Bot kinh doanh chênh lệch giá spread giữa 2 hợp đồng bọc vàng PAXG và XAUT trên cùng sàn giao dịch, tối ưu bằng lệnh Maker.

8. **Cryptocurrency Arbitrage Bot**
   - **Đường dẫn Repository**: GitHub (`joepegler/Cryptocurrency-Arbitrage-Bot`)
   - **Mô tả kỹ thuật**: Hệ thống Node.js giao dịch chênh lệch giá mua/bán giữa các sàn Bitcoin khác nhau, đảm bảo trạng thái trung lập với biến động thị trường.

9. **Long-Short Beta Trading Strategy**
   - **Đường dẫn Repository**: GitHub (`saurabh9651/long_short_beta`)
   - **Mô tả kỹ thuật**: Thuật toán quản lý danh mục Beta-neutral trong Python, thực hiện long coin/cổ phiếu Beta thấp và short Beta cao để tối ưu chỉ số Sharpe.

10. **Cryptocurrency Trading Bot RNN**
    - **Đường dẫn Repository**: GitHub (`liampgrichardson/Cryptocurrency_Trading_Bot`)
    - **Mô tả kỹ thuật**: Hệ thống giao dịch tự động kết hợp mạng nơ-ron hồi quy RNN và các chỉ báo kỹ thuật cơ bản trên cặp BTC/USDT.
