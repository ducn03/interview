# 💸 Fintech Domain — Business & Technical Deep Dive

> Tài liệu này bao quát toàn bộ **Fintech core domains**: Wallet & Balance System, Reconciliation, và Lending/BNPL. Mỗi phần đi song song cả business (nghiệp vụ, tại sao) và technical (thiết kế hệ thống, data model).

---

## PHẦN 1: BỨC TRANH TỔNG THỂ FINTECH

### 1.1 Fintech là gì — và rộng đến đâu?

**Fintech (Financial Technology)** là ứng dụng công nghệ vào dịch vụ tài chính. Không phải một ngành — mà là **một tập hợp các domain** giải quyết bài toán khác nhau trong tài chính.

```
                        FINTECH
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   PAYMENTS           LENDING            WEALTH MGMT
   ────────           ───────            ──────────
   Payment Gateway    Consumer Loan      Robo-advisor
   Wallet/E-money     BNPL               Stock Trading
   Money Transfer     SME Lending        Crypto
   QR/POS             Mortgage           Insurance
        │                  │
   BANKING             INFRASTRUCTURE
   ───────             ──────────────
   Neobank             KYC/Identity
   Core Banking        Fraud Detection
   Open Banking        Reconciliation
   BaaS                Compliance/AML
```

### 1.2 Tại sao Fintech phức tạp hơn các domain khác?

```
Domain thông thường:       Fintech:
─────────────────────      ─────────────────────────────
Bug → Fix lại              Bug → Có thể mất tiền thật
Downtime → Phiền phức      Downtime → Mất giao dịch, vi phạm SLA
Data sai → Sửa được        Data sai → Sai số dư → Kiện tụng
Scale → Thêm server        Scale → Cần đảm bảo consistency tuyệt đối
```

**Ba tính chất bất khả xâm phạm trong Fintech:**

```
1. CORRECTNESS (Chính xác):
   Số tiền phải luôn đúng, không được sai dù 1 đồng

2. CONSISTENCY (Nhất quán):
   Tổng tiền trong hệ thống phải luôn bằng nhau
   Tiền không tự nhiên sinh ra hay mất đi

3. AUDITABILITY (Có thể kiểm tra):
   Mọi giao dịch đều có dấu vết, ai làm gì lúc nào
```

---

## PHẦN 2: WALLET & BALANCE SYSTEM

### 2.1 Wallet là gì trong Fintech?

**Wallet (Ví điện tử)** là hệ thống lưu trữ và quản lý **số dư tiền của người dùng** dưới dạng điện tử, cho phép thực hiện các giao dịch tài chính mà không cần tiền mặt hoặc tài khoản ngân hàng trực tiếp.

**Wallet khác tài khoản ngân hàng thế nào?**

| | Tài khoản ngân hàng | Ví điện tử |
|---|---|---|
| Ai cấp phép | NHNN cấp phép ngân hàng | NHNN cấp phép TGTT (Trung gian thanh toán) |
| Bảo hiểm tiền gửi | Có (tối đa 125 triệu) | Không |
| Lãi suất | Có (tiết kiệm) | Thường không |
| Hạn mức | Không giới hạn | Giới hạn theo quy định (thường 100tr VND) |
| KYC | Đầy đủ, mặt đối mặt | eKYC, đơn giản hơn |
| Ví dụ VN | Vietcombank, Techcombank | Momo, ZaloPay, VNPay |

### 2.2 Các loại Wallet

#### Prepaid Wallet (Ví trả trước)
Nạp tiền trước, dùng dần. Số dư không được âm.
```
Ví dụ: Momo, ZaloPay, ShopeePay
Luồng: Nạp tiền từ ngân hàng → Số dư tăng → Dùng để thanh toán → Số dư giảm
```

#### Postpaid Wallet / Credit Wallet
Chi tiêu trước, trả sau. Có hạn mức tín dụng.
```
Ví dụ: Ví tín dụng, BNPL wallet
Luồng: Dùng để thanh toán → Nợ tăng → Trả nợ vào cuối kỳ
```

#### Custodial vs Non-custodial (chủ yếu trong crypto)
```
Custodial: Bên thứ 3 giữ private key thay bạn (Binance, Coinbase)
Non-custodial: Bạn tự giữ private key (MetaMask)
```

### 2.3 Double-Entry Ledger — Nền tảng của mọi Wallet

Đây là **khái niệm quan trọng nhất** cần nắm khi thiết kế Wallet. Tương tự kế toán doanh nghiệp, nhưng áp dụng ở cấp độ từng giao dịch người dùng.

**Nguyên tắc:** Tiền không tự nhiên sinh ra hay mất đi. Mỗi giao dịch phải có nguồn tiền ĐI và nơi tiền ĐẾN.

```
Ví dụ: User A nạp 100,000đ vào ví từ ngân hàng

KHÔNG làm thế này (single-entry):
  UPDATE wallets SET balance = balance + 100000 WHERE user_id = A
  → Không biết tiền từ đâu, không thể audit

LÀM THẾ NÀY (double-entry):
  DEBIT  (trừ): Tài khoản ngân hàng của A    -100,000đ
  CREDIT (cộng): Ví điện tử của A            +100,000đ
  → Tổng thay đổi = 0, tiền chỉ di chuyển, không mất
```

**Chart of Accounts trong Wallet System:**

```
ASSET ACCOUNTS (Tài sản — tiền thực):
  1001: Float Account (Tiền thực tại ngân hàng đối tác)
  1002: Settlement Pending

LIABILITY ACCOUNTS (Nợ phải trả — tiền của user):
  2001: User Wallets (Tổng số dư của tất cả user)
  2002: Merchant Wallets
  2003: Pending Transactions

REVENUE ACCOUNTS:
  4001: Transaction Fees
  4002: FX Spread (nếu có đổi tiền tệ)

EXPENSE ACCOUNTS:
  5001: Bank Charges
  5002: Refunds
```

