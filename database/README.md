# Database — Hiểu Từ Đầu Đến Cuối

> Tài liệu này giải thích database từ số 0. Không cần biết trước gì cả.  
> Mỗi khái niệm đều được giải thích *tại sao nó tồn tại* trước khi nói *nó là gì*.

---

# CHƯƠNG 1: Tại Sao Database Tồn Tại?

## Vấn đề ban đầu

Bạn xây một ứng dụng. Ứng dụng cần lưu dữ liệu — tên người dùng, đơn hàng, tin nhắn, bất cứ thứ gì. Đơn giản nhất: lưu vào file.

```
users.txt:
1,Nguyen Van An,an@gmail.com,25
2,Tran Thi Binh,binh@gmail.com,30
3,Le Van Chau,chau@gmail.com,28
```

Lúc đầu ổn. Sau đó vấn đề xuất hiện:

**Vấn đề 1 — Tìm kiếm chậm.**  
Muốn tìm người dùng có email "chau@gmail.com" → phải đọc từ đầu đến cuối file, so sánh từng dòng. File có 1 triệu người dùng → đọc 1 triệu dòng mỗi lần tìm. Chậm không thể chấp nhận được.

**Vấn đề 2 — Nhiều người dùng cùng lúc gây loạn.**  
Người A đang đọc file, người B đang ghi file. Người A đọc được nửa chừng thì người B ghi đè → người A nhận dữ liệu nửa cũ nửa mới, vô nghĩa. Tệ hơn: 2 người cùng ghi lúc đó → file bị corrupt hoàn toàn.

**Vấn đề 3 — Mất điện = mất dữ liệu.**  
Đang ghi 1000 đơn hàng vào file, ghi được 600 cái thì mất điện. File bây giờ có 600 đơn hàng hợp lệ và phần còn lại là rác. Không biết đâu là đâu.

**Vấn đề 4 — Dữ liệu không nhất quán.**  
Một đơn hàng tham chiếu đến user_id=999, nhưng không có user nào có id=999. Dữ liệu bị "treo lơ lửng", không có ý nghĩa.

**Database sinh ra để giải quyết đúng 4 vấn đề này.**

---

# CHƯƠNG 2: Bên Trong Database — Dữ Liệu Thực Sự Được Lưu Thế Nào

Trước khi học bất cứ thứ gì khác, bạn cần hiểu database thực sự *làm gì với dữ liệu của bạn*. Không có gì huyền bí — tất cả đều là file trên ổ cứng.

## 2.1 Tất cả cuối cùng đều là file

Cài PostgreSQL lên máy, dữ liệu nằm ở đây:

```
/var/lib/postgresql/data/base/
    └── 16384/          ← thư mục của database "myapp"
        ├── 24601       ← file lưu bảng "users"
        ├── 24605       ← file lưu bảng "orders"
        └── 24610       ← file lưu index của bảng "users"
```

Mỗi bảng = một (hoặc nhiều) file. Mở file ra bạn thấy binary data — không đọc được bằng mắt thường vì database tự định nghĩa format để lưu hiệu quả.

## 2.2 Page — Đơn vị nhỏ nhất của mọi thứ

Database không đọc/ghi từng byte hay từng row. Nó chia file thành các **page**, mỗi page thường là **8KB**.

Tại sao? Vì ổ cứng hoạt động theo cơ chế vật lý: đọc 1 byte hay đọc 8KB mất thời gian gần như nhau (đầu đọc phải di chuyển đến đúng vị trí — đó là phần tốn thời gian). Vậy thì cứ đọc 8KB một lần cho tiết kiệm công.

```
File bảng "users":
┌─────────────────────────────────────────┐
│  Page 0 (8192 bytes)                    │
│  ┌─────────────────────────────────────┐│
│  │ Header: checksum, free space ptr   ││
│  │─────────────────────────────────────││
│  │ Row 1: id=1, name="An",  age=25    ││
│  │ Row 2: id=2, name="Bình",age=30    ││
│  │ Row 3: id=3, name="Châu",age=28    ││
│  │ [còn 3KB trống]                    ││
│  └─────────────────────────────────────┘│
│  Page 1 (8192 bytes)                    │
│  ┌─────────────────────────────────────┐│
│  │ Row 4: id=4, name="Dũng",age=22    ││
│  │ Row 5: ...                         ││
│  └─────────────────────────────────────┘│
└─────────────────────────────────────────┘
```

Khi bạn chạy `SELECT * FROM users WHERE id = 3`:
1. Database biết row id=3 nằm ở Page 0
2. Đọc toàn bộ Page 0 vào RAM (8KB một lần)
3. Lọc ra row id=3
4. Trả kết quả

## 2.3 RAM Cache — Tại sao database nhanh hơn bạn nghĩ

Đọc từ ổ cứng (HDD) mất khoảng **10ms**.  
Đọc từ RAM mất khoảng **0.0001ms**.  
**Chênh nhau 100,000 lần.**

Database dành ra một vùng RAM gọi là **Buffer Pool** để cache các page thường dùng:

```
Lần đầu đọc bảng users:
  → "Page 0 có trong RAM chưa?" → Chưa
  → Đọc từ disk vào RAM (tốn 10ms)
  → Lưu vào Buffer Pool

Lần hai, ba, bốn đọc bảng users:
  → "Page 0 có trong RAM chưa?" → Rồi!
  → Lấy từ RAM luôn (tốn 0.0001ms)
```

Đây là lý do database mới khởi động chạy chậm — Buffer Pool còn trống. Sau vài phút chạy ấm, Buffer Pool đã có đủ dữ liệu thường dùng → nhanh hơn rất nhiều.

> **Buffer Pool** càng lớn → cache được nhiều page → ít phải đọc disk → nhanh hơn. Đây là lý do khi cấu hình database, câu hỏi đầu tiên luôn là "có bao nhiêu RAM?"

## 2.4 Chuyện gì xảy ra khi bạn INSERT một row?

```sql
INSERT INTO users (name, age) VALUES ('Emre', 26);
```

Nghe có vẻ đơn giản. Thực ra:

```
Bước 1: Tìm page nào còn chỗ trống
Bước 2: Ghi vào WAL (nhật ký) — giải thích ở chương sau
Bước 3: Thêm row vào page đó TRONG RAM (Buffer Pool)
Bước 4: Đánh dấu page này là "dirty" (đã thay đổi, chưa xuống disk)
Bước 5: Sau một lúc, background process ghi page dirty xuống disk
```

Bước 3 và 5 tách nhau vì lý do hiệu năng: ghi vào RAM ngay lập tức (nhanh), ghi xuống disk sau (chậm nhưng làm ngầm, không block user).

---

# CHƯƠNG 3: Index — Tại Sao Tìm Kiếm Nhanh

## 3.1 Vấn đề không có index

Bảng `users` có 10 triệu row. Bạn chạy:

```sql
SELECT * FROM users WHERE email = 'chau@gmail.com';
```

Không có index → database phải đọc từng row một, so sánh email, cho đến khi tìm thấy. Gọi là **Sequential Scan** (quét tuần tự). Với 10 triệu row → đọc tối đa 10 triệu row → chậm.

## 3.2 Index là gì — Tương tự mục lục sách

Sách 1000 trang, bạn muốn tìm từ "database". Không có mục lục → đọc từ trang 1 đến trang 1000. Có mục lục → tra mục lục, thấy "database: trang 42, 87, 156" → lật thẳng đến đúng trang.

Index trong database hoạt động y hệt: là một **cấu trúc dữ liệu riêng**, lưu giá trị cột + vị trí của row tương ứng, được sắp xếp để tìm nhanh.

```sql
CREATE INDEX idx_users_email ON users(email);
```

Sau lệnh này, database tạo thêm một file index riêng:

```
Index file (sắp xếp theo email):
  an@gmail.com     → Page 0, Row 1
  binh@gmail.com   → Page 0, Row 2
  chau@gmail.com   → Page 0, Row 3
  dung@gmail.com   → Page 1, Row 4
  ...
```

Giờ tìm "chau@gmail.com":
1. Tra vào index → "chau@gmail.com nằm ở Page 0, Row 3"
2. Đọc đúng Page 0
3. Lấy Row 3

Thay vì đọc 10 triệu row → chỉ đọc vài bước trong index. **Nhanh hơn hàng nghìn lần.**

## 3.3 Bên trong Index — Cấu trúc B-Tree

Index không phải là danh sách thẳng — nếu vậy thì tìm kiếm trong index cũng chậm. Index dùng cấu trúc gọi là **B-Tree** (Balanced Tree — cây cân bằng).

Hãy hình dung như cây thư mục:

```
                    [M]                    ← Tầng gốc (root)
                   /   \
              [F]         [T]              ← Tầng giữa
             /   \       /   \
          [C]   [H]   [P]   [W]           ← Lá (leaf) — chứa data thực
```

Tìm "chau@gmail.com" bắt đầu bằng 'c':
- Bắt đầu ở root [M]: 'c' < 'm' → đi trái
- Đến [F]: 'c' < 'f' → đi trái
- Đến [C]: tìm thấy → lấy vị trí row

Bất kể index có bao nhiêu triệu entry, cũng chỉ cần **3-4 bước** để tìm ra (vì mỗi bước loại bỏ một nửa). Đây là tìm kiếm **O(log n)** — cực kỳ hiệu quả.

**Điều quan trọng cần nhớ về leaf nodes:**  
Các node lá được nối với nhau thành danh sách liên kết:

```
[C] ↔ [D] ↔ [E] ↔ [F] ↔ [G] ↔ ...
```

Điều này cho phép range query rất nhanh:

```sql
SELECT * FROM users WHERE age BETWEEN 25 AND 30;
```

→ Tìm node lá có age=25, rồi đi theo linked list đến age=30. Không cần quay lại từ đầu.

## 3.4 Index có giá của nó

Index không miễn phí:

- **Tốn disk**: Mỗi index là một file riêng, có thể to bằng bảng chính
- **Làm chậm write**: Mỗi khi INSERT/UPDATE/DELETE, database phải cập nhật tất cả index liên quan
- **Không phải index nào cũng được dùng**: Database tự quyết định dùng index hay không dựa trên ước tính

```
Ví dụ index phản tác dụng:
  Cột "gender" chỉ có 2 giá trị: 'male', 'female'
  Query: SELECT * FROM users WHERE gender = 'male'
  → 50% bảng là kết quả → database thấy: dùng index + đọc 5 triệu row
    còn tốn hơn là quét thẳng toàn bảng
  → Database bỏ qua index, quét thẳng
```

**Quy tắc thực tế**: Index cột nào mà query thường dùng để lọc, cột có nhiều giá trị phân biệt (email, id, phone), không phải cột boolean hay status.

## 3.5 Composite Index — Index nhiều cột

```sql
CREATE INDEX idx_city_age ON users(city, age);
```

Index này hữu ích cho:

```sql
WHERE city = 'HCM'                    ✓ dùng được
WHERE city = 'HCM' AND age = 25      ✓ dùng được (hiệu quả nhất)
WHERE city = 'HCM' AND age > 20      ✓ dùng được
WHERE age = 25                        ✗ KHÔNG dùng được
```

**Quy tắc Leftmost:** Index composite chỉ dùng được nếu query bao gồm cột **đầu tiên** của index. Vì trong B-Tree, dữ liệu được sắp xếp theo cột đầu tiên trước — bỏ qua cột đầu thì không biết bắt đầu tìm từ đâu.

---

# CHƯƠNG 4: ACID — Bốn Lời Hứa Của Database

Đây là phần quan trọng nhất. ACID là lý do bạn có thể tin tưởng database với dữ liệu quan trọng.

Câu hỏi cần đặt ra trước: **Điều gì có thể đi sai?**

- Máy tính mất điện giữa chừng khi đang ghi dữ liệu
- 1000 người cùng mua cùng 1 sản phẩm cuối
- Chuyển tiền: trừ xong chưa kịp cộng thì hệ thống crash
- Dữ liệu bị ghi thiếu, ghi sai, ghi không đồng bộ

ACID là bốn cam kết của database để đảm bảo **không có chuyện nào trong số này xảy ra.**

---

## A — Atomicity (Tính Nguyên Tử)

### Vấn đề

