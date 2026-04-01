# \# 🗄️ Database Deep Dive — Toàn Tập

# 

# \---

# 

# \# PHẦN 1: INDEXING — Cơ Chế Bên Trong

# 

# \## 1.1 B-Tree Internals

# 

# \### Cấu trúc thực sự của B-Tree

# 

# B-Tree (balanced tree) trong database là \*\*B+ Tree\*\* — tất cả data nằm ở leaf nodes, internal nodes chỉ chứa key để điều hướng.

# 

# ```

# &#x20;                       \[Internal Node]

# &#x20;                       | 30 | 60 |

# &#x20;                      /     |     \\

# &#x20;             \[10|20]      \[40|50]      \[70|80]     ← Internal Nodes

# &#x20;            /  |   \\      /  |  \\      /  |  \\

# &#x20;          \[5] \[15] \[25] \[35]\[45]\[55] \[65]\[75]\[85]  ← Leaf Nodes

# &#x20;           ↕   ↕   ↕    ↕   ↕   ↕    ↕   ↕   ↕

# &#x20;          (data pointers — trỏ tới heap/row)

# &#x20;          

# &#x20;          Leaf nodes được nối với nhau bằng linked list:

# &#x20;          \[5] → \[15] → \[25] → \[35] → ... → \[85]

# ```

# 

# \*\*Tại sao dùng B+ Tree thay vì BST?\*\*

# \- BST có thể unbalanced → O(n) worst case

# \- B+ Tree luôn balanced → O(log n) guaranteed

# \- Leaf nodes linked list → range scan rất hiệu quả (không cần traverse tree)

# \- \*\*Page size\*\* phù hợp với disk I/O (thường 8KB hay 16KB) → đọc 1 lần được nhiều key

# 

# \### Chi phí duy trì Index

# 

# Mỗi lần INSERT/UPDATE/DELETE:

# 1\. Tìm vị trí đúng trong B-Tree: O(log n)

# 2\. Insert key vào leaf node

# 3\. Nếu node đầy → \*\*page split\*\*: chia đôi node, đẩy middle key lên parent

# 4\. Page split lan truyền lên có thể khiến cả tree phải rebalance

# 

# ```

# Trước split:          Sau khi insert 28 vào \[20|25|27] (full):

# &#x20;                     

# \[20 | 25 | 27]   →   \[20 | 25]  \[27 | 28]

# &#x20;                              ↑

# &#x20;                         25 được đẩy lên parent

# ```

# 

# > 💡 \*\*Ý nghĩa thực tế\*\*: Bảng insert nhiều → index bị fragmented → cần `REINDEX` hoặc `VACUUM` định kỳ.

# 

# \### Fill Factor

# PostgreSQL cho phép set fill factor (mặc định 90%):

# ```sql

# CREATE INDEX idx\_orders ON orders(created\_at) WITH (fillfactor = 70);

# ```

# \- Fill factor 70% → mỗi leaf page chỉ lấp đầy 70%, 30% dự phòng cho insert

# \- Giảm page split → write nhanh hơn

# \- Trade-off: index lớn hơn, dùng nhiều disk hơn

# 

# \---

# 

# \## 1.2 Index Types Chi Tiết

# 

# \### Hash Index

# 

# ```

# Key: "nguyen@email.com"

# &#x20;       ↓

# &#x20;  Hash Function

# &#x20;       ↓

# &#x20;  bucket\_id = 42

# &#x20;       ↓

# &#x20;  \[bucket 42] → \[(key, row\_pointer), ...]

# ```

# 

# \- Lookup O(1) — tốt cho `=` exact match

# \- \*\*Không\*\* hỗ trợ: range query, ORDER BY, LIKE

# \- PostgreSQL: Hash index không được WAL-logged trước v10 (mất khi crash)

# \- MySQL Memory engine dùng hash index mặc định

# 

# \### GiST Index (Generalized Search Tree)

# Dùng cho các kiểu dữ liệu phức tạp:

# ```sql

# \-- Geometric data

# CREATE INDEX ON places USING gist(location);

# SELECT \* FROM places WHERE location <-> point(10.8, 106.7) < 5; -- trong 5km

# 

# \-- Range types

# CREATE INDEX ON reservations USING gist(during);

# SELECT \* FROM reservations WHERE during \&\& '\[2024-01-01, 2024-01-07]'::daterange;

# ```

# 

# \### GIN Index (Generalized Inverted Index)

# Dùng cho array, jsonb, full-text search:

# ```sql

# \-- JSONB

# CREATE INDEX ON products USING gin(attributes);

# SELECT \* FROM products WHERE attributes @> '{"color": "red"}';

# 

# \-- Array

# CREATE INDEX ON articles USING gin(tags);

# SELECT \* FROM articles WHERE tags @> ARRAY\['database', 'performance'];

# 

# \-- Full-text search

# CREATE INDEX ON docs USING gin(to\_tsvector('english', content));

# ```

# 

# \*\*GIN vs GiST:\*\*

# | | GIN | GiST |

# |---|---|---|

# | Build time | Chậm | Nhanh |

# | Index size | Lớn hơn | Nhỏ hơn |

# | Lookup | Nhanh hơn | Chậm hơn |

# | Update | Chậm (pending list) | Nhanh |

# 

# \### BRIN Index (Block Range Index)

# Dùng cho dữ liệu \*\*tự nhiên có thứ tự theo thời gian\*\* (time-series, log):

# ```sql

# CREATE INDEX ON events USING brin(created\_at);

# ```

# \- Lưu min/max của mỗi block (thường 128 pages)

# \- Index rất nhỏ (vài KB so với GB data)

# \- Hiệu quả nếu dữ liệu insert theo thứ tự thời gian

# \- Kém hiệu quả nếu dữ liệu random

# 

# \---

# 

# \## 1.3 Index Strategies Nâng Cao

# 

# \### Index Merge (MySQL)

# MySQL có thể \*\*combine nhiều index\*\* cho 1 query:

# ```sql

# \-- Có index trên (status) và index trên (user\_id) riêng lẻ

# SELECT \* FROM orders WHERE status = 'pending' OR user\_id = 123;

# \-- MySQL dùng: index merge union của 2 index

# ```

# 

# Nhưng thường \*\*composite index tốt hơn index merge\*\*:

# ```sql

# \-- Tốt hơn: composite index

# CREATE INDEX ON orders(user\_id, status);

# ```

# 

# \### Index Selectivity

# ```sql

# \-- Kiểm tra selectivity (PostgreSQL)

# SELECT

# &#x20;   attname,

# &#x20;   n\_distinct,

# &#x20;   correlation  -- 1.0 = hoàn toàn sorted, 0 = random

# FROM pg\_stats

# WHERE tablename = 'orders';

# ```

# 

# \*\*Selectivity = số distinct values / tổng rows\*\*

# \- Gần 1.0 → high selectivity → index rất hiệu quả (user\_id, email)

# \- Gần 0 → low selectivity → index ít hiệu quả (status: 'active'/'inactive')

# 

# Quy tắc: Nếu query trả về > 5-10% rows, DB thường prefer seq scan hơn index scan.

# 

# \### Index Condition Pushdown (ICP) — MySQL

# ```sql

# \-- Index trên (last\_name, first\_name)

# SELECT \* FROM users WHERE last\_name LIKE 'Ng%' AND first\_name LIKE 'An%';

# ```

# Không có ICP: index tìm theo last\_name, fetch row, rồi mới check first\_name  

# Có ICP: check cả first\_name ngay tại index level → giảm row fetch

# 

# \### Invisible Index (MySQL 8+)

# ```sql

# \-- Test xem drop index có ảnh hưởng gì không

# ALTER TABLE orders ALTER INDEX idx\_status INVISIBLE;

# \-- Query optimizer bỏ qua index này

# \-- Nếu performance không đổi → có thể drop an toàn

# ALTER TABLE orders ALTER INDEX idx\_status VISIBLE;

# ```

# 

# \---

# 

# \# PHẦN 2: TRANSACTION \& LOCKING

