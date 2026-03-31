# 🔴 Redis — Giải thích từ cơ bản đến chuyên sâu

> **Redis** (Remote Dictionary Server) là một **in-memory data store** mã nguồn mở, được dùng phổ biến làm cache, message broker, và database. Dữ liệu được lưu trong RAM nên tốc độ cực nhanh.

---

## 1. Redis là gì và tại sao nó quan trọng?

Redis không chỉ là một cache layer đơn thuần. Nó là một **data structure server** — nghĩa là nó không chỉ lưu key-value dạng string mà hỗ trợ nhiều kiểu dữ liệu phức tạp như list, set, hash, sorted set, stream...

### Redis trong thực tế được dùng để làm gì?

| Use case | Ví dụ cụ thể |
|---|---|
| **Cache** | Cache kết quả query DB, cache HTML page |
| **Session store** | Lưu session user cho web app |
| **Rate limiting** | Giới hạn số request mỗi IP |
| **Leaderboard** | Bảng xếp hạng game real-time |
| **Job queue** | Hàng đợi tác vụ background |
| **Pub/Sub** | Hệ thống chat, notification real-time |
| **Distributed lock** | Tránh race condition giữa nhiều server |
| **Counting** | Đếm số lượt view, like, click |

---

## 2. Tại sao Redis nhanh?

Redis nhanh không chỉ vì dùng RAM. Có nhiều lý do cộng hưởng:

### 2.1 In-Memory

Dữ liệu hoàn toàn trong RAM → không có I/O disk trong đường hot path.

```
Redis: RAM  → ~microseconds
MySQL: Disk → ~milliseconds
```

### 2.2 Single-threaded (cho command execution)

Redis xử lý commands trên **một thread duy nhất** (core commands). Điều này:
- Tránh hoàn toàn **race condition** và **context switching**
- Mỗi command là **atomic** — không cần lock
- CPU cache locality tốt hơn

> Lưu ý: Redis 6.0+ thêm **I/O threads** để xử lý network, nhưng command execution vẫn single-threaded.

### 2.3 Non-blocking I/O (Event loop)

Redis dùng **multiplexed I/O** (epoll/kqueue) để xử lý hàng nghìn kết nối đồng thời mà không tạo thread riêng cho mỗi connection.

### 2.4 Efficient data structures

Các cấu trúc dữ liệu được tối ưu ở tầng C — ziplist, skiplist, intset — chọn encoding phù hợp tự động dựa trên kích thước dữ liệu.

**Kết quả thực tế:**
- **100,000+ operations/giây** trên hardware thông thường
- Latency thường **< 1ms**

---

## 3. Các kiểu dữ liệu trong Redis

Đây là phần quan trọng nhất cần nắm. Mỗi kiểu dữ liệu giải quyết một bài toán khác nhau.

---

### 3.1 String

Kiểu dữ liệu cơ bản nhất. Lưu text, số, hoặc binary data (tối đa 512MB).

```bash
SET user:123:name "Nguyen Van A"
GET user:123:name
# → "Nguyen Van A"

# Lưu số và tăng/giảm atomic
SET page:views 100
INCR page:views       # → 101
INCRBY page:views 10  # → 111
DECR page:views       # → 110

# Set với TTL
SET session:abc "token_xyz" EX 3600   # hết hạn sau 1 giờ
SET session:abc "token_xyz" PX 60000  # hết hạn sau 60,000ms

# Set chỉ khi chưa tồn tại (dùng cho distributed lock)
SET lock:resource "owner_id" NX EX 30
```

**Dùng khi:** Cache đơn giản, counter, session token, flag, config value.

---

### 3.2 Hash

Lưu **object dạng field-value**, tương tự row trong database hay dict trong Python.

```bash
HSET user:123 name "Nguyen Van A" age 25 email "a@gmail.com"
HGET user:123 name          # → "Nguyen Van A"
HGETALL user:123            # → tất cả fields
HMGET user:123 name email   # → lấy nhiều fields
HINCRBY user:123 age 1      # → tăng age lên 1
HDEL user:123 email         # → xóa field
HEXISTS user:123 name       # → 1 (có tồn tại)
```

**Tại sao dùng Hash thay vì JSON string?**

| | Hash | JSON String |
|---|---|---|
| Lấy 1 field | `HGET` — O(1) | Phải lấy toàn bộ rồi parse |
| Update 1 field | `HSET` — không ảnh hưởng field khác | Phải đọc → parse → modify → serialize → ghi lại |
| Bộ nhớ | Tối ưu hơn (ziplist khi nhỏ) | Tốn hơn |

