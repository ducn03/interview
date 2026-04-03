# System Design — Từ Máy Tính Đến Hệ Thống Triệu Người Dùng

> Đọc từ đầu đến cuối theo thứ tự. Mỗi chương là nền tảng của chương tiếp theo.
> Không có shortcut — hiểu máy tính hoạt động thế nào thì mới hiểu tại sao hệ thống lại được thiết kế như vậy.

---

# PHẦN I: NỀN TẢNG — MỘT MÁY TÍNH HOẠT ĐỘNG THẾ NÀO

---

# CHƯƠNG 1: Computer Architecture — Bên Trong Một Máy Tính

## 1.1 Tại sao phải bắt đầu từ đây?

Mọi quyết định trong system design — tại sao cần cache, tại sao I/O là bottleneck, tại sao thread pool có giới hạn — đều có lý do gốc rễ từ cách phần cứng hoạt động. Bỏ qua phần này thì bạn đang học thuộc lòng công thức mà không hiểu toán học đằng sau.

## 1.2 Các thành phần cốt lõi

Một máy tính hiện đại gồm bốn thành phần giao tiếp với nhau:

```
┌─────────────────────────────────────────────────────────┐
│                        CPU                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐             │
│  │  Core 0  │  │  Core 1  │  │  Core 2  │  ...        │
│  │ L1 Cache │  │ L1 Cache │  │ L1 Cache │             │
│  └────┬─────┘  └────┬─────┘  └────┬─────┘             │
│       └──────────────┴─────────────┘                   │
│                  L2 Cache (shared)                      │
│                  L3 Cache (shared)                      │
└─────────────────────┬───────────────────────────────────┘
                      │ Memory Bus
┌─────────────────────┴───────────────────────────────────┐
│                      RAM (Primary Memory)               │
│              8GB / 16GB / 64GB ...                      │
└─────────────────────┬───────────────────────────────────┘
                      │ I/O Bus (PCIe, SATA...)
          ┌───────────┴────────────┐
    ┌─────┴─────┐           ┌──────┴──────┐
    │    SSD    │           │   Network   │
    │  Storage  │           │    Card     │
    └───────────┘           └─────────────┘
```

**CPU (Central Processing Unit):** Thực thi instructions. Hiện đại có nhiều core — mỗi core là một bộ xử lý độc lập có thể chạy song song.

**RAM (Random Access Memory):** Bộ nhớ tạm thời, nhanh. Mọi thứ CPU cần xử lý phải nằm trong RAM trước. Mất điện → mất hết.

**Storage (SSD/HDD):** Lưu trữ lâu dài. Chậm hơn RAM nhiều lần. Mất điện → data vẫn còn.

**Network Card:** Kết nối với máy khác qua mạng. Từ góc nhìn CPU, đây là thiết bị I/O chậm nhất.

## 1.3 Hierarchy of Memory — Tại sao Cache tồn tại

Đây là thứ quan trọng nhất cần hiểu về phần cứng.

```
           Tốc độ          Dung lượng      Chi phí/GB
L1 Cache   0.5 ns          32-64 KB        ~$10,000
L2 Cache   5 ns            256 KB - 1 MB   ~$1,000
L3 Cache   20 ns           8-32 MB         ~$100
RAM        100 ns          8-64 GB         ~$5
SSD        100,000 ns      256GB - 4TB     ~$0.10
HDD        10,000,000 ns   1TB - 20TB      ~$0.02
Network    ~500,000 ns      ∞               ~$0.001/GB
```

**Nghịch lý:** Thứ nhanh nhất thì nhỏ nhất và đắt nhất. Thứ lớn nhất thì chậm nhất.

**Giải pháp của CPU:** Cache hierarchy — lưu dữ liệu thường dùng ở cache gần nhất.

```
CPU muốn đọc địa chỉ 0x1234:
  1. Tìm trong L1 → Có? → Đọc (0.5ns) ✓ CACHE HIT
  2. Tìm trong L2 → Có? → Đọc (5ns) ✓
  3. Tìm trong L3 → Có? → Đọc (20ns) ✓
  4. Tìm trong RAM → Đọc (100ns) và copy vào L1/L2/L3
  5. Không có trong RAM → Page Fault → Đọc từ disk (100,000ns!)
```

**Tại sao điều này quan trọng với system design?**

Vì nguyên tắc này áp dụng ở mọi tầng của hệ thống:
- Redis là "L1 cache" của database
- Database là "L2 cache" của disk
- CDN là "L1 cache" của origin server

Không hiểu tại sao cache tồn tại ở phần cứng → không hiểu tại sao cần cache ở application level.

## 1.4 Locality of Reference — Nguyên lý đằng sau cache

CPU cache hoạt động hiệu quả vì phần lớn chương trình tuân theo hai nguyên lý:

**Temporal Locality:** Dữ liệu vừa truy cập có khả năng cao được truy cập lại sớm.
```
for i in range(1000):
    x = array[5]  # array[5] được truy cập 1000 lần → giữ trong cache
```

**Spatial Locality:** Dữ liệu gần với dữ liệu vừa truy cập có khả năng cao được truy cập tiếp.
```
for i in range(1000):
    x = array[i]  # Truy cập array[0], array[1], array[2]...
                  # CPU load cả cache line (64 bytes) vào cache
                  # → array[0] đến array[7] (nếu int 8 bytes) vào cache cùng lúc
```

**Ứng dụng thực tế:** Database index dùng B-Tree thay vì hash table một phần vì B-Tree có spatial locality tốt hơn (các node liền kề nhau về mặt vật lý) → CPU cache hiệu quả hơn.

## 1.5 CPU Execution — Chương trình Thực Sự Chạy Thế Nào

```
Code bạn viết:
  x = a + b

Biên dịch thành assembly:
  MOV EAX, [addr_a]   ; Load giá trị a vào register EAX
  ADD EAX, [addr_b]   ; Cộng b vào EAX
  MOV [addr_x], EAX   ; Lưu kết quả ra memory

CPU thực thi:
  Fetch → Decode → Execute → Write Back
  (Lấy instruction) → (Hiểu nó làm gì) → (Thực thi) → (Ghi kết quả)
```

**CPU hiện đại không chạy tuần tự** — dùng kỹ thuật:
- **Pipelining:** Fetch instruction tiếp theo trong khi execute instruction hiện tại
- **Out-of-order execution:** Thực thi instructions không theo thứ tự nếu không có dependency
- **Speculative execution:** Đoán trước nhánh if/else và thực thi trước

Điều này tạo ra **memory ordering** phức tạp — lý do tại sao concurrent programming khó và cần memory barriers, mutexes.

---

# CHƯƠNG 2: Operating System — Lớp Quản Lý Tài Nguyên

## 2.1 OS là gì và tại sao cần nó

Không có OS, mỗi chương trình phải:
- Tự giao tiếp với hardware (biết từng lệnh của từng loại disk, card mạng)
- Tự quản lý memory (không được dùng vùng nhớ của chương trình khác)
- Tự chia sẻ CPU (nếu muốn chạy nhiều chương trình cùng lúc)

Không thể. **OS là lớp abstraction giữa hardware và application.**

```
┌──────────────────────────────────────┐
│         Applications                 │  ← Chrome, Node.js, PostgreSQL
├──────────────────────────────────────┤
│          System Libraries            │  ← glibc, POSIX API
├──────────────────────────────────────┤
│         System Call Interface        │  ← read(), write(), socket()...
├──────────────────────────────────────┤
│           OS Kernel                  │  ← Process mgmt, Memory mgmt, I/O
├──────────────────────────────────────┤
│           Hardware                   │  ← CPU, RAM, Disk, NIC
└──────────────────────────────────────┘
```

OS kernel chạy ở **privileged mode** (kernel space) — có thể làm mọi thứ với hardware. Application chạy ở **user space** — không thể trực tiếp đụng vào hardware, phải xin OS thông qua **system calls**.

```
Application muốn đọc file:
  1. Gọi open("/etc/config", O_RDONLY)
  2. CPU nhận system call → chuyển sang kernel mode
  3. Kernel kiểm tra permission
  4. Kernel đọc từ disk vào kernel buffer
  5. Copy từ kernel buffer vào user space buffer
  6. Trả về file descriptor
  7. CPU quay về user mode
```

Mỗi system call = context switch = tốn ~1,000ns. Lý do tại sao read() nhiều lần nhỏ chậm hơn read() 1 lần lớn.

## 2.2 Process — Đơn Vị Thực Thi

**Process** là một instance của chương trình đang chạy. Mỗi process có:

```
Process "Chrome":
┌───────────────────────────────────────┐
│ Virtual Address Space (riêng của nó)  │
│  ┌─────────────┐                      │
│  │ Code segment│  ← Instructions      │
│  │ Data segment│  ← Global variables  │
│  │    Heap     │  ← malloc() memory   │
│  │    Stack    │  ← Function calls    │
│  └─────────────┘                      │
│                                       │
│ Process Control Block (PCB):          │
│  - Process ID (PID)                   │
│  - State: Running/Ready/Blocked       │
│  - CPU registers snapshot             │
│  - Open file descriptors              │
│  - Memory mappings                    │
└───────────────────────────────────────┘
```

**Virtual Memory:** Mỗi process nghĩ nó có cả địa chỉ bộ nhớ riêng (0x0000 đến 0xFFFF...). OS + hardware (MMU) map địa chỉ ảo này sang địa chỉ vật lý thực trên RAM.

```
Process A nghĩ địa chỉ 0x1000 là của nó
Process B cũng nghĩ địa chỉ 0x1000 là của nó
→ Thực ra 2 địa chỉ vật lý khác nhau trên RAM
→ Không process nào đọc được memory của process kia (isolation)
```

**Tại sao quan trọng với system design?**
- Process isolation = crash một service không làm sập service khác
- Microservices chạy mỗi service trong process riêng → tận dụng điều này
- Container (Docker) về cơ bản là process isolation + namespace isolation

## 2.3 Thread — Đơn Vị Nhỏ Hơn Trong Process

**Thread** là luồng thực thi trong một process. Nhiều thread trong cùng process **chia sẻ memory** với nhau.

