# 🍜 F&B Domain — Business Deep Dive

> Tài liệu này giải thích toàn bộ domain **F&B (Food & Beverage)** từ góc độ business và vận hành: mô hình kinh doanh, unit economics, quản lý menu/nguyên liệu, order flow, POS, và cost control. Dành cho người muốn hiểu thật sự — đủ để phân tích nghiệp vụ và build hệ thống.

---

## 1. Tổng quan ngành F&B

### 1.1 F&B là gì trong ngữ cảnh kinh doanh?

F&B (Food & Beverage) là ngành kinh doanh ẩm thực — bao gồm **sản xuất, chế biến, và phục vụ** thức ăn/đồ uống đến tay người tiêu dùng.

Đặc điểm khiến F&B **khác hoàn toàn** với các ngành khác:

```
Sản phẩm có hạn sử dụng ngắn   → Không thể tồn kho lâu như retail
Nhu cầu theo giờ, theo mùa      → Peak hours, seasonal menu
Chi phí lao động rất cao        → 25-35% doanh thu
Sản xuất và tiêu thụ đồng thời  → Không có warehouse lớn như manufacturing
Trải nghiệm = sản phẩm          → Không gian, thái độ phục vụ cũng là "hàng hóa"
```

### 1.2 Các mô hình F&B phổ biến

```
┌─────────────────────────────────────────────────────────────────┐
│                     CÁC MÔ HÌNH F&B                             │
│                                                                  │
│  QSR          Casual Dining    Fine Dining    Café/Bar           │
│  (Quick       (Nhà hàng        (Cao cấp)      (Cà phê,          │
│  Service)     bình dân)                        đồ uống)          │
│                                                                  │
│  Fast Food    Coffee Chain     Cloud          Bakery /          │
│  Chain        (chuỗi cà phê)   Kitchen        Dessert           │
│                                (bếp ảo)                          │
└─────────────────────────────────────────────────────────────────┘
```

---

## 2. Các mô hình kinh doanh F&B

### 2.1 QSR — Quick Service Restaurant (Chuỗi thức ăn nhanh)

**Ví dụ:** KFC, McDonald's, Jollibee, Lotteria, Gà Rán ABC

**Đặc trưng:**
- Menu cố định, chuẩn hóa cao
- Thời gian phục vụ < 5 phút
- Khách tự lấy đồ hoặc mang đi (takeaway)
- Doanh thu dựa vào **volume** — phục vụ nhiều khách, giá trị mỗi bill thấp

**Unit Economics điển hình:**
```
Average Order Value (AOV):     80,000 – 150,000đ
Số khách/ngày:                 200 – 500 khách
Doanh thu/ngày:                16tr – 75tr
Food Cost:                     28 – 35%
Labor Cost:                    22 – 28%
Rent:                          8 – 12%
```

**Điểm đặc biệt:**
- Standardization là tất cả — mỗi cửa hàng phải ra sản phẩm giống hệt nhau
- Drive-thru và self-order kiosk giúp tăng throughput
- Upsell là nghệ thuật: "Anh có muốn thêm khoai tây không?"

---

### 2.2 Coffee Chain (Chuỗi cà phê)

**Ví dụ:** Highlands, The Coffee House, Phúc Long, Starbucks

**Đặc trưng:**
- Không gian ngồi lâu là selling point chính (khác QSR)
- Đồ uống là core, đồ ăn là upsell
- Khách hàng trung thành cao — loyalty program rất quan trọng
- Doanh thu chia 2 peak: sáng sớm (takeaway) và chiều/tối (dine-in)

**Unit Economics điển hình:**
```
AOV:                           60,000 – 120,000đ
Số lượt khách/ngày:            150 – 400
Doanh thu/ngày:                9tr – 48tr
Beverage Cost:                 20 – 28%  (rất thấp so với food)
Labor Cost:                    25 – 30%
Rent:                          12 – 18%  (mặt bằng đẹp = lợi thế)
```

**Tại sao beverage cost thấp hơn food cost?**
```
1 ly Caramel Latte bán 65,000đ:
  - Espresso shot:   3,000đ
  - Sữa tươi 200ml:  4,000đ
  - Syrup, topping:  2,000đ
  - Ly, ống hút:     2,000đ
─────────────────────────────
  Food cost:        11,000đ = 17%

→ Gross margin beverage rất cao → chuỗi cà phê profitable hơn
   nhà hàng thuần food nếu quản lý tốt
```

---

### 2.3 Casual Dining (Nhà hàng bình dân)

**Ví dụ:** Nhà hàng lẩu, BBQ, hải sản, cơm văn phòng chuỗi

**Đặc trưng:**
- Khách ngồi tại bàn, có nhân viên phục vụ
- Menu đa dạng hơn QSR
- Trải nghiệm ăn uống quan trọng (không chỉ là đồ ăn)
- Doanh thu dựa vào **table turnover** — xoay bàn nhanh = nhiều tiền hơn

**Khái niệm quan trọng — Table Turnover Rate:**
```
Table Turnover Rate = Số lần một bàn được dùng trong một bữa (lunch/dinner)

Ví dụ:
  Nhà hàng 20 bàn × 4 chỗ = 80 ghế
  Turnover rate buổi trưa: 2.5 lần
  → Phục vụ được 80 × 2.5 = 200 khách buổi trưa

  AOV: 120,000đ
  Doanh thu buổi trưa: 200 × 120,000 = 24,000,000đ
```

**Chỉ số quan trọng:**
```
Revenue per Available Seat Hour (RevPASH):
= Doanh thu / (Số ghế × Số giờ hoạt động)

→ Đo lường mức độ khai thác không gian nhà hàng
→ Càng cao = càng hiệu quả
```

---

### 2.4 Fine Dining (Nhà hàng cao cấp)

**Đặc trưng:**
- AOV rất cao (500,000đ – vài triệu/người)
- Volume thấp, nhưng margin cao
- Trải nghiệm là sản phẩm: không gian, ánh sáng, âm nhạc, phục vụ
- Reservation (đặt bàn trước) là bắt buộc
- Thực đơn thay đổi theo mùa (seasonal menu)