**Dùng khi:** Cache user profile, product info, bất kỳ object nào cần update từng phần.

---

### 3.3 List

Danh sách có thứ tự, cho phép thêm/xóa ở **cả hai đầu** — O(1). Thích hợp làm **queue** hoặc **stack**.

```bash
# Thêm vào đầu (left) hoặc cuối (right)
LPUSH notifications:user:123 "You have a new message"
RPUSH notifications:user:123 "Your order has shipped"

# Lấy theo index range
LRANGE notifications:user:123 0 -1   # tất cả
LRANGE notifications:user:123 0 9    # 10 phần tử đầu

# Pop (blocking — dùng cho queue)
LPOP notifications:user:123           # lấy từ đầu
RPOP notifications:user:123           # lấy từ cuối
BLPOP queue:jobs 30                   # blocking pop, chờ tối đa 30s

# Trim — giữ chỉ N phần tử mới nhất
LTRIM activity:user:123 0 99          # giữ 100 phần tử gần nhất
```

**Dùng khi:** Activity feed, recent items, job queue, undo history, chat message buffer.

---

### 3.4 Set

Tập hợp **không có thứ tự, không trùng lặp**. Hỗ trợ các phép toán tập hợp.

```bash
SADD tags:post:1 "redis" "database" "cache"
SADD tags:post:2 "redis" "nosql" "performance"

SMEMBERS tags:post:1         # → {"redis", "database", "cache"}
SISMEMBER tags:post:1 "redis" # → 1 (có trong set)
SCARD tags:post:1            # → 3 (số phần tử)

# Phép toán tập hợp
SINTER tags:post:1 tags:post:2   # → {"redis"} (giao)
SUNION tags:post:1 tags:post:2   # → tất cả tags (hợp)
SDIFF  tags:post:1 tags:post:2   # → {"database", "cache"} (hiệu)

# Xóa ngẫu nhiên (dùng cho lucky draw)
SPOP lucky:draw 1
```

**Dùng khi:** Tags, unique visitors, friend list, blacklist/whitelist, lottery.

---

### 3.5 Sorted Set (ZSet) ⭐

Set nhưng **mỗi phần tử có một score** (số thực). Phần tử được sắp xếp theo score. Đây là cấu trúc dữ liệu độc đáo và mạnh mẽ nhất của Redis.

```bash
# Thêm với score
ZADD leaderboard 9500 "player:alice"
ZADD leaderboard 8200 "player:bob"
ZADD leaderboard 9800 "player:charlie"

# Lấy theo rank (0-indexed, tăng dần)
ZRANGE leaderboard 0 -1 WITHSCORES     # tất cả, score thấp → cao
ZREVRANGE leaderboard 0 9 WITHSCORES  # top 10, score cao → thấp

# Rank của một player (0-indexed)
ZREVRANK leaderboard "player:alice"   # → 1 (hạng 2, charlie đứng đầu)

# Tăng score
ZINCRBY leaderboard 300 "player:bob"  # bob: 8200 → 8500

# Lấy theo khoảng score
ZRANGEBYSCORE leaderboard 8000 9000 WITHSCORES

# Số phần tử
ZCARD leaderboard   # → 3
```

**Implementation nội bộ:** Redis dùng **Skip List** — cho phép O(log N) với mọi thao tác.

**Dùng khi:** Leaderboard, priority queue, time-series events (dùng timestamp làm score), rate limiting.

---

### 3.6 Bitmap

Không phải kiểu riêng biệt mà là cách dùng String như một mảng bit. Cực kỳ tiết kiệm bộ nhớ.

```bash
# Đánh dấu user đã login vào ngày thứ N trong năm
SETBIT user:123:login:2024 180 1   # ngày 180 đã login
GETBIT user:123:login:2024 180     # → 1

# Đếm số ngày đã login trong năm
BITCOUNT user:123:login:2024       # → số bit = 1

# Bitwise operations (ví dụ: ai online cả tháng 1 và tháng 2)
BITOP AND result online:jan online:feb
```

**Bộ nhớ:** 1 bit/user → 1 triệu user chỉ tốn **125KB**.

**Dùng khi:** Daily active users, feature flags per user, attendance tracking.

---

### 3.7 HyperLogLog

Cấu trúc probabilistic để **đếm unique items** với sai số ~0.81%, nhưng chỉ tốn **12KB** bất kể có bao nhiêu phần tử.

