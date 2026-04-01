# 🏪 Merchant Domain — Business Deep Dive

> Tài liệu này tập trung hoàn toàn vào **business** của domain Merchant trong hệ thống Payment / Fintech. Dành cho người muốn hiểu thật sự — không chỉ biết tên thuật ngữ mà còn hiểu tại sao nó tồn tại, nó hoạt động như thế nào trong thực tế, và các bên liên quan đang nghĩ gì.

---

## 1. Merchant là ai — và tại sao họ quan trọng?

### Định nghĩa đơn giản

**Merchant** là bất kỳ cá nhân hay tổ chức nào **bán hàng hoặc dịch vụ** và muốn **nhận tiền từ khách hàng thông qua hệ thống thanh toán điện tử**.

Nhưng hiểu đúng hơn trong ngữ cảnh Payment/Fintech:

> Merchant là **đối tác kinh doanh** của Payment Provider — người trả phí để dùng hạ tầng thanh toán, và đổi lại được nhận tiền từ hàng triệu khách hàng một cách an toàn, nhanh chóng.

### Merchant KHÔNG phải là khách hàng cuối

Đây là nhầm lẫn phổ biến khi mới vào nghề.

```
Customer (người mua)   ←→   Merchant (người bán)   ←→   Payment Provider
    Trả tiền                   Nhận tiền                  Cung cấp hạ tầng
    Mua hàng/dịch vụ           Bán hàng/dịch vụ            Thu phí từ merchant
```

Trong hệ thống payment:
- **Customer** là người dùng cuối — họ dùng app, quẹt thẻ, chuyển khoản
- **Merchant** là **B2B partner** — họ ký hợp đồng, đóng phí, nhận settlement
- **Payment Provider** phục vụ merchant, merchant phục vụ customer

Khi bạn build hệ thống, merchant là **doanh nghiệp đang trả tiền cho bạn** để dùng dịch vụ. Hiểu business của họ = hiểu tại sao họ có thể rời đi.

---

## 2. Toàn bộ hệ sinh thái — ai là ai?

Mỗi giao dịch thanh toán liên quan đến rất nhiều bên. Hiểu từng bên giúp bạn hiểu tại sao dòng tiền lại phức tạp như vậy.

### 2.1 Các bên chính

```
┌─────────────────────────────────────────────────────────────────┐
│                     MỘT GIAO DỊCH THANH TOÁN                    │
│                                                                  │
│  [Customer]──────►[Merchant]──────►[Payment Gateway]            │
│      │                                      │                    │
│      │                               ┌──────┴──────┐            │
│      │                               ▼             ▼            │
│      │                          [Acquirer]    [Card Network]     │
│      │                               │        (Visa/Master)     │
│      │                               ▼             │            │
│      └────────────────────────►[Issuer Bank]◄───────┘           │
│                                                                  │
└─────────────────────────────────────────────────────────────────┘
```

#### Customer (Cardholder / Payer)
Người mua hàng, trả tiền. Trong một số tài liệu còn gọi là **payer** hoặc **end-user**.
- Họ không có hợp đồng với Payment Provider
- Họ chỉ muốn thanh toán nhanh, an toàn
- Nếu có vấn đề, họ sẽ khiếu nại với **ngân hàng của họ** (issuer), không phải merchant

#### Merchant
Người bán. Ký hợp đồng với Payment Provider để chấp nhận thanh toán điện tử.
- Chịu mọi rủi ro kinh doanh: hàng hóa, dịch vụ, refund, chargeback
- Trả phí MDR cho mỗi giao dịch thành công
- Nhận tiền qua settlement định kỳ, không phải real-time

#### Acquirer (Acquiring Bank / Ngân hàng tiếp nhận)
Ngân hàng hoặc tổ chức tài chính **đại diện cho merchant** trong hệ thống thanh toán.
- Cung cấp **merchant account** — tài khoản trung gian giữ tiền trước khi settlement
- Chịu trách nhiệm với card network (Visa/Master) nếu merchant làm sai
- **Rủi ro lớn nhất của Acquirer**: merchant gian lận, chargeback cao, không đủ tiền hoàn trả

Ví dụ ở Việt Nam: VNPAY (cả acquirer và gateway), VietcomBank, Techcombank đều có vai trò acquiring.

#### Issuer (Issuing Bank / Ngân hàng phát hành)
Ngân hàng **phát hành thẻ/ví** cho customer.
- Xét duyệt hoặc từ chối giao dịch (có đủ tiền không? Có bị block không?)
- Là người **customer gọi điện khi có vấn đề**
- Xử lý chargeback khi customer khiếu nại

Ví dụ: Techcombank phát hành thẻ Visa cho bạn → Techcombank là issuer của bạn.

#### Card Network (Payment Scheme)
Tổ chức vận hành mạng lưới thanh toán toàn cầu: **Visa, Mastercard, JCB, UnionPay**.
- Đặt ra **luật chơi**: ai được tham gia, phí interchange là bao nhiêu, chargeback xử lý thế nào
- Kết nối acquirer và issuer trên toàn cầu
- Thu **interchange fee** và **scheme fee** từ mỗi giao dịch

Ở Việt Nam còn có **NAPAS** — mạng lưới thanh toán nội địa, xử lý giao dịch thẻ ATM và chuyển khoản liên ngân hàng.

#### Payment Gateway
Cầu nối kỹ thuật — nhận request thanh toán từ merchant, route đến acquirer phù hợp.
- Merchant chỉ cần tích hợp **một gateway**, gateway lo việc kết nối nhiều acquirer/ngân hàng
- Cung cấp API, SDK, dashboard cho merchant
- Thường kiêm luôn vai trò **PSP**