```
Process "Node.js":
┌─────────────────────────────────────────────────┐
│ Shared: Code, Heap, Global variables, File FDs  │
│                                                 │
│ Thread 1           Thread 2           Thread 3  │
│ ┌──────────┐      ┌──────────┐      ┌────────┐ │
│ │ Stack 1  │      │ Stack 2  │      │Stack 3 │ │
│ │Registers │      │Registers │      │Regs    │ │
│ └──────────┘      └──────────┘      └────────┘ │
└─────────────────────────────────────────────────┘
```

**Thread vs Process:**

| | Process | Thread |
|---|---|---|
| Memory | Riêng biệt | Chia sẻ trong process |
| Tạo mới | Chậm (~1ms) | Nhanh (~10µs) |
| Context switch | Chậm (flush TLB) | Nhanh |
| Crash | Không ảnh hưởng process khác | Crash toàn process |
| Communication | IPC phức tạp | Shared memory dễ |
| Concurrency bugs | Ít | Nhiều (race conditions) |

**Context Switch là gì?**

OS có nhiều process/thread muốn chạy, nhưng CPU chỉ có N core. OS phải liên tục dừng process này, lưu trạng thái, chạy process khác:

```
Thread A đang chạy:
  OS timer interrupt → "Thời gian hết rồi"
  OS lưu: registers, stack pointer, instruction pointer của A
  OS nạp: registers, stack pointer, instruction pointer của B
  Thread B chạy
  (lặp lại mỗi ~1-10ms)
```

Context switch tốn ~1-10µs. Nếu hệ thống có 10,000 thread → CPU dành phần lớn thời gian switch context thay vì làm việc thực sự → **thrashing**.

**Ứng dụng thực tế:**
- Web server kiểu cũ: 1 thread/connection → 10,000 concurrent users = 10,000 thread → thrashing
- Web server hiện đại (Nginx, Node.js): Event loop + non-blocking I/O → 1 thread xử lý hàng nghìn connection

## 2.4 Concurrency và Race Condition

Nhiều thread cùng truy cập shared data → nguy hiểm:

```python
# Balance ban đầu: 1000
# Thread A và Thread B cùng rút 500

Thread A:                    Thread B:
balance = read()  # = 1000
                             balance = read()  # = 1000
balance = balance - 500
write(balance)    # = 500
                             balance = balance - 500
                             write(balance)    # = 500 (!!!)

# Kết quả: 500, đúng lẽ phải là 0
# Mất 500 tiền ảo!
```

**Giải pháp: Synchronization primitives**

*Mutex (Mutual Exclusion Lock):*
```python
mutex = Lock()

# Thread A:
mutex.acquire()
balance = read()
balance = balance - 500
write(balance)
mutex.release()

# Thread B phải chờ A release mới được acquire
# → Không có race condition
```

*Semaphore:* Generalized mutex — cho phép N thread cùng vào critical section thay vì chỉ 1.

```python
# Connection pool: tối đa 10 connections
semaphore = Semaphore(10)

def query():
    semaphore.acquire()  # Chờ nếu đã có 10 connection
    # ... dùng connection
    semaphore.release()  # Trả lại slot
```

**Deadlock:**

```
Thread A giữ Lock X, chờ Lock Y
Thread B giữ Lock Y, chờ Lock X
→ Cả hai chờ mãi mãi → Deadlock

Phòng tránh: Luôn acquire lock theo thứ tự cố định
```

**Ứng dụng trong system design:**
- Database locks bảo vệ ACID transactions = mutex ở tầng DB
- Redis single-threaded = tránh hoàn toàn race condition
- Connection pool dùng semaphore để giới hạn concurrent connections

## 2.5 I/O Models — Blocking, Non-blocking, Async

Đây là một trong những khái niệm quan trọng nhất để hiểu tại sao các server được thiết kế khác nhau.

**Blocking I/O (Synchronous):**

```
Thread gọi read(socket):
  → Thread bị block, không làm gì được
  → Chờ... chờ... (network có thể mất 100ms)
  → Data về → Thread tiếp tục

Vấn đề: 1 thread chỉ xử lý 1 connection tại 1 thời điểm
Apache HTTP server cũ: 1 thread/connection
1,000 concurrent users → 1,000 thread → context switching overhead lớn
```

**Non-blocking I/O:**

```
Thread gọi read(socket, O_NONBLOCK):
  → Nếu không có data: trả về EAGAIN ngay lập tức
  → Thread làm việc khác
  → Poll lại sau
  → Lãng phí CPU vì phải liên tục poll (busy waiting)
```

**I/O Multiplexing (select/poll/epoll):**

```
Thread đăng ký: "Báo cho tôi khi socket A, B, C có data"
epoll_wait() → Block thread, nhưng:
  → Khi socket A có data → wake up thread
  → Thread xử lý socket A
  → Đăng ký lại, chờ tiếp

Nginx/Node.js dùng model này:
  1 thread xử lý hàng nghìn connection
  Thread chỉ "thức" khi thực sự có việc làm
  → Event Loop
```

**Asynchronous I/O (io_uring - Linux modern):**

```
Thread submit I/O request: "Đọc file X, gọi callback khi xong"
Thread tiếp tục làm việc khác hoàn toàn
Kernel tự làm I/O ở background
Khi xong → notify thread qua completion queue
```

**Tại sao điều này quan trọng:**

```
1,000 concurrent HTTP requests, mỗi request cần 100ms DB query:

Model Blocking (Apache-style):
  1,000 thread × 100ms chờ DB = 1,000 thread blocked
  CPU không làm gì cả trong 100ms đó
  Context switching cho 1,000 thread = overhead lớn

Model Event Loop (Nginx/Node.js-style):
  1 thread
  Gửi 1,000 query đến DB
  epoll chờ... DB trả kết quả → xử lý từng cái
  CPU 100% thời gian làm việc thực sự
  → Throughput cao hơn nhiều
```

---

# CHƯƠNG 3: Networking — Máy Tính Nói Chuyện Với Nhau Thế Nào

## 3.1 OSI Model — Tại sao Cần Chia Lớp?

Giao tiếp mạng là vấn đề phức tạp. Cần truyền bytes từ máy A sang máy B ở bên kia thế giới qua nhiều loại vật lý khác nhau (cáp quang, WiFi, 4G). Giải pháp: **chia thành nhiều lớp**, mỗi lớp giải quyết một vấn đề cụ thể.

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 7: Application   HTTP, FTP, DNS, SMTP                │
│  Layer 6: Presentation  Encryption (TLS), Compression       │
│  Layer 5: Session       Session management                   │
│  Layer 4: Transport     TCP, UDP — end-to-end delivery       │
│  Layer 3: Network       IP — routing between networks        │
│  Layer 2: Data Link     Ethernet, WiFi — local delivery      │
│  Layer 1: Physical      Cables, signals, bits                │
└─────────────────────────────────────────────────────────────┘
```

**Encapsulation — Mỗi lớp thêm header:**

```
HTTP request "GET /index.html":

Application: [HTTP Header][HTTP Body]
Transport:   [TCP Header][HTTP Header][HTTP Body]
Network:     [IP Header][TCP Header][HTTP Header][HTTP Body]
Data Link:   [Eth Header][IP Header][TCP Header][HTTP Header][HTTP Body][Eth Trailer]
Physical:    1010110010101010111...
```

Khi nhận được, mỗi lớp bóc header của mình ra và đẩy phần còn lại lên lớp trên.

**Tại sao quan trọng với system design:**
- Load balancer hoạt động ở Layer 4 (TCP) hoặc Layer 7 (HTTP) — khác nhau về capability
- Firewall lọc ở Layer 3/4 (IP/TCP)
- CDN hoạt động ở Layer 7 — có thể đọc HTTP headers
- TLS encryption ở Layer 6 — transparent với application

## 3.2 TCP vs UDP — Hai Triết Lý Giao Tiếp

**TCP (Transmission Control Protocol) — Reliable, Ordered, Connected**

```
Client                              Server
  │                                    │
  ├──── SYN ──────────────────────────►│  "Tôi muốn kết nối"
  │◄─── SYN-ACK ───────────────────────┤  "OK, tôi sẵn sàng"
  ├──── ACK ──────────────────────────►│  "Bắt đầu thôi"
  │         (3-way handshake)          │
  │                                    │
  ├──── DATA packet 1 ────────────────►│
  │◄─── ACK 1 ─────────────────────────┤  "Nhận được packet 1"
  ├──── DATA packet 2 ────────────────►│
  │    (lost)                          │
  │    (timeout)                       │
  ├──── DATA packet 2 (retransmit) ───►│  Tự động gửi lại
  │◄─── ACK 2 ─────────────────────────┤
  │                                    │
  ├──── FIN ──────────────────────────►│  "Tôi xong rồi"
  │◄─── FIN-ACK ───────────────────────┤
  └──── ACK ──────────────────────────►│
```

TCP đảm bảo:
- **Reliability:** Packet bị mất → tự động gửi lại
- **Ordering:** Packet 1 luôn đến trước packet 2 (dù IP có thể reorder)
- **Flow control:** Không gửi nhanh hơn receiver xử lý được
- **Congestion control:** Không làm tắc nghẽn mạng

**Chi phí:** Handshake (1.5 round-trip), ACK overhead, retransmit delay.

**UDP (User Datagram Protocol) — Fast, Unreliable, Connectionless**

```
Client → [packet] → Server    "Tôi gửi, không biết có đến không"
Client → [packet] → Server    "Gửi tiếp, vẫn không quan tâm"
```

Không có handshake, không có ACK, không có retransmit, không có ordering.

**Tốc độ:** Rất nhanh, overhead thấp.

**Dùng khi nào:**

| TCP | UDP |
|---|---|
| HTTP/HTTPS | DNS lookup |
| Database queries | Video streaming |
| File transfer | Online games |
| Email | VoIP |
| Bất cứ thứ gì cần chính xác | Real-time, mất 1 packet chấp nhận được |

**Ví dụ thực tế:** Video call (Zoom, Meet) dùng UDP. Mất 1 frame video không sao, nhưng không thể chờ TCP retransmit (sẽ bị delay). Dữ liệu cũ đến muộn thì vô dụng.

## 3.3 TCP Connection — Chi Phí Ẩn

**3-way handshake tốn 1 round-trip time (RTT).** Nếu client ở Việt Nam, server ở US → 150ms RTT → mất 150ms chỉ để setup connection trước khi gửi 1 byte data.

**TCP Connection Pooling — Tại sao cần:**

```
Không có connection pool:
  Request 1: Tạo connection (150ms) + Query (10ms) + Close = 160ms
  Request 2: Tạo connection (150ms) + Query (10ms) + Close = 160ms
  → Tốn 150ms overhead mỗi request

