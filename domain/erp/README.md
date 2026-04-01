# 🏢 ERP Domain — Business Analyst Deep Dive

> Tài liệu này giải thích toàn bộ domain **ERP (Enterprise Resource Planning)** từ góc độ Business Analyst: hiểu nghiệp vụ từng module, mối liên hệ giữa các phòng ban, và tư duy phân tích khi build ERP nội bộ.

---

## 1. ERP là gì — và tại sao nó tồn tại?

### Vấn đề trước khi có ERP

Hãy tưởng tượng một công ty không có ERP:

```
Phòng Kế toán     → Dùng Excel riêng
Phòng Kho         → Dùng phần mềm riêng (hoặc sổ tay)
Phòng Nhân sự     → Dùng Excel riêng
Phòng Kinh doanh  → Dùng CRM riêng
Phòng Mua hàng    → Email + Excel
```

**Hậu quả:**

- Kế toán hỏi "tháng này mua hàng bao nhiêu?" → phải chờ phòng mua hàng gửi file
- Kho báo "còn 100 sản phẩm" nhưng kế toán ghi "đã xuất 120" → số liệu mâu thuẫn
- Giám đốc muốn xem báo cáo tổng hợp → phải gom dữ liệu từ 5 nơi, mất 2 ngày
- Nhân viên nghỉ việc → không ai biết công việc của họ đang ở đâu

👉 **ERP ra đời để giải quyết đúng vấn đề này**: một hệ thống duy nhất, dữ liệu dùng chung, mọi phòng ban kết nối với nhau.

### Định nghĩa thực tế

> **ERP** là hệ thống phần mềm tích hợp giúp doanh nghiệp **quản lý và kết nối toàn bộ hoạt động** — từ tài chính, kho vận, nhân sự, đến bán hàng — trên một nền tảng dữ liệu chung.

Từ khóa quan trọng nhất: **"dữ liệu chung" (single source of truth)**

---

## 2. Bức tranh tổng thể — ERP gồm những gì?

### Các module cốt lõi

```
┌─────────────────────────────────────────────────────────────┐
│                        ERP SYSTEM                           │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐      │
│  │   FINANCE    │  │  INVENTORY   │  │      HR      │      │
│  │ & ACCOUNTING │  │ & WAREHOUSE  │  │  & PAYROLL   │      │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘      │
│         │                 │                  │              │
│         └─────────────────┼──────────────────┘              │
│                    ┌──────┴───────┐                         │
│                    │  SHARED DB   │  ← Single Source        │
│                    │  (dữ liệu    │     of Truth            │
│                    │   dùng chung)│                         │
│                    └──────┬───────┘                         │
│         ┌─────────────────┼──────────────────┐              │
│  ┌──────┴───────┐  ┌──────┴───────┐  ┌───────┴──────┐      │
│  │    SALES     │  │  PURCHASING  │  │  PRODUCTION  │      │
│  │   & CRM      │  │  & PROCUREMENT│  │  (MRP/MES)  │      │
│  └──────────────┘  └──────────────┘  └──────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### Cách các module kết nối với nhau — ví dụ thực tế

```
Khách hàng đặt hàng (Sales)
        │
        ▼
Kiểm tra tồn kho (Inventory)
        │
   ┌────┴────┐
Đủ hàng   Không đủ
   │           │
   ▼           ▼
Xuất kho    Tạo PO mua thêm (Purchasing)
   │           │
   ▼           ▼
Ghi nhận    Nhập kho khi hàng về
doanh thu       │
(Finance)   Thanh toán NCC (Finance)
```

Đây chính là sức mạnh của ERP: **một hành động ở một module tự động ảnh hưởng đến các module khác**, không cần nhập tay nhiều lần.

---

## 3. Finance & Accounting — Kế toán & Tài chính

Đây là **module trung tâm** của mọi ERP. Mọi giao dịch trong hệ thống đều cuối cùng phải phản ánh vào kế toán.

### 3.1 Tại sao Finance là module quan trọng nhất?

Vì **tiền là ngôn ngữ chung** của doanh nghiệp. Dù là nhập kho, bán hàng, hay trả lương — tất cả đều có giá trị tiền tệ và phải được ghi nhận đúng.

Sai ở Finance = báo cáo tài chính sai = quyết định kinh doanh sai.

### 3.2 Double-Entry Bookkeeping — Nền tảng của kế toán

Đây là khái niệm quan trọng nhất trong kế toán, có từ thế kỷ 15 và vẫn là nền tảng của mọi phần mềm kế toán hiện đại.

**Nguyên tắc:** Mỗi giao dịch luôn có **hai vế** — Nợ (Debit) và Có (Credit) — và tổng Nợ phải luôn bằng tổng Có.

```
Ví dụ: Công ty bán hàng thu tiền mặt 10,000,000đ

Nợ (Debit)                    Có (Credit)
─────────────────────────────────────────
Tiền mặt    +10,000,000đ  │  Doanh thu    +10,000,000đ
                          │
→ Tài sản tăng            │  → Thu nhập tăng
```

```
Ví dụ: Công ty mua hàng hóa về kho, chưa trả tiền, 5,000,000đ

Nợ (Debit)                    Có (Credit)
─────────────────────────────────────────
Hàng tồn kho +5,000,000đ  │  Phải trả NCC +5,000,000đ
                          │