#### PSP (Payment Service Provider)
Cung cấp **trọn gói dịch vụ payment** cho merchant: gateway + acquiring + settlement.
- Merchant không cần ký hợp đồng riêng với từng ngân hàng
- PSP gom nhiều merchant lại, đứng ra làm **master merchant** với acquirer
- Ví dụ: Stripe (global), VNPAY, PayOS, Momo (Việt Nam)

### 2.2 Dòng tiền thực sự đi đâu?

```
Customer trả 100,000đ
         │
         ▼
Issuer Bank (ngân hàng của customer)
  Trừ 100,000đ từ tài khoản customer
  Giữ lại: Interchange fee ~1.0% = 1,000đ
         │ Chuyển 99,000đ
         ▼
Card Network (Visa/Master)
  Thu Scheme fee ~0.1% = 100đ
         │ Chuyển 98,900đ
         ▼
Acquirer (ngân hàng của merchant)
  Thu Acquirer margin ~0.2% = 200đ
         │ Chuyển 98,700đ
         ▼
Payment Gateway / PSP
  Thu Gateway fee ~0.2% = 200đ
         │
         ▼
Merchant Account (tài khoản trung gian)
  Giữ lại Rolling Reserve 10% = 10,000đ
         │
         ▼
Settlement T+1: Merchant nhận 88,700đ
(98,700đ - 10,000đ reserve)
```

> Đây là lý do MDR thường từ **1.5% – 3%**: nó bao gồm interchange fee, scheme fee, acquirer margin, và gateway fee cộng lại.

---

## 3. Phân loại Merchant

Không phải merchant nào cũng giống nhau. Phân loại đúng ảnh hưởng đến phí, giới hạn, quy trình KYB, và rủi ro.

### 3.1 Theo quy mô kinh doanh

#### Micro Merchant (Hộ kinh doanh nhỏ lẻ)
- Doanh thu < 1 tỷ/năm
- Ví dụ: quán cà phê vỉa hè, shop quần áo nhỏ, freelancer
- Thường onboard qua app tự phục vụ, KYC đơn giản (chỉ cần CCCD)
- MDR cao hơn vì volume thấp, rủi ro cao hơn
- Thanh toán chủ yếu: QR code, ví điện tử

#### SME (Small & Medium Enterprise)
- Doanh thu 1 tỷ – 500 tỷ/năm
- Ví dụ: chuỗi cửa hàng, công ty thương mại điện tử vừa, nhà hàng chain
- KYB đầy đủ: giấy phép KD, giấy tờ người đại diện, hợp đồng
- Có nhân viên sales chăm sóc
- Đàm phán được MDR thấp hơn nếu volume đủ lớn

#### Enterprise / Key Account
- Doanh thu > 500 tỷ/năm
- Ví dụ: Vincom, Thế Giới Di Động, các airline, ngân hàng
- MDR đặc biệt, thương lượng trực tiếp
- Có dedicated account manager
- Yêu cầu SLA riêng, tích hợp sâu hơn

### 3.2 Theo kênh bán hàng

#### Online Merchant
Bán hoàn toàn qua internet: website, app mobile, social commerce.
- Không có điểm vật lý
- Thanh toán: thẻ online, ví, QR, BNPL (mua trước trả sau)
- Rủi ro cao hơn offline vì không có mặt customer trực tiếp → fraud nhiều hơn
- Chargeback rate thường cao hơn

#### Offline Merchant (POS Merchant)
Bán tại cửa hàng vật lý, customer có mặt trực tiếp.
- Thanh toán: POS machine, QR in sẵn, contactless
- Rủi ro thấp hơn (có mặt customer, có camera)
- Thường MDR thấp hơn online

#### Omnichannel Merchant
Bán cả online lẫn offline — xu hướng hiện tại của retail lớn.
- Cần hệ thống payment hợp nhất: cùng dashboard, cùng báo cáo
- Phức tạp hơn về reconciliation (đối soát)

### 3.3 Theo mô hình kinh doanh

#### Direct Merchant
Merchant tự bán trực tiếp cho customer. Đơn giản nhất.
```
Customer ──► Merchant ──► Payment Provider
```

#### Marketplace / Platform Merchant
Một platform kết nối nhiều người bán (seller) với người mua. Platform thu tiền rồi chia cho các seller.
```
Customer ──► Platform ──► Seller A (sub-merchant)
                      ──► Seller B (sub-merchant)
                      ──► Platform fee
```
Ví dụ: Shopee, Lazada, Grab Food, Airbnb.

Đây là mô hình phức tạp nhất vì:
- Platform phải xử lý **split payment**: tách tiền ra cho nhiều bên
- Mỗi seller là một "sub-merchant" cần được quản lý riêng
- Compliance: platform chịu trách nhiệm KYB cho từng seller

#### Subscription Merchant
Thu tiền định kỳ (hàng tháng/năm) tự động.
```
Customer ──► Merchant
              T+0: Thu tháng 1
              T+30: Thu tháng 2 (auto)
              T+60: Thu tháng 3 (auto)
```
Ví dụ: Netflix, Spotify, các SaaS.
Cần tính năng: **recurring billing**, quản lý thẻ đã lưu (card on file), xử lý khi thẻ hết hạn.

#### On-demand / Gig Economy
Thu tiền từ customer, giữ lại phí, trả phần còn lại cho người cung cấp dịch vụ.
```
Customer trả 100,000đ cho chuyến xe
Platform giữ 20,000đ (20% commission)
Tài xế nhận 80,000đ
```
Ví dụ: Grab, Be, Gojek, Ahamove.
Cần tính năng: **payout to many** (chi trả cho hàng nghìn tài xế/ngày).

---

## 4. MDR — Trái tim của Business Model

MDR (Merchant Discount Rate) là **phí merchant trả cho mỗi giao dịch thành công**. Đây là nguồn doanh thu chính của Payment Provider.

### 4.1 MDR được tính như thế nào?

