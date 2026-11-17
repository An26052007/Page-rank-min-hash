# GIẢI THÍCH CHI TIẾT HÀM `pagerank`
> **Phần A & B — Thuật toán Lặp lũy thừa PageRank trên Đồ thị thưa**  
> **Tệp mã nguồn:** `submission.py` (Dòng 60–115)

---

## 1. BẢNG THAM SỐ ĐẦU VÀO VÀ ĐẦU RA (INPUT & OUTPUT)

```python
def pagerank(
    ids: list[str],
    out_neighbors: list[list[int]],
    beta: float,
    tol: float,
    max_iter: int,
    teleport: list[float] | None = None,
) -> list[float]
```

| Tên tham số | Kiểu dữ liệu | Nguồn truyền vào | Ý nghĩa thực tế | Giá trị mặc định |
|---|---|---|---|---|
| **`ids`** | `list[str]` | Kết quả trả về từ hàm `load_graph` | Danh sách tên của toàn bộ 4,604 bài viết Wikipedia đã sắp xếp chuẩn. | (Bắt buộc truyền) |
| **`out_neighbors`** | `list[list[int]]` | Kết quả trả về từ hàm `load_graph` | Bản đồ liên kết: danh sách các chỉ số đỉnh mà từng bài viết trỏ tới. | (Bắt buộc truyền) |
| **`beta`** | `float` | Từ tham số dòng lệnh `--beta` (hoặc test) | Hệ số giảm (damping factor): xác suất người lướt web đi theo link. | `0.85` |
| **`tol`** | `float` | Từ tham số dòng lệnh `--tol` (hoặc test) | Ngưỡng dung sai dừng lặp (tolerance): mức độ chênh lệch tối đa giữa 2 vòng. | `1e-10` |
| **`max_iter`** | `int` | Từ tham số dòng lệnh `--max-iter` | Giới hạn số vòng lặp tối đa để phòng trường hợp không hội tụ. | `1000` |
| **`teleport`** | `list[float] \| None` | Phần A truyền `None`, Phần B truyền vector chủ đề | Vector xác suất nhảy dù ngẫu nhiên. Nếu là `None` thì chia đều cho $n$ đỉnh. | `None` |

* **ĐẦU RA (Return):**
  - Trả về một danh sách gồm đúng 4,604 số thực (`list[float]`).
  - Mỗi phần tử là điểm số PageRank của bài viết tương ứng tại vị trí đó.
  - Tổng của toàn bộ các số trong danh sách luôn bằng đúng **`1.0`** (bảo toàn xác suất 100%).

---

## 2. BẢNG TỪ ĐIỂN CÁC BIẾN NỘI BỘ (VARIABLE DICTIONARY)

| Tên biến | Kiểu dữ liệu | Ý nghĩa và vai trò trong thuật toán |
|---|---|---|
| **`n`** | `int` | Số lượng đỉnh của đồ thị, được tính bằng `len(ids)` ($4604$). |
| **`v`** | `list[float]` | Vector phân phối nhảy dù chuẩn hóa (dài $n$ phần tử, tổng bằng $1.0$). |
| **`scores`** | `list[float]` | Vector điểm PageRank ở vòng lặp hiện tại $t$ (khởi tạo bằng bản sao của $v$). |
| **`dangling`** | `float` | Tổng điểm tích tụ tại các nút cụt ở vòng hiện tại ($\delta_t = \sum_{j: d_j=0} r_j^t$). |
| **`base`** | `float` | Năng lượng bù từ nút cụt chia đều cho mỗi đỉnh, bằng $\frac{\beta \delta_t}{n}$. |
| **`updated`** | `list[float]` | Vector điểm mới ở vòng $t+1$, được điền sẵn điểm nền và cộng dồn điểm qua link. |
| **`j`** | `int` | Chỉ số đỉnh nguồn đang xét ($0 \le j < n$). |
| **`neighbors`** | `list[int]` | Danh sách các chỉ số đỉnh đích mà đỉnh nguồn $j$ đặt link trỏ tới. |
| **`contribution`** | `float` | Lượng điểm mà đỉnh $j$ chia đều cho mỗi đỉnh con, bằng $\beta \frac{scores[j]}{\text{len}(neighbors)}$. |
| **`target`** | `int` | Chỉ số đỉnh đích nhận điểm đóng góp từ đỉnh nguồn $j$. |
| **`difference`** | `float` | Sai khác khoảng cách chuẩn $L_1$ giữa vector mới và vector cũ: $\sum |new - old|$. |
| **`_`** | `int` | Biến đếm vòng lặp chạy từ $0$ đến `max_iter - 1`. |