Bạn chuyển 500k từ tài khoản A sang tài khoản B. Cần 2 bước:

```
Bước 1: Trừ 500k khỏi A    → thành công ✓
Bước 2: Cộng 500k vào B    → MÁY TÍNH MẤT ĐIỆN ✗
```

A đã mất 500k. B chưa nhận được. **500k bốc hơi hoàn toàn.**

### Giải pháp: Transaction

Database cho phép gom nhiều bước thành một **transaction** — một đơn vị không thể chia cắt:

```sql
BEGIN;
  UPDATE accounts SET balance = balance - 500000 WHERE id = 'A';
  UPDATE accounts SET balance = balance + 500000 WHERE id = 'B';
COMMIT;
```

Ngữ nghĩa: Hoặc **cả hai bước đều xảy ra**, hoặc **không có bước nào xảy ra cả**. Không có trạng thái giữa chừng.

Nếu hệ thống crash sau bước 1, trước bước 2 → khi khởi động lại, database phát hiện transaction chưa hoàn thành → **tự động hủy bỏ bước 1** → A vẫn còn đủ 500k.

### Cơ chế: WAL (Write-Ahead Log)

Đây là thứ khiến Atomicity có thể hoạt động. Trước khi thay đổi bất cứ gì, database ghi nhật ký trước:

```
WAL — ghi tuần tự vào cuối file (rất nhanh):
──────────────────────────────────────────────────────
[#1001] BEGIN transaction T500
[#1002] BEFORE: accounts A = 1,000,000
[#1002] AFTER:  accounts A = 500,000
[#1003] BEFORE: accounts B = 200,000
[#1003] AFTER:  accounts B = 700,000
[#1004] COMMIT transaction T500
──────────────────────────────────────────────────────
```

**Kịch bản crash:**

```
Crash xảy ra sau #1002, trước #1003:
  → WAL có BEGIN nhưng không có COMMIT
  → Khi restart: thấy transaction chưa commit → undo #1002
  → A được hoàn lại 500k → như chưa có gì xảy ra ✓

Crash xảy ra sau #1004 (COMMIT xong):
  → WAL có đủ BEGIN, changes, COMMIT
  → Khi restart: redo lại tất cả changes từ WAL
  → Cả A và B được cập nhật đúng ✓
```

> WAL được ghi **tuần tự** (append vào cuối file) → rất nhanh, dù ổ cứng chậm đến đâu. Đây là lý do database có thể vừa an toàn vừa không quá chậm.

---

## C — Consistency (Tính Nhất Quán)

### Vấn đề

Atomicity đảm bảo transaction hoàn thành hoặc không. Nhưng nếu transaction hoàn thành mà dữ liệu vẫn sai thì sao?

```sql
-- Tài khoản A chỉ có 100k
UPDATE accounts SET balance = balance - 500000 WHERE id = 'A';
-- Transaction hoàn thành "thành công" → balance = -400,000
-- Atomicity không có lỗi gì, nhưng dữ liệu vô lý
```

### Giải pháp: Constraints (Ràng buộc)

Database cho phép định nghĩa **rules** — những điều kiện dữ liệu phải thỏa mãn mọi lúc:

```sql
CREATE TABLE accounts (
    id      VARCHAR(10) PRIMARY KEY,         -- id phải unique, không null
    balance DECIMAL CHECK (balance >= 0),    -- số dư không được âm
    user_id INT REFERENCES users(id)         -- user phải tồn tại trong bảng users
);

CREATE TABLE orders (
    id         BIGINT PRIMARY KEY,
    product_id INT REFERENCES products(id),  -- sản phẩm phải tồn tại
    quantity   INT CHECK (quantity > 0),      -- số lượng phải dương
    amount     DECIMAL NOT NULL               -- không được null
);
```

Bất kỳ transaction nào vi phạm → **database từ chối, tự rollback**:

```sql
UPDATE accounts SET balance = balance - 500000 WHERE id = 'A';
-- ERROR: new row violates check constraint "accounts_balance_check"
-- Transaction tự động rollback → A vẫn còn đủ tiền ✓
```

**Consistency là hai lớp:**

```
Lớp 1 — Database tự kiểm tra:
  PRIMARY KEY, FOREIGN KEY, UNIQUE, CHECK, NOT NULL

Lớp 2 — Developer phải tự kiểm tra (DB không biết):
  "Không cho đặt hàng khi hàng đã hết"
  "User phải verify email trước khi mua"
  "Giá không được thay đổi khi đơn đã được xác nhận"
```

---

## I — Isolation (Tính Cô Lập)

### Vấn đề — Đây là tính chất phức tạp nhất

Atomicity và Consistency xử lý một transaction đơn lẻ. Isolation xử lý khi **nhiều transaction chạy cùng lúc**.

Hệ thống thực tế có hàng nghìn transaction mỗi giây. Nếu chúng can thiệp vào nhau — dữ liệu sẽ sai theo những cách rất khó debug.

**Tình huống thực tế:** Flash sale, 1000 người cùng mua 1 sản phẩm cuối:

```
Thời gian │ User A                          │ User B
──────────┼─────────────────────────────────┼──────────────────────────────
  t=1     │ Đọc: tồn kho = 1               │
  t=2     │                                 │ Đọc: tồn kho = 1
  t=3     │ Kiểm tra: còn hàng → cho mua   │
  t=4     │                                 │ Kiểm tra: còn hàng → cho mua
  t=5     │ Trừ tồn kho → tồn kho = 0      │
  t=6     │                                 │ Trừ tồn kho → tồn kho = -1 !!!
  t=7     │ COMMIT                          │ COMMIT
```

Kết quả: 1 sản phẩm bán được cho 2 người. Tồn kho = -1. **Thảm họa.**

### Cơ chế 1: Locking

Đơn giản nhất: ai đọc/ghi row nào thì lock row đó lại, người khác phải chờ.

```sql
-- Transaction A: đọc và lock row ngay
SELECT stock FROM products WHERE id = 1 FOR UPDATE;
-- "FOR UPDATE" = lock row này lại, không ai được đọc/ghi cho đến khi tôi xong

-- Transaction B cố đọc cùng row → phải chờ A xong
-- A xong (stock = 0) → B mới được đọc → thấy stock = 0 → hết hàng ✓
```

**Vấn đề của locking:** Nhiều transaction chờ nhau → hệ thống tắc nghẽn. Càng nhiều lock → càng chậm.

### Cơ chế 2: MVCC (Multi-Version Concurrency Control)

Giải pháp hiện đại hơn. Thay vì lock, database **lưu nhiều phiên bản** của cùng một row.

Ý tưởng cốt lõi: **"Readers không block writers, writers không block readers."**

Cách hoạt động:

```
Database gắn 2 nhãn thời gian vào mỗi row:
  - xmin: transaction nào TẠO ra row này
  - xmax: transaction nào XÓA/CẬP NHẬT row này (0 = row còn sống)

Bảng accounts:
┌────────┬──────┬──────┬─────────┐
│ xmin   │ xmax │  id  │ balance │
├────────┼──────┼──────┼─────────┤
│  100   │  0   │  A   │ 1000k   │  ← row hiện tại, đang sống
└────────┴──────┴──────┴─────────┘

Transaction T200 cập nhật balance của A:
┌────────┬──────┬──────┬─────────┐
│  100   │ 200  │  A   │ 1000k   │  ← row cũ, bị T200 "xóa"
│  200   │  0   │  A   │  500k   │  ← row mới, do T200 tạo
└────────┴──────┴──────┴─────────┘
```

Một transaction T150 (bắt đầu trước T200) hỏi "balance của A là bao nhiêu?":
→ Thấy row xmin=100, xmax=200: row này tồn tại từ T100, bị "xóa" bởi T200  
→ T150 bắt đầu ở T150, T200 > T150 → với T150, T200 chưa xảy ra  
→ T150 thấy **balance = 1000k** (snapshot của thời điểm T150 bắt đầu) ✓

Một transaction T300 (bắt đầu sau T200) hỏi tương tự:  
→ T200 < T300 → T200 đã xảy ra rồi  
→ T300 thấy **balance = 500k** ✓

**Không có lock nào. Cả hai đọc được đồng thời. Không ai block ai.**

### Các mức Isolation — Trade-off giữa đúng và nhanh

Isolation hoàn hảo = mọi transaction như chạy tuần tự = rất chậm. Thực tế có 4 mức:

**Mức 1 — Read Uncommitted**

```
Đọc cả dữ liệu chưa commit của transaction khác.

T1: UPDATE balance = 500k (chưa COMMIT)
T2: SELECT balance → thấy 500k  ← "đọc bẩn" (dirty read)
T1: ROLLBACK → balance quay về 1000k
T2: Đã đưa ra quyết định dựa trên số liệu sai.
```

Gần như không ai dùng mức này trừ khi cần tốc độ tuyệt đối và chấp nhận sai.

**Mức 2 — Read Committed** *(PostgreSQL mặc định)*

```
Chỉ đọc dữ liệu đã commit. Không có dirty read.

Nhưng:
T1: SELECT balance → 1000k
                          T2: UPDATE balance = 500k; COMMIT;
T1: SELECT balance → 500k  ← CÙNG query, kết quả khác nhau (non-repeatable read)
```

Ổn cho hầu hết ứng dụng web thông thường.

**Mức 3 — Repeatable Read** *(MySQL mặc định)*

```
Đọc lại row đó luôn ra cùng kết quả trong suốt transaction.

T1: SELECT balance → 1000k
                          T2: UPDATE balance = 500k; COMMIT;
T1: SELECT balance → vẫn 1000k ✓ (snapshot không thay đổi)

Nhưng:
T1: SELECT COUNT(*) FROM orders WHERE user_id=1 → 5
                          T2: INSERT INTO orders...; COMMIT;
T1: SELECT COUNT(*) FROM orders WHERE user_id=1 → 6  ← thêm row mới (phantom read)
```

**Mức 4 — Serializable**

```
Mọi transaction như chạy tuần tự, từng cái một.
Không có bất kỳ anomaly nào.
Đổi lại: chậm hơn đáng kể.
```

**Chọn mức nào?**

| Ứng dụng | Mức nên dùng | Lý do |
|---|---|---|
| Tài khoản ngân hàng, chuyển tiền | Serializable hoặc Repeatable Read | Không thể sai |
| Đặt hàng, thanh toán | Repeatable Read | Cần chính xác |
| Web app thông thường | Read Committed | Balance giữa đúng và nhanh |
| Đọc số liệu báo cáo, analytics | Read Committed | Sai vài row không quan trọng |

---

## D — Durability (Tính Bền Vững)

### Vấn đề

```
Bạn chuyển tiền. Màn hình hiện: "Giao dịch thành công!"
3 giây sau: server mất điện, restart.
Khi lên lại: giao dịch biến mất.
```

Database nói "thành công" — nhưng dữ liệu lại mất. Đây là thứ không thể chấp nhận.

### Giải pháp: WAL + fsync

Câu trả lời đơn giản: **Database không được phép nói "thành công" cho đến khi dữ liệu thực sự an toàn trên disk.**

Cụ thể, luồng COMMIT:

```
Bạn gửi COMMIT
     │
     ▼
[1] Ghi "COMMIT" vào WAL
     │
     ▼
[2] fsync(WAL file)
     │   ← gọi thẳng vào OS: "Hãy đảm bảo file này thực sự đã xuống disk
     │      đừng giữ trong cache của mày nữa"
     │   ← Đây là bước chậm nhất: phải chờ đĩa cứng xác nhận
     │
     ▼
[3] Trả về "OK" cho bạn  ← chỉ sau khi [2] hoàn thành
     │
     ▼
[4] Ghi page thực sự xuống disk  ← làm ngầm sau đó, không cần vội
```

Nếu mất điện sau [2]: WAL đã an toàn trên disk → restart → đọc WAL → khôi phục ✓  
Nếu mất điện trước [2]: Bạn chưa nhận được "OK" → biết ngay là thất bại ✓

**fsync là chìa khóa.** Không có fsync → data có thể nằm trong OS cache → mất điện → mất data. Một số hệ thống tắt fsync để nhanh hơn — đây là đánh đổi Durability lấy performance. Chỉ làm vậy khi bạn biết mình đang làm gì.