```
MDR = Interchange Fee + Scheme Fee + Acquirer Margin + Gateway Fee

Ví dụ giao dịch thẻ Visa tại merchant online:
- Interchange Fee: 1.0%   (Issuer bank thu)
- Scheme Fee:      0.1%   (Visa thu)
- Acquirer Margin: 0.2%   (Acquiring bank thu)
- Gateway Fee:     0.2%   (Payment gateway thu)
─────────────────────────
MDR:               1.5%   (Merchant trả)
```

Payment Provider thường **gộp tất cả vào một số duy nhất** khi báo cho merchant, gọi là **blended MDR**.

### 4.2 Tại sao MDR khác nhau theo phương thức thanh toán?

| Phương thức | MDR điển hình | Lý do |
|---|---|---|
| Thẻ tín dụng Visa/Master | 1.8% – 3.0% | Interchange fee cao, rủi ro chargeback |
| Thẻ ghi nợ (debit) | 0.5% – 1.5% | Interchange thấp hơn |
| QR Code (NAPAS) | 0% – 0.5% | Chính sách ưu đãi của NAPAS/NHNN |
| Ví điện tử (Momo, ZaloPay) | 0.5% – 1.5% | Phụ thuộc hợp đồng ví |
| Chuyển khoản ngân hàng | 0% – 0.3% | Chi phí thấp nhất |
| BNPL (mua trước trả sau) | 2% – 6% | Rủi ro tín dụng cao |

**Tại sao thẻ tín dụng đắt nhất?**

- Issuer bank phải bù đắp rủi ro cho vay (customer chưa có tiền vẫn mua được)
- Chương trình rewards/cashback cho customer được tài trợ bởi interchange fee
- Rủi ro chargeback cao hơn (customer dễ khiếu nại hơn)

### 4.3 MDR ảnh hưởng đến merchant như thế nào?

```
Ví dụ: Chuỗi cà phê doanh thu 10 tỷ/tháng, 100% thanh toán điện tử

Kịch bản 1: MDR = 1.5%
→ Phí payment: 150,000,000đ/tháng = 1.8 tỷ/năm

Kịch bản 2: MDR = 0.5% (đàm phán được)
→ Phí payment: 50,000,000đ/tháng = 600 triệu/năm

Chênh lệch: 1.2 tỷ/năm
```

Đó là lý do merchant lớn **rất chú ý đến MDR** và luôn đàm phán. Với doanh nghiệp margin thấp như F&B, grocery, 1% MDR có thể là sự khác biệt giữa có lãi và hòa vốn.

### 4.4 Các mô hình định giá MDR

#### Flat Rate (Đồng giá)
Tất cả giao dịch cùng một mức phí, bất kể phương thức hay loại thẻ.
- Ưu điểm: Đơn giản, dễ dự đoán chi phí
- Ví dụ: Stripe charge 2.9% + $0.30 mọi giao dịch tại Mỹ
- Phù hợp: Merchant nhỏ, không muốn phức tạp

#### Interchange Plus (Cost Plus)
MDR = Interchange thực tế + markup cố định của provider.
- Ưu điểm: Merchant biết chính xác mình trả bao nhiêu cho mỗi loại thẻ
- Ví dụ: Interchange + 0.3% + 1,000đ/giao dịch
- Phù hợp: Enterprise, volume lớn, muốn tối ưu chi phí

#### Tiered Pricing (Phân tầng)
Chia thành 3 mức: Qualified / Mid-qualified / Non-qualified dựa trên loại thẻ và cách giao dịch.
- Phức tạp, dễ gây nhầm lẫn
- Provider đôi khi dùng để ẩn chi phí thực

#### Volume-based Discount
Càng giao dịch nhiều, MDR càng thấp.
```
Doanh thu < 1 tỷ/tháng:   MDR = 1.5%
1 tỷ – 5 tỷ/tháng:        MDR = 1.2%
> 5 tỷ/tháng:              MDR = 0.9%
```

---

## 5. Merchant Onboarding — Tại sao lại phức tạp?

Onboarding không chỉ là "tạo tài khoản". Nó là quy trình **thẩm định rủi ro** trước khi Provider đồng ý nhận tiền hộ merchant.

### 5.1 Tại sao Provider phải thận trọng?

Khi một merchant gian lận hoặc bị chargeback nhiều:
- **Provider chịu trách nhiệm** với acquirer và card network
- Provider có thể bị **phạt tiền** hoặc **mất license**
- Tiền đã settlement cho merchant → Provider phải **bù từ túi mình** nếu không đòi lại được

Đó là lý do onboarding merchant là **quy trình kiểm soát rủi ro**, không chỉ là bán hàng.

### 5.2 Các thông tin cần thu thập khi onboarding

#### Thông tin cơ bản về doanh nghiệp
- Tên doanh nghiệp, MST (mã số thuế)
- Loại hình doanh nghiệp (TNHH, Cổ phần, Hộ KD, Cá nhân)
- Ngành nghề kinh doanh → xác định MCC
- Địa chỉ đăng ký kinh doanh
- Website/app/fanpage (nếu bán online)

#### Thông tin người đại diện / chủ sở hữu
- CCCD/Hộ chiếu
- Địa chỉ thường trú
- Tỷ lệ sở hữu (nếu > 25% phải khai báo — quy định AML)

#### Giấy tờ pháp lý (KYB documents)
- Giấy chứng nhận đăng ký kinh doanh
- Giấy phép kinh doanh chuyên ngành (nếu có — ví dụ: dược phẩm, tài chính)
- Điều lệ công ty
- Biên bản họp HĐQT ủy quyền (nếu người ký không phải giám đốc)

#### Thông tin tài chính
- Tài khoản ngân hàng nhận settlement (phải đứng tên doanh nghiệp)
- Doanh thu dự kiến/tháng
- Giá trị giao dịch trung bình
- Lĩnh vực kinh doanh chính

