# 💸 Fintech Domain — Business Deep Dive

> Tài liệu này đi thật sâu vào **business** của 3 domain cốt lõi trong Fintech: Wallet & Balance System, Reconciliation, và Lending/BNPL. Mỗi phần giải thích tại sao nó tồn tại, nó vận hành như thế nào trong thực tế, ai kiếm tiền từ đâu, và những vấn đề business thực sự mà các công ty đang đối mặt.

---

## PHẦN 1 — BỨC TRANH LỚN: FINTECH HOẠT ĐỘNG NHƯ THẾ NÀO?

### 1.1 Tại sao Fintech tồn tại?

Ngân hàng truyền thống tồn tại hàng trăm năm nhưng có những vấn đề mà họ không thể hoặc không muốn giải quyết:

```
VẤN ĐỀ CỦA NGÂN HÀNG TRUYỀN THỐNG:

Tốc độ:
  Mở tài khoản → 3-5 ngày làm việc
  Chuyển khoản quốc tế → 3-7 ngày
  Vay tiêu dùng nhỏ → 1-2 tuần thẩm định

Chi phí:
  Phí chuyển tiền nội địa: vài chục nghìn đồng
  Phí chuyển tiền quốc tế: 2-5% giá trị
  Chi phí vận hành chi nhánh: cực kỳ cao

Tiếp cận:
  Cần có tài sản thế chấp để vay
  Phải đến chi nhánh để làm thủ tục
  Người thu nhập thấp, không có lịch sử tín dụng → bị loại

Fintech giải quyết những vấn đề này bằng công nghệ:
  → Mở tài khoản trong 5 phút qua app
  → Chuyển tiền tức thì với phí gần bằng 0
  → Vay tiền trong vài phút với alternative data
```

### 1.2 Ba trụ cột của Fintech

```
┌─────────────────────────────────────────────────────────┐
│                                                          │
│   MOVE MONEY          STORE MONEY         GROW MONEY    │
│   (Di chuyển tiền)    (Lưu trữ tiền)      (Sinh lời)   │
│                                                          │
│   Payment             Wallet/E-money       Lending      │
│   Transfer            Deposit              Investing    │
│   Remittance          Savings              Insurance    │
│                                                          │
│        ↕                  ↕                    ↕        │
│              RECONCILIATION (Đối soát)                  │
│         Đảm bảo mọi thứ chính xác và khớp nhau         │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

Reconciliation không phải một sản phẩm — nó là **hệ thống thần kinh nền tảng** đảm bảo Move Money và Store Money hoạt động đúng.

---

## PHẦN 2 — WALLET & BALANCE SYSTEM

### 2.1 Ví điện tử kiếm tiền từ đâu?

Đây là câu hỏi đầu tiên bất kỳ BA nào cũng cần hiểu. Tưởng đơn giản nhưng thực ra khá phức tạp.

**Vấn đề cốt lõi:** Chuyển tiền P2P, thanh toán QR — margin rất mỏng hoặc bằng 0. Vậy tại sao các công ty ví lại đốt hàng trăm triệu USD để có thêm user?

Câu trả lời: **Ví chỉ là cửa ngõ — tiền thật đến từ các dịch vụ phía sau.**

```
REVENUE STREAMS CỦA VÍ ĐIỆN TỬ:

1. TRANSACTION FEES (Phí giao dịch) — Thấp, cạnh tranh cao
   MDR từ merchant:        0.1 – 1.5% mỗi giao dịch
   Phí rút tiền:          Thường miễn phí hoặc rất thấp
   Phí chuyển tiền:       Ngày càng về 0 do cạnh tranh
   → Margin rất mỏng, cần volume cực lớn mới có lãi

2. FLOAT INCOME (Lãi từ tiền người dùng gửi vào)
   Toàn bộ số dư của user được gom vào tài khoản ngân hàng
   → Gửi kỳ hạn hoặc đầu tư ngắn hạn
   → Lãi suất ~5-7%/năm trên tổng float

   Ví dụ MoMo: 31 triệu user, trung bình 100,000đ/người
   → Float ~ 3,100 tỷ đồng
   → Lãi 6%/năm → ~186 tỷ đồng/năm từ float alone

   Lưu ý: NHNN quy định tiền người dùng phải được bảo quản
   ở tài khoản phong tỏa riêng, không được dùng tự do

3. VALUE-ADDED SERVICES (Dịch vụ giá trị gia tăng) — Đây mới là core
   Lending (cho vay):      NIM 8-15%, margin cực cao
   Insurance:              Commission 10-20% phí bảo hiểm
   Investment:             Management fee 0.5-1%/năm AUM
   Bill payment:           Phí tiện ích, hoa hồng từ nhà cung cấp
   Voucher/Deals:          Commission từ merchant promotion
   Remittance:             Spread FX + fixed fee

4. ADVERTISING & DATA
   Hiển thị quảng cáo targeted cho 30+ triệu user
   Bán insights (ẩn danh) cho merchant, brand
   White-label analytics cho đối tác

5. B2B/API SERVICES
   Cung cấp payment infrastructure cho doanh nghiệp khác
   White-label wallet cho brand
```

**Case study thực tế — MoMo:**

MoMo đạt lợi nhuận lần đầu tiên vào năm 2024 sau khi xử lý 5.5 tỷ giao dịch chỉ trong Q1/2025, xác nhận rằng mô hình super app cuối cùng cũng có thể profitable — nhưng phải mất nhiều năm.

Theo Kapronasia, dù ví điện tử ở các nước đang phát triển là mảng kinh doanh margin thấp, MoMo đã đang xây dựng hệ sinh thái để cung cấp các dịch vụ margin cao hơn — ví hiện là đối tác của hơn 50 ngân hàng và công ty tài chính, bảo hiểm. Đây chính là chiến lược: dùng ví để thu thập data và tạo quan hệ, rồi monetize qua lending và insurance.

**Tại sao các ví vẫn thua lỗ nhiều năm liền?**

Báo cáo tài chính cho thấy ZaloPay ghi nhận thua lỗ hơn 29.1 tỷ đồng trong năm 2023, dù đã giảm gần một nửa so với 2022. MoMo đạt doanh thu hơn 392 triệu USD năm 2023 nhưng vẫn lỗ sau thuế hơn 8.3 tỷ đồng. Trong khi đó VNPay và Payoo là những ví hiếm hoi đạt tăng trưởng dương cả doanh thu lẫn lợi nhuận.

Lý do thua lỗ không phải vì mô hình tệ — mà vì đang trong giai đoạn **burn money để giành thị phần**:
```
Cashback/voucher cho user mới:        Chi phí CAC cao
Trợ giá phí giao dịch về 0:          Mất MDR
Miễn phí rút tiền:                   Tốn phí ngân hàng
Marketing massive:                    Brand awareness

