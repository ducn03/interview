# \# 🏪 Merchant Domain — Payment / Fintech

# 

# > Tài liệu này giải thích toàn bộ domain \*\*Merchant\*\* trong hệ thống Payment / Fintech: từ khái niệm cơ bản, business model, data model, luồng nghiệp vụ đến tích hợp hệ thống.

# 

# \---

# 

# \## 1. Merchant là gì?

# 

# \*\*Merchant\*\* (đơn vị chấp nhận thanh toán) là \*\*cá nhân hoặc doanh nghiệp bán hàng/dịch vụ\*\* và sử dụng hệ thống payment để \*\*nhận tiền từ khách hàng\*\*.

# 

# \### Ví dụ thực tế

# 

# | Merchant | Loại | Dùng payment để làm gì |

# |---|---|---|

# | Cửa hàng trà sữa | Offline | Nhận QR Code, POS |

# | Shop thời trang Shopee | Online | Nhận thanh toán đơn hàng |

# | Ứng dụng gọi xe | Platform | Thu tiền từ hành khách, trả cho tài xế |

# | SaaS subscription | Online | Thu phí hàng tháng tự động |

# | Siêu thị lớn | Offline | POS, thẻ, QR |

# 

# \### Phân biệt các bên trong hệ sinh thái

# 

# ```

# Cardholder / Customer      Merchant              Acquirer            Issuer

# (Người mua)            (Người bán)         (Ngân hàng của        (Ngân hàng của

# &#x20;                                           merchant)              khách hàng)

# &#x20;     │                     │                    │                    │

# &#x20;     │  ── thanh toán ──>  │                    │                    │

# &#x20;     │                     │  ── request ──>    │  ── verify ──>    │

# &#x20;     │                     │                    │  <── approve ──   │

# &#x20;     │                     │  <── approved ──   │                    │

# &#x20;     │  <── receipt ──     │                    │                    │

# ```

# 

# \- \*\*Customer/Cardholder\*\*: Người mua — trả tiền

# \- \*\*Merchant\*\*: Người bán — nhận tiền

# \- \*\*Acquirer\*\*: Ngân hàng/tổ chức quản lý tài khoản của merchant (VD: VietcomBank, VNPAY)

# \- \*\*Issuer\*\*: Ngân hàng phát hành thẻ cho khách hàng (VD: Techcombank)

# \- \*\*Payment Gateway\*\*: Cầu nối kỹ thuật giữa merchant và acquirer (VD: VNPAY, PayOS, Stripe)

# \- \*\*PSP (Payment Service Provider)\*\*: Cung cấp toàn bộ dịch vụ payment cho merchant (gateway + acquirer + settlement)

# 

# \---

# 

# \## 2. Business Model — Merchant trong hệ thống Payment

# 

# \### 2.1 Cách merchant kiếm tiền và hệ thống kiếm tiền từ merchant

# 

# ```

# Customer trả 100,000đ

# &#x20;        │

# &#x20;        ▼

# &#x20;   Payment Gateway

# &#x20;        │

# &#x20;   Trừ MDR (fee)

# &#x20;        │

# &#x20;        ▼

# &#x20; Merchant nhận \~98,500đ

# ```

# 

# \*\*MDR (Merchant Discount Rate)\*\*: Phí merchant trả cho mỗi giao dịch thành công.

# 

# Ví dụ MDR = 1.5%:

# \- Giao dịch 100,000đ → Merchant nhận 98,500đ

# \- Payment provider giữ 1,500đ

# 

# \### 2.2 Các loại phí merchant phải trả

# 

# | Loại phí | Mô tả | Ví dụ |

# |---|---|---|

# | \*\*MDR\*\* | % trên mỗi giao dịch | 0.5% – 3% tùy phương thức |

# | \*\*Setup fee\*\* | Phí mở tài khoản | Miễn phí hoặc vài triệu |

# | \*\*Monthly fee\*\* | Phí duy trì hàng tháng | 0 – 500,000đ |

# | \*\*Chargeback fee\*\* | Phí khi bị khiếu nại hoàn tiền | 150,000 – 500,000đ/vụ |

# | \*\*Payout fee\*\* | Phí rút tiền về tài khoản ngân hàng | 0 – 11,000đ/lần |

# 

# \### 2.3 Các mô hình merchant phổ biến

# 

# \#### Direct Merchant

# Merchant bán trực tiếp cho end-user. Luồng đơn giản nhất.

# 

# ```

# Customer → \[Thanh toán] → Merchant

# ```

# 

# \#### Marketplace / Platform

# Merchant bán trên nền tảng trung gian. Platform thu hộ rồi phân chia.

# 

# ```

# Customer → \[Thanh toán] → Platform → \[Split] → Merchant A

# &#x20;                                             → Merchant B

# &#x20;                                             → Platform fee

# ```

# Ví dụ: Shopee, Grab, Tiki

# 

# \#### Sub-merchant (Payment Facilitation)

# Một merchant lớn (PayFac) đứng ra đăng ký thay cho nhiều merchant nhỏ.

# 

# ```

# Customer → \[Thanh toán] → PayFac (Master Merchant)

# &#x20;                              → Sub-merchant A