#### Thông tin sản phẩm/dịch vụ
- Mô tả chi tiết sản phẩm/dịch vụ bán
- Chính sách hoàn/hủy (refund policy)
- Thời gian giao hàng (nếu là ecommerce)
- Website đang hoạt động thực sự không?

### 5.3 Quy trình thẩm định (Underwriting)

Sau khi nhận đủ hồ sơ, Provider thực hiện **underwriting** — đánh giá rủi ro của merchant.

#### Kiểm tra tự động (Auto checks)
```
✓ Blacklist check: Merchant/người đại diện có trong danh sách cấm không?
  (OFAC, local regulatory blacklist, internal blacklist)

✓ Duplicate check: Tài khoản ngân hàng/MST đã đăng ký trước đây chưa?
  (Tránh một người lập nhiều merchant để bypass limit)

✓ Business verification: MST có tồn tại trên cổng thông tin thuế?

✓ Website check (nếu online merchant):
  - Website có hoạt động không?
  - Có bán đúng sản phẩm đã khai không?
  - Có đủ thông tin liên hệ, chính sách hoàn hàng không?

✓ MCC risk check: Ngành nghề có thuộc high-risk category không?
```

#### Kiểm tra thủ công (Manual review)
Với merchant có doanh thu lớn hoặc ngành nghề nhạy cảm, nhân viên review trực tiếp:
- Xác minh giấy tờ pháp lý (ảnh thật? Chưa hết hạn?)
- Gọi điện xác nhận thông tin
- Research doanh nghiệp trên Google, mạng xã hội
- Đánh giá rủi ro chargeback và fraud

#### Kết quả underwriting
- **Approve**: Merchant được cấp MID, bắt đầu nhận giao dịch
- **Approve with conditions**: Được approve nhưng với giới hạn thấp hơn, rolling reserve cao hơn
- **Request more info**: Cần bổ sung hồ sơ
- **Reject**: Từ chối, lý do được ghi lại để tránh merchant "vòng vo" đăng ký lại

### 5.4 High-Risk Merchants — Danh mục ngành nghề rủi ro cao

Một số ngành nghề được card network và acquirer xếp vào **high-risk**, yêu cầu thẩm định chặt hơn, MDR cao hơn, và rolling reserve bắt buộc.

| Mức độ rủi ro | Ngành nghề | Lý do |
|---|---|---|
| **Cực cao** (thường bị từ chối) | Cờ bạc, vũ khí, nội dung người lớn, tiền mã hóa không rõ nguồn gốc | Vi phạm pháp luật hoặc scheme rules |
| **Cao** | Dịch vụ du lịch, vé máy bay | Thời gian giữa thanh toán và nhận dịch vụ dài → chargeback cao |
| **Cao** | Thực phẩm chức năng, thuốc | Dễ bị khiếu nại về chất lượng |
| **Cao** | Subscription/recurring | Customer quên đăng ký → khiếu nại "không biết bị trừ tiền" |
| **Trung bình** | Thương mại điện tử chung | Không gặp mặt trực tiếp → gian lận cao hơn |
| **Thấp** | Siêu thị, nhà hàng, xăng dầu | Giao dịch trực tiếp, hàng hóa rõ ràng |

### 5.5 Merchant Tiering sau onboarding

Sau khi được approve, merchant được xếp **tier** ảnh hưởng đến MDR, giới hạn, và mức độ hỗ trợ:

```
TIER 1 — Standard (mặc định mới vào)
  MDR: Giá niêm yết
  Transaction limit: 10 triệu/giao dịch
  Daily limit: 100 triệu/ngày
  Support: Ticket/email

TIER 2 — Preferred (sau 3-6 tháng, volume tốt)
  MDR: Giảm 0.2-0.3%
  Transaction limit: 50 triệu/giao dịch
  Daily limit: 500 triệu/ngày
  Support: Chat + email priority

TIER 3 — Premium / Enterprise
  MDR: Đàm phán riêng
  Transaction limit: Theo thỏa thuận
  Daily limit: Theo thỏa thuận
  Support: Dedicated account manager
```

---

## 6. Vòng đời Merchant (Merchant Lifecycle)

### 6.1 Toàn bộ vòng đời

```
REGISTERED ──► UNDER REVIEW ──► ACTIVE ──► [hoạt động bình thường]
                    │                │
                    ▼                ├──► UNDER WATCH (cảnh báo)
                REJECTED             │         │
                                     │         ├──► ACTIVE (hết cảnh báo)
                                     │         └──► SUSPENDED
                                     │                   │
                                     └──► SUSPENDED ──────┼──► ACTIVE (khắc phục xong)
                                                          └──► TERMINATED
```

### 6.2 Các lý do merchant bị Suspended (Tạm khóa)

Suspended có nghĩa là **không nhận được giao dịch mới**, nhưng settlement của giao dịch cũ vẫn tiếp tục (sau khi trừ các khoản phạt/reserve).

**Lý do phổ biến:**

| Lý do | Ngưỡng thường gặp |
|---|---|
| Chargeback rate vượt ngưỡng | > 1% (Visa) hoặc > 1.5% (Mastercard) |
| Fraud rate cao đột biến | Hệ thống phát hiện pattern bất thường |
| Vi phạm Terms of Service | Bán hàng cấm, lừa dối customer |
| Hồ sơ KYB hết hạn | GPKD hoặc CCCD người đại diện hết hạn |
| Tài khoản ngân hàng bị đóng | Không thể settlement |
| Nghi ngờ rửa tiền | AML alert trigger |
| Chưa cập nhật thông tin theo yêu cầu | Compliance update |

### 6.3 Terminated — Chấm dứt hợp đồng vĩnh viễn