---

## 3. TOÀN BỘ MÃ NGUỒN

```python
def pagerank(
    ids: list[str],
    out_neighbors: list[list[int]],
    beta: float,
    tol: float,
    max_iter: int,
    teleport: list[float] | None = None,
) -> list[float]:
    n = len(ids)
    if (n == 0 or len(out_neighbors) != n or not math.isfinite(beta)
            or not 0 < beta < 1 or not math.isfinite(tol) or tol <= 0
            or not isinstance(max_iter, int) or isinstance(max_iter, bool)
            or max_iter < 1):
        raise ValueError("Tham số PageRank không hợp lệ")
    if any(not isinstance(target, int) or not 0 <= target < n
           for neighbors in out_neighbors for target in neighbors):
        raise ValueError("Chỉ số đỉnh đích không hợp lệ")

    if teleport is None:
        v = [1.0 / n] * n
    else:
        if (len(teleport) != n
                or any(not math.isfinite(value) or value < 0 for value in teleport)
                or abs(math.fsum(teleport) - 1.0) > 1e-12):
            raise ValueError("Phân phối dịch chuyển không hợp lệ")
        v = list(teleport)

    scores = v.copy()
    for _ in range(max_iter):
        dangling = sum(scores[j] for j, neighbors in enumerate(out_neighbors)
                       if not neighbors)
        base = beta * dangling / n
        updated = [base + (1.0 - beta) * value for value in v]
        for j, neighbors in enumerate(out_neighbors):
            if neighbors:
                contribution = beta * scores[j] / len(neighbors)
                for target in neighbors:
                    updated[target] += contribution
        difference = sum(abs(new - old) for new, old in zip(updated, scores))
        scores = updated
        if difference <= tol:
            break
    return scores
```

---

## 4. BÓC TÁCH TỪNG BƯỚC NHỎ VÀ GIẢI THÍCH "TẠI SAO LẠI LÀM NHƯ VẬY?"

### 🟢 Bước 1: Kiểm tra tính hợp lệ của tham số (Defensive Programming)

```python
    n = len(ids)
    if (n == 0 or len(out_neighbors) != n or not math.isfinite(beta)
            or not 0 < beta < 1 or not math.isfinite(tol) or tol <= 0
            or not isinstance(max_iter, int) or isinstance(max_iter, bool)
            or max_iter < 1):
        raise ValueError("Tham số PageRank không hợp lệ")
```
* **Tại sao lại làm như vậy?**
  1. *Tại sao kiểm tra `isinstance(max_iter, bool)`?*  
     Trong Python, `bool` là lớp con của `int` (`isinstance(True, int)` trả về `True`). Nếu người dùng truyền nhầm `True` vào `max_iter`, vòng lặp sẽ chỉ chạy đúng 1 lần! Kiểm tra này loại trừ lỗi tiềm ẩn.
  2. *Tại sao kiểm tra `0 < beta < 1`?*  
     Nếu $\beta = 1$, người lướt web không bao giờ nhảy dù, dẫn đến bị kẹt trong bẫy nhện (*Spider Traps*). Nếu $\beta = 0$, điểm số chỉ phụ thuộc vào nhảy dù mà bỏ qua toàn bộ cấu trúc đồ thị. Do đó $\beta$ bắt buộc phải nằm trong khoảng $(0, 1)$.
  3. *Tại sao kiểm tra `math.isfinite`?*  
     Loại trừ các giá trị vô cực `inf` hoặc giá trị lỗi toán học `nan`.

---

### 🟢 Bước 2: Thiết lập vector dịch chuyển $v$ và khởi tạo $r^0$

```python
    if teleport is None:
        v = [1.0 / n] * n
    else:
        if (len(teleport) != n
                or any(not math.isfinite(value) or value < 0 for value in teleport)
                or abs(math.fsum(teleport) - 1.0) > 1e-12):
            raise ValueError("Phân phối dịch chuyển không hợp lệ")
        v = list(teleport)

    scores = v.copy()
```

