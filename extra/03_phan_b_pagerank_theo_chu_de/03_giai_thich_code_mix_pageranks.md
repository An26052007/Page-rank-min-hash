# GIẢI THÍCH CHI TIẾT HÀM `mix_pageranks`
> **Phần B — Ghép vector PageRank theo trọng số (Tính chất tuyến tính của PageRank)**  
> **Tệp:** `submission.py` (Dòng 146–165)

---

## 1. YÊU CẦU ĐỀ BÀI VÀ CƠ SỞ TOÁN HỌC

Trong mục **3. Phần B** của [bai-tap-wikispeedia.md](file:///C:/Users/HP/Downloads/HKI_2026-2027/GI%E1%BA%A2I%20THU%E1%BA%ACT%20KHDL/wikispeedia-student-package/bai-tap-wikispeedia.md):
- **Mục tiêu:** Một người dùng có thể quan tâm đến nhiều lĩnh vực cùng lúc với các mức độ ưu tiên khác nhau (ví dụ: 50% Khoa học, 30% Lịch sử, 20% Địa lý). Đề bài yêu cầu kết hợp kết quả PageRank của từng chủ đề thành một vector xếp hạng tổng hợp duy nhất:
  $$r_{\mathrm{mix}} = 0.5 \cdot r_{\mathrm{Science}} + 0.3 \cdot r_{\mathrm{History}} + 0.2 \cdot r_{\mathrm{Geography}}$$
- **Tính chất tuyến tính thần kỳ:** Vì phương trình PageRank là phương trình đại số tuyến tính:
  $$(I - \beta M) r = (1-\beta) v + \beta \delta_r u$$
  Toán tử $(I - \beta M)$ có tính chất tuyến tính (Linear Operator). Do đó, nghiệm hội tụ của tổ hợp tuyến tính các vector nguồn $v$ **chính là tổ hợp tuyến tính của các nghiệm thành phần**!
  $$\text{Nghiệm ứng với } (0.5 v_1 + 0.3 v_2 + 0.2 v_3) \equiv 0.5 r_1 + 0.3 r_2 + 0.2 r_3$$
  Nhờ đó, ta **không cần phải chạy lại thuật toán PageRank lặp tốn kém**; ta chỉ việc nhân vector và cộng lại trong chớp mắt!

---

## 2. BẢNG THÔNG SỐ ĐẦU VÀO VÀ ĐẦU RA

```python
def mix_pageranks(
    topic_scores: list[list[float]], weights: list[float]
) -> list[float]:
```

| Tên tham số / Đầu ra | Kiểu dữ liệu | Nguồn gốc / Ý nghĩa thực tế | Giá trị ví dụ |
|---|---|---|---|
| **`topic_scores`** | `list[list[float]]` | Danh sách gồm $k$ vector điểm PageRank ứng với $k$ chủ đề. Mỗi vector có độ dài $n$ ($4,604$ bài viết). | `[r_science, r_history, r_geography]` |
| **`weights`** | `list[float]` | Danh sách gồm $k$ trọng số tương ứng với $k$ chủ đề. Trọng số phải không âm và có tổng bằng đúng $1.0$. | `[0.5, 0.3, 0.2]` |
| **ĐẦU RA (`->`)** | `list[float]` | Vector điểm PageRank hỗn hợp $r_{\mathrm{mix}}$, độ dài $n = 4,604$, có tổng điểm bằng đúng $1.0$. | `[r_mix[0], r_mix[1], ..., r_mix[4603]]` |

---

## 3. TOÀN BỘ MÃ NGUỒN HÀM `mix_pageranks`

```python
def mix_pageranks(
    topic_scores: list[list[float]], weights: list[float]
) -> list[float]:
    if (not topic_scores or len(topic_scores) != len(weights)
            or any(len(vector) != len(topic_scores[0]) for vector in topic_scores)
            or any(not math.isfinite(weight) or weight < 0 for weight in weights)
            or abs(math.fsum(weights) - 1.0) > 1e-12):
        raise ValueError("Vector hoặc trọng số không hợp lệ")
    return [sum(weight * vector[i] for vector, weight in zip(topic_scores, weights))
            for i in range(len(topic_scores[0]))]
```

---

## 4. TỪ ĐIỂN BIẾN NỘI BỘ (LOCAL VARIABLES)

| Tên biến | Kiểu dữ liệu | Ý nghĩa & Vai trò trong thuật toán |
|---|---|---|
| **`topic_scores`** | `list[list[float]]` | Tham số chứa $k$ vector chủ đề (ma trận $k \times n$). |
| **`weights`** | `list[float]` | Tham số chứa $k$ trọng số tương ứng với từng hàng của ma trận. |
| **`vector`** | `list[float]` | Biến cục bộ trong vòng lặp đại diện cho 1 vector điểm của 1 chủ đề (độ dài $n$). |
| **`weight`** | `float` | Trọng số thực của chủ đề đó (ví dụ: `0.5`, `0.3`, `0.2`). |
| **`i`** | `int` | Chỉ số bài viết đang được tính toán, chạy từ đỉnh $0$ đến đỉnh $n-1$ ($4,603$). |

---

## 5. GIẢI THÍCH CHI TIẾT TỪNG DÒNG CODE

### Bước 1: Kiểm tra tính hợp lệ nghiêm ngặt của dữ liệu đầu vào (Validation)

```python
    if (not topic_scores or len(topic_scores) != len(weights)
            or any(len(vector) != len(topic_scores[0]) for vector in topic_scores)
            or any(not math.isfinite(weight) or weight < 0 for weight in weights)
            or abs(math.fsum(weights) - 1.0) > 1e-12):
        raise ValueError("Vector hoặc trọng số không hợp lệ")
```

Khối lệnh `if` gồm 5 điều kiện an toàn kết hợp bằng toán tử `or`:
1. **`not topic_scores`:** Danh sách rỗng (không truyền vào vector nào).
2. **`len(topic_scores) != len(weights)`:** Số lượng vector chủ đề và số lượng trọng số bị lệch nhau (ví dụ: đưa vào 3 vector nhưng chỉ đưa 2 trọng số).
3. **`any(len(vector) != len(topic_scores[0]) for vector in topic_scores)`:** Các vector không cùng kích thước (ví dụ: vector thứ nhất dài 4,604 phần tử nhưng vector thứ hai chỉ dài 100 phần tử).
4. **`any(not math.isfinite(weight) or weight < 0 for weight in weights)`:** Trọng số bị âm (`weight < 0`) hoặc chứa giá trị bất thường như vô cực (`inf`) hoặc không xác định (`nan`).
5. **`abs(math.fsum(weights) - 1.0) > 1e-12`:** Tổng các trọng số không bằng $1.0$ (dung sai cho phép là $10^{-12}$). Đây là điều kiện bắt buộc của tổ hợp lồi để đảm bảo tổng xác suất của vector kết quả sau khi ghép vẫn bằng $1.0$.

Nếu vi phạm bất kỳ điều kiện nào ở trên $\implies$ Hàm ném ngoại lệ `ValueError`.

---

### Bước 2: Tính tổng có trọng số theo từng tọa độ đỉnh

```python
    return [sum(weight * vector[i] for vector, weight in zip(topic_scores, weights))
            for i in range(len(topic_scores[0]))]
```

Dòng code này sử dụng List Comprehension kết hợp hàm `zip()` cực kỳ tinh gọn:
- **`for i in range(len(topic_scores[0]))`:** Duyệt lần lượt qua từng bài viết $i$ từ $0$ đến $4603$.
- **`zip(topic_scores, weights)`:** Ghép cặp từng vector chủ đề với trọng số tương ứng của nó:
  $$(r_{\mathrm{Science}}, 0.5), \quad (r_{\mathrm{History}}, 0.3), \quad (r_{\mathrm{Geography}}, 0.2)$$
- **`weight * vector[i]`:** Với bài viết thứ $i$, lấy điểm số của nó trong từng chủ đề nhân với trọng số của chủ đề đó:
  $$0.5 \cdot r_{\mathrm{Science}}[i], \quad 0.3 \cdot r_{\mathrm{History}}[i], \quad 0.2 \cdot r_{\mathrm{Geography}}[i]$$
- **`sum(...)`:** Cộng dồn 3 giá trị trên lại để được điểm PageRank hỗn hợp cuối cùng của bài viết $i$:
  $$r_{\mathrm{mix}}[i] = 0.5 \cdot r_{\mathrm{Science}}[i] + 0.3 \cdot r_{\mathrm{History}}[i] + 0.2 \cdot r_{\mathrm{Geography}}[i]$$
- Kết quả trả về là một danh sách số thực độ dài đúng bằng $n = 4604$.

---

## 6. TẠI SAO LẠI LÀM NHƯ VẬY? (THIẾT KẾ KỸ THUẬT & LÝ DO CỐT LÕI)

### 1. Tại sao dùng phép ghép tuyến tính thay vì chạy lại PageRank?
- **Tiết kiệm tài nguyên khổng lồ:** Thuật toán PageRank lặp 30–50 vòng trên đồ thị 4,604 đỉnh và 119,882 cạnh, tốn hàng triệu phép tính ma trận.
- Khi người dùng muốn đổi trọng số (ví dụ: chuyển từ $[0.5, 0.3, 0.2]$ sang $[0.8, 0.1, 0.1]$), nếu phải chạy lại PageRank thì hệ thống tìm kiếm sẽ bị nghẽn (lag).
- Vì toán tử PageRank là tuyến tính, ta chỉ cần tính trước (offline) $k$ vector cơ sở cho $k$ chủ đề. Khi có truy vấn từ người dùng (online query), ta chỉ cần làm phép nhân vô hướng $k \times n$ phép tính (mất chưa tới **0.002 giây**).

### 2. Tại sao dùng `math.fsum(weights)` thay vì hàm `sum(weights)` thông thường?
- Trong chuẩn số thực IEEE 754 (dấu phẩy động 64-bit), phép cộng thông thường `sum([0.1, 0.2, 0.7])` có thể sinh ra sai số làm tròn nhỏ (ví dụ: `0.9999999999999999` hoặc `1.0000000000000002`).
- Hàm `math.fsum()` của Python sử dụng giải thuật tích lũy theo dõi nhiều giá trị trung gian (Shewchuk algorithm), đảm bảo tính tổng chính xác đến từng bit cuối cùng (exact precision), tránh trường hợp kiểm tra tổng bằng 1 bị báo lỗi oan.

### 3. Tại sao chọn ngưỡng dung sai sai số là `1e-12`?
- Máy tính không thể biểu diễn số thực tuyệt đối vô hạn. Do đó không bao giờ được so sánh bằng tuyệt đối `math.fsum(weights) == 1.0`.
- Ngưỡng `1e-12` ($10^{-12}$) đủ nhỏ để loại bỏ các trường hợp nhập sai trọng số (như tổng bằng 0.99 hay 1.05), nhưng đủ rộng để bao dung cho sai số làm tròn thập phân của phần cứng CPU.

### 4. Tại sao phải kiểm tra `math.isfinite(weight)`?
- Trong Python, giá trị `float("inf")` (vô cực) hoặc `float("nan")` (Not a Number) vẫn được coi là kiểu `float`.
- Nếu không kiểm tra `math.isfinite()`, một trọng số `nan` sẽ làm lây nhiễm (poison) toàn bộ các phần tử trong vector kết quả thành `nan`, phá hỏng hoàn toàn thuật toán xếp hạng.

---

## 7. VÍ DỤ SỐ HỌC MINH HỌA DỄ HIỂU

Giả sử ta chỉ xét một đồ thị tí hon có 2 bài viết: $A$ và $B$.
Ta có kết quả PageRank của 2 chủ đề Khoa học ($S$) và Lịch sử ($H$):
- $r_S = [0.70, 0.30]$
- $r_H = [0.20, 0.80]$

Người dùng muốn kết hợp theo tỉ lệ $60\%$ Khoa học và $40\%$ Lịch sử:
- `weights = [0.6, 0.4]` (Tổng $= 0.6 + 0.4 = 1.0$)

Hàm `mix_pageranks` sẽ tính như sau:
- **Bài viết A ($i = 0$):**
  $$r_{\mathrm{mix}}[0] = 0.6 \times 0.70 + 0.4 \times 0.20 = 0.42 + 0.08 = 0.50$$
- **Bài viết B ($i = 1$):**
  $$r_{\mathrm{mix}}[1] = 0.6 \times 0.30 + 0.4 \times 0.80 = 0.18 + 0.32 = 0.50$$

Vector trả về: `[0.50, 0.50]`. Tổng điểm $= 0.50 + 0.50 = 1.0$. Tuyệt đối chính xác!