Merchant bị terminated thường vì:
- Chargeback rate liên tục vượt ngưỡng sau nhiều lần cảnh báo
- Phát hiện gian lận, rửa tiền, hoặc bán hàng cấm
- Tự ý chấm dứt hợp đồng

**Hệ quả nghiêm trọng:** Merchant bị terminated có thể bị đưa vào **MATCH List** (Mastercard Alert to Control High-risk Merchants) hoặc **TMF** (Terminated Merchant File) — blacklist toàn cầu của card network.

Một khi vào MATCH List:
- Không một acquirer nào trên thế giới dám cấp merchant account
- Thời gian tồn tại trong list: 5 năm
- Rất khó kháng cáo

---

## 7. Settlement — Cơ chế chuyển tiền cho merchant

Settlement là một trong những khái niệm quan trọng nhất cần hiểu rõ.

### 7.1 Tại sao tiền không về ngay?

Giả sử merchant bán một món hàng lúc 9h sáng. Tại sao phải chờ đến ngày hôm sau mới nhận tiền?

**Lý do 1: Chargeback window**

Customer có thể khiếu nại và đòi hoàn tiền trong vòng **60–120 ngày** sau giao dịch. Nếu Provider settlement ngay, rồi customer khiếu nại, Provider sẽ phải bỏ tiền túi ra hoàn trả.

**Lý do 2: Xử lý batch của ngân hàng**

Hệ thống thanh toán ngân hàng (NAPAS, Vietcombank) xử lý thanh toán theo **batch**, không real-time 24/7.

**Lý do 3: Kiểm tra gian lận**

Cho thêm thời gian để hệ thống phát hiện các giao dịch gian lận trước khi tiền đến tay merchant.

### 7.2 Các chu kỳ Settlement phổ biến

| Cycle | Giải thích | Dùng khi nào |
|---|---|---|
| **T+0** (same day) | Giao dịch hôm nay → tiền về chiều tối hôm nay | Merchant VIP, phí cao hơn |
| **T+1** | Giao dịch hôm nay → tiền về ngày mai | Phổ biến nhất |
| **T+2** | Giao dịch hôm nay → tiền về ngày kia | Thẻ quốc tế, một số acquirer |
| **Weekly** | Tiền về mỗi tuần một lần | Merchant nhỏ, low volume |

**Ngày làm việc hay ngày calendar?**

Quan trọng: T+1 nghĩa là **T+1 ngày làm việc**, không phải ngày calendar.

```
Giao dịch thứ Sáu (T)
→ Settlement thứ Hai (T+1)
(Thứ 7, Chủ nhật không tính)
```

### 7.3 Cách tính Settlement

```
Settlement Amount = Gross Sales
                  - MDR Fees
                  - Refunds (của kỳ này)
                  - Chargebacks (nếu có)
                  - Rolling Reserve (giữ lại)
                  + Reserve Released (trả lại reserve cũ)
```

**Ví dụ cụ thể:**

```
Kỳ settlement: 01/12/2024

Giao dịch thành công:    200 giao dịch
Gross Sales:             80,000,000đ
MDR (1.5%):              -1,200,000đ
Refunds:                   -500,000đ
Chargeback:                       0đ
Rolling Reserve (10%):   -8,000,000đ
Reserve Released:         +3,000,000đ  (tiền giữ từ 90 ngày trước)
─────────────────────────────────────
Net Settlement:          73,300,000đ
```

### 7.4 Rolling Reserve — Chi tiết

Rolling Reserve là khoản tiền Provider **giữ lại** như một "quỹ bảo hiểm rủi ro".

**Cơ chế hoạt động:**

```
Ngày 1:  Giữ lại 10% = 10,000đ  (trả lại ngày 91)
Ngày 2:  Giữ lại 10% = 8,000đ   (trả lại ngày 92)
Ngày 3:  Giữ lại 10% = 12,000đ  (trả lại ngày 93)
...
Ngày 91: Nhận lại 10,000đ từ ngày 1
Ngày 92: Nhận lại 8,000đ từ ngày 2
```

Sau 90 ngày đầu tiên, merchant vừa bị giữ tiền mới, vừa nhận lại tiền cũ → **ổn định về sau**.

**Mức reserve phụ thuộc vào:**
- Ngành nghề (high-risk → reserve cao hơn)
- Lịch sử chargeback
- Merchant mới (chưa có lịch sử → reserve cao hơn)
- Mức độ tin cậy của merchant

**Reserve thông thường:**

| Loại merchant | Reserve rate | Thời gian giữ |
|---|---|---|
| Merchant standard, lịch sử tốt | 0% – 5% | 90 ngày |
| Merchant mới | 5% – 10% | 90 ngày |
| High-risk merchant | 10% – 15% | 120 – 180 ngày |
| Merchant đang bị điều tra | Lên đến 100% | Không xác định |

---

## 8. Chargeback — Vấn đề đau đầu nhất của merchant

### 8.1 Chargeback là gì và tại sao nó khác refund?

**Refund** là khi merchant **tự nguyện** hoàn tiền cho customer.
**Chargeback** là khi customer **bypass merchant**, khiếu nại thẳng với ngân hàng và ngân hàng **buộc** thu tiền lại từ merchant.

```
REFUND:
Customer khiếu nại với Merchant
→ Merchant đồng ý hoàn tiền
→ Merchant chủ động refund

CHARGEBACK:
Customer khiếu nại với Issuer Bank
→ Issuer điều tra
→ Issuer buộc Acquirer lấy lại tiền
→ Acquirer trừ tiền từ tài khoản merchant
→ Merchant không có quyền từ chối (trừ khi dispute thành công)
```

### 8.2 Các lý do customer thường dùng để chargeback