Logic: Giành user trước → Monetize sau
→ Giống Grab, Shopee đã làm trước đó
```

### 2.2 Wallet User Journey — Vòng đời người dùng ví

Hiểu user journey giúp hiểu tại sao mỗi tính năng tồn tại:

```
[ACQUISITION — Thu hút user mới]
  Cashback 20% lần đầu dùng
  Tặng điểm khi giới thiệu bạn
  Tích hợp sẵn vào Grab, Shopee, các app phổ biến
  → Chi phí CAC: 50,000 – 200,000đ/user tùy thị trường

[ACTIVATION — User dùng lần đầu]
  Đăng ký → Liên kết tài khoản ngân hàng → Nạp tiền → Thanh toán
  Friction phải cực thấp: toàn bộ < 5 phút
  eKYC: chụp CCCD + selfie → Auto verify

[ENGAGEMENT — Dùng thường xuyên]
  Daily habit loop:
    Trả cà phê → Nhận cashback → Thấy deal → Mua thêm
  Push notification: "Deal hôm nay: -30% tại Highlands"
  Gamification: Vòng quay may mắn, streak rewards
  → Mục tiêu: User mở app ít nhất 3-5 lần/tuần

[MONETIZATION — Kiếm tiền từ user]
  Khi user đã tin tưởng và dùng thường xuyên:
  → Offer vay tiền ("Bạn được vay đến 5 triệu, lãi 0% tháng đầu")
  → Bán bảo hiểm ("Bảo hiểm tai nạn chỉ 15,000đ/tháng")
  → Mời đầu tư ("Gửi tiết kiệm lãi 7.5%/năm")
  → Bán vé, deal ("Flash sale Vietjet -50%")

[RETENTION — Giữ chân user]
  Loyalty points: tích lũy, đổi quà
  Tier system: Silver → Gold → Platinum
  Switching cost: điểm tích lũy mất nếu rời bỏ
  Ecosystem lock-in: bill payment history, saved payees
```

### 2.3 Tại sao Balance System phức tạp hơn người nghĩ?

Nhiều người nghĩ balance chỉ là một con số. Thực tế không phải vậy.

**Vấn đề 1: Tiền "đang trên đường"**

```
User A chuyển 500,000đ cho User B lúc 11:59 PM ngày 31/12:
  → Ngày 31/12: Trừ 500,000đ từ A
  → Ngày 01/01 (ngày hôm sau): Cộng 500,000đ cho B

Trong khoảng thời gian này, tiền ở đâu?
→ Nằm trong "Suspense Account" / "In-transit Account"
→ Phải track chính xác, không được mất

Nếu hệ thống crash lúc 12:01 AM:
→ A đã bị trừ tiền
→ B chưa nhận được
→ Tiền biến mất nếu không xử lý đúng
```

**Vấn đề 2: Một user có thể có nhiều loại balance**

```
User có tổng 1,000,000đ trong ví, nhưng:

Available balance:    600,000đ  (có thể dùng ngay)
Reserved balance:     200,000đ  (đang giữ cho order đang pending)
Pending balance:      150,000đ  (chuyển khoản chưa settle)
Cashback pending:      50,000đ  (cashback chưa confirm)

Tổng:               1,000,000đ

Nếu user cố thanh toán 800,000đ → Hệ thống phải từ chối
vì Available chỉ có 600,000đ — dù Total là 1,000,000đ
```

**Vấn đề 3: Negative balance — Khi nào được phép?**

```
Prepaid wallet: KHÔNG được phép negative
  → Mọi transaction phải check available balance trước

Credit wallet / Overdraft: Được phép negative đến giới hạn
  → Hạn mức tín dụng là một dạng lending tích hợp vào ví

BNPL wallet:
  → User "mua trước" → Balance âm ngay → Trả dần sau
  → Phức tạp hơn vì phải track lịch trả và tính lãi
```

**Vấn đề 4: Concurrency — Hai giao dịch đồng thời**

```
Tình huống thực tế:
  User nhấn "Thanh toán 500k" trên app
  Đồng thời có một autopay bill 500k đang chạy
  User chỉ có 600k trong ví

Nếu xử lý không đúng:
  → Cả hai đều check balance: thấy 600k > 500k → OK
  → Cả hai đều trừ 500k
  → Balance = 600k - 500k - 500k = -400k
  → Ví bị âm 400k → CÔNG TY LỖ TIỀN THẬT

Đây không phải lý thuyết — đây là lỗi thực sự xảy ra
ở nhiều hệ thống payment nếu không xử lý concurrency đúng
```

### 2.4 Float Management — Bài toán tiền tỷ

Khi bạn build một ví có 1 triệu user, mỗi user giữ trung bình 500,000đ:

```
Tổng float = 500 tỷ đồng

Đây là tiền THẬT, đang nằm trong tài khoản ngân hàng của công ty.
Nếu công ty làm sai, 500 tỷ của user có thể mất.
```

**Bài học từ vụ sụp đổ của Synapse (2024):**

Ngày 7/6/2024, hơn 100,000 khách hàng mất quyền truy cập vào tiền tiết kiệm của họ. Tiền không biến mất — nó bị chôn vùi trong mê cung các giao dịch chưa được đối soát, tiền bị thiếu hụt, và các điểm mù tài chính. Thủ phạm là Synapse, một fintech trung gian đã sụp đổ, để lại khoản thiếu hụt 85 triệu USD giữa số tiền ngân hàng đang giữ và số tiền khách hàng được nợ.

Đây là minh chứng rõ ràng nhất: **Float management và Reconciliation không phải vấn đề technical — đây là vấn đề sống còn của business.**

**Nguyên tắc Float Management:**

```
QUY TẮC BẤT DI BẤT DỊCH:

1. Segregation (Tách biệt hoàn toàn):
   Tiền của user ≠ Tiền vận hành của công ty
   Nếu công ty phá sản, tiền user vẫn phải được bảo vệ
   → NHNN yêu cầu: Tài khoản phong tỏa riêng tại ngân hàng uy tín

2. 1:1 Coverage (Đảm bảo 1:1):
   Tổng số dư tất cả ví user = Số tiền thực trong Float Account
   Chênh lệch dù 1 đồng cũng là vấn đề nghiêm trọng