```bash
PFADD uv:page:home "user:1" "user:2" "user:3"
PFADD uv:page:home "user:1"   # trùng → không tính thêm

PFCOUNT uv:page:home   # → ≈ 3 (xấp xỉ)

# Merge nhiều HLL
PFMERGE uv:all uv:page:home uv:page:about
```

**Dùng khi:** Đếm unique visitors khi không cần chính xác tuyệt đối, trending topics.

---

### 3.8 Stream

Append-only log, tương tự Kafka nhưng nhẹ hơn. Dùng cho event sourcing và message queue có persistence.

```bash
# Thêm event (ID tự động = timestamp-sequence)
XADD events:orders * user_id 123 product_id 456 action "purchase"

# Đọc từ đầu
XRANGE events:orders - +

# Consumer group (nhiều consumer cùng xử lý)
XGROUP CREATE events:orders workers $ MKSTREAM
XREADGROUP GROUP workers consumer1 COUNT 10 STREAMS events:orders >
```

**Dùng khi:** Audit log, event sourcing, lightweight message queue.

---

### Tóm tắt chọn kiểu dữ liệu

| Bài toán | Kiểu dữ liệu |
|---|---|
| Cache đơn giản, counter | String |
| Lưu object, update từng field | Hash |
| Queue, stack, recent items | List |
| Unique members, intersection | Set |
| Leaderboard, ranking | Sorted Set |
| Đếm unique users tiết kiệm RAM | HyperLogLog |
| Feature flag per user, daily active | Bitmap |
| Event log, message queue | Stream |

---

## 4. Persistence — Lưu dữ liệu xuống disk

Redis in-memory, nhưng nếu server restart thì mất hết? Không hẳn — Redis có hai cơ chế persistence:

### 4.1 RDB (Redis Database Snapshot)

Tạo **snapshot** toàn bộ dữ liệu vào file `.rdb` theo định kỳ.

```
# redis.conf
save 900 1      # nếu có ≥1 thay đổi trong 900s → snapshot
save 300 10     # nếu có ≥10 thay đổi trong 300s → snapshot
save 60 10000   # nếu có ≥10000 thay đổi trong 60s → snapshot
```

**Ưu điểm:**
- File compact, dễ backup và restore
- Performance cao (không ảnh hưởng đến hot path)
- Startup nhanh

**Nhược điểm:**
- Có thể mất data từ lần snapshot cuối đến khi crash (vài phút)

---

### 4.2 AOF (Append Only File)

Ghi lại **mọi write command** vào file log. Khi restart, Redis replay lại toàn bộ commands.

```
# redis.conf
appendonly yes
appendfsync everysec   # flush xuống disk mỗi 1 giây
# appendfsync always   # flush sau mỗi command (an toàn nhất, chậm nhất)
# appendfsync no       # để OS quyết định (nhanh nhất, ít an toàn)
```

**Ưu điểm:**
- Mất tối đa 1 giây data (với `everysec`)
- File dễ đọc và có thể chỉnh sửa thủ công

**Nhược điểm:**
- File lớn hơn RDB
- Startup chậm hơn (phải replay)
- AOF rewrite cần để file không phình quá to

---

### 4.3 Chọn cái nào?

| | RDB | AOF | Kết hợp cả hai |
|---|---|---|---|
| Durability | Thấp | Cao | Rất cao |
| Performance | Cao | Trung bình | Trung bình |
| File size | Nhỏ | Lớn | — |
| Restart speed | Nhanh | Chậm | Chậm |
| **Dùng khi** | Cache thuần, mất ít data OK | Cần durability cao | Production quan trọng |

**Best practice cho production:** Bật cả hai — AOF làm primary, RDB làm backup.

---

## 5. Expiration (TTL)

```bash
# Đặt TTL khi tạo
SET key "value" EX 3600     # seconds
SET key "value" PX 3600000  # milliseconds
SET key "value" EXAT 1700000000  # Unix timestamp

# Đặt TTL sau khi đã tạo
EXPIRE key 3600             # seconds
PEXPIRE key 3600000         # milliseconds

# Kiểm tra TTL còn lại
TTL key    # → giây còn lại (-1 = không hết hạn, -2 = không tồn tại)
PTTL key   # → milliseconds còn lại

# Xóa TTL (làm key tồn tại mãi mãi)
PERSIST key
```

**Cơ chế xóa key hết hạn của Redis:**

Redis dùng kết hợp hai chiến lược:

1. **Lazy expiration**: Khi access một key → kiểm tra TTL → nếu hết hạn thì xóa lúc đó
2. **Active expiration**: Background job định kỳ (100ms/lần) scan ngẫu nhiên ~20 key có TTL → xóa những key đã hết hạn → nếu >25% hết hạn thì lặp lại ngay

---

## 6. Replication — Master/Replica

Redis hỗ trợ **replication** để tăng khả năng đọc và fault tolerance.

```
┌─────────────────┐
│   Master        │ ← Xử lý ghi
│   (Read/Write)  │
└────────┬────────┘
         │ Async replication
    ┌────┴────┐
    ↓         ↓
┌───────┐ ┌───────┐
│Replica│ │Replica│ ← Xử lý đọc (scale out)
│(Read) │ │(Read) │
└───────┘ └───────┘
```

```bash
# Trên replica (redis.conf)
replicaof 192.168.1.100 6379
```

**Lưu ý:** Replication là **asynchronous** — có thể có lag nhỏ giữa master và replica.

---

## 7. Redis Sentinel — High Availability

Khi Master chết, ai sẽ tự động promote Replica lên làm Master mới? → **Redis Sentinel**

```
┌──────────┐  ┌──────────┐  ┌──────────┐
│Sentinel 1│  │Sentinel 2│  │Sentinel 3│
└────┬─────┘  └────┬─────┘  └────┬─────┘
     │              │              │
     └──────────────┴──────────────┘
                    │ Monitor + Failover
              ┌─────┴──────┐
              │   Master    │
              └─────┬──────┘
                    │
             ┌──────┴──────┐
             ↓              ↓
          Replica 1      Replica 2
```

**Quy trình failover:**
1. Sentinel phát hiện Master không phản hồi
2. Các Sentinel **vote** (cần quorum — thường 2/3) để xác nhận master thực sự chết
3. Sentinel chọn và promote một Replica thành Master mới
4. Các Replica còn lại được reconfigure để follow Master mới
5. Client được notify về địa chỉ Master mới

**Yêu cầu:** Cần ít nhất **3 Sentinel** để đảm bảo quorum tránh split-brain.

---

## 8. Redis Cluster — Horizontal Scaling

Sentinel giải quyết HA nhưng không giải quyết **sharding** — phân tán dữ liệu ra nhiều node khi data quá lớn cho một máy.

### Cơ chế sharding: Hash Slot

Redis Cluster chia dữ liệu thành **16,384 hash slots**.

```
hash_slot = CRC16(key) % 16,384
```

Mỗi master node chịu trách nhiệm một dải slot:

```
Node A (Master): slot 0 – 5460
Node B (Master): slot 5461 – 10922
Node C (Master): slot 10923 – 16383
```

Mỗi master có thêm replica riêng:

```
┌──────────────────────────────────────────┐
│  Node A Master  ←→  Node A Replica       │
│  (slot 0-5460)       (backup)            │
│                                          │
│  Node B Master  ←→  Node B Replica       │
│  (slot 5461-10922)   (backup)            │
│                                          │
│  Node C Master  ←→  Node C Replica       │
│  (slot 10923-16383)  (backup)            │
└──────────────────────────────────────────┘
```

**Khi client gửi request sai node:**

```
Client → Node A (SET user:123)
Node A: "Key này ở slot 8000, Node B quản lý"
→ MOVED 8000 192.168.1.2:6379
Client → Node B ✅
```

**Limitation của Cluster:**
- Multi-key commands (MGET, MSET) chỉ hoạt động nếu tất cả key cùng slot
- Dùng **hash tags** `{user:123}:profile` để force keys vào cùng slot

---

## 9. Các lệnh quan trọng khác

### Key management

```bash
EXISTS key              # → 1 nếu tồn tại
DEL key [key ...]       # xóa, trả về số key đã xóa
UNLINK key              # xóa async (không block)
TYPE key                # → string/hash/list/set/zset/stream
RENAME key newkey       # đổi tên
KEYS pattern            # ⚠️ KHÔNG dùng production (blocking)
SCAN cursor MATCH pattern COUNT 100  # ✅ dùng thay KEYS
```

### Transactions

```bash
MULTI           # bắt đầu transaction
SET key1 "a"
SET key2 "b"
INCR counter
EXEC            # thực thi tất cả → atomic
# hoặc DISCARD để hủy
```

**Lưu ý:** Redis transaction không có rollback. Nếu một lệnh lỗi cú pháp → toàn bộ fail. Nếu lỗi runtime (ví dụ INCR trên string) → các lệnh khác vẫn chạy.

