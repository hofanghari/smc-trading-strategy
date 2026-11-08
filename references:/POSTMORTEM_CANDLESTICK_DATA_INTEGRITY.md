# POSTMORTEM LESSON LEARNING: CANDLESTICK DATA INTEGRITY & MARKET FEED SEPARATION
**Standard Operating Procedure & Engineering Invariant (`SOP-CANDLE-001` v1.0)**

---

## 1. THÔNG TIN SỰ CỐ & TỪ KHÓA TRUY XUẤT (METADATA & RETRIEVAL TAGS)
- **Mã sự cố:** `INCIDENT-20261027-CANDLE-MISMATCH`
- **Mức độ nghiêm trọng:** P0 — Critical Strategy & Data Pipeline Failure
- **Hệ thống bị ảnh hưởng:** SMC Scanner (`src/scanner.py`, `src/data_fetcher.py`, `src/mtf_strategy.py`, `src/web/app.py`)
- **Trạng thái khắc phục:** ĐÃ KHẮC PHỤC TRIỆT ĐỂ (VERIFIED FIXED 100%)
- **Từ khóa truy xuất (Keywords):**  
  `SMC`, `Price Action`, `Candlestick Data Integrity`, `Binance Testnet vs Live Market`, `Confirmed Candle Close`, `Lookahead Bias`, `Data Freshness`, `OHLCV Validation`, `TradingView Alignment`

---

## 2. PHÂN TÍCH BA NGUYÊN NHÂN GỐC RỄ & BẰNG CHỨNG (ROOT CAUSE FORENSICS)

### 🔴 Lỗi 1: Lẫn lộn môi trường thực thi mô phỏng (Testnet) với nguồn cấp dữ liệu nến (Market Data)
- **Nguyên nhân gốc:** Khi chạy Paper Trading, `BINANCE_TESTNET=true` được bật. `BinanceClient` khởi tạo `exchange.set_sandbox_mode(True)` trỏ vào `testnet.binancefuture.com`. `DataFetcher` dùng chung instance `client.exchange` này để tải nến OHLCV thay vì tải từ production live market.
- **Bằng chứng số liệu (Cùng nến 01:00 UTC `1793062800000`):**
  - **Live Futures (`fapi.binance.com`) / TradingView (`BINANCE:BTCUSDT.P`):**  
    `Open: $80,193.20 | High: $80,332.30 | Low: $80,185.60 | Close: $80,300.10 | Volume: 413.1 BTC`
  - **Testnet Sandbox (`testnet.binancefuture.com`):**  
    `Open: $80,203.40 | High: $80,286.90 | Low: $80,200.00 | Close: $80,286.90 | Volume: 17,713.1 BTC`
  - **Sai lệch:** High lệch -$45.40 (mất râu quét thanh khoản), Volume lệch 42.8 lần trên BTC và 548 lần trên DOGEUSDT (13.9 tỷ DOGE vs 25 triệu DOGE).

### 🔴 Lỗi 2: Lookahead/Repainting Bias do dùng nến HTF chưa đóng (Forming Bar)
- **Nguyên nhân gốc:** Hàm `analyze_htf` trong `src/mtf_strategy.py` truyền toàn bộ nến HTF bao gồm cả cây nến 1h đang chạy dở vào các thuật toán SMC (`find_swing_points`, `detect_structure_breaks`, `detect_liquidity_sweeps`, `detect_order_blocks`).
- **Bằng chứng:** Một cây nến 1h mới mở được 5 phút có râu nến biến động vượt đỉnh cũ tạm thời đã khiến bot vội vàng kết luận là **BOS/CHoCH hoặc Liquidity Sweep**, nhưng khi hết giờ nến rút râu đóng ngược lại thì cấu trúc đó biến mất (repainting).

### 🔴 Lỗi 3: Cache đóng băng và cơ chế kiểm tra dữ liệu cũ không hoàn thiện
- **Nguyên nhân gốc:** Trong `src/web/app.py`, endpoint `/api/candles` chỉ kiểm tra `if not candles:`. Một khi SQLite đã chứa các cây nến testnet cũ từ trước, API không bao giờ kích hoạt tải nến mới, khiến Web Dashboard rơi vào trạng thái `data_freshness: EXPIRED` và đóng băng chart nhiều giờ trước.

---

## 3. BÀI HỌC KINH NGHIỆM Ở BA CẤP ĐỘ (3-TIER LESSONS LEARNED)

### 1. Bài học Kỹ thuật (Engineering Level)
- **Nguyên tắc phân tách tuyệt đối (Strict Decoupling):** Nguồn dữ liệu thị trường (Market Data Feed) và môi trường thực thi lệnh (Order Execution Engine) là hai thực thể độc lập hoàn toàn. Dù lệnh được đặt trên Testnet, Paper Trading hay Real Trading, **DỮ LIỆU NẾN THỊ TRƯỜNG PHẢI LUÔN LUÔN LẤY TỪ PRODUCTION LIVE (`fapi.binance.com`)**.
- **Quy tắc nến đã đóng (Confirmed Closed Candle Invariant):** Cấu trúc nến, Swing Point, Order Block, FVG, BOS/CHoCH tuyệt đối chỉ được tính toán trên nến đã đóng (`iloc[:-1]`). Giá mark hiện tại (`close` của nến đang chạy) chỉ dùng để so sánh khoảng cách và kích hoạt trigger lệnh.
- **Xác thực độ mới dữ liệu (Data Freshness Gate):** Mọi API trả về nến phải kiểm tra `age_seconds = now - (last_candle_timestamp + timeframe_seconds)`. Nếu `age_seconds > tolerance`, phải lập tức cập nhật nến mới từ sàn, cấm phục vụ dữ liệu stale.

