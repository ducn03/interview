# \# 📦 Cache — Giải thích từ cơ bản đến chuyên sâu

# 

# > \*\*Cache\*\* là nơi lưu trữ tạm thời dữ liệu nhằm giúp những lần truy cập sau \*\*nhanh hơn đáng kể\*\*, bằng cách tránh phải tính toán hoặc truy xuất lại từ nguồn gốc.

# 

# \---

# 

# \## 1. Cache là gì?

# 

# Hãy tưởng tượng bạn đang tra cứu một định nghĩa trong từ điển:

# 

# \- \*\*Lần đầu\*\*: Bạn lật sách, tìm trang → mất 30 giây

# \- \*\*Lần sau\*\*: Bạn còn nhớ → gần như ngay lập tức

# 

# 👉 "Nhớ lại" chính là cache trong não bạn — lưu kết quả để không phải tìm lại từ đầu.

# 

# Trong phần mềm, ý tưởng hoàn toàn tương tự: \*\*thay vì tính lại hoặc truy vấn lại\*\*, ta lưu kết quả cũ ở chỗ nào đó nhanh hơn để dùng lại.

# 

# \---

# 

# \## 2. Vấn đề cache giải quyết

# 

# \### Không có cache

# 

# Mỗi request đều phải:

# 

# 1\. Kết nối database

# 2\. Thực thi query (có thể phức tạp, JOIN nhiều bảng)

# 3\. Database đọc dữ liệu từ \*\*disk\*\* (rất chậm)

# 4\. Trả kết quả về

# 

# \*\*Hệ quả:\*\*

# \- Response chậm (hàng trăm ms mỗi request)

# \- Database bị quá tải khi nhiều user

# \- Hệ thống dễ sập khi traffic đột biến

# 

# \### Có cache

# 

# 1\. Request đến → kiểm tra cache trước

# 2\. Nếu có → trả về \*\*ngay lập tức từ RAM\*\*

# 3\. Nếu không có → mới gọi DB, rồi lưu lại vào cache

# 

# \*\*Hệ quả:\*\*

# \- Response nhanh hơn 10–100x

# \- Giảm tải đáng kể cho database

# \- Hệ thống scale tốt hơn nhiều

# 

# \---

# 

# \## 3. Bản chất của cache: Trade-off

# 

# Cache là một \*\*sự đánh đổi có chủ đích\*\*:

# 

# | Bỏ ra | Đổi lấy |

# |---|---|

# | RAM (đắt, giới hạn) | Tốc độ cực nhanh |

# | Độ phức tạp hệ thống | Hiệu năng cao hơn |

# | Tính nhất quán tuyệt đối | Khả năng phục vụ nhiều user hơn |

# 

# \*\*Tại sao RAM nhanh hơn Disk?\*\*

# 

# | Loại bộ nhớ | Tốc độ đọc điển hình | So sánh |

# |---|---|---|

# | CPU L1 Cache | \~1 ns | Cực nhanh |

# | RAM | \~100 ns | Nhanh |

# | SSD (NVMe) | \~100 µs | Chậm hơn RAM \~1000x |

# | HDD | \~10 ms | Rất chậm |

# 

# 👉 Đó là lý do cache thường được lưu trong RAM.

# 

# \---

# 

# \## 4. Cache hoạt động như thế nào?

# 

# \### Flow cơ bản

# 

# ```

# Client → Server → \[Check Cache]

# &#x20;                      |

# &#x20;             ┌────────┴────────┐

# &#x20;          Cache HIT         Cache MISS

# &#x20;             |                   |

# &#x20;        Trả về ngay         Gọi DB

# &#x20;        từ cache            → Lưu vào cache

# &#x20;                            → Trả về client

# ```

# 

# \### Cache Hit vs Cache Miss

# 

# | | Cache Hit ✅ | Cache Miss ❌ |

# |---|---|---|

# | Dữ liệu | Có sẵn trong cache | Không có |

# | Tốc độ | Rất nhanh (\~µs) | Chậm hơn (\~ms) |

# | Nguồn dữ liệu | RAM | Database / API |

# 

# \*\*Hit Rate\*\* = Tỷ lệ request tìm thấy dữ liệu trong cache

# 

# > Mục tiêu: \*\*tối đa hóa hit rate\*\* — thường target từ 80–99% tùy hệ thống.

# 

# \---

# 

# \## 5. Tại sao cache hoạt động hiệu quả? — Locality

# 

# Cache khai thác một đặc điểm tự nhiên của hành vi truy cập dữ liệu:

