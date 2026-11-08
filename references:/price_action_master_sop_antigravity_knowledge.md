---
document_id: PA-MASTER-SOP-ANTIGRAVITY-001
title: Price Action Master SOP — SMC, Liquidity, and Order Flow
version: 2.1
language: vi
document_type: trading_strategy_knowledge
intended_consumer: Antigravity AI Agent
status: reference_ruleset_not_validated
source_basis: User-provided Price Action Master SOP V2.0
---

# PRICE ACTION MASTER SOP — ANTIGRAVITY KNOWLEDGE EDITION

## 0. Cách sử dụng tài liệu

Tài liệu này là **bộ quy tắc tham chiếu** cho phân tích Price Action, Smart Money Concepts (SMC), thanh khoản và Order Flow. Agent phải tra cứu theo từng khái niệm, setup và quy tắc quản trị rủi ro; không coi nội dung trong tài liệu là bằng chứng rằng chiến lược đã có lợi nhuận.

### Thứ tự ưu tiên khi phân tích

1. Kiểm tra dữ liệu thị trường và khung thời gian có đầy đủ, đồng bộ hay không.
2. Xác định cấu trúc thị trường và bias trên H4/H1.
3. Xác định vùng giá, thanh khoản và Point of Interest (POI).
4. Chờ trigger xác nhận trên khung thực thi.
5. Tính Stop Loss (SL), Take Profit (TP), R:R và rủi ro vốn.
6. Chỉ phát tín hiệu khi mọi điều kiện bắt buộc của setup được xác nhận; nếu thiếu dữ liệu hoặc điều kiện mâu thuẫn, ghi rõ lý do và **không phát tín hiệu**.

### Giới hạn an toàn vận hành

- Mặc định dùng tài liệu để phân tích và phát tín hiệu; không tự suy ra quyền đặt lệnh thật.
- Chỉ thực hiện giao dịch khi hệ thống bên ngoài đã cấp quyền rõ ràng. Nếu cấu hình là paper trading, tuyệt đối không gửi lệnh thật.
- Không giả định rằng một setup có win rate cao nếu chưa có kết quả backtest/out-of-sample đáng tin cậy.
- Không dùng dữ liệu tương lai, nến chưa đóng hoặc giá trị indicator được tính lại từ dữ liệu tương lai để xác nhận tín hiệu.
- Phí, spread, slippage, funding và giới hạn thanh khoản phải được tính trong kiểm thử nếu phù hợp với thị trường.

## 1. Cấu hình khung thời gian

| Vai trò | Khung thời gian | Nhiệm vụ |
|---|---|---|
| Macro structure | H4 / H1 | Market bias, cấu trúc chính, key levels, macro POI, Premium/Discount |
| Intermediate setup | M15 | Thanh khoản, inducement, vùng OB tinh chỉnh |
| Micro execution | M5 / M1 | Liquidity sweep, CISD, IFVG entry |

Khung thời gian trên là kiến trúc tham chiếu. Không mặc định mọi thị trường hoặc mọi setup đều có dữ liệu phù hợp ở tất cả khung.

## 2. Nguyên tắc Price Action V2

- **Bối cảnh quan trọng hơn mô hình nến đơn lẻ:** đánh giá nến tại POI, sau sweep, cùng volume và xác nhận cấu trúc.
- **Không mua/bán chỉ vì chạm hỗ trợ/kháng cự:** xác nhận vùng giá, thanh khoản và trigger trước.
- **Ưu tiên false breakout/reclaim có xác nhận** thay vì vào lệnh ngay khi giá phá vỡ.
- **Premium/Discount:** dùng mốc Fibonacci 0.5 của dealing range/sóng được chọn để phân loại tương đối; BUY ưu tiên Discount, SELL ưu tiên Premium.
- **Volume/Order Flow:** khi có dữ liệu phù hợp, đối chiếu POC, VAH, VAL, HVN, Anchored VWAP và Delta Volume. Không giả định các trường này có sẵn nếu nguồn dữ liệu không cung cấp.
- Không coi thuật ngữ SMC là bằng chứng trực tiếp về hành động của tổ chức; đây là cách diễn giải hành vi giá cần được kiểm định.