Với connection pool:
  Pool khởi động: Tạo sẵn 10 connections
  Request 1: Lấy connection từ pool (0ms) + Query (10ms) + Trả lại = 10ms
  Request 2: Lấy connection (0ms) + Query (10ms) + Trả lại = 10ms
  → 16x nhanh hơn
```

Đây là lý do mọi database driver, HTTP client đều có connection pool.

## 3.4 HTTP — Protocol của Web

HTTP (HyperText Transfer Protocol) là protocol tầng ứng dụng xây trên TCP.

**HTTP/1.1:**
```
Client → Server: "GET /api/users HTTP/1.1"
Server → Client: "HTTP/1.1 200 OK\n{data}"
→ 1 request/response per connection (nếu không dùng keep-alive)
→ Head-of-line blocking: Nếu request 1 chậm, request 2 phải chờ
```

**HTTP/2:**
```
Cải tiến lớn:
  - Multiplexing: Nhiều request/response trên 1 TCP connection đồng thời
  - Header compression: HPACK giảm 30-90% overhead
  - Server push: Server chủ động gửi resource trước khi client hỏi
  - Binary protocol: Thay vì text → parse nhanh hơn

head-of-line blocking vẫn còn ở TCP level
```

**HTTP/3 (QUIC):**
```
Xây trên UDP thay vì TCP!
QUIC tự implement reliability, ordering
Loại bỏ head-of-line blocking hoàn toàn
0-RTT connection (không cần 3-way handshake cho connection đã biết)
→ Dùng cho YouTube, Google Search hiện nay
```

**HTTPS = HTTP + TLS:**

```
TLS Handshake (TLS 1.3 — 1-RTT):
  Client → Server: ClientHello (cipher suites supported)
  Server → Client: ServerHello + Certificate + Finished
  Client → Server: Finished + First encrypted request

TLS thêm ~1 RTT overhead (có thể 0-RTT với TLS 1.3 session resumption)
Sau đó: Mọi data đều được encrypt → không ai đọc được giữa đường
```

## 3.5 DNS — Internet's Phone Book

Khi bạn gõ `google.com`, máy tính không biết địa chỉ IP. DNS (Domain Name System) là hệ thống phân tán dịch domain → IP.

```
Browser gõ google.com:

1. Check browser cache → Miss
2. Check OS cache → Miss
3. Check /etc/hosts → Miss
4. Hỏi Recursive Resolver (thường là ISP hoặc 8.8.8.8)

Recursive Resolver:
5. Hỏi Root Name Server: "Ai quản lý .com?"
   → "Hỏi .com TLD servers tại 192.5.6.30"
6. Hỏi .com TLD: "Ai quản lý google.com?"
   → "Hỏi Google's authoritative server tại 216.239.32.10"
7. Hỏi Google's NS: "IP của google.com là gì?"
   → "142.250.185.46"
8. Trả về 142.250.185.46 (kèm TTL)
9. Browser cache lại, kết nối đến 142.250.185.46
```

**TTL (Time To Live):** DNS record có TTL — bao lâu được cache. Nếu TTL = 300s → mỗi 5 phút client phải hỏi lại.

**Tại sao quan trọng với system design:**
- Khi thay đổi IP server → phải đợi TTL hết → DNS propagation delay
- Load balancing bằng DNS: Trả về nhiều IP, client random chọn
- Geographic routing: Trả về IP server gần nhất dựa trên IP của client (Route 53, Cloudflare)
- Health check: Nếu server chết → xóa IP đó khỏi DNS response

---

# PHẦN II: DISTRIBUTED SYSTEMS — KHI MỘT MÁY KHÔNG ĐỦ

---

# CHƯƠNG 4: Tại Sao Phân Tán Và Cái Giá Phải Trả

## 4.1 Vấn đề của một máy

Một máy đơn (dù mạnh đến đâu) có giới hạn:

```
Giới hạn vật lý:
  CPU: Tốc độ clock không tăng nhiều sau 2005 (heat dissipation)
  RAM: Đắt, tối đa vài TB (high-end server)
  Disk I/O: Bị giới hạn bởi bandwidth của storage bus
  Network: 1-100 Gbps per NIC

Giới hạn khác:
  Single point of failure: 1 máy hỏng → toàn bộ sập
  Upgrade/maintenance → downtime
  Geographic limitation: Latency cao với user ở xa
```

**Giải pháp: Phân tán tải ra nhiều máy**

Nhưng đây không phải miễn phí. Phân tán mang lại:

## 4.2 Fallacies of Distributed Computing — 8 Sai Lầm Nghĩ Rằng Đúng

Peter Deutsch và James Gosling liệt kê 8 giả định sai mà developer hay mắc khi mới làm distributed system:

**1. "Mạng đáng tin cậy"**
```
Thực tế: Packet bị mất, TCP connection bị drop, switch hỏng
→ Code phải xử lý: timeout, retry, circuit breaker
```

**2. "Latency = 0"**
```
Thực tế: Gọi sang máy khác trong cùng datacenter = 0.5ms
         Gọi sang datacenter khác = 150ms
→ N+1 calls giữa microservices = thảm họa
```

**3. "Bandwidth vô hạn"**
```
Thực tế: Gửi 1GB data qua 1Gbps link mất 8 giây
→ Phải design payload nhỏ gọn, tránh over-fetching
```

**4. "Mạng an toàn"**
```
Thực tế: Packet có thể bị intercept, replay attack, man-in-the-middle
→ TLS cho tất cả communication, kể cả internal
```

**5. "Topology không thay đổi"**
```
Thực tế: IP thay đổi, server scale up/down, network reconfigure
→ Service discovery, không hardcode IP
```

**6. "Chỉ có một administrator"**
```
Thực tế: Multiple teams, multiple cloud providers, different SLAs
→ Defense in depth, không assume tất cả đều configured đúng
```

**7. "Transport cost = 0"**
```
Thực tế: Serialization/deserialization tốn CPU
         Network bandwidth tốn tiền
→ Batch requests, efficient serialization (protobuf thay JSON)
```

**8. "Mạng đồng nhất"**
```
Thực tế: IPv4 vs IPv6, TCP vs UDP, HTTP/1 vs HTTP/2, 
         Cloud vs on-premise, firewall rules khác nhau
→ Abstraction layer, không assume cùng network stack
```

## 4.3 Failure Modes — Distributed System Chết Thế Nào

**Crash Failure:** Node đơn giản là dừng (dễ detect nhất)

**Omission Failure:** Node không respond một số request (network drop)
```
Client → Server: Request
Server: Nhận request, xử lý xong
Server → Client: Response (BỊ MẤT trên đường)
Client: Timeout, nghĩ là server không nhận được → Gửi lại
Server: Nhận request lần 2, xử lý lần 2 → Tác vụ bị thực hiện 2 lần!
→ Idempotency là bắt buộc
```

**Timing Failure:** Node quá chậm
```
Server vẫn sống nhưng xử lý mất 30 giây thay vì 100ms
Client timeout sau 5 giây → Nghĩ là failure
→ Timeout phải được set hợp lý
```

**Byzantine Failure:** Node trả về kết quả sai (hiếm nhưng nguy hiểm nhất)
```
Corrupted memory → Tính toán sai
Malicious node → Gửi false data
→ Cần majority voting để detect (3 server vote, lấy kết quả đa số)
```

---

# CHƯƠNG 5: Scalability — Thiết Kế Để Lớn

## 5.1 Vertical vs Horizontal Scaling

**Vertical Scaling (Scale Up):**

```
Server hiện tại: 8 CPU, 32GB RAM
→ Nâng lên: 64 CPU, 512GB RAM

Ưu điểm:
  - Đơn giản, không cần thay đổi code
  - Không có distributed system complexity
  - Latency thấp (không có network hop)

Nhược điểm:
  - Giới hạn vật lý (máy đắt nhất cũng chỉ đến mức nào đó)
  - SPOF — 1 máy hỏng → tất cả sập
  - Downtime khi upgrade
  - Giá tăng phi tuyến: 2x hardware ≠ 2x giá, thường là 4x
```

**Horizontal Scaling (Scale Out):**

```
1 server → 10 server × (1/10 specs)

Ưu điểm:
  - Lý thuyết vô hạn
  - Không có SPOF (nếu thiết kế đúng)
  - Rẻ hơn (commodity hardware)
  - Zero-downtime deployment

Nhược điểm:
  - Application phải stateless
  - Distributed system complexity
  - Network overhead giữa các nodes
  - Khó debug hơn
```

## 5.2 Load Balancer — Phân Phối Traffic

Load balancer nhận request và phân phối cho các server phía sau.

**Layer 4 vs Layer 7 Load Balancer:**

```
L4 (TCP/IP level):
  Chỉ nhìn vào IP address và TCP port
  Nhanh hơn (không parse HTTP)
  Không thể routing dựa trên URL, cookie, header
  Ví dụ: AWS Network Load Balancer

L7 (Application level — HTTP):
  Đọc HTTP headers, URL, cookies
  Có thể route /api → API server, /static → File server
  SSL termination (decrypt HTTPS, forward plain HTTP nội bộ)
  Chậm hơn nhưng linh hoạt hơn
  Ví dụ: Nginx, AWS Application Load Balancer
```

**Thuật toán phân phối:**

*Round Robin:*
```
Request 1 → Server A
Request 2 → Server B
Request 3 → Server C
Request 4 → Server A  (vòng lại)
→ Đơn giản nhưng không tính đến load thực tế
```

*Weighted Round Robin:*
```
Server A: weight=3 (8 core)
Server B: weight=2 (4 core)
Server C: weight=1 (2 core)
→ A nhận 3 request, B nhận 2, C nhận 1 → proportional
```

*Least Connections:*
```
Luôn gửi đến server có ít active connection nhất
→ Tốt hơn khi request có thời gian xử lý khác nhau
```

*IP Hashing:*
```
server = hash(client_ip) % num_servers
→ Cùng client → luôn đến cùng server
→ Cần thiết cho sticky sessions
→ Nhưng: server thêm/bớt → hầu hết client đổi server
→ Dùng consistent hashing để giải quyết
```

**Health Check:**

```
Mỗi 5 giây, load balancer gửi:
  GET /health → Server