→ Tài sản tăng            │  → Nợ phải trả tăng
```

**Tại sao BA cần hiểu điều này?**

Khi bạn thiết kế luồng nghiệp vụ, mỗi hành động người dùng thực hiện sẽ tạo ra **journal entry** (bút toán) tương ứng. Nếu không hiểu double-entry, bạn sẽ thiết kế sai luồng và kế toán sẽ không dùng được.

### 3.3 Chart of Accounts (CoA) — Hệ thống tài khoản kế toán

**CoA** là danh sách tất cả các tài khoản kế toán mà doanh nghiệp dùng để phân loại giao dịch.

```
LOẠI 1: TÀI SẢN (Assets)
  111 - Tiền mặt
  112 - Tiền gửi ngân hàng
  131 - Phải thu khách hàng
  152 - Nguyên vật liệu
  156 - Hàng hóa
  211 - Tài sản cố định hữu hình

LOẠI 2: NGUỒN VỐN (Equity)
  411 - Vốn đầu tư của chủ sở hữu
  421 - Lợi nhuận sau thuế chưa phân phối

LOẠI 3: NỢ PHẢI TRẢ (Liabilities)
  331 - Phải trả người bán
  333 - Thuế và các khoản phải nộp Nhà nước
  334 - Phải trả người lao động

LOẠI 4: DOANH THU (Revenue)
  511 - Doanh thu bán hàng và cung cấp dịch vụ
  515 - Doanh thu hoạt động tài chính

LOẠI 5: CHI PHÍ (Expenses)
  632 - Giá vốn hàng bán
  641 - Chi phí bán hàng
  642 - Chi phí quản lý doanh nghiệp
```

**Lưu ý khi build ERP ở Việt Nam:** CoA phải tuân theo **Thông tư 200/2014/TT-BTC** (hoặc TT 133 cho SME) của Bộ Tài chính.

### 3.4 General Ledger (Sổ Cái) — Trái tim của kế toán

**General Ledger (GL)** là nơi ghi lại **tất cả bút toán** của doanh nghiệp theo thứ tự thời gian. Mọi giao dịch trong ERP đều cuối cùng tạo ra một dòng trong GL.

```
Sổ Cái — Tài khoản 111 (Tiền mặt)
────────────────────────────────────────────────────────────────
Ngày       │ Mô tả                  │ Nợ (Dr)  │ Có (Cr)  │ Số dư
────────────────────────────────────────────────────────────────
01/12/2024 │ Số dư đầu kỳ          │          │          │ 50,000,000
03/12/2024 │ Thu tiền bán hàng HD1 │ 10,000,000│         │ 60,000,000
05/12/2024 │ Chi phí vận chuyển    │          │ 500,000  │ 59,500,000
07/12/2024 │ Thu tiền bán hàng HD2 │ 15,000,000│         │ 74,500,000
────────────────────────────────────────────────────────────────
```

### 3.5 Các báo cáo tài chính quan trọng

ERP phải tự động tổng hợp được 3 báo cáo này từ dữ liệu GL:

#### Báo cáo Kết quả Kinh doanh (P&L / Income Statement)

```
DOANH THU
  Doanh thu thuần           500,000,000đ
  Giá vốn hàng bán         (300,000,000đ)
─────────────────────────────────────────
  Lợi nhuận gộp             200,000,000đ

CHI PHÍ HOẠT ĐỘNG
  Chi phí bán hàng          (50,000,000đ)
  Chi phí quản lý           (30,000,000đ)
─────────────────────────────────────────
  Lợi nhuận trước thuế      120,000,000đ
  Thuế TNDN (20%)           (24,000,000đ)
─────────────────────────────────────────
  Lợi nhuận sau thuế         96,000,000đ
```

#### Bảng Cân đối Kế toán (Balance Sheet)

```
TÀI SẢN                     NỢ PHẢI TRẢ & VỐN CHỦ SỞ HỮU
────────────────────────    ────────────────────────────────
Tài sản ngắn hạn:           Nợ ngắn hạn:
  Tiền mặt       200tr        Phải trả NCC      100tr
  Phải thu KH    150tr        Thuế phải nộp      20tr
  Hàng tồn kho   100tr
                             Nợ dài hạn:
Tài sản dài hạn:               Vay dài hạn       200tr
  TSCĐ           300tr
                             Vốn chủ sở hữu:
                               Vốn điều lệ       330tr
                               Lợi nhuận GN      100tr
────────────────────────    ────────────────────────────────
TỔNG TÀI SẢN    750tr       TỔNG NỢ + VỐN       750tr
```

#### Báo cáo Lưu chuyển Tiền tệ (Cash Flow Statement)

Theo dõi **tiền thực sự ra vào** (không phải doanh thu ghi nhận theo accrual):
- Tiền từ hoạt động kinh doanh
- Tiền từ hoạt động đầu tư
- Tiền từ hoạt động tài chính

### 3.6 Accounts Payable (AP) — Phải trả nhà cung cấp

AP quản lý **tiền công ty nợ nhà cung cấp**.

```
Luồng AP điển hình:

[1] Mua hàng → Tạo Purchase Order (PO)
[2] Nhận hàng → Tạo Goods Receipt (GR)
[3] Nhận hóa đơn NCC → 3-way matching:
    PO ↔ GR ↔ Invoice (phải khớp nhau)
[4] Approve payment
[5] Thanh toán → Ghi nhận bút toán

Bút toán khi nhận hàng:
  Nợ: Hàng tồn kho  +10,000,000
  Có: Phải trả NCC  +10,000,000

Bút toán khi thanh toán:
  Nợ: Phải trả NCC  +10,000,000
  Có: Tiền gửi NH   +10,000,000
```

**3-way matching** là khái niệm quan trọng: trước khi thanh toán, hệ thống tự động đối chiếu 3 chứng từ phải khớp nhau để tránh thanh toán sai hoặc gian lận.

### 3.7 Accounts Receivable (AR) — Phải thu khách hàng

AR quản lý **tiền khách hàng nợ công ty**.

```
Luồng AR điển hình:

[1] Bán hàng → Tạo hóa đơn (Invoice)
[2] Giao hàng → Ghi nhận doanh thu
[3] Theo dõi công nợ: Invoice chưa được thanh toán
[4] Nhắc nợ khi quá hạn (Dunning)
[5] Thu tiền → Đối chiếu với Invoice
[6] Xử lý nợ khó đòi (bad debt)

Aging Report (Báo cáo tuổi nợ):
┌──────────────┬────────┬────────┬────────┬────────┐
│ Khách hàng   │ Chưa   │ 1-30   │ 31-60  │ > 60   │
│              │ đến hạn│  ngày  │  ngày  │  ngày  │
├──────────────┼────────┼────────┼────────┼────────┤
│ Công ty A    │ 50tr   │ 20tr   │        │        │
│ Công ty B    │        │        │ 30tr   │ 10tr   │
│ Công ty C    │ 100tr  │        │        │        │
└──────────────┴────────┴────────┴────────┴────────┘
```

**Aging Report** là báo cáo quan trọng nhất của AR — cho biết bao nhiêu tiền đang bị nợ và đã nợ bao lâu.

### 3.8 Cost Center & Profit Center

Trong doanh nghiệp lớn, không chỉ cần biết tổng lãi/lỗ — mà cần biết **phòng ban nào, dự án nào, chi nhánh nào** đang lãi/lỗ.

- **Cost Center**: Đơn vị chỉ phát sinh chi phí (phòng IT, phòng hành chính)
- **Profit Center**: Đơn vị có cả doanh thu và chi phí (chi nhánh Hà Nội, sản phẩm A)

```
Doanh thu tháng 12: 1,000,000,000đ
├── Chi nhánh Hà Nội:   400,000,000đ (Profit Center)
├── Chi nhánh HCM:      500,000,000đ (Profit Center)
└── Xuất khẩu:          100,000,000đ (Profit Center)

Chi phí IT:             50,000,000đ (Cost Center — phân bổ cho các PC)
```

---

## 4. Inventory & Warehouse — Kho Vận & Tồn Kho

### 4.1 Tại sao Inventory quan trọng?

Hàng tồn kho là **tài sản** của doanh nghiệp. Quản lý sai tồn kho dẫn đến:
- **Thiếu hàng** → mất đơn hàng, khách hàng không hài lòng
- **Thừa hàng** → đọng vốn, hàng hết hạn, chi phí lưu kho cao
- **Số liệu sai** → báo cáo tài chính sai (vì hàng tồn kho là tài sản)

### 4.2 Các luồng nhập/xuất kho cơ bản

```
NHẬP KHO (Stock In):
  ✓ Mua hàng từ NCC (Purchase Receipt)
  ✓ Sản xuất xong, nhập thành phẩm
  ✓ Khách hàng trả hàng (Return)
  ✓ Điều chuyển từ kho khác

XUẤT KHO (Stock Out):
  ✓ Bán hàng cho khách (Sales Delivery)
  ✓ Xuất để sản xuất (Material Issue)
  ✓ Trả lại nhà cung cấp (Return to Vendor)
  ✓ Điều chuyển đến kho khác
  ✓ Xuất hủy (Scrap/Write-off)

ĐIỀU CHỈNH (Adjustment):
  ✓ Kiểm kê phát hiện lệch số liệu
  ✓ Hàng bị hỏng/mất
```

### 4.3 Phương pháp định giá tồn kho

Đây là phần kế toán + kho giao nhau — rất quan trọng vì ảnh hưởng đến **Giá vốn hàng bán (COGS)** và **lợi nhuận**.

#### FIFO (First In, First Out)
Hàng nhập trước → xuất trước. Giá vốn tính theo lô nhập sớm nhất.

```
Nhập lô 1: 100 cái × 10,000đ = 1,000,000đ  (01/12)
Nhập lô 2: 100 cái × 12,000đ = 1,200,000đ  (15/12)

Bán 50 cái ngày 20/12:
→ FIFO: Giá vốn = 50 × 10,000đ = 500,000đ  (lấy từ lô 1)
```

#### LIFO (Last In, First Out)
Hàng nhập sau → xuất trước. (Ít dùng ở VN, không được phép theo VAS)

#### Weighted Average (Bình quân gia quyền)
Tính giá trung bình của tất cả lô hàng hiện có.

```
Tồn kho hiện tại: 200 cái
  Lô 1: 100 cái × 10,000đ
  Lô 2: 100 cái × 12,000đ
  Giá bình quân = (1,000,000 + 1,200,000) / 200 = 11,000đ/cái

Bán 50 cái:
→ Giá vốn = 50 × 11,000đ = 550,000đ
```

**Tại Việt Nam:** Thông tư 200 cho phép dùng FIFO hoặc Bình quân gia quyền. Phải nhất quán trong năm tài chính.

### 4.4 Warehouse Management — Quản lý kho vật lý

Không chỉ biết "còn bao nhiêu hàng" mà còn biết **hàng đang ở đâu trong kho**.

```
KHO A (Hà Nội)
├── Khu vực 1 (Nguyên liệu)
│   ├── Kệ A1-01: Vải cotton — 500kg
│   ├── Kệ A1-02: Vải polyester — 300kg
│   └── ...
├── Khu vực 2 (Thành phẩm)
│   ├── Kệ B2-01: Áo sơ mi trắng size M — 200 cái
│   ├── Kệ B2-02: Áo sơ mi trắng size L — 150 cái
│   └── ...
└── Khu vực 3 (Hàng chờ xuất)
    └── ...
