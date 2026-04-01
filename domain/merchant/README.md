# 🏪 Merchant Domain — Payment / Fintech

> Tài liệu này giải thích toàn bộ domain **Merchant** trong hệ thống Payment / Fintech: từ khái niệm cơ bản, business model, data model, luồng nghiệp vụ đến tích hợp hệ thống.

---

## 1. Merchant là gì?

**Merchant** (đơn vị chấp nhận thanh toán) là **cá nhân hoặc doanh nghiệp bán hàng/dịch vụ** và sử dụng hệ thống payment để **nhận tiền từ khách hàng**.

### Ví dụ thực tế

| Merchant | Loại | Dùng payment để làm gì |
|---|---|---|
| Cửa hàng trà sữa | Offline | Nhận QR Code, POS |
| Shop thời trang Shopee | Online | Nhận thanh toán đơn hàng |
| Ứng dụng gọi xe | Platform | Thu tiền từ hành khách, trả cho tài xế |
| SaaS subscription | Online | Thu phí hàng tháng tự động |
| Siêu thị lớn | Offline | POS, thẻ, QR |

### Phân biệt các bên trong hệ sinh thái

```
Cardholder / Customer      Merchant              Acquirer            Issuer
(Người mua)            (Người bán)         (Ngân hàng của        (Ngân hàng của
                                            merchant)              khách hàng)
      │                     │                    │                    │
      │  ── thanh toán ──>  │                    │                    │
      │                     │  ── request ──>    │  ── verify ──>    │
      │                     │                    │  <── approve ──   │
      │                     │  <── approved ──   │                    │
      │  <── receipt ──     │                    │                    │
```

- **Customer/Cardholder**: Người mua — trả tiền
- **Merchant**: Người bán — nhận tiền
- **Acquirer**: Ngân hàng/tổ chức quản lý tài khoản của merchant (VD: VietcomBank, VNPAY)
- **Issuer**: Ngân hàng phát hành thẻ cho khách hàng (VD: Techcombank)
- **Payment Gateway**: Cầu nối kỹ thuật giữa merchant và acquirer (VD: VNPAY, PayOS, Stripe)
- **PSP (Payment Service Provider)**: Cung cấp toàn bộ dịch vụ payment cho merchant (gateway + acquirer + settlement)

---

## 2. Business Model — Merchant trong hệ thống Payment

### 2.1 Cách merchant kiếm tiền và hệ thống kiếm tiền từ merchant

```
Customer trả 100,000đ
         │
         ▼
    Payment Gateway
         │
    Trừ MDR (fee)
         │
         ▼
  Merchant nhận ~98,500đ
```

**MDR (Merchant Discount Rate)**: Phí merchant trả cho mỗi giao dịch thành công.

Ví dụ MDR = 1.5%:
- Giao dịch 100,000đ → Merchant nhận 98,500đ
- Payment provider giữ 1,500đ

### 2.2 Các loại phí merchant phải trả

| Loại phí | Mô tả | Ví dụ |
|---|---|---|
| **MDR** | % trên mỗi giao dịch | 0.5% – 3% tùy phương thức |
| **Setup fee** | Phí mở tài khoản | Miễn phí hoặc vài triệu |
| **Monthly fee** | Phí duy trì hàng tháng | 0 – 500,000đ |
| **Chargeback fee** | Phí khi bị khiếu nại hoàn tiền | 150,000 – 500,000đ/vụ |
| **Payout fee** | Phí rút tiền về tài khoản ngân hàng | 0 – 11,000đ/lần |

### 2.3 Các mô hình merchant phổ biến

#### Direct Merchant
Merchant bán trực tiếp cho end-user. Luồng đơn giản nhất.

```
Customer → [Thanh toán] → Merchant
```

#### Marketplace / Platform
Merchant bán trên nền tảng trung gian. Platform thu hộ rồi phân chia.