# &#x20;                              → Sub-merchant B

# ```

# Ví dụ: Square, Stripe Connect — cho phép platform tạo merchant con.

# 

# \---

# 

# \## 3. Các khái niệm cốt lõi

# 

# \### 3.1 Merchant Account vs Business Account

# 

# | | Merchant Account | Business Account |

# |---|---|---|

# | Mục đích | Nhận thanh toán từ card/QR | Tài khoản ngân hàng thông thường |

# | Ai cấp | Acquirer / PSP | Ngân hàng |

# | Chứa tiền? | Tạm thời (pending settlement) | Có |

# 

# \### 3.2 Settlement (Thanh toán định kỳ)

# 

# Tiền từ giao dịch \*\*không về ngay\*\* tài khoản ngân hàng của merchant. Nó được \*\*giữ lại\*\* và \*\*settlement\*\* (chuyển) định kỳ.

# 

# ```

# Giao dịch thành công (T)

# &#x20;       │

# &#x20;       ▼

# &#x20; Tiền vào holding pool

# &#x20;       │

# &#x20;  T+1 hoặc T+2 ngày

# &#x20;       │

# &#x20;       ▼

# &#x20; Settlement: chuyển tiền về tài khoản ngân hàng merchant

# ```

# 

# \*\*Tại sao có độ trễ?\*\* Để xử lý refund, chargeback, kiểm tra gian lận.

# 

# \### 3.3 Rolling Reserve

# 

# Hệ thống \*\*giữ lại một phần tiền\*\* của merchant (thường 5–10%) trong một khoảng thời gian nhất định để đảm bảo nếu có chargeback thì có tiền hoàn trả.

# 

# ```

# Giao dịch 1,000,000đ

# ├── 90% → Settlement ngay → 900,000đ về tài khoản merchant

# └── 10% → Reserve 90 ngày → 100,000đ giải phóng sau 3 tháng

# ```

# 

# \### 3.4 Chargeback

# 

# Khách hàng khiếu nại với ngân hàng phát hành thẻ → ngân hàng \*\*buộc hoàn tiền\*\* về cho khách, không cần merchant đồng ý.

# 

# ```

# Customer khiếu nại "không nhận được hàng"

# &#x20;       │

# &#x20;       ▼

# Issuer bank (ngân hàng khách hàng)

# &#x20;       │

# &#x20;       ▼

# Acquirer nhận thông báo chargeback

# &#x20;       │

# &#x20;       ▼

# Trừ tiền từ tài khoản merchant

# &#x20;       │

# Merchant có quyền dispute (tranh chấp) bằng bằng chứng

# ```

# 

# Chargeback rate cao → merchant có thể bị \*\*terminate\*\* (đóng tài khoản).

# 

# \### 3.5 KYC / KYB (Know Your Customer / Know Your Business)

# 

# Quy trình \*\*xác minh danh tính\*\* merchant trước khi được phép nhận thanh toán.

# 

# | | KYC | KYB |

# |---|---|---|

# | Đối tượng | Cá nhân | Doanh nghiệp |

# | Giấy tờ | CMND/CCCD, selfie | Giấy phép KD, MST, giấy tờ người đại diện |

# | Mục đích | Chống giả mạo, rửa tiền | Chống shell company, fraud |

# 

# \### 3.6 MID (Merchant ID)

# 

# Mã định danh duy nhất của merchant trong hệ thống payment. Mỗi merchant có ít nhất một MID. Merchant lớn có thể có nhiều MID (mỗi kênh bán một MID riêng).

# 

# \### 3.7 MCC (Merchant Category Code)

# 

# Mã 4 chữ số phân loại ngành nghề của merchant. Do Visa/Mastercard quy định.

# 

# | MCC | Ngành |

# |---|---|

# | 5411 | Grocery Stores (siêu thị) |

# | 5812 | Restaurants |

# | 4111 | Local and Suburban Commuter |

# | 7011 | Hotels |

# 

# MCC ảnh hưởng đến: MDR rate, giới hạn giao dịch, phân loại rủi ro.

# 

# \---

# 

# \## 4. Data Model / Database Schema

# 

# \### 4.1 Tổng quan các entity chính

# 

# ```

# ┌─────────────┐     ┌──────────────────┐     ┌─────────────────┐

# │  Merchant   │────<│ MerchantLocation │     │ MerchantAccount │

# └──────┬──────┘     └──────────────────┘     └────────┬────────┘

# &#x20;      │                                              │

# &#x20;      │            ┌──────────────────┐              │

# &#x20;      ├────────────│  MerchantConfig  │              │

# &#x20;      │            └──────────────────┘              │

# &#x20;      │                                              │

# &#x20;      │            ┌──────────────────┐     ┌────────┴────────┐

# &#x20;      └────────────│   Transaction    │────>│   Settlement    │

# &#x20;                   └──────────────────┘     └─────────────────┘

# ```

# 

# \---

# 

# \### 4.2 Merchant (Entity chính)

# 

# ```sql

# CREATE TABLE merchants (

# &#x20;   id              UUID PRIMARY KEY DEFAULT gen\_random\_uuid(),