```

**Các khái niệm quan trọng trong WMS:**

| Khái niệm | Ý nghĩa |
|---|---|
| **SKU** (Stock Keeping Unit) | Mã định danh duy nhất cho từng sản phẩm/biến thể |
| **Lot/Batch** | Lô sản xuất — quan trọng với thực phẩm, dược phẩm (truy xuất nguồn gốc) |
| **Serial Number** | Số serial riêng từng sản phẩm — với hàng điện tử, máy móc |
| **Bin/Location** | Vị trí cụ thể trong kho (kệ, ô, tầng) |
| **Pick List** | Danh sách hàng cần lấy để đóng gói một đơn hàng |
| **Put Away** | Quy trình cất hàng mới nhập vào đúng vị trí |
| **Cycle Count** | Kiểm kê định kỳ một phần kho (không phải toàn bộ) |
| **Physical Inventory** | Kiểm kê toàn bộ kho (thường cuối năm) |

### 4.5 Reorder Point — Tự động nhắc đặt hàng

```
Reorder Point = (Mức tiêu thụ trung bình/ngày) × (Lead time ngày)
              + Safety Stock

Ví dụ:
  Tiêu thụ: 50 cái/ngày
  Lead time (từ khi đặt đến khi nhận hàng): 7 ngày
  Safety Stock: 100 cái (dự phòng)

  Reorder Point = 50 × 7 + 100 = 450 cái

→ Khi tồn kho ≤ 450 cái → hệ thống tự động tạo đề xuất mua hàng
```

### 4.6 Tồn kho và Kế toán — Liên kết quan trọng

Mỗi lần nhập/xuất kho đều tạo ra bút toán kế toán tương ứng:

```
NHẬP KHO (mua hàng):
  Nợ: Hàng tồn kho (TK 156)   +10,000,000
  Có: Phải trả NCC (TK 331)   +10,000,000

XUẤT KHO (bán hàng):
  Nợ: Giá vốn hàng bán (TK 632)  +8,000,000
  Có: Hàng tồn kho (TK 156)      +8,000,000

  Nợ: Phải thu KH (TK 131)      +12,000,000
  Có: Doanh thu (TK 511)         +12,000,000
```

---

## 5. HR & Payroll — Nhân sự & Lương

### 5.1 Phạm vi của HR trong ERP

HR module không chỉ là "quản lý hồ sơ nhân viên". Nó bao gồm toàn bộ **vòng đời nhân viên** từ khi ứng tuyển đến khi nghỉ việc.

```
RECRUIT       ONBOARD      MANAGE        DEVELOP        OFFBOARD
────────────────────────────────────────────────────────────────
Đăng tin  → Ký HĐ      → Chấm công  → Đào tạo     → Nghỉ việc
Sơ tuyển  → Cấp tài    → Tính lương → Đánh giá    → Thanh toán
Phỏng vấn   khoản       Nghỉ phép    KPI            cuối kỳ
Offer                   Kỷ luật    Thăng chức     Bàn giao
```

### 5.2 Employee Master Data — Hồ sơ nhân viên

Đây là trung tâm của HR module. Mọi thứ liên quan đến nhân viên đều tham chiếu về đây.

```
THÔNG TIN CÁ NHÂN:
  Họ tên, ngày sinh, giới tính
  CCCD/CMND, mã số thuế cá nhân
  Địa chỉ, liên hệ khẩn cấp
  Trình độ học vấn, chuyên ngành

THÔNG TIN CÔNG VIỆC:
  Mã nhân viên (Employee ID)
  Phòng ban (Department)
  Chức danh (Job Title / Position)
  Cấp bậc (Grade/Level)
  Người quản lý trực tiếp
  Ngày vào làm, loại hợp đồng
  Địa điểm làm việc

THÔNG TIN LƯƠNG & PHÚC LỢI:
  Mức lương cơ bản
  Phụ cấp (nhà ở, đi lại, ăn trưa...)
  Tài khoản ngân hàng nhận lương
  Đăng ký bảo hiểm

LỊCH SỬ:
  Lịch sử thăng chức/điều chuyển
  Lịch sử thay đổi lương
  Lịch sử hợp đồng
```

### 5.3 Timekeeping — Chấm công

Chấm công là **đầu vào quan trọng nhất** để tính lương đúng.

**Các phương thức chấm công:**

| Phương thức | Ưu điểm | Nhược điểm |
|---|---|---|
| Vân tay / Face ID | Chính xác, không thể chấm hộ | Tốn thiết bị |
| Thẻ từ | Rẻ, phổ biến | Có thể chấm hộ |
| App mobile (GPS) | Phù hợp remote/field | Phụ thuộc điện thoại |
| Excel/Manual | Đơn giản | Dễ sai sót, gian lận |

**Các loại giờ làm việc cần theo dõi:**

```
Giờ làm việc tiêu chuẩn:  8 giờ/ngày, 40 giờ/tuần
Giờ làm thêm (OT):
  - OT ngày thường:    × 1.5
  - OT ngày nghỉ:      × 2.0
  - OT ngày lễ:        × 3.0

Nghỉ phép:
  - Phép năm (Annual Leave):   12 ngày/năm (theo luật VN)
  - Phép ốm (Sick Leave)
  - Phép thai sản (Maternity): 6 tháng
  - Phép không lương
  - Nghỉ lễ tết: ~11 ngày/năm
```

### 5.4 Payroll — Tính lương

Đây là nghiệp vụ phức tạp nhất trong HR, liên quan đến nhiều quy định pháp luật.

**Công thức lương thực nhận (Take-home Pay):**

```
LƯƠNG GROSS (Tổng thu nhập):
  Lương cơ bản                    15,000,000
  Phụ cấp (nhà ở, đi lại, ăn)     3,000,000
  Thưởng KPI tháng                 2,000,000
  Lương OT                         1,500,000
