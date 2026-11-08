# TRADING RULESET & SOP FOR ANTIGRAVITY AI AGENT
> **Version:** 1.0  
> **Target Engine:** Antigravity AI Agent / Trading Bot  
> **Source Material:** Notebook "Chiến lược giao dịch TradingView tối ưu" (31 Sources)  
> **Document Purpose:** Systematized knowledge base, structured logic (IF-THEN), and decision-making framework for automated market analysis, signal validation, and risk management.

---

## 1. AGENT SYSTEM DIRECTIVE & CONTEXT

```yaml
agent_config:
  agent_name: "Antigravity Execution & Strategy Engine"
  primary_task: "Identify high-probability trade setups, validate multi-timeframe confluence, execute risk management protocols, and generate trade signals."
  supported_markets: ["Crypto Futures", "Forex", "XAUUSD (Gold)", "Indices"]
  execution_timeframes: ["1m", "5m", "15m", "1H", "4H"]
  core_philosophy: "Smart Money Concepts (SMC) + Quantitative Indicator Filters + Strict Capital Preservation"
```

---

## 2. CORE CONCEPTS & DEFINITIONS PROTOCOL

### 2.1. Market Structure & Trend Bias
- **BOS (Break of Structure):** Giá phá vỡ đỉnh/đáy liền kề theo xu hướng hiện tại, xác nhận sự tiếp diễn xu hướng.
- **CHoCH (Change of Character):** Giá phá vỡ đỉnh/đáy tạo ra bước chuyển đổi xu hướng từ Tăng -> Giảm hoặc Giảm -> Tăng.
- **Liquidity Draw (Lực hút thanh khoản):** Giá luôn có xu hướng chuyển động về phía các vùng tập trung nhiều thanh khoản (Stop Losses) trước khi đảo chiều.

### 2.2. Supply & Demand / Order Block (OB)
- **Extreme Order Block:** Vùng Cung/Cầu xa nhất ở gốc của sóng đẩy (có tỷ lệ thắng cao nhất, R:R tối ưu).
- **Decision Order Block:** Vùng Cung/Cầu nằm giữa con sóng, gây ra phá vỡ cấu trúc gần nhất (rủi ro hơn, cần thêm xác nhận).
- **POI (Point of Interest):** Vùng giá quan trọng trên khung lớn (4H/1H) bao gồm Order Block, FVG, hoặc đỉnh/đáy thanh khoản.

### 2.3. Imbalance & Fair Value Gap (FVG)
- **FVG (Fair Value Gap):** Mẫu hình 3 nến trong đó giá của nến 1 và nến 3 không chạm nhau, tạo ra khoảng trống giá mất cân bằng (Imbalance).
- **Phân biệt Imbalance & FVG:** FVG là khoảng trống giá cụ thể hình thành từ mô hình 3 nến, còn Imbalance là khái niệm chung về sự lệch pha giữa lực Mua và Bán dẫn đến nến thân dài mạnh mẽ.

### 2.4. Liquidity & Inducement (Thanh khoản & Bẫy)
- **Equal Highs / Equal Lows (EQH/EQL):** Các đỉnh hoặc đáy bằng nhau, nơi tập trung lượng lớn lệnh Cắt lỗ (Stop Loss) của Trader cá nhân.
- **BSL (Buy Side Liquidity):** Thanh khoản phía Mua (nằm trên các đỉnh cũ/EQH).
- **SSL (Sell Side Liquidity):** Thanh khoản phía Bán (nằm dưới các đáy cũ/EQL).
- **Inducement (IDM - Bẫy dẫn dụ):** Điểm bẫy dụ phe Mua/Bán vào lệnh sớm trước khi giá thật sự chuyển động về vùng POI thực sự.

### 2.5. Pricing Zones (Vùng giá Chiết khấu & Cao cấp)
- **Premium Zone (Vùng Cao cấp):** Vùng giá nằm trên mức Fibonacci 0.5 (Ưu tiên tìm lệnh BÁN).
- **Discount Zone (Vùng Chiết khấu):** Vùng giá nằm dưới mức Fibonacci 0.5 (Ưu tiên tìm lệnh MUA).
- **Optimal Trade Entry (OTE):** Vùng Fibonacci 0.618 - 0.786.

### 2.6. Volume Profile & Market Profile
- **POC (Point of Control):** Mức giá có khối lượng giao dịch lớn nhất trong phiên/chu kỳ.
- **VAH (Value Area High):** Mức giá cao nhất của vùng giá trị (chiếm 68%-70% tổng khối lượng).
- **VAL (Value Area Low):** Mức giá thấp nhất của vùng giá trị.
- **HVN (High Volume Node):** Vùng khối lượng tích lũy lớn -> Hỗ trợ/Kháng cự mạnh.
- **LVN (Low Volume Node):** Vùng khối lượng cực ít -> Giá biến động rất nhanh khi đi qua.

---

## 3. MASTER STRATEGY SETUPS (IF-THEN LOGIC FOR AGENT)