| Lý do (Reason Code) | Ý nghĩa thực tế | Tần suất |
|---|---|---|
| **Item not received** | Không nhận được hàng | Rất cao |
| **Item not as described** | Hàng khác với mô tả | Cao |
| **Unauthorized transaction** | "Tôi không thực hiện giao dịch này" (thẻ bị đánh cắp) | Cao |
| **Credit not processed** | Đã refund nhưng tiền chưa về | Trung bình |
| **Duplicate processing** | Bị trừ tiền 2 lần | Thấp |
| **Friendly fraud** | Customer đã nhận hàng nhưng vẫn claim "không nhận được" | **Ngày càng phổ biến** |

### 8.3 Friendly Fraud — Vấn đề ngày càng lớn

**Friendly fraud** (còn gọi là **chargeback fraud**) xảy ra khi customer:
- Nhận được hàng/dịch vụ bình thường
- Nhưng vẫn khiếu nại với ngân hàng là "không nhận được" hoặc "không thực hiện giao dịch"
- Mục đích: lấy lại tiền mà vẫn giữ hàng

Đây là vấn đề cực kỳ khó xử lý vì:
- Issuer bank thường **đứng về phía customer** (customer mà!)
- Merchant phải có bằng chứng rất rõ ràng để dispute thành công

### 8.4 Chargeback Process — Từng bước

```
[1] Customer gọi điện cho Issuer Bank
    "Tôi không nhận được hàng, tôi muốn khiếu nại"

[2] Issuer Bank mở dispute
    Tạm thời hoàn tiền cho customer (provisional credit)

[3] Issuer gửi chargeback request đến Card Network (Visa/Master)
    Card Network forward đến Acquirer

[4] Acquirer nhận chargeback
    Trừ tiền từ Merchant Account ngay lập tức
    Notify cho Payment Gateway

[5] Gateway notify Merchant
    "Bạn nhận được chargeback #CB-2024-XXX"
    "Số tiền: 500,000đ"
    "Lý do: Item not received"
    "Hạn để dispute: 20 ngày"

[6] Merchant có 2 lựa chọn:

    OPTION A: Chấp nhận chargeback
    → Không làm gì → tiền mất → record chargeback

    OPTION B: Dispute (kháng cáo)
    → Cung cấp bằng chứng trong thời hạn:
      - Proof of delivery (biên lai giao hàng)
      - Hợp đồng/điều khoản đã ký
      - Communication với customer
      - IP address, device fingerprint
      - Tracking number

[7] Acquirer review bằng chứng → gửi lên Card Network

[8] Card Network ra quyết định
    WIN: Tiền trả lại cho merchant
    LOSE: Tiền mất vĩnh viễn, thêm chargeback fee
```

### 8.5 Chargeback Ratio — Ngưỡng nguy hiểm

Card network theo dõi **chargeback ratio** của từng merchant mỗi tháng:

```
Chargeback Ratio = Số chargeback trong tháng / Số transaction trong tháng
```

**Ngưỡng cảnh báo của Visa:**

| Ratio | Hậu quả |
|---|---|
| < 0.65% | Bình thường |
| 0.65% – 0.9% | **Early Warning** — nhận cảnh báo |
| 0.9% – 1.8% | **Standard Monitoring Program** — phạt $50/chargeback |
| > 1.8% | **Excessive Chargeback Program** — phạt $300/chargeback, nguy cơ mất merchant account |

### 8.6 Cách merchant tự bảo vệ khỏi chargeback

**Phòng ngừa trước giao dịch:**
- Mô tả sản phẩm rõ ràng, ảnh thực tế
- Chính sách hoàn hàng rõ ràng, dễ tìm
- Tên hiển thị trên sao kê ngân hàng phải nhận ra được (không dùng tên công ty pháp lý xa lạ)
- 3DS (3D Secure) cho giao dịch online → chuyển rủi ro chargeback sang issuer

**Khi xử lý giao dịch:**
- Lưu đầy đủ log: IP, device, browser fingerprint
- Xác nhận email + đường dẫn tracking
- Với hàng giá trị cao: yêu cầu chữ ký khi nhận

**Khi nhận chargeback:**
- Respond đúng hạn (đừng để quá deadline)
- Chuẩn bị bằng chứng có tổ chức, rõ ràng
- Với chargeback nhỏ (< 100,000đ): đôi khi accept còn rẻ hơn tranh cãi

---

## 9. KYC / KYB — Xác minh để bảo vệ hệ thống

### 9.1 Tại sao phải KYC/KYB?

Không chỉ là quy định pháp luật. Đây là **bảo vệ cả hệ thống**:

- **AML (Anti-Money Laundering)**: Tránh merchant dùng hệ thống để rửa tiền
- **CTF (Counter-Terrorism Financing)**: Tránh tài trợ khủng bố
- **Fraud Prevention**: Tránh kẻ xấu lập merchant ảo để lừa đảo
- **Regulatory Compliance**: NHNN Việt Nam bắt buộc

### 9.2 KYC cho cá nhân vs KYB cho doanh nghiệp

**KYC (Know Your Customer) — Cá nhân:**

```
Bắt buộc:
✓ CCCD/CMND/Hộ chiếu (còn hạn)
✓ Selfie với CCCD (liveness check)
✓ Địa chỉ thường trú

Bổ sung cho merchant cá nhân doanh thu lớn:
✓ Sao kê ngân hàng 3 tháng gần nhất
✓ Chứng minh nguồn thu nhập
```

**KYB (Know Your Business) — Doanh nghiệp:**

```
Bắt buộc:
✓ Giấy chứng nhận đăng ký doanh nghiệp (GPKD)
✓ MST (mã số thuế)
✓ KYC của người đại diện pháp luật
✓ KYC của các cổ đông sở hữu > 25%

Theo ngành nghề:
✓ Giấy phép chuyên ngành (nếu cần):
  - Y tế: giấy phép dược/bệnh viện
  - Tài chính: giấy phép NHNN
  - Giáo dục: giấy phép Bộ GD

Bổ sung cho doanh nghiệp lớn:
✓ Báo cáo tài chính 2 năm gần nhất (hoặc được kiểm toán)
✓ Sơ đồ cơ cấu sở hữu (ownership structure)
✓ Danh sách UBO (Ultimate Beneficial Owner — người thực sự kiểm soát)
```

