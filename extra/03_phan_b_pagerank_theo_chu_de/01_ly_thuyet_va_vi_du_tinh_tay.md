# BÀI HỌC LÝ THUYẾT: PAGERANK THEO CHỦ ĐỀ VÀ TÍNH CHẤT TUYẾN TÍNH
> **Dành cho người mới bắt đầu — Từ số 0 đến hiểu bản chất**  
> **Thư mục:** `extra/phan_b_pagerank_theo_chu_de/`

---

## 🧭 1. Vấn đề thực tế: Tại sao PageRank toàn cục là chưa đủ?

Ở Bài học 01, ta đã biết thuật toán **PageRank toàn cục** tính ra một bảng xếp hạng duy nhất cho toàn bộ thế giới:
- Trang web nào có nhiều liên kết chất lượng nhất sẽ đứng đầu.

Nhưng hãy hình dung tình huống tìm kiếm sau:
Hai người cùng lên mạng gõ một từ khóa duy nhất: **"Apple"**:
- **Bác nông dân:** Muốn tìm hiểu về **quả táo, cách chăm bón vườn táo**.
- **Kỹ sư công nghệ:** Muốn tìm hiểu về **công ty Apple, điện thoại iPhone, máy Mac**.

Nếu chỉ dùng PageRank toàn cục:
Trang web của hãng công nghệ Apple có hàng triệu liên kết nên sẽ **luôn đứng trên** các trang nông nghiệp. Bác nông dân sẽ không thể tìm thấy thông tin mình cần!

$\implies$ Để giải quyết bài toán này, Đại học Stanford đã phát minh ra: **Topic-Sensitive PageRank (PageRank theo chủ đề / Cá nhân hóa)**.

---

## 🎯 2. Ý tưởng cốt lõi: Chú khỉ có sở thích cá nhân

Thay vì để chú khỉ nhảy dù ngẫu nhiên về bất kỳ đâu, ta huấn luyện cho chú một **sở thích chuyên biệt**:
- Giả sử chú khỉ là một **chuyên gia Khoa học (Science)**:
  - Khi chú đi qua các cây cầu bình thường ($\beta = 85\%$): Chú vẫn di chuyển theo các đường link thực tế của mạng web.
  - Khi chú nhảy dù ngẫu nhiên ($1 - \beta = 15\%$): **Chú CHỈ NHẢY DÙ vào các hòn đảo thuộc danh mục Khoa học**!

### Vector dịch chuyển theo chủ đề $v$:
Gọi $S$ là tập hợp tất cả các bài viết thuộc chủ đề đang xét (ví dụ: Khoa học).
$$v_i = \begin{cases} \frac{1}{|S|}, & \text{nếu bài viết } i \text{ thuộc chủ đề } S \\ 0, & \text{nếu bài viết } i \text{ nằm ngoài chủ đề } S \end{cases}$$

- Với các đảo thuộc Khoa học: Nhận xác suất nhảy dù đều nhau $\frac{1}{|S|}$.
- Với các đảo ngoài Khoa học: Xác suất nhảy dù trực tiếp bằng $0$.

---

## ⚖️ 3. Quy tắc bù nút cụt đặc biệt của bài toán

Đây là quy tắc bạn cần nắm vững để hiểu bản chất:

> [!NOTE]
> **Quy ước chuẩn của Bài học 04:**
> 1. **Dòng năng lượng nhảy dù $(1-\beta)$:** Phân phối theo sở thích chủ đề $\implies$ Dùng $(1-\beta) v_i$.
> 2. **Dòng năng lượng bù nút cụt ($\delta_t$):** Vẫn chia đều trên toàn bộ $n$ bài viết $\implies$ Dùng $\beta \delta_t \frac{1}{n}$.

Công thức cập nhật hoàn chỉnh:
$$r_i^{t+1} = \beta \sum_{j \to i} \frac{r_j^t}{d_j} + \beta \delta_t \frac{1}{n} + (1-\beta) v_i$$

