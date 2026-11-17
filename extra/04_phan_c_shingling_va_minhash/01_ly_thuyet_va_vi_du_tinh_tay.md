# BÀI HỌC LÝ THUYẾT: SHINGLING, MINHASH VÀ ĐỘ ĐO JACCARD
> **Dành cho người mới bắt đầu — Từ số 0 đến hiểu bản chất**  
> **Thư mục:** `extra/phan_c_shingling_va_minhash/`

---

## 📄 1. Bài toán thực tế: So sánh tương đồng văn bản dữ liệu lớn

Hãy tưởng tượng bạn là một kỹ sư tại Google hoặc một giảng viên cần chấm thi:
- Có hàng triệu bài viết hoặc bài luận của sinh viên.
- Bạn cần tìm xem **hai bài viết nào giống nhau (hoặc nghi ngờ sao chép / đạo văn của nhau)**.

### Tại sao các cách thông thường đều thất bại?
1. **Phép so sánh bằng trực tiếp (`bài_1 == bài_2`):**
   - Người sao chép chỉ cần sửa 1 dấu chấm, thêm 1 dấu cách, hoặc đổi 1 chữ hoa thành chữ thường $\implies$ Phép so sánh báo ngay là `Khác nhau 100%`!
2. **Mã băm mật mã (MD5, SHA-256):**
   - Thuật toán mật mã được thiết kế với hiệu ứng "thác đổ" (*Avalanche effect*): Chỉ cần đổi đúng 1 dấu cách, mã băm đầu ra sẽ khác biệt hoàn toàn $\implies$ Không dùng để đo độ giống nhau được.
3. **So sánh từng từ từng câu:**
   - Với 4,604 bài viết, số lượng cặp cần so sánh là:
     $$\frac{4604 \times 4603}{2} \approx 10.6 \text{ TRIỆU CẶP BÀI}$$
   - Nếu mỗi cặp bài phải quét qua từng câu, hệ thống máy chủ sẽ bị treo hoàn toàn!

$\implies$ Để giải quyết bài toán này, các nhà khoa học dữ liệu sử dụng bộ 3 công cụ: **K-Shingling**, **Hệ số Jaccard** và **Chữ ký MinHash**.

---

## 🪟 2. K-Shingling là gì? (Kỹ thuật Cửa sổ trượt)

Thay vì nhìn bài viết dưới dạng một đoạn văn dài, ta dùng một **khung kính nhỏ gồm $k$ ký tự liên tiếp** và trượt từ đầu đến cuối văn bản. Mỗi chuỗi con $k$ ký tự nhìn thấy được gọi là một **Shingle**.

Đề bài bài tập quy định **$k = 9$ ký tự Unicode liên tiếp**.

### Ví dụ trực quan với $k = 4$ trên chuỗi `"mèo con ngủ"`:
Đặt khung kính 4 ký tự và trượt từng bước một ký tự:
1. `[mèo ]`
2. `[èo c]`
3. `[o co]`
4. `[ con]`
5. `[con ]`
6. `[on n]`
7. `[n ng]`
8. `[ ngủ]`

Mỗi bài viết được chuyển hóa thành một **tập hợp (`set`) các shingle phân biệt**.
- Nếu hai bài viết có nhiều câu từ giống nhau, chúng sẽ chia sẻ rất nhiều shingle chung, bất kể thứ tự các câu có bị đảo lộn đôi chút!

---

## 📐 3. Hệ số tương đồng Jaccard (Jaccard Similarity)

Giả sử Bài 1 có tập shingle là $A$, Bài 2 có tập shingle là $B$.  
**Hệ số Jaccard** đo tỷ lệ diện tích trùng nhau của hai tập hợp:

$$J(A, B) = \frac{|A \cap B|}{|A \cup B|} = \frac{\text{Số lượng shingle CHUNG}}{\text{Tổng số shingle PHÂN BIỆT của cả 2 bài}}$$

```
[         Tập A         ]   [         Tập B         ]
           \                     /
            \   PHẦN CHUNG      /
             [  (Giao nhau)   ]
-----------------------------------------------------------
     Jaccard = |GIAO NHAU| / |HỢP TOÀN BỘ HAI TẬP|
```

- $J = 1.0$ ($100\%$): Hai bài có tập shingle giống hệt nhau.
- $J = 0.0$ ($0\%$): Hai bài hoàn toàn không có chuỗi 9 ký tự nào chung.
- $J = 0.6$ ($60\%$): Hai bài trùng lặp nội dung rất cao.

---

## 🚧 4. Nút thắt Big Data: Tại sao tập Shingle vẫn quá nặng?

Một bài viết Wikipedia trung bình có khoảng **$20,000$ shingle**.
- Mỗi shingle 9 ký tự tốn khoảng 9–18 byte bộ nhớ.
- Một bài viết tốn khoảng: $20,000 \times 18 \text{ bytes} \approx 360 \text{ KB}$.
- Toàn bộ 4,604 bài viết sẽ ngốn gần **2 GB RAM** chỉ để lưu các tập shingle!
- Mỗi lần so sánh hai tập hợp gồm 20,000 phần tử, máy tính phải duyệt qua hàng chục nghìn phép so sánh chuỗi $\implies$ So sánh hàng triệu cặp sẽ cực kỳ chậm chạp.