Server trả 200 OK:
  → Alive, tiếp tục gửi traffic

Server không respond hoặc trả 5xx:
  → Mark as down, ngừng gửi traffic
  → Alert on-call engineer

Health check endpoint phải:
  - Kiểm tra DB connection còn sống không
  - Kiểm tra critical dependencies
  - Trả về trong < 100ms
  - Không phải chính health check là bottleneck
```

**Session Persistence (Sticky Sessions):**

```
Vấn đề: User login → token lưu trong Server A memory
         Request tiếp → load balancer gửi đến Server B → 401 Unauthorized!

Giải pháp 1 — Sticky Session:
  Load balancer nhớ: User X → luôn gửi đến Server A
  Nhược: Server A hỏng → User X mất session

Giải pháp 2 — Stateless Server (TỐTT HƠN):
  Lưu session trong Redis/DB, không trong server memory
  Mọi server đều có thể xử lý mọi user
  → Horizontal scaling thực sự
```

## 5.3 Caching — Tầng Tốc Độ Giữa Mọi Thứ

Cache là bản copy của data ở vị trí nhanh hơn, gần hơn.

**Tại sao cache hiệu quả — Power Law / Zipf's Law:**

Trong hầu hết hệ thống thực tế, 20% data chiếm 80% request (law 80/20). Twitter: 0.01% tweets (viral) chiếm 20% read traffic. Netflix: 20% content chiếm 80% views.

→ Cache 20% hot data → phục vụ 80% request mà không cần đến DB.

**Cache Hit/Miss:**

```
Cache Hit Rate = (requests served from cache) / (total requests)

Nếu hit rate = 90%:
  90% request → Cache (0.1ms)
  10% request → DB (10ms)
  Average: 0.9×0.1 + 0.1×10 = 0.09 + 1 = 1.09ms

So với không cache:
  100% request → DB (10ms)
  Average: 10ms

→ 9x cải thiện chỉ với 90% hit rate
```

**Cache Invalidation — Vấn đề khó nhất:**

```
Phil Karlton: "There are only two hard things in Computer Science:
cache invalidation and naming things."

Vấn đề: Data trong DB thay đổi → Cache cũ → Serve stale data

Chiến lược:
  TTL (Time-To-Live): Cache tự expire sau N giây
    → Đơn giản nhưng có thể serve stale data trong N giây
    → Đúng với data không cần real-time (product info: 5 phút là OK)

  Write-invalidate: Khi ghi DB → xóa cache key
    → Cache miss lần đầu sau update, nhưng không serve stale
    → Vấn đề: Thundering herd nếu nhiều key bị xóa cùng lúc

  Write-through: Khi ghi DB → cập nhật cache luôn
    → Cache luôn fresh
    → Vấn đề: Write chậm hơn, cache có data ít được đọc
```

**Cache Stampede (Thundering Herd):**

```
Tình huống:
  Key "popular_product" expire lúc 12:00:00
  Lúc 12:00:00, có 10,000 concurrent request cho key này
  
  Tất cả 10,000 → Cache miss
  Tất cả 10,000 → Query DB đồng thời
  DB bị overload → Slow query → Timeout → DB chết
  
Giải pháp — Mutex/Lock:
  1 request được phép query DB, những cái khác đợi
  Request đó populate cache
  Những cái khác đọc từ cache
  
  if cache.get(key) is None:
      lock = redis.set(f"lock:{key}", 1, nx=True, ex=10)
      if lock:
          value = db.query(...)
          cache.set(key, value)
          redis.delete(f"lock:{key}")
      else:
          time.sleep(0.1)
          return cache.get(key)  # retry

Giải pháp — Jitter + Soft TTL:
  Thay vì expire đúng giờ, expire sớm hơn ngẫu nhiên 0-10%
  Không bao giờ có 10,000 request miss cùng lúc
```

## 5.4 Database Scaling

**Read Replicas:**

```
                    ┌─────────────────────┐
   Writes ─────────►│   Primary (Master)  │
                    └──────────┬──────────┘
                               │ Replication (async hoặc sync)
              ┌────────────────┼────────────────┐
              ▼                ▼                ▼
         [Replica 1]      [Replica 2]      [Replica 3]
              ▲                ▲                ▲
   Reads ─────┴────────────────┴────────────────┘
         (load balanced)

Replication lag: Thường < 1ms trong cùng datacenter
                 Vài giây trong cross-region

Dùng khi: Read >> Write (80-20 rule phổ biến)
```

**Connection Pooling:**

```
Vấn đề: Mỗi DB connection tốn:
  - ~5-10MB RAM trên DB server
  - ~50ms setup time
  - Threads trên DB side

100 app servers × 100 connections = 10,000 connections
PostgreSQL max_connections = 100-1000 (nếu cao hơn → OOM)

Giải pháp: PgBouncer/ProxySQL đứng trước DB
  10,000 client connections → PgBouncer → 100 real DB connections
  PgBouncer multiplexes connections

Pool modes:
  Session pooling: 1 client = 1 DB connection trong suốt session
  Transaction pooling: 1 DB connection = nhiều client transactions
  Statement pooling: 1 DB connection = nhiều statements (không support transactions)
```

**Sharding:**

```
Bảng users có 1 tỷ rows, 1TB data → 1 server không đủ

Horizontal sharding (partition rows):
  Shard 0: user_id 0 → 250M
  Shard 1: user_id 250M → 500M
  Shard 2: user_id 500M → 750M
  Shard 3: user_id 750M → 1B

Application phải tính: shard = user_id / 250M
Hoặc: shard = hash(user_id) % 4

Cross-shard queries:
  SELECT COUNT(*) FROM users WHERE country = 'VN'
  → Phải query tất cả 4 shards → Merge kết quả
  → Chậm và phức tạp

Resharding: Thêm shard 5 → Phải di chuyển data
  → Consistent hashing giảm thiểu lượng data phải move
```

---

# CHƯƠNG 6: Availability và Reliability

## 6.1 Định nghĩa và SLA

**Availability:** Tỷ lệ thời gian hệ thống hoạt động đúng.

```
Availability = Uptime / (Uptime + Downtime)

99%    = 3.65 ngày downtime/năm    (không acceptable với production)
99.9%  = 8.76 giờ downtime/năm    (acceptable cho nhiều non-critical)
99.95% = 4.38 giờ downtime/năm
99.99% = 52.6 phút downtime/năm   (high availability)
99.999% = 5.26 phút downtime/năm  (five nines — banking, telecom)
```

**SLA (Service Level Agreement):** Cam kết availability với khách hàng.
**SLO (Service Level Objective):** Target nội bộ (thường cao hơn SLA một chút làm buffer).
**SLI (Service Level Indicator):** Metric thực tế đo được.

**Availability của hệ thống phụ thuộc vào availability của từng component:**

```
Sequential dependencies (tất cả phải hoạt động):
  A (99.9%) → B (99.9%) → C (99.9%)
  System = 99.9% × 99.9% × 99.9% = 99.7%
  
  Thêm component → Availability giảm!

Parallel redundancy (chỉ cần 1 trong N):
  A (99.9%) + A_replica (99.9%)
  System = 1 - (1-99.9%)² = 1 - (0.1%)² = 1 - 0.0001% = 99.9999%
  
  → Parallel redundancy tăng availability đáng kể
```

## 6.2 SPOF và Redundancy

**SPOF (Single Point of Failure):** Thứ mà nếu hỏng → toàn bộ sập.

```
Hệ thống thường gặp:
  Single DB server → Primary + Replica
  Single load balancer → Active + Standby LB
  Single datacenter → Multi-region deployment
  Single DNS provider → Multiple DNS providers
  Single ISP → Multiple network uplinks
```

**Eliminating SPOF ở mọi tầng:**

```
Tầng Network:
  Multiple ISPs, BGP routing
  Redundant network switches

Tầng Server:
  Redundant power supplies
  RAID cho disks
  Multiple NICs (bonding)

Tầng Application:
  Multiple instances behind load balancer
  Stateless servers
  Health checks + auto-restart

Tầng Database:
  Primary + Replicas
  Automatic failover (Patroni for PostgreSQL)
  Regular backups + PITR

Tầng Geographic:
  Multiple availability zones
  Multi-region active-active hoặc active-passive
```

## 6.3 Replication — Đồng Bộ Dữ Liệu Giữa Nodes

**Synchronous Replication:**

```
Client → Primary: WRITE data
Primary → Replica: Replicate
Primary ← Replica: ACK "Written"
Primary → Client: "Success" ← Chỉ trả về sau khi Replica confirm

Đảm bảo: Không mất data nếu Primary crash sau khi confirm
Chi phí: Latency cao hơn (phải chờ Replica), Replica slow → ảnh hưởng Primary
```

**Asynchronous Replication:**

```
Client → Primary: WRITE data
Primary → Client: "Success" ← Trả về ngay, không chờ Replica
Primary → Replica: Replicate (background)

Đảm bảo: Nhanh hơn, Primary không bị block
Chi phí: Có thể mất data nếu Primary crash trước khi Replica nhận
         Replica có thể lag behind Primary (replication lag)
```

**Replication Lag — Vấn đề thực tế:**

```
User update profile name → Ghi vào Primary
User reload page → Đọc từ Replica (để giảm tải Primary)
Replica lag = 2 giây → User thấy tên cũ!

Giải pháp:
  1. Read-your-writes: Sau khi write, đọc từ Primary trong N giây
  2. Monotonic reads: Cùng user → luôn đọc từ cùng Replica
  3. Reduce replica lag: Tăng bandwidth replication, tối ưu replica hardware
```

## 6.4 Consensus — Nhiều Nodes Đồng Ý Với Nhau Thế Nào

**Vấn đề:** Trong distributed system, làm sao để 3 nodes đồng ý về "node nào là Primary"?

Không thể dùng "majority vote" đơn giản vì:
```
3 nodes: A, B, C
A nghĩ B là primary
B nghĩ A là primary
C bị network partition, không liên lạc được với ai
→ A và B cùng process writes → Split-brain → Data diverge
```

**Raft Consensus Algorithm (dễ hiểu hơn Paxos):**

```
Roles: Leader, Follower, Candidate

1. Ban đầu: Tất cả là Follower
2. Election Timeout: Follower không nhận được heartbeat từ Leader
   → Chuyển thành Candidate
   → Vote cho chính mình
   → Request vote từ các nodes khác