**Cơ cấu chi phí:**
```
Food Cost:       25 – 35%  (nguyên liệu cao cấp)
Labor Cost:      35 – 45%  (nhiều nhân viên, tay nghề cao)
Rent:             5 – 10%  (thường ở vị trí đắc địa)
Marketing:        5 – 8%   (PR, review, event)
```

---

### 2.5 Cloud Kitchen / Ghost Kitchen (Bếp ảo)

**Đặc trưng:**
- Không có không gian dine-in — chỉ nấu và giao đi
- Doanh thu 100% từ delivery (GrabFood, ShopeeFood, Baemin)
- Chi phí mặt bằng rất thấp (không cần mặt tiền đẹp)
- Một bếp có thể vận hành nhiều "thương hiệu" khác nhau

**Mô hình đặc biệt — Multi-brand Kitchen:**
```
Một bếp duy nhất, vận hành đồng thời:
  Brand A: Cơm tấm (target: dân văn phòng)
  Brand B: Bún bò (target: buổi sáng)
  Brand C: Salad healthy (target: gym goer)

→ Chi phí bếp/nhân sự chia cho 3 brand
→ Revenue đến từ 3 luồng khác nhau
→ Rủi ro thấp hơn vì diversified
```

**Unit Economics:**
```
Doanh thu:                 Từ app delivery
Commission cho app:        25 – 30% doanh thu (!!)
Food cost:                 30 – 38%
Labor:                     15 – 20%  (ít hơn dine-in vì không phục vụ)
Rent:                      3 – 8%    (không cần mặt tiền)

→ Commission cho Grab/Shopee là chi phí lớn nhất sau food cost
→ Nhiều cloud kitchen phải bán giá cao hơn để bù commission
```

---

### 2.6 Franchise Model

**Đặc trưng:**
- Franchisor (thương hiệu gốc) bán quyền kinh doanh cho Franchisee (người mua franchise)
- Franchisee trả: **Initial Fee** (phí ban đầu) + **Royalty Fee** (% doanh thu hàng tháng)
- Franchisor cung cấp: thương hiệu, công thức, training, hệ thống

**Cơ cấu phí franchise điển hình:**
```
Initial Fee:        100tr – 1 tỷ (tùy thương hiệu)
Royalty Fee:        3 – 8% doanh thu/tháng
Marketing Fee:      1 – 3% doanh thu/tháng

Ví dụ: Doanh thu 500tr/tháng, royalty 5%
→ Franchisee trả franchisor: 25,000,000đ/tháng
```

**Góc nhìn Franchisor (brand lớn):**
```
Doanh thu của Franchisor:
  Royalty từ 100 cửa hàng franchise × 500tr/tháng × 5%
  = 2,500,000,000đ/tháng chỉ từ royalty
  
→ Không cần vốn mở cửa hàng, không chịu rủi ro vận hành
→ Đây là lý do các chuỗi F&B lớn đẩy mạnh franchise
```

---

## 3. Unit Economics — Đo lường sức khỏe tài chính F&B

### 3.1 Các chỉ số quan trọng nhất

#### Food Cost % (FC%)

```
Food Cost % = (Giá vốn nguyên liệu / Doanh thu) × 100

Ví dụ:
  Doanh thu tháng:    500,000,000đ
  Chi phí nguyên liệu: 150,000,000đ
  Food Cost % = 30%

Benchmark ngành:
  Đồ uống (beverage):  18 – 25%
  Fast food:           28 – 35%
  Casual dining:       28 – 35%
  Fine dining:         25 – 35%

→ FC% càng thấp = margin càng cao
→ Nhưng không được giảm FC% bằng cách dùng nguyên liệu kém
```

#### Labor Cost % (LC%)

```
Labor Cost % = (Tổng chi phí nhân sự / Doanh thu) × 100

Bao gồm:
  - Lương cơ bản
  - OT
  - BHXH công ty đóng
  - Thưởng, phúc lợi

Benchmark:
  QSR:              22 – 28%
  Casual dining:    28 – 35%
  Fine dining:      35 – 45%
  Coffee chain:     25 – 32%
```

#### Prime Cost

```
Prime Cost = Food Cost + Labor Cost

Đây là chỉ số quan trọng nhất trong F&B

Nếu Prime Cost > 65%: Nguy hiểm — khó có lãi
Mục tiêu tốt:         Prime Cost < 60%
Xuất sắc:             Prime Cost < 55%

Ví dụ:
  Doanh thu:    500,000,000đ
  Food cost:    30% = 150,000,000đ
  Labor cost:   28% = 140,000,000đ
  Prime Cost:   58% = 290,000,000đ  ✓ Tốt
```

#### EBITDA Margin

```
EBITDA = Doanh thu - Food Cost - Labor - Rent - Other OPEX

Doanh thu:           500,000,000đ   100%
Food Cost:          -150,000,000đ   -30%
Labor Cost:         -140,000,000đ   -28%
Rent:                -60,000,000đ   -12%
Utilities:           -20,000,000đ    -4%
Other (marketing,   -25,000,000đ    -5%
  packaging, etc.)
─────────────────────────────────────────
EBITDA:             105,000,000đ    21%

Benchmark:
  < 10%: Rất thấp, cần cải thiện
  10-15%: Trung bình
  15-20%: Tốt
  > 20%: Xuất sắc
```

#### Average Order Value (AOV)

```
AOV = Tổng doanh thu / Số bill (hóa đơn)

Quan trọng vì:
  - Tăng AOV = tăng doanh thu không cần thêm khách
  - Cách tăng AOV: upsell, combo, add-on, size upgrade
```

#### Table Turnover Rate

