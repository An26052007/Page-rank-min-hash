# BÀI HỌC LÝ THUYẾT: PAGERANK TOÀN CỤC (GLOBAL PAGERANK)
> **Dành cho người mới bắt đầu — Từ số 0 đến hiểu bản chất**  
> **Thư mục:** `extra/phan_a_pagerank_toan_cuc/`

---

## 🏝️ 1. Câu chuyện hòn đảo: Mạng Internet thực chất là gì?

Hãy tưởng tượng toàn bộ mạng Internet giống như một quần đảo gồm hàng triệu hòn đảo:
- Mỗi **trang web** (hoặc bài viết Wikipedia) là một **hòn đảo** (trong toán học gọi là **Đỉnh / Node**).
- Mỗi **đường link** (siêu liên kết) là một **cây cầu một chiều** nối từ đảo này sang đảo khác (gọi là **Cạnh có hướng / Directed Edge**).

```
[ Đảo A ] ═════ Cầu một chiều ═════> [ Đảo B ]
```

**Câu hỏi lớn của các nhà sáng lập Google (năm 1998):**  
*Làm sao để biết hòn đảo nào là "thủ đô" sầm uất, uy tín nhất, và hòn đảo nào chỉ là "bãi đất hoang" hẻo lánh?*

---

## 🗳️ 2. Ý tưởng ban đầu: Cơ chế bỏ phiếu (Voting Model)

Ý tưởng tự nhiên đầu tiên:
> **"Nếu hòn đảo A xây một cây cầu dẫn sang đảo B, điều đó chứng tỏ đảo A đang BỎ MỘT PHIẾU BẦU công nhận đảo B là nơi đáng để đến."**

Đảo nào càng nhận được nhiều cây cầu trỏ vào, đảo đó càng quan trọng!

### Nhưng cách bỏ phiếu ngây thơ này gặp phải 2 bất công lớn:
1. **Không phải lá phiếu nào cũng có giá trị như nhau:**
   - Một phiếu bầu từ một tờ báo lớn (như *Báo Tuổi Trẻ*, *Wikipedia*) phải có uy tín gấp triệu lần một phiếu bầu từ một trang web rác mới tạo hôm qua.
2. **Kẻ dễ dãi chia nhỏ giá trị:**
   - Nếu một trang web tốt bụng đặt tới $1,000$ đường link trỏ ra $1,000$ nơi khác nhau, giá trị uy tín mà mỗi trang đích nhận được phải bị chia đều ra $\frac{1}{1000}$.

---

## 🐒 3. Mô hình Chú khỉ lướt web ngẫu nhiên (Random Surfer)

Để giải quyết bài toán trên một cách công bằng tuyệt đối, các nhà khoa học đã nghĩ ra mô hình **Người lướt web ngẫu nhiên**:

Hãy thả một chú khỉ tinh nghịch lên quần đảo:
1. Chú khỉ bắt đầu ở một hòn đảo ngẫu nhiên.
2. Ở mỗi hòn đảo, chú nhìn quanh xem có bao nhiêu cây cầu dẫn ra ngoài ($d_j$ cây cầu). Chú nhắm mắt chọn bừa một cây cầu để chạy sang đảo tiếp theo.
3. Chú khỉ cứ chạy như vậy hàng triệu lần.
4. **Hòn đảo nào mà chú khỉ ghé thăm nhiều nhất, đảo đó chính là nơi quan trọng nhất!** Tỷ lệ phần trăm thời gian chú khỉ dừng chân tại mỗi đảo chính là **Điểm PageRank** của đảo đó.

---

## ⚠️ 4. Hai cái bẫy chết người trên mạng và Cách giải cứu

Khi cho chú khỉ chạy trên mạng Internet thực tế, mô hình sẽ gặp 2 hiểm họa:

### Bẫy 1: Spider Traps (Bẫy nhện)
- **Hiện tượng:** Có một nhóm 3 hòn đảo chỉ xây cầu qua lại cho nhau: $A \to B \to C \to A$. Không có bất kỳ cây cầu nào dẫn ra thế giới bên ngoài.
- **Hậu quả:** Khi chú khỉ vô tình lọt vào nhóm đảo này, chú sẽ bị kẹt vĩnh viễn bên trong. Điểm số của 3 đảo này sẽ hút sạch điểm của toàn bộ mạng Internet!
- **Cách giải cứu (Hệ số giảm $\beta = 0.85$):**
  Ta trang bị cho chú khỉ một chiếc **dù bay (Teleportation)**:
  - Với xác suất **$85\%$ ($\beta = 0.85$)**: Chú khỉ ngoan ngoãn đi qua các cây cầu bình thường.
  - Với xác suất **$15\%$ ($1 - \beta = 0.15$)**: Chú khỉ cảm thấy chán nản, chú bấm nút bung dù bay để **nhảy ngẫu nhiên tới bất kỳ hòn đảo nào trên toàn thế giới** với xác suất đều $\frac{1}{n}$.
  - Nhờ chiếc dù bay này, chú khỉ sẽ dễ dàng thoát khỏi mọi bẫy nhện!