---

## Nhìn ACID Qua Một Ví Dụ Hoàn Chỉnh

Bạn mua laptop 25 triệu trên sàn TMĐT. Hệ thống cần làm 3 việc đồng thời:

```sql
BEGIN;
  -- Trừ tiền
  UPDATE accounts
  SET balance = balance - 25000000
  WHERE user_id = 'you';

  -- Tạo đơn hàng
  INSERT INTO orders (user_id, product_id, amount, status)
  VALUES ('you', 'laptop_123', 25000000, 'confirmed');

  -- Trừ tồn kho
  UPDATE inventory
  SET stock = stock - 1
  WHERE product_id = 'laptop_123';
COMMIT;
```

| Tính chất | Bảo vệ bạn khỏi điều gì |
|---|---|
| **Atomicity** | Trừ tiền xong nhưng tạo đơn hàng lỗi → tiền được hoàn lại tự động |
| **Consistency** | Tồn kho không thể xuống -1, balance không thể âm |
| **Isolation** | 100 người cùng mua laptop cuối → chỉ đúng 1 người mua được |
| **Durability** | "Đặt hàng thành công" thì dù server cháy ngay sau đó → đơn hàng vẫn còn |

---

# CHƯƠNG 5: Query Optimizer — Database Thực Ra Không Chạy Query Của Bạn

Đây là thứ hầu hết người dùng DB không biết.

Khi bạn viết SQL, bạn mô tả **bạn muốn gì**, không phải **làm thế nào**. Database tự quyết định cách thực thi hiệu quả nhất.

## 5.1 SQL là ngôn ngữ khai báo

```sql
SELECT u.name, COUNT(o.id) as order_count
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.city = 'HCM'
GROUP BY u.id, u.name
ORDER BY order_count DESC
LIMIT 10;
```

Bạn không nói: "Đọc bảng users trước, rồi với mỗi user đọc orders..."  
Bạn nói: "Cho tôi 10 user ở HCM có nhiều đơn hàng nhất."  
**Database tự figure out cách làm.**

## 5.2 Thứ tự thực thi thực sự

Bạn viết SQL theo thứ tự: SELECT → FROM → WHERE → GROUP BY → ORDER BY  
Database thực thi theo thứ tự khác hoàn toàn:

```
1. FROM      — Xác định bảng nào cần dùng
2. JOIN      — Kết hợp các bảng
3. WHERE     — Lọc row (lọc càng sớm càng tốt → giảm data cần xử lý)
4. GROUP BY  — Nhóm lại
5. HAVING    — Lọc nhóm (nếu có)
6. SELECT    — Chọn cột cần trả về
7. ORDER BY  — Sắp xếp
8. LIMIT     — Cắt kết quả
```

**Tại sao điều này quan trọng?**

```sql
-- Query này LỖI — vì WHERE chạy trước SELECT
-- Alias "order_count" trong SELECT chưa tồn tại khi WHERE chạy
SELECT COUNT(id) as order_count
FROM orders
WHERE order_count > 5;  ← ERROR: column "order_count" does not exist

-- Đúng: dùng HAVING (chạy sau GROUP BY)
SELECT user_id, COUNT(id) as order_count
FROM orders
GROUP BY user_id
HAVING COUNT(id) > 5;  ← HAVING chạy sau GROUP BY → hợp lệ
```

## 5.3 EXPLAIN — Nhìn vào não của Database

```sql
EXPLAIN ANALYZE
SELECT u.name, COUNT(o.id)
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.city = 'HCM'
GROUP BY u.id;
```

Kết quả:

```
HashAggregate  (cost=1250..1280 rows=3000)
               (actual time=45ms..47ms rows=2800)
  →  Hash Join  (cost=450..1150 rows=20000)
                (actual time=12ms..38ms rows=19500)
       Hash Cond: (o.user_id = u.id)
       →  Seq Scan on orders
                (actual time=0ms..15ms rows=50000)
       →  Index Scan on users  (using idx_city)
                Index Cond: (city = 'HCM')
                (actual time=0ms..10ms rows=3800)
```

**Đọc từ trong ra ngoài** (indent sâu nhất chạy trước):

```
1. Index Scan on users  → Dùng index để lấy users ở HCM (3800 users)
2. Seq Scan on orders   → Quét toàn bộ bảng orders (50000 rows) — đáng lo!
3. Hash Join            → Kết hợp users và orders
4. HashAggregate        → GROUP BY + COUNT
```

**Các dấu hiệu cần chú ý:**

```
Seq Scan trên bảng lớn → Thiếu index, cần thêm
cost ước tính vs actual chênh nhiều (10x+) → Statistics lỗi thời, cần ANALYZE
rows ước tính sai nhiều → Database chọn plan sai → query chậm
```

---

# CHƯƠNG 6: Normalization — Thiết Kế Bảng Đúng Cách

## 6.1 Vấn đề khi thiết kế bảng sai

Tưởng tượng bảng quản lý đơn hàng này:

```
orders:
┌──────────┬──────────────┬──────────────┬───────────────┬────────────┐
│ order_id │ customer_name│ customer_city│ product_name  │ quantity   │
├──────────┼──────────────┼──────────────┼───────────────┼────────────┤
│ 1        │ Nguyen An    │ HCM          │ Laptop Dell   │ 1          │
│ 2        │ Nguyen An    │ HCM          │ Chuột Logitech│ 2          │
│ 3        │ Tran Binh    │ HN           │ Laptop Dell   │ 1          │
└──────────┴──────────────┴──────────────┴───────────────┴────────────┘
```

Vấn đề:

- **Dữ liệu lặp lại**: "Nguyen An", "HCM" lặp 2 lần. Triệu đơn hàng → lặp triệu lần
- **Cập nhật nguy hiểm**: An chuyển từ HCM sang HN → phải UPDATE tất cả row của An. Quên 1 row → dữ liệu không nhất quán
- **Xóa mất dữ liệu**: Xóa đơn hàng cuối của "Tran Binh" → mất luôn thông tin về Binh

## 6.2 Normalization — Loại Bỏ Lặp Lại

Normalization là quá trình tổ chức lại bảng để loại bỏ dữ liệu lặp.

**1NF — Mỗi ô chứa một giá trị duy nhất:**

```
❌ Sai:
┌──────────┬──────────────────────────────┐
│ order_id │ products                     │
│ 1        │ "Laptop Dell, Chuột Logitech"│  ← nhiều giá trị trong 1 ô
└──────────┴──────────────────────────────┘

✅ Đúng: Mỗi sản phẩm một dòng riêng
┌──────────┬───────────────┐
│ order_id │ product       │
│ 1        │ Laptop Dell   │
│ 1        │ Chuột Logitech│
└──────────┴───────────────┘
```

**2NF — Tách thứ chỉ phụ thuộc vào một phần của key:**

```
❌ Sai: (order_id, product_id) là primary key
        nhưng product_name chỉ phụ thuộc product_id, không phụ thuộc order_id
┌──────────┬────────────┬──────────────┬──────────┐
│ order_id │ product_id │ product_name │ quantity │
│ 1        │ P1         │ Laptop Dell  │ 1        │
│ 2        │ P1         │ Laptop Dell  │ 2        │  ← lặp product_name
└──────────┴────────────┴──────────────┴──────────┘

✅ Đúng: Tách ra 2 bảng
  products: (product_id, product_name)
  order_items: (order_id, product_id, quantity)
```

**3NF — Tách thứ phụ thuộc vào cột không phải key:**

```
❌ Sai: city phụ thuộc zip_code, không phụ thuộc trực tiếp vào user_id
┌─────────┬──────────┬──────────┐
│ user_id │ zip_code │ city     │
│ 1       │ 700000   │ HCM      │
│ 2       │ 700000   │ HCM      │  ← lặp city
└─────────┴──────────┴──────────┘

✅ Đúng: Tách ra
  users: (user_id, zip_code)
  zip_codes: (zip_code, city)
```

**Kết quả sau khi normalize bảng orders ở đầu:**

```
users:     (user_id, name, city_id)
cities:    (city_id, city_name)
products:  (product_id, product_name, price)
orders:    (order_id, user_id, created_at)
order_items:(order_id, product_id, quantity)
```

Bây giờ:
- An chuyển thành phố → UPDATE 1 row trong bảng `users` → tự động đúng ở mọi chỗ
- Xóa đơn hàng → thông tin khách hàng và sản phẩm vẫn còn
- Không có gì bị lặp → tiết kiệm storage, dễ maintain

## 6.3 Khi nào Denormalize?

Normalize xong rồi, đôi khi lại cần **denormalize** (làm ngược lại) vì performance.

```sql
-- Mỗi lần xem đơn hàng, phải JOIN 5 bảng:
SELECT o.id, u.name, c.city_name, p.product_name, oi.quantity
FROM orders o
JOIN users u ON o.user_id = u.user_id
JOIN cities c ON u.city_id = c.city_id
JOIN order_items oi ON o.id = oi.order_id
JOIN products p ON oi.product_id = p.product_id;
```

Nếu query này chạy 10,000 lần/giây và vẫn chậm, bạn có thể thêm cột thẳng vào `orders`:

```sql
ALTER TABLE orders ADD COLUMN user_name VARCHAR(100);
ALTER TABLE orders ADD COLUMN user_city VARCHAR(100);
```

Đổi lại: khi user đổi tên, phải UPDATE cả bảng orders. Đó là trade-off bạn phải chủ động quyết định.

**Quy tắc:** Normalize trước. Chỉ denormalize khi có vấn đề performance thực tế và đã đo đạc.

---

# CHƯƠNG 7: SQL vs NoSQL — Chọn Đúng Công Cụ

## 7.1 Tại sao NoSQL xuất hiện?

SQL database (MySQL, PostgreSQL...) rất tốt, nhưng đến một quy mô nhất định gặp giới hạn:

- **Khó scale ngang (horizontal)**: Thêm máy chủ vào không đơn giản như với NoSQL
- **Schema cứng nhắc**: Muốn thêm cột mới → migration, đôi khi lock table hàng giờ
- **Không phù hợp mọi kiểu dữ liệu**: Graph relationships, time-series, document...

NoSQL sinh ra để giải quyết những giới hạn này — **đánh đổi ACID nghiêm ngặt lấy flexibility và scale**.

## 7.2 Các loại NoSQL và khi nào dùng

**Document Store (MongoDB, Firestore)**

```json
// Lưu "document" — giống JSON, flexible schema
{
  "_id": "user_123",
  "name": "Nguyen An",
  "address": {
    "city": "HCM",
    "district": "Q1"
  },
  "orders": [
    {"id": 1, "amount": 150000},
    {"id": 2, "amount": 280000}
  ]
}
```

Dùng khi: Data có cấu trúc lồng nhau phức tạp, schema thay đổi thường xuyên, cần đọc toàn bộ "document" cùng lúc.

Không dùng khi: Cần JOIN nhiều loại data, cần ACID nghiêm ngặt.

**Key-Value Store (Redis)**

```
"session:abc123"  →  "{user_id: 1, token: '...', expires: 3600}"
"user:1:profile"  →  "{name: 'An', city: 'HCM'}"
"product:456"     →  "{name: 'Laptop', price: 25000000}"
```

Dùng khi: Cache, session, leaderboard, rate limiting — thứ gì cần đọc/ghi **cực nhanh** với key biết trước.

Không dùng khi: Cần tìm kiếm theo giá trị, cần query phức tạp.

**Wide Column Store (Cassandra)**

```
Partition key: sensor_id
Clustering key: timestamp

sensor_001 | 2024-01-01 00:00 | temp=25.3 | humidity=60
sensor_001 | 2024-01-01 00:01 | temp=25.4 | humidity=61
sensor_001 | 2024-01-01 00:02 | temp=25.2 | humidity=59
```

Dùng khi: Time-series data, IoT, log — **ghi rất nhiều, đọc theo time range**.

Không dùng khi: Cần JOIN, cần query linh hoạt theo nhiều cột.

**Graph Database (Neo4j)**

```
(Nguyen An) --[BẠN BÈ]--> (Tran Binh)
(Tran Binh) --[BẠN BÈ]--> (Le Chau)
(Nguyen An) --[THÍCH]--> (Laptop Dell)
```