### Lua Scripting

```bash
# Atomic check-and-set (không thể làm với MULTI/EXEC)
EVAL "
  local val = redis.call('GET', KEYS[1])
  if val == ARGV[1] then
    return redis.call('SET', KEYS[1], ARGV[2])
  end
  return 0
" 1 mykey oldvalue newvalue
```

Lua script chạy **atomic** — không có command nào khác chen vào giữa.

### Pipeline

```bash
# Gửi nhiều commands cùng lúc, giảm round-trip
redis-cli --pipe << EOF
SET key1 val1
SET key2 val2
SET key3 val3
EOF
```

Trong code (Node.js):
```javascript
const pipeline = redis.pipeline();
pipeline.set('key1', 'val1');
pipeline.set('key2', 'val2');
pipeline.get('key1');
await pipeline.exec(); // Chỉ 1 round-trip thay vì 3
```

---

## 10. Các Use Case thực tế và cách implement

### 10.1 Session Store

```bash
# Login
SET session:{token} {user_id:123,role:admin} EX 86400

# Check session
GET session:{token}

# Logout
DEL session:{token}

# Refresh session (reset TTL)
EXPIRE session:{token} 86400
```

---

### 10.2 Rate Limiting

**Sliding window với Sorted Set:**

```bash
# Mỗi request của IP:
ZADD ratelimit:192.168.1.1 {current_timestamp} {request_id}
ZREMRANGEBYSCORE ratelimit:192.168.1.1 0 {timestamp - window_size}
count = ZCARD ratelimit:192.168.1.1
EXPIRE ratelimit:192.168.1.1 {window_size}

# Nếu count > limit → reject
```

**Fixed window đơn giản hơn với String:**

```bash
current = INCR ratelimit:{ip}:{minute}
if current == 1:
    EXPIRE ratelimit:{ip}:{minute} 60
if current > 100:
    # reject
```

---

### 10.3 Distributed Lock

```bash
# Acquire lock
SET lock:{resource} {unique_id} NX EX 30
# NX = chỉ set nếu chưa có
# EX 30 = tự giải phóng sau 30s (tránh deadlock)

# Release lock (phải dùng Lua để atomic check-and-delete)
EVAL "
  if redis.call('GET', KEYS[1]) == ARGV[1] then
    return redis.call('DEL', KEYS[1])
  end
  return 0
" 1 lock:{resource} {unique_id}
```

**Tại sao cần check unique_id khi release?**

Tránh trường hợp: Lock A hết hạn → Lock B acquire → Lock A cố gắng release → xóa nhầm Lock B.

---

### 10.4 Leaderboard Real-time

```bash
# Cập nhật score
ZADD leaderboard {score} {user_id}
# hoặc tăng score
ZINCRBY leaderboard {delta} {user_id}

# Top 10
ZREVRANGE leaderboard 0 9 WITHSCORES

# Rank của user cụ thể (1-indexed)
rank = ZREVRANK leaderboard {user_id}
score = ZSCORE leaderboard {user_id}

# Lấy surrounding players (±5 vị trí quanh user)
ZREVRANGE leaderboard {rank-5} {rank+5} WITHSCORES
```

---

### 10.5 Pub/Sub — Real-time Notification

```bash
# Publisher
PUBLISH channel:notifications:user:123 '{"type":"message","from":"user:456"}'

# Subscriber (blocking)
SUBSCRIBE channel:notifications:user:123

# Pattern subscribe
PSUBSCRIBE channel:notifications:*
```

**Hạn chế của Pub/Sub:** Messages không được persist — nếu subscriber offline thì mất. Dùng **Stream** nếu cần reliability.

---

## 11. Memory Management

### Xem memory usage

```bash
INFO memory
# → used_memory_human: 1.5G
# → mem_fragmentation_ratio: 1.2 (lý tưởng ~1.0-1.5)

MEMORY USAGE key    # bộ nhớ của một key cụ thể (bytes)
MEMORY DOCTOR       # phân tích và gợi ý
```

### Eviction khi đầy RAM

Cấu hình trong `redis.conf`:

```
maxmemory 2gb
maxmemory-policy allkeys-lru
```

| Policy | Hành vi |
|---|---|
| `noeviction` | Báo lỗi khi đầy (mặc định) |
| `allkeys-lru` | Xóa key ít dùng nhất trong toàn bộ keyspace |
| `volatile-lru` | Xóa key ít dùng nhất trong số các key có TTL |
| `allkeys-lfu` | Xóa key ít được access nhất (theo tần suất) |
| `volatile-ttl` | Xóa key có TTL ngắn nhất trước |
| `allkeys-random` | Xóa ngẫu nhiên |