# 

# \### Temporal Locality (Cục bộ theo thời gian)

# 

# > Dữ liệu vừa được truy cập có \*\*khả năng cao\*\* sẽ được truy cập lại trong thời gian ngắn.

# 

# Ví dụ: Trang chủ của một website, thông tin sản phẩm nổi bật.

# 

# \### Spatial Locality (Cục bộ theo không gian)

# 

# > Dữ liệu gần nhau về mặt địa chỉ thường được truy cập cùng nhau.

# 

# Ví dụ: Đọc một record → thường cũng cần các field liên quan ngay bên cạnh.

# 

# > ⚠️ Nếu dữ liệu hoàn toàn ngẫu nhiên, không có pattern → cache gần như vô dụng.

# 

# \---

# 

# \## 6. Các khái niệm cốt lõi

# 

# \### 6.1 TTL (Time To Live)

# 

# Thời gian sống của một entry trong cache. Sau khi hết TTL, entry bị xóa và phải load lại từ nguồn.

# 

# ```

# User A  ──→ Cache \[user:123, TTL=5min] ──→ DB

# &#x20;             ↑                              |

# &#x20;             └──────────────────────────────┘

# &#x20;                    5 phút sau → xóa

# ```

# 

# \*\*Chọn TTL như thế nào?\*\*

# 

# | Loại dữ liệu | TTL gợi ý |

# |---|---|

# | Cấu hình hệ thống | 1 giờ – 24 giờ |

# | Thông tin sản phẩm | 5 – 30 phút |

# | Session user | 15 – 60 phút |

# | Kết quả tìm kiếm | 1 – 5 phút |

# | Giá cổ phiếu / tỷ giá | Không nên cache hoặc TTL rất ngắn |

# 

# \### 6.2 Eviction Policy (Chiến lược xóa khi đầy)

# 

# Cache có dung lượng giới hạn. Khi đầy, phải quyết định xóa entry nào.

# 

# | Chiến lược | Cơ chế | Dùng khi nào |

# |---|---|---|

# | \*\*LRU\*\* (Least Recently Used) | Xóa entry lâu nhất chưa dùng | Phổ biến nhất, phù hợp hầu hết |

# | \*\*LFU\*\* (Least Frequently Used) | Xóa entry ít được truy cập nhất | Khi cần ưu tiên theo tần suất |

# | \*\*FIFO\*\* | Xóa entry vào trước nhất | Đơn giản, ít dùng trong production |

# | \*\*TTL-based\*\* | Xóa khi hết hạn, không cần đầy | Kết hợp với các chiến lược trên |

# | \*\*Random\*\* | Xóa ngẫu nhiên | Đơn giản, hiệu quả không cao |

# 

# \### 6.3 Cache Invalidation (Xóa cache chủ động)

# 

# Khi dữ liệu nguồn thay đổi, cache phải được cập nhật hoặc xóa.

# 

# Đây là bài toán khó nhất trong caching:

# 

# > \*"There are only two hard things in Computer Science: cache invalidation and naming things."\*

# > — Phil Karlton

# 

# \*\*Các cách invalidate cache:\*\*

# 

# \- \*\*Xóa trực tiếp\*\*: Khi update DB, xóa luôn key cache liên quan

# \- \*\*TTL tự nhiên\*\*: Để cache tự hết hạn

# \- \*\*Event-driven\*\*: Dùng message queue để notify khi data thay đổi

# \- \*\*Versioned keys\*\*: Đổi tên key mỗi khi update (`user:123:v2`)

# 

# \---

# 

# \## 7. Cache Patterns

# 

# \### 7.1 Cache Aside (Lazy Loading) ⭐ Phổ biến nhất

# 

# \*\*Flow đọc:\*\*

# 1\. App kiểm tra cache

# 2\. Miss → App gọi DB

# 3\. App lưu kết quả vào cache

# 4\. Trả về client

# 

# \*\*Flow ghi:\*\*

# \- Ghi vào DB, sau đó xóa hoặc update cache

# 

# ```

# App ──→ Cache (miss) ──→ DB ──→ App ──→ Cache (lưu lại)

# ```

# 

# \*\*Ưu điểm:\*\* Linh hoạt, dễ implement, cache chỉ chứa dữ liệu thực sự được dùng  

# \*\*Nhược điểm:\*\* 3 lần round-trip khi miss; có thể bị stale nếu không invalidate đúng

# 