3. Daily Reconciliation (Đối soát hàng ngày):
   Mỗi ngày phải verify: Internal records = Bank statement
   Phát hiện chênh lệch trong ngày, không để tích tụ

4. Liquidity Management (Quản lý thanh khoản):
   Đủ tiền mặt để xử lý rút tiền của user bất cứ lúc nào
   Tránh "bank run" scenario: nhiều user rút cùng lúc
```

### 2.5 Super App Strategy — Tại sao MoMo, ZaloPay đều muốn trở thành Super App?

Thị trường mobile payment Việt Nam được định giá khoảng 40.5 tỷ USD vào năm 2026. MoMo có hơn 31 triệu người dùng và 140,000 điểm thanh toán, phát triển từ một ví điện tử thành một super app được hỗ trợ bởi AI cung cấp các dịch vụ từ thanh toán hóa đơn đến giao dịch chứng khoán.

**Logic của Super App Strategy:**

```
BÀI TOÁN KINH TẾ:

Chi phí thu hút 1 user mới (CAC):          150,000đ
Lifetime Value (LTV) của user chỉ dùng ví: 50,000đ/năm × 3 năm = 150,000đ
→ Break even chỉ, không có lãi

LTV khi user dùng full ecosystem:
  Payment fee:        30,000đ/năm
  Lending interest:  150,000đ/năm  (vay 2tr × 15% × 0.5 năm)
  Insurance:          20,000đd/năm (commission)
  Investment:         15,000đ/năm  (fee on 1tr AUM)
  Deals/voucher:      10,000đ/năm  (commission)
  ─────────────────────────────────────────
  LTV tổng:          225,000đ/năm × 5 năm = 1,125,000đ
→ LTV/CAC = 7.5x → Rất tốt

KẾT LUẬN: Ví là loss leader để dẫn dắt user vào ecosystem
          Profit thực đến từ lending và value-added services
```

**ZaloPay vs MoMo — Hai chiến lược khác nhau:**

```
ZALOPAY — WeChat Pay Model:
  Leveraging Zalo (100 triệu users) → Chi phí CAC gần bằng 0
  User của Zalo → Thêm tính năng payment vào app quen dùng
  Giống WeChat Pay: messaging + payment trong 1 app
  Strength: Distribution cost thấp nhất thị trường

MOMO — Standalone Super App:
  Build từ đầu, không có social network backing
  Phải đầu tư mạnh vào marketing và cashback
  Compensate bằng: product tốt hơn, ecosystem rộng hơn
  Strength: Payment-first focus, deeper fintech integration
```

---

## PHẦN 3 — RECONCILIATION: XƯƠNG SỐNG CỦA FINTECH

### 3.1 Tại sao Reconciliation quan trọng đến vậy?

Hầu hết mọi người nghĩ Reconciliation là công việc back-office nhàm chán của kế toán. Thực tế:

> **Reconciliation là lý do duy nhất khiến bạn có thể tin tưởng con số trong hệ thống của mình.**

**Bối cảnh thực tế:**

Những gì trông giống một giao dịch khách hàng đơn lẻ, thực tế bị phân mảnh qua nhiều nền tảng ghi nhận các giai đoạn khác nhau của cùng một giao dịch. Các file settlement bị delay và thường gom nhiều giao dịch thành một khoản thanh toán duy nhất; ngân hàng hiển thị số tiền ròng mà không có chi tiết từng giao dịch; và ngành payments toàn cầu tiếp tục phân mảnh khi các rails và ecosystem mới xuất hiện.

Reconciliation accounts for 30%-40% of a company's back-office labor costs. Và mặc dù các lãnh đạo tài chính đồng ý rằng automated reconciliation là crucial, nhiều công ty vẫn tiếp tục dựa vào các phương pháp thủ công hoặc batch.

**Tại sao một giao dịch đơn giản lại gây ra mớ hỗn độn:**

```
Khách hàng mua áo 300,000đ bằng thẻ Visa lúc 23:50 ngày 31/12:

HỆ THỐNG MERCHANT ghi nhận: 300,000đ doanh thu, ngày 31/12
PAYMENT GATEWAY ghi nhận:   $12.90 (tỷ giá lúc đó), ngày 31/12
ACQUIRER BANK ghi nhận:     Bank processes overnight → ngày 01/01
CARD NETWORK (Visa):        Settlement T+2 → ngày 02/01
BANK STATEMENT merchant:    Nhận tiền (sau phí MDR) ngày 03/01

5 hệ thống, 5 ngày khác nhau, 2 loại tiền tệ, các amount khác nhau.
Tất cả đều "đúng" theo góc nhìn của riêng hệ thống đó.
→ Ai đúng? Reconciliation trả lời câu hỏi này.
```

### 3.2 Các loại chênh lệch thực tế

Không phải mọi chênh lệch đều nghiêm trọng như nhau. Phân loại đúng để xử lý đúng:

**Loại 1: Timing Difference (Chênh lệch thời gian) — Phổ biến nhất, ít nguy hiểm nhất**

```
Nguyên nhân:
  Giao dịch 23:59 ngày T → Gateway ghi T, Bank ghi T+1
  Timezone: Hệ thống UTC+0 vs hệ thống UTC+7
  Settlement batch: Gom nhiều ngày → Một lần chuyển khoản

Cách xử lý:
  Không cần lo nếu resolved trong 1-3 ngày
  Hệ thống tự match khi settlement về
  Alert nếu quá 5 ngày vẫn chưa match
```

**Loại 2: Fee Discrepancy (Chênh lệch phí) — Thường xuyên, cần theo dõi**

```
Nguyên nhân:
  MDR tính theo flat rate vs tiered rate
  FX conversion spread thay đổi theo thời điểm
  Phí ẩn: scheme fee, cross-border fee, international fee
  Gateway charge sai rate so với hợp đồng

Ví dụ:
  Expected MDR: 1.5% × 1,000,000đ = 15,000đ
  Actual charged: 17,500đ
  Chênh lệch: 2,500đ → Vẻ nhỏ, nhưng × 10,000 giao dịch/ngày
  = 25,000,000đ/ngày chênh lệch → BIG DEAL

Cách xử lý:
  Reconcile fee schedule định kỳ với gateway
  Đặt tolerance level (ví dụ: chấp nhận ±0.01%)
  Flag và investigate nếu vượt tolerance