**Nguyên tắc bất biến:**
```
SUM(Asset Accounts) = SUM(Liability Accounts) + SUM(Equity)

Nếu công thức này không cân bằng → Có bug hoặc fraud trong hệ thống
```

### 2.4 Các loại giao dịch trong Wallet

```
TOPUP (Nạp tiền):
  Debit:  Float Account (tiền từ bank vào hệ thống)   +100,000
  Credit: User Wallet A                                +100,000
  → Số dư tổng hệ thống tăng 100,000

PAYMENT (Thanh toán):
  Debit:  User Wallet A                               -50,000
  Credit: Merchant Wallet B                           +50,000
  → Số dư tổng hệ thống không đổi (chỉ di chuyển)

TRANSFER P2P (Chuyển tiền):
  Debit:  User Wallet A                               -30,000
  Credit: User Wallet B                               +30,000
  → Số dư tổng hệ thống không đổi

WITHDRAWAL (Rút tiền):
  Debit:  User Wallet A                               -200,000
  Credit: Float Account (tiền ra khỏi hệ thống)      -200,000
  → Số dư tổng hệ thống giảm 200,000

FEE (Thu phí):
  Debit:  User Wallet A                               -1,000
  Credit: Revenue Account (Fee Income)                +1,000
  → Tiền di chuyển từ user sang revenue của platform
```

### 2.5 Balance Architecture — Thiết kế số dư

Không phải chỉ có một con số "balance" — trong thực tế cần nhiều loại số dư:

```
TOTAL BALANCE (Tổng số dư):
  Tất cả tiền trong ví, bao gồm cả tiền đang bị giữ

AVAILABLE BALANCE (Số dư khả dụng):
  Tiền có thể dùng ngay
  = Total Balance - Pending/Reserved Amount

PENDING BALANCE (Số dư đang xử lý):
  Tiền đang trong giao dịch chưa hoàn tất
  Ví dụ: Vừa nhận chuyển khoản nhưng chưa settle

RESERVED BALANCE (Số dư bị giữ):
  Tiền bị hold cho một mục đích cụ thể
  Ví dụ: Đã tạo lệnh thanh toán nhưng chưa confirm
```

**Ví dụ thực tế:**
```
User có:
  Total Balance:     500,000đ
  Pending (nhận):   +100,000đ  (chuyển khoản đang vào)
  Reserved:         -200,000đ  (đang hold cho đơn hàng)

  Available Balance = 500,000 - 200,000 = 300,000đ
  → User chỉ được dùng 300,000đ, dù total là 500,000đ
```

### 2.6 Data Model — Wallet System

```sql
-- Accounts: Mỗi "ví" hay "tài khoản" trong hệ thống
CREATE TABLE accounts (
    id              UUID PRIMARY KEY,
    account_ref     VARCHAR(50) UNIQUE NOT NULL,  -- ACC-USER-00123
    account_type    VARCHAR(30) NOT NULL,
    -- user_wallet | merchant_wallet | float | fee_revenue | ...
    owner_id        UUID,            -- user_id hoặc merchant_id
    currency        CHAR(3) NOT NULL DEFAULT 'VND',
    status          VARCHAR(20) DEFAULT 'active',
    -- active | suspended | closed
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Ledger entries: MỌI giao dịch đều tạo ít nhất 2 dòng ở đây
-- Đây là bảng KHÔNG BAO GIỜ được UPDATE hay DELETE
CREATE TABLE ledger_entries (
    id              UUID PRIMARY KEY,
    entry_ref       VARCHAR(100) UNIQUE NOT NULL,
    transaction_id  UUID NOT NULL REFERENCES transactions(id),
    account_id      UUID NOT NULL REFERENCES accounts(id),
    entry_type      VARCHAR(10) NOT NULL, -- 'debit' | 'credit'
    amount          BIGINT NOT NULL,      -- Lưu bằng đơn vị nhỏ nhất (đồng)
    -- KHÔNG dùng DECIMAL/FLOAT để tránh lỗi làm tròn
    currency        CHAR(3) NOT NULL DEFAULT 'VND',
    balance_after   BIGINT NOT NULL,      -- Số dư sau khi entry này
    description     TEXT,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
    -- KHÔNG có updated_at — ledger là immutable
);

-- Transactions: Nhóm các ledger entries lại thành một giao dịch
CREATE TABLE transactions (
    id              UUID PRIMARY KEY,
    transaction_ref VARCHAR(100) UNIQUE NOT NULL,
    transaction_type VARCHAR(50) NOT NULL,
    -- topup | payment | transfer | withdrawal | fee | refund
    status          VARCHAR(30) NOT NULL,
    -- pending | processing | completed | failed | reversed
    initiator_id    UUID,           -- Ai tạo giao dịch
    amount          BIGINT NOT NULL,
    currency        CHAR(3) NOT NULL DEFAULT 'VND',
    description     TEXT,
    metadata        JSONB,
    idempotency_key VARCHAR(255) UNIQUE, -- Tránh duplicate
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    completed_at    TIMESTAMPTZ
);

-- Account Balances: Cache số dư hiện tại (tính từ ledger)
CREATE TABLE account_balances (
    account_id      UUID PRIMARY KEY REFERENCES accounts(id),
    total_balance   BIGINT NOT NULL DEFAULT 0,
    reserved_amount BIGINT NOT NULL DEFAULT 0,
    -- available = total - reserved (tính khi cần, không lưu riêng)
    last_entry_id   UUID,           -- Entry cuối cùng tạo ra balance này
    updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

**Tại sao lưu amount bằng BIGINT (đồng) thay vì DECIMAL?**
```
DECIMAL(15,2) lưu 1.10đ → Thực ra là 1.0999999... trong IEEE 754
→ Sau hàng triệu giao dịch, sai số tích lũy

BIGINT lưu 110 (xu) → Chính xác tuyệt đối
→ Quy ước: 1đ VND = 1 đơn vị, hoặc 1đ = 100 xu (nếu có phần nhỏ)
```

### 2.7 Race Condition — Vấn đề nguy hiểm nhất

**Kịch bản nguy hiểm:**

```
User có 100,000đ trong ví.
Đồng thời 2 request thanh toán 80,000đ:

Thread 1:                    Thread 2:
Read balance: 100,000        Read balance: 100,000
Check: 100k >= 80k? YES      Check: 100k >= 80k? YES
...                          Deduct 80k → balance = 20,000
Deduct 80k → balance = 20,000
→ Cả hai đều thành công!
→ Tổng chi: 160,000đ > 100,000đ ban đầu
→ Ví bị âm → LỖ TIỀN THẬT
```

**Giải pháp 1: Optimistic Locking**

```sql
-- Khi update balance, check version không thay đổi
UPDATE account_balances
SET total_balance = total_balance - 80000,
    version = version + 1,
    updated_at = NOW()
WHERE account_id = 'ACC-123'
  AND version = 5          -- version lúc đọc
  AND total_balance >= 80000;

-- Nếu affected rows = 0 → Version đã thay đổi → Retry
-- Nếu affected rows = 1 → Thành công
```

**Giải pháp 2: Database-level Locking (SELECT FOR UPDATE)**

```sql
BEGIN;
  SELECT total_balance, reserved_amount
  FROM account_balances
  WHERE account_id = 'ACC-123'
  FOR UPDATE;  -- Lock row này lại

  -- Kiểm tra available balance
  -- Nếu đủ → tạo ledger entry + update balance
  -- Nếu không đủ → ROLLBACK

COMMIT;
-- Lock giải phóng sau COMMIT/ROLLBACK
```

**Giải pháp 3: Idempotency Key**

```
Mọi giao dịch đều có idempotency_key duy nhất.
Nếu cùng key được gửi 2 lần → Chỉ xử lý 1 lần, lần sau trả response cũ.

Client: POST /transfer
        Idempotency-Key: TXN-USER-123-ORDER-456-20241201

Server lần 1: Key chưa tồn tại → Xử lý → Lưu key + result
Server lần 2: Key đã tồn tại → Trả result cũ → Không xử lý lại
```

### 2.8 Transaction State Machine

```
                    ┌─────────┐
                    │ PENDING │ ← Giao dịch vừa được tạo
                    └────┬────┘
                         │
                    ┌────▼────┐
                    │PROCESSING│ ← Đang xử lý (reserve balance)
                    └────┬────┘
              ┌──────────┼──────────┐
              ▼          ▼          ▼
        ┌──────────┐ ┌────────┐ ┌────────┐
        │COMPLETED │ │ FAILED │ │EXPIRED │
        └────┬─────┘ └────────┘ └────────┘
             │
        ┌────▼─────┐
        │ REVERSED │ ← Giao dịch bị đảo ngược (refund, dispute)
        └──────────┘

Quy tắc:
  PENDING → PROCESSING → COMPLETED ✓ (happy path)
  PENDING → PROCESSING → FAILED    ✓ (release reserved balance)
  PENDING → EXPIRED                ✓ (timeout)
  COMPLETED → REVERSED             ✓ (refund/chargeback)
  COMPLETED → COMPLETED            ✗ KHÔNG được phép
  FAILED → COMPLETED               ✗ KHÔNG được phép
```

### 2.9 Float Management — Tiền thật ở đâu?

Khi 1 triệu user mỗi người có 100,000đ trong ví → Hệ thống đang "giữ" 100 tỷ đồng tiền thật.

**Tiền này để ở đâu?**

```
Toàn bộ tiền user gửi vào ví
    → Được gom vào tài khoản ngân hàng tập trung
       gọi là FLOAT ACCOUNT (hoặc Escrow Account)
    → Phải tách biệt hoàn toàn với tiền vận hành của công ty
    → NHNN yêu cầu: Tiền người dùng phải được đảm bảo 1:1
       (không được dùng để đầu tư hay vận hành)

Bất cứ lúc nào:
  SUM(tất cả số dư ví người dùng) = Số tiền trong Float Account

Nếu không cân bằng → Vi phạm quy định → Có thể mất giấy phép
```

---

## PHẦN 3: RECONCILIATION — ĐỐI SOÁT

### 3.1 Reconciliation là gì và tại sao quan trọng?

**Reconciliation (Đối soát)** là quy trình **đối chiếu, so sánh dữ liệu** giữa hai hoặc nhiều hệ thống để đảm bảo chúng khớp nhau.

**Tại sao cần Reconciliation?**

```
Trong một giao dịch thanh toán, có ít nhất 3-4 hệ thống tham gia:
  1. Hệ thống của bạn (internal)
  2. Payment Gateway
  3. Acquiring Bank
  4. Card Network

Mỗi hệ thống tự ghi nhận giao dịch theo cách riêng.
Không có gì đảm bảo chúng ghi nhận giống nhau 100%.

Sự cố thực tế:
  - Network timeout: Bạn ghi nhận "failed", Gateway ghi nhận "success"
  - Duplicate charge: Gateway charge 2 lần, bạn chỉ biết 1 lần
  - Settlement sai: Bank chuyển 98,500 nhưng hệ thống expect 99,000
  - Giao dịch "ma": Có trong bank statement nhưng không có trong hệ thống
```

**Hậu quả nếu không Reconcile:**
```
Mất tiền (không biết)          → Thiệt hại tài chính trực tiếp
Báo cáo tài chính sai          → Quyết định kinh doanh sai
Vi phạm quy định               → Phạt tiền, mất giấy phép
Khách hàng bị charge sai       → Khiếu nại, chargeback
Merchant nhận sai tiền          → Khiếu nại, mất đối tác
```

### 3.2 Các loại Reconciliation

#### Internal Reconciliation (Đối soát nội bộ)

Đối chiếu giữa các bảng/hệ thống trong nội bộ:

```
Kiểm tra 1: Transaction table vs Ledger entries
  Mỗi transaction "completed" phải có đủ ledger entries
  Tổng debit = Tổng credit trong mỗi transaction