## 3. Từ điển khái niệm chuẩn

### 3.1 Market structure

- **Swing High / Swing Low:** theo SOP nguồn, dùng mô hình 5 nến: nến trung tâm có đỉnh cao nhất/đáy thấp nhất và được kẹp bởi hai nến có đỉnh thấp hơn/đáy cao hơn ở mỗi bên. Cần chờ đủ nến bên phải để xác nhận; không đánh dấu swing sớm bằng dữ liệu chưa hoàn tất.
- **BOS (Break of Structure):** phá vỡ cấu trúc theo định nghĩa mà hệ thống triển khai. Cần cấu hình rõ cách xác định swing và điều kiện xác nhận bằng giá đóng cửa.
- **Protected Key Level:** swing low/high được xem là mốc bảo vệ sau cú BOS. Theo SOP nguồn, đảo chiều cấu trúc chỉ được xác nhận khi nến đóng cửa vượt qua protected level liên quan.
- **Level Hold:** giá kiểm tra một vùng hỗ trợ/kháng cự và giữ được vùng đó.
- **Level Reclaim:** giá xuyên qua key level rồi đóng cửa trở lại phía bên kia theo hướng lấy lại vùng giá. Cần xác định cụ thể nến xác nhận và khoảng thời gian được coi là “ngay lập tức” trong code.

### 3.2 Order Block

- **Unmitigated Order Block (OB):** vùng nến được xác định là nguồn của một nhịp đẩy mạnh có FVG/imbalance; theo SOP nguồn, giá chưa quay lại chạm vùng đó.
- **Extreme OB:** OB ở gốc của nhịp đẩy.
- **Decision OB:** OB ở giữa nhịp đẩy, được cho là trực tiếp dẫn tới phá vỡ cản gần nhất.
- **Breaker Block:** OB thất bại sau khi giá xuyên qua, có thể được diễn giải là đổi vai trò hỗ trợ/kháng cự.
- Việc xác định biên OB (thân nến hay toàn bộ râu nến), mức mitigation và tiêu chí “nhịp đẩy mạnh” phải được cố định trong code/config; tài liệu nguồn chưa quy định đầy đủ.

### 3.3 Imbalance

- **FVG (Fair Value Gap):** vùng mất cân bằng trong cấu trúc 3 nến; cách tính chuẩn thường được triển khai bằng khoảng cách giữa râu nến 1 và râu nến 3 theo hướng chuyển động. Cần thống nhất điều kiện chính xác cho FVG tăng và giảm trong code.
- **IFVG (Inversion FVG):** FVG bị giá xuyên qua hoàn toàn và được dùng như vùng cản theo hướng ngược lại.
- **BPR (Balanced Price Range):** vùng chồng lấp giữa FVG theo hai hướng ngược nhau.
- Quy tắc nguồn chưa định nghĩa ngưỡng “xuyên qua hoàn toàn”, dung sai, hoặc cách xử lý gap; các giá trị này cần được cấu hình rõ.

### 3.4 Liquidity

- **BSL (Buy-Side Liquidity):** thanh khoản được cho là nằm trên các đỉnh cũ.
- **SSL (Sell-Side Liquidity):** thanh khoản được cho là nằm dưới các đáy cũ.
- **Inducement (IDM):** đỉnh/đáy ngắn hạn trước POI khung lớn, được diễn giải là điểm dẫn dụ thanh khoản.
- **Single Sweep:** quét một vùng thanh khoản rồi đảo chiều.
- **Double Sweep:** quét lần đầu, hồi tạo cấu trúc ngắn hạn, sau đó quét lần hai trước khi giá có thể di chuyển theo hướng đảo chiều.
- Phải lưu mức thanh khoản tham chiếu, thời điểm sweep và tiêu chí xác nhận. Không gắn nhãn sweep chỉ vì giá chạm gần một đỉnh/đáy.