```
Chỉ áp dụng với mô hình dine-in.

Turnover Rate = Số khách phục vụ / Số ghế

Peak lunch, casual dining tốt: 1.5 – 2.5 lần
Peak dinner:                   1.2 – 2.0 lần

Cách tăng turnover:
  - Giảm thời gian chờ món
  - Menu đơn giản, quyết định nhanh
  - Thanh toán nhanh (QR, không chờ nhân viên)
  - Không khuyến khích ngồi quá lâu (no free wifi ở QSR)
```

### 3.2 Break-even Analysis — Khi nào hòa vốn?

```
Fixed Costs/tháng:
  Rent:                   50,000,000đ
  Labor cố định:          80,000,000đ
  Utilities cố định:       5,000,000đ
  Khấu hao thiết bị:      10,000,000đ
  Tổng Fixed Cost:       145,000,000đ

Variable Cost:
  Food cost:              30% doanh thu

Contribution Margin = 1 - Variable Cost % = 70%

Break-even Revenue = Fixed Cost / Contribution Margin
                   = 145,000,000 / 0.70
                   = 207,142,857đ/tháng

→ Cần doanh thu > 207 triệu/tháng để bắt đầu có lãi
→ = ~6.9 triệu/ngày = ~70 khách × 100,000đ AOV
```

---

## 4. Menu Engineering — Quản lý menu, giá, nguyên liệu

### 4.1 Menu không chỉ là danh sách món ăn

Menu là **công cụ kinh doanh** — được thiết kế để tối đa hóa lợi nhuận, không chỉ để khách biết bán gì.

**Menu Engineering Matrix** — Phân loại món ăn theo 2 chiều:

```
                HIGH Popularity
                      │
         PLOW HORSE   │   STAR ⭐
         (Bán nhiều   │  (Bán nhiều,
          nhưng margin│   margin cao)
          thấp)       │
LOW ─────────────────────────────── HIGH
Margin                │              Margin
         DOG 🐕        │   PUZZLE ❓
         (Bán ít,     │  (Margin cao
          margin thấp)│   nhưng ít người
                      │   gọi)
                      │
                LOW Popularity
```

**Chiến lược cho từng nhóm:**

| Nhóm | Đặc điểm | Chiến lược |
|---|---|---|
| **Star ⭐** | Bán chạy + margin cao | Giữ nguyên, đặt vị trí nổi bật trên menu |
| **Plow Horse** | Bán chạy nhưng margin thấp | Tăng giá nhẹ hoặc giảm portion size |
| **Puzzle ❓** | Margin cao nhưng ít người gọi | Đổi tên, mô tả hấp dẫn hơn, đặt vị trí tốt hơn |
| **Dog 🐕** | Ít người gọi + margin thấp | Xem xét loại khỏi menu |

### 4.2 Menu Pricing — Định giá món ăn

#### Phương pháp Cost-Plus (Phổ biến nhất)

```
Selling Price = Food Cost / Target Food Cost %

Ví dụ: Món Bò Lúc Lắc
  Nguyên liệu:
    Bò philê 150g:     45,000đ
    Rau củ:             8,000đ
    Gia vị, dầu ăn:     5,000đ
    Tổng food cost:    58,000đ

  Target FC% = 30%
  Selling Price = 58,000 / 0.30 = 193,333đ
  → Làm tròn lên: 195,000đ hoặc 200,000đ
```

#### Phương pháp Competitive Pricing

```
Không chỉ nhìn vào cost — còn nhìn vào đối thủ:
  Đối thủ A bán cùng món: 180,000đ
  Đối thủ B bán cùng món: 220,000đ

  Nếu mình định vị premium: 210,000đ
  Nếu mình định vị value:   175,000đ

→ Price anchoring: đặt một món giá cao để món giá trung bình
  trông "hợp lý" hơn
```

#### Psychological Pricing

```
Tránh số tròn: 99,000 thay vì 100,000
Anchor pricing: Đặt món 500,000đ để món 250,000đ trông rẻ
Bundle/Combo: 2 món + nước = 180,000đ (mỗi món riêng = 110,000đ)
```

### 4.3 Recipe Management — Quản lý công thức

**Recipe (Công thức nấu ăn)** trong hệ thống F&B không chỉ là hướng dẫn nấu — nó là **tiêu chuẩn hóa** và **kiểm soát chi phí**.

```
RECIPE: Bạch Tuộc Nướng (1 phần)
────────────────────────────────────────
Nguyên liệu:          Định lượng    Đơn giá    Thành tiền
Bạch tuộc tươi        200g          120,000/kg    24,000
Sả tươi               20g             15,000/kg       300
Ớt tươi               10g             30,000/kg       300
Nước mắm              15ml             40,000/L        600
Dầu mè                5ml             80,000/L        400
Tỏi                   10g             25,000/kg       250
Than nướng             50g              8,000/kg       400
Đĩa, khăn (consumables)                               500
────────────────────────────────────────────────────────
Food Cost:                                         26,750đ
Selling Price:                                    95,000đ
Food Cost %:                                       28.2%
────────────────────────────────────────────────────────
```

**Tại sao Recipe quan trọng trong hệ thống?**

```
1. Standardization:
   Mọi cửa hàng trong chuỗi ra món giống hệt nhau

2. Cost Control:
   Biết chính xác food cost của từng món
   → Phát hiện khi bếp dùng quá định lượng

3. Inventory Planning:
   Bán 100 phần Bạch Tuộc = cần 20kg bạch tuộc
   → Tính toán được lượng cần mua/nhập kho

4. Waste Tracking:
   Nếu xuất kho 25kg bạch tuộc nhưng chỉ bán 100 phần (= 20kg)
   → 5kg hao hụt → đi đâu? Thừa? Hỏng? Bị lấy?
```

### 4.4 Modifier & Variant — Tùy chỉnh món ăn

Khách hàng không bao giờ chỉ gọi đúng theo menu — họ muốn tùy chỉnh.

