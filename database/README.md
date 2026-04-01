# 🗄️ Database Deep Dive — Toàn Tập
 
---
 
# PHẦN 1: INDEXING — Cơ Chế Bên Trong
 
## 1.1 B-Tree Internals
 
### Cấu trúc thực sự của B-Tree
 
B-Tree (balanced tree) trong database là **B+ Tree** — tất cả data nằm ở leaf nodes, internal nodes chỉ chứa key để điều hướng.
 
```
                        [Internal Node]
                        | 30 | 60 |
                       /     |     \
              [10|20]      [40|50]      [70|80]     ← Internal Nodes
             /  |   \      /  |  \      /  |  \
           [5] [15] [25] [35][45][55] [65][75][85]  ← Leaf Nodes
            ↕   ↕   ↕    ↕   ↕   ↕    ↕   ↕   ↕
           (data pointers — trỏ tới heap/row)
           
           Leaf nodes được nối với nhau bằng linked list:
           [5] → [15] → [25] → [35] → ... → [85]
```
 
**Tại sao dùng B+ Tree thay vì BST?**
- BST có thể unbalanced → O(n) worst case
- B+ Tree luôn balanced → O(log n) guaranteed
- Leaf nodes linked list → range scan rất hiệu quả (không cần traverse tree)
- **Page size** phù hợp với disk I/O (thường 8KB hay 16KB) → đọc 1 lần được nhiều key
 
### Chi phí duy trì Index
 
Mỗi lần INSERT/UPDATE/DELETE:
1. Tìm vị trí đúng trong B-Tree: O(log n)
2. Insert key vào leaf node
3. Nếu node đầy → **page split**: chia đôi node, đẩy middle key lên parent
4. Page split lan truyền lên có thể khiến cả tree phải rebalance
 
```
Trước split:          Sau khi insert 28 vào [20|25|27] (full):
                      
[20 | 25 | 27]   →   [20 | 25]  [27 | 28]
                               ↑
                          25 được đẩy lên parent
```
 
> 💡 **Ý nghĩa thực tế**: Bảng insert nhiều → index bị fragmented → cần `REINDEX` hoặc `VACUUM` định kỳ.
 
### Fill Factor
PostgreSQL cho phép set fill factor (mặc định 90%):
```sql
CREATE INDEX idx_orders ON orders(created_at) WITH (fillfactor = 70);
```
- Fill factor 70% → mỗi leaf page chỉ lấp đầy 70%, 30% dự phòng cho insert
- Giảm page split → write nhanh hơn
- Trade-off: index lớn hơn, dùng nhiều disk hơn
 
---
 
## 1.2 Index Types Chi Tiết
 
### Hash Index
 
```
Key: "nguyen@email.com"
        ↓
   Hash Function
        ↓
   bucket_id = 42
        ↓
   [bucket 42] → [(key, row_pointer), ...]
```
 
- Lookup O(1) — tốt cho `=` exact match
- **Không** hỗ trợ: range query, ORDER BY, LIKE
- PostgreSQL: Hash index không được WAL-logged trước v10 (mất khi crash)
- MySQL Memory engine dùng hash index mặc định
 
### GiST Index (Generalized Search Tree)
Dùng cho các kiểu dữ liệu phức tạp:
```sql
-- Geometric data
CREATE INDEX ON places USING gist(location);
SELECT * FROM places WHERE location <-> point(10.8, 106.7) < 5; -- trong 5km
 
-- Range types
CREATE INDEX ON reservations USING gist(during);
SELECT * FROM reservations WHERE during && '[2024-01-01, 2024-01-07]'::daterange;
```
 
### GIN Index (Generalized Inverted Index)
Dùng cho array, jsonb, full-text search:
```sql
-- JSONB
CREATE INDEX ON products USING gin(attributes);
SELECT * FROM products WHERE attributes @> '{"color": "red"}';
 
-- Array
CREATE INDEX ON articles USING gin(tags);
SELECT * FROM articles WHERE tags @> ARRAY['database', 'performance'];
 
-- Full-text search
CREATE INDEX ON docs USING gin(to_tsvector('english', content));
```
 
**GIN vs GiST:**
| | GIN | GiST |
|---|---|---|
| Build time | Chậm | Nhanh |
| Index size | Lớn hơn | Nhỏ hơn |
| Lookup | Nhanh hơn | Chậm hơn |
| Update | Chậm (pending list) | Nhanh |
 
### BRIN Index (Block Range Index)
Dùng cho dữ liệu **tự nhiên có thứ tự theo thời gian** (time-series, log):
```sql
CREATE INDEX ON events USING brin(created_at);
```
- Lưu min/max của mỗi block (thường 128 pages)
- Index rất nhỏ (vài KB so với GB data)
- Hiệu quả nếu dữ liệu insert theo thứ tự thời gian
- Kém hiệu quả nếu dữ liệu random
 
---
 
## 1.3 Index Strategies Nâng Cao
 
### Index Merge (MySQL)
MySQL có thể **combine nhiều index** cho 1 query:
```sql
-- Có index trên (status) và index trên (user_id) riêng lẻ
SELECT * FROM orders WHERE status = 'pending' OR user_id = 123;
-- MySQL dùng: index merge union của 2 index
```
 
Nhưng thường **composite index tốt hơn index merge**:
```sql
-- Tốt hơn: composite index
CREATE INDEX ON orders(user_id, status);
```
 
### Index Selectivity
```sql
-- Kiểm tra selectivity (PostgreSQL)
SELECT
    attname,
    n_distinct,
    correlation  -- 1.0 = hoàn toàn sorted, 0 = random
FROM pg_stats
WHERE tablename = 'orders';
```
 
**Selectivity = số distinct values / tổng rows**
- Gần 1.0 → high selectivity → index rất hiệu quả (user_id, email)
- Gần 0 → low selectivity → index ít hiệu quả (status: 'active'/'inactive')
 
Quy tắc: Nếu query trả về > 5-10% rows, DB thường prefer seq scan hơn index scan.
 
### Index Condition Pushdown (ICP) — MySQL
```sql
-- Index trên (last_name, first_name)
SELECT * FROM users WHERE last_name LIKE 'Ng%' AND first_name LIKE 'An%';
```
Không có ICP: index tìm theo last_name, fetch row, rồi mới check first_name  
Có ICP: check cả first_name ngay tại index level → giảm row fetch
 
### Invisible Index (MySQL 8+)
```sql
-- Test xem drop index có ảnh hưởng gì không
ALTER TABLE orders ALTER INDEX idx_status INVISIBLE;
-- Query optimizer bỏ qua index này
-- Nếu performance không đổi → có thể drop an toàn
ALTER TABLE orders ALTER INDEX idx_status VISIBLE;
```
 