Kiểm tra 2: Ledger entries vs Account balances
  Balance hiện tại của account = SUM(tất cả entries của account)

Kiểm tra 3: Tổng balance người dùng vs Float Account
  SUM(user wallets) = Float account balance tại ngân hàng
```

#### External Reconciliation (Đối soát bên ngoài)

Đối chiếu hệ thống nội bộ với đối tác bên ngoài:

```
Internal vs Payment Gateway:
  So sánh từng giao dịch: ID, amount, status, timestamp

Internal vs Bank Statement:
  So sánh số tiền thực tế nhận/gửi với ghi nhận nội bộ

Internal vs Merchant:
  Đối chiếu số tiền đã settlement với merchant
```

### 3.3 Settlement Reconciliation — Chi tiết nhất

Đây là loại reconciliation phức tạp và quan trọng nhất trong Payment.

**Luồng tiền cần reconcile:**

```
Ngày T: 1000 giao dịch thành công
  Tổng amount: 500,000,000đ
  Tổng fee (MDR 1.5%): 7,500,000đ
  Expected settlement: 492,500,000đ

Ngày T+1: Bank chuyển khoản về
  Actual received: 492,350,000đ

Chênh lệch: 150,000đ — đi đâu?
  → Có thể: Phí chuyển khoản ngân hàng?
             Tỷ giá FX rounding?
             Một giao dịch bị tính sai fee?
             → Phải điều tra từng dòng
```

**Quy trình Reconciliation từng bước:**

```
[1] DATA COLLECTION (Thu thập dữ liệu)
    Thu thập từ tất cả nguồn:
    - Export từ hệ thống nội bộ
    - Download file từ Payment Gateway (CSV/SFTP)
    - Download Bank Statement (MT940, CSV)
    - File settlement từ Acquirer

[2] DATA NORMALIZATION (Chuẩn hóa dữ liệu)
    Vấn đề: Mỗi nguồn có format khác nhau
    
    Hệ thống nội bộ:
      txn_id: TXN-20241201-ABCDE
      amount: 150000
      status: completed
    
    Gateway:
      reference: PAY-2024-12-01-XYZ
      gross_amount: 150000
      net_amount: 147750
      status: SUCCESS
    
    → Phải map: TXN-20241201-ABCDE ↔ PAY-2024-12-01-XYZ
    → Chuẩn hóa về cùng format để so sánh

[3] MATCHING (Khớp lệnh)
    So sánh từng cặp giao dịch:
    
    MATCHED ✓: Tìm thấy trong cả hai nguồn, amount khớp
    UNMATCHED:
      - Có trong internal, không có trong gateway
      - Có trong gateway, không có trong internal
      - Có trong cả hai nhưng amount khác nhau
      - Có trong cả hai nhưng status khác nhau

[4] EXCEPTION HANDLING (Xử lý ngoại lệ)
    Phân loại từng exception:
    
    TYPE A — Timing difference (Chênh lệch thời gian):
      Giao dịch cuối ngày → Gateway ghi ngày T,
      Internal ghi ngày T+1 do timezone
      → Không phải lỗi, tự match ngày hôm sau
    
    TYPE B — Pending settlement:
      Giao dịch T đang chờ settle → Chưa có trong bank statement
      → Theo dõi, expect ngày T+1 hoặc T+2
    
    TYPE C — True exception (Lỗi thật):
      Amount khác nhau → Điều tra ngay
      Có ở gateway không có ở internal → Giao dịch "ma"?
      Có ở internal không có ở gateway → Hệ thống ghi nhận sai?

[5] RESOLUTION (Giải quyết)
    Mỗi exception có action cụ thể:
      Giao dịch "ma" → Điều tra fraud team
      Amount sai → Điều chỉnh kế toán, liên hệ gateway
      Duplicate → Refund cho user, thu hồi từ merchant

[6] SIGN-OFF (Xác nhận hoàn tất)
    Manager review và approve
    Ghi nhận vào hệ thống: "Recon ngày T: BALANCED"
    Lưu report + evidence