# 

# \## 2.1 Locking — Cơ Chế Chi Tiết

# 

# \### Lock Granularity (từ thô → tinh)

# 

# ```

# Database Lock

# &#x20;   └── Table Lock

# &#x20;           └── Page Lock

# &#x20;                   └── Row Lock

# &#x20;                           └── Column Lock (hiếm)

# ```

# 

# \*\*Trade-off\*\*: Lock càng tinh → concurrency càng cao, overhead quản lý lock càng lớn.

# 

# \### Lock Modes

# 

# | Lock | Ký hiệu | Tương thích với | Dùng khi |

# |---|---|---|---|

# | \*\*Access Share\*\* | AS | Tất cả trừ ACCESS EXCLUSIVE | SELECT |

# | \*\*Row Share\*\* | RS | Tất cả trừ EXCLUSIVE, ACCESS EXCLUSIVE | SELECT FOR UPDATE |

# | \*\*Row Exclusive\*\* | RX | AS, RS, RX | INSERT, UPDATE, DELETE |

# | \*\*Share Update Exclusive\*\* | SUX | AS | VACUUM, CREATE INDEX CONCURRENTLY |

# | \*\*Share\*\* | S | AS, S | CREATE INDEX (non-concurrent) |

# | \*\*Share Row Exclusive\*\* | SRX | AS | Trigger operations |

# | \*\*Exclusive\*\* | X | AS | Hiếm dùng trực tiếp |

# | \*\*Access Exclusive\*\* | AX | Không có | ALTER TABLE, DROP TABLE |

# 

# \### SELECT FOR UPDATE vs SELECT FOR SHARE

# 

# ```sql

# \-- FOR UPDATE: lock row, không ai UPDATE/DELETE/SELECT FOR UPDATE được

# BEGIN;

# SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;

# \-- Row bị lock → transaction khác phải chờ

# UPDATE accounts SET balance = balance - 100 WHERE id = 1;

# COMMIT;

# 

# \-- FOR SHARE: lock row, cho phép SELECT khác nhưng không cho UPDATE

# SELECT \* FROM orders WHERE id = 1 FOR SHARE;

# 

# \-- SKIP LOCKED: bỏ qua row đang bị lock (dùng cho job queue)

# SELECT \* FROM jobs WHERE status = 'pending'

# ORDER BY created\_at

# FOR UPDATE SKIP LOCKED

# LIMIT 1;

# 

# \-- NOWAIT: không chờ, raise error ngay nếu bị lock

# SELECT \* FROM inventory WHERE id = 1 FOR UPDATE NOWAIT;

# ```

# 

# \### SKIP LOCKED — Pattern Job Queue

# ```sql

# \-- Worker pattern: mỗi worker lấy 1 job không bị worker khác lấy

# BEGIN;

# SELECT id, payload FROM job\_queue

# WHERE status = 'pending'

# ORDER BY priority DESC, created\_at ASC

# FOR UPDATE SKIP LOCKED

# LIMIT 1;

# 

# \-- Process job...

# UPDATE job\_queue SET status = 'done' WHERE id = $1;

# COMMIT;

# ```

# 

# \---

# 

# \## 2.2 Deadlock

# 

# \### Deadlock xảy ra thế nào

# 

# ```

# Transaction T1                    Transaction T2

# ──────────────                    ──────────────

# LOCK row A (success)              LOCK row B (success)

# LOCK row B (waiting...)           LOCK row A (waiting...)

# &#x20;        ↑                                 ↑

# &#x20;        └─────────── DEADLOCK ────────────┘

# ```

# 

# \*\*Ví dụ thực tế — chuyển tiền:\*\*

# ```sql

# \-- T1: Chuyển từ account 1 → account 2

# UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- lock row 1

# UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- chờ lock row 2

# 

# \-- T2: Chuyển từ account 2 → account 1 (đồng thời)

# UPDATE accounts SET balance = balance - 50 WHERE id = 2;  -- lock row 2

# UPDATE accounts SET balance = balance + 50 WHERE id = 1;  -- chờ lock row 1

# 

# \-- → DEADLOCK

# ```

# 

# \### Phát hiện và xử lý Deadlock

# DB tự detect deadlock bằng \*\*wait-for graph\*\*:

# ```

# T1 → waits for → T2

# T2 → waits for → T1

# → Cycle detected → victim (T1 hoặc T2) bị abort và rollback

# ```

# 

# \*\*Cách phòng tránh deadlock:\*\*

# 

# 1\. \*\*Lock theo thứ tự cố định:\*\*

# ```sql

# \-- Luôn lock account ID nhỏ hơn trước

# BEGIN;

# SELECT \* FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;

# \-- Bây giờ T1 và T2 đều cố lock row 1 trước → không deadlock

# ```

# 

# 2\. \*\*Giữ transaction ngắn nhất có thể:\*\*

# ```sql

# \-- BAD: Xử lý nghiệp vụ phức tạp trong transaction

# BEGIN;

# SELECT ...   -- lock

# \-- 2 giây xử lý...

# UPDATE ...

# COMMIT;

# 

# \-- GOOD: Chuẩn bị xong rồi mới open transaction

# \-- Xử lý nghiệp vụ...

# BEGIN;

# UPDATE ...   -- nhanh gọn

# COMMIT;

# ```

# 

# 3\. \*\*Lock ở mức thấp nhất cần thiết:\*\* Dùng row lock thay vì table lock.

# 

# 4\. \*\*Retry logic trong application:\*\*

# ```python

# MAX\_RETRIES = 3

# for attempt in range(MAX\_RETRIES):

# &#x20;   try:

# &#x20;       with db.transaction():

# &#x20;           transfer(from\_id, to\_id, amount)

# &#x20;       break

# &#x20;   except DeadlockError:

# &#x20;       if attempt == MAX\_RETRIES - 1:

# &#x20;           raise

# &#x20;       time.sleep(0.1 \* (2 \*\* attempt))  # exponential backoff

# ```

# 

# \---

# 

# \## 2.3 MVCC — Multi-Version Concurrency Control

# 

# \### Ý tưởng cốt lõi

# Thay vì lock khi đọc, DB \*\*giữ nhiều version\*\* của mỗi row. Reader thấy snapshot tại thời điểm transaction bắt đầu, writer tạo version mới.

# 

# \*\*"Readers don't block writers, writers don't block readers"\*\*

# 

# \### PostgreSQL MVCC Implementation

# 

# ```

# Mỗi row có hidden columns:

# ┌──────────┬──────────┬────────────┬────────────┐

# │  xmin    │  xmax    │    data    │   ctid     │

# ├──────────┼──────────┼────────────┼────────────┤

# │ txid tạo │ txid xóa │ actual data│ physical   │

# │          │ (0=alive)│            │ location   │

# └──────────┴──────────┴────────────┴────────────┘

# ```

# 

# \*\*Luồng UPDATE trong PostgreSQL:\*\*

# ```

# Row gốc:    xmin=100, xmax=0,   data="old value"

# &#x20;               ↓

# Transaction 200 UPDATE:

# Row cũ:     xmin=100, xmax=200, data="old value"   ← mark deleted

# Row mới:    xmin=200, xmax=0,   data="new value"   ← new version

# ```

# 

# Transaction với txid=150 (bắt đầu trước T200) vẫn thấy "old value" vì xmin=100 < 150 < xmax=200.

# 

# \### Visibility Rules

# Row R visible với transaction T khi:

# \- `R.xmin < T.txid` (row được tạo trước T)

# \- `R.xmin` đã committed

# \- `R.xmax = 0` HOẶC `R.xmax > T.txid` (row chưa bị xóa, hoặc bị xóa sau T)

# \- `R.xmax` chưa committed (nếu đang xóa)

# 

# \### MVCC và vấn đề Table Bloat

# PostgreSQL không xóa row ngay → \*\*dead tuples\*\* tích tụ:

# ```sql

# \-- Kiểm tra dead tuples

# SELECT relname, n\_dead\_tup, n\_live\_tup,