```
MODIFIER (Tùy chỉnh bắt buộc hoặc tùy chọn):

Cà phê sữa đá:
  Size: [S / M / L]  ← Required modifier
  Đường: [Ít / Vừa / Nhiều / Không đường]  ← Required
  Đá: [Ít đá / Vừa / Nhiều đá / Không đá]  ← Required
  Topping thêm: [Kem cheese / Trân châu / Thạch]  ← Optional, có phụ thu

Phở bò:
  Tái/Chín/Tái chín: ← Required
  Size tô: [Nhỏ/Vừa/Lớn]  ← Required, giá khác nhau
  Thêm: [Thêm bánh phở / Thêm thịt]  ← Optional, có phụ thu
```

**Modifier ảnh hưởng đến:**
- Giá bán (size lớn hơn, topping thêm = giá cao hơn)
- Recipe (ít đường → không cho thêm syrup → food cost thay đổi nhẹ)
- KDS (Kitchen Display System) — bếp cần biết rõ từng tùy chỉnh

### 4.5 Menu Lifecycle — Vòng đời của một menu

```
[1] MENU PLANNING (Lập kế hoạch)
    - Nghiên cứu xu hướng, đối thủ
    - Tính toán food cost từng món
    - Đảm bảo balance: variety nhưng không quá nhiều

[2] TESTING & TASTING (Thử nghiệm)
    - Bếp trưởng thử nghiệm công thức
    - Internal tasting session
    - Điều chỉnh recipe, định lượng

[3] COSTING (Tính giá vốn)
    - Tính food cost chính xác
    - Set selling price theo target margin

[4] LAUNCH (Ra mắt)
    - Training nhân viên (bếp, phục vụ)
    - Setup trong POS
    - Marketing (post social media, in menu mới)

[5] MONITORING (Theo dõi)
    - Theo dõi số lượng bán theo từng món
    - Kiểm tra food cost thực tế vs lý thuyết
    - Ghi nhận feedback khách hàng

[6] OPTIMIZATION (Tối ưu)
    - Apply Menu Engineering Matrix
    - Điều chỉnh giá, đổi tên, đổi vị trí trên menu
    - Loại món Dog, thêm món mới

Chu kỳ: Thường 3-6 tháng/lần review toàn bộ menu
         Seasonal items: theo mùa/sự kiện
```

---

## 5. Order Flow — Từ lúc khách gọi món đến khi ra bàn

### 5.1 Tổng quan các kênh đặt hàng

```
DINE-IN                    TAKEAWAY              DELIVERY
──────────────             ──────────            ──────────
Khách tự gọi               Khách đến lấy         App (Grab, Shopee)
  → Waiter ghi order          → Gọi điện          Website/App riêng
  → POS nhập order            → Walk-in order     Gọi điện đặt trước
  → KDS bếp nhận              → Online pre-order
  → Phục vụ ra bàn
  
Self-order kiosk
QR order tại bàn
Tablet order
```

### 5.2 Dine-in Order Flow — Chi tiết từng bước

```
[1] SEATING (Xếp bàn)
    Khách đến → Host/Hostess đón
    → Kiểm tra bàn trống (Floor Map)
    → Dẫn khách → Đánh dấu bàn "Occupied" trong hệ thống
    → Ghi số lượng khách (pax)

[2] ORDER TAKING (Ghi order)
    Waiter mang menu đến → Khách chọn món

    Phương thức ghi order:
    A. Handheld POS (máy cầm tay)
       Waiter ghi trực tiếp → Sync lên server → KDS bếp nhận ngay
    
    B. QR Order (Khách tự gọi)
       Khách quét QR tại bàn → Gọi trên điện thoại
       → Order vào hệ thống → KDS bếp nhận

    C. Paper order (truyền thống)
       Waiter ghi tay → Mang phiếu vào bếp → Sau đó nhập POS
       → Chậm, dễ sai, không real-time

[3] ORDER ROUTING (Phân luồng về bếp)
    Không phải tất cả order đều đến cùng một nơi:

    Lẩu BBQ:
      Khu bếp lạnh: Thịt, rau, hải sản
      Khu bar:      Đồ uống

    Nhà hàng đầy đủ:
      Hot kitchen:  Món nóng (xào, chiên, hấp)
      Cold station: Salad, món nguội, sushi
      Grill:        Nướng, BBQ
      Bar:          Đồ uống, cocktail, dessert

    → Mỗi station có KDS riêng, chỉ hiển thị phần việc của mình

[4] KITCHEN DISPLAY SYSTEM (KDS)
    Bếp nhận order qua màn hình — không dùng giấy

    Mỗi order hiển thị:
      Bàn số:    5
      Số khách:  4
      Order:     2x Bò lúc lắc
                 1x Gà nướng muối ớt (KHÔNG CHUA)
                 1x Rau xào tỏi
      Time:      14:32 (đã chờ 8 phút)  ← màu vàng cảnh báo
      Ghi chú:   Khách dị ứng đậu phộng

    Color coding:
      Xanh:   Mới vào, còn trong thời gian chuẩn
      Vàng:   Sắp quá thời gian chuẩn
      Đỏ:     Đã quá thời gian → cần ưu tiên

[5] PREPARATION (Chế biến)
    Bếp thực hiện theo đúng recipe
    → Thực hiện xong → Tap "Done" trên KDS
    → Hệ thống ghi nhận thời gian hoàn thành

[6] EXPEDITING (Kiểm soát ra món)
    Expo (người kiểm soát cuối) kiểm tra trước khi ra bàn:
      - Đúng món?
      - Đủ phần?
      - Trình bày đúng chuẩn?
      - Còn nóng?

[7] SERVING (Phục vụ)
    Waiter mang ra bàn
    → Báo tên món khi đặt xuống
    → Hệ thống cập nhật: "Món đã ra bàn"

[8] BILL REQUEST (Yêu cầu thanh toán)
    Khách yêu cầu bill:
      - Waiter gọi bill trên handheld
      - Hoặc khách nhấn nút gọi tại bàn
      - Hoặc quét QR để xem bill và thanh toán

[9] PAYMENT (Thanh toán)
    (Chi tiết ở mục 6)

[10] TABLE RESET (Dọn bàn)
    Waiter dọn bàn → Đánh dấu bàn trống
    → Sẵn sàng cho khách tiếp theo
```