# &#x20;   merchant\_code   VARCHAR(20) UNIQUE NOT NULL,   -- MID: VN-2024-00123

# &#x20;   business\_name   VARCHAR(255) NOT NULL,          -- Tên đăng ký kinh doanh

# &#x20;   display\_name    VARCHAR(255),                   -- Tên hiển thị với khách hàng

# &#x20;   merchant\_type   VARCHAR(50) NOT NULL,           -- individual | business | enterprise

# &#x20;   category\_code   VARCHAR(10) NOT NULL,           -- MCC: 5812

# &#x20;   status          VARCHAR(30) NOT NULL DEFAULT 'pending',

# &#x20;                   -- pending | under\_review | active | suspended | terminated

# 

# &#x20;   -- KYC / KYB

# &#x20;   kyc\_status      VARCHAR(30) DEFAULT 'not\_started',

# &#x20;                   -- not\_started | in\_progress | submitted | approved | rejected

# &#x20;   kyc\_verified\_at TIMESTAMPTZ,

# 

# &#x20;   -- Contact

# &#x20;   email           VARCHAR(255) NOT NULL,

# &#x20;   phone           VARCHAR(20),

# &#x20;   website         VARCHAR(500),

# 

# &#x20;   -- Business address

# &#x20;   country\_code    CHAR(2) NOT NULL DEFAULT 'VN',

# &#x20;   province        VARCHAR(100),

# &#x20;   district        VARCHAR(100),

# &#x20;   address         TEXT,

# 

# &#x20;   -- Tier \& Risk

# &#x20;   merchant\_tier   VARCHAR(20) DEFAULT 'standard', -- standard | premium | enterprise

# &#x20;   risk\_level      VARCHAR(20) DEFAULT 'medium',   -- low | medium | high

# &#x20;   chargeback\_rate DECIMAL(5,4) DEFAULT 0,         -- 0.0150 = 1.5%

# 

# &#x20;   -- Timestamps

# &#x20;   created\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

# &#x20;   updated\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

# &#x20;   activated\_at    TIMESTAMPTZ,

# &#x20;   terminated\_at   TIMESTAMPTZ

# );

# ```

# 

# \---

# 

# \### 4.3 MerchantConfig (Cấu hình thương mại)

# 

# ```sql

# CREATE TABLE merchant\_configs (

# &#x20;   id              UUID PRIMARY KEY DEFAULT gen\_random\_uuid(),

# &#x20;   merchant\_id     UUID NOT NULL REFERENCES merchants(id),

# 

# &#x20;   -- Fee configuration

# &#x20;   mdr\_rate        DECIMAL(6,4) NOT NULL DEFAULT 0.0150, -- 1.50%

# &#x20;   mdr\_fixed\_fee   DECIMAL(12,2) DEFAULT 0,              -- Phí cố định/giao dịch (đồng)

# &#x20;   payout\_fee      DECIMAL(12,2) DEFAULT 0,              -- Phí rút tiền

# 

# &#x20;   -- Transaction limits

# &#x20;   min\_txn\_amount  DECIMAL(15,2) DEFAULT 1000,

# &#x20;   max\_txn\_amount  DECIMAL(15,2) DEFAULT 50000000,       -- 50 triệu/giao dịch

# &#x20;   daily\_limit     DECIMAL(15,2) DEFAULT 500000000,      -- 500 triệu/ngày

# 

# &#x20;   -- Settlement

# &#x20;   settlement\_cycle VARCHAR(20) DEFAULT 'T+1',            -- T+0 | T+1 | T+2 | weekly

# &#x20;   settlement\_time  TIME DEFAULT '08:00:00',              -- Giờ settlement mỗi ngày

# 

# &#x20;   -- Rolling reserve

# &#x20;   reserve\_rate    DECIMAL(5,4) DEFAULT 0,                -- 0.10 = 10%

# &#x20;   reserve\_days    INTEGER DEFAULT 90,

# 

# &#x20;   -- Payment methods được phép

# &#x20;   allowed\_methods JSONB DEFAULT '\["card","qr","bank\_transfer"]',

# 

# &#x20;   -- Webhook

# &#x20;   webhook\_url     VARCHAR(500),

# &#x20;   webhook\_secret  VARCHAR(255),

# 

# &#x20;   created\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

# &#x20;   updated\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()

# );

# ```

# 

# \---

# 

# \### 4.4 MerchantBankAccount (Tài khoản nhận settlement)

# 

# ```sql

# CREATE TABLE merchant\_bank\_accounts (

# &#x20;   id              UUID PRIMARY KEY DEFAULT gen\_random\_uuid(),

# &#x20;   merchant\_id     UUID NOT NULL REFERENCES merchants(id),

# 

# &#x20;   bank\_code       VARCHAR(20) NOT NULL,   -- VCB, TCB, MB, ...

# &#x20;   bank\_name       VARCHAR(255) NOT NULL,

# &#x20;   account\_number  VARCHAR(50) NOT NULL,

# &#x20;   account\_name    VARCHAR(255) NOT NULL,  -- Tên chủ tài khoản (phải khớp)

# &#x20;   branch          VARCHAR(255),

# 

# &#x20;   is\_primary      BOOLEAN DEFAULT FALSE,  -- Tài khoản settlement chính

