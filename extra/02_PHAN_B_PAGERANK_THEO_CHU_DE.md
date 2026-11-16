# BÀI 02: PHẦN B — PAGERANK THEO CHỦ ĐỀ VÀ TÍNH CHẤT TUYẾN TÍNH (TOPIC-SENSITIVE PAGERANK)

---

## 1. Tại sao lại cần PageRank theo chủ đề?

Hãy xem xét một ví dụ thực tế:
Khi một người dùng gõ từ khóa tìm kiếm: **"Jaguar"**, công cụ tìm kiếm nên trả về kết quả gì?
- Nếu người tìm kiếm là một **nhà sinh vật học**, anh ta muốn đọc về **loài báo đốm Jaguar**.
- Nếu người tìm kiếm là một **người mê xe hơi**, anh ta muốn đọc về **hãng xe hơi thể thao cao cấp Jaguar**.
- Nếu là một **lập trình viên Apple**, anh ta có thể muốn xem thông tin về hệ điều hành **Mac OS X Jaguar**.

Thuật toán **PageRank toàn cục** ở Phần A tính ra một điểm số cố định cho mỗi trang trên toàn mạng, bất kể người tìm kiếm là ai. Nếu trang web của hãng xe Jaguar nhận được nhiều link hơn, nó sẽ luôn đứng trên bài viết về loài báo, gây thất vọng cho nhà sinh vật học.

Năm 2002, nhà khoa học Taher Haveliwala (Đại học Stanford) đề xuất thuật toán **Topic-Sensitive PageRank (PageRank theo chủ đề / Cá nhân hóa)**:
> Thay vì một bảng xếp hạng chung duy nhất, ta tính sẵn các bảng xếp hạng riêng biệt cho từng chủ đề lớn (Khoa học, Lịch sử, Địa lý,...). Khi biết người dùng đang quan tâm đến lĩnh vực nào, ta sẽ dùng bảng xếp hạng của lĩnh vực đó!

---

## 2. Mô hình Người lướt web có sở thích (Biased Random Surfer)

Ở Phần A, người lướt web khi cảm thấy chán nản ($15\%$ xác suất) sẽ "nhảy dù" ngẫu nhiên về **bất kỳ trang nào** trong $n$ trang của mạng lưới với xác suất đều $\frac{1}{n}$.

Ở Phần B, ta giả định người lướt web là một **người yêu khoa học**:
- Khi anh ta "nhảy dù", anh ta **chỉ nhảy vào các trang thuộc danh mục Khoa học**!
- Các trang không thuộc Khoa học sẽ có xác suất nhảy dù bằng $0$.

### Vector dịch chuyển theo chủ đề $v$:
Gọi $S$ là tập hợp tất cả các bài viết thuộc chủ đề đang xét (ví dụ: Khoa học).
Vector dịch chuyển $v$ được xác định:
$$v_i = \begin{cases} \frac{1}{|S|}, & \text{nếu bài viết } i \text{ thuộc chủ đề } S \\ 0, & \text{nếu bài viết } i \text{ KHÔNG thuộc chủ đề } S \end{cases}$$

Trong dữ liệu Wikispeedia, ba chủ đề chính thức gồm:
1. **Science** (`subject.Science`): Gồm **1,105 bài viết** $\implies v_i = \frac{1}{1105}$.
2. **History** (`subject.History`): Gồm **525 bài viết** $\implies v_i = \frac{1}{525}$.
3. **Geography** (`subject.Geography`): Gồm **1,063 bài viết** $\implies v_i = \frac{1}{1063}$.

---

## 3. Quy tắc bù nút cụt đặc biệt của bài toán

Đây là điểm mấu chốt kỹ thuật mà sinh viên thường hay nhầm lẫn nhất:

> [!IMPORTANT]
> **Quy ước chuẩn của Bài học 04:**
> - Nhảy dù ngẫu nhiên: Nhảy theo sở thích chủ đề $\implies$ Dùng vector $(1-\beta)v_i$.
> - Bù thất thoát do nút cụt (Dead ends $\delta_t$): Vẫn phân phối đều trên toàn bộ mạng lưới $\implies$ Dùng $\beta \delta_t \frac{1}{n}$ (phân phối đều $u$).

Công thức cập nhật hoàn chỉnh:
$$r_i^{t+1} = \beta \sum_{j \to i} \frac{r_j^t}{d_j} + \beta \delta_t \frac{1}{n} + (1-\beta) v_i$$