Dùng khi: Mạng xã hội, recommendation engine, fraud detection — thứ gì mà **mối quan hệ giữa các entity** là trọng tâm.

Không dùng khi: Data phẳng, không có nhiều relationships.

## 7.3 Bảng So Sánh Thực Tế

| Bài toán | Dùng gì | Lý do |
|---|---|---|
| User account, đơn hàng | PostgreSQL / MySQL | Cần ACID, relationships |
| Cache API response | Redis | Nhanh, TTL tự động |
| Sản phẩm catalog linh hoạt | MongoDB | Schema thay đổi thường |
| Log hệ thống | Cassandra / ClickHouse | Write nhiều, đọc theo time |
| "Người bạn biết" | Neo4j | Graph traversal |
| Session người dùng | Redis | Key-value, cần nhanh |
| Analytics, báo cáo | ClickHouse / BigQuery | Column-oriented, aggregate nhanh |

---

# CHƯƠNG 8: Scale — Khi Một Máy Không Đủ

## 8.1 Vertical Scaling — Nâng Cấp Máy Chủ

Đơn giản nhất: mua máy mạnh hơn (thêm RAM, CPU nhanh hơn, SSD nhanh hơn).

```
Trước: 8 core, 32GB RAM  →  Sau: 32 core, 128GB RAM
```

**Ưu:** Không cần thay đổi code hay kiến trúc.  
**Nhược:** Có giới hạn vật lý. Máy mạnh nhất cũng chỉ đến một mức. Giá tăng theo hàm mũ. Single point of failure.

## 8.2 Read Replica — Chia Tải Đọc

90% traffic của hầu hết ứng dụng là **đọc**, không phải ghi. Giải pháp: có nhiều bản sao của database, chỉ dùng để đọc.

```
                ┌────────────────────┐
  WRITE ──────► │   Primary DB       │
                └─────────┬──────────┘
                           │ replication (liên tục sync)
              ┌────────────┼────────────┐
              ▼            ▼            ▼
         [Replica 1]  [Replica 2]  [Replica 3]
              ▲            ▲            ▲
  READ ───────┴────────────┴────────────┘
         (load balancer phân phối đều)
```

Ứng dụng ghi vào Primary, đọc từ bất kỳ Replica nào.

**Lưu ý quan trọng — Replication Lag:**  
Replication không tức thì. Sau khi ghi vào Primary, mất vài millisecond đến vài giây dữ liệu mới xuất hiện ở Replica.

```
Tình huống:
  User cập nhật avatar → ghi vào Primary
  User reload trang → đọc từ Replica (chưa có avatar mới)
  → User thấy avatar cũ → confusing

Giải pháp: Sau khi write, đọc từ Primary trong vài giây.
           Hoặc route cùng user về cùng replica trong cùng session.
```

## 8.3 Sharding — Chia Nhỏ Database

Khi Primary đã quá tải cả write lẫn read, cần **sharding**: chia data ra nhiều database server.

```
Không sharding:          Sau khi sharding (theo user_id):
┌──────────────┐         
│  1 DB chứa   │         DB Shard 0: users id 1→1,000,000
│  tất cả data │   →     DB Shard 1: users id 1,000,001→2,000,000
│              │         DB Shard 2: users id 2,000,001→3,000,000
└──────────────┘         
```

Mỗi shard là một database độc lập. Ứng dụng tự tính "user này thuộc shard nào" và kết nối đúng shard.

**Cách tính shard (Hash-based):**

```python
def get_shard(user_id):
    return hash(user_id) % total_shards
    # user_id=1 → hash=7823 → 7823 % 3 = 2 → Shard 2
    # user_id=2 → hash=1234 → 1234 % 3 = 1 → Shard 1
```

**Vấn đề của sharding:**

```sql
-- Query dễ — chỉ cần 1 shard:
SELECT * FROM users WHERE id = 123;
-- Biết ngay shard nào → query 1 DB ✓

-- Query khó — phải hỏi tất cả shards:
SELECT COUNT(*) FROM users WHERE city = 'HCM';
-- city không phải shard key → không biết shard nào có → hỏi tất cả → gộp lại
-- Tốn gấp N lần so với không sharding ✗
```

**Chọn shard key đúng** là quyết định quan trọng nhất khi sharding:
- Nên là cột thường dùng nhất trong WHERE
- Nên phân phối đều (tránh một shard nhận 80% data)
- Không nên là timestamp (mọi write đổ vào shard mới nhất)

## 8.4 Connection Pooling — Vấn Đề Ẩn

Mỗi kết nối đến database tốn ~5MB RAM và thời gian để tạo (~50ms). Ứng dụng có 1000 request đồng thời → cần 1000 kết nối → 5GB RAM chỉ để giữ kết nối.

Giải pháp: **Connection Pool** — tạo sẵn N kết nối, tái sử dụng.

```
Không có pool:
  Request 1 → tạo kết nối (50ms) → dùng → đóng
  Request 2 → tạo kết nối (50ms) → dùng → đóng

Có pool:
  Request 1 → lấy kết nối sẵn có (~0ms) → dùng → trả lại pool
  Request 2 → lấy kết nối sẵn có (~0ms) → dùng → trả lại pool
```

PgBouncer (PostgreSQL), HikariCP (Java), là những connection pool phổ biến. Đây là thứ **bắt buộc phải có** trong production.

---

# CHƯƠNG 9: Những Lỗi Phổ Biến Và Cách Tránh

## N+1 Query — Kẻ Giết Hiệu Năng Thầm Lặng

```python
# Lấy 100 users — 1 query
users = db.query("SELECT * FROM users LIMIT 100")

# Với MỖI user, lấy orders của họ — 100 queries!
for user in users:
    user.orders = db.query(
        "SELECT * FROM orders WHERE user_id = %s", user.id
    )

# Tổng: 101 queries → N+1 problem
# Với 100 users và latency 1ms/query → 101ms
# Với 10,000 users → 10,001ms → 10 giây → không thể dùng được
```

**Giải pháp:**

```sql
-- 1 query duy nhất với JOIN
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
LIMIT 100;

-- Hoặc 2 queries (thay vì 101):
-- Query 1: lấy 100 users
SELECT * FROM users LIMIT 100;

-- Query 2: lấy tất cả orders của 100 users đó cùng lúc
SELECT * FROM orders
WHERE user_id IN (1, 2, 3, ..., 100);
```

## Không Dùng Index Vì Viết Query Sai

```sql
-- ❌ Dùng function trên cột → index bị bỏ qua
WHERE YEAR(created_at) = 2024
WHERE LOWER(email) = 'an@gmail.com'
WHERE balance + 100 > 1000

-- ✅ Viết lại để index được dùng
WHERE created_at >= '2024-01-01' AND created_at < '2025-01-01'
WHERE email = 'an@gmail.com'  -- lưu email đã lowercase từ đầu
WHERE balance > 900
```

## SELECT * Khi Không Cần

```sql
-- ❌ Kéo tất cả 50 cột, chỉ dùng 3
SELECT * FROM users WHERE id = 1;

-- ✅ Chỉ lấy cần thiết → ít data truyền qua mạng, có thể dùng covering index
SELECT id, name, email FROM users WHERE id = 1;
```

## Không Dùng Transaction Khi Cần

```python
# ❌ Nếu bước 2 lỗi, bước 1 đã xảy ra rồi → dữ liệu không nhất quán
db.execute("UPDATE accounts SET balance = balance - 500000 WHERE id = 'A'")
db.execute("UPDATE accounts SET balance = balance + 500000 WHERE id = 'B'")

# ✅ Wrap trong transaction
with db.transaction():
    db.execute("UPDATE accounts SET balance = balance - 500000 WHERE id = 'A'")
    db.execute("UPDATE accounts SET balance = balance + 500000 WHERE id = 'B'")
# Nếu bất kỳ bước nào lỗi → cả 2 đều rollback tự động
```

---

# CHƯƠNG 10: Deadlock — Khi Hai Transaction Chờ Nhau Mãi Mãi

## 10.1 Deadlock là gì

Chương 4 đề cập đến locking như một cơ chế của Isolation. Nhưng locking sinh ra một vấn đề nguy hiểm: **deadlock**.

Hình dung:

```
Transaction A đang giữ lock trên row "Tài khoản X"
Transaction B đang giữ lock trên row "Tài khoản Y"

Transaction A cần lock "Tài khoản Y" → chờ B nhả
Transaction B cần lock "Tài khoản X" → chờ A nhả

→ Cả hai chờ nhau mãi mãi. Không ai tiến lên được.
```

Trong thực tế:

```sql
-- Transaction A (chuyển tiền từ X sang Y)
BEGIN;
SELECT * FROM accounts WHERE id = 'X' FOR UPDATE;  -- lock X thành công
-- ... xử lý một lúc ...
SELECT * FROM accounts WHERE id = 'Y' FOR UPDATE;  -- chờ B nhả lock Y ← bị chặn

-- Transaction B (chạy đồng thời, chuyển tiền từ Y sang X)
BEGIN;
SELECT * FROM accounts WHERE id = 'Y' FOR UPDATE;  -- lock Y thành công
-- ... xử lý một lúc ...
SELECT * FROM accounts WHERE id = 'X' FOR UPDATE;  -- chờ A nhả lock X ← bị chặn

-- → DEADLOCK: A chờ B, B chờ A
```

## 10.2 Database Phát Hiện và Xử Lý Deadlock Thế Nào

Database không thể ngồi chờ mãi. PostgreSQL chạy một **deadlock detector** định kỳ (mặc định mỗi 1 giây). Khi phát hiện vòng lặp chờ:

```
1. Vẽ "wait-for graph" — đồ thị ai đang chờ ai
2. Phát hiện cycle trong đồ thị → đó là deadlock
3. Chọn một transaction làm "nạn nhân" (thường là transaction tốn ít chi phí hơn để rollback)
4. Rollback transaction nạn nhân
5. Trả lỗi về client: "ERROR: deadlock detected"
6. Transaction còn lại tiếp tục chạy bình thường
```

```
Kết quả ví dụ trên:
  Transaction A nhận lỗi "deadlock detected" → rollback toàn bộ
  Transaction B được tiếp tục → hoàn thành thành công
  Application phải bắt lỗi này và thử lại Transaction A từ đầu.
```

Application cần xử lý lỗi này đúng cách:

```python
import psycopg2
from psycopg2 import errors
import time

def transfer_money(from_id, to_id, amount, max_retries=3):
    for attempt in range(max_retries):
        try:
            with db.transaction():
                db.execute("UPDATE accounts SET balance = balance - %s WHERE id = %s",
                           (amount, from_id))
                db.execute("UPDATE accounts SET balance = balance + %s WHERE id = %s",
                           (amount, to_id))
            return True  # thành công
        except errors.DeadlockDetected:
            if attempt == max_retries - 1:
                raise  # hết retry, raise lên
            time.sleep(0.1 * (attempt + 1))  # chờ rồi thử lại
    return False
```

## 10.3 Cách Tránh Deadlock

**Nguyên tắc vàng: Luôn lock theo cùng một thứ tự.**

```python
# ❌ Dễ deadlock — thứ tự lock khác nhau
def transfer_A_to_B():
    lock("account_X")  # lock X trước
    lock("account_Y")  # rồi lock Y

def transfer_B_to_A():
    lock("account_Y")  # lock Y trước ← ngược chiều!
    lock("account_X")  # rồi lock X

# ✅ Không deadlock — luôn lock theo id tăng dần
def transfer(from_id, to_id, amount):
    first_id  = min(from_id, to_id)
    second_id = max(from_id, to_id)
    # Dù transfer theo chiều nào, thứ tự lock luôn: id nhỏ → id lớn
    db.execute("SELECT * FROM accounts WHERE id = %s FOR UPDATE", (first_id,))
    db.execute("SELECT * FROM accounts WHERE id = %s FOR UPDATE", (second_id,))
    # ... thực hiện transfer ...
```

**Kỹ thuật khác:**