# &#x20;      round(n\_dead\_tup::numeric / nullif(n\_live\_tup + n\_dead\_tup, 0) \* 100, 2) AS dead\_pct

# FROM pg\_stat\_user\_tables

# ORDER BY n\_dead\_tup DESC;

# 

# \-- VACUUM reclaim space từ dead tuples

# VACUUM orders;

# VACUUM FULL orders;  -- Aggressive, lock table, reclaim disk space

# AUTOVACUUM tự chạy nhưng có thể cần tune

# ```

# 

# \### Transaction Snapshot

# ```sql

# \-- Xem snapshot hiện tại

# SELECT pg\_current\_snapshot();

# \-- Kết quả: 100:105:100,102

# \-- Nghĩa: xmin=100, xmax=105, in-progress={100,102}

# \-- → Transactions 100 và 102 đang chạy, chưa visible

# ```

# 

# \---

# 

# \## 2.4 Two-Phase Locking (2PL)

# 

# Đảm bảo serializability:

# 1\. \*\*Growing phase\*\*: Transaction chỉ acquire lock, không release

# 2\. \*\*Shrinking phase\*\*: Transaction chỉ release lock, không acquire thêm

# 

# ```

# Growing Phase          Shrinking Phase

# ──────────────────     ─────────────────

# Acquire lock A    │    Release lock A

# Acquire lock B    │    Release lock B

# Acquire lock C    │    Release lock C

# &#x20;                 ↑

# &#x20;            Lock Point (max locks held)

# ```

# 

# \*\*Strict 2PL\*\* (S2PL): Giữ tất cả locks đến khi COMMIT/ROLLBACK — phổ biến hơn, tránh cascading rollback.

# 

# \---

# 

# \## 2.5 Distributed Transactions — Two-Phase Commit (2PC)

# 

# Khi transaction span nhiều database nodes:

# 

# ```

# Phase 1 — Prepare:

# Coordinator → "Prepare to commit?" → Node A, Node B, Node C

# Node A → "Ready" ✓

# Node B → "Ready" ✓

# Node C → "Ready" ✓

# 

# Phase 2 — Commit:

# Coordinator → "Commit!" → Node A, Node B, Node C

# ```

# 

# \*\*Vấn đề của 2PC:\*\*

# \- \*\*Blocking protocol\*\*: Nếu coordinator crash sau Phase 1, các nodes bị treo (không thể tự quyết)

# \- \*\*Performance\*\*: 2 round trips qua network → latency cao

# \- Giải pháp hiện đại: \*\*Saga pattern\*\* (choreography/orchestration) thay vì 2PC

# 

# \---

# 

# \# PHẦN 3: QUERY OPTIMIZER \& EXECUTION PLAN

# 

# \## 3.1 Query Lifecycle

# 

# ```

# SQL Text

# &#x20;  ↓

# \[Parser] — Kiểm tra syntax, tạo parse tree

# &#x20;  ↓

# \[Analyzer/Rewriter] — Kiểm tra semantic, resolve tên bảng/cột, expand views

# &#x20;  ↓

# \[Planner/Optimizer] — Tạo execution plan tối ưu

# &#x20;  ↓

# \[Executor] — Thực thi plan

# &#x20;  ↓

# Result

# ```

# 

# \## 3.2 Cost-Based Optimizer

# 

# Optimizer tạo nhiều plan, ước tính cost của mỗi plan, chọn plan có cost thấp nhất.

# 

# \### Statistics — Nền tảng của Optimizer

# 

# ```sql

# \-- PostgreSQL thu thập statistics

# ANALYZE orders;

# 

# \-- Xem statistics

# SELECT \*

# FROM pg\_stats

# WHERE tablename = 'orders' AND attname = 'status';

# ```

# 

# Statistics bao gồm:

# \- `n\_distinct`: số distinct values (-1 = unique)

# \- `most\_common\_vals`: top N values phổ biến nhất

# \- `most\_common\_freqs`: frequency của các common values

# \- `histogram\_bounds`: phân phối data

# \- `correlation`: mức độ sorted của cột so với physical order

# 

# \### Ước tính Selectivity

# 

# ```sql

# \-- Query: WHERE status = 'pending'

# \-- most\_common\_vals = {active, pending, done}

# \-- most\_common\_freqs = {0.6, 0.3, 0.1}

# \-- → Selectivity = 0.3 → 30% rows sẽ được trả về

# \-- → Nếu bảng có 100,000 rows → estimated 30,000 rows

# ```

# 

# \### Tại sao Optimizer sai?

# 1\. \*\*Statistics cũ\*\* (chưa ANALYZE sau bulk insert)

# 2\. \*\*Correlated columns\*\* (city và zip\_code tương quan nhau nhưng optimizer coi là độc lập)

# 3\. \*\*Non-uniform distribution\*\* (99% rows có status='active', 1% khác)

# 4\. \*\*High row estimation error\*\* → sai join order → plan tệ

# 

# ```sql

# \-- Force statistics update

# ANALYZE VERBOSE orders;

# 

# \-- Extended statistics cho correlated columns (PG 10+)

# CREATE STATISTICS stats\_city\_zip ON city, zip\_code FROM users;

# ANALYZE users;

# ```

# 

# \---

# 

# \## 3.3 Join Algorithms Chi Tiết

# 

# \### Nested Loop Join

# ```

# FOR each row r1 IN outer\_table:

# &#x20;   FOR each row r2 IN inner\_table:

# &#x20;       IF r1.key == r2.key:

# &#x20;           output(r1, r2)

# ```

# \- \*\*Complexity\*\*: O(n × m)

# \- \*\*Tốt khi\*\*: outer table nhỏ, inner table có index

# \- \*\*Memory\*\*: O(1) — không cần buffer

# 

# \### Hash Join

# ```

# Phase 1 — Build:

# &#x20; FOR each row r IN smaller\_table:

# &#x20;     hash\_table\[hash(r.key)] = r

# 

# Phase 2 — Probe:

# &#x20; FOR each row r IN larger\_table:

# &#x20;     IF hash\_table\[hash(r.key)] exists:

# &#x20;         output(match)

# ```

# \- \*\*Complexity\*\*: O(n + m)

# \- \*\*Tốt khi\*\*: không có useful index, large tables

# \- \*\*Memory\*\*: O(smaller\_table) — cần RAM để hold hash table

# \- \*\*Grace Hash Join\*\*: Khi hash table không vừa RAM → spill to disk, chia thành batches

# 

# \### Sort-Merge Join

# ```

# Phase 1 — Sort: Sort cả 2 tables theo join key

# Phase 2 — Merge: Merge 2 sorted streams như merge sort

# ```

# \- \*\*Complexity\*\*: O(n log n + m log m) nếu chưa sorted

# \- \*\*Complexity\*\*: O(n + m) nếu đã sorted

# \- \*\*Tốt khi\*\*: Data đã sorted (index), range joins, non-equi joins

# 

# \### Khi nào dùng join nào?

# 

# | Tình huống | Join được chọn |

# |---|---|

# | Outer nhỏ + inner có index | Nested Loop |

# | Cả 2 lớn, không có index | Hash Join |

# | Cả 2 đã sorted / có sorted index | Merge Join |

# | Nhiều rows trùng join key | Hash Join |

# | `<`, `>`, range conditions | Merge Join hoặc Nested Loop |

# 

# \---

# 

# \## 3.4 Đọc EXPLAIN ANALYZE Chi Tiết

# 

# ```sql

# EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)

# SELECT u.name, COUNT(o.id)

# FROM users u

# JOIN orders o ON u.id = o.user\_id

# WHERE u.created\_at > '2024-01-01'

# GROUP BY u.id, u.name;

# ```

# 

# \*\*Kết quả mẫu:\*\*

# ```

# HashAggregate  (cost=1250.00..1280.00 rows=3000 width=40)

# &#x20;              (actual time=45.123..47.456 rows=2800 loops=1)

# &#x20; Buffers: shared hit=850 read=120