```
Customer → [Thanh toán] → Platform → [Split] → Merchant A
                                              → Merchant B
                                              → Platform fee
```
Ví dụ: Shopee, Grab, Tiki

#### Sub-merchant (Payment Facilitation)
Một merchant lớn (PayFac) đứng ra đăng ký thay cho nhiều merchant nhỏ.

```
Customer → [Thanh toán] → PayFac (Master Merchant)
                               → Sub-merchant A
                               → Sub-merchant B
```
Ví dụ: Square, Stripe Connect — cho phép platform tạo merchant con.

---

## 3. Các khái niệm cốt lõi

### 3.1 Merchant Account vs Business Account

| | Merchant Account | Business Account |
|---|---|---|
| Mục đích | Nhận thanh toán từ card/QR | Tài khoản ngân hàng thông thường |
| Ai cấp | Acquirer / PSP | Ngân hàng |
| Chứa tiền? | Tạm thời (pending settlement) | Có |

### 3.2 Settlement (Thanh toán định kỳ)

Tiền từ giao dịch **không về ngay** tài khoản ngân hàng của merchant. Nó được **giữ lại** và **settlement** (chuyển) định kỳ.

```
Giao dịch thành công (T)
        │
        ▼
  Tiền vào holding pool
        │
   T+1 hoặc T+2 ngày
        │
        ▼
  Settlement: chuyển tiền về tài khoản ngân hàng merchant
```

**Tại sao có độ trễ?** Để xử lý refund, chargeback, kiểm tra gian lận.

### 3.3 Rolling Reserve

Hệ thống **giữ lại một phần tiền** của merchant (thường 5–10%) trong một khoảng thời gian nhất định để đảm bảo nếu có chargeback thì có tiền hoàn trả.

```
Giao dịch 1,000,000đ
├── 90% → Settlement ngay → 900,000đ về tài khoản merchant
└── 10% → Reserve 90 ngày → 100,000đ giải phóng sau 3 tháng
```

### 3.4 Chargeback

Khách hàng khiếu nại với ngân hàng phát hành thẻ → ngân hàng **buộc hoàn tiền** về cho khách, không cần merchant đồng ý.

```
Customer khiếu nại "không nhận được hàng"
        │
        ▼
Issuer bank (ngân hàng khách hàng)
        │
        ▼
Acquirer nhận thông báo chargeback
        │
        ▼
Trừ tiền từ tài khoản merchant
        │
Merchant có quyền dispute (tranh chấp) bằng bằng chứng
```

Chargeback rate cao → merchant có thể bị **terminate** (đóng tài khoản).

### 3.5 KYC / KYB (Know Your Customer / Know Your Business)

Quy trình **xác minh danh tính** merchant trước khi được phép nhận thanh toán.

| | KYC | KYB |
|---|---|---|
| Đối tượng | Cá nhân | Doanh nghiệp |
| Giấy tờ | CMND/CCCD, selfie | Giấy phép KD, MST, giấy tờ người đại diện |
| Mục đích | Chống giả mạo, rửa tiền | Chống shell company, fraud |

### 3.6 MID (Merchant ID)

Mã định danh duy nhất của merchant trong hệ thống payment. Mỗi merchant có ít nhất một MID. Merchant lớn có thể có nhiều MID (mỗi kênh bán một MID riêng).

### 3.7 MCC (Merchant Category Code)

Mã 4 chữ số phân loại ngành nghề của merchant. Do Visa/Mastercard quy định.

| MCC | Ngành |
|---|---|
| 5411 | Grocery Stores (siêu thị) |
| 5812 | Restaurants |
| 4111 | Local and Suburban Commuter |
| 7011 | Hotels |

MCC ảnh hưởng đến: MDR rate, giới hạn giao dịch, phân loại rủi ro.

---

## 4. Data Model / Database Schema

### 4.1 Tổng quan các entity chính