### 3.5 CISD

**CISD (Change in State of Delivery)** là trigger diễn giải sự thay đổi trạng thái phân phối giá. Theo SOP nguồn, tín hiệu xuất hiện khi nến thân mạnh đóng cửa vượt qua vùng giá mở/đóng của cụm nến trước đó liên quan đến cú sweep, cho thấy phe mua/bán đang chiếm ưu thế.

Để triển khai máy móc, cần xác định rõ:
- cụm nến nào được dùng làm tham chiếu;
- điều kiện thân nến mạnh (nếu có ngưỡng);
- giá nào phải bị vượt qua;
- yêu cầu nến đóng cửa;
- cách phân biệt CISD tăng và giảm.

Nếu các tham số trên chưa được cấu hình, CISD chỉ được ghi là nhận định định tính, không được xem là điều kiện máy móc đã xác thực.

### 3.6 Volume và Order Flow

- **POC:** mức giá có khối lượng giao dịch lớn nhất trong profile được chọn.
- **VAH / VAL:** biên trên/dưới của vùng giá trị theo cấu hình Volume Profile.
- **HVN:** vùng có hoạt động khối lượng tương đối cao.
- **AVWAP (Anchored VWAP):** VWAP tính từ một mốc neo cụ thể, ví dụ đỉnh/đáy hoặc sự kiện đã xác định.
- **Delta Volume:** chênh lệch khối lượng mua chủ động và bán chủ động theo phương pháp dữ liệu của nền tảng.
- Nguồn dữ liệu, session, cách tính profile, anchor và timezone phải nhất quán. Không thay thế dữ liệu thiếu bằng giá trị tự bịa.

## 4. Setup thư viện

### Quy tắc chung cho mọi setup

Mỗi setup phải có:
- `setup_id` duy nhất;
- bias và khung thời gian;
- điều kiện bắt buộc tách biệt với điều kiện hỗ trợ;
- trigger chỉ xác nhận khi nến cần thiết đã đóng;
- entry, invalidation/SL, TP và R:R tính từ giá entry thực tế dự kiến;
- trạng thái dữ liệu (đầy đủ/thiếu);
- lý do chấp nhận hoặc từ chối;
- ghi nhận phí và slippage trong backtest.

Không được tự động tạo điều kiện SELL đối xứng từ setup BUY bên dưới nếu chưa có đặc tả và kiểm thử riêng.

### Setup 1 — Double Sweep + CISD Reversal

`setup_id: PA_V2_DOUBLE_SWEEP_CISD_01`

**Khung:** macro H4/H1; thực thi M15/M5.

**Điều kiện BUY theo tài liệu nguồn:**
1. Giá ở Discount Zone (Fib dưới 0.5) của sóng H1/H4 được chọn.
2. Giá sweep SSL lần thứ nhất, ví dụ Asian/London Low hoặc EQL.
3. Giá bật lên tạo FVG ngắn hạn.
4. Giá quay lại sweep lần hai, xuyên đáy của sweep đầu.
5. Nến M15/M5 đóng cửa có CISD tăng; SOP nguồn mô tả thân nến tăng bao trùm nến xả trước đó.
6. Volume tại nến Double Sweep tăng mạnh so với mức tham chiếu được xác định.

**Entry:** BUY Market sau khi nến CISD đóng cửa, hoặc BUY Limit tại IFVG nếu có vùng IFVG hợp lệ.

**SL:** dưới râu nến thấp nhất của Double Sweep cộng buffer. SOP nguồn ghi `2 pips`; đơn vị buffer cần điều chỉnh theo sản phẩm/thị trường và tick size, không dùng nguyên xi cho crypto.

**TP:** TP1 tại BSL gần nhất; TP2 tại VAH hoặc đỉnh phiên trước. R:R mục tiêu tối thiểu theo nguồn: `1:2.5`.