**Câu hỏi lớn của Andrei Broder (nhà khoa học của Altavista/Compaq):**  
*Liệu có cách nào nén một tập hợp 20,000 phần tử thành một vector siêu nhỏ chỉ gồm **128 con số nguyên**, mà khi so sánh 128 con số đó, ta vẫn biết chính xác độ tương đồng Jaccard không?*

**Câu trả lời là CÓ: Đó chính là MinHash!**

---

## ⚡ 5. Thuật toán MinHash và Định lý kỳ diệu

### 5.1. Băm Shingle thành số nguyên
Đầu tiên, mỗi shingle chuỗi được chuyển thành một con số nguyên bằng hàm SHA-256:
$$x = \text{SHA256}(s) \bmod p \quad (\text{với } p = 2^{31} - 1 = 2147483647)$$

### 5.2. Hàm băm Affine và Chữ ký MinHash
Ta sử dụng **128 hàm băm ngẫu nhiên** có dạng:
$$h_j(x) = (a_j \cdot x + b_j) \bmod p, \quad j = 0, 1, \dots, 127$$

Với mỗi bài viết $D$, ta áp dụng hàm băm $h_j$ lên tất cả các shingle của bài viết, rồi **chỉ nhặt lấy con số NHỎ NHẤT (giá trị min)**:
$$\sigma_j(D) = \min_{x \in S(D)} h_j(x)$$

Làm như vậy với 128 hàm băm, ta thu được **Chữ ký MinHash (MinHash Signature)** gồm đúng 128 số nguyên:
$$\text{Signature}(D) = [\sigma_0(D), \sigma_1(D), \dots, \sigma_{127}(D)]$$

### 5.3. Định lý MinHash kinh điển
> **Xác suất để giá trị min của hai tập hợp A và B trùng nhau đúng bằng Hệ số tương đồng Jaccard thực tế giữa chúng!**
> $$P(\sigma_j(A) == \sigma_j(B)) = J(A, B)$$

#### Chứng minh bằng trực giác cực kỳ dễ hiểu:
Hãy xếp tất cả các shingle có trong $A \cup B$ ra một chiếc bàn. Khi áp dụng một hàm băm ngẫu nhiên, các shingle sẽ được xáo trộn thứ tự một cách ngẫu nhiên.
Shingle nào có giá trị băm nhỏ nhất sẽ về vị trí số 1.
- Shingle về vị trí số 1 này bắt buộc phải thuộc về $A$, thuộc về $B$, hoặc thuộc về cả hai.
- Để $\min h(A) == \min h(B)$, shingle về nhất **bắt buộc phải nằm trong phần giao nhau $A \cap B$**!
- Vì hàm băm là ngẫu nhiên đều, xác suất để shingle về nhất rơi trúng vào phần giao $A \cap B$ chính bằng:
  $$\frac{\text{Số lượng phần tử trong } A \cap B}{\text{Tổng số lượng phần tử trong } A \cup B} = J(A, B)!$$

---

## 🧮 6. VÍ DỤ MINH HỌA TÍNH MINHASH BẰNG SỐ TỪNG BƯỚC

Hãy cùng tính thử một ví dụ thu nhỏ để bạn thấy từng con số vận hành:

### Giả thiết:
Toàn bộ kho từ vựng chỉ có 5 shingle: $\{s_1, s_2, s_3, s_4, s_5\}$.
Mã số nguyên của 5 shingle này lần lượt là:
$$x(s_1) = 1, \quad x(s_2) = 2, \quad x(s_3) = 3, \quad x(s_4) = 4, \quad x(s_5) = 5$$

Ta có hai bài viết:
- **Bài A** chứa 3 shingle: $\{s_1, s_2, s_4\} \implies \text{Mã số } = \{1, 2, 4\}$
- **Bài B** chứa 3 shingle: $\{s_2, s_3, s_4\} \implies \text{Mã số } = \{2, 3, 4\}$

---

### 🟢 BƯỚC 1: TÍNH JACCARD CHÍNH XÁC
- Phần giao (chung): $A \cap B = \{s_2, s_4\} \implies |A \cap B| = 2$ phần tử.
- Phần hợp (tất cả): $A \cup B = \{s_1, s_2, s_3, s_4\} \implies |A \cup B| = 4$ phần tử.
$$\implies J(A, B) = \frac{|A \cap B|}{|A \cup B|} = \frac{2}{4} = \mathbf{0.5} \quad (50\%)$$

---

### 🟢 BƯỚC 2: TÍNH CHỮ KÝ MINHASH QUA 4 HÀM BĂM
Giả sử ta dùng số nguyên tố nhỏ $p = 7$, và 4 hàm băm:
1. $h_0(x) = (1x + 1) \bmod 7$
2. $h_1(x) = (3x + 2) \bmod 7$
3. $h_2(x) = (5x + 4) \bmod 7$
4. $h_3(x) = (2x + 6) \bmod 7$