---
 
# PHẦN 2: TRANSACTION & LOCKING
 
## 2.1 Locking — Cơ Chế Chi Tiết
 
### Lock Granularity (từ thô → tinh)
 
```
Database Lock
    └── Table Lock
            └── Page Lock
                    └── Row Lock
                            └── Column Lock (hiếm)
```
 
**Trade-off**: Lock càng tinh → concurrency càng cao, overhead quản lý lock càng lớn.
 
### Lock Modes
 
| Lock | Ký hiệu | Tương thích với | Dùng khi |
|---|---|---|---|
| **Access Share** | AS | Tất cả trừ ACCESS EXCLUSIVE | SELECT |
| **Row Share** | RS | Tất cả trừ EXCLUSIVE, ACCESS EXCLUSIVE | SELECT FOR UPDATE |
| **Row Exclusive** | RX | AS, RS, RX | INSERT, UPDATE, DELETE |
| **Share Update Exclusive** | SUX | AS | VACUUM, CREATE INDEX CONCURRENTLY |
| **Share** | S | AS, S | CREATE INDEX (non-concurrent) |
| **Share Row Exclusive** | SRX | AS | Trigger operations |
| **Exclusive** | X | AS | Hiếm dùng trực tiếp |
| **Access Exclusive** | AX | Không có | ALTER TABLE, DROP TABLE |
 
### SELECT FOR UPDATE vs SELECT FOR SHARE
 
```sql
-- FOR UPDATE: lock row, không ai UPDATE/DELETE/SELECT FOR UPDATE được
BEGIN;
SELECT balance FROM accounts WHERE id = 1 FOR UPDATE;
-- Row bị lock → transaction khác phải chờ
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
COMMIT;
 
-- FOR SHARE: lock row, cho phép SELECT khác nhưng không cho UPDATE
SELECT * FROM orders WHERE id = 1 FOR SHARE;
 
-- SKIP LOCKED: bỏ qua row đang bị lock (dùng cho job queue)
SELECT * FROM jobs WHERE status = 'pending'
ORDER BY created_at
FOR UPDATE SKIP LOCKED
LIMIT 1;
 
-- NOWAIT: không chờ, raise error ngay nếu bị lock
SELECT * FROM inventory WHERE id = 1 FOR UPDATE NOWAIT;
```
 
### SKIP LOCKED — Pattern Job Queue
```sql
-- Worker pattern: mỗi worker lấy 1 job không bị worker khác lấy
BEGIN;
SELECT id, payload FROM job_queue
WHERE status = 'pending'
ORDER BY priority DESC, created_at ASC
FOR UPDATE SKIP LOCKED
LIMIT 1;
 
-- Process job...
UPDATE job_queue SET status = 'done' WHERE id = $1;
COMMIT;
```
 
---
 
## 2.2 Deadlock
 
### Deadlock xảy ra thế nào
 
```
Transaction T1                    Transaction T2
──────────────                    ──────────────
LOCK row A (success)              LOCK row B (success)
LOCK row B (waiting...)           LOCK row A (waiting...)
         ↑                                 ↑
         └─────────── DEADLOCK ────────────┘
```
 
**Ví dụ thực tế — chuyển tiền:**
```sql
-- T1: Chuyển từ account 1 → account 2
UPDATE accounts SET balance = balance - 100 WHERE id = 1; -- lock row 1
UPDATE accounts SET balance = balance + 100 WHERE id = 2; -- chờ lock row 2
 
-- T2: Chuyển từ account 2 → account 1 (đồng thời)
UPDATE accounts SET balance = balance - 50 WHERE id = 2;  -- lock row 2
UPDATE accounts SET balance = balance + 50 WHERE id = 1;  -- chờ lock row 1
 
-- → DEADLOCK
```
 
### Phát hiện và xử lý Deadlock
DB tự detect deadlock bằng **wait-for graph**:
```
T1 → waits for → T2
T2 → waits for → T1
→ Cycle detected → victim (T1 hoặc T2) bị abort và rollback
```
 
**Cách phòng tránh deadlock:**
 
1. **Lock theo thứ tự cố định:**
```sql
-- Luôn lock account ID nhỏ hơn trước
BEGIN;
SELECT * FROM accounts WHERE id IN (1, 2) ORDER BY id FOR UPDATE;
-- Bây giờ T1 và T2 đều cố lock row 1 trước → không deadlock
```
 
2. **Giữ transaction ngắn nhất có thể:**
```sql
-- BAD: Xử lý nghiệp vụ phức tạp trong transaction
BEGIN;
SELECT ...   -- lock
-- 2 giây xử lý...
UPDATE ...
COMMIT;
 
-- GOOD: Chuẩn bị xong rồi mới open transaction
-- Xử lý nghiệp vụ...
BEGIN;
UPDATE ...   -- nhanh gọn
COMMIT;
```
 
3. **Lock ở mức thấp nhất cần thiết:** Dùng row lock thay vì table lock.
 
4. **Retry logic trong application:**
```python
MAX_RETRIES = 3
for attempt in range(MAX_RETRIES):
    try:
        with db.transaction():
            transfer(from_id, to_id, amount)
        break
    except DeadlockError:
        if attempt == MAX_RETRIES - 1:
            raise
        time.sleep(0.1 * (2 ** attempt))  # exponential backoff
```
 
---
 
## 2.3 MVCC — Multi-Version Concurrency Control
 
### Ý tưởng cốt lõi
Thay vì lock khi đọc, DB **giữ nhiều version** của mỗi row. Reader thấy snapshot tại thời điểm transaction bắt đầu, writer tạo version mới.
 
**"Readers don't block writers, writers don't block readers"**
 
### PostgreSQL MVCC Implementation
 
```
Mỗi row có hidden columns:
┌──────────┬──────────┬────────────┬────────────┐
│  xmin    │  xmax    │    data    │   ctid     │
├──────────┼──────────┼────────────┼────────────┤
│ txid tạo │ txid xóa │ actual data│ physical   │
│          │ (0=alive)│            │ location   │
└──────────┴──────────┴────────────┴────────────┘
```
 
**Luồng UPDATE trong PostgreSQL:**
```
Row gốc:    xmin=100, xmax=0,   data="old value"
                ↓
Transaction 200 UPDATE:
Row cũ:     xmin=100, xmax=200, data="old value"   ← mark deleted
Row mới:    xmin=200, xmax=0,   data="new value"   ← new version
```
 
Transaction với txid=150 (bắt đầu trước T200) vẫn thấy "old value" vì xmin=100 < 150 < xmax=200.
 