```sql
-- 1. Lock nhiều row cùng lúc ngay từ đầu (thay vì từng cái một)
SELECT * FROM accounts
WHERE id IN ('X', 'Y')
ORDER BY id          -- thứ tự nhất quán đảm bảo không deadlock
FOR UPDATE;

-- 2. NOWAIT — thất bại ngay nếu không lock được (không chờ → không deadlock)
SELECT * FROM accounts WHERE id = 'X' FOR UPDATE NOWAIT;
-- → ERROR: could not obtain lock on row (thay vì chờ mãi)
-- Application retry sau một khoảng thời gian ngắn

-- 3. SKIP LOCKED — bỏ qua row đang bị lock (dùng cho job queue)
SELECT * FROM tasks
WHERE status = 'pending'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 1;
-- → Worker 1 lấy task đầu tiên chưa bị ai giữ
-- → Worker 2 chạy đồng thời lấy task tiếp theo, không chờ Worker 1
-- → Không bao giờ deadlock
```

## 10.4 Lock Timeout — Tránh Chờ Vô Hạn

Ngay cả khi không có deadlock, chờ lock quá lâu cũng là vấn đề:

```sql
-- Đặt thời gian tối đa chờ lock (milliseconds)
SET lock_timeout = '5000';   -- chờ tối đa 5 giây
SET lock_timeout = '500';    -- chờ tối đa 0.5 giây (strict hơn)

-- Nếu không lock được trong thời gian đó:
-- ERROR: canceling statement due to lock timeout

-- Đặt statement timeout — hủy query nếu chạy quá lâu
SET statement_timeout = '30000';  -- tối đa 30 giây
```

---

# CHƯƠNG 11: MVCC Cleanup — Vấn Đề Ít Ai Biết Của PostgreSQL

Chương 4 giải thích MVCC lưu nhiều phiên bản của row. Nhưng có một câu hỏi chưa được trả lời: **những phiên bản cũ đó đi đâu?**

## 11.1 Dead Tuples — Xác Chết Trong Database

Khi bạn UPDATE một row trong PostgreSQL:

```sql
UPDATE users SET city = 'HN' WHERE id = 1;
```

PostgreSQL **không xóa row cũ**. Nó:
1. Đánh dấu row cũ là "đã chết" (`xmax` = transaction hiện tại)
2. Tạo row mới với giá trị mới

```
Trước UPDATE:
┌──────┬──────┬──────┬───────┐
│ xmin │ xmax │  id  │ city  │
│  100 │    0 │   1  │ HCM   │  ← đang sống (xmax=0 nghĩa là chưa bị xóa)
└──────┴──────┴──────┴───────┘

Sau UPDATE bởi transaction 500:
┌──────┬──────┬──────┬───────┐
│  100 │  500 │   1  │ HCM   │  ← dead tuple — xác chết, chiếm chỗ
│  500 │    0 │   1  │ HN    │  ← row mới, đang sống
└──────┴──────┴──────┴───────┘
```

Row cũ vẫn nằm nguyên trong file, chiếm đúng chỗ đó. Tích lũy lâu ngày:

```
1 triệu UPDATE/ngày × 365 ngày
= 365 triệu dead tuples tích lũy
= Table phình to gấp nhiều lần so với data thực
= Query ngày càng chậm vì phải đọc qua dead tuples
= Index scan tốn hơn vì index cũng trỏ đến dead tuples
```

## 11.2 VACUUM — Người Dọn Dẹp

PostgreSQL có tiến trình chạy ngầm tên là **autovacuum**, nhiệm vụ dọn dẹp dead tuples tự động.

```
VACUUM làm những gì:

1. Quét qua toàn bộ table, xác định dead tuples
   (dead tuple = xmax != 0 VÀ transaction đó đã commit VÀ
    không còn transaction nào cần đọc phiên bản cũ đó nữa)

2. Đánh dấu không gian của dead tuples là "free space"
   → PostgreSQL có thể tái sử dụng cho INSERT/UPDATE mới

3. Cập nhật Free Space Map (FSM)
   → Database biết chỗ nào còn trống để INSERT nhanh

4. Cập nhật Visibility Map (VM)
   → Giúp Index Only Scan nhanh hơn (biết page nào toàn row "sạch")

5. Cập nhật statistics (pg_statistic)
   → Query optimizer dùng để ước tính số row, chọn execution plan đúng

Lưu ý quan trọng:
  VACUUM thường KHÔNG trả không gian về OS.
  Nó chỉ cho phép PostgreSQL tái sử dụng không gian đó.
  
  VACUUM FULL mới thực sự co file lại và trả về OS
  — nhưng cần LOCK EXCLUSIVE (không ai đọc/ghi được trong lúc đó).
  Dùng cẩn thận trong production.
```

Bạn có thể chạy thủ công hoặc kiểm tra tình trạng:

```sql
-- Chạy vacuum thủ công
VACUUM users;                  -- dọn dead tuples, không lock
VACUUM ANALYZE users;          -- dọn + cập nhật statistics
VACUUM FULL users;             -- co file lại (⚠ lock table — dùng cẩn thận)
VACUUM VERBOSE users;          -- xem chi tiết đang làm gì

-- Kiểm tra table nào cần vacuum nhất
SELECT
    relname                                          AS table_name,
    n_live_tup                                       AS live_rows,
    n_dead_tup                                       AS dead_rows,
    round(n_dead_tup::numeric / nullif(n_live_tup + n_dead_tup, 0) * 100, 1) AS dead_pct,
    last_autovacuum,
    last_autoanalyze
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;
```

## 11.3 Autovacuum — Cấu Hình Đúng Cách

Autovacuum tự động chạy khi table đủ "bẩn". Ngưỡng mặc định:

```
Chạy VACUUM khi: dead_tuples > 50 + (0.2 × tổng_số_row)
Chạy ANALYZE khi: changed_rows > 50 + (0.1 × tổng_số_row)

Ví dụ: bảng users có 1 triệu row
  VACUUM khi: 200,050 dead tuples
  ANALYZE khi: 100,050 rows thay đổi
```

Với table lớn (10 triệu row), ngưỡng này quá cao — phải tích lũy 2 triệu dead tuples mới vacuum. Điều chỉnh per-table:

```sql
ALTER TABLE orders SET (
    autovacuum_vacuum_scale_factor = 0.01,   -- vacuum khi 1% rows là dead (thay vì 20%)
    autovacuum_analyze_scale_factor = 0.005, -- analyze khi 0.5% rows thay đổi
    autovacuum_vacuum_cost_delay = 2         -- ms delay giữa các bước (giảm để vacuum nhanh hơn)
);
```

## 11.4 Transaction ID Wraparound — Thảm Họa Âm Thầm

MVCC dùng **Transaction ID (XID)** — một số 32-bit. Tức là tối đa ~2.1 tỷ transaction. Sau đó bộ đếm quay về 0 (wraparound).

Vấn đề: nếu XID cũ được tái sử dụng, database không còn biết row nào cũ hơn row nào — có thể coi row mới là "trong tương lai" → không đọc được. Đây là lỗi **nghiêm trọng**, dữ liệu có thể biến mất.

```
PostgreSQL cảnh báo khi còn 40 triệu XID:
  WARNING: database "myapp" must be vacuumed within 177009986 transactions
  HINT: To avoid a database shutdown, execute a database-wide VACUUM in "myapp".

Nếu không vacuum kịp thời, đến 3 triệu XID cuối:
  ERROR: database is not accepting commands to avoid wraparound data loss
  → PostgreSQL dừng nhận kết nối mới, bắt buộc chạy VACUUM FREEZE trước
```

VACUUM FREEZE là giải pháp: đánh dấu các row đủ già là "frozen" (không còn phụ thuộc XID nữa), thoát khỏi vòng wraparound.

```sql
-- Kiểm tra "tuổi" của database (khoảng cách đến wraparound)
SELECT datname, age(datfrozenxid) AS xid_age,
       2147483648 - age(datfrozenxid) AS xids_remaining
FROM pg_database
ORDER BY xid_age DESC;

-- Nếu xid_age > 1.5 tỷ: cần chú ý
-- Nếu xid_age > 1.9 tỷ: khẩn cấp

-- Chạy VACUUM FREEZE thủ công
VACUUM FREEZE users;             -- freeze table cụ thể
VACUUM FREEZE;                   -- freeze toàn bộ database (lâu)
```

**Giải pháp:** Đừng bao giờ tắt autovacuum. Monitor `age(datfrozenxid)` bằng Prometheus/Grafana. Set cảnh báo khi > 1 tỷ.

---

# CHƯƠNG 12: Execution Plan Chi Tiết — Đọc EXPLAIN Như Chuyên Gia

Chương 5 giới thiệu EXPLAIN nhưng chưa đi sâu. Đây là phần đọc execution plan thực sự.

## 12.1 Các Loại Scan

Khi database cần đọc dữ liệu từ table, có nhiều chiến lược khác nhau — mỗi cái tốt trong hoàn cảnh khác nhau.

**Sequential Scan (Seq Scan)**

```
Seq Scan on users  (cost=0.00..1842.00 rows=100000 width=50)
```

Đọc toàn bộ table từ đầu đến cuối theo thứ tự vật lý. Database dùng khi:
- Không có index phù hợp
- Query lấy > ~5–10% số row (index overhead lớn hơn lợi ích)
- Table nhỏ (đọc thẳng nhanh hơn tra index)

**Index Scan**

```
Index Scan using idx_email on users  (cost=0.43..8.45 rows=1 width=50)
  Index Cond: (email = 'an@gmail.com')
```

Dùng B-tree index để tìm vị trí row, rồi nhảy đến **heap** (file chứa data thực) để lấy toàn bộ cột. Tốt khi lấy ít row. Nhược điểm: mỗi row cần 2 lần đọc — đọc index rồi đọc heap. Nếu row rải rác nhiều page → nhiều lần đọc heap → chậm.

**Index Only Scan**

```
Index Only Scan using idx_email_name on users  (cost=0.43..4.45 rows=1 width=20)
  Index Cond: (email = 'an@gmail.com')
  Heap Fetches: 0
```

Tất cả data cần thiết đã có trong index — không cần đọc heap. Cực kỳ nhanh. Xảy ra khi SELECT chỉ lấy đúng các cột có trong index, gọi là **covering index**:

```sql
-- Tạo covering index — bao gồm cột cần SELECT trong INCLUDE
CREATE INDEX idx_covering ON users(email) INCLUDE (name, age);

-- Query này sẽ dùng Index Only Scan:
SELECT name, age FROM users WHERE email = 'an@gmail.com';
-- Tất cả 3 cột (email, name, age) đều có trong index → không cần đụng đến heap
```

**Bitmap Scan**

```
Bitmap Heap Scan on users  (cost=125.00..892.00 rows=5000 width=50)
  Recheck Cond: (city = 'HCM')
  →  Bitmap Index Scan on idx_city  (cost=0.00..124.00 rows=5000)
       Index Cond: (city = 'HCM')
```

Trường hợp trung gian — cần lấy lượng row vừa phải. Database làm 2 bước:

1. **Bitmap Index Scan**: Quét index, tạo một bitmap trong RAM — đánh dấu page nào có row thỏa điều kiện
2. **Bitmap Heap Scan**: Sắp xếp các page đã đánh dấu theo thứ tự vật lý, đọc từng page một lần duy nhất

Hiệu quả hơn Index Scan khi row rải rác nhiều page, vì tránh đọc cùng page nhiều lần.

Bitmap của nhiều index có thể kết hợp với AND/OR:

```sql
-- Dùng 2 index, kết hợp bitmap
SELECT * FROM users WHERE city = 'HCM' AND age = 25;

-- EXPLAIN có thể hiện:
BitmapAnd
  →  Bitmap Index Scan on idx_city   (city = 'HCM')
  →  Bitmap Index Scan on idx_age    (age = 25)
-- Kết quả: chỉ page có CẢ HAI điều kiện
```

## 12.2 Các Loại Join

**Hash Join**

```
Hash Join  (cost=450.00..1150.00 rows=20000 width=80)
  Hash Cond: (o.user_id = u.id)
  →  Seq Scan on orders           ← outer (table lớn)
  →  Hash                         ← build hash table từ table nhỏ
       →  Index Scan on users
```