### 5.3 KPI của Order Flow

```
Kitchen Display Time (KDT):
  Thời gian từ khi order vào đến khi bếp hoàn thành
  Target: 8-12 phút (casual dining)
          3-5 phút (QSR)

Ticket Time (End-to-end):
  Thời gian từ khi khách gọi đến khi nhận món
  Target: 12-18 phút (casual dining)
          < 5 phút (QSR)

Order Accuracy Rate:
  % order ra đúng so với tổng order
  Target: > 99%

Table Waiting Time:
  Thời gian chờ từ khi khách vào đến khi được xếp bàn
  Target: < 5 phút (dine-in)
```

### 5.4 Delivery Order Flow

```
[1] Khách đặt trên app (GrabFood, ShopeeFood)
         │
[2] Hệ thống nhận order
    → Auto-accept hoặc manual accept (nếu bếp quá tải)
    → Xác nhận thời gian chuẩn bị
         │
[3] In ticket hoặc hiển thị trên KDS
    → Bếp chuẩn bị (ưu tiên ngang với dine-in)
         │
[4] Đóng gói
    → Bao bì phù hợp (tránh đổ, giữ nhiệt)
    → Label: tên món, dị ứng, cửa hàng, thời gian
         │
[5] Shipper đến lấy
    → Xác nhận order ID
    → Bàn giao → hệ thống cập nhật "Picked up"
         │
[6] Giao đến khách
    → Cập nhật "Delivered"
    → Doanh thu ghi nhận
         │
[7] Settlement với app
    → Hàng tuần/tháng app chuyển tiền về
    → Trừ commission (25-30%)
```

---

## 6. POS & Hệ thống thanh toán trong F&B

### 6.1 POS trong F&B khác với POS thông thường

POS trong F&B không chỉ là "máy tính tiền" — nó là **hệ thống điều phối vận hành**:

```
POS thông thường (retail):
  Quét mã → Tính tiền → Thu tiền
  Đơn giản, linear

POS F&B:
  Ghi order → Route đến bếp/bar → Track thời gian
  → Quản lý bàn → Tách/gộp bill → Tính tiền
  → Nhiều phương thức thanh toán → Báo cáo
  Phức tạp hơn nhiều
```

### 6.2 Các tính năng cốt lõi của POS F&B

**Table Management (Quản lý bàn):**
```
Floor Map — Sơ đồ mặt bằng nhà hàng:

[B1: Trống] [B2: 4/6 người] [B3: Bill] [B4: Trống]
[B5: Đặt trước 7pm]         [B6: 2/4 người]
[B7: Trống] [B8: Đang dọn]  [B9: 3/4 người]

Màu sắc:
  Xanh lá: Trống, sẵn sàng
  Đỏ:      Có khách đang dùng
  Cam:     Đã có bill, chờ thanh toán
  Xám:     Đang dọn / không dùng
  Tím:     Đặt trước
```

**Order Management:**
```
Thêm/sửa/xóa món (trước khi gửi bếp)
Ghi chú cho từng món (ít cay, không đường...)
Void món (sau khi gửi bếp — cần lý do và quyền)
Course management: Gửi khai vị trước, món chính sau
Fire (gửi bếp làm ngay) vs Hold (chờ lệnh)
```

**Split Bill (Tách hóa đơn):**
```
Chia đều: 4 người, tổng 400,000đ → mỗi người 100,000đ

Chia theo món:
  Người A: Bò lúc lắc + 1 bia      = 250,000đ
  Người B: Cơm sườn + nước ép      = 150,000đ

Chia một phần: Người A trả 300,000đ, còn lại nhóm trả
```

**Promotion & Discount:**
```
Loại discount:
  % discount: Giảm 10% tổng bill
  Fixed amount: Giảm 50,000đ
  Item discount: Món này giảm 20%
  BOGO: Mua 1 tặng 1
  Happy hour: 3-5pm giảm 20% đồ uống
  Member discount: Thành viên Gold giảm 15%

Quy tắc:
  Ai được phép apply discount? (phân quyền)
  Có cần approval manager không?
  Discount tối đa bao nhiêu không cần approval?
```

### 6.3 Payment Flow trong F&B

```
[1] Khách yêu cầu bill
         │
[2] Waiter/Cashier in hoặc hiển thị bill
    Chi tiết:
      Bàn 5                    15/12/2024  19:45
      ─────────────────────────────────────────
      2x Bò lúc lắc    @195,000    390,000
      1x Gà nướng       @185,000    185,000
      1x Rau xào tỏi    @65,000      65,000
      1x Bia Tiger      @45,000      45,000
      1x Nước cam       @55,000      55,000
      ─────────────────────────────────────────
      Subtotal:                     740,000
      VAT 10%:                       74,000
      ─────────────────────────────────────────
      TOTAL:                        814,000
         │
[3] Khách chọn phương thức thanh toán:

    TIỀN MẶT:
    → Nhập số tiền khách đưa
    → Hệ thống tính tiền thối
    → Mở ngăn kéo tiền
    → In receipt

    THẺ (Credit/Debit):
    → Kết nối với EDC (Electronic Data Capture) machine
    → Khách quẹt/chạm thẻ
    → Chờ approve từ ngân hàng
    → In slip + receipt

    QR CODE (VietQR, Momo, ZaloPay):
    → Hiển thị QR trên màn hình
    → Khách quét → Thanh toán trên app
    → POS nhận callback "Payment successful"
    → In receipt

    VOUCHER / GIFT CARD:
    → Nhập mã voucher
    → Hệ thống verify (còn hạn? Đủ tiền?)
    → Trừ vào tổng bill
    → Phần còn lại thanh toán bằng phương thức khác

    MIXED PAYMENT:
    → Voucher 100,000 + Tiền mặt 714,000
    → Hệ thống hỗ trợ nhiều payment method trong 1 bill
         │
[4] Bill đóng (Closed)
    → Table status → Available
    → Doanh thu ghi nhận
    → Inventory tự động trừ (nếu tích hợp)
```