**Câu hỏi:** *"Các bài viết không thuộc Khoa học có bị điểm 0 không?"*  
**Trả lời:** **Không**. Chúng không nhận được điểm nhảy dù trực tiếp ($v_i = 0$), nhưng nếu một bài viết khoa học có đặt link dẫn tới chúng, chúng vẫn nhận được điểm lan truyền qua cây cầu! Điều này phản ánh chính xác: *Bài viết này có liên quan đến khoa học dù bản thân nó không phải là bài khoa học thuần túy.*

---

## 🧮 4. VÍ DỤ MINH HỌA BẰNG SỐ TỪNG BƯỚC

Hãy lấy lại **đúng mạng web 3 hòn đảo** ở Bài học 01 để bạn thấy rõ sự khác biệt giữa PageRank toàn cục và PageRank theo chủ đề:

```
     ┌──────────┐
     │          ▼
  [ Đảo A ] ⇄ [ Đảo B ] ────> [ Đảo C (Nút cụt!) ]
```
- Đảo $A$: $d_A = 1$ (cầu sang $B$).
- Đảo $B$: $d_B = 2$ (cầu sang $A$ và $C$).
- Đảo $C$: $d_C = 0$ (nút cụt).
- Tham số: $n = 3$, $\beta = 0.8$.

### Giả thiết chủ đề:
Giả sử chỉ có **duy nhất Đảo $A$ thuộc danh mục Khoa học** ($S = \{A\}$).
- Do đó: $|S| = 1$.
- Vector phân phối chủ đề:
  $$v_A = 1.0, \quad v_B = 0.0, \quad v_C = 0.0$$

---

### 🟢 BƯỚC 0: KHỞI TẠO ĐIỂM (Vòng $t = 0$)
Khởi tạo điểm bằng chính vector chủ đề:
$$r_A^0 = 1.0, \quad r_B^0 = 0.0, \quad r_C^0 = 0.0$$
$$\text{Tổng ban đầu} = 1.0 + 0.0 + 0.0 = 1.0$$

---

### 🟢 BƯỚC 1: TÍNH ĐIỂM VÒNG 1 (Vòng $t = 1$)

#### 1. Điểm thất thoát ở nút cụt:
Chỉ có đảo $C$ là nút cụt, và ở vòng 0 điểm của $C$ là $0.0$:
$$\delta_0 = r_C^0 = 0.0$$
$$\implies \text{Bù nút cụt} = \beta \delta_0 \frac{1}{n} = 0.0$$

#### 2. Điểm nhảy dù theo chủ đề $(1 - \beta) v$:
Với $1 - \beta = 0.2$:
- Đảo $A$: $0.2 \times v_A = 0.2 \times 1.0 = \mathbf{0.2}$
- Đảo $B$: $0.2 \times v_B = 0.2 \times 0.0 = \mathbf{0.0}$
- Đảo $C$: $0.2 \times v_C = 0.2 \times 0.0 = \mathbf{0.0}$

#### 3. Lan truyền điểm qua các cây cầu:
- **Đảo $A$ nhận từ $B$:**
  $$r_A^1 = \text{Nhảy dù } A + \beta \times \frac{r_B^0}{d_B} = 0.2 + 0.8 \times \frac{0.0}{2} = \mathbf{0.2}$$
- **Đảo $B$ nhận từ $A$:**
  Đảo $A$ có $r_A^0 = 1.0$ và $d_A = 1$, dồn toàn bộ điểm sang $B$:
  $$r_B^1 = \text{Nhảy dù } B + \beta \times \frac{r_A^0}{d_A} = 0.0 + 0.8 \times \frac{1.0}{1} = \mathbf{0.8}$$
- **Đảo $C$ nhận từ $B$:**
  $$r_C^1 = \text{Nhảy dù } C + \beta \times \frac{r_B^0}{d_B} = 0.0 + 0.8 \times \frac{0.0}{2} = \mathbf{0.0}$$

#### 4. Tổng xác suất sau vòng 1:
$$\sum r_i^1 = 0.2 + 0.8 + 0.0 = \mathbf{1.0} \quad \text{(Chính xác tuyệt đối!)}$$