─────────────────────────────────────────────
  TỔNG GROSS:                     21,500,000

KHẤU TRỪ BẢO HIỂM (Nhân viên đóng):
  BHXH 8%:          21,500,000 × 8%  =  1,720,000  (*)
  BHYT 1.5%:        21,500,000 × 1.5% =   322,500
  BHTN 1%:          21,500,000 × 1%   =   215,000
─────────────────────────────────────────────
  Tổng BH nhân viên:                   2,257,500

THU NHẬP CHỊU THUẾ:
  Gross - BH nhân viên - Giảm trừ bản thân (11tr) - Giảm trừ phụ thuộc
  = 21,500,000 - 2,257,500 - 11,000,000 = 8,242,500

THUẾ TNCN (theo biểu lũy tiến):
  5 triệu đầu × 5%  =  250,000
  3,242,500 × 10%   =  324,250
  Tổng thuế TNCN:      574,250

─────────────────────────────────────────────
LƯƠNG NET = 21,500,000 - 2,257,500 - 574,250 = 18,668,250đ
```

(*) Mức đóng BHXH tính trên mức lương BH (thường = lương cơ bản + phụ cấp cố định, tối đa 20 × lương cơ sở)

**Chi phí thực tế của công ty (Cost to Company):**

```
Lương Gross nhân viên:    21,500,000
BHXH công ty đóng 17%:     3,655,000
BHYT công ty đóng 3%:        645,000
BHTN công ty đóng 1%:        215,000
Kinh phí Công đoàn 2%:       430,000
─────────────────────────────────────
TỔNG CHI PHÍ CÔNG TY:     26,445,000
```

### 5.5 Payroll Cycle — Chu kỳ tính lương

```
[1] Đóng dữ liệu chấm công (cuối tháng)
         │
[2] Nhập các thay đổi trong tháng:
    - Nhân viên mới, nghỉ việc
    - Thay đổi lương
    - Thưởng, phạt
    - Nghỉ phép không lương
         │
[3] Chạy tính lương (Payroll Run)
    - Tính gross, khấu trừ, net
    - Tính BH, thuế TNCN
         │
[4] Review & Approve (Kế toán, HR Manager)
         │
[5] Chuyển khoản lương (Payroll Transfer)
    - Kết nối ngân hàng, chuyển khoản hàng loạt
         │
[6] Phát phiếu lương (Payslip)
         │
[7] Ghi nhận bút toán kế toán:
    Nợ: Chi phí lương (TK 334)
    Có: Tiền gửi NH / Phải trả NLĐ
         │
[8] Nộp báo cáo BHXH, Quyết toán thuế TNCN
```

### 5.6 Kết nối HR với các module khác

```
HR → Finance:
  Chi phí lương → General Ledger
  Phân bổ chi phí theo Cost Center/Department

HR → Project Management (nếu có):
  Timesheet nhân viên → Chi phí dự án

HR → Inventory (với công ty sản xuất):
  Giờ công sản xuất → Giá thành sản phẩm
```

---

## 6. Sales & CRM — Bán hàng & Khách hàng

### 6.1 CRM vs Sales trong ERP — Khác nhau thế nào?

```
CRM (Customer Relationship Management):
  → Quản lý quan hệ khách hàng
  → Theo dõi leads, cơ hội, hoạt động sales
  → Lịch sử tương tác với khách hàng
  → Dự báo doanh thu (Sales Forecast)

Sales Order Management:
  → Xử lý đơn hàng đã chốt
  → Giao hàng, xuất hóa đơn
  → Công nợ khách hàng
```

CRM là **trước khi bán được hàng**. Sales Order là **sau khi khách hàng đồng ý mua**.

### 6.2 Sales Pipeline — Phễu bán hàng

```
LEAD (Khách hàng tiềm năng)
  → Ai đó quan tâm đến sản phẩm
  → Nguồn: website, sự kiện, referral, cold call

QUALIFIED LEAD (Lead đã đánh giá)
  → Xác nhận có nhu cầu thực sự, có ngân sách, có quyền quyết định

OPPORTUNITY (Cơ hội)
  → Đang thương lượng, gửi báo giá
  → Có % win probability

PROPOSAL/QUOTE (Báo giá)
  → Gửi báo giá chính thức

NEGOTIATION (Đàm phán)
  → Thương lượng giá, điều khoản

CLOSED WON ✓ → Tạo Sales Order
CLOSED LOST ✗ → Ghi lý do thua, học bài
```

**Win Rate** = Closed Won / Tổng Opportunities → KPI quan trọng của đội sales

### 6.3 Sales Order — Đơn hàng bán

Sau khi khách hàng đồng ý, tạo **Sales Order (SO)** — đây là văn bản cam kết bán hàng.

```
SALES ORDER #SO-2024-001234
────────────────────────────────────────────────
Khách hàng:    Công ty ABC
Ngày đặt:      01/12/2024
Ngày giao dự kiến: 05/12/2024
Điều khoản TT: Net 30 (thanh toán trong 30 ngày)