**Từ chối setup nếu:** thiếu xác nhận sweep lần hai, CISD chưa đóng nến, dữ liệu volume không có nhưng lại được coi là điều kiện bắt buộc, hoặc R:R sau chi phí thấp hơn ngưỡng.

### Setup 2 — Level Reclaim + IFVG

`setup_id: PA_V2_LEVEL_RECLAIM_IFVG_02`

**Khung:** macro H1; thực thi M15/M5.

**Điều kiện BUY theo tài liệu nguồn:**
1. Giá xuyên xuống key support quan trọng trên H1.
2. Nến M15/M5 rút chân và đóng trở lại phía trên key level.
3. Nhịp bật tạo IFVG tăng bằng cách xuyên qua FVG giảm trước đó.
4. Giá retest IFVG hoặc key level đã reclaim.

**Entry:** BUY Limit tại IFVG.

**SL:** dưới râu nến thấp nhất của cú fakeout.

**TP:** đỉnh H1 gần nhất. R:R mục tiêu tối thiểu theo nguồn: `1:2.0`.

**Điểm cần cấu hình:** thời gian tối đa giữa cú xuyên level và reclaim; định nghĩa IFVG hợp lệ; cách xử lý khi giá không retest; lệnh limit không khớp phải hết hạn theo quy tắc được cấu hình.

### Setup 3 — Extreme OB + Inducement Clearance

`setup_id: PA_V2_EXTREME_OB_IDM_03`

**Khung:** macro H4/H1; thực thi M15/M5.

**Điều kiện BUY theo tài liệu nguồn:**
1. Xác định Extreme OB trên H1 tại gốc của nhịp BOS.
2. Có IDM (đáy M15 hình thành trước khi giá chạm OB).
3. Giá sweep thanh khoản IDM và chạm Extreme OB.
4. Có Bullish Engulfing hoặc Pinbar rút chân trên M15/M5 tại OB.

**Entry:** BUY Market sau trigger xác nhận.

**SL:** dưới đáy vùng Extreme OB.

**TP:** TP1 tại IDM High; TP2 tại Extreme Supply H1 đối diện. R:R mục tiêu tối thiểu theo nguồn: `1:3.0`.

**Điểm cần cấu hình:** biên OB, tiêu chí BOS, định nghĩa IDM, độ sâu chạm OB và tiêu chuẩn nến Engulfing/Pinbar.

### Setup 4 — POC + Rejection Pinbar + Delta/Volume Spike

`setup_id: PA_V2_VOLUME_PROFILE_POC_REJECTION_04`

**Khung:** macro H1; thực thi M15.

**Điều kiện BUY theo tài liệu nguồn:**
1. Xu hướng H1 tăng.
2. Giá pullback về POC hoặc HVN của Session Volume Profile.
3. POC trùng với AVWAP hoặc Fibonacci 0.618.
4. Có Pinbar từ chối giá tăng (rút chân) tại POC kèm volume tăng mạnh.

**Entry:** BUY khi nến Pinbar đóng cửa.

**SL:** dưới râu Pinbar hoặc dưới VAL.

**TP:** VAH hoặc đỉnh cũ.

**R:R:** tài liệu nguồn không nêu ngưỡng số riêng cho setup này. Áp dụng bộ lọc R:R chung tối thiểu `1:2.0` nếu cấu hình chiến lược tổng thể yêu cầu; không tuyên bố đây là ngưỡng gốc riêng của setup.

**Điểm cần cấu hình:** session, phương pháp tính Volume Profile, ngưỡng volume spike, cách chọn AVWAP và xử lý khi POC/HVN không có dữ liệu.

## 5. Risk & capital management

### 5.1 Vốn dùng để tính rủi ro