### Visibility Rules
Row R visible với transaction T khi:
- `R.xmin < T.txid` (row được tạo trước T)
- `R.xmin` đã committed
- `R.xmax = 0` HOẶC `R.xmax > T.txid` (row chưa bị xóa, hoặc bị xóa sau T)
- `R.xmax` chưa committed (nếu đang xóa)
 
### MVCC và vấn đề Table Bloat
PostgreSQL không xóa row ngay → **dead tuples** tích tụ:
```sql
-- Kiểm tra dead tuples
SELECT relname, n_dead_tup, n_live_tup,
       round(n_dead_tup::numeric / nullif(n_live_tup + n_dead_tup, 0) * 100, 2) AS dead_pct
FROM pg_stat_user_tables
ORDER BY n_dead_tup DESC;
 
-- VACUUM reclaim space từ dead tuples
VACUUM orders;
VACUUM FULL orders;  -- Aggressive, lock table, reclaim disk space
AUTOVACUUM tự chạy nhưng có thể cần tune
```
 
### Transaction Snapshot
```sql
-- Xem snapshot hiện tại
SELECT pg_current_snapshot();
-- Kết quả: 100:105:100,102
-- Nghĩa: xmin=100, xmax=105, in-progress={100,102}
-- → Transactions 100 và 102 đang chạy, chưa visible
```
 
---
 
## 2.4 Two-Phase Locking (2PL)
 
Đảm bảo serializability:
1. **Growing phase**: Transaction chỉ acquire lock, không release
2. **Shrinking phase**: Transaction chỉ release lock, không acquire thêm
 
```
Growing Phase          Shrinking Phase
──────────────────     ─────────────────
Acquire lock A    │    Release lock A
Acquire lock B    │    Release lock B
Acquire lock C    │    Release lock C
                  ↑
             Lock Point (max locks held)
```
 
**Strict 2PL** (S2PL): Giữ tất cả locks đến khi COMMIT/ROLLBACK — phổ biến hơn, tránh cascading rollback.
 
---
 
## 2.5 Distributed Transactions — Two-Phase Commit (2PC)
 
Khi transaction span nhiều database nodes:
 
```
Phase 1 — Prepare:
Coordinator → "Prepare to commit?" → Node A, Node B, Node C
Node A → "Ready" ✓
Node B → "Ready" ✓
Node C → "Ready" ✓
 
Phase 2 — Commit:
Coordinator → "Commit!" → Node A, Node B, Node C
```
 
**Vấn đề của 2PC:**
- **Blocking protocol**: Nếu coordinator crash sau Phase 1, các nodes bị treo (không thể tự quyết)
- **Performance**: 2 round trips qua network → latency cao
- Giải pháp hiện đại: **Saga pattern** (choreography/orchestration) thay vì 2PC
 
---
 
# PHẦN 3: QUERY OPTIMIZER & EXECUTION PLAN
 
## 3.1 Query Lifecycle
 
```
SQL Text
   ↓
[Parser] — Kiểm tra syntax, tạo parse tree
   ↓
[Analyzer/Rewriter] — Kiểm tra semantic, resolve tên bảng/cột, expand views
   ↓
[Planner/Optimizer] — Tạo execution plan tối ưu
   ↓
[Executor] — Thực thi plan
   ↓
Result
```
 
## 3.2 Cost-Based Optimizer
 
Optimizer tạo nhiều plan, ước tính cost của mỗi plan, chọn plan có cost thấp nhất.
 
### Statistics — Nền tảng của Optimizer
 
```sql
-- PostgreSQL thu thập statistics
ANALYZE orders;
 
-- Xem statistics
SELECT *
FROM pg_stats
WHERE tablename = 'orders' AND attname = 'status';
```
 
Statistics bao gồm:
- `n_distinct`: số distinct values (-1 = unique)
- `most_common_vals`: top N values phổ biến nhất
- `most_common_freqs`: frequency của các common values
- `histogram_bounds`: phân phối data
- `correlation`: mức độ sorted của cột so với physical order
 
### Ước tính Selectivity
 
```sql
-- Query: WHERE status = 'pending'
-- most_common_vals = {active, pending, done}
-- most_common_freqs = {0.6, 0.3, 0.1}
-- → Selectivity = 0.3 → 30% rows sẽ được trả về
-- → Nếu bảng có 100,000 rows → estimated 30,000 rows
```
 
### Tại sao Optimizer sai?
1. **Statistics cũ** (chưa ANALYZE sau bulk insert)
2. **Correlated columns** (city và zip_code tương quan nhau nhưng optimizer coi là độc lập)
3. **Non-uniform distribution** (99% rows có status='active', 1% khác)
4. **High row estimation error** → sai join order → plan tệ
 
```sql
-- Force statistics update
ANALYZE VERBOSE orders;
 
-- Extended statistics cho correlated columns (PG 10+)
CREATE STATISTICS stats_city_zip ON city, zip_code FROM users;
ANALYZE users;
```
 
---
 
## 3.3 Join Algorithms Chi Tiết
 
### Nested Loop Join
```
FOR each row r1 IN outer_table:
    FOR each row r2 IN inner_table:
        IF r1.key == r2.key:
            output(r1, r2)
```
- **Complexity**: O(n × m)
- **Tốt khi**: outer table nhỏ, inner table có index
- **Memory**: O(1) — không cần buffer
 
### Hash Join
```
Phase 1 — Build:
  FOR each row r IN smaller_table:
      hash_table[hash(r.key)] = r
 
Phase 2 — Probe:
  FOR each row r IN larger_table:
      IF hash_table[hash(r.key)] exists:
          output(match)
```
- **Complexity**: O(n + m)
- **Tốt khi**: không có useful index, large tables
- **Memory**: O(smaller_table) — cần RAM để hold hash table
- **Grace Hash Join**: Khi hash table không vừa RAM → spill to disk, chia thành batches
 
### Sort-Merge Join
```
Phase 1 — Sort: Sort cả 2 tables theo join key
Phase 2 — Merge: Merge 2 sorted streams như merge sort
```
- **Complexity**: O(n log n + m log m) nếu chưa sorted
- **Complexity**: O(n + m) nếu đã sorted
- **Tốt khi**: Data đã sorted (index), range joins, non-equi joins
 
### Khi nào dùng join nào?
 
| Tình huống | Join được chọn |
|---|---|
| Outer nhỏ + inner có index | Nested Loop |
| Cả 2 lớn, không có index | Hash Join |
| Cả 2 đã sorted / có sorted index | Merge Join |
| Nhiều rows trùng join key | Hash Join |
| `<`, `>`, range conditions | Merge Join hoặc Nested Loop |
 