SẢN PHẨM:
┌─────────────┬──────┬──────────┬─────────────┐
│ SKU         │ SL   │ Đơn giá  │ Thành tiền  │
├─────────────┼──────┼──────────┼─────────────┤
│ SHIRT-M-WHT │  50  │ 200,000  │ 10,000,000  │
│ SHIRT-L-BLK │  30  │ 220,000  │  6,600,000  │
└─────────────┴──────┴──────────┴─────────────┘
Tổng trước VAT:      16,600,000
VAT 10%:              1,660,000
TỔNG CỘNG:           18,260,000
```

**Sau khi tạo SO, các bước tiếp theo:**

```
SO tạo ra
    │
    ├──► Inventory: Kiểm tra tồn kho → Reserve hàng
    │
    ├──► Warehouse: Tạo Pick List → Đóng gói → Xuất kho
    │
    ├──► Finance: Tạo hóa đơn (Invoice) khi xuất kho
    │
    └──► AR: Theo dõi công nợ → Nhắc thanh toán khi đến hạn
```

### 6.4 Pricing — Quản lý giá bán

Một hệ thống ERP tốt cần hỗ trợ nhiều loại giá:

```
PRICE LIST (Bảng giá):
  Giá lẻ (Retail Price):         250,000đ/cái
  Giá sỉ (Wholesale Price):      200,000đ/cái
  Giá đại lý (Dealer Price):     180,000đ/cái
  Giá xuất khẩu (Export Price):  $8/cái

DISCOUNT (Chiết khấu):
  Chiết khấu theo sản lượng:
    > 100 cái: -5%
    > 500 cái: -10%
    > 1000 cái: -15%
  
  Chiết khấu theo khách hàng VIP:
    Hạng Gold: -8%
    Hạng Platinum: -12%

  Chiết khấu theo thời gian:
    Thanh toán trong 7 ngày: -2% (Early Payment Discount)
```

### 6.5 Customer Master — Hồ sơ khách hàng

```
THÔNG TIN ĐỊNH DANH:
  Mã khách hàng (Customer Code)
  Tên, MST, địa chỉ, liên hệ
  Loại khách hàng (B2B / B2C)

THÔNG TIN THƯƠNG MẠI:
  Điều khoản thanh toán (Payment Terms): Net 15, Net 30, COD...
  Hạn mức tín dụng (Credit Limit): 100,000,000đ
  Dư nợ hiện tại (Outstanding Balance)
  Lịch sử mua hàng

  → Nếu dư nợ > Credit Limit: Cảnh báo khi tạo SO mới

PHÂN LOẠI KHÁCH HÀNG:
  Theo doanh thu: A (>100tr/tháng), B (10-100tr), C (<10tr)
  Theo ngành: Retail, Wholesale, Export
  Theo vùng: HCM, HN, Miền Trung, Miền Nam
```

---

## 7. Purchasing & Procurement — Mua hàng

### 7.1 Purchasing vs Procurement

| | Purchasing | Procurement |
|---|---|---|
| Phạm vi | Giao dịch mua cụ thể | Toàn bộ quy trình từ nhu cầu → thanh toán |
| Bao gồm | Đặt hàng, nhận hàng | Lập kế hoạch, tìm NCC, đàm phán, mua, thanh toán |

ERP thường gọi chung là **Procurement** vì bao gồm toàn bộ.

### 7.2 Procure-to-Pay (P2P) — Toàn bộ luồng mua hàng

```
[1] DEMAND (Nhu cầu)
    Purchase Requisition (PR) — Phiếu đề nghị mua hàng
    "Phòng Kho cần mua thêm 1000 cái áo size M"
    → Người lập: Nhân viên kho / bộ phận phát sinh nhu cầu
    → Phê duyệt: Trưởng bộ phận, sau đó Giám đốc (nếu vượt hạn mức)

[2] SOURCING (Tìm nhà cung cấp)
    Request for Quotation (RFQ) — Yêu cầu báo giá
    → Gửi yêu cầu báo giá đến nhiều NCC
    → So sánh giá, chất lượng, thời gian giao hàng
    → Chọn NCC phù hợp

[3] ORDERING (Đặt hàng)
    Purchase Order (PO) — Đơn đặt hàng
    → Văn bản chính thức gửi NCC
    → Sau khi NCC xác nhận: hợp đồng ràng buộc

[4] RECEIVING (Nhận hàng)
    Goods Receipt (GR) — Phiếu nhập kho
    → Đối chiếu hàng nhận với PO
    → Nhập kho, tạo bút toán kế toán

[5] INVOICE PROCESSING (Xử lý hóa đơn)
    → Nhận hóa đơn từ NCC
    → 3-way matching: PO ↔ GR ↔ Invoice
    → Phê duyệt thanh toán

[6] PAYMENT (Thanh toán)
    → Chuyển khoản cho NCC
    → Ghi nhận bút toán: Tất toán công nợ phải trả
```

### 7.3 Vendor Master — Hồ sơ nhà cung cấp

Tương tự Customer Master nhưng cho NCC:

```
Thông tin pháp lý: Tên, MST, địa chỉ
Điều khoản thanh toán: Net 30, trả trước 30%...
Thông tin ngân hàng: để chuyển tiền
Danh mục hàng cung cấp: NCC này bán gì
Lịch sử mua hàng: đã mua bao nhiêu, chất lượng thế nào
Đánh giá NCC (Vendor Rating): chất lượng, thời gian giao hàng, giá cả
```

---

## 8. Mối liên kết giữa các Module — Điều quan trọng nhất

Hiểu từng module chưa đủ — quan trọng hơn là hiểu **chúng kết nối với nhau như thế nào**.

### 8.1 Ma trận tương tác giữa các module

```
         Finance  Inventory  HR  Sales  Purchasing
Finance    ──       ✓✓✓      ✓✓   ✓✓       ✓✓
Inventory  ✓✓✓      ──       ─    ✓✓✓      ✓✓✓
HR         ✓✓       ─        ──   ─         ─
Sales      ✓✓       ✓✓✓      ─    ──        ─
Purchasing ✓✓       ✓✓✓      ─    ─         ──