### 6.4 End-of-Day (EOD) — Đóng ca

```
Cuối mỗi ca/ngày, cashier thực hiện EOD:

[1] Cash Count (Đếm tiền mặt):
    Tiền mặt thực tế trong két: 2,450,000đ
    Hệ thống ghi nhận:          2,380,000đ
    Float (tiền thối ban đầu):    500,000đ
    → Doanh thu tiền mặt thực = 2,450,000 - 500,000 = 1,950,000đ
    → Doanh thu hệ thống =       2,380,000 - 500,000 = 1,880,000đ
    → Chênh lệch (Over/Short):      +70,000đ (thừa 70k)
       → Cần điều tra: nhân viên trả thừa? Nhập sai?

[2] Reconcile by Payment Method:
    Tiền mặt:    1,950,000đ (thực tế) vs 1,880,000đ (hệ thống)
    Thẻ:         3,200,000đ
    QR:          2,100,000đ
    Voucher:       300,000đ
    ─────────────────────────────────────────
    Tổng:        7,550,000đ (thực tế)

[3] Z-Report:
    Báo cáo tổng kết ca — in từ POS:
    - Tổng doanh thu
    - Doanh thu theo phương thức TT
    - Số bill
    - AOV
    - Void/Discount summary
    - Tax collected

[4] Cash Drop:
    Nộp tiền mặt vào két lớn hoặc ngân hàng
    Ký biên nhận
```

---

## 7. Inventory & Cost Control — Kiểm soát chi phí

### 7.1 Tại sao Food Cost thực tế luôn cao hơn lý thuyết?

```
Lý thuyết: Bán 100 phần Bò Lúc Lắc × 58,000đ food cost = 5,800,000đ
Thực tế:   Xuất kho nguyên liệu:                         = 7,200,000đ

Chênh lệch 1,400,000đ đi đâu?

Các nguyên nhân:
  1. WASTE (Hao hụt tự nhiên):
     - Bò mua về: 10kg → Sau khi tỉa, làm sạch: 8.5kg (85%)
     - Rau cải: 5kg → Sau nhặt: 4.2kg (84%)
     - Hao hụt chế biến là bình thường — phải tính vào recipe

  2. SPOILAGE (Hỏng, thối):
     - Nguyên liệu để quá hạn
     - Bảo quản không đúng
     - Mất điện tủ lạnh

  3. OVER-PORTIONING (Bếp cho quá định lượng):
     - Recipe: 150g bò/phần, bếp cho 170g
     - × 100 phần = 2kg thịt thừa = ~90,000đ/ngày = ~2.7tr/tháng

  4. EMPLOYEE MEALS (Bữa ăn nhân viên):
     - Nếu không tính riêng → bị tính vào food cost

  5. THEFT (Trộm cắp):
     - Nhân viên lấy nguyên liệu về
     - Bán ngoài mà không nhập order vào POS

  6. SPILLAGE & MISTAKES (Đổ vỡ, làm hỏng):
     - Đổ đồ uống
     - Nấu sai phải làm lại
```

### 7.2 Inventory Management trong F&B

**Đặc thù của kho F&B so với kho thông thường:**

```
Kho thông thường:        Kho F&B:
Hàng tháng/năm          Hàng ngày, hàng tuần
Sản phẩm lâu bền        Nguyên liệu tươi sống
Ít loại, nhiều lượng    Nhiều loại, ít lượng mỗi loại
Kiểm kê tháng/năm       Kiểm kê ngày/tuần
```

**Phân loại kho trong F&B:**

```
DRY STORAGE (Kho khô):
  Gạo, bột mì, đường, dầu ăn, đồ hộp
  Nhiệt độ: 15-21°C, khô ráo
  Nhập/xuất: Weekly hoặc 2 lần/tuần

REFRIGERATED STORAGE (Tủ lạnh):
  Thịt, cá, sữa, rau củ chế biến sẵn
  Nhiệt độ: 0-4°C
  Nhập: Hàng ngày
  
FROZEN STORAGE (Tủ đông):
  Thịt dự trữ, hải sản, kem, thực phẩm đông lạnh
  Nhiệt độ: -18°C trở xuống
  Nhập: 2-3 lần/tuần

BAR STORAGE:
  Rượu, bia, đồ uống đóng chai
  Có thể lock riêng — high value, dễ bị lấy

DRY GOODS (Vật tư tiêu hao):
  Bao bì, khăn giấy, ly nhựa, ống hút
  Không phải food cost nhưng vẫn phải quản lý
```

### 7.3 Par Level & Ordering — Đặt hàng tự động

**Par Level** = Lượng tồn kho tối thiểu cần có tại một thời điểm.

```
Par Level = Mức tiêu thụ trung bình × Số ngày đặt hàng
          + Safety stock

Ví dụ: Thịt bò phi-lê
  Tiêu thụ: 3kg/ngày
  Giao hàng mỗi 2 ngày (Lead time = 2 ngày)
  Safety stock: 1 ngày = 3kg

  Par Level = 3 × 2 + 3 = 9kg

Quy trình đặt hàng:
  Kiểm kê mỗi sáng: Còn 5kg bò
  Par Level: 9kg
  → Order thêm: 9 - 5 = 4kg (làm tròn lên 5kg cho tiện)
```

**Order guide — Template đặt hàng:**

```
DAILY ORDER GUIDE — 15/12/2024
────────────────────────────────────────────────────────
Nguyên liệu   │ Par Level │ Tồn kho │ Cần đặt │ Ghi chú
──────────────┼───────────┼─────────┼─────────┼──────────
Bò phi-lê     │    9kg    │    5kg  │   4kg   │
Cá hồi        │    5kg    │    2kg  │   3kg   │
Rau cải xanh  │    8kg    │    3kg  │   5kg   │ Kiểm tra chất lượng
Sữa tươi      │   20L     │    8L   │  12L    │
Trứng gà      │   100 quả │   40 quả│  60 quả │
────────────────────────────────────────────────────────
```

