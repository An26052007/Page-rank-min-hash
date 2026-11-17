# BÀI 01: PHẦN A — THUẬT TOÁN PAGERANK TOÀN CỤC (GLOBAL PAGERANK)

---

## 1. Trực giác đời thực: PageRank là gì?

Vào những năm 1990, khi bạn gõ tìm kiếm trên mạng, các công cụ tìm kiếm cũ chỉ đơn thuần đếm xem từ khóa xuất hiện bao nhiêu lần trong bài viết. Điều này dẫn đến tình trạng các trang web rác chỉ cần lặp đi lặp lại từ khóa hàng nghìn lần là được đẩy lên đầu.

Hai nhà sáng lập của Google (Larry Page và Sergey Brin) đã phát minh ra **PageRank** dựa trên một ý tưởng cách mạng:
> **"Mỗi đường link dẫn từ trang A sang trang B có thể coi như một PHIẾU BẦU của trang A dành cho trang B."**

Tuy nhiên, không phải phiếu bầu nào cũng có giá trị như nhau:
- Một phiếu bầu từ trang báo uy tín như `Báo Tuổi Trẻ` hay `Wikipedia` có giá trị gấp hàng triệu lần một phiếu bầu từ một blog cá nhân vô danh.
- Nếu một trang bầu cho quá nhiều người (ví dụ trang có 100 đường link ra ngoài), thì giá trị phiếu bầu mà mỗi người nhận được sẽ bị chia nhỏ ra ($\frac{1}{100}$).

```
[ Trang lớn có điểm cao ] ---- (chia đều điểm) ----> [ Trang được bầu ]
```

---

## 2. Mô hình Người lướt web ngẫu nhiên (Random Surfer Model)

Để tính điểm PageRank, các nhà khoa học tưởng tượng có một **người lướt web ngẫu nhiên**:
1. Anh ta bắt đầu ở một trang bất kỳ.
2. Tại mỗi trang, anh ta nhắm mắt nhấp ngẫu nhiên vào một trong các đường link có trên trang đó để đi tiếp.
3. Sau khi anh ta lướt hàng triệu lần, **tỷ lệ thời gian anh ta dừng chân ở trang nào nhiều nhất** chính là **điểm PageRank (độ quan trọng)** của trang đó!

Nhưng trên Internet thực tế, mô hình này gặp **2 cái bẫy lớn**:

### Cái bẫy 1: Spider Traps (Bẫy nhện)
- **Hiện tượng:** Có một nhóm trang web chỉ trỏ link vòng tròn cho nhau ($A \to B \to C \to A$) mà không có link ra ngoài thế giới. Người lướt web khi đi vào đây sẽ bị kẹt vĩnh viễn, khiến điểm của các trang này hút hết toàn bộ điểm của mạng Internet.
- **Giải pháp:** Đưa vào **Hệ số dịch chuyển ngẫu nhiên (Damping factor $\beta = 0.85$)**:
  - Với xác suất $85\%$, người lướt web tiếp tục nhấp vào link trên trang.
  - Với xác suất $15\%$ ($1 - \beta$), anh ta cảm thấy chán nản, mở thanh địa chỉ trình duyệt ra và gõ ngẫu nhiên một địa chỉ bất kỳ trên toàn bộ mạng lưới để nhảy tới (*Teleportation*).
  - Nhờ cú "nhảy dù" này, anh ta sẽ thoát khỏi mọi bẫy nhện!

### Cái bẫy 2: Dead Ends (Nút cụt)
- **Hiện tượng:** Một trang không có bất kỳ đường link nào trỏ ra ngoài ($d_j = 0$). Khi người lướt web đi vào trang này, anh ta không thể đi tiếp, làm cho tổng xác suất của toàn mạng bị hao hụt dần về 0 sau mỗi bước lặp.
- **Giải pháp:** Nếu rơi vào nút cụt, anh ta sẽ tự động "nhảy dù" đều về một trang ngẫu nhiên trong toàn bộ $n$ trang của hệ thống với xác suất đều $\frac{1}{n}$.

---

## 3. Công thức toán học và Phương pháp lặp lũy thừa (Power Iteration)

### 3.1. Công thức cập nhật đồng thời
Tại mỗi vòng lặp $t$, điểm số mới $r_i^{t+1}$ của đỉnh $i$ được tính bằng tổng của 3 thành phần:

$$r_i^{t+1} = \underbrace{\beta \sum_{j \to i} \frac{r_j^t}{d_j}}_{\text{(1) Nhận từ các trang trỏ tới}} + \underbrace{\beta \delta_t \frac{1}{n}}_{\text{(2) Bù từ các nút cụt}} + \underbrace{(1-\beta)\frac{1}{n}}_{\text{(3) Nhảy dù ngẫu nhiên}}$$