✓✓✓ = Kết nối chặt chẽ, thường xuyên
✓✓  = Kết nối thường xuyên
─   = Ít hoặc không kết nối trực tiếp
```

### 8.2 Kịch bản tổng hợp — Một đơn hàng bán đi qua bao nhiêu module?

```
Khách hàng đặt mua 100 cái áo, trả sau 30 ngày

[CRM]
  → Lead chuyển thành Opportunity
  → Gửi báo giá → Khách hàng đồng ý

[SALES]
  → Tạo Sales Order #SO-001
  → Kiểm tra Credit Limit của khách hàng (OK)

[INVENTORY]
  → Kiểm tra tồn kho: Còn 150 cái → Đủ
  → Reserve 100 cái cho SO-001
  → Tạo Delivery Order (DO)

[WAREHOUSE]
  → Tạo Pick List
  → Nhân viên kho lấy hàng → Đóng gói
  → Xuất kho 100 cái → Ghi nhận GI (Goods Issue)

[INVENTORY - Cập nhật]
  → Tồn kho: 150 → 50 cái

[FINANCE - AR]
  → Tạo Invoice #INV-001: 18,260,000đ
  → Bút toán:
    Nợ: Phải thu KH (131)     +18,260,000
    Có: Doanh thu (511)        +16,600,000
    Có: Thuế VAT phải nộp (333) +1,660,000
  → Bút toán Giá vốn:
    Nợ: Giá vốn HB (632)      +12,000,000
    Có: Hàng tồn kho (156)    +12,000,000

[30 NGÀY SAU - FINANCE - AR]
  → Khách hàng chuyển khoản 18,260,000đ
  → Bút toán:
    Nợ: Tiền gửi NH (112)     +18,260,000
    Có: Phải thu KH (131)     +18,260,000
  → Invoice đánh dấu PAID
```

### 8.3 Kịch bản mua hàng — Tự động tạo PO khi thiếu hàng

```
[INVENTORY]
  → Tồn kho áo size M = 45 cái
  → Reorder Point = 50 cái
  → Trigger: Tồn kho dưới ngưỡng!

[PURCHASING - Auto]
  → Hệ thống tạo tự động Purchase Requisition
  → "Đề nghị mua 200 cái áo size M"
  → Gửi phê duyệt

[PURCHASING - Sau khi approve]
  → Tạo PO gửi NCC
  → NCC giao hàng sau 5 ngày

[INVENTORY]
  → Nhập kho 200 cái
  → Tồn kho: 45 + 200 = 245 cái

[FINANCE - AP]
  → Nhận hóa đơn NCC
  → 3-way match OK
  → Thanh toán → Ghi nhận bút toán
```

---

## 9. Reporting & Analytics — Báo cáo trong ERP

### 9.1 Các loại báo cáo quan trọng theo module

**Finance:**
- Bảng cân đối kế toán (Balance Sheet)
- Báo cáo KQKD (P&L)
- Lưu chuyển tiền tệ (Cash Flow)
- Aging Report (tuổi nợ phải thu / phải trả)
- Trial Balance (Bảng cân đối số phát sinh)

**Inventory:**
- Báo cáo tồn kho tại một thời điểm
- Báo cáo nhập/xuất/tồn theo kỳ
- Báo cáo hàng chậm luân chuyển (Slow-moving)
- Báo cáo hàng sắp hết hạn (Near-expiry)
- Kết quả kiểm kê (Variance Report)

**HR:**
- Bảng lương tổng hợp theo tháng
- Báo cáo BHXH nộp hàng tháng
- Quyết toán thuế TNCN năm
- Báo cáo turnover (tỷ lệ nghỉ việc)
- Headcount theo phòng ban

**Sales:**
- Doanh thu theo nhân viên sales, theo khu vực, theo sản phẩm
- Tỷ lệ chuyển đổi Pipeline
- Top khách hàng
- So sánh Actual vs Target

### 9.2 Dashboard Executive — Giám đốc cần nhìn gì?

```
┌─────────────────────────────────────────────────┐
│             EXECUTIVE DASHBOARD                  │
│                    Tháng 12/2024                 │
├──────────────┬──────────────┬───────────────────┤
│ Doanh thu    │ Lợi nhuận    │ Tồn kho           │
│ 2.1 tỷ      │ 320 triệu    │ 1.8 tỷ            │
│ ↑ 15% vs T11│ ↑ 8% vs T11  │ Vòng quay: 4.2x   │
├──────────────┼──────────────┼───────────────────┤
│ Phải thu KH  │ Phải trả NCC │ Tiền mặt          │
│ 850 triệu   │ 420 triệu    │ 1.2 tỷ            │
│ Quá hạn: 12%│ Đến hạn: 3   │ Dự báo 30 ngày:   │
│             │ ngày tới     │ +200 triệu        │
└─────────────┴──────────────┴───────────────────┘
```

---

## 10. Tư duy BA khi làm việc với ERP

### 10.1 Framework phân tích nghiệp vụ ERP

Khi gặp bất kỳ yêu cầu nào về ERP, hỏi theo framework này:

**1. Ai làm? (Actor)**
```
Nhân viên kho? Kế toán? Manager? Hệ thống tự động?
```

**2. Làm gì? (Action)**
```
Nhập liệu? Phê duyệt? Xem báo cáo? Xuất file?
```

**3. Dữ liệu đầu vào là gì? (Input)**
```
Từ module nào? Giấy tờ gì? Ai nhập?
```

**4. Kết quả tạo ra là gì? (Output)**
```
Chứng từ gì? Bút toán gì? Thông báo đi đâu?
```

**5. Điều kiện phê duyệt là gì? (Approval)**
```
Ai approve? Ngưỡng bao nhiêu? Escalate khi nào?
```

**6. Ảnh hưởng đến module nào khác? (Integration)**
```
Hành động này trigger gì ở Finance? Inventory? HR?
```

### 10.2 Những sai lầm phổ biến khi build ERP nội bộ

**Sai lầm 1: Bỏ qua double-entry accounting**

Nhiều team build "kế toán" nhưng không hiểu nguyên tắc double-entry → hệ thống không balance → kế toán không dùng được.

**Sai lầm 2: Không phân biệt Document Date và Posting Date**

```
Document Date: Ngày trên chứng từ thực tế (ngày hóa đơn: 31/12)
Posting Date:  Ngày ghi nhận vào sổ kế toán (có thể: 02/01 năm sau)