---
 
## 3.4 Đọc EXPLAIN ANALYZE Chi Tiết
 
```sql
EXPLAIN (ANALYZE, BUFFERS, FORMAT TEXT)
SELECT u.name, COUNT(o.id)
FROM users u
JOIN orders o ON u.id = o.user_id
WHERE u.created_at > '2024-01-01'
GROUP BY u.id, u.name;
```
 
**Kết quả mẫu:**
```
HashAggregate  (cost=1250.00..1280.00 rows=3000 width=40)
               (actual time=45.123..47.456 rows=2800 loops=1)
  Buffers: shared hit=850 read=120
  ->  Hash Join  (cost=450.00..1150.00 rows=20000 width=16)
                 (actual time=12.345..38.901 rows=19500 loops=1)
        Hash Cond: (o.user_id = u.id)
        Buffers: shared hit=800 read=120
        ->  Seq Scan on orders o
              (cost=0.00..600.00 rows=50000 width=8)
              (actual time=0.012..15.234 rows=50000 loops=1)
              Buffers: shared hit=400
        ->  Hash  (cost=400.00..400.00 rows=4000 width=12)
                  (actual time=12.100..12.100 rows=3800 loops=1)
              Buckets: 4096  Batches: 1  Memory Usage: 256kB
              ->  Index Scan using idx_users_created on users u
                    (cost=0.00..400.00 rows=4000 width=12)
                    (actual time=0.021..10.234 rows=3800 loops=1)
                    Index Cond: (created_at > '2024-01-01')
                    Buffers: shared hit=400 read=120
```
 
### Giải thích từng phần
 
**Cost: `cost=start_cost..total_cost`**
- `start_cost`: Chi phí trước khi trả row đầu tiên (sorting, hashing)
- `total_cost`: Tổng chi phí để trả tất cả rows
- Đơn vị tương đối, không phải milliseconds
 
**Rows estimation vs actual:**
- `rows=3000` (estimate) vs `rows=2800` (actual) → sai lệch ~7% → tốt
- Nếu sai lệch > 10x → vấn đề statistics → cần ANALYZE
 
**Buffers:**
- `shared hit=850`: 850 pages đọc từ cache (shared_buffers) → tốt
- `shared read=120`: 120 pages phải đọc từ disk → I/O xảy ra
- `shared written`: pages dirty được flush → cẩn thận
 
**Loops:**
- `loops=1`: Node chạy 1 lần
- `loops=1000`: Node chạy 1000 lần (Nested Loop inner) → nhân actual time với loops
 
### Red Flags trong Execution Plan
 
```
❌ Seq Scan trên bảng lớn → cần index
❌ rows estimate << actual rows (10x+) → statistics lỗi thời
❌ Hash Batches > 1 → hash table spill to disk → tăng work_mem
❌ Nested Loop với loops rất lớn → join order sai
❌ Sort trên large dataset → có thể cần index để avoid sort
❌ Buffer read rất cao → cache miss, I/O bound
```
 
### Hints (khi Optimizer sai)
 
```sql
-- PostgreSQL: không có official hints, dùng config workaround
SET enable_seqscan = off;  -- Force index scan
SET enable_hashjoin = off; -- Force merge/nested join
SET enable_nestloop = off;
 
-- MySQL: hints chính thức
SELECT /*+ USE_INDEX(orders idx_status) */ * FROM orders;
SELECT /*+ NO_HASH_JOIN(u o) */ u.*, o.* FROM users u JOIN orders o;
 
-- Reset về mặc định
RESET enable_seqscan;
```
 
---
 
## 3.5 Common Table Expressions (CTE) và Performance
 
```sql
-- CTE mặc định trong PostgreSQL < 12: optimization fence
-- Optimizer KHÔNG thể push conditions vào trong CTE
WITH active_users AS (
    SELECT * FROM users WHERE status = 'active'
)
SELECT * FROM active_users WHERE city = 'HCM';
-- PG < 12: materialize toàn bộ active_users TRƯỚC, rồi mới filter city
-- PG >= 12: inline CTE, optimize như subquery
 
-- Force materialization (khi muốn CTE chạy 1 lần, dùng nhiều chỗ)
WITH MATERIALIZED expensive_calc AS (
    SELECT user_id, complex_calculation(data) AS result
    FROM raw_data
)
SELECT * FROM expensive_calc WHERE ...
UNION ALL
SELECT * FROM expensive_calc WHERE ...;
 
-- Force inline (PG >= 12)
WITH NOT MATERIALIZED users AS (
    SELECT * FROM users WHERE status = 'active'
)
SELECT * FROM users WHERE city = 'HCM';
```
 
---
 
# PHẦN 4: SHARDING & REPLICATION
 
## 4.1 Replication Chi Tiết
 
### Statement-Based Replication
Primary gửi **SQL statements** tới replica:
```
Primary: UPDATE orders SET status = 'done' WHERE created_at < NOW() - INTERVAL '7 days'
Replica: Thực thi lại query này
```
- Ưu: Log nhỏ
- Nhược: Non-deterministic functions (`NOW()`, `RAND()`) cho kết quả khác nhau
 
### Row-Based Replication (mặc định MySQL)
Primary gửi **actual row changes**:
```
Primary: [table=orders, op=UPDATE, pk=123, before={status:'pending'}, after={status:'done'}]
Replica: Apply change trực tiếp
```
- Ưu: Deterministic, an toàn hơn
- Nhược: Log lớn hơn với bulk updates
 
### WAL Shipping (PostgreSQL)
Primary ghi Write-Ahead Log, gửi WAL segments tới replica:
 
```
Primary:
  Transaction → WAL buffer → WAL files (pg_wal/)
                                   ↓
                              WAL Sender ──── network ────→ WAL Receiver
                                                                  ↓
                                                           WAL files on replica
                                                                  ↓
                                                           Startup Process (apply)
                                                                  ↓
                                                           Replica DB
```
 
### Synchronous vs Asynchronous Replication
 
```sql
-- PostgreSQL synchronous replication
-- Primary chờ replica confirm trước khi commit
synchronous_standby_names = 'replica1'
 
-- Các modes:
synchronous_commit = on          -- chờ replica WAL flush
synchronous_commit = remote_write -- chờ replica nhận (không cần flush)
synchronous_commit = local        -- chỉ đảm bảo local WAL, async với replica
synchronous_commit = off          -- async hoàn toàn (có thể mất data nếu crash)
```
 
**RPO (Recovery Point Objective):**
- Sync replication: RPO = 0 (không mất data)
- Async replication: RPO > 0 (mất data trong replication lag window)
 