### 9.3 UBO (Ultimate Beneficial Owner) — Điều ít người biết

**UBO** là người thực sự hưởng lợi từ doanh nghiệp — không phải chỉ người đứng tên.

Ví dụ cấu trúc phức tạp:
```
Merchant: Công ty A (Việt Nam)
  └─► 70% sở hữu bởi Công ty B (Singapore)
            └─► 60% sở hữu bởi Ông Nguyễn Văn X (cá nhân)

→ UBO thực sự là Ông Nguyễn Văn X (sở hữu 70% × 60% = 42% > 25%)
→ Phải KYC Ông X dù ông không đứng tên Công ty A
```

Tại sao quan trọng? Kẻ xấu thường che giấu danh tính sau các lớp công ty shell.

### 9.4 Ongoing KYC — Không chỉ làm một lần

KYC/KYB không kết thúc sau onboarding. Provider phải duy trì:

- **Kiểm tra định kỳ**: Re-verify giấy tờ khi hết hạn (GPKD gia hạn hàng năm, CCCD 15 năm/lần)
- **Event-triggered review**: Khi phát hiện thay đổi bất thường (doanh thu tăng đột biến, ngành nghề thay đổi)
- **AML screening**: Quét liên tục danh sách blacklist quốc tế
- **Transaction monitoring**: Theo dõi pattern giao dịch bất thường

---

## 10. Merchant Risk Management

### 10.1 Risk Score — Đánh giá rủi ro merchant

Mỗi merchant được gán một **risk score** dựa trên nhiều yếu tố:

```
Risk Score = f(
  Ngành nghề (MCC risk),
  Lịch sử chargeback,
  Lịch sử fraud,
  Thời gian hoạt động,
  Volume giao dịch vs dự kiến ban đầu,
  Pattern giao dịch (giờ giấc, địa lý),
  Tình trạng pháp lý,
  Kết quả AML screening
)
```

### 10.2 Các dấu hiệu rủi ro (Red Flags)

**Dấu hiệu gian lận từ phía merchant:**

```
🚩 Doanh thu thực tế cao hơn nhiều so với khai báo ban đầu
   → Có thể đang làm merchant hộ người khác (third-party merchant)

🚩 Nhiều giao dịch giá trị nhỏ liên tiếp, sau đó một giao dịch lớn
   → Kỹ thuật "structuring" để tránh ngưỡng kiểm tra

🚩 Giao dịch vào giờ bất thường (2-4 giờ sáng liên tục)

🚩 Nhiều thẻ khác nhau giao dịch từ cùng IP/device
   → Card testing

🚩 Refund rate đột ngột tăng cao
   → Có thể đang rửa tiền qua cơ chế refund

🚩 Ngành nghề khai báo là "tư vấn" nhưng giao dịch là mua đi bán lại
   → Khai báo sai MCC
```

**Dấu hiệu kỹ thuật rửa tiền qua merchant:**

```
SCHEME: Structuring (smurfing)
- Chia tiền bẩn thành nhiều giao dịch nhỏ dưới ngưỡng báo cáo
- Ví dụ: 50 giao dịch 9,900,000đ thay vì 1 giao dịch 495,000,000đ

SCHEME: Refund laundering
- Giao dịch hợp lệ → rút tiền qua refund về thẻ khác
- Thẻ khác là thẻ của đồng phạm

SCHEME: Ghost merchant
- Lập merchant ảo, không bán gì thật
- Customer thực ra là đồng phạm, "mua hàng" giả
- Tiền bẩn → qua giao dịch → thành tiền settlement hợp lệ
```

### 10.3 Transaction Monitoring Rules (Ví dụ)

Provider thường có các rule tự động để phát hiện bất thường:

```
RULE 1: Single large transaction
  IF transaction_amount > 3x average_transaction_amount
  THEN flag for review

RULE 2: Velocity check
  IF transactions_per_hour > 50  (bất thường với merchant nhỏ)
  THEN flag + notify risk team

RULE 3: Chargeback spike
  IF chargeback_this_week > 2x average_chargeback_per_week
  THEN escalate to risk team

RULE 4: Night time anomaly
  IF hour IN (01:00-05:00)
  AND transaction_count > 10
  AND merchant_category NOT IN ('pharmacy', '24h_store', 'gas_station')
  THEN flag for review

RULE 5: Multiple cards same device
  IF DISTINCT(card_fingerprint) > 5
  AND time_window = '1 hour'
  AND same device_fingerprint
  THEN auto-suspend + alert
```

---

## 11. Merchant Experience — Nhìn từ góc độ merchant

Hiểu business của merchant cũng có nghĩa là hiểu họ quan tâm đến điều gì.

### 11.1 Những gì merchant quan tâm nhất

**1. Tỷ lệ giao dịch thành công (Success Rate / Approval Rate)**

```
Success Rate = Giao dịch completed / Tổng giao dịch attempted

Merchant mong đợi: > 95%
Dưới 90%: Merchant sẽ cân nhắc chuyển provider
```

Mỗi 1% decline = tiền mất thật. Với merchant doanh thu 1 tỷ/tháng, success rate 95% vs 98% = **30 triệu đồng/tháng chênh lệch**.

**2. Tốc độ settlement**

Merchant nhỏ cần tiền để tái đầu tư. T+1 vs T+2 với họ là rất khác nhau.

**3. MDR và transparency**

Merchant không thích bị ẩn phí. Họ muốn biết chính xác mỗi giao dịch mất bao nhiêu.