```

**Loại 3: Missing Transaction — Nguy hiểm**

```
Scenario A: Có trong Internal, không có ở External
  → Internal ghi nhận thành công nhưng gateway không có
  → Khách hàng bị trừ tiền nhưng merchant không nhận được?
  → Tiền đi đâu?

Scenario B: Có ở External, không có trong Internal
  → Gateway ghi nhận và settlement đã về
  → Internal không hề biết giao dịch này tồn tại
  → Ai nhận tiền này?

Scenario C: "Ghost transaction" (Giao dịch ma)
  → Có tiền về từ bank nhưng không khớp với giao dịch nào
  → Có thể là: nhầm lẫn từ bank, hoặc gian lận từ bên trong

Tất cả 3 scenarios đều phải escalate ngay — không được để qua ngày
```

**Loại 4: Duplicate Transaction — Rất nguy hiểm**

```
Nguyên nhân phổ biến:
  Network timeout → Client retry → Hai giao dịch cùng được xử lý
  Bug trong hệ thống → Double charge
  Manual error → Nhập tay hai lần

Hậu quả:
  User bị charge 2 lần → Khiếu nại → Chargeback
  Merchant nhận 2 lần nhưng settlement chỉ trả 1 lần → Mất tiền
  Audit trail sai → Báo cáo tài chính không tin cậy

Phòng ngừa:
  Idempotency key: Mỗi giao dịch có unique key,
  gọi nhiều lần chỉ process 1 lần
```

### 3.3 Settlement Reconciliation — Luồng tiền thực tế

Đây là loại reconciliation phức tạp và quan trọng nhất trong Payment Fintech:

**Vấn đề cơ bản:**

```
Bạn là Payment Provider. Hôm qua có 5,000 giao dịch:
  Tổng amount charged:  2,500,000,000đ
  MDR collected (1.5%):    37,500,000đ
  Expected settlement:  2,462,500,000đ (trả cho merchant)

Hôm nay ngân hàng chuyển về: 2,461,850,000đ

Chênh lệch: 650,000đ — ở đâu ra?

Bạn PHẢI tìm được:
  Phí chuyển khoản ngân hàng: 550,000đ (10 lệnh × 55,000đ)
  2 giao dịch refund phát sinh tối qua: 100,000đ
  → Giải thích được 650,000đ → BALANCED ✓

Nếu không giải thích được → Vấn đề nghiêm trọng
```

**Luồng Settlement thực tế của một Payment Provider:**

```
NGÀY T (Giao dịch xảy ra):
  ┌─────────────────────────────────────────────────────┐
  │ 08:00 - 22:00: Giao dịch diễn ra                   │
  │   Mỗi giao dịch: Ghi nhận internal + notify gateway│
  │   Gateway: Authorize & capture → Hold tiền          │
  └─────────────────────────────────────────────────────┘
                           ↓
NGÀY T+1 (Settlement processing):
  ┌─────────────────────────────────────────────────────┐
  │ 00:00 - 06:00: Batch close                         │
  │   Gateway gom tất cả giao dịch ngày T               │
  │   Tính net settlement (gross - fees - refunds)      │
  │   Gửi settlement file (CSV/SFTP) cho chúng ta       │
  │                                                      │
  │ 06:00 - 10:00: Reconciliation team làm việc        │
  │   Download settlement file từ gateway               │
  │   Download bank statement                           │
  │   Match từng giao dịch                              │
  │   Flag các exceptions                               │
  │                                                      │
  │ 10:00 - 14:00: Investigate exceptions              │
  │   Contact gateway nếu cần                          │
  │   Escalate các trường hợp nghiêm trọng              │
  │                                                      │
  │ 14:00: Settlement release                          │
  │   Approve các merchant settlement đã reconcile      │
  │   Initiate bank transfer cho merchant               │
  └─────────────────────────────────────────────────────┘
                           ↓
NGÀY T+2 (Merchant nhận tiền):
  ┌─────────────────────────────────────────────────────┐
  │ Merchant nhận tiền vào tài khoản ngân hàng          │
  │ Merchant tự reconcile: settlement nhận = expected?  │
  └─────────────────────────────────────────────────────┘
```

### 3.4 Multi-source Reconciliation — Khi có nhiều gateway

Thực tế, một công ty Fintech thường làm việc với nhiều payment partner cùng lúc:

```
Một ngày giao dịch điển hình:

VNPay Gateway:     1,200 giao dịch, 450,000,000đ
Momo:                800 giao dịch, 180,000,000đ
ZaloPay:             600 giao dịch, 120,000,000đ
Thẻ Visa/Master:     400 giao dịch, 380,000,000đ
Chuyển khoản ngân hàng: 200 giao dịch, 120,000,000đ
─────────────────────────────────────────────────────
Total:             3,200 giao dịch, 1,250,000,000đ

Vấn đề:
  Mỗi gateway có settlement time khác nhau (T+1, T+2, weekly)
  Mỗi gateway có format file khác nhau
  Mỗi gateway có reference ID format khác nhau
  Mỗi gateway có fee structure khác nhau

→ Reconciliation team phải master tất cả các format này
→ Đây là lý do automation và standardization quan trọng
```

**Exception Categories và SLA xử lý:**

```
PRIORITY 1 — URGENT (Xử lý trong 2 giờ):
  Missing funds: Tiền về ngân hàng nhưng không match được giao dịch
  Duplicate charges: User bị charge 2 lần
  Amount mismatch > 1,000,000đ: Chênh lệch lớn bất thường
  → Nguy cơ fraud hoặc system error nghiêm trọng

PRIORITY 2 — HIGH (Xử lý trong ngày):
  Missing transactions: Có ở internal không có ở gateway (hoặc ngược lại)
  Amount mismatch 100,000 – 1,000,000đ
  Settlement delay quá T+3
  → Cần investigate nhưng không nhất thiết là fraud

PRIORITY 3 — NORMAL (Xử lý trong 3 ngày):
  Fee discrepancy nhỏ: < 100,000đ
  Timing difference chưa resolve sau 3 ngày
  Format inconsistency không ảnh hưởng amount
  → Thường có giải thích hợp lý, verify và đóng

PRIORITY 4 — LOW (Xử lý trong tuần):
  Rounding differences: < 1,000đ
  Known timing patterns: Cuối tuần/lễ
  → Đánh dấu "accepted variance", không cần action