Cách hoạt động:
1. Đọc toàn bộ table nhỏ hơn (users), tạo **hash table** trong RAM, key = join column
2. Scan table lớn hơn (orders), với mỗi row tra hash table → O(1) mỗi lần tra
3. Tốt cho join hai table lớn, không cần index trên join column

Nhược điểm: cần RAM để chứa hash table. Nếu hash table không vừa RAM → "hash spill" xuống disk → chậm hẳn.

**Nested Loop Join**

```
Nested Loop  (cost=0.43..1200.00 rows=100 width=80)
  →  Index Scan on users WHERE city='HCM'     ← outer: 100 rows
  →  Index Scan on orders WHERE user_id=$1    ← inner: chạy 100 lần
```

Với mỗi row ở outer (users ở HCM: 100 rows), tìm row tương ứng ở inner (orders của user đó). Tốt khi outer ít row và inner có index. Tệ khi outer nhiều row — O(n × m).

**Merge Join**

```
Merge Join  (cost=25000.00..45000.00 rows=50000 width=80)
  Merge Cond: (u.id = o.user_id)
  →  Sort on users.id        ← hoặc Index Scan nếu đã có index
  →  Sort on orders.user_id  ← hoặc Index Scan nếu đã có index
```

Cả hai table được sort theo join column, sau đó merge song song như merge sort. Tốt khi:
- Cả hai table đã có index trên join column (không cần sort)
- Kết quả rất lớn (hash table không vừa RAM)

## 12.3 Đọc Cost và Rows

```
Hash Join  (cost=450.00..1150.00 rows=20000 width=48)
            ↑             ↑         ↑            ↑
     startup cost   total cost   est. rows   bytes/row (avg)

(actual time=12.345..38.901 rows=19500 loops=1)
              ↑               ↑         ↑
       thời gian row đầu  tổng thời gian  thực tế
```

Đơn vị **cost** không phải millisecond — là đơn vị tương đối:
- 1 sequential page read = 1.0
- 1 random page read = 4.0 (mặc định, SSD có thể điều chỉnh xuống 1.1)
- 1 CPU operation = 0.01

**Dấu hiệu cần chú ý trong execution plan:**

```
1. Seq Scan trên table lớn (rows > 100,000)
   → Thiếu index? Query có dùng function trên cột không?

2. Ước tính rows sai nhiều:
   est. rows=1000  nhưng  actual rows=500000
   → Statistics lỗi thời → chạy: ANALYZE table_name;
   → Hoặc data phân phối không đều (ví dụ: 90% user ở HCM)
     → dùng extended statistics hoặc partial index

3. Hash Batches > 1 trong Hash Join:
   Hash Batches: 8  (initial: 1)
   → Hash table không vừa RAM → spill xuống disk
   → Tăng work_mem: SET work_mem = '256MB';

4. Sort Method: external merge  (trong Sort node)
   → Sort không vừa RAM → chậm đáng kể
   → Tăng work_mem

5. Loops rất lớn trong Nested Loop:
   (actual rows=10 loops=50000)
   → Chạy inner query 50,000 lần → cần index tốt hơn hoặc đổi join type
```

## 12.4 Parallel Query

PostgreSQL có thể chia query thành nhiều worker:

```
Gather  (cost=1000.00..500000.00 rows=1000000 width=50)
  Workers Planned: 4
  Workers Launched: 4
  →  Parallel Seq Scan on big_table
       Worker 0: actual rows=250000
       Worker 1: actual rows=249800
       Worker 2: actual rows=250100
       Worker 3: actual rows=250100
```

Tự động bật cho query scan table lớn. Điều chỉnh:

```sql
-- Tăng workers cho query nặng:
SET max_parallel_workers_per_gather = 8;

-- Cho một query cụ thể:
SET LOCAL max_parallel_workers_per_gather = 4;
SELECT COUNT(*), AVG(amount) FROM orders WHERE created_at > '2024-01-01';

-- Force parallel (để test):
SET parallel_setup_cost = 0;
SET parallel_tuple_cost = 0;

-- Tắt parallel (nếu gây vấn đề):
SET max_parallel_workers_per_gather = 0;
```

## 12.5 Cập Nhật Statistics

Optimizer dùng statistics để ước tính số row và chọn execution plan. Statistics cũ → plan sai → query chậm.

```sql
-- Cập nhật statistics một table:
ANALYZE users;

-- Cập nhật toàn bộ database:
ANALYZE;

-- Xem statistics hiện tại của một cột:
SELECT attname, n_distinct, correlation
FROM pg_stats
WHERE tablename = 'users';

-- n_distinct: số giá trị phân biệt ước tính
--   > 0: con số tuyệt đối (ví dụ: 5000 emails phân biệt)
--   < 0: tỉ lệ (ví dụ: -0.9 nghĩa là 90% giá trị phân biệt)
-- correlation: mức độ thứ tự vật lý trùng với thứ tự logic
--   1.0 = hoàn toàn theo thứ tự → index scan rất hiệu quả
--   0.0 = ngẫu nhiên → random I/O nhiều hơn

-- Extended statistics cho cột có correlation:
CREATE STATISTICS stat_city_age ON city, age FROM users;
ANALYZE users;
-- Giờ optimizer biết: nếu city='HCM' thì age phân phối thế nào
```

---

# CHƯƠNG 13: Partitioning — Chia Bảng Lớn Thành Nhiều Phần

## 13.1 Partitioning Khác Sharding Ở Chỗ Nào

Chương 8 nói về Sharding — chia data ra nhiều **server**. Partitioning là khái niệm khác: chia một table lớn thành nhiều phần nhỏ hơn **trên cùng một server**, trong suốt với application.

```
Sharding:       [Server 1: users 1-1M]   [Server 2: users 1M-2M]   [Server 3: users 2M-3M]
                  ↑                         ↑                         ↑
                  App phải biết server nào chứa data nào

Partitioning:   [Server duy nhất]
                  └── table "orders" (logical)
                        ├── orders_2022 (physical file)
                        ├── orders_2023 (physical file)
                        └── orders_2024 (physical file)
                  ↑
                  App chỉ thấy 1 table "orders" — không biết bên trong có partition
```

## 13.2 Range Partitioning — Chia Theo Khoảng Giá Trị

Phổ biến nhất với time-series data:

```sql
-- Tạo bảng cha (không chứa data trực tiếp)
CREATE TABLE orders (
    id          BIGSERIAL,
    created_at  TIMESTAMP NOT NULL,
    user_id     INT,
    amount      DECIMAL,
    status      VARCHAR(20)
) PARTITION BY RANGE (created_at);

-- Tạo các partition con (chứa data thực sự)
CREATE TABLE orders_2023_q1
    PARTITION OF orders
    FOR VALUES FROM ('2023-01-01') TO ('2023-04-01');

CREATE TABLE orders_2023_q2
    PARTITION OF orders
    FOR VALUES FROM ('2023-04-01') TO ('2023-07-01');

CREATE TABLE orders_2023_q3
    PARTITION OF orders
    FOR VALUES FROM ('2023-07-01') TO ('2023-10-01');

CREATE TABLE orders_2023_q4
    PARTITION OF orders
    FOR VALUES FROM ('2023-10-01') TO ('2024-01-01');

CREATE TABLE orders_2024_q1
    PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2024-04-01');

-- Partition mặc định cho data không khớp range nào (tùy chọn)
CREATE TABLE orders_default
    PARTITION OF orders DEFAULT;
```

Bây giờ khi INSERT, PostgreSQL tự routing đến đúng partition:

```sql
INSERT INTO orders (created_at, user_id, amount, status)
VALUES ('2024-02-15', 123, 500000, 'confirmed');
-- → Tự động vào orders_2024_q1, không cần app làm gì
```

Khi SELECT, database biết ngay chỉ cần đọc partition phù hợp — **Partition Pruning**:

```sql
SELECT * FROM orders
WHERE created_at BETWEEN '2024-06-01' AND '2024-06-30';

-- EXPLAIN hiện:
Append
  →  Seq Scan on orders_2024_q2   ← chỉ đọc partition này
       Filter: (created_at BETWEEN '2024-06-01' AND '2024-06-30')
-- orders_2023_*, orders_2024_q1, orders_2024_q3, q4 bị bỏ qua hoàn toàn ✓
```

## 13.3 List Partitioning — Chia Theo Danh Sách Giá Trị

```sql
CREATE TABLE users (
    id      BIGSERIAL,
    region  VARCHAR(10) NOT NULL,
    name    VARCHAR(100),
    email   VARCHAR(200)
) PARTITION BY LIST (region);

CREATE TABLE users_sea   PARTITION OF users FOR VALUES IN ('VN', 'SG', 'TH', 'ID', 'MY', 'PH');
CREATE TABLE users_ea    PARTITION OF users FOR VALUES IN ('JP', 'KR', 'CN', 'TW', 'HK');
CREATE TABLE users_eu    PARTITION OF users FOR VALUES IN ('DE', 'FR', 'GB', 'NL', 'SE');
CREATE TABLE users_amer  PARTITION OF users FOR VALUES IN ('US', 'CA', 'BR', 'MX');
CREATE TABLE users_other PARTITION OF users DEFAULT;
```

Hữu ích khi query thường lọc theo một cột categorical có ít giá trị phân biệt nhưng phân chia data tự nhiên (region, country, tenant_id trong multi-tenant app).

## 13.4 Hash Partitioning — Chia Đều

Khi không có cột tự nhiên để chia theo range hay list, dùng hash để phân phối đều:

```sql
CREATE TABLE events (
    id      BIGSERIAL,
    user_id INT NOT NULL,
    type    VARCHAR(50),
    payload JSONB,
    ts      TIMESTAMP
) PARTITION BY HASH (user_id);

-- 8 partitions phân phối đều theo hash(user_id)
CREATE TABLE events_p0 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 0);
CREATE TABLE events_p1 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 1);
CREATE TABLE events_p2 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 2);
CREATE TABLE events_p3 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 3);
CREATE TABLE events_p4 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 4);
CREATE TABLE events_p5 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 5);
CREATE TABLE events_p6 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 6);
CREATE TABLE events_p7 PARTITION OF events FOR VALUES WITH (MODULUS 8, REMAINDER 7);
-- hash(user_id) % 8 = 0 → p0, = 1 → p1, v.v.
```

## 13.5 Lợi Ích Thực Tế

**1. Partition Pruning — Query chỉ đọc đúng partition cần thiết**

```sql
SELECT SUM(amount) FROM orders WHERE created_at >= '2024-01-01';
-- Chỉ scan orders_2024_q1, q2, q3, q4 thay vì toàn bộ 5 năm data.
```

**2. Xóa data cũ cực kỳ nhanh**

```sql
-- ❌ Cách cũ: DELETE hàng triệu row — chậm, tạo dead tuples, cần vacuum sau
DELETE FROM orders WHERE created_at < '2022-01-01';
-- → Có thể mất hàng giờ, I/O cao, bloat table

-- ✅ Cách mới với partitioning: DROP partition — tức thì, không để lại gì
DROP TABLE orders_2021_q1;  -- tức thì
DROP TABLE orders_2021_q2;  -- tức thì
DROP TABLE orders_2021_q3;  -- tức thì
DROP TABLE orders_2021_q4;  -- tức thì
-- → Toàn bộ data 2021 biến mất trong vài milliseconds
```

Hoặc DETACH để giữ lại partition (chỉ không còn là phần của table cha):

```sql
ALTER TABLE orders DETACH PARTITION orders_2021_q1;
-- Giờ orders_2021_q1 là table độc lập, có thể archive hoặc xóa sau
```

**3. Index nhỏ hơn, nằm gọn trong RAM**

```
Không partition: index trên 100 triệu row = 5GB → không vừa RAM → random I/O từ disk
Có partition: index trên mỗi 3 triệu row = 150MB × nhiều partitions → partition hiện tại vừa vào RAM
```

**4. Maintenance nhanh hơn và ít ảnh hưởng**

```sql
-- VACUUM chỉ chạy trên partition cần thiết
VACUUM ANALYZE orders_2024_q1;

-- Thêm constraint mới: chỉ ảnh hưởng partition mới, không lock partition cũ
ALTER TABLE orders_2024_q2 ADD CONSTRAINT check_amount CHECK (amount > 0) NOT VALID;
VALIDATE CONSTRAINT check_amount;  -- validate không lock write
```