### Replication Lag
 
```sql
-- Kiểm tra replication lag (PostgreSQL)
SELECT
    client_addr,
    state,
    sent_lsn,
    write_lsn,
    flush_lsn,
    replay_lsn,
    sent_lsn - replay_lsn AS replication_lag_bytes,
    write_lag,
    flush_lag,
    replay_lag
FROM pg_stat_replication;
 
-- Trên replica:
SELECT now() - pg_last_xact_replay_timestamp() AS replication_delay;
```
 
**Nguyên nhân replication lag:**
- Replica bị overloaded (disk I/O, CPU)
- Long-running queries trên replica block apply
- Network bandwidth
 
---
 
## 4.2 Sharding Strategies
 
### Range-Based Sharding
 
```
Shard 0: user_id [1 → 1,000,000)
Shard 1: user_id [1,000,000 → 2,000,000)
Shard 2: user_id [2,000,000 → 3,000,000)
```
 
**Ưu điểm:**
- Range queries hiệu quả (theo user_id range → biết ngay shard nào)
- Dễ split shard khi một shard quá lớn
 
**Nhược điểm:**
- **Hot shard**: New users luôn vào shard mới nhất → uneven load
- Cần rebalance khi thêm shard
 
### Hash-Based Sharding
 
```
shard = hash(user_id) % num_shards
 
user_id=1 → hash=7823 → 7823 % 4 = 3 → Shard 3
user_id=2 → hash=1234 → 1234 % 4 = 2 → Shard 2
```
 
**Ưu điểm:**
- Phân phối đều
- Không có hot shard
 
**Nhược điểm:**
- Range queries phải query tất cả shards
- Thêm/bớt shard → rehash tất cả data (giải pháp: **Consistent Hashing**)
 
### Consistent Hashing
 
```
                0°
               ───
         270° |   | 90°
               ───
               180°
 
Nodes và data đều được hash vào một vòng tròn:
Node A = 30°
Node B = 120°
Node C = 240°
 
Key k1 = 80° → đi clockwise → Node B
Key k2 = 150° → đi clockwise → Node C
Key k3 = 290° → đi clockwise → Node A
```
 
Khi thêm Node D = 90°:
- Chỉ data từ 30°→90° (trước đây của B) cần chuyển sang D
- Phần còn lại không bị ảnh hưởng
 
**Virtual Nodes (vnodes)**: Mỗi physical node được map tới nhiều điểm trên ring → phân phối đều hơn.
 
### Directory-Based Sharding
 
```
Lookup Table:
┌──────────────┬────────┐
│ shard_key    │ shard  │
├──────────────┼────────┤
│ user_id 1-50 │ Shard0 │
│ user_id 51-80│ Shard1 │
│ VIP users    │ ShardV │
└──────────────┴────────┘
```
 
**Ưu điểm**: Linh hoạt tuyệt đối, có thể custom mapping
**Nhược điểm**: Lookup table là single point of failure, thêm latency
 
---
 
## 4.3 Cross-Shard Challenges
 
### Cross-Shard Query
 
```sql
-- BAD: Query data trên nhiều shards
SELECT u.name, SUM(o.amount)
FROM users u JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
-- Cần query tất cả shards, aggregate ở application layer → chậm
```
 
**Giải pháp:**
1. **Denormalize**: Copy user_name vào orders table — tránh join cross-shard
2. **Fan-out query**: Application query song song tất cả shards, merge kết quả
3. **Global tables**: Bảng nhỏ, đọc nhiều (lookup tables) → replicate trên tất cả shards
 
### Shard Key Selection — Quan trọng nhất
 
**Tiêu chí chọn shard key:**
 
1. **Cardinality cao** → phân bố đều
2. **Query locality** → các query liên quan thường query cùng shard
3. **Không thay đổi** → user_id tốt hơn email (email có thể đổi)
4. **Tránh hotspot** → timestamp thường tệ (mọi write vào cùng shard)
 
```
User-facing app → shard by user_id
Multi-tenant SaaS → shard by tenant_id  
E-commerce orders → shard by user_id (không phải order_id!)
→ Tất cả orders của user X trên cùng shard → query orders của user X hiệu quả
```
 
### Global IDs (cross-shard unique IDs)
 
```
Auto-increment không dùng được (mỗi shard có ID riêng → conflict)
 
Giải pháp 1 — UUID:
→ 128-bit, globally unique, nhưng không sorted → index fragmentation
 
Giải pháp 2 — Snowflake ID (Twitter):
┌──────────────────┬────────────┬─────────────┐
│   41 bits        │  10 bits   │   12 bits   │
│   timestamp (ms) │  machine   │  sequence   │
└──────────────────┴────────────┴─────────────┘
→ K-sorted (roughly time-ordered), 4096 IDs/ms/machine
 
Giải pháp 3 — DB sequence per shard với offset:
Shard 0: 1, 4, 7, 10... (start=1, increment=3)
Shard 1: 2, 5, 8, 11... (start=2, increment=3)
Shard 2: 3, 6, 9, 12... (start=3, increment=3)
```
 
---
 
## 4.4 Read/Write Splitting
 
```
Application
    │
    ├── Write ──→ Primary DB
    │
    └── Read ───→ Load Balancer ──→ Replica 1
                                 ──→ Replica 2
                                 ──→ Replica 3
```
 
**Vấn đề: Read-your-writes consistency**
```python
# T1: User update profile
db_primary.execute("UPDATE users SET bio = 'new bio' WHERE id = 1")
 
# T2: User reload page — đọc từ replica
bio = db_replica.fetchone("SELECT bio FROM users WHERE id = 1")
# Nếu replica lag 1s → user thấy bio cũ → confusing!
```
 
**Giải pháp:**
1. Sau write, đọc từ primary trong N giây
2. Dùng session-based routing: cùng session → cùng replica
3. Ghi timestamp của write, replica check "đã apply đến timestamp này chưa"
 
---
 
# PHẦN 5: NoSQL DEEP DIVE
 
## 5.1 Redis — Kiến Trúc & Use Cases
 
### Data Structures
 