**4. Dashboard và reporting**

Merchant cần biết:
- Hôm nay thu bao nhiêu?
- Settlement đang ở đâu?
- Chargeback nào đang pending?
- Giao dịch nào failed và lý do gì?

**5. Hỗ trợ khi có vấn đề**

Khi có sự cố (giao dịch failed bất thường, settlement trễ), merchant muốn được phản hồi **trong vòng 1 giờ**, không phải 24 giờ.

### 11.2 Lý do merchant rời bỏ provider

```
Xếp hạng phổ biến:

#1: Success rate thấp / Tỷ lệ decline cao
    → Mất doanh thu trực tiếp

#2: MDR cao hơn đối thủ
    → Sau khi so sánh, merchant chuyển provider

#3: Settlement trễ / Không minh bạch
    → Dòng tiền bị ảnh hưởng

#4: Hỗ trợ chậm
    → Khi có sự cố, merchant cần phản hồi nhanh

#5: Dashboard / API tệ
    → Khó tích hợp, khó theo dõi
```

---

## 12. Merchant Segments trong thực tế Việt Nam

### 12.1 Đặc điểm thị trường Việt Nam

- **QR Code** là phương thức tăng trưởng nhanh nhất (hỗ trợ bởi chính sách không dùng tiền mặt của NHNN)
- **NAPAS** là hạ tầng thanh toán nội địa, QR NAPAS có MDR ưu đãi
- Nhiều merchant vừa và nhỏ vẫn ngại vì không hiểu cơ chế phí và settlement
- **Ví điện tử** (Momo, ZaloPay, VNPay) cạnh tranh mạnh với payment gateway truyền thống

### 12.2 Phân loại merchant theo ngành ở Việt Nam

**F&B (Food & Beverage)**
- Volume cao, giá trị trung bình thấp (50,000 – 200,000đ/giao dịch)
- QR là chính, POS phụ
- Quan tâm nhiều đến: tốc độ in receipt, tỷ lệ thành công, MDR thấp
- Pain point: Peak hours (12h, 18-19h) → cần hệ thống ổn định

**Retail / Fashion**
- Volume trung bình, giá trị cao hơn
- Cần omnichannel: vừa có POS tại shop, vừa có online
- Quan tâm: reconciliation dễ dàng, báo cáo tổng hợp

**E-commerce**
- Hoàn toàn online, cần tích hợp API
- Rủi ro fraud cao hơn → cần 3DS, risk scoring
- Quan tâm: success rate, checkout experience mượt mà

**Travel / Vé máy bay / Khách sạn**
- Giá trị giao dịch cao (vài triệu đến vài chục triệu)
- Thời gian giữa thanh toán và dùng dịch vụ dài → chargeback risk cao
- Cần hold payment / pre-authorization

**Giáo dục / Học phí**
- Thanh toán theo kỳ, giá trị lớn
- Cần installment (trả góp) hoặc partial payment
- Ít chargeback

---

## 13. Tổng kết — Mental Model cho Merchant Domain

### Nhìn merchant như thế nào cho đúng?

```
Merchant là KHÁCH HÀNG B2B của bạn.
Họ trả MDR để bạn giải quyết vấn đề cho họ:
  → Nhận tiền từ khách hàng một cách an toàn
  → Không cần tự xây dựng hạ tầng thanh toán
  → Tập trung vào kinh doanh cốt lõi

Bạn kiếm tiền khi: Merchant thành công → volume cao → MDR nhiều hơn
Bạn mất tiền khi: Chargeback, fraud, merchant rời bỏ
```

### Framework để hiểu bất kỳ tình huống nào

Khi gặp bất kỳ vấn đề nào trong merchant domain, hỏi 3 câu:

**1. Ai chịu rủi ro ở đây?**
→ Merchant? Provider? Acquirer? Customer?

**2. Dòng tiền đi như thế nào?**
→ Tiền từ customer → qua ai → giữ ở đâu → đến merchant khi nào?

**3. Ai có thể bị thiệt hại nếu xảy ra sự cố?**
→ Hiểu điều này = hiểu tại sao có rolling reserve, chargeback fee, KYB, v.v.

---

### Bảng thuật ngữ nhanh

| Thuật ngữ | Ý nghĩa một câu |
|---|---|
| **Merchant** | Đơn vị bán hàng, nhận tiền qua hệ thống payment |
| **Acquirer** | Ngân hàng đứng ra bảo lãnh cho merchant với card network |
| **Issuer** | Ngân hàng phát hành thẻ cho customer |
| **MDR** | Phí % merchant trả trên mỗi giao dịch thành công |
| **Settlement** | Chuyển tiền từ merchant account về ngân hàng của merchant |
| **Rolling Reserve** | Tiền giữ lại làm quỹ bảo hiểm, trả lại sau 90 ngày |
| **Chargeback** | Khách hàng khiếu nại ngân hàng, bắt buộc hoàn tiền |
| **Friendly Fraud** | Customer nhận hàng rồi vẫn khiếu nại để lấy lại tiền |
| **KYC/KYB** | Xác minh danh tính merchant trước khi cho phép hoạt động |
| **UBO** | Người thực sự kiểm soát doanh nghiệp (đằng sau các lớp công ty) |
| **MID** | Mã định danh duy nhất của merchant trong hệ thống |
| **MCC** | Mã ngành nghề 4 chữ số, ảnh hưởng đến MDR và rủi ro |
| **MATCH List** | Blacklist toàn cầu của card network cho merchant bị terminate |
| **Underwriting** | Quy trình thẩm định rủi ro trước khi approve merchant |
| **Interchange** | Phí từ acquirer trả cho issuer trên mỗi giao dịch |
| **Success Rate** | % giao dịch thành công, chỉ số quan trọng nhất với merchant |