3. Nếu nhận được majority votes → Trở thành Leader
   (majority = N/2 + 1 nodes)

4. Leader gửi heartbeat đến tất cả → Prevent new election

5. Write:
   Client → Leader
   Leader → Log entry (uncommitted)
   Leader → Replicate to Followers
   Majority ACK → Commit entry
   Leader → Client: "Success"

6. Nếu Leader chết:
   Followers không nhận heartbeat → Election mới
   Node có log mới nhất thắng election → New Leader
```

Raft đảm bảo: Chỉ có 1 leader tại mọi thời điểm, data được committed thì không bị mất.

Được dùng trong: etcd (Kubernetes), CockroachDB, TiKV, Consul.

---

# CHƯƠNG 7: CAP Theorem và Consistency Models

## 7.1 CAP Theorem — Giới Hạn Của Distributed Systems

**CAP Theorem** (Brewer, 2000): Distributed system không thể đồng thời đảm bảo cả 3:

```
C — Consistency: 
  Mọi node đều thấy data giống nhau cùng lúc.
  Đọc luôn trả về giá trị mới nhất đã được write.

A — Availability:
  Mọi request đều nhận được response (không phải error).
  Không nhất thiết là giá trị mới nhất.

P — Partition Tolerance:
  Hệ thống tiếp tục hoạt động dù có network partition
  (một số nodes không liên lạc được với nhau).
```

**Network partition LUÔN có thể xảy ra** trong distributed system. Vậy phải chọn P. Trade-off thực sự là **C vs A**.

```
Khi có network partition:
  Cluster bị chia thành 2 nhóm: {A, B} và {C, D}
  
  Chọn Consistency (CP):
    Nhóm nhỏ hơn ({C, D}) từ chối service requests
    → Availability bị hy sinh
    → Nhưng không bao giờ trả về stale data
    
  Chọn Availability (AP):
    Tất cả nodes tiếp tục service requests
    → Data có thể diverge giữa 2 nhóm
    → Sau khi partition heal → Conflict resolution cần thiết
```

**Ví dụ thực tế:**

```
CP systems:
  PostgreSQL (sync replication): Primary từ chối write nếu Replica không ACK
  HBase, Zookeeper: Dùng Raft/Paxos
  → Phù hợp: Banking, financial transactions

AP systems:
  Cassandra, DynamoDB: Vẫn accept writes dù có partition
  → Eventual consistency: Sau partition heal, data sẽ sync
  → Phù hợp: Social media, shopping cart, non-critical data

Lưu ý: Trong thực tế, "CA" không có nghĩa lý (partition không thể tránh khỏi)
```

## 7.2 PACELC — Mở Rộng Thực Tế Hơn

CAP chỉ nói về khi có partition. **PACELC** nói thêm: ngay cả khi bình thường (no partition), vẫn phải trade-off:

```
If Partition: chọn A hoặc C
Else (bình thường): chọn L (Latency) hoặc C (Consistency)

Vì sao?
  Strong Consistency → Phải đợi tất cả replicas acknowledge
  → Latency cao hơn

  Low Latency → Chỉ đợi 1 node (có thể stale)
  → Consistency kém hơn
```

**Ví dụ phân loại:**

```
PA/EL: DynamoDB, Cassandra
  Partition → Availability; Normal → Latency
  
PC/EC: PostgreSQL sync, MySQL with sync replication
  Partition → Consistency; Normal → Consistency (chờ replica)
  
PA/EC: Cosmos DB (configurable)
  Partition → Availability; Normal → Consistency
```

## 7.3 Consistency Models — Nhiều Mức Độ Nhất Quán

Không phải chỉ "strong" hay "eventual" — có nhiều mức độ:

**Linearizability (Strongest):**
```
Mọi operation có vẻ xảy ra tại đúng 1 điểm trong thời gian thực.
Đọc luôn thấy giá trị của write gần nhất.

Ví dụ:
  T=1: A write x=1
  T=2: B read x → PHẢI thấy x=1
  T=2: C read x → PHẢI thấy x=1 (không thể thấy x=0)
```

**Sequential Consistency:**
```
Kết quả giống như tất cả operations được thực thi theo một thứ tự tuần tự nào đó.
Thứ tự đó phải nhất quán với thứ tự của từng process riêng lẻ.
Cho phép lag nhỏ nhưng mọi process thấy cùng một sequence.
```

**Causal Consistency:**
```
Nếu A causes B (B phụ thuộc A), mọi process thấy A trước B.
Events không có quan hệ nhân quả có thể thấy theo thứ tự khác.

Ví dụ:
  Alice đăng bài → Bob comment vào bài đó
  Charlie PHẢI thấy bài trước comment (causal order)
  Charlie có thể thấy 2 bài của Alice theo thứ tự khác nhau
  (vì chúng không có causal relationship)
```

**Eventual Consistency (Weakest):**
```
Nếu không có write mới → Cuối cùng tất cả replicas sẽ converge
về cùng giá trị.

Không có đảm bảo về thời gian.
Không có đảm bảo về thứ tự.

Amazon: "Shopping cart cho phép stale data (user có thể thấy
item đã xóa một lúc), nhưng add-to-cart KHÔNG BAO GIỜ bị mất"
→ Highly available, eventual consistent
```

**Chọn consistency model:**

```
Cần strong consistency:
  ✓ Bank account balance
  ✓ Inventory count (1 item cuối)
  ✓ Distributed lock (ai lock file này)
  
Chấp nhận eventual consistency:
  ✓ Social media like count
  ✓ View count trên video
  ✓ Shopping cart (với conflict resolution)
  ✓ DNS propagation (có thể mất vài phút để spread)
```

---

# CHƯƠNG 8: Messaging — Giao Tiếp Bất Đồng Bộ

## 8.1 Tại Sao Cần Async Communication

Synchronous call: A gọi B, chờ B trả lời mới tiếp tục.

```
Vấn đề khi chaining:
  User request → Order Service → Payment Service → Inventory → Email

  Nếu Email Service chậm (5 giây) → User chờ 5 giây!
  Nếu Email Service chết → Toàn bộ order fail?!
  
  Temporal coupling: Tất cả services phải available cùng lúc
```

Async messaging: A gửi message vào queue, tiếp tục. B đọc queue và xử lý.

```
User request → Order Service → Queue (order.created)
User nhận response ngay lập tức

Background:
  Payment Service đọc queue → Process payment
  Inventory Service đọc queue → Reserve item
  Email Service đọc queue → Send confirmation

Nếu Email Service chết:
  Message vẫn trong queue
  Email Service khởi động lại → Đọc message → Gửi email
  → Eventual delivery, không mất data
```

## 8.2 Message Queue vs Event Streaming

**Message Queue (RabbitMQ, ActiveMQ, SQS):**

```
Producer → [Queue] → Consumer

Semantics:
  Mỗi message được xử lý bởi đúng 1 consumer
  Message bị xóa sau khi consumer ACK
  Push-based hoặc pull-based
  
Dùng khi:
  Task distribution (job queue)
  Work needs to be done exactly once
  Consumer slower than producer → buffer
```

**Event Streaming (Kafka, Kinesis):**

```
Producer → [Topic/Stream] → Consumer Group A
                          → Consumer Group B
                          → Consumer Group C

Semantics:
  Message được giữ lại (retention period)
  Multiple consumer groups đọc độc lập
  Consumer tự track offset (vị trí đọc)
  Pull-based
  
Dùng khi:
  Event sourcing
  Multiple consumers cần cùng data
  Audit log, replay events
  Real-time analytics
```

## 8.3 Kafka Deep Dive

Kafka là distributed event streaming platform, không phải simple queue.

**Architecture:**

```
Producers                  Kafka Cluster              Consumers
                    ┌──────────────────────┐
[Order Service] ──► │  Broker 0            │ ◄── [Payment Service]
[User Service]  ──► │    Topic: orders     │ ◄── [Analytics Service]
                    │      Partition 0     │ ◄── [Email Service]
                    │      Partition 1     │
                    │      Partition 2     │
                    │  Broker 1            │
                    │    (replicas)        │
                    └──────────────────────┘
```

**Partition — Đơn Vị Parallelism:**

```
Topic "orders" → 3 partitions
  Partition 0: order_id % 3 = 0
  Partition 1: order_id % 3 = 1
  Partition 2: order_id % 3 = 2

Consumer Group "payment-processors" có 3 consumers:
  Consumer A → Partition 0 (parallel)
  Consumer B → Partition 1 (parallel)
  Consumer C → Partition 2 (parallel)

Throughput = 3 × single partition throughput

Ordering đảm bảo TRONG partition, không phải across partitions.
→ Nếu cần order tất cả events của user X → Partition by user_id
```

**Offset:**

```
Partition 0: [msg1] [msg2] [msg3] [msg4] [msg5]
              offset=0    1      2      3      4

Consumer đọc đến offset 3:
  → Biết cần đọc tiếp từ offset 4
  → Offset được commit vào __consumer_offsets topic trong Kafka

Consumer crash ở offset 3:
  → Restart → Đọc offset từ Kafka → Biết cần bắt đầu từ offset 4
  → Không xử lý lại msgs đã processed
  
Consumer muốn replay:
  → Reset offset về 0 → Đọc lại từ đầu
  → Kafka giữ message theo retention policy (7 ngày, 100GB...)
```

**Exactly-Once Semantics:**

```
At-most-once: Gửi message, không quan tâm có received không
  → Có thể mất message

At-least-once: Retry nếu không nhận được ACK
  → Có thể duplicate message
  → Consumer phải idempotent

Exactly-once: Kafka's idempotent producer + transactional API
  → Producer dedup ở broker level
  → Consumer+DB trong cùng transaction
  → Đắt nhất nhưng an toàn nhất
  
Thực tế: Most systems dùng at-least-once + idempotent consumer
```

---

# CHƯƠNG 9: Microservices Architecture

## 9.1 Monolith Trước — Hiểu Vấn Đề Rồi Mới Hiểu Giải Pháp

**Monolithic Architecture:**

```
┌─────────────────────────────────────────────────────────┐
│                 Single Deployable Unit                  │
│                                                         │
│  [User Module] [Order Module] [Payment Module]          │
│  [Catalog]     [Search]       [Recommendation]          │
│  [Analytics]   [Notification] [Auth]                    │
│                                                         │
│  Shared Database                                        │
└─────────────────────────────────────────────────────────┘
```

**Khi monolith bắt đầu đau:**

```
Scale:
  Search cần nhiều CPU (indexing), Payment cần nhiều RAM
  → Phải scale TOÀN BỘ app, dù chỉ 1 phần cần scale
  → Waste resources