# &#x20;   is\_verified     BOOLEAN DEFAULT FALSE,  -- Đã xác minh quyền sở hữu

# &#x20;   verified\_at     TIMESTAMPTZ,

# &#x20;   status          VARCHAR(20) DEFAULT 'active', -- active | inactive

# 

# &#x20;   created\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()

# );

# ```

# 

# \---

# 

# \### 4.5 Transaction (Giao dịch)

# 

# ```sql

# CREATE TABLE transactions (

# &#x20;   id              UUID PRIMARY KEY DEFAULT gen\_random\_uuid(),

# &#x20;   transaction\_ref VARCHAR(100) UNIQUE NOT NULL, -- Mã GD duy nhất toàn hệ thống

# &#x20;   merchant\_id     UUID NOT NULL REFERENCES merchants(id),

# &#x20;   merchant\_ref    VARCHAR(255),               -- Mã đơn hàng của merchant

# 

# &#x20;   -- Amounts

# &#x20;   amount          DECIMAL(15,2) NOT NULL,

# &#x20;   currency        CHAR(3) NOT NULL DEFAULT 'VND',

# &#x20;   fee\_amount      DECIMAL(15,2) NOT NULL DEFAULT 0,

# &#x20;   net\_amount      DECIMAL(15,2) NOT NULL,     -- amount - fee\_amount

# 

# &#x20;   -- Status

# &#x20;   status          VARCHAR(30) NOT NULL,

# &#x20;   -- initiated | pending | processing | completed | failed | refunded | chargeback

# 

# &#x20;   -- Payment method

# &#x20;   payment\_method  VARCHAR(50) NOT NULL,        -- card | qr | bank\_transfer | wallet

# &#x20;   payment\_channel VARCHAR(50),                 -- visa | mastercard | napas | momo | ...

# 

# &#x20;   -- Card info (masked)

# &#x20;   card\_last4      CHAR(4),

# &#x20;   card\_brand      VARCHAR(20),

# &#x20;   card\_bank       VARCHAR(100),

# 

# &#x20;   -- Settlement

# &#x20;   is\_settled      BOOLEAN DEFAULT FALSE,

# &#x20;   settlement\_id   UUID,                        -- Gắn với settlement batch

# &#x20;   settled\_at      TIMESTAMPTZ,

# 

# &#x20;   -- Metadata

# &#x20;   ip\_address      INET,

# &#x20;   device\_type     VARCHAR(50),

# &#x20;   description     TEXT,

# &#x20;   metadata        JSONB,                       -- Dữ liệu thêm của merchant

# 

# &#x20;   created\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

# &#x20;   updated\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

# &#x20;   completed\_at    TIMESTAMPTZ

# );

# ```

# 

# \---

# 

# \### 4.6 Settlement (Thanh toán định kỳ)

# 

# ```sql

# CREATE TABLE settlements (

# &#x20;   id              UUID PRIMARY KEY DEFAULT gen\_random\_uuid(),

# &#x20;   settlement\_ref  VARCHAR(100) UNIQUE NOT NULL,

# &#x20;   merchant\_id     UUID NOT NULL REFERENCES merchants(id),

# &#x20;   bank\_account\_id UUID NOT NULL REFERENCES merchant\_bank\_accounts(id),

# 

# &#x20;   -- Period

# &#x20;   period\_from     TIMESTAMPTZ NOT NULL,

# &#x20;   period\_to       TIMESTAMPTZ NOT NULL,

# 

# &#x20;   -- Amounts

# &#x20;   gross\_amount    DECIMAL(15,2) NOT NULL,   -- Tổng doanh thu

# &#x20;   fee\_amount      DECIMAL(15,2) NOT NULL,   -- Tổng phí

# &#x20;   refund\_amount   DECIMAL(15,2) DEFAULT 0,  -- Tổng hoàn tiền

# &#x20;   chargeback\_amount DECIMAL(15,2) DEFAULT 0,

# &#x20;   reserve\_amount  DECIMAL(15,2) DEFAULT 0,  -- Tiền giữ lại (rolling reserve)

# &#x20;   net\_amount      DECIMAL(15,2) NOT NULL,   -- Thực nhận

# 

# &#x20;   txn\_count       INTEGER NOT NULL,          -- Số giao dịch trong kỳ

# 

# &#x20;   -- Status

# &#x20;   status          VARCHAR(30) NOT NULL DEFAULT 'pending',

# &#x20;   -- pending | processing | completed | failed

# 

# &#x20;   -- Bank transfer

# &#x20;   bank\_ref        VARCHAR(255),              -- Mã chuyển khoản ngân hàng

# &#x20;   transferred\_at  TIMESTAMPTZ,

# 

# &#x20;   created\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),

# &#x20;   updated\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()

# );

# ```

# 

# \---

# 

# \### 4.7 KYC Documents

# 

# ```sql

# CREATE TABLE merchant\_kyc\_documents (

# &#x20;   id              UUID PRIMARY KEY DEFAULT gen\_random\_uuid(),

# &#x20;   merchant\_id     UUID NOT NULL REFERENCES merchants(id),

# 