# 👉 \*\*Dùng khi:\*\* Đọc nhiều hơn ghi, không yêu cầu cache luôn up-to-date tuyệt đối.

# 

# \---

# 

# \### 7.2 Read Through

# 

# \*\*Flow:\*\* App chỉ nói chuyện với cache. Cache layer tự biết cách load từ DB khi miss.

# 

# ```

# App ──→ Cache ──→ (nếu miss) DB

# &#x20;             ←── (tự lưu lại)

# ```

# 

# \*\*Ưu điểm:\*\* Logic đơn giản hơn ở tầng App  

# \*\*Nhược điểm:\*\* Cache layer phức tạp hơn (phải biết kết nối DB)

# 

# 👉 \*\*Dùng khi:\*\* Muốn tách bạch hoàn toàn logic cache khỏi App.

# 

# \---

# 

# \### 7.3 Write Through

# 

# \*\*Flow ghi:\*\* Ghi đồng thời vào cache VÀ DB trong cùng một transaction.

# 

# ```

# App ──→ Cache ──→ DB (đồng bộ)

# ```

# 

# \*\*Ưu điểm:\*\* Cache luôn nhất quán với DB  

# \*\*Nhược điểm:\*\* Ghi chậm hơn (phải chờ cả DB); cache có thể chứa nhiều dữ liệu không bao giờ được đọc

# 

# 👉 \*\*Dùng khi:\*\* Cần tính nhất quán cao giữa cache và DB.

# 

# \---

# 

# \### 7.4 Write Back (Write Behind)

# 

# \*\*Flow ghi:\*\* Ghi vào cache trước, DB được cập nhật \*\*bất đồng bộ\*\* sau.

# 

# ```

# App ──→ Cache (ghi ngay) ──→ DB (async, sau vài giây/phút)

# ```

# 

# \*\*Ưu điểm:\*\* Tốc độ ghi rất nhanh; giảm số lần write vào DB (batching)  

# \*\*Nhược điểm:\*\* Nguy cơ mất data nếu cache crash trước khi sync xuống DB

# 

# 👉 \*\*Dùng khi:\*\* Hệ thống cần throughput ghi cực cao (gaming leaderboard, analytics, IoT).

# 

# \---

# 

# \### So sánh các patterns

# 

# | Pattern | Tốc độ đọc | Tốc độ ghi | Độ nhất quán | Độ phức tạp |

# |---|---|---|---|---|

# | Cache Aside | ⭐⭐⭐ | ⭐⭐⭐ | Trung bình | Thấp |

# | Read Through | ⭐⭐⭐⭐ | ⭐⭐⭐ | Trung bình | Trung bình |

# | Write Through | ⭐⭐⭐⭐ | ⭐⭐ | Cao | Trung bình |

# | Write Back | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | Thấp | Cao |

# 

# \---

# 

# \## 8. Các vấn đề thực tế thường gặp

# 

# \### 8.1 Cache Stampede (Thundering Herd)

# 

# \*\*Vấn đề:\*\* Nhiều cache key hết hạn cùng lúc (hoặc một key hot bị xóa) → hàng trăm request đồng thời ập vào DB.

# 

# ```

# TTL hết → 500 requests cùng gọi DB → DB chết

# ```

# 

# \*\*Giải pháp:\*\*

# \- \*\*Mutex/Lock\*\*: Chỉ 1 request được phép rebuild cache, các request khác chờ

# \- \*\*Staggered TTL\*\*: Thêm random jitter vào TTL (ví dụ: 5 phút ± 30 giây) để tránh expire cùng lúc

# \- \*\*Background refresh\*\*: Refresh cache proactively trước khi hết TTL

# \- \*\*Probabilistic early expiration\*\*: Bắt đầu refresh sớm hơn khi TTL còn ít

# 

# \---

# 

# \### 8.2 Cache Penetration

# 

# \*\*Vấn đề:\*\* Attacker (hoặc bug) liên tục request những key \*\*không tồn tại\*\* trong cả cache lẫn DB. Cache luôn miss → DB bị spam.

# 

# ```

# GET /user/99999999 (không tồn tại)

# → Cache miss

# → DB query (không có)

# → Không lưu cache

# → Lặp lại mãi → DB quá tải

# ```

# 

# \*\*Giải pháp:\*\*

# \- \*\*Cache null value\*\*: Lưu giá trị `null` hoặc `"NOT\_FOUND"` vào cache với TTL ngắn

# \- \*\*Bloom Filter\*\*: Kiểm tra nhanh key có tồn tại không trước khi query DB (false positive nhưng không false negative)