### SETUP 1: SMC A+ Trade Setup (Multi-Timeframe Confirmation)
```yaml
setup_id: "SMC_A_PLUS_01"
timeframes:
  macro: "4H / 1H"
  micro: "5m / 1m"
conditions_buy:
  - IF: "Giá chạm vùng Discount Zone (Fib < 0.5) của sóng 4H/1H"
  - AND: "Giá chạm vào Extreme POI / Demand Block trên 1H"
  - AND: "Xuất hiện Liquidity Sweep (quét đáy EQL / SSL cũ)"
  - AND: "Trên khung 5m/1m xuất hiện CHoCH tăng kèm FVG (Imbalance)"
  - THEN: "Đặt lệnh BUY Limit tại FVG khung nhỏ hoặc Market BUY khi giá retest FVG"
stop_loss: "Đặt dưới đáy của cây nến Sweep Liquidity + 1-2 pips buffer"
take_profit: "TP1 tại BSL gần nhất; TP2 tại Vùng Supply 1H đối ứng (Target R:R >= 1:3)"
```

### SETUP 2: Liquidity Sweep & Go (Bẫy Thanh Khoản & Đảo Chiều Nhanh)
```yaml
setup_id: "LIQUIDITY_SWEEP_02"
timeframes: ["5m", "15m"]
conditions_buy:
  - IF: "Giá phá xuống dưới đáy EQH/EQL hoặc đáy phiên Á/Âu (Quét SSL)"
  - AND: "Nến quét thanh khoản rút chân mạnh (Pinbar/Hammer) hoặc nến tiếp theo đóng cửa ngược lại bên trong vùng giá cũ"
  - AND: "Volume tại nến quét thanh khoản tăng vọt (Spike Volume)"
  - THEN: "Mở lệnh BUY Market ngay khi nến xác nhận đóng cửa"
stop_loss: "Đặt ngay dưới râu nến quét thanh khoản"
take_profit: "Đỉnh cũ gần nhất hoặc BSL đối diện (R:R 1:2 đến 1:4)"
```

### SETUP 3: Scalping 5 Phút Bộ 3 EMA (25, 50, 100)
```yaml
setup_id: "SCALPING_EMA_3_03"
timeframes: ["5m"]
indicators:
  - EMA_25: "Xu hướng ngắn hạn"
  - EMA_50: "Xu hướng trung hạn"
  - EMA_100: "Bộ lọc xu hướng dài hạn"
conditions_buy:
  - IF: "EMA 25 > EMA 50 > EMA 100 (Sắp xếp theo thứ tự xu hướng tăng rõ ràng)"
  - AND: "Giá quay trở lại hồi (Pullback) chạm vào vùng giữa EMA 25 và EMA 50"
  - AND: "Xuất hiện nến xác nhận tăng giá (Bullish Engulfing / Hammer / Pinbar) tại vùng EMA"
  - THEN: "Mở lệnh BUY Market"
stop_loss: "Đặt dưới đường EMA 100 hoặc đáy swing low gần nhất"
take_profit: "Tỷ lệ R:R cố định 1:1.5 hoặc 1:2"
```

### SETUP 4: Scalping 1 Phút EMA 200 + RSI + Engulfing
```yaml
setup_id: "SCALPING_1M_RSI_200EMA_04"
timeframes: ["1m"]
indicators:
  - EMA_200: "Bộ lọc xu hướng chính"
  - RSI_14: "Động lượng"
conditions_buy:
  - IF: "Giá nằm trên đường EMA 200 (Xác nhận xu hướng tăng)"
  - AND: "RSI vọt lên trên đường 50 (Xác nhận động lượng Mua chiếm ưu thế)"
  - AND: "Xuất hiện mô hình nến Bullish Engulfing (Nến nhấn chìm tăng)"
  - THEN: "Mở lệnh BUY ngay khi nến Engulfing đóng cửa"
stop_loss: "Đặt dưới đáy của nến Engulfing 1-2 pips"
take_profit: "R:R 1:1.5 hoặc 1:2"
```

### SETUP 5: Scalping 1 Phút Fibonacci Golden Zone (0.618 - 0.5)
```yaml
setup_id: "SCALPING_FIBO_1M_05"
timeframes: ["1m"]
conditions_buy:
  - IF: "Giá tạo một sóng tăng rõ ràng phá đỉnh trước đó (BOS 1m)"
  - AND: "Kéo Fibonacci Retracement từ Đáy sóng đến Đỉnh sóng"
  - AND: "Giá hồi quy về Vùng Vàng Fibonacci 0.5 - 0.618 (hoặc 0.786)"
  - AND: "Xuất hiện nến đảo chiều tăng giá tại vùng Fibo"
  - THEN: "Mở lệnh BUY tại 0.618"
stop_loss: "Đặt dưới mức Fibonacci 0.786 hoặc dưới đáy sóng khởi đầu"
take_profit: "TP1 tại mức Fibo 0 (Đỉnh cũ); TP2 tại Mức Fibonacci Extension 1.272 / 1.618"
```