# &#x20; ->  Hash Join  (cost=450.00..1150.00 rows=20000 width=16)

# &#x20;                (actual time=12.345..38.901 rows=19500 loops=1)

# &#x20;       Hash Cond: (o.user\_id = u.id)

# &#x20;       Buffers: shared hit=800 read=120

# &#x20;       ->  Seq Scan on orders o

# &#x20;             (cost=0.00..600.00 rows=50000 width=8)

# &#x20;             (actual time=0.012..15.234 rows=50000 loops=1)

# &#x20;             Buffers: shared hit=400

# &#x20;       ->  Hash  (cost=400.00..400.00 rows=4000 width=12)

# &#x20;                 (actual time=12.100..12.100 rows=3800 loops=1)

# &#x20;             Buckets: 4096  Batches: 1  Memory Usage: 256kB

# &#x20;             ->  Index Scan using idx\_users\_created on users u

# &#x20;                   (cost=0.00..400.00 rows=4000 width=12)

# &#x20;                   (actual time=0.021..10.234 rows=3800 loops=1)

# &#x20;                   Index Cond: (created\_at > '2024-01-01')

# &#x20;                   Buffers: shared hit=400 read=120

# ```

# 

# \### Giải thích từng phần

# 

# \*\*Cost: `cost=start\_cost..total\_cost`\*\*

# \- `start\_cost`: Chi phí trước khi trả row đầu tiên (sorting, hashing)

# \- `total\_cost`: Tổng chi phí để trả tất cả rows

# \- Đơn vị tương đối, không phải milliseconds

# 

# \*\*Rows estimation vs actual:\*\*

# \- `rows=3000` (estimate) vs `rows=2800` (actual) → sai lệch \~7% → tốt

# \- Nếu sai lệch > 10x → vấn đề statistics → cần ANALYZE

# 

# \*\*Buffers:\*\*

# \- `shared hit=850`: 850 pages đọc từ cache (shared\_buffers) → tốt

# \- `shared read=120`: 120 pages phải đọc từ disk → I/O xảy ra

# \- `shared written`: pages dirty được flush → cẩn thận

# 

# \*\*Loops:\*\*

# \- `loops=1`: Node chạy 1 lần

# \- `loops=1000`: Node chạy 1000 lần (Nested Loop inner) → nhân actual time với loops

# 

# \### Red Flags trong Execution Plan

# 

# ```

# ❌ Seq Scan trên bảng lớn → cần index

# ❌ rows estimate << actual rows (10x+) → statistics lỗi thời

# ❌ Hash Batches > 1 → hash table spill to disk → tăng work\_mem

# ❌ Nested Loop với loops rất lớn → join order sai

# ❌ Sort trên large dataset → có thể cần index để avoid sort

# ❌ Buffer read rất cao → cache miss, I/O bound

# ```

# 

# \### Hints (khi Optimizer sai)

# 

# ```sql

# \-- PostgreSQL: không có official hints, dùng config workaround

# SET enable\_seqscan = off;  -- Force index scan

# SET enable\_hashjoin = off; -- Force merge/nested join

# SET enable\_nestloop = off;

# 

# \-- MySQL: hints chính thức

# SELECT /\*+ USE\_INDEX(orders idx\_status) \*/ \* FROM orders;

# SELECT /\*+ NO\_HASH\_JOIN(u o) \*/ u.\*, o.\* FROM users u JOIN orders o;

# 

# \-- Reset về mặc định

# RESET enable\_seqscan;

# ```

# 

# \---

# 

# \## 3.5 Common Table Expressions (CTE) và Performance

# 

# ```sql

# \-- CTE mặc định trong PostgreSQL < 12: optimization fence

# \-- Optimizer KHÔNG thể push conditions vào trong CTE

# WITH active\_users AS (

# &#x20;   SELECT \* FROM users WHERE status = 'active'

# )

# SELECT \* FROM active\_users WHERE city = 'HCM';

# \-- PG < 12: materialize toàn bộ active\_users TRƯỚC, rồi mới filter city

# \-- PG >= 12: inline CTE, optimize như subquery

# 

# \-- Force materialization (khi muốn CTE chạy 1 lần, dùng nhiều chỗ)

# WITH MATERIALIZED expensive\_calc AS (

# &#x20;   SELECT user\_id, complex\_calculation(data) AS result

# &#x20;   FROM raw\_data

# )

# SELECT \* FROM expensive\_calc WHERE ...

# UNION ALL

# SELECT \* FROM expensive\_calc WHERE ...;

# 

# \-- Force inline (PG >= 12)

# WITH NOT MATERIALIZED users AS (

# &#x20;   SELECT \* FROM users WHERE status = 'active'

# )

# SELECT \* FROM users WHERE city = 'HCM';

# ```

# 

# \---

# 

# \# PHẦN 4: SHARDING \& REPLICATION

# 

# \## 4.1 Replication Chi Tiết

# 

# \### Statement-Based Replication

# Primary gửi \*\*SQL statements\*\* tới replica:

# ```

# Primary: UPDATE orders SET status = 'done' WHERE created\_at < NOW() - INTERVAL '7 days'

# Replica: Thực thi lại query này

# ```

# \- Ưu: Log nhỏ

# \- Nhược: Non-deterministic functions (`NOW()`, `RAND()`) cho kết quả khác nhau

# 

# \### Row-Based Replication (mặc định MySQL)

# Primary gửi \*\*actual row changes\*\*:

# ```

# Primary: \[table=orders, op=UPDATE, pk=123, before={status:'pending'}, after={status:'done'}]

# Replica: Apply change trực tiếp

# ```

# \- Ưu: Deterministic, an toàn hơn

# \- Nhược: Log lớn hơn với bulk updates

# 

# \### WAL Shipping (PostgreSQL)

# Primary ghi Write-Ahead Log, gửi WAL segments tới replica:

# 

# ```

# Primary:

# &#x20; Transaction → WAL buffer → WAL files (pg\_wal/)

# &#x20;                                  ↓

# &#x20;                             WAL Sender ──── network ────→ WAL Receiver

# &#x20;                                                                 ↓

# &#x20;                                                          WAL files on replica

# &#x20;                                                                 ↓

# &#x20;                                                          Startup Process (apply)

# &#x20;                                                                 ↓

# &#x20;                                                          Replica DB

# ```

# 

# \### Synchronous vs Asynchronous Replication

# 

# ```sql

# \-- PostgreSQL synchronous replication

# \-- Primary chờ replica confirm trước khi commit

# synchronous\_standby\_names = 'replica1'

# 

# \-- Các modes:

# synchronous\_commit = on          -- chờ replica WAL flush

# synchronous\_commit = remote\_write -- chờ replica nhận (không cần flush)

# synchronous\_commit = local        -- chỉ đảm bảo local WAL, async với replica

# synchronous\_commit = off          -- async hoàn toàn (có thể mất data nếu crash)

# ```

# 

# \*\*RPO (Recovery Point Objective):\*\*

# \- Sync replication: RPO = 0 (không mất data)

# \- Async replication: RPO > 0 (mất data trong replication lag window)

# 

# \### Replication Lag

# 

# ```sql

# \-- Kiểm tra replication lag (PostgreSQL)

# SELECT

# &#x20;   client\_addr,

# &#x20;   state,

# &#x20;   sent\_lsn,

# &#x20;   write\_lsn,

# &#x20;   flush\_lsn,

# &#x20;   replay\_lsn,

# &#x20;   sent\_lsn - replay\_lsn AS replication\_lag\_bytes,

# &#x20;   write\_lag,

# &#x20;   flush\_lag,

# &#x20;   replay\_lag

# FROM pg\_stat\_replication;

# 

# \-- Trên replica:

# SELECT now() - pg\_last\_xact\_replay\_timestamp() AS replication\_delay;

# ```

# 

# \*\*Nguyên nhân replication lag:\*\*

# \- Replica bị overloaded (disk I/O, CPU)

# \- Long-running queries trên replica block apply

# \- Network bandwidth

# 

# \---

# 

# \## 4.2 Sharding Strategies