# &#x20;   doc\_type        VARCHAR(50) NOT NULL,

# &#x20;   -- cccd | passport | business\_license | tax\_certificate

# &#x20;   -- bank\_statement | director\_id | company\_charter

# 

# &#x20;   file\_url        VARCHAR(500) NOT NULL,

# &#x20;   file\_name       VARCHAR(255),

# &#x20;   file\_size       INTEGER,

# 

# &#x20;   status          VARCHAR(30) DEFAULT 'pending',

# &#x20;   -- pending | approved | rejected

# 

# &#x20;   reviewer\_id     UUID,                      -- Staff review

# &#x20;   reviewer\_note   TEXT,

# &#x20;   reviewed\_at     TIMESTAMPTZ,

# 

# &#x20;   expires\_at      TIMESTAMPTZ,               -- Giấy tờ hết hạn

# &#x20;   created\_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()

# );

# ```

# 

# \---

# 

# \### 4.8 Quan hệ tổng thể

# 

# ```

# merchants (1) ──────── (1) merchant\_configs

# merchants (1) ──────── (N) merchant\_bank\_accounts

# merchants (1) ──────── (N) merchant\_kyc\_documents

# merchants (1) ──────── (N) transactions

# merchants (1) ──────── (N) settlements

# settlements (1) ─────── (N) transactions \[thông qua settlement\_id]

# ```

# 

# \---

# 

# \## 5. Luồng nghiệp vụ

# 

# \### 5.1 Luồng Onboarding (Đăng ký merchant)

# 

# ```

# Merchant submit form

# &#x20;       │

# &#x20;       ▼

# \[1] Tạo merchant với status = "pending"

# &#x20;       │

# &#x20;       ▼

# \[2] Merchant upload KYC/KYB documents

# &#x20;       │

# &#x20;       ▼

# \[3] System auto-check (nếu có AI/OCR):

# &#x20;   - Đọc thông tin CCCD/GPKD

# &#x20;   - Kiểm tra blacklist

# &#x20;   - Kiểm tra trùng lặp tài khoản

# &#x20;       │

# &#x20;       ▼

# \[4] Status → "under\_review"

# &#x20;       │

# &#x20;       ▼

# \[5] Staff review thủ công:

# &#x20;   - Xác minh giấy tờ hợp lệ

# &#x20;   - Kiểm tra ngành nghề (MCC phù hợp?)

# &#x20;   - Đánh giá rủi ro

# &#x20;       │

# &#x20;  ┌────┴────┐

# Approved   Rejected

# &#x20;  │           │

# &#x20;  ▼           ▼

# \[6] Setup:   Gửi email

# &#x20; - MID      lý do từ chối

# &#x20; - API keys

# &#x20; - Webhook

# &#x20; - MDR rate

# &#x20;  │

# &#x20;  ▼

# Status → "active"

# &#x20;  │

# &#x20;  ▼

# \[7] Gửi email chào mừng + credentials

# ```

# 

# \*\*State machine của merchant status:\*\*

# 

# ```

# pending ──→ under\_review ──→ active ──→ suspended ──→ terminated

# &#x20;               │                          ↑               ↑

# &#x20;               └──→ rejected              │               │

# &#x20;                                   (vi phạm TOS)  (chargeback cao,

# &#x20;                                                   fraud phát hiện)

# ```

# 

# \---

# 

# \### 5.2 Luồng giao dịch (Transaction Flow)

# 

# \#### Thanh toán QR Code

# 

# ```

# \[1] Merchant tạo payment request

# &#x20;   POST /payments → trả về QR code / payment\_url

# 

# \[2] Customer quét QR → thanh toán trên app ngân hàng

# 

# \[3] Bank xử lý → notify về Payment Gateway

# 

# \[4] Gateway cập nhật transaction status = "completed"

# 

# \[5] Gateway gửi Webhook đến merchant

# 

# \[6] Merchant update đơn hàng

# ```

# 

# Chi tiết hơn với trạng thái:

# 

# ```

# initiated → pending → processing → completed

# &#x20;                             └──→ failed

# &#x20;                             └──→ timeout (QR hết hạn)

# ```

# 

# \#### Refund Flow

# 

# ```

# Merchant gọi API refund (trong vòng X ngày)

# &#x20;       │

# &#x20;       ▼

# Kiểm tra:

# \- Giao dịch gốc có tồn tại?

# \- Đã completed chưa?

# \- Số tiền refund ≤ số tiền gốc?

# \- Chưa bị refund toàn phần?

# &#x20;       │

# &#x20;       ▼

# Tạo refund transaction (amount âm)

# &#x20;       │

# &#x20;       ▼

# Trừ tiền từ balance/settlement tiếp theo của merchant

# &#x20;       │

# &#x20;       ▼

# Hoàn tiền về phương thức thanh toán gốc của customer

# &#x20;       │

# &#x20;       ▼

# Webhook → merchant biết refund thành công

# ```

# 

# \---

# 

# \### 5.3 Luồng Settlement

# 

# ```

# Mỗi ngày lúc 08:00 (ví dụ cycle T+1):

# 

# \[1] Job chạy: lấy tất cả transaction ngày hôm qua

# &#x20;   WHERE merchant\_id = X