- Dùng **vốn thực sự có thể chịu lỗ** theo quy tắc tài khoản, không mặc định vốn danh nghĩa là vốn chịu rủi ro.
- Ví dụ nguồn: tài khoản prop firm danh nghĩa `$50,000`, giới hạn drawdown `$2,000`; dùng `$2,000` làm cơ sở rủi ro. Với risk 1%, số tiền rủi ro là `$20`.
- Với tài khoản nhỏ, không tăng khối lượng theo cảm xúc. Chỉ cân nhắc tăng size theo quy tắc đã kiểm thử và có profit buffer.

### 5.2 Công thức position sizing

Công thức khái niệm:

`Risk Amount = Risk Capital × Risk Percentage`

`Position Size = Risk Amount / Estimated Loss Per Unit at SL`

Trong đó `Estimated Loss Per Unit at SL` phải tính theo đặc tính sản phẩm, contract size, tick/pip value, quy đổi tiền tệ và chi phí dự kiến. Công thức `Lot_Size = (Real_Account_Capital × Risk_Percentage) / (Stop_Loss_Pips × Pip_Value)` chỉ dùng trực tiếp khi định nghĩa pip/lot phù hợp với sản phẩm.

- Risk per trade theo nguồn: `1.0%–1.5%` vốn thực.
- Giới hạn drawdown ngày theo nguồn: `4.0%`; khi chạm ngưỡng, Circuit Breaker dừng giao dịch trong `24 giờ`.
- Các ngưỡng trên là thông số tham chiếu của tài liệu, cần được đối chiếu với cấu hình hệ thống và giới hạn rủi ro cao hơn nếu có.
- Tổng rủi ro của các lệnh tương quan hoặc đang mở cần được tính ở cấp danh mục; tài liệu nguồn chưa quy định giới hạn tổng này.

### 5.3 Quản lý lệnh

- Khi giá đạt `+1.5R`: dời SL về hòa vốn theo quy tắc nguồn.
- Khi giá đạt `+2.0R`: chốt `50%` khối lượng; phần còn lại dùng trailing stop.
- Trailing stop: theo swing high/low M15 gần nhất hoặc EMA 25.
- Trước khi áp dụng tự động, phải kiểm tra cách tính R, phí, slippage, gap, minimum order size và quy tắc sửa/hủy lệnh của sàn.
- “Hòa vốn” nên bao gồm phí và chi phí dự kiến nếu hệ thống cần bảo toàn PnL ròng; chỉ dời về entry có thể vẫn tạo lỗ ròng.

## 6. Checklist bắt buộc trước khi phát tín hiệu

Tất cả mục bắt buộc phải được xác nhận hoặc đánh dấu `UNKNOWN`. Không được biến `UNKNOWN` thành `PASS`.

1. **Macro Trend & Context:** bias H4/H1 và cấu trúc thị trường có được xác định từ nến đã đóng?
2. **Pricing Zone:** BUY có ở Discount, SELL có ở Premium theo dealing range đã chọn?
3. **Liquidity:** có sweep/inducement/double sweep đúng định nghĩa setup không?
4. **Location:** giá có nằm tại POI hợp lệ (Extreme OB, Reclaim, POC, IFVG...)?
5. **Execution Trigger:** trigger M15/M5 đã đóng nến và đáp ứng điều kiện setup?
6. **Risk-to-Reward:** R:R từ entry dự kiến đến TP phù hợp, sau khi tính SL và chi phí ước lượng?
7. **Data quality:** timestamp, OHLCV, timeframe và nguồn dữ liệu có đầy đủ, không bị trùng hoặc lệch?
8. **Risk limits:** rủi ro lệnh, tổng rủi ro mở và drawdown ngày có nằm trong giới hạn?
9. **Execution mode:** chế độ paper/live được xác định rõ; không gửi lệnh nếu quyền thực thi không được cấp rõ ràng.

### Quy tắc quyết định