# 

# \### Range-Based Sharding

# 

# ```

# Shard 0: user\_id \[1 → 1,000,000)

# Shard 1: user\_id \[1,000,000 → 2,000,000)

# Shard 2: user\_id \[2,000,000 → 3,000,000)

# ```

# 

# \*\*Ưu điểm:\*\*

# \- Range queries hiệu quả (theo user\_id range → biết ngay shard nào)

# \- Dễ split shard khi một shard quá lớn

# 

# \*\*Nhược điểm:\*\*

# \- \*\*Hot shard\*\*: New users luôn vào shard mới nhất → uneven load

# \- Cần rebalance khi thêm shard

# 

# \### Hash-Based Sharding

# 

# ```

# shard = hash(user\_id) % num\_shards

# 

# user\_id=1 → hash=7823 → 7823 % 4 = 3 → Shard 3

# user\_id=2 → hash=1234 → 1234 % 4 = 2 → Shard 2

# ```

# 

# \*\*Ưu điểm:\*\*

# \- Phân phối đều

# \- Không có hot shard

# 

# \*\*Nhược điểm:\*\*

# \- Range queries phải query tất cả shards

# \- Thêm/bớt shard → rehash tất cả data (giải pháp: \*\*Consistent Hashing\*\*)

# 

# \### Consistent Hashing

# 

# ```

# &#x20;               0°

# &#x20;              ───

# &#x20;        270° |   | 90°

# &#x20;              ───

# &#x20;              180°

# 

# Nodes và data đều được hash vào một vòng tròn:

# Node A = 30°

# Node B = 120°

# Node C = 240°

# 

# Key k1 = 80° → đi clockwise → Node B

# Key k2 = 150° → đi clockwise → Node C

# Key k3 = 290° → đi clockwise → Node A

# ```

# 

# Khi thêm Node D = 90°:

# \- Chỉ data từ 30°→90° (trước đây của B) cần chuyển sang D

# \- Phần còn lại không bị ảnh hưởng

# 

# \*\*Virtual Nodes (vnodes)\*\*: Mỗi physical node được map tới nhiều điểm trên ring → phân phối đều hơn.

# 

# \### Directory-Based Sharding

# 

# ```

# Lookup Table:

# ┌──────────────┬────────┐

# │ shard\_key    │ shard  │

# ├──────────────┼────────┤

# │ user\_id 1-50 │ Shard0 │

# │ user\_id 51-80│ Shard1 │

# │ VIP users    │ ShardV │

# └──────────────┴────────┘

# ```

# 

# \*\*Ưu điểm\*\*: Linh hoạt tuyệt đối, có thể custom mapping

# \*\*Nhược điểm\*\*: Lookup table là single point of failure, thêm latency

# 

# \---

# 

# \## 4.3 Cross-Shard Challenges

# 

# \### Cross-Shard Query

# 

# ```sql

# \-- BAD: Query data trên nhiều shards

# SELECT u.name, SUM(o.amount)

# FROM users u JOIN orders o ON u.id = o.user\_id

# GROUP BY u.id;

# \-- Cần query tất cả shards, aggregate ở application layer → chậm

# ```

# 

# \*\*Giải pháp:\*\*

# 1\. \*\*Denormalize\*\*: Copy user\_name vào orders table — tránh join cross-shard

# 2\. \*\*Fan-out query\*\*: Application query song song tất cả shards, merge kết quả

# 3\. \*\*Global tables\*\*: Bảng nhỏ, đọc nhiều (lookup tables) → replicate trên tất cả shards

# 

# \### Shard Key Selection — Quan trọng nhất

# 

# \*\*Tiêu chí chọn shard key:\*\*

# 

# 1\. \*\*Cardinality cao\*\* → phân bố đều

# 2\. \*\*Query locality\*\* → các query liên quan thường query cùng shard

# 3\. \*\*Không thay đổi\*\* → user\_id tốt hơn email (email có thể đổi)

# 4\. \*\*Tránh hotspot\*\* → timestamp thường tệ (mọi write vào cùng shard)

# 

# ```

# User-facing app → shard by user\_id

# Multi-tenant SaaS → shard by tenant\_id  

# E-commerce orders → shard by user\_id (không phải order\_id!)

# → Tất cả orders của user X trên cùng shard → query orders của user X hiệu quả

# ```

# 

# \### Global IDs (cross-shard unique IDs)

# 

# ```

# Auto-increment không dùng được (mỗi shard có ID riêng → conflict)

# 

# Giải pháp 1 — UUID:

# → 128-bit, globally unique, nhưng không sorted → index fragmentation

# 

# Giải pháp 2 — Snowflake ID (Twitter):

# ┌──────────────────┬────────────┬─────────────┐

# │   41 bits        │  10 bits   │   12 bits   │

# │   timestamp (ms) │  machine   │  sequence   │

# └──────────────────┴────────────┴─────────────┘

# → K-sorted (roughly time-ordered), 4096 IDs/ms/machine

# 

# Giải pháp 3 — DB sequence per shard với offset:

# Shard 0: 1, 4, 7, 10... (start=1, increment=3)

# Shard 1: 2, 5, 8, 11... (start=2, increment=3)

# Shard 2: 3, 6, 9, 12... (start=3, increment=3)

# ```

# 

# \---

# 

# \## 4.4 Read/Write Splitting

# 

# ```

# Application

# &#x20;   │

# &#x20;   ├── Write ──→ Primary DB

# &#x20;   │

# &#x20;   └── Read ───→ Load Balancer ──→ Replica 1

# &#x20;                                ──→ Replica 2

# &#x20;                                ──→ Replica 3

# ```

# 

# \*\*Vấn đề: Read-your-writes consistency\*\*

# ```python

# \# T1: User update profile

# db\_primary.execute("UPDATE users SET bio = 'new bio' WHERE id = 1")

# 

# \# T2: User reload page — đọc từ replica

# bio = db\_replica.fetchone("SELECT bio FROM users WHERE id = 1")

# \# Nếu replica lag 1s → user thấy bio cũ → confusing!

# ```

# 

# \*\*Giải pháp:\*\*

# 1\. Sau write, đọc từ primary trong N giây

# 2\. Dùng session-based routing: cùng session → cùng replica

# 3\. Ghi timestamp của write, replica check "đã apply đến timestamp này chưa"

# 

# \---

# 

# \# PHẦN 5: NoSQL DEEP DIVE

# 

# \## 5.1 Redis — Kiến Trúc \& Use Cases

# 

# \### Data Structures

# 

# ```redis

# \# String — cache, counter, session

# SET user:1:name "Nguyen Van An"

# INCR page:views:homepage          # atomic counter

# SETEX session:abc123 3600 "{...}" # TTL 1 giờ

# 

# \# Hash — object storage

# HSET user:1 name "An" email "an@example.com" age 25

# HGET user:1 email

# HGETALL user:1

# 

# \# List — queue, timeline, history

# LPUSH notifications:user:1 "New message"    # prepend

# RPUSH job\_queue "task\_payload"              # append

# BRPOP job\_queue 0                           # blocking pop (job worker)

# LRANGE timeline:user:1 0 19                 # 20 most recent

# 

# \# Set — unique items, tags, mutual friends

# SADD user:1:following 2 3 4 5

# SADD user:2:following 1 3 6 7

# SINTER user:1:following user:2:following    # mutual: {3}

# SUNION user:1:following user:2:following    # all: {1,2,3,4,5,6,7}

# 

# \# Sorted Set — leaderboard, priority queue

# ZADD leaderboard 1500 "user:1"

# ZADD leaderboard 2300 "user:2"

# ZADD leaderboard 1800 "user:3"

# ZREVRANGE leaderboard 0 9 WITHSCORES        # top 10

# ZRANK leaderboard "user:1"                  # rank của user 1

# 

# \# HyperLogLog — approximate distinct count, memory efficient

# PFADD unique\_visitors:2024-01-01 "user:1" "user:2" "user:3"

# PFCOUNT unique\_visitors:2024-01-01          # \~3 (approximate)