* **Tại sao lại làm như vậy?**
  - *Khi `teleport is None` (Phần A toàn cục):* Mọi đỉnh đều bình đẳng, xác suất nhảy dù vào mỗi bài viết là $\frac{1}{n} = \frac{1}{4604}$.
  - *Khi có `teleport` (Phần B chủ đề):* Phải kiểm tra tổng xác suất có đúng bằng $1.0$ không bằng `math.fsum` (chính xác hơn hàm `sum` thông thường với số thực).
  - *Tại sao dùng `scores = v.copy()`?*  
    Quy ước toán học của Power Iteration: điểm bắt đầu ở vòng 0 bằng chính vector dịch chuyển $r^0 = v$. Dùng `.copy()` để tránh việc sửa đổi làm ảnh hưởng đến mảng gốc `v`.

---

### 🟢 Bước 3: Thu gom điểm nút cụt (Dead-end Handling)

```python
    for _ in range(max_iter):
        dangling = sum(scores[j] for j, neighbors in enumerate(out_neighbors)
                       if not neighbors)
        base = beta * dangling / n
```

* **Tại sao lại làm như vậy?**
  - Trong Wikispeedia có đúng **17 nút cụt** (bài viết không có link trỏ ra ngoài).
  - Nếu không xử lý, người lướt web đi vào 17 bài này sẽ bị kẹt lại và rơi xuống biển, khiến tổng điểm của toàn mạng lưới cứ hao hụt dần sau mỗi vòng lặp.
  - Câu lệnh `if not neighbors` lọc ra đúng các bài có danh sách đích rỗng. Biến `dangling` gom toàn bộ số điểm này lại, sau đó nhân với $\beta$ và chia đều lại cho toàn bộ $n$ bài viết: $\text{base} = \frac{\beta \delta_t}{n}$.
  - Nhờ vậy, **tổng điểm của toàn mạng lưới luôn bảo toàn chính xác bằng 1.0** mà không cần phải dùng lệnh chia chuẩn hóa nhân tạo!

---

### 🟢 Bước 4: Khởi tạo vector mới và Kỹ thuật lan truyền Forward-Push

```python
        updated = [base + (1.0 - beta) * value for value in v]
        for j, neighbors in enumerate(out_neighbors):
            if neighbors:
                contribution = beta * scores[j] / len(neighbors)
                for target in neighbors:
                    updated[target] += contribution
```

* **Tại sao lại làm như vậy? (Kỹ thuật Đồ thị thưa đỉnh cao):**
  - *Điền sẵn năng lượng nền:* Mỗi đỉnh nhận trước năng lượng bù nút cụt (`base`) cộng với năng lượng nhảy dù ngẫu nhiên $(1-\beta)v_i$.
  - *Kỹ thuật Forward-Push (Bắn điểm về phía trước):*
    - Cách thông thường (Backward): Đỉnh $i$ phải đi tìm xem trên toàn thế giới có những đỉnh $j$ nào trỏ vào nó $\implies$ Rất tốn công tìm kiếm!
    - Cách Forward-Push: Duyệt xuôi theo `out_neighbors`. Đỉnh $j$ có bậc ra `len(neighbors)` $\implies$ Nó chia đều điểm của nó cho các con: `contribution = beta * scores[j] / len(neighbors)` rồi cộng trực tiếp vào `updated[target]`.
  - Toàn bộ 119,882 cạnh chỉ được duyệt đúng một lần theo một chiều duy nhất $\implies$ **Tốc độ lặp chỉ mất 0.005 giây mỗi vòng!**

---

### 🟢 Bước 5: Kiểm tra điều kiện hội tụ chuẩn $L_1$

```python
        difference = sum(abs(new - old) for new, old in zip(updated, scores))
        scores = updated
        if difference <= tol:
            break
    return scores
```

* **Tại sao lại làm như vậy?**
  - `difference`: Tổng độ lệch tuyệt đối giữa các phần tử của 2 vòng liên tiếp ($\lVert r^{t+1} - r^t \rVert_1$).
  - Khi `difference <= tol` ($10^{-10}$): Nghĩa là điểm số của 4,604 bài viết đã dừng dao động và đạt trạng thái cân bằng bền vững (hội tụ). Lập tức ngắt vòng lặp (`break`) để tiết kiệm thời gian, không cần chạy cố đến 1000 vòng.
  - Trả về `scores` thỏa mãn yêu cầu đề bài.