### Bẫy 2: Dead Ends (Nút cụt)
- **Hiện tượng:** Có những hòn đảo chỉ có cầu dẫn vào, nhưng **hoàn toàn không có cầu dẫn ra ngoài** ($d_j = 0$).
- **Hậu quả:** Khi chú khỉ đi vào đây, chú rơi xuống biển và biến mất! Lượng điểm số của toàn mạng lưới bị bốc hơi dần về 0 sau mỗi vòng lặp.
- **Cách giải cứu:**
  Nếu chú khỉ lọt vào nút cụt, hệ thống sẽ tự động kích hoạt dù bay đưa chú **chia đều xác suất cho toàn bộ $n$ hòn đảo** trên thế giới:
  $$\text{Bù nút cụt} = \beta \delta_t \frac{1}{n}$$
  (Trong đó $\delta_t$ là tổng lượng điểm bị rơi vào các nút cụt).

---

## 📐 5. Công thức PageRank hoàn chỉnh

Sau khi kết hợp cả 2 cơ chế giải cứu, tại mỗi vòng lặp $t+1$, điểm số của hòn đảo $i$ được tính bằng tổng của 3 dòng năng lượng:

$$r_i^{t+1} = \underbrace{\beta \sum_{j \to i} \frac{r_j^t}{d_j}}_{\text{(1) Nhận từ các cầu dẫn tới}} + \underbrace{\beta \delta_t \frac{1}{n}}_{\text{(2) Bù từ các nút cụt}} + \underbrace{(1-\beta)\frac{1}{n}}_{\text{(3) Nhảy dù ngẫu nhiên}}$$

- $\sum_{i=1}^n r_i^t = 1.0$ (Tổng điểm của toàn bộ thế giới luôn được bảo toàn bằng đúng 100%).

---

## 🧮 6. VÍ DỤ MINH HỌA TÍNH TOÁN BẰNG SỐ TỪNG BƯỚC (CỰC KỲ CHI TIẾT)

Để bạn hiểu cặn kẽ từng con số, ta hãy cùng nhau tự tay tính một mạng web mini gồm đúng **3 hòn đảo**: $A, B, C$.

### Sơ đồ liên kết của 3 đảo:
```
     ┌──────────┐
     │          ▼
  [ Đảo A ] ⇄ [ Đảo B ] ────> [ Đảo C (Nút cụt!) ]
```
- **Đảo $A$:** Có 1 cầu dẫn sang $B$ ($d_A = 1$).
- **Đảo $B$:** Có 2 cầu, một dẫn sang $A$, một dẫn sang $C$ ($d_B = 2$).
- **Đảo $C$:** Không có cầu nào dẫn ra ngoài! ($C$ là **nút cụt**, $d_C = 0$).

### Thiết lập tham số (chọn số tròn để dễ tính tay):
- Số đỉnh: $n = 3$.
- Hệ số giảm: $\beta = 0.8$ (suy ra $1 - \beta = 0.2$).

---

### 🟢 BƯỚC 0: KHỞI TẠO ĐIỂM BAN ĐẦU (Vòng $t = 0$)
Ban đầu chia đều điểm cho cả 3 đảo:
$$r_A^0 = \frac{1}{3} \approx 0.3333, \quad r_B^0 = \frac{1}{3} \approx 0.3333, \quad r_C^0 = \frac{1}{3} \approx 0.3333$$
$$\text{Tổng ban đầu} = 0.3333 + 0.3333 + 0.3333 = 1.0$$

---

### 🟢 BƯỚC 1: TÍNH ĐIỂM VÒNG 1 (Vòng $t = 1$)

#### 1. Tính lượng điểm rơi vào nút cụt ($\delta_0$):
Chỉ có đảo $C$ là nút cụt:
$$\delta_0 = r_C^0 = \frac{1}{3} \approx 0.3333$$