```

### 3.5 Reconciliation Failure — Khi mọi thứ sai

**Case study: Synapse collapse (2024)**

Synapse Financial Technologies đã sử dụng phần mềm để theo dõi payments bởi các fintech companies và phân bổ payments một cách phù hợp. Tuy nhiên, sự thất bại của Synapse trong việc thực hiện điều này đúng cách đã dẫn đến khoản thiếu hụt lên đến 95 triệu USD giữa số tiền các ngân hàng đang giữ và số tiền nợ người dùng fintech. Khi Synapse phá sản vào tháng 4/2024, các ngân hàng đối tác không thể xác định số tiền nào đang được giữ thuộc về tổ chức nào.

Đây là hậu quả khi reconciliation bị bỏ qua trong nhiều năm:

```
Synapse là fintech middleware — kết nối:
  Fintech apps (Yotta, Juno, etc.) ←→ Synapse ←→ Partner Banks

Mô hình:
  Fintech app nhận tiền của user
  → Gửi cho Synapse giữ hộ
  → Synapse deposit vào nhiều ngân hàng partner khác nhau
  → Synapse track sổ sách "ai có bao nhiêu tiền ở ngân hàng nào"

Vấn đề:
  Synapse track sổ sách sai — nhưng không ai biết
  Fintech apps trust Synapse
  Ngân hàng partner trust Synapse
  Không có independent reconciliation nào verify số liệu của Synapse

Khi Synapse sụp đổ:
  Sổ sách của Synapse: User A có 10,000$
  Thực tế tại ngân hàng: Chỉ có 9,200$ liên quan đến User A
  800$ đi đâu? Không ai biết.
  × 100,000 users → 85 triệu USD "mất tích"
```

**Bài học business:**

```
1. Trust nhưng verify — dù là đối tác lớn
2. Independent reconciliation là bắt buộc, không phải optional
3. Daily reconciliation, không phải monthly
4. Audit trail phải immutable — không ai được sửa sau khi ghi
5. Regulatory oversight là bảo vệ, không phải gánh nặng
```

---

## PHẦN 4 — LENDING & BNPL: BUSINESS MODEL THẬT SỰ

### 4.1 Tại sao Lending là thánh địa của Fintech?

```
SO SÁNH MARGIN CÁC DỊCH VỤ FINTECH:

Payment (MDR):            1.5% một lần, mỗi giao dịch
→ User thanh toán 1tr → Thu 15,000đ → Không cộng dồn

Wallet float income:      6-7%/năm trên số dư
→ User giữ 1tr trong ví → Thu 60,000-70,000đ/năm

Consumer lending:         15-40%/năm Net Interest Margin
→ User vay 1tr trong 1 năm → Thu 150,000-400,000đ

→ Lending profitable hơn payment 10-20 lần
→ Đây là lý do mọi payment company đều muốn làm lending
```

**Nguồn vốn cho vay đến từ đâu?**

```
Model 1 — Balance Sheet Lending:
  Dùng tiền của công ty để cho vay
  Risk: Công ty chịu toàn bộ rủi ro tín dụng
  Reward: Giữ 100% interest income
  Ví dụ: Apple Pay Later (dùng tiền của Apple)

Model 2 — Funding Partner:
  Hợp tác với ngân hàng/quỹ
  Fintech: Source loan, underwrite, service
  Bank: Cung cấp vốn
  Revenue split: Fintech giữ servicing fee, share interest với bank
  Ví dụ: MoMo + TPBank cho Laterpay Wallet
         Shopee + Sea Money

Model 3 — Securitization:
  Gom nhiều khoản vay thành pool
  Bán pool này cho nhà đầu tư dưới dạng ABS (Asset-Backed Securities)
  Fintech nhận vốn ngay, nhà đầu tư nhận lãi từ pool
  Ví dụ: Affirm, SoFi (US market)

Model 4 — P2P Lending:
  Kết nối người vay và người cho vay trực tiếp
  Fintech: Platform fee + servicing fee
  Không cần vốn lớn, nhưng risk management phức tạp
  Việt Nam: Chưa có khung pháp lý chính thức
```

### 4.2 BNPL — Tại sao nó bùng nổ?

Khối lượng BNPL toàn cầu là 50 tỷ USD năm 2019 và tăng gấp 10 lần lên 500 tỷ USD năm 2024. BNPL kết hợp việc bán một sản phẩm với một khoản vay tiêu dùng. Nó bao gồm ba tính năng được tạo ra bởi tiến bộ công nghệ: cải thiện trải nghiệm người dùng, quy trình cấp vốn nhanh hơn, và khả năng sử dụng dữ liệu số để chấm điểm khách hàng mà không cần điền form phức tạp.

**Ba bên trong BNPL và ai được lợi gì:**

```
CONSUMER (Người mua):
  ✓ Mua hàng giá cao ngay, trả dần
  ✓ Lãi 0% nếu trả đúng hạn (pay-in-4 model)
  ✓ Không cần thẻ tín dụng
  ✓ Approval trong giây, không paperwork
  ✗ Bẫy nợ nếu dùng nhiều BNPL cùng lúc

MERCHANT (Người bán):
  ✓ Conversion rate tăng 20-30%
     "Khách hàng dám mua item 2tr vì biết chỉ trả 500k trước"
  ✓ AOV (Average Order Value) tăng 40-60%
  ✓ BNPL provider chịu rủi ro tín dụng, không phải merchant
  ✓ Merchant nhận 100% tiền ngay từ BNPL provider
  ✗ MDR cao hơn: 3-8% thay vì 1.5% của thẻ

BNPL PROVIDER:
  Revenue đến từ đâu?
  → Merchant fee (3-8%): Nguồn thu chính
  → Late fee: Nhỏ nhưng high-margin
  → Interest (installment có lãi): 15-30%/năm

  <cite>BNPL providers chủ yếu monetize qua merchant fees, không phải lãi hay phí phạt</cite>
  <cite>Chỉ 4.1% khoản vay BNPL bị tính late fee, và 1.83% không thu hồi được — nghĩa là 98% được trả đầy đủ</cite>
```

**Tại sao BNPL lấy được MDR cao hơn thẻ từ merchant?**

```
BNPL pitch cho merchant:
  "Thẻ tín dụng MDR 2% → Chỉ làm khách hàng thanh toán dễ hơn"
  "BNPL MDR 5% → Tăng conversion 30%, tăng AOV 50%"

  Ví dụ cho merchant:
    100 khách có ý định mua, item 2,000,000đ
    Không có BNPL: 60 người mua → Revenue 120,000,000đ
    Có BNPL: 80 người mua (thêm 20 người dám mua vì pay later)
             → Revenue 160,000,000đ

    Tăng revenue: 40,000,000đ
    Chi phí BNPL (5%): 8,000,000đ (tính trên 160tr)
    Chi phí thẻ (2%): 2,400,000đ (tính trên 120tr)
    Chi phí tăng thêm: 5,600,000đ
    Net gain từ BNPL: 40,000,000 - 5,600,000 = 34,400,000đ ✓