#### Bảng tính giá trị băm cho từng shingle:
| Shingle $x$ | $h_0(x)$ | $h_1(x)$ | $h_2(x)$ | $h_3(x)$ |
|:---:|:---:|:---:|:---:|:---:|
| $x = 1$ | $(1\cdot 1+1)\%7 = \mathbf{2}$ | $(3\cdot 1+2)\%7 = \mathbf{5}$ | $(5\cdot 1+4)\%7 = \mathbf{2}$ | $(2\cdot 1+6)\%7 = \mathbf{1}$ |
| $x = 2$ | $(1\cdot 2+1)\%7 = \mathbf{3}$ | $(3\cdot 2+2)\%7 = \mathbf{1}$ | $(5\cdot 2+4)\%7 = \mathbf{0}$ | $(2\cdot 2+6)\%7 = \mathbf{3}$ |
| $x = 3$ | $(1\cdot 3+1)\%7 = \mathbf{4}$ | $(3\cdot 3+2)\%7 = \mathbf{4}$ | $(5\cdot 3+4)\%7 = \mathbf{5}$ | $(2\cdot 3+6)\%7 = \mathbf{5}$ |
| $x = 4$ | $(1\cdot 4+1)\%7 = \mathbf{5}$ | $(3\cdot 4+2)\%7 = \mathbf{0}$ | $(5\cdot 4+4)\%7 = \mathbf{3}$ | $(2\cdot 4+6)\%7 = \mathbf{0}$ |

---

#### Tìm giá trị MIN cho Bài A (gồm $x = \{1, 2, 4\}$):
- Với $h_0$: các giá trị là $\{h_0(1), h_0(2), h_0(4)\} = \{2, 3, 5\} \implies \min = \mathbf{2}$.
- Với $h_1$: các giá trị là $\{h_1(1), h_1(2), h_1(4)\} = \{5, 1, 0\} \implies \min = \mathbf{0}$.
- Với $h_2$: các giá trị là $\{h_2(1), h_2(2), h_2(4)\} = \{2, 0, 3\} \implies \min = \mathbf{0}$.
- Với $h_3$: các giá trị là $\{h_3(1), h_3(2), h_3(4)\} = \{1, 3, 0\} \implies \min = \mathbf{0}$.

$$\implies \text{Chữ ký của Bài A: } \sigma(A) = [\mathbf{2}, \mathbf{0}, \mathbf{0}, \mathbf{0}]$$

---

#### Tìm giá trị MIN cho Bài B (gồm $x = \{2, 3, 4\}$):
- Với $h_0$: các giá trị là $\{h_0(2), h_0(3), h_0(4)\} = \{3, 4, 5\} \implies \min = \mathbf{3}$.
- Với $h_1$: các giá trị là $\{h_1(2), h_1(3), h_1(4)\} = \{1, 4, 0\} \implies \min = \mathbf{0}$.
- Với $h_2$: các giá trị là $\{h_2(2), h_2(3), h_2(4)\} = \{0, 5, 3\} \implies \min = \mathbf{0}$.
- Với $h_3$: các giá trị là $\{h_3(2), h_3(3), h_3(4)\} = \{3, 5, 0\} \implies \min = \mathbf{0}$.

$$\implies \text{Chữ ký của Bài B: } \sigma(B) = [\mathbf{3}, \mathbf{0}, \mathbf{0}, \mathbf{0}]$$

---

### 🟢 BƯỚC 3: ĐO ĐỘ TƯƠNG ĐỒNG ƯỚC LƯỢNG QUA CHỮ KÝ
Đặt hai chữ ký cạnh nhau và so sánh từng vị trí:
$$\sigma(A) = [\mathbf{2}, \mathbf{0}, \mathbf{0}, \mathbf{0}]$$
$$\sigma(B) = [\mathbf{3}, \mathbf{0}, \mathbf{0}, \mathbf{0}]$$

1. Vị trí 0: $2 \ne 3 \implies$ Không trùng
2. Vị trí 1: $0 == 0 \implies$ **TRÙNG (1)**
3. Vị trí 2: $0 == 0 \implies$ **TRÙNG (1)**
4. Vị trí 3: $0 == 0 \implies$ **TRÙNG (1)**

$$\text{Số vị trí trùng} = 3 \text{ trên tổng số } 4 \text{ tọa độ}$$
$$\implies \text{Jaccard ước lượng } \hat{J}(A, B) = \frac{3}{4} = \mathbf{0.75}$$

*(Ở đây với 4 hàm băm, ước lượng là 0.75 so với giá trị thật 0.5. Khi ta tăng số hàm băm lên **128 hàm băm** như trong bài tập thật, ước lượng sẽ hội tụ cực kỳ chính xác sát với giá trị Jaccard thực tế theo luật số lớn!)*

---

### 🏁 KẾT LUẬN TỔNG QUAN:
- Thay vì phải giữ hàng nghìn từ của bài viết, ta chỉ cần lưu đúng **128 con số nguyên** (chiếm đúng 512 byte).
- Việc so sánh hai bài viết rút ngắn xuống còn phép so sánh 128 số nguyên $\implies$ Chạy nhanh gấp hàng nghìn lần!