#### 2. Tính năng lượng nền cố định cho mỗi đảo:
- Năng lượng bù từ nút cụt chia đều cho 3 đảo:
  $$\text{Bù nút cụt} = \beta \delta_0 \frac{1}{n} = 0.8 \times \frac{1}{3} \times \frac{1}{3} = \frac{0.8}{9} \approx 0.0889$$
- Năng lượng nhảy dù ngẫu nhiên chia đều cho 3 đảo:
  $$\text{Nhảy dù} = (1 - \beta) \frac{1}{n} = 0.2 \times \frac{1}{3} = \frac{0.2}{3} \approx 0.0667$$
- **Tổng điểm nền mỗi đảo nhận được trước khi xét cầu:**
  $$\text{base} = 0.0889 + 0.0667 = 0.1556$$

#### 3. Lan truyền điểm qua các cây cầu:
- **Đảo $A$ nhận từ ai?**
  Chỉ có đảo $B$ trỏ sang $A$. Đảo $B$ có $d_B = 2$ cầu, nên $B$ chia một nửa điểm cho $A$:
  $$\text{Cầu tới } A = \beta \times \frac{r_B^0}{d_B} = 0.8 \times \frac{1/3}{2} = \frac{0.8}{6} \approx 0.1333$$
  $$\implies r_A^1 = \text{base} + 0.1333 = 0.1556 + 0.1333 = \mathbf{0.2889}$$

- **Đảo $B$ nhận từ ai?**
  Chỉ có đảo $A$ trỏ sang $B$. Đảo $A$ có $d_A = 1$ cầu, nên $A$ dành trọn điểm cho $B$:
  $$\text{Cầu tới } B = \beta \times \frac{r_A^0}{d_A} = 0.8 \times \frac{1/3}{1} = \frac{0.8}{3} \approx 0.2667$$
  $$\implies r_B^1 = \text{base} + 0.2667 = 0.1556 + 0.2667 = \mathbf{0.4223}$$

- **Đảo $C$ nhận từ ai?**
  Chỉ có đảo $B$ trỏ sang $C$. Đảo $B$ chia một nửa điểm cho $C$:
  $$\text{Cầu tới } C = \beta \times \frac{r_B^0}{d_B} = 0.8 \times \frac{1/3}{2} = \frac{0.8}{6} \approx 0.1333$$
  $$\implies r_C^1 = \text{base} + 0.1333 = 0.1556 + 0.1333 = \mathbf{0.2889}$$

#### 4. Kiểm tra tổng xác suất sau vòng 1:
$$\sum r_i^1 = 0.2889 + 0.4223 + 0.2889 = \mathbf{1.0001 \approx 1.0} \quad \text{(Hoàn toàn bảo toàn!)}$$

---

### 🟢 BƯỚC 2: TÍNH TIẾP VÒNG 2 (Vòng $t = 2$)

Điểm mới từ vòng 1 là: $r_A^1 = 0.2889$, $r_B^1 = 0.4223$, $r_C^1 = 0.2889$.
1. Lượng điểm nút cụt mới: $\delta_1 = r_C^1 = 0.2889$.
2. Điểm nền mới:
   $$\text{base} = 0.8 \times \frac{0.2889}{3} + \frac{0.2}{3} = 0.0770 + 0.0667 = 0.1437$$
3. Lan truyền:
   - $r_A^2 = 0.1437 + 0.8 \times \frac{0.4223}{2} = 0.1437 + 0.1689 = \mathbf{0.3126}$
   - $r_B^2 = 0.1437 + 0.8 \times \frac{0.2889}{1} = 0.1437 + 0.2311 = \mathbf{0.3748}$
   - $r_C^2 = 0.1437 + 0.8 \times \frac{0.4223}{2} = 0.1437 + 0.1689 = \mathbf{0.3126}$
4. Kiểm tra tổng:
   $$0.3126 + 0.3748 + 0.3126 = \mathbf{1.0000}$$

---

### 🏁 KẾT LUẬN TỪ VÍ DỤ:
Khi lặp thêm khoảng 20 vòng nữa, điểm số sẽ không đổi (hội tụ):
- **Đảo $B$ luôn đứng đầu (~38%)**: Vì nó là nút giao thông trung tâm nhận trọn cầu từ $A$.
- **Đảo $A$ và $C$ đứng sau (~31%)**: Vì chúng chỉ nhận được một nửa lượng cầu từ $B$.

Đó chính là cách PageRank xếp hạng toàn bộ 4,604 bài viết Wikipedia trong bài tập của bạn!