→ Merchant chấp nhận MDR cao hơn vì ROI rõ ràng
```

### 4.3 Credit Risk — Bài toán cốt lõi của Lending

**Tại sao traditional bank không làm được điều Fintech làm?**

```
Truyền thống:
  Thẩm định khoản vay 2tr → Mất 2 tuần, 5 cuộc gọi, 10 tờ giấy
  Chi phí thẩm định: ~200,000đ
  → Không profitable với khoản vay nhỏ

Fintech:
  Thẩm định khoản vay 2tr → 30 giây, 0 tờ giấy
  Chi phí thẩm định: ~2,000đ (điện toán đám mây)
  → Profitable vì chi phí cực thấp

Thế mạnh của Fintech chính là DATA:
```

**Alternative Data — Tại sao data của ví điện tử là "vàng" để đánh giá tín dụng:**

```
Ngân hàng truyền thống nhìn vào:
  Lịch sử tín dụng CIC (chỉ ~40% dân số VN có)
  Thu nhập (cần chứng minh bằng giấy tờ)
  Tài sản thế chấp
  → Loại trừ 60% dân số không có lịch sử tín dụng

MoMo/ZaloPay nhìn vào:
  BEHAVIORAL DATA:
    Tần suất thanh toán: "User này dùng ví 15 lần/tháng vs 2 lần/tháng"
    Merchants thường xuyên: "Siêu thị, cà phê, thuốc" vs "bar, casino"
    Time pattern: "Thanh toán đúng giờ, không thanh toán lúc 3am"
    Average transaction size và variance
    
  PAYMENT HISTORY:
    Trả bill điện nước, internet đúng hạn không?
    Trả nợ bạn bè (P2P) có đúng hẹn không?
    Lịch sử topup: Đều đặn hay bất thường?
    
  SOCIAL SIGNALS:
    Bao nhiêu người trong network của user?
    Network của user có ai đang nợ xấu không?
    
  DEVICE DATA:
    Dùng điện thoại mới hay cũ?
    Thay SIM thường xuyên? (Red flag)
    Location pattern có consistent không?

→ Từ tất cả data này → Credit score trong vài giây
→ Score chính xác hơn CIC cho phân khúc thin-file
```

**Risk-based Pricing — Định giá theo rủi ro:**

```
Truyền thống: Tất cả mọi người cùng lãi suất
→ Người rủi ro thấp trợ cấp cho người rủi ro cao

Fintech: Giá theo risk profile
  Score 800-850 (Low risk):   Lãi suất 9%/năm, hạn mức 10tr
  Score 700-800 (Medium):     Lãi suất 15%/năm, hạn mức 5tr
  Score 600-700 (High):       Lãi suất 25%/năm, hạn mức 2tr
  Score <600:                 Từ chối hoặc lãi suất rất cao

Benefits:
  → Người tốt được lãi suất tốt (fairness)
  → Người rủi ro cao vẫn được vay (inclusion)
  → Provider tối ưu được risk/return
```

### 4.4 Loan Lifecycle Business View

**Từ góc độ business, một khoản vay đi qua các giai đoạn:**

**Giai đoạn 1: Origination (Tạo khoản vay)**

```
Mục tiêu business: Approve đúng người, đúng số tiền, đúng lãi suất
Câu hỏi cần trả lời:
  "User này có khả năng trả không?" → DTI, income estimation
  "User này có muốn trả không?" → Character assessment, fraud check
  "Nếu không trả được, chúng ta mất bao nhiêu?" → LGD (Loss Given Default)
  "Xác suất default là bao nhiêu?" → PD (Probability of Default)

KPI quan trọng:
  Approval Rate: % đơn được duyệt / tổng đơn nhận
  Tốt:  60-70% (không quá chặt, không quá lỏng)
  
  Auto-Decision Rate: % đơn được system tự quyết định
  Tốt: > 85% (ít phải manual review)
  
  Application-to-Fund Time: Từ đăng ký đến nhận tiền
  BNPL target: < 30 giây
  Personal loan target: < 24 giờ
```

**Giai đoạn 2: Servicing (Quản lý khoản vay)**

```
Sau khi giải ngân, việc không kém phần quan trọng:

Billing:    Tính toán đúng số tiền cần trả mỗi kỳ
            Bao gồm: Gốc + Lãi + Phí phạt (nếu có)
            
Payment Collection:
            Tự động trừ từ ví / tài khoản ngân hàng đã liên kết
            Reminder trước ngày đến hạn 3 ngày, 1 ngày
            
Account Management:
            Cho phép trả trước (early repayment)
            Cho phép gia hạn (extension) trong một số trường hợp
            Điều chỉnh repayment schedule nếu có hardship claim

KPI quan trọng:
  On-time Payment Rate: % khoản trả đúng hạn
  Tốt: > 90%
  
  Early Repayment Rate: % khách trả trước hạn
  Cao = Bị mất interest income → Cần xem xét prepayment penalty
```

**Giai đoạn 3: Collections (Thu hồi nợ quá hạn)**

```
Đây là giai đoạn tốn kém và nhạy cảm nhất:

DPD 1-10:   Soft reminder — SMS, push notification tự động
            "Khoản vay của bạn đến hạn, vui lòng thanh toán"
            Chi phí: Gần bằng 0 (automated)
            
DPD 11-30:  Escalated reminder — Gọi điện
            "Xin chào, chúng tôi muốn hỗ trợ anh/chị..."
            Có thể đề nghị: Gia hạn thêm 7 ngày, trả tối thiểu
            Chi phí: Cao hơn (human agent)
            
DPD 31-90:  Formal collection
            Thư chính thức, cảnh báo credit score
            Đề nghị restructure (giãn nợ)
            Chi phí: Đáng kể
            
DPD 91-180: Third-party collection agency
            Bán hoặc thuê công ty thu hồi nợ chuyên nghiệp
            Recovery rate: 40-60%
            
DPD > 180:  Write-off
            Xóa khỏi sổ sách (kế toán)
            Vẫn có thể cố gắng thu hồi nhưng không kỳ vọng
            Báo cáo lên CIC → Ảnh hưởng credit history của user

Chú ý: Quy định về đòi nợ rất chặt ở Việt Nam
  KHÔNG được: Quấy rối, đe dọa, liên hệ người thân không liên quan
  → Vi phạm → Phạt nặng + Thiệt hại reputation