```redis
# String — cache, counter, session
SET user:1:name "Nguyen Van An"
INCR page:views:homepage          # atomic counter
SETEX session:abc123 3600 "{...}" # TTL 1 giờ
 
# Hash — object storage
HSET user:1 name "An" email "an@example.com" age 25
HGET user:1 email
HGETALL user:1
 
# List — queue, timeline, history
LPUSH notifications:user:1 "New message"    # prepend
RPUSH job_queue "task_payload"              # append
BRPOP job_queue 0                           # blocking pop (job worker)
LRANGE timeline:user:1 0 19                 # 20 most recent
 
# Set — unique items, tags, mutual friends
SADD user:1:following 2 3 4 5
SADD user:2:following 1 3 6 7
SINTER user:1:following user:2:following    # mutual: {3}
SUNION user:1:following user:2:following    # all: {1,2,3,4,5,6,7}
 
# Sorted Set — leaderboard, priority queue
ZADD leaderboard 1500 "user:1"
ZADD leaderboard 2300 "user:2"
ZADD leaderboard 1800 "user:3"
ZREVRANGE leaderboard 0 9 WITHSCORES        # top 10
ZRANK leaderboard "user:1"                  # rank của user 1
 
# HyperLogLog — approximate distinct count, memory efficient
PFADD unique_visitors:2024-01-01 "user:1" "user:2" "user:3"
PFCOUNT unique_visitors:2024-01-01          # ~3 (approximate)
 
# Stream — event log, message queue
XADD events * action "purchase" user_id 123 amount 50000
XREAD COUNT 10 STREAMS events 0             # read events
XGROUP CREATE events processors $           # consumer group
```
 
### Redis Persistence
 
**RDB (Snapshotting):**
```
# redis.conf
save 900 1      # Sau 900s nếu ≥1 key thay đổi
save 300 10     # Sau 300s nếu ≥10 keys thay đổi
save 60 10000   # Sau 60s nếu ≥10000 keys thay đổi
```
- Fork process → dump data ra file .rdb
- Compact, nhanh khi restart
- Có thể mất data giữa 2 lần snapshot
 
**AOF (Append-Only File):**
```
# redis.conf
appendonly yes
appendfsync everysec    # flush mỗi giây (balance performance/durability)
# appendfsync always   # flush mỗi write (chậm nhất, an toàn nhất)
# appendfsync no       # OS quyết định (nhanh nhất, kém an toàn)
```
- Log mọi write command
- RPO tốt hơn RDB
- File lớn hơn, restart chậm hơn
 
**Hybrid (Redis 4.0+):** Dùng cả RDB + AOF:
```
aof-use-rdb-preamble yes
```
 
### Redis Cluster
 
```
3 master + 3 replica:
 
Master 0 (slots 0-5460)       ← Replica 0
Master 1 (slots 5461-10922)   ← Replica 1
Master 2 (slots 10923-16383)  ← Replica 2
 
Total: 16384 hash slots
slot = CRC16(key) % 16384
```
 
**Multi-key operations:**
```redis
# Keys trên cùng slot: dùng hash tags
SET {user:1}.name "An"
SET {user:1}.email "an@example.com"
# {user:1} → cùng slot → MGET hoạt động
MGET {user:1}.name {user:1}.email
```
 
### Cache Patterns
 
**Cache-Aside (Lazy Loading):**
```python
def get_user(user_id):
    # 1. Check cache
    user = redis.get(f"user:{user_id}")
    if user:
        return json.loads(user)
    
    # 2. Cache miss → load from DB
    user = db.query("SELECT * FROM users WHERE id = %s", user_id)
    
    # 3. Write to cache
    redis.setex(f"user:{user_id}", 3600, json.dumps(user))
    return user
```
 
**Write-Through:**
```python
def update_user(user_id, data):
    # Write to DB
    db.execute("UPDATE users SET ... WHERE id = %s", user_id)
    # Immediately update cache
    redis.setex(f"user:{user_id}", 3600, json.dumps(data))
```
 
**Cache Stampede Prevention:**
```python
# Vấn đề: Nhiều requests cùng cache miss → thundering herd
# Giải pháp: Mutex lock
def get_user_with_lock(user_id):
    cache_key = f"user:{user_id}"
    lock_key = f"lock:user:{user_id}"
    
    user = redis.get(cache_key)
    if user:
        return json.loads(user)
    
    # Try to acquire lock
    if redis.set(lock_key, 1, nx=True, ex=10):  # nx=only if not exists
        try:
            user = db.query(...)
            redis.setex(cache_key, 3600, json.dumps(user))
        finally:
            redis.delete(lock_key)
    else:
        # Another worker is fetching, wait and retry
        time.sleep(0.1)
        return get_user_with_lock(user_id)  # retry
    
    return user
```
 
---
 
## 5.2 Cassandra — Wide Column Store
 
### Data Model
 
```
Keyspace (~ Database)
  └── Table
        └── Partition (rows với cùng partition key)
              └── Rows (sorted by clustering key)
```
 
```cql
CREATE TABLE time_series_data (
    sensor_id    UUID,
    recorded_at  TIMESTAMP,
    temperature  FLOAT,
    humidity     FLOAT,
    PRIMARY KEY (sensor_id, recorded_at)  -- sensor_id: partition key, recorded_at: clustering key
) WITH CLUSTERING ORDER BY (recorded_at DESC);
 
-- Query hiệu quả: cùng partition
SELECT * FROM time_series_data
WHERE sensor_id = ? AND recorded_at > ? AND recorded_at < ?;
```
 
### Consistent Hashing & Replication
 
```
Token Ring:
 
     Node A (token 0)
    /                \
Node D              Node B
(token 270)        (token 90)
    \                /
     Node C (token 180)
 
Replication Factor = 3:
data với token 50 → lưu tại: Node B (90), Node C (180), Node D (270)
```
 
### Tunable Consistency
 
```cql
-- Write consistency
INSERT INTO ... USING CONSISTENCY QUORUM;
-- QUORUM = majority of replicas phải acknowledge
 
-- Read consistency
SELECT ... FROM ... USING CONSISTENCY LOCAL_ONE;
-- LOCAL_ONE = 1 replica trong local datacenter phải respond
 
-- Common levels:
-- ONE: 1 replica (fastest, weakest)
-- QUORUM: majority (n/2 + 1) — balance
-- ALL: tất cả (strongest, slowest)
-- LOCAL_QUORUM: majority trong local DC (multi-DC setup)
```
 
**Eventual Consistency mechanism:**
- **Read Repair**: Khi đọc, coordinator so sánh versions, update replica lỗi thời
- **Hinted Handoff**: Nếu replica down, node khác giữ tạm updates, deliver khi replica up lại
- **Anti-Entropy (Merkle Trees)**: Background process sync data giữa replicas
 
### Cassandra Write Path
 
```
1. Write → Commit Log (WAL) — durability
2. Write → MemTable (in-memory) — fast write
3. Khi MemTable đầy → flush → SSTable (immutable, sorted)
4. Periodically → Compaction: merge SSTables, remove tombstones
```
 