# 

# \# Stream — event log, message queue

# XADD events \* action "purchase" user\_id 123 amount 50000

# XREAD COUNT 10 STREAMS events 0             # read events

# XGROUP CREATE events processors $           # consumer group

# ```

# 

# \### Redis Persistence

# 

# \*\*RDB (Snapshotting):\*\*

# ```

# \# redis.conf

# save 900 1      # Sau 900s nếu ≥1 key thay đổi

# save 300 10     # Sau 300s nếu ≥10 keys thay đổi

# save 60 10000   # Sau 60s nếu ≥10000 keys thay đổi

# ```

# \- Fork process → dump data ra file .rdb

# \- Compact, nhanh khi restart

# \- Có thể mất data giữa 2 lần snapshot

# 

# \*\*AOF (Append-Only File):\*\*

# ```

# \# redis.conf

# appendonly yes

# appendfsync everysec    # flush mỗi giây (balance performance/durability)

# \# appendfsync always   # flush mỗi write (chậm nhất, an toàn nhất)

# \# appendfsync no       # OS quyết định (nhanh nhất, kém an toàn)

# ```

# \- Log mọi write command

# \- RPO tốt hơn RDB

# \- File lớn hơn, restart chậm hơn

# 

# \*\*Hybrid (Redis 4.0+):\*\* Dùng cả RDB + AOF:

# ```

# aof-use-rdb-preamble yes

# ```

# 

# \### Redis Cluster

# 

# ```

# 3 master + 3 replica:

# 

# Master 0 (slots 0-5460)       ← Replica 0

# Master 1 (slots 5461-10922)   ← Replica 1

# Master 2 (slots 10923-16383)  ← Replica 2

# 

# Total: 16384 hash slots

# slot = CRC16(key) % 16384

# ```

# 

# \*\*Multi-key operations:\*\*

# ```redis

# \# Keys trên cùng slot: dùng hash tags

# SET {user:1}.name "An"

# SET {user:1}.email "an@example.com"

# \# {user:1} → cùng slot → MGET hoạt động

# MGET {user:1}.name {user:1}.email

# ```

# 

# \### Cache Patterns

# 

# \*\*Cache-Aside (Lazy Loading):\*\*

# ```python

# def get\_user(user\_id):

# &#x20;   # 1. Check cache

# &#x20;   user = redis.get(f"user:{user\_id}")

# &#x20;   if user:

# &#x20;       return json.loads(user)

# &#x20;   

# &#x20;   # 2. Cache miss → load from DB

# &#x20;   user = db.query("SELECT \* FROM users WHERE id = %s", user\_id)

# &#x20;   

# &#x20;   # 3. Write to cache

# &#x20;   redis.setex(f"user:{user\_id}", 3600, json.dumps(user))

# &#x20;   return user

# ```

# 

# \*\*Write-Through:\*\*

# ```python

# def update\_user(user\_id, data):

# &#x20;   # Write to DB

# &#x20;   db.execute("UPDATE users SET ... WHERE id = %s", user\_id)

# &#x20;   # Immediately update cache

# &#x20;   redis.setex(f"user:{user\_id}", 3600, json.dumps(data))

# ```

# 

# \*\*Cache Stampede Prevention:\*\*

# ```python

# \# Vấn đề: Nhiều requests cùng cache miss → thundering herd

# \# Giải pháp: Mutex lock

# def get\_user\_with\_lock(user\_id):

# &#x20;   cache\_key = f"user:{user\_id}"

# &#x20;   lock\_key = f"lock:user:{user\_id}"

# &#x20;   

# &#x20;   user = redis.get(cache\_key)

# &#x20;   if user:

# &#x20;       return json.loads(user)

# &#x20;   

# &#x20;   # Try to acquire lock

# &#x20;   if redis.set(lock\_key, 1, nx=True, ex=10):  # nx=only if not exists

# &#x20;       try:

# &#x20;           user = db.query(...)

# &#x20;           redis.setex(cache\_key, 3600, json.dumps(user))

# &#x20;       finally:

# &#x20;           redis.delete(lock\_key)

# &#x20;   else:

# &#x20;       # Another worker is fetching, wait and retry

# &#x20;       time.sleep(0.1)

# &#x20;       return get\_user\_with\_lock(user\_id)  # retry

# &#x20;   

# &#x20;   return user

# ```

# 

# \---

# 

# \## 5.2 Cassandra — Wide Column Store

# 

# \### Data Model

# 

# ```

# Keyspace (\~ Database)

# &#x20; └── Table

# &#x20;       └── Partition (rows với cùng partition key)

# &#x20;             └── Rows (sorted by clustering key)

# ```

# 

# ```cql

# CREATE TABLE time\_series\_data (

# &#x20;   sensor\_id    UUID,

# &#x20;   recorded\_at  TIMESTAMP,

# &#x20;   temperature  FLOAT,

# &#x20;   humidity     FLOAT,

# &#x20;   PRIMARY KEY (sensor\_id, recorded\_at)  -- sensor\_id: partition key, recorded\_at: clustering key

# ) WITH CLUSTERING ORDER BY (recorded\_at DESC);

# 

# \-- Query hiệu quả: cùng partition

# SELECT \* FROM time\_series\_data

# WHERE sensor\_id = ? AND recorded\_at > ? AND recorded\_at < ?;

# ```

# 

# \### Consistent Hashing \& Replication

# 

# ```

# Token Ring:

# 

# &#x20;    Node A (token 0)

# &#x20;   /                \\

# Node D              Node B

# (token 270)        (token 90)

# &#x20;   \\                /

# &#x20;    Node C (token 180)

# 

# Replication Factor = 3:

# data với token 50 → lưu tại: Node B (90), Node C (180), Node D (270)

# ```

# 

# \### Tunable Consistency

# 

# ```cql

# \-- Write consistency

# INSERT INTO ... USING CONSISTENCY QUORUM;

# \-- QUORUM = majority of replicas phải acknowledge

# 

# \-- Read consistency

# SELECT ... FROM ... USING CONSISTENCY LOCAL\_ONE;

# \-- LOCAL\_ONE = 1 replica trong local datacenter phải respond

# 

# \-- Common levels:

# \-- ONE: 1 replica (fastest, weakest)

# \-- QUORUM: majority (n/2 + 1) — balance

# \-- ALL: tất cả (strongest, slowest)

# \-- LOCAL\_QUORUM: majority trong local DC (multi-DC setup)

# ```

# 

# \*\*Eventual Consistency mechanism:\*\*

# \- \*\*Read Repair\*\*: Khi đọc, coordinator so sánh versions, update replica lỗi thời

# \- \*\*Hinted Handoff\*\*: Nếu replica down, node khác giữ tạm updates, deliver khi replica up lại

# \- \*\*Anti-Entropy (Merkle Trees)\*\*: Background process sync data giữa replicas

# 

# \### Cassandra Write Path

# 

# ```

# 1\. Write → Commit Log (WAL) — durability

# 2\. Write → MemTable (in-memory) — fast write

# 3\. Khi MemTable đầy → flush → SSTable (immutable, sorted)

# 4\. Periodically → Compaction: merge SSTables, remove tombstones

# ```

# 

# \*\*Tombstones\*\*: Cassandra không xóa ngay, ghi tombstone marker:

# ```

# ⚠️ Too many tombstones → query chậm → cần monitor và compact

# ```

# 

# \---

# 

# \## 5.3 MongoDB — Document Store

# 

# \### Document Model

# 

# ```json

# {

# &#x20; "\_id": ObjectId("..."),

# &#x20; "user\_id": 123,

# &#x20; "status": "active",

# &#x20; "address": {

# &#x20;   "city": "HCM",

# &#x20;   "district": "Q1"

# &#x20; },

# &#x20; "tags": \["vip", "early\_adopter"],

# &#x20; "orders": \[

# &#x20;   {"id": 1, "amount": 150000, "date": ISODate("2024-01-01")},

# &#x20;   {"id": 2, "amount": 280000, "date": ISODate("2024-02-01")}