Deployment:
  100 developer cùng commit vào 1 codebase
  1 change nhỏ → Build + Test toàn bộ → Deploy toàn bộ
  Deploy 1 service → Downtime toàn bộ (nếu không blue-green)

Technology:
  Tất cả phải dùng cùng language, framework, DB
  Khó adopt công nghệ mới cho 1 phần

Fault isolation:
  Memory leak trong Analytics → Toàn bộ app OOM
  Bug trong Notification → Order bị ảnh hưởng
```

## 9.2 Microservices — Chia Nhỏ Theo Domain

```
[API Gateway] ─┬──► [User Service]      → [User DB]
               ├──► [Order Service]     → [Order DB]
               ├──► [Payment Service]   → [Payment DB]
               ├──► [Catalog Service]   → [Catalog DB]
               └──► [Search Service]    → [Elasticsearch]

Communication:
  Sync: REST/gRPC khi cần response ngay
  Async: Kafka khi không cần response ngay
```

**Bounded Context (Domain-Driven Design):**

```
Mỗi service sở hữu domain của nó:
  Order Service: Tất cả về đơn hàng (tạo, sửa, hủy, trạng thái)
  Payment Service: Tất cả về thanh toán (không biết về order logic)
  
"User" trong Order Service ≠ "User" trong Payment Service:
  Order Service: user_id, address, cart
  Payment Service: user_id, payment_methods, billing_address
  → Mỗi service tự định nghĩa model của nó
  → Không share database trực tiếp