```
┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐
│  Merchant   │────<│ MerchantLocation │     │ MerchantAccount │
└──────┬──────┘     └──────────────────┘     └────────┬────────┘
       │                                              │
       │            ┌──────────────────┐              │
       ├────────────│  MerchantConfig  │              │
       │            └──────────────────┘              │
       │                                              │
       │            ┌──────────────────┐     ┌────────┴────────┐
       └────────────│   Transaction    │────>│   Settlement    │
                    └──────────────────┘     └─────────────────┘
```

---

### 4.2 Merchant (Entity chính)

```sql
CREATE TABLE merchants (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    merchant_code   VARCHAR(20) UNIQUE NOT NULL,   -- MID: VN-2024-00123
    business_name   VARCHAR(255) NOT NULL,          -- Tên đăng ký kinh doanh
    display_name    VARCHAR(255),                   -- Tên hiển thị với khách hàng
    merchant_type   VARCHAR(50) NOT NULL,           -- individual | business | enterprise
    category_code   VARCHAR(10) NOT NULL,           -- MCC: 5812
    status          VARCHAR(30) NOT NULL DEFAULT 'pending',
                    -- pending | under_review | active | suspended | terminated

    -- KYC / KYB
    kyc_status      VARCHAR(30) DEFAULT 'not_started',
                    -- not_started | in_progress | submitted | approved | rejected
    kyc_verified_at TIMESTAMPTZ,

    -- Contact
    email           VARCHAR(255) NOT NULL,
    phone           VARCHAR(20),
    website         VARCHAR(500),

    -- Business address
    country_code    CHAR(2) NOT NULL DEFAULT 'VN',
    province        VARCHAR(100),
    district        VARCHAR(100),
    address         TEXT,

    -- Tier & Risk
    merchant_tier   VARCHAR(20) DEFAULT 'standard', -- standard | premium | enterprise
    risk_level      VARCHAR(20) DEFAULT 'medium',   -- low | medium | high
    chargeback_rate DECIMAL(5,4) DEFAULT 0,         -- 0.0150 = 1.5%

    -- Timestamps
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    activated_at    TIMESTAMPTZ,
    terminated_at   TIMESTAMPTZ
);
```

---

### 4.3 MerchantConfig (Cấu hình thương mại)