**Tombstones**: Cassandra không xóa ngay, ghi tombstone marker:
```
⚠️ Too many tombstones → query chậm → cần monitor và compact
```
 
---
 
## 5.3 MongoDB — Document Store
 
### Document Model
 
```json
{
  "_id": ObjectId("..."),
  "user_id": 123,
  "status": "active",
  "address": {
    "city": "HCM",
    "district": "Q1"
  },
  "tags": ["vip", "early_adopter"],
  "orders": [
    {"id": 1, "amount": 150000, "date": ISODate("2024-01-01")},
    {"id": 2, "amount": 280000, "date": ISODate("2024-02-01")}
  ]
}
```
 
### Aggregation Pipeline
 
```javascript
db.orders.aggregate([
  // Stage 1: Filter
  { $match: { status: "completed", created_at: { $gte: ISODate("2024-01-01") } } },
  
  // Stage 2: Join với users collection
  { $lookup: {
      from: "users",
      localField: "user_id",
      foreignField: "_id",
      as: "user"
  }},
  
  // Stage 3: Unwind array
  { $unwind: "$user" },
  
  // Stage 4: Group và aggregate
  { $group: {
      _id: "$user.city",
      total_revenue: { $sum: "$amount" },
      order_count: { $count: {} },
      avg_order: { $avg: "$amount" }
  }},
  
  // Stage 5: Sort
  { $sort: { total_revenue: -1 } },
  
  // Stage 6: Limit
  { $limit: 10 }
])
```
 
### MongoDB Indexing
 
```javascript
// Compound index
db.orders.createIndex({ user_id: 1, created_at: -1 })
 
// Partial index
db.orders.createIndex(
  { created_at: 1 },
  { partialFilterExpression: { status: "pending" } }
)
 
// Text index
db.articles.createIndex({ title: "text", content: "text" })
db.articles.find({ $text: { $search: "database performance" } })
 
// Wildcard index (cho JSONB-like queries)
db.products.createIndex({ "attributes.$**": 1 })
 
// Explain
db.orders.find({ user_id: 123 }).explain("executionStats")
```
 
### Transactions trong MongoDB (4.0+)
 
```javascript
// Multi-document ACID transactions
const session = db.getMongo().startSession();
session.startTransaction();
try {
  db.accounts.updateOne(
    { _id: fromId },
    { $inc: { balance: -amount } },
    { session }
  );
  db.accounts.updateOne(
    { _id: toId },
    { $inc: { balance: +amount } },
    { session }
  );
  session.commitTransaction();
} catch (error) {
  session.abortTransaction();
  throw error;
} finally {
  session.endSession();
}
```
 
---
 
# PHẦN 6: PERFORMANCE TUNING & MONITORING
 
## 6.1 PostgreSQL Configuration Tuning
 
### Memory Settings
 
```ini
# postgresql.conf
 
# Shared buffer — cache cho PostgreSQL (25% RAM thường tốt)
shared_buffers = 4GB        # server 16GB RAM
 
# Working memory per sort/hash operation
# Tổng memory = work_mem × max_connections × operations_per_query
work_mem = 64MB             # tăng nếu sort/hash spill to disk
 
# Maintenance operations (VACUUM, CREATE INDEX)
maintenance_work_mem = 512MB
 
# Effective cache size — hint cho planner về OS cache size
# Không allocate thực, chỉ ảnh hưởng cost estimation
effective_cache_size = 12GB  # ~75% RAM
```
 
### Checkpoint & WAL
 
```ini
# Checkpoint frequency
checkpoint_completion_target = 0.9  # spread checkpoint I/O
checkpoint_timeout = 15min          # max time between checkpoints
max_wal_size = 4GB                  # trigger checkpoint khi WAL đạt size này
 
# WAL configuration
wal_buffers = 64MB
wal_compression = on               # compress WAL, tốt cho replication
```
 
### Connection Settings
 
```ini
max_connections = 200              # Tổng connections
# Mỗi connection ~ 5-10MB RAM
# Nếu nhiều connections → dùng PgBouncer thay vì tăng max_connections
 
# Autovacuum tuning
autovacuum_vacuum_scale_factor = 0.1    # 10% dead tuples → vacuum
autovacuum_analyze_scale_factor = 0.05  # 5% changes → analyze
autovacuum_vacuum_cost_delay = 2ms      # tránh vacuum ảnh hưởng production
```
 
---
 
## 6.2 Slow Query Identification
 
### PostgreSQL
 
```sql
-- Enable slow query logging
-- postgresql.conf:
log_min_duration_statement = 1000  -- Log queries > 1 giây
log_statement = 'none'             -- Không log tất cả queries
 
-- pg_stat_statements extension
CREATE EXTENSION pg_stat_statements;
 
-- Top 10 slowest queries
SELECT
    round(total_exec_time / 1000 / 60, 2) AS total_minutes,
    round(mean_exec_time::numeric, 2) AS mean_ms,
    calls,
    round(stddev_exec_time::numeric, 2) AS stddev_ms,
    rows,
    query
FROM pg_stat_statements
ORDER BY total_exec_time DESC
LIMIT 10;
 
-- Queries với high I/O
SELECT
    calls,
    round(total_exec_time / calls, 2) AS avg_ms,
    shared_blks_hit,
    shared_blks_read,
    round(shared_blks_hit::numeric / nullif(shared_blks_hit + shared_blks_read, 0) * 100, 2) AS hit_rate_pct,
    query
FROM pg_stat_statements
WHERE shared_blks_read > 1000
ORDER BY shared_blks_read DESC;
```
 
### MySQL
 
```sql
-- Slow query log
SET GLOBAL slow_query_log = ON;
SET GLOBAL long_query_time = 1;  -- queries > 1s
SET GLOBAL log_queries_not_using_indexes = ON;
 
-- Performance Schema
SELECT
    DIGEST_TEXT,
    COUNT_STAR,
    round(SUM_TIMER_WAIT / 1e12, 2) AS total_seconds,
    round(AVG_TIMER_WAIT / 1e9, 2) AS avg_ms,
    SUM_ROWS_EXAMINED,
    SUM_ROWS_SENT
FROM performance_schema.events_statements_summary_by_digest
ORDER BY SUM_TIMER_WAIT DESC
LIMIT 10;
```
 
---
 
## 6.3 Key Metrics để Monitor
 
### Database Health Metrics
 