Quan trọng cho: Đóng sổ cuối kỳ, kỳ kế toán
```

**Sai lầm 3: Không có Audit Trail**

Mọi thay đổi trong ERP phải được log: ai sửa, sửa gì, lúc nào. Đây là yêu cầu pháp lý và nghiệp vụ.

**Sai lầm 4: Xóa dữ liệu thay vì Cancel/Reverse**

Trong kế toán, không bao giờ được xóa chứng từ đã phê duyệt. Phải tạo chứng từ đảo ngược (Reversal Entry).

**Sai lầm 5: Không có phân quyền theo vai trò**

```
Nguyên tắc Segregation of Duties:
  Người tạo PO ≠ Người approve PO
  Người nhập hàng ≠ Người approve thanh toán
  Người tạo nhân viên mới ≠ Người phê duyệt lương
```

### 10.3 Câu hỏi BA phải hỏi khi gathering requirements

**Về nghiệp vụ:**
- Quy trình hiện tại đang làm như thế nào? (As-Is)
- Pain point lớn nhất là gì?
- Ai là người dùng cuối của tính năng này?
- Tần suất thực hiện: hàng ngày, hàng tuần, hàng tháng?

**Về phê duyệt:**
- Ai có quyền approve? Cần bao nhiêu cấp?
- Ngưỡng phê duyệt là bao nhiêu?
- Nếu người approve vắng mặt thì sao? (Delegation)

**Về tích hợp:**
- Dữ liệu này đến từ đâu?
- Sau khi thực hiện, thông tin được gửi đến đâu?
- Có cần tích hợp với hệ thống bên ngoài không? (ngân hàng, BHXH, thuế)

**Về báo cáo:**
- Ai cần xem báo cáo này?
- Cần xem theo chiều nào? (theo thời gian, theo phòng ban, theo sản phẩm)
- Cần export ra format gì?

---

## 11. Tổng kết

### Mental Model cho ERP

```
ERP = Một bộ não chung cho cả doanh nghiệp

Mỗi phòng ban làm việc của mình,
nhưng dữ liệu được chia sẻ tự động,
không cần email qua lại, không cần nhập lại,
không sợ số liệu mâu thuẫn.
```

### Thứ tự quan trọng khi build ERP nội bộ

```
[1] Finance & Accounting — nền tảng, build trước
    Vì: Mọi module đều tạo ra bút toán kế toán

[2] Inventory — kết nối chặt với Finance
    Vì: Hàng tồn kho là tài sản, phải phản ánh đúng

[3] Purchasing — kết nối với Inventory và Finance (AP)
    Vì: Mua hàng → nhập kho → tạo công nợ phải trả

[4] Sales — kết nối với Inventory và Finance (AR)
    Vì: Bán hàng → xuất kho → tạo công nợ phải thu

[5] HR & Payroll — tương đối độc lập hơn
    Nhưng: Chi phí lương phải feed vào Finance

[6] CRM — thường build sau cùng hoặc tích hợp từ ngoài vào
```

### Bảng thuật ngữ nhanh

| Thuật ngữ | Ý nghĩa |
|---|---|
| **GL (General Ledger)** | Sổ cái — nơi tổng hợp tất cả bút toán |
| **CoA (Chart of Accounts)** | Hệ thống tài khoản kế toán |
| **Journal Entry** | Bút toán kế toán (Nợ - Có) |
| **AP (Accounts Payable)** | Phải trả NCC |
| **AR (Accounts Receivable)** | Phải thu KH |
| **PO (Purchase Order)** | Đơn đặt hàng mua |
| **SO (Sales Order)** | Đơn hàng bán |
| **GR (Goods Receipt)** | Phiếu nhập kho |
| **GI (Goods Issue)** | Phiếu xuất kho |
| **3-way Match** | Đối chiếu PO + GR + Invoice trước khi thanh toán |
| **FIFO / WA** | Phương pháp tính giá tồn kho |
| **SKU** | Mã định danh sản phẩm/biến thể |
| **Reorder Point** | Ngưỡng tồn kho tối thiểu để đặt hàng |
| **Cost Center** | Đơn vị chỉ phát sinh chi phí |
| **Profit Center** | Đơn vị có cả doanh thu và chi phí |
| **Payroll Run** | Quy trình tính lương hàng tháng |
| **Credit Limit** | Hạn mức tín dụng cho phép nợ của khách hàng |
| **Aging Report** | Báo cáo phân tích tuổi nợ |
| **Audit Trail** | Lịch sử mọi thay đổi trong hệ thống |
| **Segregation of Duties** | Nguyên tắc phân tách quyền hạn để tránh gian lận |
| **P2P (Procure to Pay)** | Toàn bộ luồng từ nhu cầu mua đến thanh toán NCC |
| **O2C (Order to Cash)** | Toàn bộ luồng từ đặt hàng đến thu tiền |