```sql
CREATE TABLE merchant_configs (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    merchant_id     UUID NOT NULL REFERENCES merchants(id),

    -- Fee configuration
    mdr_rate        DECIMAL(6,4) NOT NULL DEFAULT 0.0150, -- 1.50%
    mdr_fixed_fee   DECIMAL(12,2) DEFAULT 0,              -- Phí cố định/giao dịch (đồng)
    payout_fee      DECIMAL(12,2) DEFAULT 0,              -- Phí rút tiền

    -- Transaction limits
    min_txn_amount  DECIMAL(15,2) DEFAULT 1000,
    max_txn_amount  DECIMAL(15,2) DEFAULT 50000000,       -- 50 triệu/giao dịch
    daily_limit     DECIMAL(15,2) DEFAULT 500000000,      -- 500 triệu/ngày

    -- Settlement
    settlement_cycle VARCHAR(20) DEFAULT 'T+1',            -- T+0 | T+1 | T+2 | weekly
    settlement_time  TIME DEFAULT '08:00:00',              -- Giờ settlement mỗi ngày

    -- Rolling reserve
    reserve_rate    DECIMAL(5,4) DEFAULT 0,                -- 0.10 = 10%
    reserve_days    INTEGER DEFAULT 90,

    -- Payment methods được phép
    allowed_methods JSONB DEFAULT '["card","qr","bank_transfer"]',

    -- Webhook
    webhook_url     VARCHAR(500),
    webhook_secret  VARCHAR(255),

    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### 4.4 MerchantBankAccount (Tài khoản nhận settlement)

```sql
CREATE TABLE merchant_bank_accounts (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    merchant_id     UUID NOT NULL REFERENCES merchants(id),

    bank_code       VARCHAR(20) NOT NULL,   -- VCB, TCB, MB, ...
    bank_name       VARCHAR(255) NOT NULL,
    account_number  VARCHAR(50) NOT NULL,
    account_name    VARCHAR(255) NOT NULL,  -- Tên chủ tài khoản (phải khớp)
    branch          VARCHAR(255),

    is_primary      BOOLEAN DEFAULT FALSE,  -- Tài khoản settlement chính
    is_verified     BOOLEAN DEFAULT FALSE,  -- Đã xác minh quyền sở hữu
    verified_at     TIMESTAMPTZ,
    status          VARCHAR(20) DEFAULT 'active', -- active | inactive

    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### 4.5 Transaction (Giao dịch)

```sql
CREATE TABLE transactions (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    transaction_ref VARCHAR(100) UNIQUE NOT NULL, -- Mã GD duy nhất toàn hệ thống
    merchant_id     UUID NOT NULL REFERENCES merchants(id),
    merchant_ref    VARCHAR(255),               -- Mã đơn hàng của merchant

    -- Amounts
    amount          DECIMAL(15,2) NOT NULL,
    currency        CHAR(3) NOT NULL DEFAULT 'VND',
    fee_amount      DECIMAL(15,2) NOT NULL DEFAULT 0,
    net_amount      DECIMAL(15,2) NOT NULL,     -- amount - fee_amount

    -- Status
    status          VARCHAR(30) NOT NULL,
    -- initiated | pending | processing | completed | failed | refunded | chargeback

    -- Payment method
    payment_method  VARCHAR(50) NOT NULL,        -- card | qr | bank_transfer | wallet
    payment_channel VARCHAR(50),                 -- visa | mastercard | napas | momo | ...

    -- Card info (masked)
    card_last4      CHAR(4),
    card_brand      VARCHAR(20),
    card_bank       VARCHAR(100),

    -- Settlement
    is_settled      BOOLEAN DEFAULT FALSE,
    settlement_id   UUID,                        -- Gắn với settlement batch
    settled_at      TIMESTAMPTZ,

    -- Metadata
    ip_address      INET,
    device_type     VARCHAR(50),
    description     TEXT,
    metadata        JSONB,                       -- Dữ liệu thêm của merchant

    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at    TIMESTAMPTZ
);
```

---

### 4.6 Settlement (Thanh toán định kỳ)

```sql
CREATE TABLE settlements (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    settlement_ref  VARCHAR(100) UNIQUE NOT NULL,
    merchant_id     UUID NOT NULL REFERENCES merchants(id),
    bank_account_id UUID NOT NULL REFERENCES merchant_bank_accounts(id),

    -- Period
    period_from     TIMESTAMPTZ NOT NULL,
    period_to       TIMESTAMPTZ NOT NULL,

    -- Amounts
    gross_amount    DECIMAL(15,2) NOT NULL,   -- Tổng doanh thu
    fee_amount      DECIMAL(15,2) NOT NULL,   -- Tổng phí
    refund_amount   DECIMAL(15,2) DEFAULT 0,  -- Tổng hoàn tiền
    chargeback_amount DECIMAL(15,2) DEFAULT 0,
    reserve_amount  DECIMAL(15,2) DEFAULT 0,  -- Tiền giữ lại (rolling reserve)
    net_amount      DECIMAL(15,2) NOT NULL,   -- Thực nhận

    txn_count       INTEGER NOT NULL,          -- Số giao dịch trong kỳ

    -- Status
    status          VARCHAR(30) NOT NULL DEFAULT 'pending',
    -- pending | processing | completed | failed

    -- Bank transfer
    bank_ref        VARCHAR(255),              -- Mã chuyển khoản ngân hàng
    transferred_at  TIMESTAMPTZ,

    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### 4.7 KYC Documents

```sql
CREATE TABLE merchant_kyc_documents (
    id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    merchant_id     UUID NOT NULL REFERENCES merchants(id),

    doc_type        VARCHAR(50) NOT NULL,
    -- cccd | passport | business_license | tax_certificate
    -- bank_statement | director_id | company_charter

    file_url        VARCHAR(500) NOT NULL,
    file_name       VARCHAR(255),
    file_size       INTEGER,

    status          VARCHAR(30) DEFAULT 'pending',
    -- pending | approved | rejected

    reviewer_id     UUID,                      -- Staff review
    reviewer_note   TEXT,
    reviewed_at     TIMESTAMPTZ,

    expires_at      TIMESTAMPTZ,               -- Giấy tờ hết hạn
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

### 4.8 Quan hệ tổng thể

```
merchants (1) ──────── (1) merchant_configs
merchants (1) ──────── (N) merchant_bank_accounts
merchants (1) ──────── (N) merchant_kyc_documents
merchants (1) ──────── (N) transactions
merchants (1) ──────── (N) settlements
settlements (1) ─────── (N) transactions [thông qua settlement_id]
```

---

## 5. Luồng nghiệp vụ

### 5.1 Luồng Onboarding (Đăng ký merchant)

```
Merchant submit form
        │
        ▼
[1] Tạo merchant với status = "pending"
        │
        ▼
[2] Merchant upload KYC/KYB documents
        │
        ▼
[3] System auto-check (nếu có AI/OCR):
    - Đọc thông tin CCCD/GPKD
    - Kiểm tra blacklist
    - Kiểm tra trùng lặp tài khoản
        │
        ▼
[4] Status → "under_review"
        │
        ▼
[5] Staff review thủ công:
    - Xác minh giấy tờ hợp lệ
    - Kiểm tra ngành nghề (MCC phù hợp?)
    - Đánh giá rủi ro
        │
   ┌────┴────┐
Approved   Rejected
   │           │
   ▼           ▼
[6] Setup:   Gửi email
  - MID      lý do từ chối
  - API keys
  - Webhook
  - MDR rate
   │
   ▼
Status → "active"
   │
   ▼
[7] Gửi email chào mừng + credentials
```

**State machine của merchant status:**

```
pending ──→ under_review ──→ active ──→ suspended ──→ terminated
                │                          ↑               ↑
                └──→ rejected              │               │
                                    (vi phạm TOS)  (chargeback cao,
                                                    fraud phát hiện)
```

---

### 5.2 Luồng giao dịch (Transaction Flow)

#### Thanh toán QR Code

```
[1] Merchant tạo payment request
    POST /payments → trả về QR code / payment_url

[2] Customer quét QR → thanh toán trên app ngân hàng

[3] Bank xử lý → notify về Payment Gateway

[4] Gateway cập nhật transaction status = "completed"

[5] Gateway gửi Webhook đến merchant

[6] Merchant update đơn hàng
```

Chi tiết hơn với trạng thái:

```
initiated → pending → processing → completed
                              └──→ failed
                              └──→ timeout (QR hết hạn)
```

#### Refund Flow

```
Merchant gọi API refund (trong vòng X ngày)
        │
        ▼
Kiểm tra:
- Giao dịch gốc có tồn tại?
- Đã completed chưa?
- Số tiền refund ≤ số tiền gốc?
- Chưa bị refund toàn phần?
        │
        ▼
Tạo refund transaction (amount âm)
        │
        ▼
Trừ tiền từ balance/settlement tiếp theo của merchant
        │
        ▼
Hoàn tiền về phương thức thanh toán gốc của customer
        │
        ▼
Webhook → merchant biết refund thành công
```

---

### 5.3 Luồng Settlement

```
Mỗi ngày lúc 08:00 (ví dụ cycle T+1):

[1] Job chạy: lấy tất cả transaction ngày hôm qua
    WHERE merchant_id = X
    AND status = 'completed'
    AND is_settled = FALSE

[2] Tính toán:
    gross_amount = SUM(amount)
    fee_amount   = SUM(fee_amount)
    net_amount   = gross_amount - fee_amount - refund - chargeback - reserve

[3] Tạo settlement record (status = "pending")

[4] Gắn các transactions vào settlement
    UPDATE transactions SET settlement_id = X, is_settled = TRUE

[5] Gửi lệnh chuyển tiền đến banking partner
    (Napas, SWIFT, internal transfer)

[6] Đợi banking confirm
    → status = "completed" + bank_ref
    → hoặc "failed" → retry / alert

[7] Gửi settlement report email / webhook đến merchant
```

**Settlement report mỗi kỳ:**

```
Kỳ: 01/12/2024 – 01/12/2024
─────────────────────────────
Tổng giao dịch:      150
Doanh thu gộp:   45,000,000đ
Phí giao dịch:     -675,000đ  (1.5%)
Hoàn tiền:        -500,000đ
Chargeback:              0đ
Rolling reserve:  -4,382,500đ (10%)
─────────────────────────────
Thực nhận:       39,442,500đ
Tài khoản nhận:  Vietcombank - 0123456789
```

---

### 5.4 Luồng Payout (Rút tiền chủ động)

Khác với settlement tự động, **payout** là khi merchant chủ động yêu cầu rút tiền ngay.

```
Merchant request payout
        │
        ▼
Kiểm tra:
- available_balance có đủ không?
- Tài khoản ngân hàng đã verify?
- Không đang bị suspended?
- Trong giờ hành chính ngân hàng?
        │
        ▼
Tạo payout request
Trừ available_balance ngay (hold)
        │
        ▼
Gửi chuyển khoản → Banking API
        │
   ┌────┴────┐
Success    Failed
   │           │
   ▼           ▼
Cập nhật    Hoàn lại
status      balance
completed   → alert merchant
   │
   ▼
Webhook + Email thông báo
```

---

## 6. Tích hợp hệ thống (API & Webhook)

### 6.1 API Design cho merchant

#### Authentication

Merchant dùng **API Key** để xác thực. Có 2 loại:

```
Public Key  (pk_live_xxx): Dùng ở frontend (tạo payment token)
Secret Key  (sk_live_xxx): Dùng ở backend (tạo payment, refund, query)
```

Gọi API:
```http
POST /v1/payments
Authorization: Bearer sk_live_xxxxxxxxxxxx
Content-Type: application/json
```

---

#### Tạo Payment Request

```http
POST /v1/payments
Authorization: Bearer {secret_key}

{
  "amount": 150000,
  "currency": "VND",
  "payment_method": "qr",
  "merchant_ref": "ORDER-2024-00123",  // Mã đơn hàng của merchant
  "description": "Thanh toán đơn hàng #123",
  "customer": {
    "name": "Nguyen Van A",
    "email": "a@email.com",
    "phone": "0901234567"
  },
  "metadata": {
    "product_ids": ["p1", "p2"],
    "channel": "mobile_app"
  },
  "return_url": "https://shop.com/payment/result",
  "cancel_url":  "https://shop.com/payment/cancel",
  "expire_in":   900   // QR hết hạn sau 15 phút
}
```

Response:
```json
{
  "transaction_ref": "TXN-20241201-ABCDE",
  "status": "pending",
  "amount": 150000,
  "currency": "VND",
  "qr_code": "00020101021238...",          // QR string
  "qr_image_url": "https://cdn.../qr.png", // QR image
  "payment_url": "https://pay.../TXN-xxx", // Link thanh toán
  "expired_at": "2024-12-01T09:15:00Z",
  "created_at": "2024-12-01T09:00:00Z"
}
```

---

#### Query Transaction

```http
GET /v1/payments/{transaction_ref}
Authorization: Bearer {secret_key}
```

Response:
```json
{
  "transaction_ref": "TXN-20241201-ABCDE",
  "merchant_ref": "ORDER-2024-00123",
  "status": "completed",
  "amount": 150000,
  "fee_amount": 2250,
  "net_amount": 147750,
  "payment_method": "qr",
  "payment_channel": "vcb",
  "completed_at": "2024-12-01T09:03:45Z"
}
```

---

#### Refund

```http
POST /v1/payments/{transaction_ref}/refunds
Authorization: Bearer {secret_key}

{
  "amount": 150000,      // Có thể refund một phần
  "reason": "customer_request",
  "merchant_ref": "REFUND-ORDER-00123"
}
```

---

#### Query Settlement

```http
GET /v1/settlements?from=2024-12-01&to=2024-12-31
Authorization: Bearer {secret_key}
```

---

### 6.2 Webhook

Webhook là cơ chế **hệ thống chủ động thông báo** đến merchant khi có sự kiện xảy ra. Merchant không cần polling liên tục.

#### Các events quan trọng

| Event | Khi nào trigger |
|---|---|
| `payment.completed` | Giao dịch thành công |
| `payment.failed` | Giao dịch thất bại |
| `payment.expired` | QR/link hết hạn |
| `refund.completed` | Hoàn tiền thành công |
| `refund.failed` | Hoàn tiền thất bại |
| `settlement.completed` | Đã chuyển tiền về ngân hàng |
| `chargeback.received` | Nhận khiếu nại từ khách hàng |
| `merchant.suspended` | Tài khoản bị tạm khóa |

#### Webhook payload

```json
POST https://merchant-shop.com/webhooks/payment
Headers:
  X-Webhook-Signature: sha256=abcdef123456...
  X-Event-Type: payment.completed
  X-Delivery-ID: wh_01HXXX

Body:
{
  "event_type": "payment.completed",
  "event_id": "evt_01HXXX",
  "created_at": "2024-12-01T09:03:45Z",
  "data": {
    "transaction_ref": "TXN-20241201-ABCDE",
    "merchant_ref": "ORDER-2024-00123",
    "status": "completed",
    "amount": 150000,
    "net_amount": 147750,
    "completed_at": "2024-12-01T09:03:45Z"
  }
}
```

#### Webhook Signature Verification

Merchant phải **verify chữ ký** để đảm bảo webhook đến từ hệ thống thực sự, không phải kẻ tấn công.

```python
import hmac
import hashlib

def verify_webhook(payload_body: bytes, signature_header: str, secret: str) -> bool:
    expected = hmac.new(
        key=secret.encode(),
        msg=payload_body,
        digestmod=hashlib.sha256
    ).hexdigest()

    received = signature_header.replace("sha256=", "")
    return hmac.compare_digest(expected, received)
```

#### Webhook Retry Policy

Nếu merchant server trả về status khác 2xx → hệ thống retry:

```
Attempt 1: Ngay lập tức
Attempt 2: 1 phút sau
Attempt 3: 5 phút sau
Attempt 4: 30 phút sau
Attempt 5: 2 giờ sau
Attempt 6: 24 giờ sau
→ Nếu vẫn fail: mark webhook failed, alert merchant
```

---

### 6.3 Idempotency — Tránh xử lý trùng lặp

Merchant có thể gọi API nhiều lần (retry khi timeout). Hệ thống dùng **Idempotency Key** để đảm bảo chỉ tạo một giao dịch dù gọi nhiều lần.

```http
POST /v1/payments
Idempotency-Key: ORDER-2024-00123-attempt-1

{...}
```

- Lần đầu: tạo payment mới
- Lần sau với cùng key: trả về response cũ, không tạo mới

---

## 7. Bảo mật trong Merchant System

### 7.1 PCI-DSS Compliance

Nếu merchant xử lý dữ liệu thẻ, phải tuân thủ **PCI-DSS** (Payment Card Industry Data Security Standard).

Nguyên tắc chính:
- **Không bao giờ** lưu CVV
- **Không lưu** full card number dạng plaintext
- Card number phải được **tokenized** (thay bằng token vô nghĩa)
- Môi trường xử lý thẻ phải isolated

```
Card number: 4111 1111 1111 1234
          ↓  Tokenization
Token:      tok_live_4xNzM8p...

→ Lưu token, không lưu card number
→ Khi charge: gửi token, không gửi card number
```

### 7.2 API Key Management

```
- Secret key chỉ dùng ở server-side, KHÔNG expose ở frontend
- Hỗ trợ key rotation (tạo key mới, vô hiệu hóa key cũ)
- Key có prefix để phân biệt môi trường:
  pk_test_xxx / sk_test_xxx  → Sandbox
  pk_live_xxx / sk_live_xxx  → Production
- Log mọi API call với key nào gọi
```

### 7.3 Fraud Detection

Các signal phổ biến để phát hiện fraud:

```
- Nhiều giao dịch failed liên tiếp (card testing)
- Số tiền bất thường (quá lớn so với lịch sử)
- Nhiều giao dịch từ cùng IP/device ngắn thời gian
- Địa chỉ billing và shipping khác xa nhau
- Merchant mới + transaction lớn ngay lập tức
- Chargeback rate > 1%
```

---

## 8. Merchant Lifecycle — Tổng quan

```
                    ┌──────────────┐
                    │  REGISTERED  │ ← Merchant điền form
                    └──────┬───────┘
                           │
                    ┌──────▼───────┐
                    │ UNDER REVIEW │ ← KYC/KYB đang xét duyệt
                    └──────┬───────┘
                    ┌──────┴───────┐
                    │   REJECTED   │ ← Không đủ điều kiện
                    └──────────────┘
                           │ Approved
                    ┌──────▼───────┐
                    │    ACTIVE    │ ← Đang hoạt động bình thường
                    └──────┬───────┘
              ┌────────────┼────────────┐
              │            │            │
       ┌──────▼───┐  ┌─────▼──────┐  ┌─▼──────────┐
       │SUSPENDED │  │UNDER WATCH │  │  DORMANT   │
       │(tạm khóa)│  │(cảnh báo)  │  │(không dùng)│
       └──────┬───┘  └─────┬──────┘  └────────────┘
              │             │
              └──────┬──────┘
                     │
              ┌──────▼───────┐
              │  TERMINATED  │ ← Đóng vĩnh viễn
              └──────────────┘
```

---

## 9. Checklist khi build Merchant System

### Thiết kế

- [ ] Phân biệt rõ Merchant entity và User account (một merchant có thể nhiều user)
- [ ] Thiết kế multi-currency từ đầu nếu cần
- [ ] Tách biệt môi trường sandbox và production ngay từ đầu

### Tài chính

- [ ] Mọi phép tính tiền dùng `DECIMAL`, tuyệt đối không dùng `FLOAT`
- [ ] Lưu cả `gross_amount`, `fee_amount`, `net_amount` — không tính lại từ rate
- [ ] Audit log cho mọi thay đổi về số dư và settlement

### API

- [ ] Implement idempotency key cho mọi write endpoint
- [ ] Version API từ đầu (`/v1/`)
- [ ] Rate limiting theo merchant ID
- [ ] Webhook có retry + signature verification

### Bảo mật

- [ ] Không log sensitive data (card number, secret key)
- [ ] API key có thể revoke ngay lập tức
- [ ] Mọi action quan trọng cần audit trail

---

## 10. Tóm tắt

```
Merchant = Đơn vị bán hàng nhận thanh toán qua hệ thống payment

Vòng đời:  Đăng ký → KYC → Active → Giao dịch → Settlement

Dòng tiền: Customer thanh toán → Gateway giữ → Settlement → Ngân hàng merchant

Key concepts:
  MID       — Mã định danh merchant
  MCC       — Mã ngành nghề
  MDR       — Phí % trên giao dịch
  Settlement — Chuyển tiền định kỳ về tài khoản
  Chargeback — Khách hàng khiếu nại, bị trừ tiền bắt buộc
  KYC/KYB   — Xác minh danh tính trước khi hoạt động
  Webhook   — Hệ thống chủ động notify merchant khi có event
```