---

### 🟢 BƯỚC 2: TÍNH ĐIỂM VÒNG 2 (Vòng $t = 2$)

Từ vòng 1: $r_A^1 = 0.2$, $r_B^1 = 0.8$, $r_C^1 = 0.0$.
1. Lượng nút cụt: $\delta_1 = r_C^1 = 0.0 \implies \text{Bù nút cụt} = 0.0$.
2. Lan truyền:
   - **Đảo $A$:** Nhận nhảy dù $0.2$ + nhận từ $B$:
     $$r_A^2 = 0.2 + 0.8 \times \frac{0.8}{2} = 0.2 + 0.32 = \mathbf{0.52}$$
   - **Đảo $B$:** Nhận nhảy dù $0.0$ + nhận từ $A$:
     $$r_B^2 = 0.0 + 0.8 \times \frac{0.2}{1} = \mathbf{0.16}$$
   - **Đảo $C$:** Nhận nhảy dù $0.0$ + nhận từ $B$:
     $$r_C^2 = 0.0 + 0.8 \times \frac{0.8}{2} = \mathbf{0.32}$$
3. Tổng xác suất:
   $$0.52 + 0.16 + 0.32 = \mathbf{1.00}$$

---

### 📊 SO SÁNH KẾT QUẢ VỚI PAGERANK TOÀN CỤC:
- Ở **PageRank toàn cục** (Bài 01): Đảo $B$ luôn đứng đầu (~38%), đảo $A$ chỉ được ~31%.
- Ở **PageRank theo chủ đề Khoa học** (Bài này): **Đảo $A$ lập tức vọt lên đứng đầu (52%)** vì chú khỉ ưu tiên nhảy dù vào đảo $A$!

---

## 🪄 5. Phép màu của Tính chất Tuyến tính (Linearity)

Bây giờ đến phần thú vị nhất của đề bài: **Ghép các chủ đề**.

Giả sử một người dùng có sở thích hỗn hợp:
- $50\%$ Khoa học (Topic 1)
- $30\%$ Lịch sử (Topic 2)
- $20\%$ Địa lý (Topic 3)

### Liệu ta có phải chạy lại thuật toán lặp 100 vòng nữa không?
**KHÔNG CẦN!**  
Vì phương trình PageRank là phương trình tuyến tính, nên nghiệm dừng của tổ hợp lồi các vector chủ đề:
$$v_{\text{mix}} = 0.5 \cdot v_{\text{Science}} + 0.3 \cdot v_{\text{History}} + 0.2 \cdot v_{\text{Geography}}$$
chính bằng **tổ hợp trực tiếp của các nghiệm đã tính trước đó**:
$$r_{\text{mix}} = 0.5 \cdot r_{\text{Science}} + 0.3 \cdot r_{\text{History}} + 0.2 \cdot r_{\text{Geography}}$$

### Ví dụ số học kiểm chứng:
Giả sử điểm của bài viết `Albert_Einstein` trong 3 bảng xếp hạng là:
- $r_{\text{Science}} = 0.0080$ (trong Khoa học điểm rất cao)
- $r_{\text{History}} = 0.0030$ (trong Lịch sử điểm trung bình)
- $r_{\text{Geography}} = 0.0005$ (trong Địa lý điểm thấp)

Khi người dùng tìm kiếm với tỷ lệ $(0.5, 0.3, 0.2)$, điểm số hỗn hợp của Einstein là:
$$r_{\text{mix}} = (0.5 \times 0.0080) + (0.3 \times 0.0030) + (0.2 \times 0.0005)$$
$$r_{\text{mix}} = 0.0040 + 0.0009 + 0.0001 = \mathbf{0.0050}$$

Phép nhân cộng này diễn ra trong **0.001 giây**, giúp công cụ tìm kiếm phục vụ hàng triệu người dùng khác nhau với các sở thích cá nhân khác nhau mà máy chủ không hề bị quá tải!