# &#x20; ]

# }

# ```

# 

# \### Aggregation Pipeline

# 

# ```javascript

# db.orders.aggregate(\[

# &#x20; // Stage 1: Filter

# &#x20; { $match: { status: "completed", created\_at: { $gte: ISODate("2024-01-01") } } },

# &#x20; 

# &#x20; // Stage 2: Join với users collection

# &#x20; { $lookup: {

# &#x20;     from: "users",

# &#x20;     localField: "user\_id",

# &#x20;     foreignField: "\_id",

# &#x20;     as: "user"

# &#x20; }},

# &#x20; 

# &#x20; // Stage 3: Unwind array

# &#x20; { $unwind: "$user" },

# &#x20; 

# &#x20; // Stage 4: Group và aggregate

# &#x20; { $group: {

# &#x20;     \_id: "$user.city",

# &#x20;     total\_revenue: { $sum: "$amount" },

# &#x20;     order\_count: { $count: {} },

# &#x20;     avg\_order: { $avg: "$amount" }

# &#x20; }},

# &#x20; 

# &#x20; // Stage 5: Sort

# &#x20; { $sort: { total\_revenue: -1 } },

# &#x20; 

# &#x20; // Stage 6: Limit

# &#x20; { $limit: 10 }

# ])

# ```

# 

# \### MongoDB Indexing

# 

# ```javascript

# // Compound index

# db.orders.createIndex({ user\_id: 1, created\_at: -1 })

# 

# // Partial index

# db.orders.createIndex(

# &#x20; { created\_at: 1 },

# &#x20; { partialFilterExpression: { status: "pending" } }

# )

# 

# // Text index

# db.articles.createIndex({ title: "text", content: "text" })

# db.articles.find({ $text: { $search: "database performance" } })

# 

# // Wildcard index (cho JSONB-like queries)

# db.products.createIndex({ "attributes.$\*\*": 1 })

# 

# // Explain

# db.orders.find({ user\_id: 123 }).explain("executionStats")

# ```

# 

# \### Transactions trong MongoDB (4.0+)

# 

# ```javascript

# // Multi-document ACID transactions

# const session = db.getMongo().startSession();

# session.startTransaction();

# try {

# &#x20; db.accounts.updateOne(

# &#x20;   { \_id: fromId },

# &#x20;   { $inc: { balance: -amount } },

# &#x20;   { session }

# &#x20; );

# &#x20; db.accounts.updateOne(

# &#x20;   { \_id: toId },

# &#x20;   { $inc: { balance: +amount } },

# &#x20;   { session }

# &#x20; );

# &#x20; session.commitTransaction();

# } catch (error) {

# &#x20; session.abortTransaction();

# &#x20; throw error;

# } finally {

# &#x20; session.endSession();

# }

# ```

# 

# \---

# 

# \# PHẦN 6: PERFORMANCE TUNING \& MONITORING

# 

# \## 6.1 PostgreSQL Configuration Tuning

# 

# \### Memory Settings

# 

# ```ini

# \# postgresql.conf

# 

# \# Shared buffer — cache cho PostgreSQL (25% RAM thường tốt)

# shared\_buffers = 4GB        # server 16GB RAM

# 

# \# Working memory per sort/hash operation

# \# Tổng memory = work\_mem × max\_connections × operations\_per\_query

# work\_mem = 64MB             # tăng nếu sort/hash spill to disk

# 

# \# Maintenance operations (VACUUM, CREATE INDEX)

# maintenance\_work\_mem = 512MB

# 

# \# Effective cache size — hint cho planner về OS cache size

# \# Không allocate thực, chỉ ảnh hưởng cost estimation

# effective\_cache\_size = 12GB  # \~75% RAM

# ```

# 

# \### Checkpoint \& WAL

# 

# ```ini

# \# Checkpoint frequency

# checkpoint\_completion\_target = 0.9  # spread checkpoint I/O

# checkpoint\_timeout = 15min          # max time between checkpoints

# max\_wal\_size = 4GB                  # trigger checkpoint khi WAL đạt size này

# 

# \# WAL configuration

# wal\_buffers = 64MB

# wal\_compression = on               # compress WAL, tốt cho replication

# ```

# 

# \### Connection Settings

# 

# ```ini

# max\_connections = 200              # Tổng connections

# \# Mỗi connection \~ 5-10MB RAM

# \# Nếu nhiều connections → dùng PgBouncer thay vì tăng max\_connections

# 

# \# Autovacuum tuning

# autovacuum\_vacuum\_scale\_factor = 0.1    # 10% dead tuples → vacuum

# autovacuum\_analyze\_scale\_factor = 0.05  # 5% changes → analyze

# autovacuum\_vacuum\_cost\_delay = 2ms      # tránh vacuum ảnh hưởng production

# ```

# 

# \---

# 

# \## 6.2 Slow Query Identification

# 

# \### PostgreSQL

# 

# ```sql

# \-- Enable slow query logging

# \-- postgresql.conf:

# log\_min\_duration\_statement = 1000  -- Log queries > 1 giây

# log\_statement = 'none'             -- Không log tất cả queries

# 

# \-- pg\_stat\_statements extension

# CREATE EXTENSION pg\_stat\_statements;

# 

# \-- Top 10 slowest queries

# SELECT

# &#x20;   round(total\_exec\_time / 1000 / 60, 2) AS total\_minutes,

# &#x20;   round(mean\_exec\_time::numeric, 2) AS mean\_ms,

# &#x20;   calls,

# &#x20;   round(stddev\_exec\_time::numeric, 2) AS stddev\_ms,

# &#x20;   rows,

# &#x20;   query

# FROM pg\_stat\_statements

# ORDER BY total\_exec\_time DESC

# LIMIT 10;

# 

# \-- Queries với high I/O

# SELECT

# &#x20;   calls,

# &#x20;   round(total\_exec\_time / calls, 2) AS avg\_ms,

# &#x20;   shared\_blks\_hit,

# &#x20;   shared\_blks\_read,

# &#x20;   round(shared\_blks\_hit::numeric / nullif(shared\_blks\_hit + shared\_blks\_read, 0) \* 100, 2) AS hit\_rate\_pct,

# &#x20;   query

# FROM pg\_stat\_statements

# WHERE shared\_blks\_read > 1000

# ORDER BY shared\_blks\_read DESC;

# ```

# 

# \### MySQL

# 

# ```sql

# \-- Slow query log

# SET GLOBAL slow\_query\_log = ON;

# SET GLOBAL long\_query\_time = 1;  -- queries > 1s

# SET GLOBAL log\_queries\_not\_using\_indexes = ON;

# 

# \-- Performance Schema

# SELECT

# &#x20;   DIGEST\_TEXT,

# &#x20;   COUNT\_STAR,

# &#x20;   round(SUM\_TIMER\_WAIT / 1e12, 2) AS total\_seconds,

# &#x20;   round(AVG\_TIMER\_WAIT / 1e9, 2) AS avg\_ms,

# &#x20;   SUM\_ROWS\_EXAMINED,

# &#x20;   SUM\_ROWS\_SENT

# FROM performance\_schema.events\_statements\_summary\_by\_digest

# ORDER BY SUM\_TIMER\_WAIT DESC

# LIMIT 10;

# ```

# 

# \---

# 

# \## 6.3 Key Metrics để Monitor

# 

# \### Database Health Metrics

# 

# ```sql

# \-- PostgreSQL: Cache hit rate (nên > 99%)

# SELECT

# &#x20;   sum(heap\_blks\_hit) / nullif(sum(heap\_blks\_hit) + sum(heap\_blks\_read), 0) AS table\_hit\_rate,

# &#x20;   sum(idx\_blks\_hit) / nullif(sum(idx\_blks\_hit) + sum(idx\_blks\_read), 0) AS index\_hit\_rate

# FROM pg\_statio\_user\_tables;

# 

# \-- Connection utilization

# SELECT count(\*), state FROM pg\_stat\_activity GROUP BY state;

# 