```sql
-- PostgreSQL: Cache hit rate (nên > 99%)
SELECT
    sum(heap_blks_hit) / nullif(sum(heap_blks_hit) + sum(heap_blks_read), 0) AS table_hit_rate,
    sum(idx_blks_hit) / nullif(sum(idx_blks_hit) + sum(idx_blks_read), 0) AS index_hit_rate
FROM pg_statio_user_tables;
 
-- Connection utilization
SELECT count(*), state FROM pg_stat_activity GROUP BY state;
 
-- Table bloat (dead tuples)
SELECT
    schemaname,
    tablename,
    pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS total_size,
    n_dead_tup,
    n_live_tup,
    round(n_dead_tup::numeric / nullif(n_live_tup, 0) * 100, 2) AS dead_ratio
FROM pg_stat_user_tables
WHERE n_live_tup > 10000
ORDER BY n_dead_tup DESC;
 
-- Index usage (unused indexes — overhead không cần thiết)
SELECT
    schemaname,
    tablename,
    indexname,
    idx_scan,
    pg_size_pretty(pg_relation_size(indexrelid)) AS index_size
FROM pg_stat_user_indexes
WHERE idx_scan = 0
  AND indexrelid NOT IN (SELECT conindid FROM pg_constraint)  -- exclude constraint indexes
ORDER BY pg_relation_size(indexrelid) DESC;
 
-- Lock waits
SELECT
    blocked_locks.pid AS blocked_pid,
    blocking_locks.pid AS blocking_pid,
    blocked_activity.query AS blocked_query,
    blocking_activity.query AS blocking_query,
    now() - blocked_activity.query_start AS wait_duration
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_locks blocking_locks
    ON blocking_locks.locktype = blocked_locks.locktype
    AND blocking_locks.relation = blocked_locks.relation
    AND blocking_locks.granted = true
    AND blocked_locks.granted = false
JOIN pg_catalog.pg_stat_activity blocked_activity ON blocked_activity.pid = blocked_locks.pid
JOIN pg_catalog.pg_stat_activity blocking_activity ON blocking_activity.pid = blocking_locks.pid;
```
 
### Golden Signals (4 metrics quan trọng nhất)
 
| Signal | Metric | Tool |
|---|---|---|
| **Latency** | Query response time (p50, p95, p99) | pg_stat_statements, slow query log |
| **Traffic** | Queries per second (QPS), TPS | pg_stat_database |
| **Errors** | Failed transactions, deadlocks | pg_stat_database.xact_rollback |
| **Saturation** | CPU, memory, disk I/O, connections | pg_stat_bgwriter, OS metrics |
 
---
 
## 6.4 Query Optimization Checklist
 
```
□ 1. Chạy EXPLAIN ANALYZE — hiểu execution plan
□ 2. Tìm Seq Scan trên bảng lớn — thêm index
□ 3. Kiểm tra row estimation vs actual — nếu sai nhiều → ANALYZE
□ 4. Kiểm tra N+1 queries trong application logs
□ 5. Xem buffer read cao — cache miss, cần tăng shared_buffers
□ 6. Sort/Hash spill to disk — tăng work_mem
□ 7. Index scan nhưng filter nhiều — cần composite/partial index tốt hơn
□ 8. Kiểm tra unused indexes — drop để giảm write overhead
□ 9. Kiểm tra table bloat — VACUUM nếu cần
□ 10. Connection pool đủ chưa — tránh connection exhaustion
```
 
---
 
## 6.5 Connection Pooling — PgBouncer
 
```ini
# pgbouncer.ini
[databases]
mydb = host=127.0.0.1 port=5432 dbname=mydb
 
[pgbouncer]
pool_mode = transaction       # transaction-level pooling (hiệu quả nhất)
max_client_conn = 1000        # max connections từ app
default_pool_size = 20        # connections thực tới PostgreSQL
min_pool_size = 5
reserve_pool_size = 5
server_idle_timeout = 600
```
 
**Pool modes:**
- **session**: 1 server connection per client session (ít hiệu quả nhất, tương thích cao nhất)
- **transaction**: Trả lại connection sau mỗi transaction (phổ biến nhất)
- **statement**: Trả lại sau mỗi statement (không hỗ trợ multi-statement transactions)
 
```
Không có PgBouncer:
1000 app connections → 1000 PostgreSQL connections → 5-10GB RAM chỉ cho connections
 
Với PgBouncer (transaction mode):
1000 app connections → 20 PostgreSQL connections → tiết kiệm RAM, tốt hơn nhiều
```
 
---
 
## 6.6 Partitioning (Table Partitioning)
 
```sql
-- Range Partitioning theo thời gian
CREATE TABLE events (
    id          BIGSERIAL,
    user_id     INT,
    event_type  VARCHAR(50),
    created_at  TIMESTAMP NOT NULL,
    data        JSONB
) PARTITION BY RANGE (created_at);
 
-- Tạo partitions theo tháng
CREATE TABLE events_2024_01 PARTITION OF events
    FOR VALUES FROM ('2024-01-01') TO ('2024-02-01');
CREATE TABLE events_2024_02 PARTITION OF events
    FOR VALUES FROM ('2024-02-01') TO ('2024-03-01');
 
-- Mỗi partition có index riêng
CREATE INDEX ON events_2024_01(user_id);
CREATE INDEX ON events_2024_02(user_id);
 
-- Query tự động route tới partition đúng (partition pruning)
EXPLAIN SELECT * FROM events
WHERE created_at BETWEEN '2024-01-15' AND '2024-01-20';
-- → Chỉ scan events_2024_01, không đụng vào partitions khác
```
 
**Lợi ích:**
- **Partition pruning**: Query chỉ scan partitions cần thiết
- **Partition-wise**: DROP partition cũ thay vì DELETE → cực nhanh
- Mỗi partition nhỏ hơn → index nhỏ hơn → fit vào memory tốt hơn
- Autovacuum hiệu quả hơn trên partitions nhỏ
 
---
 
## 6.7 Tóm Tắt Performance Tuning Hierarchy
 
```
1. Application Level (quan trọng nhất, dễ fix nhất)
   ├── Loại bỏ N+1 queries
   ├── Cache với Redis
   ├── Connection pooling
   └── Batch operations thay vì individual queries
 
2. Query Level
   ├── EXPLAIN ANALYZE → tìm bottleneck
   ├── Thêm/optimize index
   ├── Rewrite queries (tránh function on columns, etc.)
   └── Denormalize khi cần
 
3. Schema Level
   ├── Partitioning cho bảng lớn
   ├── Archiving data cũ
   └── Normalize/denormalize có chủ đích
 
4. Database Configuration
   ├── Memory (shared_buffers, work_mem)
   ├── Checkpoint & WAL settings
   └── Autovacuum tuning
 
5. Infrastructure Level (đắt nhất, cuối cùng)
   ├── Read replicas
   ├── Faster storage (NVMe SSD)
   ├── More RAM
   └── Sharding
```