### 7.4 FIFO trong F&B — Cực kỳ quan trọng

FIFO không chỉ là phương pháp kế toán — trong F&B nó là **quy trình vận hành bắt buộc** để tránh hư hỏng.

```
FIFO trong thực tế:

Khi nhập hàng mới:
  → Đặt hàng mới VÀO SAU / VÀO DƯỚI
  → Hàng cũ ra trước

Label hàng hóa:
  RECEIVE DATE: 15/12/2024
  USE BY: 18/12/2024

Quy tắc 4 màu nhãn (best practice):
  Thứ 2: Nhãn Đỏ
  Thứ 3: Nhãn Xanh lá
  Thứ 4: Nhãn Xanh dương
  Thứ 5: Nhãn Vàng
  → Nhân viên nhìn màu biết ngay hàng nhập ngày nào
```

### 7.5 Stocktaking — Kiểm kê

**Mục đích:** Đối chiếu tồn kho thực tế với số liệu hệ thống, tính toán food cost chính xác.

**Các loại kiểm kê:**

```
DAILY COUNT (Kiểm kê hàng ngày):
  - Các mặt hàng có giá trị cao: thịt, hải sản, rượu
  - Thực hiện đầu ca hoặc cuối ca
  - So sánh: Tồn đầu + Nhập - Xuất = Tồn cuối
  - Phát hiện chênh lệch ngay trong ngày

WEEKLY COUNT (Kiểm kê hàng tuần):
  - Toàn bộ kho dry goods và frozen
  - Tính food cost tuần

MONTHLY COUNT (Kiểm kê tháng):
  - Kiểm kê toàn diện
  - Báo cáo tài chính tháng
  - Variance analysis chi tiết
```

**Inventory Variance Report:**

```
Tháng 12/2024 — Báo cáo chênh lệch tồn kho

Nguyên liệu  │ Lý thuyết │ Thực tế │ Chênh lệch │ Giá trị CL │ % CL
─────────────┼───────────┼─────────┼────────────┼────────────┼──────
Thịt bò      │    85kg   │   83kg  │    -2kg    │  -240,000  │ -2.4%
Tôm          │    40kg   │   39kg  │    -1kg    │  -180,000  │ -2.5%
Rượu vang    │    24 chai│  22 chai│   -2 chai  │  -600,000  │ -8.3% ⚠️
Bia Tiger    │   200 lon │ 198 lon │    -2 lon  │   -18,000  │ -1.0%

⚠️ Rượu vang chênh 8.3% — cần điều tra
   Có thể: Nhân viên uống? Bán nhưng không nhập POS?
```

### 7.6 Theoretical vs Actual Food Cost

Đây là phân tích quan trọng nhất để kiểm soát chi phí:

```
THEORETICAL FOOD COST:
  Tính dựa trên recipe × số món đã bán theo POS
  = Nếu bếp làm đúng 100% theo công thức, chi phí phải là X

ACTUAL FOOD COST:
  Tính dựa trên kiểm kê thực tế
  = (Tồn đầu kỳ + Nhập) - Tồn cuối kỳ

VARIANCE:
  = Actual - Theoretical

Ví dụ:
  Theoretical FC: 28.5% doanh thu = 142,500,000đ
  Actual FC:      31.2% doanh thu = 156,000,000đ
  Variance:        2.7%           =  13,500,000đ

Nguyên nhân variance:
  Over-portioning:    ~40% variance
  Spoilage/Waste:     ~30% variance
  Theft:              ~15% variance
  Recipe không chuẩn: ~15% variance

→ Hành động: Đào tạo lại bếp về định lượng,
              Kiểm tra quy trình bảo quản,
              Tăng cường giám sát
```

---

## 8. Hệ thống trong F&B — Tổng quan các phần mềm

### 8.1 Tech Stack của một chuỗi F&B hiện đại

```
┌─────────────────────────────────────────────────────────────┐
│                    CUSTOMER TOUCHPOINTS                      │
│  App mobile │ Website │ QR Order │ Kiosk │ Delivery App     │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│                   ORDER MANAGEMENT                           │
│              POS System (trung tâm)                         │
│         Table Mgmt │ KDS │ Order Routing                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
┌────────────┐  ┌────────────┐  ┌────────────┐
│  INVENTORY │  │  PAYMENT   │  │    CRM     │
│  & RECIPE  │  │ PROCESSING │  │  LOYALTY   │
│  MANAGEMENT│  │            │  │            │
└────────────┘  └────────────┘  └────────────┘
        │              │              │
        └──────────────┼──────────────┘
                       ▼
              ┌────────────────┐
              │   REPORTING    │
              │  & ANALYTICS   │
              │  (Dashboard)   │
              └────────────────┘
```

### 8.2 Tích hợp quan trọng

**POS ↔ Kitchen Display System (KDS):**
```
Real-time order routing
Thời gian chuẩn bị tracking
Bump (xác nhận hoàn thành) → POS biết món đã sẵn sàng
```

**POS ↔ Inventory:**
```
Mỗi bill đóng → Tự động trừ nguyên liệu theo recipe
Tồn kho real-time
Cảnh báo khi gần hết nguyên liệu
```

**POS ↔ Payment Gateway:**
```
QR payment, thẻ tín dụng
Split payment
Refund processing
```

**POS ↔ Delivery Apps (Grab, Shopee):**
```
Order nhận từ app → Tự động vào POS/KDS
Không cần nhân viên nhìn màn hình riêng của từng app
Menu sync: Thay đổi giá/sold out → Tự động update trên app
```

**POS ↔ Loyalty/CRM:**
```
Nhận điểm khi thanh toán
Redeem điểm khi trả bill
Nhận diện khách quen → Hiển thị lịch sử, preference
```