# 

# \---

# 

# \### 8.3 Cache Breakdown (Hot Key Expiry)

# 

# \*\*Vấn đề:\*\* Một key \*\*cực kỳ hot\*\* bỗng nhiên hết TTL → tất cả request ập vào DB cho đúng 1 key đó.

# 

# Khác với Stampede ở chỗ: chỉ 1 key, nhưng traffic vào key đó rất lớn.

# 

# \*\*Giải pháp:\*\*

# \- \*\*Mutex\*\*: Khóa khi rebuild cache

# \- \*\*Không dùng TTL cho hot key\*\*: Quản lý invalidation thủ công

# \- \*\*Replica cache\*\*: Nhiều bản copy của hot key

# 

# \---

# 

# \### 8.4 Cache Avalanche

# 

# \*\*Vấn đề:\*\* Toàn bộ cache server bị down (crash, restart), hoặc rất nhiều key cùng hết TTL → 100% traffic đổ vào DB.

# 

# \*\*Giải pháp:\*\*

# \- \*\*Circuit Breaker\*\*: Tự động reject request khi DB quá tải thay vì để hệ thống chết hoàn toàn

# \- \*\*Multi-layer cache\*\*: L1 (local) + L2 (distributed Redis) → nếu Redis down vẫn có local cache

# \- \*\*Staggered TTL\*\* (tương tự Stampede)

# \- \*\*Rate limiting\*\*: Giới hạn số request vào DB khi cache down

# 

# \---

# 

# \## 9. Các loại cache trong thực tế

# 

# \### 9.1 Local Cache (In-process)

# 

# Cache nằm \*\*trong bộ nhớ của chính process\*\* chạy ứng dụng.

# 

# \- \*\*Ví dụ:\*\* `HashMap` trong Java, `dict` trong Python, `node-cache` trong Node.js

# \- \*\*Ưu điểm:\*\* Cực nhanh (không có network), zero latency

# \- \*\*Nhược điểm:\*\* Không chia sẻ được giữa nhiều instance; bị mất khi restart; khó đồng bộ

# 

# 👉 \*\*Dùng khi:\*\* Single-instance app, dữ liệu ít thay đổi (config, lookup tables).

# 

# \---

# 

# \### 9.2 Distributed Cache

# 

# Cache \*\*chạy trên server riêng biệt\*\*, nhiều instance app cùng chia sẻ.

# 

# | | Redis | Memcached |

# |---|---|---|

# | Cấu trúc dữ liệu | String, Hash, List, Set, ZSet, ... | Chỉ String |

# | Persistence | Có (RDB, AOF) | Không |

# | Clustering | Có (Redis Cluster) | Có (nhưng phức tạp hơn) |

# | Pub/Sub | Có | Không |

# | Phổ biến | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ |

# 

# 👉 \*\*Redis\*\* là lựa chọn mặc định trong hầu hết hệ thống hiện đại.

# 

# \---

# 

# \### 9.3 Client-Side Cache (Browser)

# 

# Trình duyệt cache HTML, CSS, JS, hình ảnh, API response.

# 

# \*\*Cơ chế:\*\* HTTP headers

# \- `Cache-Control: max-age=3600` — cache trong 1 giờ

# \- `ETag` — kiểm tra xem file có thay đổi không

# \- `Last-Modified` — so sánh thời gian cập nhật

# 

# \---

# 

# \### 9.4 CDN Cache

# 

# \*\*CDN (Content Delivery Network)\*\* cache tài nguyên tĩnh ở các \*\*server gần người dùng\*\* về mặt địa lý.

# 

# ```

# User ở Hà Nội → CDN server tại Hà Nội (cache) → nhanh

# User ở Hà Nội → Origin server ở US (không cache) → chậm

# ```

# 

# \*\*Ví dụ:\*\* Cloudflare, AWS CloudFront, Fastly

# 

# \---

# 

# \### 9.5 Database Cache

# 

# Chính bản thân database cũng có cache nội bộ:

# 

# \- \*\*Buffer Pool\*\* (MySQL InnoDB, PostgreSQL): Cache các data page thường xuyên đọc trong RAM

# \- \*\*Query Cache\*\*: Cache kết quả của các query giống nhau (MySQL đã deprecated vì nhiều vấn đề)

# \- \*\*Index\*\*: Không phải cache nhưng cũng giảm số lần đọc disk

# 

# \---

# 

# \## 10. Stale Data — Vấn đề không thể tránh