**5. Sub-partitioning — Chia nhiều cấp**

```sql
-- Chia theo năm, rồi trong mỗi năm chia theo region
CREATE TABLE orders_2024 PARTITION OF orders
    FOR VALUES FROM ('2024-01-01') TO ('2025-01-01')
    PARTITION BY LIST (region);

CREATE TABLE orders_2024_sea PARTITION OF orders_2024 FOR VALUES IN ('VN', 'SG', 'TH');
CREATE TABLE orders_2024_eu  PARTITION OF orders_2024 FOR VALUES IN ('DE', 'FR', 'GB');
```

---

# CHƯƠNG 14: Full-Text Search — Tìm Kiếm Nội Dung Văn Bản

## 14.1 Tại Sao LIKE Không Đủ

```sql
-- ❌ Vấn đề 1: Không dùng được index (trừ prefix search: 'laptop%')
WHERE name LIKE '%laptop gaming%'
-- → Seq Scan toàn bộ table dù có index

-- ❌ Vấn đề 2: Không hiểu ngôn ngữ
-- Tìm "chạy" không ra "đang chạy", "chạy bộ", "chạy nhanh"
-- Tìm "Laptop" không ra "laptop", "LAPTOP"

-- ❌ Vấn đề 3: Không có ranking
-- Không biết kết quả nào liên quan hơn kết quả nào

-- ❌ Vấn đề 4: Không xử lý được các từ biến thể
-- "running" ≠ "run" ≠ "ran" (các dạng chia của cùng 1 từ)
```

## 14.2 Full-Text Search Trong PostgreSQL

PostgreSQL có hệ thống full-text search built-in.

**Hai kiểu dữ liệu cơ bản:**

```sql
-- tsvector: văn bản đã được xử lý (stemming, stop words, position)
SELECT to_tsvector('english', 'The quick brown fox jumps over the lazy dog');
-- → 'brown':3 'dog':9 'fox':4 'jump':5 'lazi':8 'quick':2
-- "the", "over" bị loại (stop words)
-- "jumps" → "jump", "lazy" → "lazi" (stemming)

-- tsquery: câu truy vấn
SELECT to_tsquery('english', 'laptop & gaming');
SELECT to_tsquery('english', 'laptop | notebook');
SELECT to_tsquery('english', 'laptop & !second-hand');
SELECT plainto_tsquery('english', 'laptop gaming');  -- tự động thêm &
SELECT websearch_to_tsquery('english', 'laptop gaming -used');  -- cú pháp web
```

**Tìm kiếm cơ bản:**

```sql
-- Tìm sản phẩm có "laptop" và "gaming"
SELECT * FROM products
WHERE to_tsvector('english', name || ' ' || description)
      @@ to_tsquery('english', 'laptop & gaming');

-- Ranking kết quả theo độ liên quan
SELECT name,
       ts_rank(to_tsvector('english', name || ' ' || description),
               to_tsquery('english', 'laptop & gaming')) AS rank
FROM products
WHERE to_tsvector('english', name || ' ' || description)
      @@ to_tsquery('english', 'laptop & gaming')
ORDER BY rank DESC
LIMIT 10;
```

**Tối ưu với index:**

Tính `to_tsvector` mỗi query vừa chậm vừa không dùng được index. Giải pháp: lưu sẵn vào một cột:

```sql
-- Thêm cột lưu tsvector được tính trước
ALTER TABLE products ADD COLUMN search_vector tsvector;

-- Tính và lưu
UPDATE products
SET search_vector = to_tsvector('english', coalesce(name,'') || ' ' || coalesce(description,''));

-- Tự động cập nhật khi data thay đổi
CREATE FUNCTION products_search_vector_trigger() RETURNS trigger AS $$
BEGIN
    NEW.search_vector := to_tsvector('english',
        coalesce(NEW.name, '') || ' ' ||
        coalesce(NEW.description, '')
    );
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER products_search_vector_update
    BEFORE INSERT OR UPDATE ON products
    FOR EACH ROW EXECUTE FUNCTION products_search_vector_trigger();

-- Tạo GIN index (tốt nhất cho full-text search)
CREATE INDEX idx_products_search ON products USING GIN (search_vector);

-- Giờ query nhanh:
SELECT name, ts_rank(search_vector, query) AS rank
FROM products, to_tsquery('english', 'laptop & gaming') query
WHERE search_vector @@ query
ORDER BY rank DESC;
```

## 14.3 GIN vs GiST Index Cho Full-Text Search

```
GIN (Generalized Inverted Index):
  + Tìm kiếm nhanh hơn
  + Phù hợp cho static hoặc ít thay đổi data
  - Tạo/update chậm hơn
  - Tốn nhiều disk hơn
  → Dùng cho production search

GiST (Generalized Search Tree):
  + Tạo/update nhanh hơn
  + Tốn ít disk hơn
  - Tìm kiếm chậm hơn một chút
  → Dùng khi data thay đổi thường xuyên
```

## 14.4 Highlight và Snippet

```sql
-- Highlight từ khóa trong kết quả
SELECT
    name,
    ts_headline('english', description,
                to_tsquery('english', 'laptop & gaming'),
                'MaxWords=50, MinWords=20, StartSel=<b>, StopSel=</b>'
               ) AS snippet
FROM products
WHERE search_vector @@ to_tsquery('english', 'laptop & gaming')
ORDER BY ts_rank(search_vector, to_tsquery('english', 'laptop & gaming')) DESC;

-- Kết quả: "...high-performance <b>gaming</b> <b>laptop</b> with RTX 4080..."
```

## 14.5 Khi Nào Cần Elasticsearch

PostgreSQL full-text search đủ tốt cho hầu hết ứng dụng. Nhưng có những trường hợp cần Elasticsearch:

```
Dùng PostgreSQL FTS khi:
  - Tìm kiếm trong ứng dụng thông thường (blog, e-commerce nhỏ-vừa)
  - Data đã trong PostgreSQL, không muốn thêm hạ tầng
  - Cần search + filter SQL kết hợp (JOIN với bảng khác)
  - < 10 triệu documents

Cân nhắc Elasticsearch khi:
  - Search là core feature (ứng dụng tìm kiếm chính)
  - > 100 triệu documents
  - Cần autocomplete, fuzzy search (typo tolerance)
  - Cần analytics trên full-text data
  - Cần distributed search across nhiều node
  - Cần real-time indexing cực kỳ nhanh
```

---

# CHƯƠNG 15: Backup và Recovery — Khi Mọi Thứ Đi Sai

## 15.1 Các Loại Sự Cố Và Cách Phục Hồi

```
Loại 1: Transaction lỗi (thường xuyên)
  Nguyên nhân: constraint violation, deadlock, application bug
  Phục hồi: Transaction tự rollback — ACID đảm bảo
  Thời gian: Tức thì, tự động

Loại 2: Crash database server (thỉnh thoảng)
  Nguyên nhân: mất điện, OOM killer, hardware fault
  Phục hồi: WAL recovery khi khởi động lại
  Thời gian: Vài giây đến vài phút

Loại 3: Lỗi application (nguy hiểm)
  Nguyên nhân: DELETE/UPDATE nhầm, bug code
  Phục hồi: Point-in-time recovery từ backup
  Thời gian: Phụ thuộc vào kích thước database và khi nào phát hiện

Loại 4: Disk failure / datacenter outage (hiếm nhưng thảm họa)
  Nguyên nhân: hardware chết, thiên tai
  Phục hồi: Restore từ off-site backup hoặc failover sang replica
  Thời gian: Phút đến giờ
```

## 15.2 Backup Types

**Logical Backup (pg_dump)**

```bash
# Backup một database thành SQL file
pg_dump myapp > myapp_2024_01_15.sql

# Backup dạng custom format (nhanh hơn, có thể restore một phần)
pg_dump -Fc myapp > myapp_2024_01_15.dump

# Restore
psql myapp < myapp_2024_01_15.sql
pg_restore -d myapp myapp_2024_01_15.dump

# Backup chỉ schema (không có data)
pg_dump --schema-only myapp > schema.sql

# Backup chỉ một table
pg_dump -t users myapp > users_backup.sql
```

Ưu điểm: đơn giản, portable, có thể restore một phần.  
Nhược điểm: chậm với database lớn, không hỗ trợ point-in-time recovery.

**Physical Backup (pg_basebackup + WAL archiving)**

```bash
# Tạo base backup (copy toàn bộ data directory)
pg_basebackup -h localhost -U replicator -D /backup/base -Ft -z -P

# Cấu hình WAL archiving trong postgresql.conf:
# wal_level = replica
# archive_mode = on
# archive_command = 'cp %p /wal_archive/%f'
# → Mỗi WAL segment được lưu vào /wal_archive/

# Kết hợp base backup + WAL archive = có thể khôi phục về bất kỳ thời điểm nào
# (Point-in-Time Recovery — PITR)
```

## 15.3 Point-in-Time Recovery (PITR)

Kịch bản: developer chạy nhầm `DELETE FROM orders;` lúc 14:35.

```bash
# Giả sử có:
# - Base backup từ 02:00 sáng nay
# - WAL archive liên tục từ 02:00 đến 14:35

# Bước 1: Dừng PostgreSQL
systemctl stop postgresql

# Bước 2: Restore base backup
rm -rf /var/lib/postgresql/data/*
tar -xzf /backup/base.tar.gz -C /var/lib/postgresql/data/

# Bước 3: Tạo file recovery.conf (hoặc trong postgresql.conf tùy version)
cat > /var/lib/postgresql/data/recovery.signal << EOF
EOF

cat >> /var/lib/postgresql/data/postgresql.conf << EOF
restore_command = 'cp /wal_archive/%f %p'
recovery_target_time = '2024-01-15 14:34:59'  # 1 giây trước khi DELETE
recovery_target_action = 'promote'
EOF

# Bước 4: Khởi động PostgreSQL
# → PostgreSQL tự động apply WAL từ 02:00 đến 14:34:59
# → Dừng đúng trước thời điểm DELETE
systemctl start postgresql

# Kết quả: Database khôi phục về trạng thái 14:34:59
# → Mất tối đa 1 phút data (từ 14:34 đến 14:35)
# thay vì mất toàn bộ orders từ 02:00 trở đi
```

## 15.4 Backup Strategy Thực Tế

```
Quy tắc 3-2-1:
  3 bản sao data
  2 loại storage khác nhau (disk + cloud)
  1 bản off-site (datacenter khác, cloud khác)

Lịch backup thực tế cho production:
  - WAL archiving: real-time, liên tục (RPO gần như 0)
  - Full backup: 1 lần/ngày (vào 2:00 AM)
  - Kiểm tra restore: 1 lần/tuần (tự động, trên server test)
  - Giữ backup: 30 ngày local, 1 năm cold storage

RPO (Recovery Point Objective): Mất tối đa bao nhiêu data?
  Với WAL archiving: < 1 phút

RTO (Recovery Time Objective): Mất bao lâu để phục hồi?
  Database 100GB: ~30 phút để restore + apply WAL
  Database 10TB: vài giờ → cần streaming replica + failover
```

---

# CHƯƠNG 16: Connection Lifecycle — Hành Trình Của Một Kết Nối

## 16.1 Kết Nối Không Miễn Phí

Khi application kết nối đến PostgreSQL, nhiều thứ xảy ra:

```
1. TCP handshake (3-way): client ↔ postmaster process
   → ~1ms nếu cùng datacenter

2. SSL/TLS handshake (nếu bật):
   → Thêm ~10ms

3. Authentication:
   → Gửi username/password
   → Database xác thực (so sánh pg_hba.conf, verify password)
   → ~1-2ms

4. PostgreSQL fork backend process:
   → postmaster tạo một process con riêng cho connection này
   → Copy shared memory mappings
   → Load session-level caches
   → ~20-50ms và ~5-10MB RAM

5. Kết nối sẵn sàng
   → Tổng thời gian: ~50-100ms mỗi kết nối mới
   → RAM: ~5-10MB mỗi kết nối idle
```

Ứng dụng web có 500 concurrent users, mỗi request tạo kết nối mới:
- 500 request/s × 50ms = tất cả thời gian dành cho việc kết nối, chưa làm gì
- 500 kết nối × 8MB = 4GB RAM chỉ để giữ kết nối

