# GIẢI THÍCH CHI TIẾT HÀM `jaccard` VÀ `signature_similarity`
> **Phần C — Tính độ tương đồng Jaccard chính xác và Ước lượng qua Chữ ký MinHash**  
> **Tệp:** `submission.py` (Dòng 233–259)

---

## 1. YÊU CẦU ĐỀ BÀI VÀ CƠ SỞ TOÁN HỌC

Trong mục **4.3. Ước lượng tương đồng** của [bai-tap-wikispeedia.md](file:///C:/Users/HP/Downloads/HKI_2026-2027/GI%E1%BA%A2I%20THU%E1%BA%ACT%20KHDL/wikispeedia-student-package/bai-tap-wikispeedia.md):

1. **Độ tương đồng Jaccard chính xác (`jaccard`):**
   - Đo lường mức độ trùng lặp nội dung giữa hai văn bản $A$ và $B$ thông qua hai tập chuỗi shingle gốc $S_A$ và $S_B$:
     $$J(A, B) = \frac{|S_A \cap S_B|}{|S_A \cup S_B|}$$
   - Giá trị luôn nằm trong đoạn $[0.0, 1.0]$: Bằng $0.0$ nếu không có từ nào chung, bằng $1.0$ nếu hai bài viết giống hệt nhau về tập shingle.
2. **Độ tương đồng ước lượng qua chữ ký MinHash (`signature_similarity`):**
   - Theo **Định lý cơ bản của MinHash** (Broder, 1997): Xác suất để hai văn bản có cùng giá trị băm nhỏ nhất bằng chính xác hệ số Jaccard của chúng:
     $$P(h(A) = h(B)) = J(A, B)$$
   - Vì vậy, với vector chữ ký 128 chiều $\sigma(A)$ và $\sigma(B)$, ta ước lượng $J(A, B)$ bằng tỷ lệ số tọa độ trùng nhau:
     $$\hat{J}(A, B) = \frac{1}{128} \sum_{j=0}^{127} \mathbb{I}(\sigma_j(A) == \sigma_j(B))$$
   - **Cảnh báo sống còn từ đề bài:** Tuyệt đối không được biến đổi chữ ký thành kiểu `set` rồi gọi Jaccard! Phải so sánh theo **từng cặp vị trí tương ứng** $j = 0, \dots, 127$.
3. **Quy tắc xử lý trường hợp biên (Tập rỗng):**
   - Nếu cả hai bài viết đều rỗng ($S_A = \emptyset$ và $S_B = \emptyset$): Độ tương đồng được quy ước là **`1.0`** (vì hai bài rỗng được coi là giống hệt nhau).
   - Nếu chỉ có đúng một bài viết rỗng ($S_A = \emptyset$ hoặc $S_B = \emptyset$): Độ tương đồng bằng **`0.0`**.

---

## 2. BẢNG THÔNG SỐ ĐẦU VÀO VÀ ĐẦU RA

### 2.1. Hàm `jaccard`
```python
def jaccard(set_a: set[str], set_b: set[str]) -> float:
```

| Tham số / Đầu ra | Kiểu dữ liệu | Ý nghĩa thực tế | Giá trị ví dụ |
|---|---|---|---|
| **`set_a`** | `set[str]` | Tập hợp các chuỗi shingle 9 ký tự của bài viết thứ nhất. | `{"Édouard m", "douard ma", ...}` |
| **`set_b`** | `set[str]` | Tập hợp các chuỗi shingle 9 ký tự của bài viết thứ hai. | `{"Claude Mo", "laude Mon", ...}` |
| **ĐẦU RA (`->`)** | `float` | Hệ số tương đồng Jaccard chính xác trong đoạn $[0.0, 1.0]$. | `0.14285714285714285` |

### 2.2. Hàm `signature_similarity`
```python
def signature_similarity(
    sig_a: list[int], sig_b: list[int], empty_a: bool, empty_b: bool
) -> float:
```

| Tham số / Đầu ra | Kiểu dữ liệu | Ý nghĩa thực tế | Giá trị ví dụ |
|---|---|---|---|
| **`sig_a`** | `list[int]` | Chữ ký MinHash gồm 128 số nguyên của bài viết $A$. | `[381920, 104829, ..., 829104]` |
| **`sig_b`** | `list[int]` | Chữ ký MinHash gồm 128 số nguyên của bài viết $B$. | `[381920, 994821, ..., 829104]` |
| **`empty_a`** | `bool` | Cờ đánh dấu bài viết $A$ có tập shingle rỗng hay không. | `False` |
| **`empty_b`** | `bool` | Cờ đánh dấu bài viết $B$ có tập shingle rỗng hay không. | `False` |
| **ĐẦU RA (`->`)** | `float` | Hệ số tương đồng Jaccard ước lượng $\hat{J}(A, B) \in [0.0, 1.0]$. | `0.1484375` (tức $19/128$) |

---

## 3. TOÀN BỘ MÃ NGUỒN

```python
def jaccard(set_a: set[str], set_b: set[str]) -> float:
    union_size = len(set_a) + len(set_b) - len(set_a & set_b)
    return len(set_a & set_b) / union_size if union_size else 1.0


def signature_similarity(
    sig_a: list[int], sig_b: list[int], empty_a: bool, empty_b: bool
) -> float:
    if not sig_a or len(sig_a) != len(sig_b):
        raise ValueError("Chữ ký phải có cùng độ dài dương")
    if empty_a or empty_b:
        return 1.0 if empty_a and empty_b else 0.0
    return sum(a == b for a, b in zip(sig_a, sig_b)) / len(sig_a)
```

---

## 4. TỪ ĐIỂN BIẾN NỘI BỘ (LOCAL VARIABLES)

| Tên biến | Kiểu dữ liệu | Ý nghĩa & Vai trò trong thuật toán |
|---|---|---|
| **`set_a & set_b`** | `set[str]` | Tập hợp giao chứa các chuỗi shingle cùng xuất hiện ở cả hai bài viết. |
| **`union_size`** | `int` | Kích thước của tập hợp hợp $|S_A \cup S_B|$, tính theo công thức bù trừ $|A| + |B| - |A \cap B|$. |
| **`sig_a`**, **`sig_b`** | `list[int]` | Hai vector chữ ký MinHash 128 phần tử cần so sánh. |
| **`empty_a`**, **`empty_b`** | `bool` | Cờ báo hiệu trạng thái rỗng nguyên bản của hai tập shingle. |
| **`a`**, **`b`** | `int` | Giá trị băm nhỏ nhất tại vị trí thứ $j$ của bài $A$ và bài $B$ ($j = 0 \dots 127$). |
| **`a == b`** | `bool` | Phép so sánh bằng tại vị trí $j$ (trả về `True` nếu trùng, `False` nếu lệch). |

---

## 5. GIẢI THÍCH CHI TIẾT TỪNG DÒNG CODE

### 5.1. Hàm `jaccard`

```python
    union_size = len(set_a) + len(set_b) - len(set_a & set_b)
```
- **Kỹ thuật tối ưu bộ nhớ RAM:**
  - Thông thường người mới học sẽ viết: `len(set_a | set_b)`.
  - Nhưng toán tử `|` (hợp hai tập) sẽ buộc hệ điều hành phải cấp phát một vùng nhớ khổng lồ để gom toàn bộ hàng chục ngàn phần tử của cả hai tập hợp lại.
  - Áp dụng nguyên lý bù trừ tập hợp:
    $$|A \cup B| = |A| + |B| - |A \cap B|$$
  - `len(set_a)` và `len(set_b)` lấy tức thì trong $O(1)$.
  - Phép giao `set_a & set_b` chỉ duyệt qua tập nhỏ hơn và có kích thước bé hơn nhiều so với tập hợp hợp. Nhờ vậy chương trình chạy nhanh hơn và tiết kiệm RAM đáng kể.

```python
    return len(set_a & set_b) / union_size if union_size else 1.0
```
- **Xử lý toàn diện mọi trường hợp:**
  - Nếu `union_size > 0` (ít nhất một bài có dữ liệu): Lấy số phần tử chung chia cho tổng số phần tử hợp: $\frac{|A \cap B|}{|A \cup B|}$.
    - Nếu 1 bài có dữ liệu và 1 bài rỗng: Phép giao $= 0$, `union_size > 0` $\implies 0 / \text{union\_size} = 0.0$.
  - Nếu `union_size == 0` (nghĩa là cả 2 tập $A$ và $B$ đều rỗng): Biểu thức `if union_size` trả về `False`, nhánh `else` kích hoạt và trả về ngay **`1.0`** theo đúng quy định.

---

### 5.2. Hàm `signature_similarity`

```python
    if not sig_a or len(sig_a) != len(sig_b):
        raise ValueError("Chữ ký phải có cùng độ dài dương")
```
- Đảm bảo 2 chữ ký không rỗng và phải có cùng số chiều (đều dài đúng 128 số nguyên). Nếu khác độ dài, không thể so sánh từng vị trí được $\implies$ Ném ngoại lệ `ValueError`.

```python
    if empty_a or empty_b:
        return 1.0 if empty_a and empty_b else 0.0
```
- Kiểm tra trạng thái rỗng thông qua 2 cờ `empty_a` và `empty_b`:
  - `if empty_a and empty_b`: Cả 2 bài đều rỗng $\implies$ Trả về **`1.0`**.
  - Nếu chỉ 1 trong 2 bài rỗng $\implies$ Trả về **`0.0`**.

```python
    return sum(a == b for a, b in zip(sig_a, sig_b)) / len(sig_a)
```
- **`zip(sig_a, sig_b)`:** Ghép đôi song song từng cặp tọa độ:
  $$(\sigma_0(A), \sigma_0(B)), (\sigma_1(A), \sigma_1(B)), \dots, (\sigma_{127}(A), \sigma_{127}(B))$$
- **`a == b`:** So sánh từng cặp số. Trong Python, giá trị `True` được coi là `1`, `False` là `0`.
- **`sum(...)`:** Đếm tổng số vị trí mà hai chữ ký trùng nhau hoàn toàn (ví dụ trùng $19$ trên $128$ vị trí).
- **`/ len(sig_a)`:** Chia cho tổng số chiều ($128$) để ra tỷ lệ trùng lặp:
  $$\hat{J} = \frac{19}{128} \approx 0.1484375$$

---

## 6. TẠI SAO LẠI LÀM NHƯ VẬY? (THIẾT KẾ KỸ THUẬT & LÝ DO CỐT LÕI)

### 1. Tại sao tính kích thước hợp bằng bù trừ thay vì `set_a | set_b`?
- Các bài viết Wikipedia dài có từ $20,000$ đến $40,000$ shingle.
- Mỗi chuỗi shingle trong Python tốn khoảng 50–60 byte bộ nhớ.
- Khi so sánh 100 cặp bài viết, nếu tạo tập hợp hợp `set_a | set_b`, Python sẽ liên tục cấp phát và giải phóng các khối bộ nhớ hàng chục Megabyte trên RAM, gây áp lực lớn cho bộ thu gom rác (Garbage Collector).
- Công thức $|A| + |B| - |A \cap B|$ triệt tiêu hoàn toàn chi phí tạo tập hợp hợp, chỉ tính toán giao điểm tối thiểu.

### 2. Tại sao so sánh từng tọa độ mà CẤM chuyển chữ ký thành `set`?
- Hãy xem ví dụ này:
  - Chữ ký $A$: $[10, 20, 30, 40]$
  - Chữ ký $B$: $[40, 30, 20, 10]$
- Nếu so sánh theo từng tọa độ:
  - Vị trí 0: $10 \ne 40$
  - Vị trí 1: $20 \ne 30$
  - Vị trí 2: $30 \ne 20$
  - Vị trí 3: $40 \ne 10$
  $\implies$ Số vị trí trùng $= 0 \implies \hat{J} = 0.0$.
- Nhưng nếu bạn biến thành `set`:
  - `set(A) = {10, 20, 30, 40}`
  - `set(B) = {10, 20, 30, 40}`
  - Tính Jaccard trên set: $\frac{|\{10,20,30,40\}|}{|\{10,20,30,40\}|} = 1.0$ (SAI HOÀN TOÀN!).
- Chữ ký MinHash là **vector có thứ tự**, trong đó mỗi vị trí $j$ gắn liền với một hàm băm $h_j$ duy nhất. Chuyển sang `set` làm mất hoàn toàn thứ tự hàm băm và dẫn đến kết quả sai lệch nghiêm trọng.

### 3. Tại sao phải truyền thêm 2 cờ `empty_a` và `empty_b`?
- Khi một bài viết rỗng, hàm `minhash` trả về danh sách gồm 128 số $p$ (giá trị sentinel).
- Nếu ta so sánh 2 bài viết rỗng: Cả hai đều là $[p, p, \dots, p]$. Khi so khớp vị trí sẽ trùng $128/128 = 1.0$.
- Tuy nhiên, nếu có một bài viết không rỗng nhưng do trùng hợp hy hữu một hàm băm nào đó sinh ra giá trị $p$ (hoặc trường hợp đồ thị kiểm thử nhỏ), việc dựa vào giá trị số có thể gây mơ hồ.
- Việc truyền cờ rõ ràng `empty_a, empty_b` giúp hàm xử lý quy ước biên một cách trực diện, minh bạch và an toàn tuyệt đối.

---

## 7. VÍ DỤ SỐ HỌC MINH HỌA TRỰC QUAN

Giả sử ta so sánh 2 bài viết về hội họa: $A$ (Họa sĩ Édouard Manet) và $B$ (Họa sĩ Claude Monet).
Để dễ hình dung, giả sử chữ ký chỉ gồm 8 hàm băm ($j = 0 \dots 7$):

| Hàm băm ($j$) | $\sigma_j(A)$ | $\sigma_j(B)$ | So sánh ($a == b$) | Điểm |
|:---:|:---:|:---:|:---:|:---:|
| $0$ | **`102`** | **`102`** | $102 == 102$ | **1** |
| $1$ | `55` | `99` | $55 \ne 99$ | 0 |
| $2$ | **`890`** | **`890`** | $890 == 890$ | **1** |
| $3$ | **`12`** | **`12`** | $12 == 12$ | **1** |
| $4$ | `44` | `11` | $44 \ne 11$ | 0 |
| $5$ | **`789`** | **`789`** | $789 == 789$ | **1** |
| $6$ | `31` | `50` | $31 \ne 50$ | 0 |
| $7$ | **`99`** | **`99`** | $99 == 99$ | **1** |

### Tính toán kết quả:
- Tổng số vị trí trùng nhau:
  $$\text{matches} = 1 + 0 + 1 + 1 + 0 + 1 + 0 + 1 = 5$$
- Độ tương đồng ước lượng $\hat{J}(A, B)$:
  $$\hat{J}(A, B) = \frac{5}{8} = 0.625 \quad (62.5\%)$$
- Kết luận: Hai bài viết có khoảng $62.5\%$ nội dung tương đồng về mặt từ vựng và cấu trúc câu!