### SETUP 6: MACD Crossover + EMA 200 Filter
```yaml
setup_id: "MACD_EMA200_06"
timeframes: ["15m", "1H"]
indicators:
  - EMA_200: "Bộ lọc xu hướng"
  - MACD: "(12, 26, 9)"
conditions_buy:
  - IF: "Giá nằm trên EMA 200"
  - AND: "Đường MACD giao cắt lên trên đường Signal (MACD Cross Over)"
  - AND: "Điểm giao cắt diễn ra ở phía dưới đường 0 (Histogram âm dần và chuyển sang dương)"
  - THEN: "Mở lệnh BUY"
stop_loss: "Đáy gần nhất trước điểm giao cắt MACD"
take_profit: "Chốt lời khi MACD cắt xuống hoặc đạt R:R 1:2"
```

### SETUP 7: Volume Profile POC Bounce Strategy
```yaml
setup_id: "VOLUME_PROFILE_POC_07"
timeframes: ["15m", "1H"]
indicators:
  - Volume_Profile: "Fixed Range / Session Volume Profile"
conditions_buy:
  - IF: "Giá đang trong xu hướng tăng trên khung 1H"
  - AND: "Giá điều chỉnh về vùng POC (Point of Control) của phiên giao dịch trước hoặc vùng tích lũy cũ"
  - AND: "Vùng POC trùng với vùng hỗ trợ/Order Block"
  - AND: "Khối lượng Bán giảm dần khi tiếp cận POC (Low Selling Volume)"
  - THEN: "Mở lệnh BUY khi xuất hiện nến phản ứng tại POC"
stop_loss: "Đặt dưới đường VAL (Value Area Low) hoặc dưới cụm nến tích lũy POC"
take_profit: "Hướng tới vùng VAH (Value Area High) hoặc đỉnh cũ"
```

### SETUP 8: Williams Vix Fix Bottom Detector (Nhận Diện Đáy Thị Trường)
```yaml
setup_id: "WILLIAMS_VIX_FIX_BOTTOM_08"
timeframes: ["1H", "4H", "1D"]
indicators:
  - Williams_Vix_Fix: "Chỉ báo biến động đáy"
  - Stochastic_Oscillator: "(14, 3, 3)"
conditions_buy:
  - IF: "Cột chỉ báo Williams Vix Fix xuất hiện màu đỏ/cam (Spike biến động đạt cực đại - Cảnh báo đáy)"
  - AND: "Stochastic nằm trong vùng quá bán (< 20) và giao cắt đi lên"
  - AND: "Giá chạm vùng hỗ trợ cứng / Demand Zone khung D1/4H"
  - THEN: "Kích hoạt tín hiệu MUA ĐÁY (Bottom Catching Signal)"
stop_loss: "Đặt dưới đáy thấp nhất của nhịp xả giá"
take_profit: "Theo dõi đảo chiều xu hướng lớn, Target R:R 1:3 đến 1:5"
```

---

## 4. RISK MANAGEMENT & POSITION SIZING PROTOCOL

```yaml
risk_management_rules:
  max_risk_per_trade: "1.0% - 2.0% tổng tài khoản"
  max_daily_drawdown: "5.0% (Nếu chạm ngưỡng, Agent dừng giao dịch trong 24 giờ)"
  min_risk_reward_ratio: "1:2 (Không vào lệnh có R:R < 1:1.5)"
  
position_sizing_formula:
  lot_size: "(Total_Account_Balance * Risk_Percentage) / (Stop_Loss_Pips * Pip_Value)"

trade_management:
  move_sl_to_be: "Khi giá đạt 1.5R, tự động dời Stop Loss về hòa vốn (Breakeven)"
  partial_take_profit: "Chốt 50% khối lượng tại TP1 (1.5R - 2R), cho 50% còn lại chạy theo Trailing Stop"
  trailing_stop: "Dời Stop Loss theo Swing Low/High khung 1m/5m hoặc theo đường EMA 25"
```

---

## 5. PRE-TRADE EXECUTION CHECKLIST (VERIFICATION SOP FOR AGENT)

Trước khi phát tín hiệu hoặc đặt lệnh tự động, Agent **BẮT BUỘC** kiểm tra 5 tiêu chí:

- [ ] **1. Trend Alignment:** Khung lớn (1H/4H) và khung nhỏ (5m/1m) có đồng thuận xu hướng không?
- [ ] **2. Location (Vùng giá):** Giá đang ở Premium (cho lệnh Sell) hay Discount (cho lệnh Buy)?
- [ ] **3. Liquidity Status:** Đã có cú quét thanh khoản (Liquidity Sweep / Inducement) chưa?
- [ ] **4. Confirmation Signal:** Có nến xác nhận (Engulfing/Pinbar/CHoCH/FVG) chưa?
- [ ] **5. Risk Parameters:** Khoảng cách Stop Loss có an toàn không? Tỷ lệ R:R có đạt tối thiểu 1:1.5 không?

---
*Tài liệu này được trích xuất và chuẩn hóa từ hệ thống 31 video bài giảng về Smart Money Concepts, Scalping, Price Action và Chỉ báo kỹ thuật.*