```

**Microservices Trade-offs:**

```
Lợi ích:
  ✓ Độc lập deploy (team A deploy service của mình không ảnh hưởng team B)
  ✓ Scale độc lập (Payment cần HA, scale riêng)
  ✓ Fault isolation (Search chết không ảnh hưởng Checkout)
  ✓ Technology heterogeneity (Python cho ML, Go cho high-throughput)
  ✓ Team autonomy (Conway's Law)

Chi phí:
  ✗ Distributed system complexity (network failures, latency)
  ✗ Data consistency (không có cross-service transactions đơn giản)
  ✗ Operational complexity (deploy N services, monitoring N services)
  ✗ Testing khó hơn (integration test phức tạp)
  ✗ Latency: Function call (ns) → Network call (ms)
```

## 9.3 API Gateway — Entry Point của Microservices

```
External Clients
       │
       ▼
┌─────────────────────────────────────────────────────┐
│                   API Gateway                       │
│                                                     │
│  Authentication/Authorization                       │
│  Rate Limiting                                      │
│  Request Routing                                    │
│  Load Balancing                                     │
│  SSL Termination                                    │
│  Request/Response Transformation                    │
│  Circuit Breaking                                   │
│  Logging & Monitoring                               │
└──────────┬──────────┬──────────┬────────────────────┘
           │          │          │
           ▼          ▼          ▼
    [User Service] [Order] [Payment]
```

**Tại sao không để client gọi thẳng vào services?**

```
Client phải biết địa chỉ của từng service
Client phải tự handle authentication cho từng service
Client phải tự handle retry, timeout, circuit breaker
Mobile app: Cần gọi 5 services để hiển thị 1 page → 5 round trips → chậm

API Gateway giải quyết:
  1 entry point → Gateway fan-out ra các services → Aggregate response
  Cross-cutting concerns (auth, rate limit) ở 1 chỗ
  Clients không biết internal architecture
```

**BFF (Backend For Frontend):**

```
Thay vì 1 API Gateway chung:
  Mobile App → Mobile BFF (trả về data compact cho mobile)
  Web App → Web BFF (trả về data đầy đủ hơn)
  Third Party → Public API (rate limited, documented)

Tại sao: Mobile cần data khác với web (ít field hơn, format khác)
         Không muốn public API bị ảnh hưởng bởi mobile-specific changes
```

## 9.4 Service Discovery — Services Tìm Nhau Thế Nào

```
Vấn đề: Microservices scale in/out liên tục
        IP thay đổi, port thay đổi
        Không thể hardcode "Payment Service tại 10.0.0.5:8080"

Giải pháp: Service Registry

Client-side discovery:
  Service đăng ký vào Registry khi khởi động
  Client query Registry để tìm IP:port của service
  Client tự load balance
  
  [Payment Service] ──register──► [Registry: Consul/Etcd]
  [Order Service] ──query──► [Registry] ──returns──► 10.0.0.5:8080
  [Order Service] ──call──► 10.0.0.5:8080

Server-side discovery:
  Client → Load Balancer → LB query Registry → Route to service
  Client không cần biết Registry
  Đơn giản hơn với client, thêm hop mạng
```

## 9.5 Saga Pattern — Distributed Transactions

**Vấn đề:**
```
Đặt hàng cần:
  1. Order Service: Tạo order
  2. Payment Service: Charge thẻ
  3. Inventory Service: Reserve item
  4. Shipping Service: Schedule shipment

Không có distributed transaction (mỗi service có DB riêng).
Step 3 fail → Step 1, 2 đã xảy ra → Inconsistent!
```

**Saga: Chuỗi local transactions với compensating transactions**

```
Choreography Saga (event-driven):

Order Service:    CREATE order (PENDING) 
                  → emit "order.created"

Payment Service:  listen "order.created"
                  CHARGE card
                  → emit "payment.completed"
                  
Inventory Service: listen "payment.completed"
                  RESERVE item
                  → emit "inventory.reserved"

If inventory fail:
  Inventory Service: emit "inventory.failed"
  
  Payment Service: listen "inventory.failed"
                   REFUND card
                   → emit "payment.refunded"
                   
  Order Service: listen "payment.refunded"
                 UPDATE order (FAILED)
```

**Orchestration Saga:**
```
Saga Orchestrator điều phối tất cả (như conductor):

Orchestrator:
  1. Call Payment → OK
  2. Call Inventory → FAIL
  3. Call Payment.compensate() → Refund
  4. Update order status = FAILED
  5. Notify user

Đơn giản hơn choreography về logic
Thêm single point of failure (orchestrator)
```

---

# CHƯƠNG 10: Caching Strategies và CDN

## 10.1 Cache ở Mọi Tầng

```
Browser Cache
    ↓
CDN (Edge Cache)
    ↓
Load Balancer Cache (ít dùng)
    ↓
Application Cache (Redis/Memcached)
    ↓
Database Query Cache
    ↓
Database Buffer Pool (in-memory pages)
    ↓
OS Page Cache
    ↓
CPU Cache
    ↓
Disk
```

Mỗi tầng có hit → không cần xuống tầng dưới. Tầng càng cao → nhanh nhất.

## 10.2 CDN — Content Delivery Network

**Tại sao cần CDN:**

```
User ở Hà Nội, server ở Virginia (US):
  RTT ≈ 150ms
  Tải ảnh 100KB: 150ms latency + transfer time

User ở Hà Nội, CDN PoP ở Singapore:
  RTT ≈ 30ms
  Tải ảnh 100KB: 30ms latency + transfer time (cache hit)
  → 5x nhanh hơn
```

**CDN hoạt động thế nào:**

```
Lần đầu user từ VN request image.jpg:
  CDN PoP Singapore: Cache miss
  → Fetch từ Origin Server (Virginia): 150ms
  → Cache tại Singapore: image.jpg (TTL = 24h)
  → Trả về user: 150ms (chậm lần đầu)

Lần thứ 2 user khác từ VN request image.jpg:
  CDN PoP Singapore: Cache hit!
  → Trả về từ Singapore: 30ms (nhanh hơn 5x)
```

**Push vs Pull CDN:**

```
Pull (Lazy):
  CDN fetch từ origin khi có request và cache miss
  Không cần upload gì cả
  Lần đầu request slow (origin fetch)
  
Push (Eager):
  Upload file lên CDN trước
  CDN distribute đến tất cả PoPs
  Không có cold start latency
  Phức tạp hơn khi update file

Trong thực tế: Hầu hết dùng Pull với TTL hợp lý
```

**Cache Invalidation ở CDN:**

```
Deploy code mới → CSS/JS thay đổi → CDN vẫn serve file cũ

Giải pháp 1 — URL versioning:
  /static/app.v2.3.1.css (thay đổi filename khi content thay đổi)
  CDN cache mãi mãi (max-age=31536000)
  Deploy mới → URL mới → CDN tự cache file mới
  → Không cần invalidate

Giải pháp 2 — Content-based hashing:
  /static/app.8f3d2a.css (hash của content)
  Same content → Same URL → Cache hit
  Changed content → New URL → Cache miss → Fetch mới
  
Giải pháp 3 — Purge API:
  CDN cung cấp API để purge cache
  Deploy mới → Gọi CDN API → Purge /static/app.css
  Lần sau request → Cache miss → Fetch từ origin
  → Immediate update nhưng tốn CDN API call
```

## 10.3 Redis — Distributed Cache

Redis (Remote Dictionary Server) không chỉ là cache — là in-memory data structure store.

**Data Structures:**

```
String: SET key value / GET key
  → Cache, counter, session token
  SET session:abc123 '{"user_id": 1}' EX 3600

Hash: HSET key field value / HGET key field
  → Object storage
  HSET user:1 name "An" email "an@gmail.com" score 100

List: LPUSH/RPUSH/LPOP/RPOP
  → Queue, recent items
  RPUSH job_queue '{"task": "send_email"}'
  BLPOP job_queue 0  (blocking pop — worker pattern)

Set: SADD/SREM/SMEMBERS/SINTER/SUNION
  → Unique items, tags, mutual friends
  SADD user:1:friends 2 3 4
  SADD user:2:friends 1 3 5
  SINTER user:1:friends user:2:friends  → {3} (mutual friends)

Sorted Set: ZADD/ZRANGE/ZRANK
  → Leaderboard, priority queue, rate limiting
  ZADD leaderboard 1500 "user:1"
  ZADD leaderboard 2300 "user:2"
  ZREVRANGE leaderboard 0 9 WITHSCORES  (top 10)
```

**Redis Persistence:**

```
Redis là in-memory → Restart mất hết? Có option:

RDB (Snapshotting):
  Mỗi N giây, fork process và dump memory ra file
  SAVE 900 1  (save sau 900s nếu ≥1 key thay đổi)
  SAVE 60 10000 (save sau 60s nếu ≥10000 thay đổi)
  
  Nhanh khi restart (load 1 file)
  Có thể mất data giữa 2 lần snapshot

AOF (Append-Only File):
  Log mọi write command vào file
  appendfsync everysec (flush mỗi giây)
  
  Durable hơn RDB (mất tối đa 1 giây data)
  File lớn hơn, restart chậm hơn
  Hỗ trợ AOF rewrite (compact log)

Hybrid (Redis 4.0+):
  RDB snapshot + AOF từ checkpoint đó
  Restart nhanh (load RDB) + Durable (replay AOF)
```

---

# CHƯƠNG 11: Observability — Biết Hệ Thống Đang Làm Gì

## 11.1 Ba Trụ Cột: Metrics, Logs, Traces

**Metrics — Số liệu tổng hợp:**

```
"Hệ thống đang hoạt động tốt không?"

Counter: Chỉ tăng
  http_requests_total{method="GET", status="200"} 12345

Gauge: Tăng/giảm
  memory_used_bytes 4294967296
  active_connections 847

Histogram: Phân phối giá trị
  http_request_duration_seconds_bucket{le="0.1"} 9000
  http_request_duration_seconds_bucket{le="0.5"} 9800
  http_request_duration_seconds_bucket{le="1.0"} 9950
  → p90 < 0.5s, p99 < 1.0s
```

**Logs — Sự kiện cụ thể:**

```
"Chuyện gì đã xảy ra?"

Structured logging (JSON):
{
  "timestamp": "2024-01-01T12:00:00Z",
  "level": "ERROR",
  "service": "order-service",
  "trace_id": "abc123",
  "user_id": 42,
  "message": "Payment failed",
  "error": "Card declined",
  "duration_ms": 150
}

Unstructured logging (bad):
  "2024-01-01 ERROR Payment failed for user 42"

Tại sao structured: Query được, filter được
  → Elasticsearch/Loki: "Tất cả ERROR của user_id=42 trong 1 giờ qua"
```

**Distributed Tracing — Theo dõi request qua nhiều services:**

```
"Request này chậm vì service nào?"

Request từ user:
  Trace ID: abc-123
  
  API Gateway [Span 1: 5ms]
    → Order Service [Span 2: 150ms]
        → DB Query [Span 3: 140ms] ← CHẬM!
        → Redis [Span 4: 1ms]
    → Payment Service [Span 5: 20ms]

Mỗi service inject Trace ID vào log và forward đến next service.
Jaeger/Zipkin thu thập tất cả spans → Reconstruct toàn bộ trace.
```

## 11.2 The Four Golden Signals

**1. Latency — Bao lâu để xử lý 1 request:**
```
Dùng percentile, không dùng average:
  p50: Người dùng điển hình
  p95: 1/20 người dùng
  p99: 1/100 người dùng (thường là worst case thực tế)
  p99.9: Cực đoan (spike, outliers)

Không dùng average: 1000 request × 10ms + 1 request × 10,000ms
  Average = ~20ms (có vẻ OK)
  p99.9 = 10,000ms (1/1000 user chờ 10 giây!)
```

**2. Traffic — Bao nhiêu demand:**
```
QPS (Queries Per Second)
RPM (Requests Per Minute)
Bytes/second (cho storage, network)

Dùng để:
  Capacity planning
  Detect anomalies (đột tăng = DDoS, viral; đột giảm = bug)
  Compare với baseline (today vs last week same time)
```

**3. Errors — Bao nhiêu request fail:**
```
Error rate = errors / total × 100%

Phân loại:
  4xx (client errors): 400, 401, 403, 404
    → Không phải lỗi của server, nhưng cần monitor 404 spike (broken links)
  5xx (server errors): 500, 502, 503, 504
    → Lỗi của server → Alert ngay

SLO ví dụ: Error rate < 0.1% trong 5 phút rolling window
```

**4. Saturation — Hệ thống gần đầy chưa:**
```
CPU: > 80% sustained → Scale
Memory: > 90% → Danger zone (OOM killer)
Disk: > 85% → Alert (logs và data grow fast)
Network: > 80% bandwidth → Congestion
DB connections: > 80% pool size → Add more connections

Saturation thường là leading indicator:
  High CPU → Latency sẽ tăng → Error rate sẽ tăng
  → Monitor saturation để act TRƯỚC khi user bị ảnh hưởng
```

## 11.3 Alerting — Khi Nào Thì Báo Động

```
Nguyên tắc alert tốt:

1. Actionable: Mỗi alert phải có hành động cụ thể
   BAD: "CPU > 80%" (phải làm gì? Không rõ)
   GOOD: "Error rate > 1% trong 5 phút → Check /health, logs"

2. Alert on symptoms, not causes:
   BAD: "Replica lag > 100ms" (user không cảm nhận được)
   GOOD: "p99 latency > 2s" (user cảm nhận được)

3. Tránh alert fatigue:
   Quá nhiều false positive → Engineer ignore alerts → Miss real issue
   → Tune thresholds cẩn thận, có grace period

4. Severity levels:
   P0 (Critical): Production down, revenue impact → Page immediately
   P1 (High): Degraded service, potential revenue → Page in 15 min
   P2 (Medium): Non-critical issue → Ticket, fix next business day
   P3 (Low): Minor issue → Backlog
```

---

# PHẦN III: THIẾT KẾ HỆ THỐNG THỰC TẾ

---

# CHƯƠNG 12: Framework Thiết Kế Hệ Thống

## 12.1 Quy Trình Thiết Kế (RESHADED framework)

**1. Requirements (5 phút):**
```
Functional requirements: Hệ thống làm gì?
  - User có thể làm gì?
  - Core features là gì? (MVP)
  
Non-functional requirements: Hệ thống phải tốt thế nào?
  - Scale: Bao nhiêu user? QPS?
  - Availability: 99.9% hay 99.99%?
  - Consistency: Strong hay eventual?
  - Latency: p99 < bao nhiêu ms?
  - Durability: Có thể mất data không?
```

**2. Estimation (5 phút):**
```
DAU: Daily Active Users
QPS: (DAU × requests_per_user) / 86400
Peak QPS: ~2-10x average QPS
Storage: Tính per-year, per-record
Bandwidth: QPS × average_payload_size
```

**3. High-Level Design (10 phút):**
```
Vẽ các components chính:
  Client → API Gateway → Services → Storage
  
Identify primary components:
  - Database type (SQL vs NoSQL)
  - Caching layer
  - Message queue (nếu cần async)
  - CDN (nếu có static content)
```

**4. Deep Dive (20 phút):**
```
Đi sâu vào từng component:
  Database schema
  API design
  Caching strategy
  Algorithm choice
```

**5. Trade-offs và Bottlenecks:**
```
Hỏi và trả lời:
  - Điểm yếu nhất của thiết kế này là gì?
  - Nếu traffic tăng 10x thì phần nào break trước?
  - Trade-offs: Đã đánh đổi gì?
```

## 12.2 Back-of-Envelope Estimation

**Các số cần nhớ:**

```
Latency:
  L1 cache: 0.5ns
  RAM access: 100ns
  SSD read: 0.1ms
  Network (same DC): 0.5ms
  Network (cross-region): 150ms
  HDD seek: 10ms

Throughput:
  SSD sequential read: 500MB/s
  Network: 1-100 Gbps
  Single server: ~10K-100K QPS (depends on workload)
  
Size:
  1 char = 1 byte
  1 UUID = 16 bytes
  1 row (typical): 100-1000 bytes
  1 image (avatar): 100KB
  1 HD video per second: ~2MB
  
Time:
  1 day = 86,400 seconds ≈ 10^5 seconds
  1 month = 2.5 × 10^6 seconds
  1 year = 3 × 10^7 seconds
```

**Ví dụ: Design Instagram (Photos)**

```
Assumptions:
  500M users, 1M active daily (DAU = 1M)
  Each user uploads 1 photo/day average
  Read:Write ratio = 100:1 (mỗi ảnh upload được xem 100 lần)

Write QPS:
  1M uploads / 86400s ≈ 12 uploads/second

Read QPS:
  12 × 100 = 1200 reads/second
  Peak: ~5000 reads/second

Storage (photos):
  1 photo ≈ 1MB average
  12 uploads/s × 86400 × 365 ≈ 378B uploads/year
  378B × 1MB = 378PB/year ← Need object storage (S3)

Storage (metadata):
  1 record (user_id, photo_url, caption, created_at) ≈ 500 bytes
  378B records × 500 bytes = 189TB/year ← Need sharded DB

Bandwidth:
  Upload: 12 × 1MB = 12MB/s
  Download: 1200 × 1MB = 1.2GB/s
  → Need CDN for downloads
```

## 12.3 Thiết Kế URL Shortener

**Requirements:**
```
Functional:
  - Tạo short URL từ long URL
  - Redirect short → long
  - Custom alias (tùy chọn)
  - Analytics (click count)

Non-functional:
  - 100M URLs total
  - 1B redirects/ngày
  - Redirect latency < 10ms (p99)
  - Availability: 99.9%
  - Data không bao giờ mất
```

**Estimation:**
```
Redirects QPS: 1B / 86400 ≈ 12K reads/second
               Peak: ~50K reads/second

New URLs QPS: 100M / (3 years × 365 × 86400) ≈ 1 write/second
              → Write rất ít so với read (12000:1)

Storage:
  1 URL record: short(7B) + long(100B) + metadata(50B) = 157 bytes
  100M × 157B = 15.7GB ← Rất nhỏ, fit vào RAM!

Bandwidth:
  Read: 50K × 157B = 7.85MB/s (nhỏ)
```

**High-Level Design:**

```
User → [API Gateway] → [URL Service] → [Cache (Redis)] → [DB]
                                              ↑ miss
                                           [DB read]

Write flow:
  POST /shorten { url: "https://very.long.url" }
  → Generate short code
  → Store: {short: "abc123", long: "https://...", created_at}
  → Cache: cache.set("abc123", "https://...", ttl=24h)
  → Return: "https://short.ly/abc123"

Read flow:
  GET /abc123
  → Cache lookup: O(1)
  → Hit: 301/302 Redirect
  → Miss: DB lookup → Cache populate → Redirect
```

**Short Code Generation:**

```
Base62 (a-zA-Z0-9): 62^7 = 3.5 tỷ combinations
(100M URLs cần, có 35x buffer)

Option 1 — Hash-based:
  hash = md5(long_url)  # 128-bit
  short = base62(hash[:7])  # Lấy 7 chars đầu
  
  Collision risk: 3.5B / 100M = 3.5% chance
  → Cần collision detection: Check DB, retry with different segment
  
Option 2 — Counter + Base62 (tốt hơn):
  global_counter = AtomicCounter()
  short = base62(global_counter.increment())
  
  Đơn giản, no collision
  Sequential → Predictable (minor security concern)
  Single point of failure nếu counter là 1 server
  → Dùng distributed ID generator (Twitter Snowflake)

Option 3 — Pre-generated pool:
  Background job generate random 7-char codes
  Store vào "available_codes" table
  Service lấy code từ pool khi cần
  → No collision, fast, but complex
```

**Redirect: 301 vs 302:**

```
301 Permanent Redirect:
  Browser cache permanently
  Subsequent requests không qua server
  → Giảm server load
  → Nhưng: Không track clicks sau lần đầu, khó update

302 Temporary Redirect:
  Browser không cache
  Mọi request đều qua server
  → Có thể track mọi click
  → Server load cao hơn
  
Chọn 302 nếu cần analytics, 301 nếu muốn giảm load tối đa
```

**Scale:**

```
Read scaling:
  Redis cache toàn bộ 15.7GB (fit vào 1 Redis instance 32GB)
  Cache hit rate ≈ 99% (Zipf's law — 20% URLs = 80% traffic)
  → Với 99% hit rate, DB chỉ nhận 1% × 50K = 500 QPS
  → 1 DB server đủ

High availability:
  Redis: Sentinel hoặc Cluster mode
  DB: Primary + 2 Replicas
  API servers: >= 3 instances behind LB
  Deploy: Multi-AZ

Analytics:
  Ghi click events vào Kafka (async, non-blocking cho redirect)
  Analytics consumer xử lý background
  → Redirect latency không bị ảnh hưởng
```

## 12.4 Thiết Kế Social Media Feed (Twitter/Instagram)

**Requirements:**
```
Functional:
  - Đăng post/tweet
  - Follow/Unfollow user
  - Xem home feed (posts của people I follow)
  - Xem profile feed

Non-functional:
  - 300M DAU
  - Đăng: 1,700 tweets/second
  - Đọc feed: 300K reads/second (cực kỳ read-heavy)
  - Feed load < 200ms (p99)
```

**Vấn đề cốt lõi: Fan-out**

```
User A có 1M followers đăng 1 tweet:
  → Cần đưa tweet này vào feed của 1M followers
  → 1M write operations cho 1 single tweet
  → 1700 tweets/second × average followers... = Enormous write load

Đây là tradeoff giữa:
  Fast write (write fan-out) vs Fast read (pre-computed feed)
  Slow write (no fan-out) vs Slow read (compute feed at read time)
```

**Pull Model (Fan-out on Read):**

```
Khi user mở feed:
  SELECT posts FROM posts
  WHERE user_id IN (
    SELECT following_id FROM follows WHERE follower_id = me
  )
  ORDER BY created_at DESC
  LIMIT 20;

Write: O(1) — chỉ lưu 1 post
Read: O(following × posts) — Query chậm nếu follow nhiều người

Ví dụ: User follow 5,000 accounts
  → Merge 5,000 timelines
  → Đọc 5,000 × N posts → Sort → Take top 20
  → Không thể chịu được
```

**Push Model (Fan-out on Write):**

```
Khi user đăng tweet:
  FOR EACH follower of user:
    INSERT INTO feed_{follower_id} (tweet_id, created_at)
  
  1M followers → 1M inserts PER TWEET
  1700 tweets/second, average 100 followers = 170K inserts/second
  1 tweet từ celebrity 1M followers = 1M inserts tức thì

Write: O(followers) — Đắt khi celebrity tweet
Read: O(1) — Đọc pre-computed feed cực nhanh

SELECT tweet_id FROM feed_{me} ORDER BY created_at DESC LIMIT 20
→ Already computed, index on (user_id, created_at)
```

**Hybrid Model (Twitter thực tế):**

```
Regular users (< 1M followers): Push model
  → Fan-out khi tweet, feed pre-computed

Celebrities (> 1M followers): Pull model
  → Không fan-out
  → Khi user load feed: Merge pre-computed feed + celebrity tweets

Algorithm:
  Load home feed = 
    pre_computed_feed (push model, regular follows)
    + recent_celebrity_tweets (pull model, fetched live)
    + ranked by ML model (không chỉ chronological)
```

**Feed Storage:**

```
Redis Sorted Set (tốt cho timeline):
  Key: feed:{user_id}
  Score: timestamp
  Value: tweet_id

ZADD feed:123 1704067200 "tweet:456"
ZREVRANGE feed:123 0 19  (20 tweets mới nhất)

TTL: 30 ngày (cũ hơn → xóa, lazy load từ DB nếu user scroll xuống)
```

## 12.5 Thiết Kế Hệ Thống Chat (WhatsApp)

**Requirements:**
```
Functional:
  - 1-1 messaging
  - Group chat (tối đa 200 members)
  - Online/offline status
  - Read receipts (delivered, read)
  - Media sharing

Non-functional:
  - 2B users, 100M DAU
  - 100B messages/ngày
  - Delivery latency < 100ms (cùng region)
  - Messages không bao giờ mất
```

**Challenge: Realtime Bidirectional Communication**

HTTP truyền thống là request-response — client hỏi, server trả lời. Chat cần **server push**: Server gửi message cho client khi có message mới.

**WebSocket:**

```
HTTP Upgrade handshake:
  Client → Server: GET /chat HTTP/1.1
                   Upgrade: websocket
                   Connection: Upgrade
  
  Server → Client: HTTP/1.1 101 Switching Protocols
                   Upgrade: websocket
  
Sau đó: Full-duplex TCP connection
  Server → Client: Bất kỳ lúc nào (no request needed)
  Client → Server: Bất kỳ lúc nào
  
  Low latency (persistent connection, no HTTP overhead per message)
```

**Architecture:**

```
[Client A] ──WebSocket──► [Chat Server 1] ──► [Message Queue (Kafka)]
[Client B] ──WebSocket──► [Chat Server 2]          ↓
[Client C] ──WebSocket──► [Chat Server 1]   [Message Service]
                                                    ↓
                                            [Message DB (Cassandra)]
                                                    ↓
                                            [Notification Service] (push nếu offline)
```

**Message Flow:**

```
User A gửi message cho User B:

1. A → WebSocket → Chat Server 1: "Hello B"
2. Chat Server 1 → Message DB: Lưu message
3. Chat Server 1 → Kafka: message.sent event
4. Message Router: "B đang connect đến Chat Server 2"
5. Chat Server 2 → B: Deliver message qua WebSocket
6. B → Chat Server 2: ACK "delivered"
7. Chat Server 2 → A: "Message delivered" 
8. B đọc message → Chat Server 2 → A: "Message read"
```

**User Presence (Online/Offline):**

```
Vấn đề: Biết user nào đang online?

Solution:
  Khi user kết nối WebSocket → SET user:{id}:online 1 EX 30 (Redis)
  Mỗi 25 giây, client gửi heartbeat → Renew TTL
  Nếu không có heartbeat 30 giây → Key expire → User offline

Query presence:
  GET user:{id}:online → Exists = Online, không tồn tại = Offline
  
  "Is user online?" = O(1) Redis lookup
```

**Message Storage — Cassandra:**

```
Tại sao Cassandra:
  100B messages/ngày = 1.16M messages/second
  Write-heavy → Cassandra's strength
  Cần query: "Give me last 50 messages of conversation X"
  → Range query trên partition key

Schema:
  Table: messages
  Partition key: conversation_id
  Clustering key: message_id (time-based, descending)
  
  → Tất cả messages của 1 conversation cùng partition
  → Range query "last N messages" = single partition scan = O(log N)
  
  CQLSH:
  CREATE TABLE messages (
    conversation_id uuid,
    message_id timeuuid,
    sender_id bigint,
    content text,
    media_url text,
    PRIMARY KEY (conversation_id, message_id)
  ) WITH CLUSTERING ORDER BY (message_id DESC);
```

---

# TỔNG KẾT — Roadmap Học System Design

```
Foundation (Hiểu trước khi làm):
  Computer Architecture
  → OS: Process, Thread, I/O Models
  → Networking: OSI, TCP/UDP, HTTP, DNS
  → Concurrency: Race conditions, Locks, Deadlocks

Distributed Systems Basics:
  → Fallacies of distributed computing
  → Failure modes
  → CAP Theorem, Consistency Models
  → Consensus (Raft)

Scalability Patterns:
  → Horizontal/Vertical scaling
  → Load balancing
  → Caching (Redis, CDN)
  → Database scaling (Replicas, Sharding)

Architecture Patterns:
  → Microservices vs Monolith
  → API Gateway, Service Discovery
  → Messaging (Kafka)
  → Saga Pattern

Observability:
  → Metrics, Logs, Traces
  → Four Golden Signals
  → Alerting

Thực hành:
  → URL Shortener
  → Social Feed
  → Chat System
  → Search Autocomplete
  → Rate Limiter
  → Notification System
  → Video Streaming (YouTube)
  → Ride Sharing (Uber)

Mỗi hệ thống: Requirements → Estimation → Design → Trade-offs
```

**Câu hỏi cần hỏi khi thiết kế bất kỳ hệ thống nào:**

```
1. Scale: Bao nhiêu user, QPS, data?
2. Read/Write ratio: Optimize cho cái nào?
3. Consistency requirement: Strong hay eventual?
4. Availability requirement: Bao nhiêu 9?
5. Latency requirement: p99 phải là bao nhiêu?
6. Bottleneck: Phần nào sẽ fail trước?
7. SPOF: Có single point of failure không?
8. Hot spots: Có data partition nào nhận traffic lớn không thường?
9. Security: Auth, authorization, encryption in transit và at rest?
10. Cost: Trade-off giữa hiệu năng và chi phí?
```