**Câu hỏi thường gặp:** *"Nếu một bài viết không thuộc chủ đề Khoa học thì điểm PageRank Khoa học của nó có bằng 0 không?"*
**Trả lời:** **KHÔNG**.
Bài viết đó nhận $v_i = 0$ (không được nhảy dù trực tiếp), nhưng nó vẫn có thể nhận điểm từ các bài viết khoa học khác trỏ link tới nó! Điểm số này phản ánh mức độ liên quan gián tiếp của bài viết đối với lĩnh vực khoa học.

---

## 4. Tính chất Tuyến tính và Phép ghép Vector ($r_{\text{mix}}$)

### 4.1. Bài toán thực tế
Giả sử có một người dùng vừa thích Khoa học ($50\%$), vừa thích Lịch sử ($30\%$), vừa thích Địa lý ($20\%$).
Liệu ta có phải chạy lại toàn bộ thuật toán PageRank lặp hàng chục vòng từ đầu với phân phối kết hợp:
$$v_{\text{mix}} = 0.5 \cdot v_{\text{Science}} + 0.3 \cdot v_{\text{History}} + 0.2 \cdot v_{\text{Geography}}$$
không?

**Câu trả lời là KHÔNG CẦN!**

### 4.2. Tính chất tuyến tính kỳ diệu của PageRank
Phương trình cân bằng PageRank có dạng đại số tuyến tính:
$$r = \beta M r + \beta \delta_r u + (1-\beta) v$$
Vì toán tử PageRank là tuyến tính đối với vector nguồn dịch chuyển $v$, nên nếu:
$$v_{\text{mix}} = \sum_{k} w_k v_k \quad \text{với } \sum w_k = 1, w_k \ge 0$$
Thì nghiệm dừng tương ứng thỏa mãn chính xác:
$$r_{\text{mix}} = \sum_{k} w_k r_k$$

Cụ thể trong bài tập:
$$r_{\text{mix}} = 0.5 \cdot r_{\text{Science}} + 0.3 \cdot r_{\text{History}} + 0.2 \cdot r_{\text{Geography}}$$

Bạn chỉ cần lấy tổng có trọng số của 3 vector đã tính ở trước đó. Phép tính này chỉ tốn vài mili-giây thay vì phải lặp lại hàng trăm vòng lặp tính toán!

---

## 5. Giải thích code trong `submission.py`

### 5.1. Hàm `topic_distribution`
Đọc `categories.tsv` và gán xác suất $\frac{1}{|S|}$ cho các bài thuộc chủ đề:

```python
def topic_distribution(ids: list[str], categories_path: Path, prefix: str) -> list[float]:
    known_ids = set(ids)
    members = set() # Dùng set để mỗi bài chỉ tính 1 lần dù có nhiều nhãn con

    with categories_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            article, label = line.rstrip("\r\n").split("\t")
            if article not in known_ids:
                raise ValueError(f"Nhãn tham chiếu ID không tồn tại: {article}")
            # Khớp nhãn theo tiền tố (Ví dụ: subject.Science hoặc subject.Science.Physics)
            if label == prefix or label.startswith(prefix + "."):
                members.add(article)

    if not members:
        raise ValueError(f"Chủ đề không có bài viết: {prefix}")

    probability = 1.0 / len(members)
    # Các bài trong chủ đề nhận 1/|S|, các bài còn lại nhận 0.0
    return [probability if article in members else 0.0 for article in ids]
```

### 5.2. Hàm `mix_pageranks`
Ghép các vector điểm theo trọng số:

```python
def mix_pageranks(topic_scores: list[list[float]], weights: list[float]) -> list[float]:
    # Kiểm tra tính hợp lệ của trọng số (tổng phải xấp xỉ 1.0, không có số âm)
    if (not topic_scores or len(topic_scores) != len(weights)
            or any(len(vector) != len(topic_scores[0]) for vector in topic_scores)
            or any(not math.isfinite(weight) or weight < 0 for weight in weights)
            or abs(math.fsum(weights) - 1.0) > 1e-12):
        raise ValueError("Vector hoặc trọng số không hợp lệ")

    # Nhân từng tọa độ với trọng số tương ứng và cộng lại
    n_nodes = len(topic_scores[0])
    return [
        sum(weight * vector[i] for vector, weight in zip(topic_scores, weights))
        for i in range(n_nodes)
    ]
```