### 2. Bài học Chiến lược (Quantitative & SMC Strategy Level)
- **"Garbage In, Garbage Out":** Thuật toán SMC/Price Action phụ thuộc 100% vào độ chính xác từng tick giá High/Low. Một cây râu nến ảo của Testnet lệch vài chục giá sẽ phá hủy toàn bộ hệ thống tính toán:
  - Nhận diện sai đỉnh/đáy được bảo vệ (Protected High/Low).
  - Vẽ ra các Order Block và FVG "ma" không hề tồn tại trên thị trường thật.
  - Kích hoạt Liquidity Sweep giả, đưa bot vào bẫy đu đỉnh/bắt đáy ngược xu hướng.
  - Mức SL/TP tính theo cấu trúc ảo dẫn đến tỷ lệ Risk:Reward bị méo mó và dính SL sớm.

### 3. Bài học Quy trình AI Agent (Autonomous Agent Governance Level)
- **Không bao giờ thỏa mãn chỉ vì Unit Test PASS:** Unit test sử dụng mock data có thể pass 100% trong khi hệ thống đang chạy trên dữ liệu rác ngoài đời thực.
- **Đối chiếu thực nghiệm đa nguồn (Empirical Cross-Verification):** Trước khi tối ưu hoặc kết luận hệ thống hoạt động đúng, AI Agent bắt buộc phải:
  1. Lấy dữ liệu live từ sàn gốc (`curl https://fapi.binance.com`).
  2. Lấy dữ liệu qua API backend của bot (`curl http://127.0.0.1:8080/api/candles`).
  3. Lấy dữ liệu tham chiếu từ TradingView (`BINANCE:<SYMBOL>USDT.P`).
  4. Đối chiếu từng trường Open, High, Low, Close, Volume theo từng timestamp. Nếu lệch bất kỳ số thập phân nào -> **DỪNG LẠI VÀ ĐIỀU TRA NGAY LẬP TỨC**.

---

## 4. QUY TẮC KỸ THUẬT BẮT BUỘC (MANDATORY TECHNICAL PREVENTATIVE RULES)

1. **RULE-01 (Nguồn nến bất biến):** `DataFetcher` bắt buộc sở hữu `market_exchange` trỏ trực tiếp tới `https://fapi.binance.com/fapi/v1`. Tuyệt đối cấm truyền instance sandbox/testnet vào pipeline phân tích kỹ thuật.
2. **RULE-02 (Loại trừ nến đang chạy trên HTF):** `analyze_htf()` luôn nhận mặc định `drop_unclosed=True`. Dữ liệu nến đưa vào tính toán cấu trúc vĩ mô bắt buộc phải là `htf_df.iloc[:-1]`.
3. **RULE-03 (Bảo toàn bất biến toán học OHLCV):** Mọi nến ghi nhận vào DB hoặc đưa vào thuật toán phải thỏa mãn:
   - `High >= max(Open, Close)`
   - `Low <= min(Open, Close)`
   - `Volume >= 0`
   - Timestamp đơn điệu tăng dần nghiêm ngặt (Strictly Monotonic Ascending) và không có bản ghi trùng lặp.
4. **RULE-04 (Hỗ trợ Force Refresh trên API):** Endpoint `/api/candles` hỗ trợ tham số `refresh=true` để phục vụ tự động làm mới khi phát hiện dữ liệu cũ.
5. **RULE-05 (Thử nghiệm hồi quy tự động):** Mọi thay đổi liên quan đến nến phải vượt qua bài kiểm thử [`tests/test_candlestick_forensic.py`](file:///home/tts/binance-smc-bot/tests/test_candlestick_forensic.py).

---

## 5. CHECKLIST KIỂM TRA TRƯỚC KHI CAN THIỆP SMC (PRE-FLIGHT CHECKLIST)

Trước khi bất kỳ AI Agent nào sửa code hoặc tối ưu logic SMC, PHẢI kiểm tra đủ 5 điều kiện:
- [ ] **1. Nguồn nến:** `DataFetcher.market_exchange` có đang trỏ về `fapi.binance.com` không? Có bị dính sandbox/testnet URL không?
- [ ] **2. Đối chiếu nến:** Chọn 1 cây nến đã đóng gần nhất của BTCUSDT và ETHUSDT, chạy curl so sánh với `fapi.binance.com`. OHLCV có khớp 100% không?
- [ ] **3. Trạng thái nến:** Thuật toán phân tích cấu trúc có loại bỏ nến chưa đóng (forming candle) không (`drop_unclosed=True`)?
- [ ] **4. Độ tươi dữ liệu:** Trạng thái `/api/candles` có hiển thị `REALTIME` với độ trễ < 90s không?
- [ ] **5. Test Suite:** Bài test `test_candlestick_forensic.py` có PASS 100% không?