**Best practice:**
- Nếu Redis là **cache thuần**: `allkeys-lru` hoặc `allkeys-lfu`
- Nếu Redis vừa cache vừa lưu data quan trọng: `volatile-lru` (chỉ xóa key có TTL)

---

## 12. Monitoring & Debugging

```bash
# Thông tin tổng quan
INFO all           # tất cả metrics
INFO server        # version, uptime
INFO clients       # connected_clients
INFO stats         # hits, misses, ops/sec
INFO replication   # master/replica status

# Theo dõi commands real-time (KHÔNG dùng production)
MONITOR

# Slow log
CONFIG SET slowlog-log-slower-than 10000  # 10ms
SLOWLOG GET 10    # 10 slow queries gần nhất
SLOWLOG LEN
SLOWLOG RESET

# Debug latency
redis-cli --latency
redis-cli --latency-history
redis-cli --latency-dist

# Thống kê hit/miss rate
INFO stats | grep keyspace
```

**Hit rate tính thủ công:**

```
Hit Rate = keyspace_hits / (keyspace_hits + keyspace_misses)
```

---

## 13. Bẫy phổ biến khi dùng Redis

### ❌ Dùng KEYS trong production

```bash
KEYS user:*    # Blocking! Scan toàn bộ keyspace → freeze Redis
```

✅ Thay bằng:
```bash
SCAN 0 MATCH user:* COUNT 100
```

---

### ❌ Không set TTL

Không set TTL cho cache keys → RAM dần đầy → eviction hoặc OOM.

✅ Luôn set TTL cho mọi cache key.

---

### ❌ Lưu object lớn (Big Key)

Key có value > 10KB (string) hoặc collection > 10,000 phần tử → blocking khi serialize/delete.

✅ Chia nhỏ, hoặc dùng `UNLINK` thay `DEL` (async delete).

---

### ❌ Không dùng connection pool

Tạo kết nối mới cho mỗi request → overhead lớn.

✅ Luôn dùng connection pool (ioredis, lettuce, jedis đều hỗ trợ).

---

### ❌ Dùng SELECT để tách môi trường

Redis có 16 database (SELECT 0-15), nhưng:
- Cluster không support SELECT
- Không có isolation thực sự về performance

✅ Dùng key prefix: `dev:user:123`, `prod:user:123`

---

## 14. Khi nào KHÔNG nên dùng Redis?

- **Dữ liệu lớn hơn RAM**: Redis in-memory — nếu data hàng trăm GB thì không phù hợp
- **Cần complex queries**: JOIN, aggregation phức tạp → dùng DB quan hệ
- **Cần ACID đầy đủ**: Redis transaction thiếu rollback
- **Data quan trọng cần durability cao**: Cân nhắc dùng DB chính, Redis làm cache phụ

---

## 15. Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Data structures | String, Hash, List, Set, ZSet, ... | Chỉ String |
| Persistence | Có (RDB, AOF) | Không |
| Replication | Có | Không (native) |
| Cluster | Có | Không (client-side) |
| Pub/Sub | Có | Không |
| Lua scripting | Có | Không |
| Transactions | Có | Không |
| Multi-threading | I/O threads (v6+) | Multi-threaded |
| Memory efficiency | Tốt | Tốt hơn một chút với simple string |
| **Kết luận** | ✅ Chọn mặc định | Chỉ khi cần multi-thread thuần túy |

---

## 16. Tóm tắt

```
Redis = In-memory data structure store
      = Cache + Session + Queue + Pub/Sub + Lock + ...

Nhanh vì: RAM + Single-threaded (no lock) + Non-blocking I/O
Flexible vì: 8+ kiểu dữ liệu cho mọi bài toán
Production-ready vì: Persistence + Replication + Sentinel + Cluster
```

**Key takeaways:**

- Chọn đúng **data structure** là 80% thành công khi dùng Redis
- Luôn **set TTL** và cấu hình **maxmemory-policy** phù hợp
- **SCAN** thay **KEYS**, **UNLINK** thay **DEL** cho big keys
- Dùng **Pipeline** để giảm round-trip khi thực hiện nhiều commands
- **Sentinel** cho HA, **Cluster** cho horizontal scaling
- Redis là **cache + utility store**, không phải primary database cho business-critical data