# &#x20;   AND status = 'completed'

# &#x20;   AND is\_settled = FALSE

# 

# \[2] Tính toán:

# &#x20;   gross\_amount = SUM(amount)

# &#x20;   fee\_amount   = SUM(fee\_amount)

# &#x20;   net\_amount   = gross\_amount - fee\_amount - refund - chargeback - reserve

# 

# \[3] Tạo settlement record (status = "pending")

# 

# \[4] Gắn các transactions vào settlement

# &#x20;   UPDATE transactions SET settlement\_id = X, is\_settled = TRUE

# 

# \[5] Gửi lệnh chuyển tiền đến banking partner

# &#x20;   (Napas, SWIFT, internal transfer)

# 

# \[6] Đợi banking confirm

# &#x20;   → status = "completed" + bank\_ref

# &#x20;   → hoặc "failed" → retry / alert

# 

# \[7] Gửi settlement report email / webhook đến merchant

# ```

# 

# \*\*Settlement report mỗi kỳ:\*\*

# 

# ```

# Kỳ: 01/12/2024 – 01/12/2024

# ─────────────────────────────

# Tổng giao dịch:      150

# Doanh thu gộp:   45,000,000đ

# Phí giao dịch:     -675,000đ  (1.5%)

# Hoàn tiền:        -500,000đ

# Chargeback:              0đ

# Rolling reserve:  -4,382,500đ (10%)

# ─────────────────────────────

# Thực nhận:       39,442,500đ

# Tài khoản nhận:  Vietcombank - 0123456789

# ```

# 

# \---

# 

# \### 5.4 Luồng Payout (Rút tiền chủ động)

# 

# Khác với settlement tự động, \*\*payout\*\* là khi merchant chủ động yêu cầu rút tiền ngay.

# 

# ```

# Merchant request payout

# &#x20;       │

# &#x20;       ▼

# Kiểm tra:

# \- available\_balance có đủ không?

# \- Tài khoản ngân hàng đã verify?

# \- Không đang bị suspended?

# \- Trong giờ hành chính ngân hàng?

# &#x20;       │

# &#x20;       ▼

# Tạo payout request

# Trừ available\_balance ngay (hold)

# &#x20;       │

# &#x20;       ▼

# Gửi chuyển khoản → Banking API

# &#x20;       │

# &#x20;  ┌────┴────┐

# Success    Failed

# &#x20;  │           │

# &#x20;  ▼           ▼

# Cập nhật    Hoàn lại

# status      balance

# completed   → alert merchant

# &#x20;  │

# &#x20;  ▼

# Webhook + Email thông báo

# ```

# 

# \---

# 

# \## 6. Tích hợp hệ thống (API \& Webhook)

# 

# \### 6.1 API Design cho merchant

# 

# \#### Authentication

# 

# Merchant dùng \*\*API Key\*\* để xác thực. Có 2 loại:

# 

# ```

# Public Key  (pk\_live\_xxx): Dùng ở frontend (tạo payment token)

# Secret Key  (sk\_live\_xxx): Dùng ở backend (tạo payment, refund, query)

# ```

# 

# Gọi API:

# ```http

# POST /v1/payments

# Authorization: Bearer sk\_live\_xxxxxxxxxxxx

# Content-Type: application/json

# ```

# 

# \---

# 

# \#### Tạo Payment Request

# 

# ```http

# POST /v1/payments

# Authorization: Bearer {secret\_key}

# 

# {

# &#x20; "amount": 150000,

# &#x20; "currency": "VND",

# &#x20; "payment\_method": "qr",

# &#x20; "merchant\_ref": "ORDER-2024-00123",  // Mã đơn hàng của merchant

# &#x20; "description": "Thanh toán đơn hàng #123",

# &#x20; "customer": {

# &#x20;   "name": "Nguyen Van A",

# &#x20;   "email": "a@email.com",

# &#x20;   "phone": "0901234567"

# &#x20; },

# &#x20; "metadata": {

# &#x20;   "product\_ids": \["p1", "p2"],

# &#x20;   "channel": "mobile\_app"

# &#x20; },

# &#x20; "return\_url": "https://shop.com/payment/result",

# &#x20; "cancel\_url":  "https://shop.com/payment/cancel",

# &#x20; "expire\_in":   900   // QR hết hạn sau 15 phút

# }

# ```

# 

# Response:

# ```json

# {

# &#x20; "transaction\_ref": "TXN-20241201-ABCDE",

# &#x20; "status": "pending",

# &#x20; "amount": 150000,

# &#x20; "currency": "VND",

# &#x20; "qr\_code": "00020101021238...",          // QR string

# &#x20; "qr\_image\_url": "https://cdn.../qr.png", // QR image

# &#x20; "payment\_url": "https://pay.../TXN-xxx", // Link thanh toán

# &#x20; "expired\_at": "2024-12-01T09:15:00Z",

# &#x20; "created\_at": "2024-12-01T09:00:00Z"

# }

# ```

# 

# \---

# 

# \#### Query Transaction

# 

# ```http

# GET /v1/payments/{transaction\_ref}

# Authorization: Bearer {secret\_key}

# ```

# 

# Response:

# ```json

# {

# &#x20; "transaction\_ref": "TXN-20241201-ABCDE",

# &#x20; "merchant\_ref": "ORDER-2024-00123",

# &#x20; "status": "completed",

# &#x20; "amount": 150000,

# &#x20; "fee\_amount": 2250,

# &#x20; "net\_amount": 147750,

# &#x20; "payment\_method": "qr",

# &#x20; "payment\_channel": "vcb",

# &#x20; "completed\_at": "2024-12-01T09:03:45Z"

# }

# ```

# 

# \---

# 

# \#### Refund

# 

# ```http

# POST /v1/payments/{transaction\_ref}/refunds

# Authorization: Bearer {secret\_key}

# 

# {

# &#x20; "amount": 150000,      // Có thể refund một phần

# &#x20; "reason": "customer\_request",

# &#x20; "merchant\_ref": "REFUND-ORDER-00123"

# }

# ```

# 

# \---

# 

# \#### Query Settlement

# 

# ```http

# GET /v1/settlements?from=2024-12-01\&to=2024-12-31

# Authorization: Bearer {secret\_key}

# ```

# 

# \---

# 

# \### 6.2 Webhook

# 

# Webhook là cơ chế \*\*hệ thống chủ động thông báo\*\* đến merchant khi có sự kiện xảy ra. Merchant không cần polling liên tục.

# 

# \#### Các events quan trọng

# 

# | Event | Khi nào trigger |

# |---|---|

# | `payment.completed` | Giao dịch thành công |

# | `payment.failed` | Giao dịch thất bại |

# | `payment.expired` | QR/link hết hạn |

# | `refund.completed` | Hoàn tiền thành công |

# | `refund.failed` | Hoàn tiền thất bại |

# | `settlement.completed` | Đã chuyển tiền về ngân hàng |

# | `chargeback.received` | Nhận khiếu nại từ khách hàng |

# | `merchant.suspended` | Tài khoản bị tạm khóa |

# 

# \#### Webhook payload

# 

# ```json

# POST https://merchant-shop.com/webhooks/payment

# Headers:

# &#x20; X-Webhook-Signature: sha256=abcdef123456...

# &#x20; X-Event-Type: payment.completed

# &#x20; X-Delivery-ID: wh\_01HXXX

# 

# Body:

# {

# &#x20; "event\_type": "payment.completed",

# &#x20; "event\_id": "evt\_01HXXX",

# &#x20; "created\_at": "2024-12-01T09:03:45Z",

# &#x20; "data": {

# &#x20;   "transaction\_ref": "TXN-20241201-ABCDE",

# &#x20;   "merchant\_ref": "ORDER-2024-00123",

# &#x20;   "status": "completed",

# &#x20;   "amount": 150000,

# &#x20;   "net\_amount": 147750,

# &#x20;   "completed\_at": "2024-12-01T09:03:45Z"

# &#x20; }

# }

# ```

# 

# \#### Webhook Signature Verification

# 

# Merchant phải \*\*verify chữ ký\*\* để đảm bảo webhook đến từ hệ thống thực sự, không phải kẻ tấn công.

# 

# ```python

# import hmac

# import hashlib

# 

# def verify\_webhook(payload\_body: bytes, signature\_header: str, secret: str) -> bool:

# &#x20;   expected = hmac.new(

# &#x20;       key=secret.encode(),

# &#x20;       msg=payload\_body,

# &#x20;       digestmod=hashlib.sha256

# &#x20;   ).hexdigest()

# 

# &#x20;   received = signature\_header.replace("sha256=", "")

# &#x20;   return hmac.compare\_digest(expected, received)

# ```

# 

# \#### Webhook Retry Policy

# 

# Nếu merchant server trả về status khác 2xx → hệ thống retry:

# 

# ```

# Attempt 1: Ngay lập tức

# Attempt 2: 1 phút sau

# Attempt 3: 5 phút sau

# Attempt 4: 30 phút sau

# Attempt 5: 2 giờ sau

# Attempt 6: 24 giờ sau

# → Nếu vẫn fail: mark webhook failed, alert merchant

# ```

# 

# \---

# 

# \### 6.3 Idempotency — Tránh xử lý trùng lặp

# 

# Merchant có thể gọi API nhiều lần (retry khi timeout). Hệ thống dùng \*\*Idempotency Key\*\* để đảm bảo chỉ tạo một giao dịch dù gọi nhiều lần.

# 

# ```http

# POST /v1/payments

# Idempotency-Key: ORDER-2024-00123-attempt-1

# 

# {...}

# ```

# 

# \- Lần đầu: tạo payment mới

# \- Lần sau với cùng key: trả về response cũ, không tạo mới

# 

# \---

# 

# \## 7. Bảo mật trong Merchant System

# 

# \### 7.1 PCI-DSS Compliance

# 

# Nếu merchant xử lý dữ liệu thẻ, phải tuân thủ \*\*PCI-DSS\*\* (Payment Card Industry Data Security Standard).

# 

# Nguyên tắc chính:

# \- \*\*Không bao giờ\*\* lưu CVV

# \- \*\*Không lưu\*\* full card number dạng plaintext