---

## 9. Loyalty & CRM trong F&B

### 9.1 Tại sao Loyalty quan trọng đặc biệt với F&B?

```
Chi phí thu hút khách mới (CAC):   ~5-7x
Chi phí giữ chân khách cũ:         ~1x

Khách hàng trung thành:
  → Ghé thăm thường xuyên hơn
  → Chi tiêu nhiều hơn mỗi lần (~67% nhiều hơn)
  → Giới thiệu bạn bè (word of mouth)
  → Ít nhạy cảm với giá hơn
```

### 9.2 Các mô hình loyalty phổ biến

**Points-based (Tích điểm):**
```
Mỗi 10,000đ = 1 điểm
100 điểm = Đổi được 20,000đ
```

**Tier-based (Hạng thành viên):**
```
Silver: Chi tiêu 0-2tr/tháng → 5% discount
Gold:   Chi tiêu 2-5tr/tháng → 10% discount + birthday gift
Platinum: > 5tr/tháng → 15% discount + ưu tiên đặt bàn
```

**Stamp Card (Thẻ dấu — phổ biến với coffee):**
```
Mua 9 ly → Tặng 1 ly
Đơn giản, không cần app
```

**Subscription (Thẻ tháng):**
```
199,000đ/tháng → Uống không giới hạn cà phê đen
299,000đ/tháng → Uống không giới hạn mọi đồ uống
```

---

## 10. Đặc thù vận hành theo từng ca

### 10.1 Cấu trúc ca làm việc

```
OPENING CA SÁNG (6:00 – 7:00 trước khi mở):
  □ Kiểm tra nhiệt độ tủ lạnh/tủ đông
  □ Nhận nguyên liệu giao buổi sáng
  □ Kiểm kê nguyên liệu tươi
  □ Mise en place (chuẩn bị sẵn nguyên liệu)
  □ Setup bàn ghế, kiểm tra vệ sinh
  □ Test POS, máy in hóa đơn, KDS
  □ Đếm tiền mặt float

PEAK HOURS (11:30-13:30 trưa; 18:00-20:30 tối):
  → Toàn bộ nhân sự tập trung
  → Không làm việc phụ — chỉ phục vụ
  → Manager tại floor, không trong văn phòng

CLOSING (Sau khi đóng cửa):
  □ EOD trên POS
  □ Đếm tiền mặt, nộp két
  □ Kiểm kê nguyên liệu còn lại
  □ Vệ sinh bếp, thiết bị
  □ Bảo quản nguyên liệu đúng cách
  □ Lập order cho ngày hôm sau
  □ Báo cáo ca cho quản lý
```

---

## 11. Tổng kết — Mental Model cho F&B Domain

### Ba trục quan trọng nhất

```
      QUALITY
         │
         │   F&B thành công
         │   nằm ở điểm cân bằng
         │   của 3 trục này
COST ────┼──── SPEED
         │
```

- **Tăng speed** (phục vụ nhanh hơn) → Nguy cơ giảm quality
- **Giảm cost** (dùng nguyên liệu rẻ hơn) → Nguy cơ giảm quality
- **Tăng quality** (nguyên liệu tốt hơn, phục vụ cẩn thận hơn) → Tăng cost và giảm speed

### Framework phân tích bất kỳ vấn đề F&B nào

**Khi doanh thu giảm → hỏi:**
```
Volume giảm (ít khách hơn)?
  → Marketing? Vị trí? Cạnh tranh mới?

AOV giảm (khách gọi ít hơn)?
  → Menu? Upsell? Giá?

Cả hai giảm?
  → Vấn đề nghiêm trọng hơn — trải nghiệm, chất lượng
```

**Khi lợi nhuận giảm dù doanh thu ổn → hỏi:**
```
Food cost tăng?
  → Giá nguyên liệu tăng? Over-portioning? Waste? Theft?

Labor cost tăng?
  → Scheduling không hiệu quả? OT nhiều?

Rent/fixed cost tăng?
  → Doanh thu không tăng kịp chi phí cố định?
```

### Bảng thuật ngữ nhanh

| Thuật ngữ | Ý nghĩa |
|---|---|
| **AOV** | Average Order Value — giá trị trung bình mỗi hóa đơn |
| **Prime Cost** | Food Cost + Labor Cost — chỉ số sức khỏe quan trọng nhất |
| **Food Cost %** | % chi phí nguyên liệu trên doanh thu |
| **Table Turnover** | Số lần một bàn được dùng trong một ca |
| **RevPASH** | Doanh thu trên mỗi ghế mỗi giờ |
| **KDS** | Kitchen Display System — màn hình hiển thị order tại bếp |
| **POS** | Point of Sale — hệ thống bán hàng |
| **Recipe/Formula** | Công thức chuẩn hóa định lượng nguyên liệu từng món |
| **Mise en Place** | Chuẩn bị nguyên liệu sẵn trước giờ cao điểm |
| **Par Level** | Mức tồn kho tối thiểu cần duy trì |
| **FIFO** | First In First Out — hàng nhập trước xuất trước |
| **Variance** | Chênh lệch giữa food cost lý thuyết và thực tế |
| **Spoilage** | Nguyên liệu bị hỏng, phải bỏ |
| **Over-portioning** | Bếp cho quá định lượng theo recipe |
| **Void** | Hủy một món trên POS sau khi đã nhập |
| **EOD** | End of Day — quy trình đóng ca cuối ngày |
| **Float** | Tiền mặt giữ lại trong két để thối khách |
| **Cover** | Một lượt khách (4 covers = 4 người đã ăn) |
| **Upsell** | Gợi ý khách mua thêm/nâng cấp để tăng AOV |
| **Cloud Kitchen** | Bếp chỉ bán delivery, không có dine-in |
| **Theoretical FC** | Food cost tính theo recipe × số món bán |
| **Actual FC** | Food cost thực tế theo kiểm kê |