Trong đó:
- $n = 4604$: Tổng số bài viết.
- $d_j$: Số link trỏ ra của bài viết $j$ ($d_j > 0$).
- $j \to i$: Tập các bài viết $j$ có đường link trỏ tới bài viết $i$.
- $\delta_t = \sum_{j \in \text{dead\_ends}} r_j^t$: Tổng điểm của tất cả các bài viết là nút cụt ở vòng $t$. Lượng điểm này được chia đều cho toàn bộ $n$ bài viết.

### 3.2. Quá trình lặp lũy thừa (Power Iteration)
1. **Khởi tạo (vòng $t = 0$):** Chia đều điểm cho tất cả các trang:
   $$r_i^0 = \frac{1}{n} = \frac{1}{4604}$$
2. **Cập nhật:** Áp dụng công thức trên để tính ra vector điểm mới $r^{t+1}$.
3. **Kiểm tra hội tụ (Điều kiện dừng):**
   Tính tổng độ lệch tuyệt đối giữa vòng mới và vòng cũ (gọi là Chuẩn khoảng cách $L_1$):
   $$\lVert r^{t+1} - r^t \rVert_1 = \sum_{i=1}^n |r_i^{t+1} - r_i^t|$$
   - Nếu $\lVert r^{t+1} - r^t \rVert_1 \le 10^{-10}$ (sai số cực nhỏ): Thuật toán đã hội tụ, dừng lặp và trả về kết quả!
   - Nếu đã lặp tới `max_iter = 1000` vòng mà chưa đạt thì cũng dừng lại.

> [!CAUTION]
> **Quy định nghiêm ngặt của đề bài:**
> Không được chuẩn hóa lại vector ở cuối mỗi vòng lặp (ví dụ không được làm lệnh `r = r / sum(r)`). Nếu bạn tính đúng công thức nút cụt $\delta_t$, tổng xác suất $\sum r_i$ sẽ **tự động luôn luôn bằng đúng 1.0**. Chuẩn hóa nhân tạo sẽ che giấu lỗi code tính sai nút cụt.

---

## 4. Tại sao không dùng Ma trận lớn? (Kỹ thuật Đồ thị thưa)

Nếu bạn học đại số tuyến tính, PageRank thường được viết dạng nhân ma trận:
$$r^{t+1} = M \cdot r^t$$
Nhưng với đồ thị lớn:
- Ma trận $M$ có kích thước $4604 \times 4604 \approx 21.2 \text{ triệu ô}$. Mỗi vòng lặp tốn hơn 21 triệu phép tính nhân!
- Trên thực tế, mỗi bài viết chỉ có trung bình khoảng 26 đường link. Hơn $99\%$ số ô trong ma trận là số 0!
- **Kỹ thuật Đồ thị thưa (Sparse Representation):** Thay vì tạo ma trận khổng lồ đầy số 0, ta chỉ duyệt qua danh sách các cạnh thực sự tồn tại ($119,882$ cạnh). Nhờ vậy, một vòng lặp chỉ tốn chưa tới $0.005$ giây trên máy tính!

---

## 5. Giải thích từng dòng code hàm `pagerank` trong `submission.py`

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

    # 1. Kiểm tra tính hợp lệ của tham số đầu vào (Tránh lỗi hệ thống)
    if (n == 0 or len(out_neighbors) != n or not math.isfinite(beta)
            or not 0 < beta < 1 or not math.isfinite(tol) or tol <= 0
            or not isinstance(max_iter, int) or isinstance(max_iter, bool)
            or max_iter < 1):
        raise ValueError("Tham số PageRank không hợp lệ")

    # 2. Xác định vector phân phối nhảy dù v (Mặc định là phân phối đều 1/n)
    if teleport is None:
        v = [1.0 / n] * n
    else:
        v = list(teleport)

    # 3. Khởi tạo vector điểm ban đầu r^0 = v
    scores = v.copy()

    # 4. Vòng lặp lũy thừa (Power Iteration)
    for _ in range(max_iter):
        # Bước 4.1: Tính tổng điểm tích tụ ở các nút cụt (không có link ra)
        dangling = sum(scores[j] for j, neighbors in enumerate(out_neighbors) if not neighbors)

        # Bước 4.2: Điểm nền nhận được từ nút cụt chia đều
        base = beta * dangling / n

        # Bước 4.3: Điền trước điểm nền + điểm nhảy dù ngẫu nhiên cho tất cả các đỉnh
        updated = [base + (1.0 - beta) * value for value in v]

        # Bước 4.4: Lan truyền điểm qua các đường link thực tế (Duyệt đồ thị thưa)
        for j, neighbors in enumerate(out_neighbors):
            if neighbors:
                # Trang j chia đều điểm của nó cho tất cả các trang đích mà nó trỏ tới
                contribution = beta * scores[j] / len(neighbors)
                for target in neighbors:
                    updated[target] += contribution

        # Bước 4.5: Tính sai số L1 giữa vòng mới và vòng cũ
        difference = sum(abs(new - old) for new, old in zip(updated, scores))
        scores = updated

        # Nếu sai số nhỏ hơn ngưỡng dung sai tol (1e-10) thì dừng sớm
        if difference <= tol:
            break

    return scores
```