```

### 3.4 Data Model — Reconciliation System

```sql
-- Recon Run: Một lần chạy đối soát
CREATE TABLE recon_runs (
    id              UUID PRIMARY KEY,
    recon_type      VARCHAR(50) NOT NULL,
    -- settlement | internal_ledger | bank_statement | gateway
    period_date     DATE NOT NULL,
    source_a        VARCHAR(100) NOT NULL,  -- 'internal'
    source_b        VARCHAR(100) NOT NULL,  -- 'gateway_vnpay'
    status          VARCHAR(30) DEFAULT 'pending',
    -- pending | running | completed | failed | signed_off
    total_records_a INTEGER,
    total_records_b INTEGER,
    matched_count   INTEGER DEFAULT 0,
    unmatched_count INTEGER DEFAULT 0,
    total_amount_a  BIGINT,
    total_amount_b  BIGINT,
    variance_amount BIGINT,  -- A - B
    run_by          UUID,
    signed_off_by   UUID,
    signed_off_at   TIMESTAMPTZ,
    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Recon Items: Từng cặp giao dịch được đối chiếu
CREATE TABLE recon_items (
    id              UUID PRIMARY KEY,
    recon_run_id    UUID NOT NULL REFERENCES recon_runs(id),
    match_status    VARCHAR(30) NOT NULL,
    -- matched | unmatched_a_only | unmatched_b_only | amount_mismatch | status_mismatch

    -- Source A (internal)
    internal_txn_id UUID,
    internal_ref    VARCHAR(255),
    internal_amount BIGINT,
    internal_status VARCHAR(30),
    internal_date   TIMESTAMPTZ,

    -- Source B (external)
    external_ref    VARCHAR(255),
    external_amount BIGINT,
    external_status VARCHAR(30),
    external_date   TIMESTAMPTZ,
    external_fee    BIGINT,

    -- Variance
    amount_variance BIGINT,  -- internal_amount - external_amount

    -- Resolution
    resolution_status VARCHAR(30) DEFAULT 'open',
    -- open | investigating | resolved | accepted_variance
    resolution_note TEXT,
    resolved_by     UUID,
    resolved_at     TIMESTAMPTZ,

    created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

### 3.5 Reconciliation Dashboard — Những gì cần monitor

```
DAILY RECON DASHBOARD — 01/12/2024
────────────────────────────────────────────────────────
SETTLEMENT RECONCILIATION:
  Transactions processed:     2,847
  Matched:                    2,831  (99.4%) ✓
  Unmatched:                     16  (0.6%)
    - Amount mismatch:              8
    - Internal only:                5
    - External only:                3

  Total Amount (Internal):    892,450,000đ
  Total Amount (Gateway):     892,437,500đ
  Variance:                    12,500đ ⚠️

ACTION REQUIRED:
  [URGENT] 3 transactions có trong gateway, không có internal
  → Có thể mất ghi nhận giao dịch — điều tra ngay
  
  [NORMAL] 8 amount mismatch — likely FX rounding
  → Review và accept nếu < 1,000đ/giao dịch
────────────────────────────────────────────────────────
```

---

## PHẦN 4: LENDING & BNPL

### 4.1 Lending là gì trong Fintech?

**Lending (Cho vay)** là hoạt động cung cấp tiền cho người đi vay với điều kiện **hoàn trả gốc + lãi** theo thời hạn thỏa thuận.

**Tại sao Lending là core business của Fintech?**

```
Net Interest Margin (NIM):
  Lãi suất cho vay:  15%/năm
  Chi phí vốn:        5%/năm (lãi suất huy động)
  NIM:               10%/năm → Đây là lợi nhuận cốt lõi

So sánh với Payment:
  MDR Payment: 1.5% một lần trên giao dịch
  Lending: 15%/năm liên tục → Profitable hơn nhiều
  
→ Mọi nền tảng Fintech lớn đều hướng đến Lending
```

### 4.2 Các loại Lending trong Fintech

#### Consumer Lending (Vay tiêu dùng cá nhân)

```
Đặc điểm:
  Người vay: Cá nhân
  Mục đích: Mua sắm, sửa nhà, chi tiêu cá nhân
  Số tiền: 1tr – 100tr VND
  Kỳ hạn: 3 – 36 tháng
  Lãi suất: 15 – 50%/năm (tùy rủi ro)
  
Ví dụ: FE Credit, Home Credit, MCredit, Vay qua Momo
```

#### SME Lending (Vay doanh nghiệp vừa và nhỏ)

```
Đặc điểm:
  Người vay: Doanh nghiệp SME
  Mục đích: Vốn lưu động, mua hàng tồn kho, mở rộng
  Số tiền: 50tr – 5 tỷ VND
  Kỳ hạn: 3 – 24 tháng
  Tài sản đảm bảo: Hàng tồn kho, công nợ, TSCĐ (hoặc tín chấp)
  
Ví dụ: Funding Societies, Validus, KiotViet Capital
```

#### Embedded Lending (Vay nhúng vào sản phẩm khác)

```
Đặc điểm:
  Vay ngay tại điểm mua hàng hoặc trong app
  Không cần đến ngân hàng
  Quyết định trong vài giây đến vài phút
  
Ví dụ:
  Mua hàng trên Shopee → ShopeePayLater
  Đặt đồ ăn Grab → GrabPaylater
  Merchant dùng VNPAY → Vay vốn lưu động ngay trên dashboard
```

### 4.3 BNPL — Buy Now Pay Later

**BNPL** là một dạng tín dụng ngắn hạn cho phép người mua **chia nhỏ thanh toán** thành nhiều đợt, thường không tính lãi (hoặc lãi rất thấp) nếu trả đúng hạn.

**Tại sao BNPL bùng nổ?**

```
Góc độ Người mua:
  - Không cần thẻ tín dụng
  - Không cần chứng minh thu nhập phức tạp
  - Phê duyệt trong vài giây
  - Không lãi nếu trả đúng hạn

Góc độ Merchant:
  - Tăng conversion rate (khách dám mua hàng giá cao hơn)
  - Tăng AOV (Average Order Value)
  - BNPL provider chịu rủi ro tín dụng, không phải merchant

Góc độ BNPL Provider:
  - Thu MDR từ merchant (thường 3-8%, cao hơn thẻ)
  - Thu lãi phạt nếu user trả trễ
  - Thu subscription fee (một số mô hình)
```

**Các mô hình BNPL phổ biến:**

```
PAY IN 4 (Trả 4 lần):
  Mua hàng 1,200,000đ
  Trả ngay:          300,000đ (25%)
  Trả sau 2 tuần:    300,000đ
  Trả sau 4 tuần:    300,000đ
  Trả sau 6 tuần:    300,000đ
  Không lãi nếu đúng hạn

PAY LATER (Trả sau 30 ngày):
  Mua hàng → Trả toàn bộ sau 30 ngày
  Không lãi trong 30 ngày
  Quá hạn: lãi phạt ~2-3%/tháng

INSTALLMENT (Trả góp có lãi):
  Mua hàng 10,000,000đ
  Trả góp 6 tháng × 1,800,000đ
  (Bao gồm lãi suất 15%/năm)
```

### 4.4 Credit Risk — Đánh giá rủi ro tín dụng

Đây là **core competency** của Lending — quyết định ai được vay, vay bao nhiêu, lãi suất bao nhiêu.

**5C của tín dụng (Khung đánh giá truyền thống):**

```
CHARACTER (Tính cách, ý chí trả nợ):
  Lịch sử tín dụng trước đây
  Đã từng vỡ nợ, nợ xấu?
  Thời gian quan hệ với tổ chức tín dụng
  → Nguồn: CIC (Credit Information Center - VN), bureau khác

CAPACITY (Khả năng trả nợ):
  Thu nhập hàng tháng
  Tổng nợ hiện tại / Thu nhập (Debt-to-Income ratio)
  DTI < 40%: Tốt
  DTI 40-50%: Cảnh báo
  DTI > 50%: Từ chối
  → Nguồn: Sao kê ngân hàng, hợp đồng lao động

CAPITAL (Tài sản, vốn):
  Tổng tài sản của người vay
  Tiết kiệm, đầu tư, bất động sản
  → "Nếu mất việc, họ có thể dùng gì để trả?"

COLLATERAL (Tài sản đảm bảo):
  Tài sản thế chấp (nếu có): nhà, xe, hàng tồn kho
  LTV (Loan-to-Value): Vay 700tr, nhà 1 tỷ → LTV 70%
  LTV càng thấp → Càng an toàn

CONDITIONS (Điều kiện bên ngoài):
  Tình hình kinh tế vĩ mô
  Ngành nghề của người vay (có rủi ro không?)
  Mục đích vay
```

**Credit Scoring trong Fintech hiện đại:**

```
Dữ liệu truyền thống (Traditional Data):
  Lịch sử tín dụng CIC
  Thu nhập, việc làm
  Tài sản đảm bảo

Alternative Data (Dữ liệu thay thế — Fintech đặc trưng):
  Lịch sử giao dịch trên ví/app:
    → Tần suất giao dịch, số tiền, merchant category
    → "Người này chi tiêu có kiểm soát không?"
  
  Lịch sử thanh toán hóa đơn:
    → Điện, nước, internet — trả đúng hạn không?
  
  Social data (nếu cho phép):
    → Mạng lưới kết nối, thời gian dùng app
  
  Merchant data (với SME lending):
    → Doanh thu POS, lịch sử settlement
    → "Shop này bán được không?"
  
  Device data:
    → Thiết bị mới hay cũ, thay đổi SIM thường xuyên không?

→ Kết hợp tất cả → Credit Score 300-850 (tương tự FICO)
→ Score cao → Lãi suất thấp, hạn mức cao
→ Score thấp → Lãi suất cao, hạn mức thấp, hoặc từ chối
```

### 4.5 Loan Lifecycle — Vòng đời khoản vay

```
[1] APPLICATION (Đăng ký vay)
    User nhập thông tin:
      Số tiền muốn vay, kỳ hạn, mục đích
      CCCD/CMND
      Thông tin việc làm, thu nhập
    
    Upload documents:
      Sao kê ngân hàng 3 tháng
      Hợp đồng lao động
      (hoặc eKYC + data từ hệ thống)

[2] UNDERWRITING (Thẩm định)
    Auto decision (vài giây - vài phút):
      Credit score check
      Fraud check (có phải người dùng thật không?)
      Rule-based: DTI, income threshold
      ML model scoring
    
    Manual review (nếu cần):
      Hồ sơ không rõ ràng
      Khoản vay lớn
      Flag từ fraud detection
    
    Kết quả:
      APPROVED: Số tiền, lãi suất, kỳ hạn được phê duyệt
      COUNTER-OFFER: Phê duyệt một phần (ít hơn, ngắn hơn)
      REJECTED: Lý do từ chối

[3] OFFER & ACCEPTANCE (Chào vay & Chấp nhận)
    Hiển thị loan offer:
      Số tiền: 5,000,000đ
      Kỳ hạn: 6 tháng
      Lãi suất: 18%/năm = 1.5%/tháng
      Phí giải ngân: 1% = 50,000đ
      Số tiền nhận thực: 4,950,000đ
      Lịch trả hàng tháng:
        Tháng 1: 875,000đ (gốc 833,333 + lãi 75,000 - phí 33,333)
        ...
    
    User ký điện tử hợp đồng (e-signature)

[4] DISBURSEMENT (Giải ngân)
    Chuyển tiền vào ví hoặc tài khoản ngân hàng
    Ghi nhận: Khoản vay active
    Bút toán kế toán:
      Debit: Loan Receivable (Dư nợ cho vay)   +5,000,000
      Credit: Funding Account (Nguồn vốn)       -5,000,000

[5] REPAYMENT (Trả nợ)
    Theo lịch hàng tháng:
    
    Trả đúng hạn:
      Debit: Cash/Wallet (tiền vào)             +875,000
      Credit: Loan Receivable (giảm gốc)        -833,333
      Credit: Interest Income (lãi)              -41,667
                                                (điều chỉnh theo amortization)
    
    Trả trễ:
      Tính thêm late fee
      Update credit score (negative)
      Trigger collection process

[6] COLLECTION (Thu hồi nợ xấu)
    DPD (Days Past Due) — số ngày quá hạn:
    
    DPD 1-30:   SMS/app reminder tự động
    DPD 31-60:  Gọi điện nhắc nhở
    DPD 61-90:  Cảnh báo chính thức, phí phạt tăng
    DPD 91-180: Chuyển sang đội thu hồi nợ chuyên nghiệp
    DPD > 180:  Write-off (xóa nợ khỏi sổ sách), bán nợ cho AMC

[7] CLOSURE (Tất toán)
    Trả hết gốc + lãi + phí (nếu có)
    Giải phóng tài sản đảm bảo (nếu có)
    Update credit history
    Loan status → CLOSED
```

### 4.6 Loan Amortization — Tính lịch trả nợ

**Amortization** là bảng tính chi tiết lịch trả nợ hàng kỳ.

```
Khoản vay: 6,000,000đ, 6 tháng, lãi suất 18%/năm (1.5%/tháng)
Phương pháp: Equal installment (PMT cố định)

PMT = P × [r(1+r)^n] / [(1+r)^n - 1]
    = 6,000,000 × [0.015 × 1.015^6] / [1.015^6 - 1]
    = 6,000,000 × 0.01640 / 0.09344
    = 1,051,512đ/tháng

AMORTIZATION TABLE:
Tháng │ PMT       │ Lãi       │ Gốc       │ Dư nợ còn
──────┼───────────┼───────────┼───────────┼───────────
0     │           │           │           │ 6,000,000
1     │ 1,051,512 │    90,000 │   961,512 │ 5,038,488
2     │ 1,051,512 │    75,577 │   975,935 │ 4,062,553
3     │ 1,051,512 │    60,938 │   990,574 │ 3,071,979
4     │ 1,051,512 │    46,080 │ 1,005,432 │ 2,066,547
5     │ 1,051,512 │    30,998 │ 1,020,514 │ 1,046,033
6     │ 1,051,512 │    15,690 │ 1,035,822 │       211 (rounding)

Tổng lãi phải trả: 319,283đ
```

### 4.7 NPL — Non-Performing Loan (Nợ xấu)

```
Phân loại nợ (theo quy định Việt Nam - Thông tư 11/2021):

Nhóm 1 — Nợ đủ tiêu chuẩn:   DPD 0-10 ngày
Nhóm 2 — Nợ cần chú ý:        DPD 11-30 ngày
Nhóm 3 — Nợ dưới tiêu chuẩn:  DPD 31-90 ngày
Nhóm 4 — Nợ nghi ngờ:         DPD 91-180 ngày
Nhóm 5 — Nợ có khả năng mất:  DPD > 180 ngày

NPL = Nợ nhóm 3 + 4 + 5

NPL Ratio = Tổng dư nợ xấu / Tổng dư nợ cho vay

NHNN yêu cầu: NPL Ratio < 3%
Fintech tốt:  NPL Ratio < 5%
Nguy hiểm:    NPL Ratio > 10%
```

**Provisioning (Trích lập dự phòng):**

```
Phải trích lập dự phòng cho từng nhóm nợ:

Nhóm 1: 0%       dự phòng
Nhóm 2: 5%       dự phòng
Nhóm 3: 20%      dự phòng
Nhóm 4: 50%      dự phòng
Nhóm 5: 100%     dự phòng

Ví dụ:
  Dư nợ Nhóm 3: 1,000,000đ → Trích 200,000đ dự phòng
  Dư nợ Nhóm 5: 500,000đ   → Trích 500,000đ dự phòng

Bút toán:
  Debit:  Chi phí dự phòng rủi ro tín dụng
  Credit: Quỹ dự phòng rủi ro tín dụng
```

### 4.8 Data Model — Lending System

```sql
-- Loan Applications
CREATE TABLE loan_applications (
    id                  UUID PRIMARY KEY,
    application_ref     VARCHAR(100) UNIQUE NOT NULL,
    applicant_id        UUID NOT NULL,
    loan_type           VARCHAR(50) NOT NULL,
    -- consumer | sme | bnpl | revolving

    -- Requested terms
    requested_amount    BIGINT NOT NULL,
    requested_tenure    INTEGER NOT NULL,  -- tháng
    purpose             VARCHAR(100),

    -- Decision
    status              VARCHAR(30) NOT NULL DEFAULT 'submitted',
    -- submitted | under_review | approved | counter_offered
    -- rejected | expired | withdrawn
    approved_amount     BIGINT,
    approved_tenure     INTEGER,
    approved_rate       DECIMAL(8,4),  -- lãi suất %/năm
    rejection_reason    VARCHAR(255),

    -- Scoring
    credit_score        INTEGER,         -- 300-850
    risk_grade          VARCHAR(5),      -- A, B, C, D, E
    debt_to_income      DECIMAL(5,4),    -- 0.35 = 35%

    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    decided_at          TIMESTAMPTZ
);

-- Loans (Active loan contracts)
CREATE TABLE loans (
    id                  UUID PRIMARY KEY,
    loan_ref            VARCHAR(100) UNIQUE NOT NULL,
    application_id      UUID REFERENCES loan_applications(id),
    borrower_id         UUID NOT NULL,

    -- Terms
    principal_amount    BIGINT NOT NULL,  -- Số tiền vay
    disbursed_amount    BIGINT NOT NULL,  -- Số tiền giải ngân (sau phí)
    outstanding_balance BIGINT NOT NULL,  -- Dư nợ hiện tại
    interest_rate       DECIMAL(8,4) NOT NULL,  -- %/năm
    tenure_months       INTEGER NOT NULL,
    disbursed_at        TIMESTAMPTZ,
    maturity_date       DATE NOT NULL,    -- Ngày đáo hạn

    -- Status
    status              VARCHAR(30) NOT NULL DEFAULT 'active',
    -- active | completed | defaulted | written_off
    loan_classification INTEGER DEFAULT 1,  -- 1-5 (nhóm nợ)
    dpd                 INTEGER DEFAULT 0,  -- Days Past Due

    -- Collection
    next_due_date       DATE,
    next_due_amount     BIGINT,
    total_overdue       BIGINT DEFAULT 0,

    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Repayment Schedule
CREATE TABLE repayment_schedules (
    id                  UUID PRIMARY KEY,
    loan_id             UUID NOT NULL REFERENCES loans(id),
    installment_no      INTEGER NOT NULL,  -- Kỳ thứ mấy
    due_date            DATE NOT NULL,
    principal_amount    BIGINT NOT NULL,
    interest_amount     BIGINT NOT NULL,
    total_due           BIGINT NOT NULL,   -- principal + interest
    paid_amount         BIGINT DEFAULT 0,
    paid_at             TIMESTAMPTZ,
    status              VARCHAR(20) DEFAULT 'pending',
    -- pending | paid | partial | overdue | waived
    UNIQUE(loan_id, installment_no)
);

-- Loan Transactions
CREATE TABLE loan_transactions (
    id                  UUID PRIMARY KEY,
    loan_id             UUID NOT NULL REFERENCES loans(id),
    transaction_type    VARCHAR(50) NOT NULL,
    -- disbursement | repayment | fee | interest_accrual
    -- late_fee | waiver | write_off
    amount              BIGINT NOT NULL,
    principal_portion   BIGINT DEFAULT 0,
    interest_portion    BIGINT DEFAULT 0,
    fee_portion         BIGINT DEFAULT 0,
    balance_after       BIGINT NOT NULL,
    created_at          TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
```

---

## PHẦN 5: KẾT NỐI CÁC DOMAIN

### 5.1 Wallet + Lending = Super App

```
User dùng ví điện tử hàng ngày:
  → Lịch sử giao dịch → Credit scoring
  → Score tốt → Offer vay vốn ngay trong app
  → Giải ngân vào ví → Dùng ví để trả hàng tháng
  → Repayment history → Tăng credit score
  → Score cao → Hạn mức vay tăng, lãi suất giảm

Vòng lặp giá trị:
  Ví → Data → Credit → Loan → Repayment → Better Credit → Better Loan
```

### 5.2 Merchant + Wallet + Lending = Embedded Finance

```
Merchant dùng Payment terminal của bạn:
  → Dữ liệu doanh thu hàng ngày
  → Sau 3 tháng: "Doanh thu ổn định, offer vay 50tr"
  → Merchant vay → Mua thêm hàng → Doanh thu tăng
  → Doanh thu tăng → Trả nợ từ settlement tự động
  → Win-win: Merchant có vốn, Platform có lãi vay

Đây là mô hình đang được: Grab, Shopee, KiotViet, VNPAY áp dụng
```

### 5.3 Reconciliation kết nối tất cả

```
Reconciliation đảm bảo:

Wallet recon:
  Mọi thay đổi balance đều có ledger entry tương ứng
  SUM(user balances) = Float account

Payment recon:
  Mọi giao dịch trong hệ thống khớp với gateway
  Settlement nhận đúng số tiền kỳ vọng

Lending recon:
  Dư nợ trong hệ thống = Dư nợ thực tế
  Tiền repayment nhận đúng, ghi nhận đúng kỳ
  Trích lập dự phòng đúng theo phân loại nợ
```

---

## PHẦN 6: COMPLIANCE & REGULATION

### 6.1 Khung pháp lý Fintech tại Việt Nam

```
NHNN (Ngân hàng Nhà nước) quản lý:

Giấy phép cần thiết:
  Ví điện tử / TGTT:   Nghị định 101/2012 + TT 23/2019
  Cho vay tiêu dùng:   Thông tư 43/2016 (Công ty tài chính)
  Cho vay P2P:         Đang sandbox, chưa có khung pháp lý chính thức

Giới hạn Ví điện tử:
  Số dư tối đa:     100,000,000đ (cá nhân)
  Tổng GD/tháng:    100,000,000đ (cá nhân chưa xác thực đầy đủ)
  Nạp/rút:          Phải qua tài khoản ngân hàng chính chủ

AML/CFT Requirements:
  Báo cáo giao dịch đáng ngờ (STR) → Cục Phòng chống rửa tiền
  Báo cáo giao dịch tiền mặt lớn (CTR): > 300,000,000đ/ngày
  Lưu trữ hồ sơ KYC: ít nhất 5 năm sau khi kết thúc quan hệ
```

### 6.2 AML (Anti-Money Laundering) trong thực tế

```
Các giao dịch cần flag để review:

STRUCTURING:
  Nhiều giao dịch nhỏ hơn ngưỡng báo cáo
  → 5 giao dịch 60tr thay vì 1 giao dịch 300tr

LAYERING:
  Tiền chuyển qua nhiều tài khoản nhanh chóng
  → A → B → C → D trong vòng 1 giờ

UNUSUAL PATTERN:
  Tài khoản không active đột ngột nhận nhiều tiền
  Chuyển tiền sang nhiều tài khoản khác nhau ngay sau

GEOGRAPHIC RISK:
  Giao dịch từ/đến các quốc gia trong danh sách FATF
```

---

## PHẦN 7: TỔNG KẾT

### Mental Model cho Fintech

```
Fintech = Dùng công nghệ để:
  1. Di chuyển tiền nhanh hơn, rẻ hơn (Payment/Wallet)
  2. Cấp phát vốn hiệu quả hơn (Lending)
  3. Đảm bảo mọi thứ chính xác và tuân thủ (Recon/Compliance)

Ba câu hỏi luôn cần hỏi:
  "Tiền đang ở đâu?" → Wallet/Balance
  "Tiền có đúng không?" → Reconciliation
  "Rủi ro là gì?" → Credit Risk/Compliance
```

### Bảng thuật ngữ tổng hợp

| Thuật ngữ | Ý nghĩa |
|---|---|
| **Double-entry** | Mỗi giao dịch có 2 vế: Debit và Credit |
| **Ledger** | Sổ ghi nhận tất cả entries — immutable |
| **Float Account** | Tài khoản ngân hàng giữ tiền thật của user |
| **Available Balance** | Số dư có thể dùng ngay (trừ reserved) |
| **Race Condition** | Hai request đồng thời gây ra kết quả sai |
| **Idempotency** | Gọi nhiều lần cho cùng kết quả, không duplicate |
| **Reconciliation** | Đối soát số liệu giữa các hệ thống |
| **Variance** | Chênh lệch sau khi đối soát |
| **DPD** | Days Past Due — số ngày quá hạn |
| **NPL** | Non-Performing Loan — nợ xấu |
| **Provisioning** | Trích lập dự phòng rủi ro tín dụng |
| **Amortization** | Lịch trả nợ chi tiết gốc + lãi từng kỳ |
| **Credit Score** | Điểm tín dụng đánh giá khả năng trả nợ |
| **DTI** | Debt-to-Income ratio — tỷ lệ nợ/thu nhập |
| **LTV** | Loan-to-Value — tỷ lệ vay/tài sản đảm bảo |
| **BNPL** | Buy Now Pay Later — mua trước trả sau |
| **NIM** | Net Interest Margin — biên lãi suất ròng |
| **AML** | Anti-Money Laundering — phòng chống rửa tiền |
| **KYC/eKYC** | Xác minh danh tính (điện tử) |
| **STR** | Suspicious Transaction Report — báo cáo giao dịch đáng ngờ |
| **Write-off** | Xóa nợ khỏi sổ sách khi không thể thu hồi |
| **Embedded Finance** | Tích hợp dịch vụ tài chính vào sản phẩm phi tài chính |