```

**Giai đoạn 4: Portfolio Management (Quản lý danh mục)**

```
Nhìn toàn bộ portfolio, không chỉ từng khoản vay:

Vintage Analysis:
  Các khoản vay originate tháng 1/2024 → Default rate ra sao sau 6 tháng?
  So với khoản vay tháng 2/2024?
  → Phát hiện model drift: "Model scoring đang kém hơn so với trước?"

Concentration Risk:
  Không nên 80% portfolio là khoản vay ngành X
  Nếu ngành X gặp khó khăn → Toàn bộ portfolio đều xấu
  → Diversify theo ngành nghề, địa lý, income segment

Yield Management:
  Expected yield vs Actual yield
  "Kỳ vọng thu 15% lãi, thực tế chỉ thu 12% do default cao"
  → Điều chỉnh lãi suất, acceptance criteria
```

### 4.5 Các mô hình BNPL — Không phải chỉ một loại

**Pay-in-4 (Trả 4 lần, không lãi)**

```
Mua 1,200,000đ:
  Hôm nay:     -300,000đ  (25%)
  2 tuần sau:  -300,000đ  (25%)
  4 tuần sau:  -300,000đ  (25%)
  6 tuần sau:  -300,000đ  (25%)

Revenue model của provider:
  Thu từ merchant: 5% × 1,200,000 = 60,000đ
  Chi phí vốn (tự fund 75% trong 6 tuần): ~15,000đ
  Gross margin: 45,000đ per transaction
  
Rủi ro:
  User không trả kỳ 2,3,4 → Provider lỗ 900,000đ
  Chỉ sustainable nếu default rate < 5%
```

**Pay Later 30 days (Trả sau 30 ngày, lãi 0%)**

```
Mua hàng hôm nay, trả toàn bộ sau 30 ngày.
Nếu quá hạn: Lãi phạt 2-3%/tháng.

Đây thực chất là interest-free loan 30 ngày
Revenue đến từ: Merchant fee + Late fee

Use case điển hình:
  Mua hàng cuối tháng khi chưa nhận lương
  Chờ đến ngày lương → Trả luôn
```

**Installment loan có lãi (Trả góp dài hạn)**

```
Mua điện thoại 15,000,000đ, trả 12 tháng, lãi 18%/năm

Đây là personal loan gắn với điểm mua hàng.
Revenue model:
  Interest income 18%/năm = 1,350,000đ lãi + merchant fee

Khác biệt với pay-in-4:
  - Dài hơn (12 tháng vs 6 tuần)
  - Có lãi
  - Cần underwriting kỹ hơn
  - Regulated như consumer lending thông thường
```

### 4.6 BNPL cho B2B — Xu hướng mới

B2B BNPL cho phép người mua doanh nghiệp mua hàng hóa hoặc dịch vụ ngay lập tức trong khi trì hoãn thanh toán trong một khoảng thời gian nhất định — thường là 30, 60, hoặc 90 ngày. Không giống tín dụng thương mại truyền thống, B2B BNPL cung cấp phê duyệt nhanh hơn, điều khoản đơn giản hơn, và khả năng tiếp cận cao hơn. BNPL provider trả cho nhà cung cấp ngay lập tức, đảm bảo dòng tiền ngay và loại bỏ độ trễ thanh toán.

```
Ví dụ thực tế B2B BNPL:

Cửa hàng bán lẻ nhỏ mua hàng từ nhà phân phối:
  Nhà phân phối: "Tôi cần tiền ngay"
  Cửa hàng: "Tôi chưa bán được hàng, chưa có tiền"
  → Vấn đề cổ điển: Cả hai đều kẹt vốn lưu động

B2B BNPL giải quyết:
  BNPL provider trả nhà phân phối ngay 100%
  Cửa hàng được NET-30 hoặc NET-60
  Cửa hàng bán hàng, nhận tiền, rồi trả BNPL provider

Ai kiếm tiền:
  BNPL provider thu phí từ nhà phân phối (2-5%)
  Nhà phân phối chấp nhận vì bán được hàng nhanh hơn
  Cửa hàng có vốn lưu động mà không cần vay ngân hàng
```

---

## PHẦN 5 — HÀNH TRÌNH TỪ VÍ → SUPER APP → EMBEDDED FINANCE

### 5.1 Tại sao mọi Fintech đều đi theo con đường này?

```
GIAI ĐOẠN 1 — Payment:
  "Chúng tôi giúp bạn thanh toán dễ hơn"
  Revenue: Transaction fee
  Problem: Margin quá thấp

GIAI ĐOẠN 2 — Wallet:
  "Chúng tôi giữ tiền của bạn"
  Revenue: Float income + Transaction fee
  Problem: Quy định chặt, phải 1:1 coverage

GIAI ĐOẠN 3 — Super App:
  "Chúng tôi là trung tâm tài chính của bạn"
  Revenue: Lending + Insurance + Investment + ...
  This is where real money is

GIAI ĐOẠN 4 — Embedded Finance:
  "Chúng tôi đưa dịch vụ tài chính vào MỌI nơi"
  Revenue: B2B fee từ non-financial companies
  Future state
```

### 5.2 Data Flywheel — Vòng bánh đà dữ liệu

```
Đây là competitive moat thực sự của Fintech lớn:

User dùng ví nhiều hơn
        ↓
Nhiều data hơn về hành vi
        ↓
Credit scoring chính xác hơn
        ↓
Offer lending phù hợp hơn
        ↓
User vay và trả đúng hạn
        ↓
Default rate thấp
        ↓
Chi phí vốn thấp hơn
        ↓
Lãi suất competitive hơn
        ↓
Thu hút thêm user vay
        ↓ (quay lại đầu)

Kẻ đến sau không thể cạnh tranh vì:
  - Ít data hơn → Credit model kém hơn
  - Model kém hơn → Default rate cao hơn
  - Default rate cao hơn → Chi phí vốn cao hơn
  - Chi phí vốn cao hơn → Lãi suất cao hơn
  - Lãi suất cao hơn → Ít user muốn vay
  → Vòng tròn ngược
```

---

## PHẦN 6 — COMPLIANCE & REGULATION: BUSINESS IMPACT

### 6.1 Tại sao regulation là business issue, không chỉ là legal issue?

```
Ví dụ thực tế tác động của quy định:

NHNN giới hạn số dư ví: 100 triệu đồng/cá nhân
→ Tác động: User dùng ví cho đại trà, không thể thay thế hoàn toàn ngân hàng
→ Strategy implication: Tập trung low-value, high-frequency transactions

NHNN yêu cầu KYC đầy đủ cho giao dịch > 20 triệu:
→ Tác động: Friction cho user lần đầu giao dịch lớn
→ Strategy implication: Cải thiện eKYC experience để giảm drop-off

P2P lending sandbox vẫn chưa có khung pháp lý:
→ Tác động: Không thể scale P2P lending tại VN hợp pháp
→ Strategy implication: Partner với licensed FIs thay vì direct lending
```

### 6.2 Regulatory Arbitrage — Cách Fintech vượt qua rào cản

```
"Nếu không thể làm trực tiếp, hợp tác với người có license"

MoMo + TPBank → Laterpay (BNPL):
  MoMo: Có data, có user, có distribution
  TPBank: Có lending license, có vốn
  → MoMo originate loans, TPBank fund và own the loan
  → Win-win: MoMo kiếm servicing fee, TPBank kiếm interest

Shopee + SeaBank:
  Shopee: Có merchant data (biết ai bán tốt, ai bán xấu)
  SeaBank: Có banking license
  → Shopee cung cấp data để underwrite merchant loans
  → SeaBank làm lending chính thức

Embedded Finance model:
  Non-bank + Licensed bank = Fintech service
  Ai có data → Ai có license → Deal
```

---

## PHẦN 7 — MENTAL MODELS CHO FINTECH BUSINESS

### 7.1 Ba câu hỏi luôn phải hỏi

```
Khi gặp bất kỳ vấn đề nào trong Fintech:

1. "TIỀN ĐANG Ở ĐÂU?"
   → Trace dòng tiền từ điểm A đến điểm B
   → Ai đang "giữ" tiền tại mỗi thời điểm
   → Nếu hệ thống crash lúc này, tiền có bị mất không?

2. "AI CHỊU RỦI RO?"
   → Credit risk: Ai mất tiền nếu user không trả?
   → Fraud risk: Ai mất tiền nếu giao dịch là gian lận?
   → Operational risk: Ai chịu trách nhiệm nếu hệ thống sai?
   → Regulatory risk: Ai bị phạt nếu vi phạm quy định?

3. "AI KIẾM TIỀN TỪ ĐÂU?"
   → Transaction fee? Interest? Float? Commission?
   → Incentive alignment: Ai benefit khi user bị hại?
   → Sustainable không? Hay đang burn cash?
```

### 7.2 Các chỉ số business quan trọng nhất

```
WALLET / PAYMENT:
  GMV (Gross Merchandise Value):   Tổng giá trị giao dịch
  TPV (Total Payment Volume):      Tổng volume payment
  Take Rate:                       Revenue / GMV (thường 0.3-1.5%)
  Active Users (MAU/DAU):          User thực sự dùng, không chỉ install
  Transaction per User:            Engagement level

LENDING:
  AUM / Loan Book:                 Tổng dư nợ đang cho vay
  NPL Ratio:                       % nợ xấu / tổng dư nợ
  Net Interest Margin (NIM):       Lãi vay - Chi phí vốn
  Cost of Risk:                    Chi phí dự phòng / Loan Book
  Risk-Adjusted Return:            NIM - Cost of Risk
  CAC (Customer Acquisition Cost): Chi phí để có 1 borrower mới
  LTV (Lifetime Value):            Tổng thu nhập từ 1 borrower

RECONCILIATION:
  Match Rate:                      % giao dịch match tự động
  Exception Rate:                  % cần manual investigation
  Time-to-Resolve:                 Thời gian giải quyết exception
  False Positive Rate:             % flag nhầm (không thực sự là lỗi)
```

---

## Tổng kết — Bức tranh liên kết

```
┌─────────────────────────────────────────────────────────┐
│                  FINTECH ECOSYSTEM                       │
│                                                          │
│  USER DATA (hành vi, giao dịch, lịch sử)               │
│       ↓              ↓              ↓                   │
│   WALLET          LENDING        INSURANCE              │
│  (Low margin)    (High margin)   (Medium margin)        │
│       ↓              ↓              ↓                   │
│         RECONCILIATION (đảm bảo tất cả chính xác)      │
│                        ↓                                │
│            FINANCIAL REPORTING & COMPLIANCE             │
│                                                          │
│  Competitive moat: DATA FLYWHEEL                        │
│  Nhiều user → Nhiều data → Model tốt hơn →             │
│  Risk thấp hơn → Lãi suất tốt hơn → Nhiều user hơn    │
└─────────────────────────────────────────────────────────┘
```

**Bảng thuật ngữ**

| Thuật ngữ | Ý nghĩa một câu |
|---|---|
| **GMV / TPV** | Tổng giá trị giao dịch chạy qua hệ thống |
| **Take Rate** | % doanh thu trên tổng GMV |
| **Float** | Tiền người dùng gửi vào ví, nằm trong tài khoản ngân hàng của công ty |
| **Float Income** | Lãi ngân hàng kiếm được từ float |
| **NIM** | Net Interest Margin — chênh lệch lãi suất cho vay và lãi suất huy động vốn |
| **Cost of Risk** | Chi phí dự phòng nợ xấu tính theo % loan book |
| **NPL** | Non-Performing Loan — khoản vay quá hạn > 90 ngày |
| **DPD** | Days Past Due — số ngày quá hạn |
| **PD** | Probability of Default — xác suất vỡ nợ |
| **LGD** | Loss Given Default — % mất mát khi vỡ nợ |
| **Alternative Data** | Dữ liệu phi truyền thống để đánh giá tín dụng |
| **Origination** | Giai đoạn tạo khoản vay mới |
| **Vintage Analysis** | Phân tích performance theo thời điểm originate |
| **Settlement** | Chuyển tiền thực tế sau khi giao dịch được settle |
| **Reconciliation** | Đối soát số liệu giữa các hệ thống |
| **Timing Difference** | Chênh lệch do hai hệ thống ghi nhận khác ngày |
| **Exception** | Giao dịch không match được tự động, cần xử lý thủ công |
| **Data Flywheel** | Vòng bánh đà: nhiều user → nhiều data → model tốt hơn → nhiều user hơn |
| **Embedded Finance** | Đưa dịch vụ tài chính vào các sản phẩm phi tài chính |
| **Balance Sheet Lending** | Dùng vốn công ty tự cho vay |
| **Securitization** | Gom các khoản vay thành pool và bán cho nhà đầu tư |