# \-- Table bloat (dead tuples)

# SELECT

# &#x20;   schemaname,

# &#x20;   tablename,

# &#x20;   pg\_size\_pretty(pg\_total\_relation\_size(schemaname||'.'||tablename)) AS total\_size,

# &#x20;   n\_dead\_tup,

# &#x20;   n\_live\_tup,

# &#x20;   round(n\_dead\_tup::numeric / nullif(n\_live\_tup, 0) \* 100, 2) AS dead\_ratio

# FROM pg\_stat\_user\_tables

# WHERE n\_live\_tup > 10000

# ORDER BY n\_dead\_tup DESC;

# 

# \-- Index usage (unused indexes — overhead không cần thiết)

# SELECT

# &#x20;   schemaname,

# &#x20;   tablename,

# &#x20;   indexname,

# &#x20;   idx\_scan,

# &#x20;   pg\_size\_pretty(pg\_relation\_size(indexrelid)) AS index\_size

# FROM pg\_stat\_user\_indexes

# WHERE idx\_scan = 0

# &#x20; AND indexrelid NOT IN (SELECT conindid FROM pg\_constraint)  -- exclude constraint indexes

# ORDER BY pg\_relation\_size(indexrelid) DESC;

# 

# \-- Lock waits

# SELECT

# &#x20;   blocked\_locks.pid AS blocked\_pid,

# &#x20;   blocking\_locks.pid AS blocking\_pid,

# &#x20;   blocked\_activity.query AS blocked\_query,

# &#x20;   blocking\_activity.query AS blocking\_query,

# &#x20;   now() - blocked\_activity.query\_start AS wait\_duration

# FROM pg\_catalog.pg\_locks blocked\_locks

# JOIN pg\_catalog.pg\_locks blocking\_locks

# &#x20;   ON blocking\_locks.locktype = blocked\_locks.locktype

# &#x20;   AND blocking\_locks.relation = blocked\_locks.relation

# &#x20;   AND blocking\_locks.granted = true

# &#x20;   AND blocked\_locks.granted = false

# JOIN pg\_catalog.pg\_stat\_activity blocked\_activity ON blocked\_activity.pid = blocked\_locks.pid

# JOIN pg\_catalog.pg\_stat\_activity blocking\_activity ON blocking\_activity.pid = blocking\_locks.pid;

# ```

# 

# \### Golden Signals (4 metrics quan trọng nhất)

# 

# | Signal | Metric | Tool |

# |---|---|---|

# | \*\*Latency\*\* | Query response time (p50, p95, p99) | pg\_stat\_statements, slow query log |

# | \*\*Traffic\*\* | Queries per second (QPS), TPS | pg\_stat\_database |

# | \*\*Errors\*\* | Failed transactions, deadlocks | pg\_stat\_database.xact\_rollback |

# | \*\*Saturation\*\* | CPU, memory, disk I/O, connections | pg\_stat\_bgwriter, OS metrics |

# 

# \---

# 

# \## 6.4 Query Optimization Checklist

# 

# ```

# □ 1. Chạy EXPLAIN ANALYZE — hiểu execution plan

# □ 2. Tìm Seq Scan trên bảng lớn — thêm index

# □ 3. Kiểm tra row estimation vs actual — nếu sai nhiều → ANALYZE

# □ 4. Kiểm tra N+1 queries trong application logs

# □ 5. Xem buffer read cao — cache miss, cần tăng shared\_buffers

# □ 6. Sort/Hash spill to disk — tăng work\_mem

# □ 7. Index scan nhưng filter nhiều — cần composite/partial index tốt hơn

# □ 8. Kiểm tra unused indexes — drop để giảm write overhead

# □ 9. Kiểm tra table bloat — VACUUM nếu cần

# □ 10. Connection pool đủ chưa — tránh connection exhaustion

# ```

# 

# \---

# 

# \## 6.5 Connection Pooling — PgBouncer

# 

# ```ini

# \# pgbouncer.ini

# \[databases]

# mydb = host=127.0.0.1 port=5432 dbname=mydb

# 

# \[pgbouncer]

# pool\_mode = transaction       # transaction-level pooling (hiệu quả nhất)

# max\_client\_conn = 1000        # max connections từ app

# default\_pool\_size = 20        # connections thực tới PostgreSQL

# min\_pool\_size = 5

# reserve\_pool\_size = 5

# server\_idle\_timeout = 600

# ```

# 

# \*\*Pool modes:\*\*

# \- \*\*session\*\*: 1 server connection per client session (ít hiệu quả nhất, tương thích cao nhất)

# \- \*\*transaction\*\*: Trả lại connection sau mỗi transaction (phổ biến nhất)

# \- \*\*statement\*\*: Trả lại sau mỗi statement (không hỗ trợ multi-statement transactions)

# 

# ```

# Không có PgBouncer:

# 1000 app connections → 1000 PostgreSQL connections → 5-10GB RAM chỉ cho connections

# 

# Với PgBouncer (transaction mode):

# 1000 app connections → 20 PostgreSQL connections → tiết kiệm RAM, tốt hơn nhiều

# ```

# 

# \---

# 

# \## 6.6 Partitioning (Table Partitioning)

# 

# ```sql

# \-- Range Partitioning theo thời gian

# CREATE TABLE events (

# &#x20;   id          BIGSERIAL,

# &#x20;   user\_id     INT,

# &#x20;   event\_type  VARCHAR(50),

# &#x20;   created\_at  TIMESTAMP NOT NULL,

# &#x20;   data        JSONB

# ) PARTITION BY RANGE (created\_at);

# 

# \-- Tạo partitions theo tháng

# CREATE TABLE events\_2024\_01 PARTITION OF events

# &#x20;   FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');

# CREATE TABLE events\_2024\_02 PARTITION OF events

# &#x20;   FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');

# 

# \-- Mỗi partition có index riêng

# CREATE INDEX ON events\_2024\_01(user\_id);

# CREATE INDEX ON events\_2024\_02(user\_id);

# 

# \-- Query tự động route tới partition đúng (partition pruning)

# EXPLAIN SELECT \* FROM events

# WHERE created\_at BETWEEN '2024-01-15' AND '2024-01-20';

# \-- → Chỉ scan events\_2024\_01, không đụng vào partitions khác

# ```

# 

# \*\*Lợi ích:\*\*

# \- \*\*Partition pruning\*\*: Query chỉ scan partitions cần thiết

# \- \*\*Partition-wise\*\*: DROP partition cũ thay vì DELETE → cực nhanh

# \- Mỗi partition nhỏ hơn → index nhỏ hơn → fit vào memory tốt hơn

# \- Autovacuum hiệu quả hơn trên partitions nhỏ

# 

# \---

# 

# \## 6.7 Tóm Tắt Performance Tuning Hierarchy

# 

# ```

# 1\. Application Level (quan trọng nhất, dễ fix nhất)

# &#x20;  ├── Loại bỏ N+1 queries

# &#x20;  ├── Cache với Redis

# &#x20;  ├── Connection pooling

# &#x20;  └── Batch operations thay vì individual queries

# 

# 2\. Query Level

# &#x20;  ├── EXPLAIN ANALYZE → tìm bottleneck

# &#x20;  ├── Thêm/optimize index

# &#x20;  ├── Rewrite queries (tránh function on columns, etc.)

# &#x20;  └── Denormalize khi cần

# 

# 3\. Schema Level

# &#x20;  ├── Partitioning cho bảng lớn

# &#x20;  ├── Archiving data cũ

# &#x20;  └── Normalize/denormalize có chủ đích

# 

# 4\. Database Configuration

# &#x20;  ├── Memory (shared\_buffers, work\_mem)

# &#x20;  ├── Checkpoint \& WAL settings

# &#x20;  └── Autovacuum tuning

# 

# 5\. Infrastructure Level (đắt nhất, cuối cùng)

# &#x20;  ├── Read replicas

# &#x20;  ├── Faster storage (NVMe SSD)

# &#x20;  ├── More RAM

# &#x20;  └── Sharding

# ```