# 

# Cache \*\*KHÔNG phải source of truth\*\*. Nó là bản sao, và bản sao có thể lỗi thời.

# 

# ```

# 1\. Cache lưu: user:123 = {name: "Nguyễn Văn A"}

# 2\. User đổi tên thành "Nguyễn Văn B" trong DB

# 3\. Cache chưa update

# 4\. Request tiếp theo nhận được tên cũ "Nguyễn Văn A"

# ```

# 

# \*\*Mức độ chấp nhận stale data phụ thuộc vào business:\*\*

# 

# | Dữ liệu | Stale OK? |

# |---|---|

# | Số like của bài post | Vài phút OK |

# | Thông tin sản phẩm | Vài phút OK |

# | Số dư tài khoản ngân hàng | Không OK — cần real-time |

# | Tồn kho sản phẩm (flash sale) | Không OK — cần real-time |

# 

# \---

# 

# \## 11. Khi nào KHÔNG nên dùng cache?

# 

# \- \*\*Dữ liệu thay đổi quá thường xuyên\*\*: Cache liên tục bị invalidate → hit rate thấp → không có lợi

# \- \*\*Yêu cầu real-time tuyệt đối\*\*: Số dư tài khoản, trạng thái đặt vé, tồn kho flash sale

# \- \*\*Dữ liệu mỗi user khác nhau hoàn toàn\*\* và ít được reuse

# \- \*\*Hệ thống nhỏ, traffic thấp\*\*: Thêm cache chỉ làm tăng độ phức tạp mà không có lợi rõ ràng

# \- \*\*Dữ liệu nhạy cảm\*\* cần kiểm soát truy cập chặt chẽ (lưu cache sai chỗ có thể leak)

# 

# \---

# 

# \## 12. Checklist khi thiết kế cache

# 

# Trước khi thêm cache, hãy tự hỏi:

# 

# \- \[ ] \*\*Dữ liệu có được reuse không?\*\* Nếu mỗi request cần dữ liệu khác nhau → cache vô nghĩa

# \- \[ ] \*\*Hit rate dự kiến là bao nhiêu?\*\* Dưới \~50% → xem lại có nên cache không

# \- \[ ] \*\*TTL hợp lý là bao lâu?\*\* Quá dài → stale data; quá ngắn → hit rate thấp

# \- \[ ] \*\*Invalidation strategy là gì?\*\* Ai xóa cache khi data thay đổi?

# \- \[ ] \*\*Có xử lý được Cache Stampede không?\*\* Đặc biệt với hot key

# \- \[ ] \*\*Cache failure thì sao?\*\* App có fallback về DB được không?

# \- \[ ] \*\*Security\*\*: Cache có lưu dữ liệu nhạy cảm không? Có bị user A đọc data của user B không?

# 

# \---

# 

# \## 13. Mindset đúng về cache

# 

# > Cache không phải source of truth. Nó là \*\*bản sao tạm thời để tăng tốc\*\*, và bản sao có thể sai.

# 

# Hai điều luôn phải chấp nhận khi dùng cache:

# 

# 1\. \*\*Complexity\*\*: Hệ thống phức tạp hơn — phải quản lý invalidation, handle stampede, monitor hit rate

# 2\. \*\*Eventual consistency\*\*: Có khoảng thời gian ngắn mà cache và DB không đồng bộ

# 

# Hãy thiết kế hệ thống sao cho \*\*khoảng thời gian không đồng bộ đó nằm trong ngưỡng chấp nhận được\*\* của business.

# 

# \---

# 

# \## 14. Tóm tắt

# 

# ```

# Cache = Lưu kết quả cũ vào bộ nhớ nhanh hơn

# &#x20;     → Lần sau không cần tính lại / query lại

# &#x20;     → Đánh đổi: RAM + complexity <> tốc độ + scalability

# ```

# 

# \*\*Key takeaways:\*\*

# 

# \- Cache tăng tốc hệ thống bằng cách khai thác Temporal \& Spatial Locality

# \- Bản chất là trade-off: tốc độ đổi lấy tính nhất quán và độ phức tạp

# \- Pattern phổ biến nhất: \*\*Cache Aside\*\* — check → miss → load DB → store

# \- Vấn đề khó nhất: \*\*Cache Invalidation\*\* — khi nào xóa cache?

# \- Redis là lựa chọn distributed cache mặc định trong industry

# \- Luôn hỏi: "Hệ thống của mình có thể chấp nhận stale data bao lâu?"