# \- Card number phải được \*\*tokenized\*\* (thay bằng token vô nghĩa)

# \- Môi trường xử lý thẻ phải isolated

# 

# ```

# Card number: 4111 1111 1111 1234

# &#x20;         ↓  Tokenization

# Token:      tok\_live\_4xNzM8p...

# 

# → Lưu token, không lưu card number

# → Khi charge: gửi token, không gửi card number

# ```

# 

# \### 7.2 API Key Management

# 

# ```

# \- Secret key chỉ dùng ở server-side, KHÔNG expose ở frontend

# \- Hỗ trợ key rotation (tạo key mới, vô hiệu hóa key cũ)

# \- Key có prefix để phân biệt môi trường:

# &#x20; pk\_test\_xxx / sk\_test\_xxx  → Sandbox

# &#x20; pk\_live\_xxx / sk\_live\_xxx  → Production

# \- Log mọi API call với key nào gọi

# ```

# 

# \### 7.3 Fraud Detection

# 

# Các signal phổ biến để phát hiện fraud:

# 

# ```

# \- Nhiều giao dịch failed liên tiếp (card testing)

# \- Số tiền bất thường (quá lớn so với lịch sử)

# \- Nhiều giao dịch từ cùng IP/device ngắn thời gian

# \- Địa chỉ billing và shipping khác xa nhau

# \- Merchant mới + transaction lớn ngay lập tức

# \- Chargeback rate > 1%

# ```

# 

# \---

# 

# \## 8. Merchant Lifecycle — Tổng quan

# 

# ```

# &#x20;                   ┌──────────────┐

# &#x20;                   │  REGISTERED  │ ← Merchant điền form

# &#x20;                   └──────┬───────┘

# &#x20;                          │

# &#x20;                   ┌──────▼───────┐

# &#x20;                   │ UNDER REVIEW │ ← KYC/KYB đang xét duyệt

# &#x20;                   └──────┬───────┘

# &#x20;                   ┌──────┴───────┐

# &#x20;                   │   REJECTED   │ ← Không đủ điều kiện

# &#x20;                   └──────────────┘

# &#x20;                          │ Approved

# &#x20;                   ┌──────▼───────┐

# &#x20;                   │    ACTIVE    │ ← Đang hoạt động bình thường

# &#x20;                   └──────┬───────┘

# &#x20;             ┌────────────┼────────────┐

# &#x20;             │            │            │

# &#x20;      ┌──────▼───┐  ┌─────▼──────┐  ┌─▼──────────┐

# &#x20;      │SUSPENDED │  │UNDER WATCH │  │  DORMANT   │

# &#x20;      │(tạm khóa)│  │(cảnh báo)  │  │(không dùng)│

# &#x20;      └──────┬───┘  └─────┬──────┘  └────────────┘

# &#x20;             │             │

# &#x20;             └──────┬──────┘

# &#x20;                    │

# &#x20;             ┌──────▼───────┐

# &#x20;             │  TERMINATED  │ ← Đóng vĩnh viễn

# &#x20;             └──────────────┘

# ```

# 

# \---

# 

# \## 9. Checklist khi build Merchant System

# 

# \### Thiết kế

# 

# \- \[ ] Phân biệt rõ Merchant entity và User account (một merchant có thể nhiều user)

# \- \[ ] Thiết kế multi-currency từ đầu nếu cần

# \- \[ ] Tách biệt môi trường sandbox và production ngay từ đầu

# 

# \### Tài chính

# 

# \- \[ ] Mọi phép tính tiền dùng `DECIMAL`, tuyệt đối không dùng `FLOAT`

# \- \[ ] Lưu cả `gross\_amount`, `fee\_amount`, `net\_amount` — không tính lại từ rate

# \- \[ ] Audit log cho mọi thay đổi về số dư và settlement

# 

# \### API

# 

# \- \[ ] Implement idempotency key cho mọi write endpoint

# \- \[ ] Version API từ đầu (`/v1/`)

# \- \[ ] Rate limiting theo merchant ID

# \- \[ ] Webhook có retry + signature verification

# 

# \### Bảo mật

# 

# \- \[ ] Không log sensitive data (card number, secret key)

# \- \[ ] API key có thể revoke ngay lập tức

# \- \[ ] Mọi action quan trọng cần audit trail

# 

# \---

# 

# \## 10. Tóm tắt

# 

# ```

# Merchant = Đơn vị bán hàng nhận thanh toán qua hệ thống payment

# 

# Vòng đời:  Đăng ký → KYC → Active → Giao dịch → Settlement

# 

# Dòng tiền: Customer thanh toán → Gateway giữ → Settlement → Ngân hàng merchant

# 

# Key concepts:

# &#x20; MID       — Mã định danh merchant

# &#x20; MCC       — Mã ngành nghề

# &#x20; MDR       — Phí % trên giao dịch

# &#x20; Settlement — Chuyển tiền định kỳ về tài khoản

# &#x20; Chargeback — Khách hàng khiếu nại, bị trừ tiền bắt buộc

# &#x20; KYC/KYB   — Xác minh danh tính trước khi hoạt động

# &#x20; Webhook   — Hệ thống chủ động notify merchant khi có event

# ```