- `PASS`: tất cả điều kiện bắt buộc của setup và checklist đều đạt.
- `REJECT`: có điều kiện bắt buộc không đạt.
- `UNKNOWN`: dữ liệu hoặc định nghĩa chưa đủ để kết luận. Không phát tín hiệu giao dịch.
- Nếu R:R dưới ngưỡng yêu cầu, từ chối setup; không kéo TP xa hơn một cách tùy tiện chỉ để đạt R:R.

## 7. Định dạng đầu ra khuyến nghị cho Agent

Mỗi lần đánh giá setup, xuất một bản ghi có cấu trúc:

```yaml
timestamp_utc: null
symbol: null
market_type: null
setup_id: null
direction: null
status: "UNKNOWN" # PASS | REJECT | UNKNOWN
timeframes:
  macro: ["4H", "1H"]
  setup: ["15m"]
  execution: ["5m", "1m"]
market_bias: null
entry:
  type: null
  price: null
stop_loss:
  price: null
  rationale: null
take_profit:
  - price: null
    rationale: null
risk:
  risk_capital: null
  risk_percent: null
  risk_amount: null
  estimated_costs: null
rr_after_costs: null
conditions:
  - name: null
    result: "UNKNOWN" # PASS | FAIL | UNKNOWN
    evidence: null
data_quality:
  source: null
  last_closed_candle_time: null
  missing_fields: []
rejection_reasons: []
notes: []
execution_mode: "paper"
```

Không được điền giá, indicator, volume hoặc bằng chứng bằng suy đoán. Dùng `null`/`UNKNOWN` khi không có dữ liệu.

## 8. Những điểm cần kiểm định trước khi dùng thực tế

Tài liệu nguồn đưa ra các quy tắc và ngưỡng nhưng không kèm bằng chứng backtest trong file này. Trước khi coi chiến lược là sẵn sàng vận hành, cần:

1. Chuyển mọi thuật ngữ định tính thành quy tắc có thể kiểm thử, gồm BOS, OB, sweep, CISD, FVG/IFVG, volume spike và session.
2. Xác định chính xác symbol, sàn, timeframe, timezone, phí, spread/slippage, funding và quy tắc khớp lệnh.
3. Backtest không lookahead; tính chi phí thực tế; xử lý lệnh limit không khớp và nhiều trigger cùng nến.
4. Dùng out-of-sample / walk-forward và kiểm tra nhiều chế độ thị trường, không chỉ tối ưu trên một giai đoạn.
5. Báo cáo tối thiểu: số giao dịch, expectancy, profit factor, win rate, max drawdown, R-multiple distribution, chi phí và độ ổn định theo tài sản/timeframe.
6. Không tuyên bố “tỷ lệ thắng cao nhất”, “Smart Money xác nhận” hoặc “có lợi nhuận” nếu chưa có số liệu kiểm chứng.
7. Giữ nguyên chế độ paper trading cho đến khi có quy trình phê duyệt riêng cho live trading.

## 9. Giới hạn và các điểm chưa được định nghĩa trong nguồn

- Bốn setup được mô tả chủ yếu cho chiều BUY; điều kiện SELL đầy đủ chưa có trong tài liệu nguồn.
- Một số khái niệm chưa có ngưỡng định lượng đủ để code trực tiếp, gồm “volume spike”, “thân nến mạnh”, “ngay lập tức”, “nhịp đẩy mạnh” và các biên vùng OB.
- Setup 4 chưa nêu ngưỡng R:R riêng.
- Buffer `2 pips` không dùng chung được cho mọi loại tài sản.
- Chưa có dữ liệu hiệu suất, win rate, expectancy hoặc kết quả backtest trong tài liệu nguồn.
- Các khái niệm SMC/Order Flow là khung diễn giải; cần định nghĩa nhất quán và kiểm định định lượng trước khi dùng làm điều kiện tự động.

---
**Kết thúc tài liệu.** Đây là tài liệu kiến thức và đặc tả tham chiếu; không phải cam kết lợi nhuận hay xác nhận chiến lược đã được kiểm chứng.