→ **Connection pool là bắt buộc**, không phải optional.

## 16.2 Connection Pool Hoạt Động Thế Nào

```
Không có pool:
  App Request 1 → [kết nối] → [query] → [đóng kết nối]
  App Request 2 → [kết nối] → [query] → [đóng kết nối]  ← tạo lại từ đầu
  App Request 3 → [kết nối] → [query] → [đóng kết nối]

Có pool (ví dụ: pool size = 20):
  Startup: Tạo sẵn 20 kết nối đến PostgreSQL

  App Request 1 → [lấy kết nối từ pool] → [query] → [trả kết nối về pool]
  App Request 2 → [lấy kết nối từ pool] → [query] → [trả kết nối về pool]
  App Request 21 → [chờ kết nối trống] → [lấy kết nối] → [query] → [trả về]
```

**PgBouncer — Connection Pooler cho PostgreSQL:**

```ini
# pgbouncer.ini
[databases]
myapp = host=127.0.0.1 port=5432 dbname=myapp

[pgbouncer]
listen_port = 6432
listen_addr = *
auth_type = md5
pool_mode = transaction  # pool theo transaction (khuyến nghị)
max_client_conn = 1000   # tối đa 1000 client kết nối đến PgBouncer
default_pool_size = 20   # PgBouncer giữ 20 kết nối thực đến PostgreSQL
min_pool_size = 5
reserve_pool_size = 5    # dự phòng cho lúc cao điểm
```

**Các chế độ pool:**

```
Session mode:
  Kết nối PgBouncer ↔ PostgreSQL gắn với client connection trong suốt session
  Ít tiết kiệm nhất, tương thích cao nhất

Transaction mode (khuyến nghị):
  Kết nối được tái sử dụng sau mỗi transaction
  Rất tiết kiệm: 1000 client conn → chỉ cần 20 PG conn
  ⚠ Không hỗ trợ: prepared statements qua session, SET cho session config

Statement mode:
  Tái sử dụng sau mỗi statement
  Tiết kiệm nhất nhưng hạn chế nhiều nhất
  Không hỗ trợ multi-statement transactions
```

## 16.3 Cấu Hình Connection Pool Đúng Cách

Câu hỏi thường gặp: pool size bao nhiêu là đủ?

**Công thức thực tế:**

```
Pool size = (số CPU core × 2) + số disk spindle

Ví dụ: 8 core, SSD (coi là 1 "spindle"):
  Pool size = 8 × 2 + 1 = 17 ≈ 20

Lý do: PostgreSQL là CPU-bound + I/O-bound.
  Quá nhiều kết nối đồng thời → context switching overhead
  Thường 20-100 kết nối là optimal cho hầu hết workload
  1000 kết nối không nhanh hơn 100 kết nối — thậm chí chậm hơn
```

**Monitoring connection pool:**

```sql
-- Xem kết nối hiện tại
SELECT state, count(*), wait_event_type, wait_event
FROM pg_stat_activity
WHERE datname = 'myapp'
GROUP BY state, wait_event_type, wait_event
ORDER BY count DESC;

-- state = 'active': đang thực thi query
-- state = 'idle': kết nối rảnh, chờ lệnh tiếp theo
-- state = 'idle in transaction': trong transaction nhưng không làm gì (nguy hiểm!)
-- state = 'idle in transaction (aborted)': transaction bị lỗi, chưa rollback (rất nguy hiểm)

-- Tìm long-running queries
SELECT pid, now() - pg_stat_activity.query_start AS duration, query
FROM pg_stat_activity
WHERE (now() - pg_stat_activity.query_start) > interval '5 minutes'
  AND state = 'active';

-- Kill query cụ thể
SELECT pg_cancel_backend(pid);   -- gửi SIGINT (cancel query, giữ kết nối)
SELECT pg_terminate_backend(pid); -- gửi SIGTERM (đóng kết nối luôn)
```

---

# CHƯƠNG 17: Monitoring và Vận Hành Production

## 17.1 Các Chỉ Số Quan Trọng Cần Monitor

```
Nhóm 1 — Hiệu năng query:
  - Query latency (p50, p95, p99)
  - Slow queries (> 1s, > 5s)
  - Queries/second
  - Cache hit rate (Buffer Pool)

Nhóm 2 — Tài nguyên hệ thống:
  - CPU usage của PostgreSQL processes
  - RAM: shared_buffers hit rate, total memory usage
  - Disk I/O: read/write IOPS, disk latency
  - Disk space: data dir, WAL dir, backup dir

Nhóm 3 — Sức khoẻ database:
  - Số kết nối (active, idle, idle in transaction)
  - Lock waits và deadlocks
  - Replication lag (nếu có replica)
  - Autovacuum hoạt động có đúng không

Nhóm 4 — Safety:
  - Transaction ID age (nguy cơ wraparound)
  - Backup age (bản backup cuối cùng bao lâu trước?)
  - WAL archive lag
```

## 17.2 Queries Monitoring Quan Trọng

```sql
-- Bật pg_stat_statements extension (một lần)
CREATE EXTENSION pg_stat_statements;

-- Top 10 queries chậm nhất (tổng thời gian)
SELECT
    round(total_exec_time::numeric, 2) AS total_ms,
    calls,
    round(mean_exec_time::numeric, 2) AS avg_ms,
    round(stddev_exec_time::numeric, 2) AS stddev_ms,
    left(query, 100) AS query_preview
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;

-- Top queries theo số lần gọi
SELECT calls, round(mean_exec_time::numeric, 2) AS avg_ms, query
FROM pg_stat_statements
ORDER BY calls DESC
LIMIT 10;

-- Queries với high rows/exec (có thể đang trả về quá nhiều data)
SELECT calls,
       rows / calls AS rows_per_call,
       round(mean_exec_time::numeric, 2) AS avg_ms,
       left(query, 80) AS query
FROM pg_stat_statements
WHERE calls > 100
ORDER BY rows / calls DESC
LIMIT 10;

-- Buffer cache hit rate (nên > 99% cho OLTP)
SELECT
    sum(blks_hit) AS cache_hits,
    sum(blks_read) AS disk_reads,
    round(sum(blks_hit) * 100.0 / nullif(sum(blks_hit) + sum(blks_read), 0), 2) AS hit_rate_pct
FROM pg_stat_database
WHERE datname = 'myapp';

-- Table bloat (dead tuples nhiều)
SELECT relname, n_live_tup, n_dead_tup,
       round(n_dead_tup * 100.0 / nullif(n_live_tup + n_dead_tup, 0), 1) AS dead_pct,
       pg_size_pretty(pg_total_relation_size(oid)) AS total_size
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC
LIMIT 20;

-- Index không được dùng (candidates để xóa)
SELECT schemaname, tablename, indexname,
       pg_size_pretty(pg_relation_size(indexrelid)) AS index_size,
       idx_scan AS times_used
FROM pg_stat_user_indexes
JOIN pg_index USING (indexrelid)
WHERE NOT indisprimary
  AND NOT indisunique
  AND idx_scan < 50         -- dùng ít hơn 50 lần kể từ lần restart cuối
  AND pg_relation_size(indexrelid) > 1024 * 1024  -- lớn hơn 1MB
ORDER BY pg_relation_size(indexrelid) DESC;
```

## 17.3 Cấu Hình PostgreSQL Quan Trọng

```ini
# postgresql.conf — các thông số quan trọng nhất

# Memory
shared_buffers = 4GB              # 25% RAM — buffer pool chính
effective_cache_size = 12GB       # 75% RAM — hint cho optimizer (không cấp phát thật)
work_mem = 64MB                   # RAM cho sort/hash mỗi operation (cẩn thận: multiplied by connections × ops)
maintenance_work_mem = 512MB      # RAM cho VACUUM, CREATE INDEX

# WAL và Durability
wal_level = replica               # cần cho replication và PITR
synchronous_commit = on           # off = nhanh hơn nhưng mất tối đa wal_writer_delay data
checkpoint_completion_target = 0.9 # spread I/O evenly
max_wal_size = 4GB                # WAL tối đa trước khi force checkpoint

# Connections
max_connections = 200             # tổng kết nối tối đa
# Lưu ý: mỗi kết nối ~5-10MB shared memory
# Với PgBouncer, giữ max_connections thấp (~100-200)

# Query Planning
random_page_cost = 1.1            # SSD: đặt 1.1-2.0 (default 4.0 cho HDD)
effective_io_concurrency = 200    # SSD: 200+, HDD: 2-4
default_statistics_target = 100  # tăng lên 200-500 cho cột có phân phối không đều

# Logging
log_min_duration_statement = 1000 # log query chạy hơn 1 giây
log_checkpoints = on
log_lock_waits = on               # log khi chờ lock > deadlock_timeout
log_temp_files = 0                # log khi tạo temp file (sort/hash spill)
```

---

# Tổng Kết — Bức Tranh Toàn Cảnh

```
Bạn viết SQL
     │
     ▼
[Query Parser & Planner]
  Phân tích cú pháp SQL
  Xem statistics (pg_statistic): bao nhiêu rows, phân phối thế nào
  Tạo nhiều execution plan (Seq Scan, Index Scan, Hash Join, Merge Join...)
  Dùng cost model để chọn plan rẻ nhất
     │
     ▼
[Executor]
  Thực thi plan từng node (từ trong ra ngoài)
  Kiểm tra Buffer Pool — page đã có trong RAM chưa?
  Cache miss → đọc từ disk vào Buffer Pool
  Cache hit → lấy từ RAM (0.0001ms)
     │
     ▼
[Transaction Manager]
  Ghi WAL trước khi thay đổi (Atomicity, Durability)
  Kiểm tra Constraints (Consistency)
  MVCC: xem transaction nào được thấy gì (Isolation)
  Locking: ngăn conflict khi cần
  Phát hiện Deadlock nếu có
     │
     ▼
[Storage Engine]
  Đọc/ghi Pages (8KB mỗi page)
  Buffer Pool management (dirty pages, eviction)
  Background checkpointer: flush dirty pages xuống disk
  Autovacuum: dọn dead tuples, cập nhật statistics
     │
     ▼
[WAL Writer]
  fsync WAL trước khi commit → đảm bảo Durability
  WAL Archiving → enable PITR
     │
     ▼
Disk
  Data files (tables, indexes)
  WAL files
  pg_stat files
```

## Khi Nào Dùng Gì — Bảng Quyết Định Nhanh

| Tình huống | Giải pháp |
|---|---|
| Query chậm | EXPLAIN ANALYZE → thêm index đúng chỗ |
| Query vẫn chậm dù có index | Xem index có được dùng không? Viết lại query tránh function trên cột |
| Write chậm | Kiểm tra số lượng index (mỗi write phải update tất cả) |
| Table phình to | VACUUM ANALYZE; kiểm tra autovacuum có chạy không |
| 1000 users cùng mua 1 sản phẩm | SELECT ... FOR UPDATE; hoặc tăng isolation level |
| Deadlock thường xuyên | Lock theo thứ tự nhất quán; dùng NOWAIT |
| Kết nối chậm | Thêm PgBouncer connection pool |
| Database quá lớn, query chậm | Partitioning theo time hoặc category |
| Xóa data cũ mất hàng giờ | Partitioning + DROP PARTITION |
| Cần tìm kiếm văn bản | Full-text search với tsvector + GIN index |
| Cần analytics nhanh trên data lớn | ClickHouse hoặc PostgreSQL + partitioning + columnar extension |
| Data mất sau crash | Kiểm tra fsync=on, WAL archiving bật |
| Cần khôi phục về thời điểm cụ thể | PITR với base backup + WAL archive |

**Nếu chỉ nhớ một điều từ tài liệu này:**

> Database là hệ thống được thiết kế để lưu dữ liệu **nhanh, đúng, và bền vững** — ngay cả khi nhiều người dùng cùng lúc và máy chủ có thể crash bất kỳ lúc nào.  
> ACID là tập hợp những đảm bảo cụ thể để làm được điều đó.  
> Mọi thứ khác (index, sharding, replication, partitioning, vacuum) là tối ưu hóa phía trên nền tảng đó.