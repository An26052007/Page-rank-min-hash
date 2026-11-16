# BÀI 00: TỔNG QUAN VÀ BỐI CẢNH BỘ DỮ LIỆU WIKISPEEDIA

---

## 1. Trò chơi Wikispeedia là gì?

Hãy tưởng tượng bạn đang mở trang Wikipedia tiếng Anh bài viết về quả táo (**Apple**). Đề bài yêu cầu bạn phải tìm cách đi đến bài viết về đại văn hào **William Shakespeare**, nhưng bạn **không được dùng ô tìm kiếm (Search)**. Bạn chỉ được phép nhấp chuột vào các đường link màu xanh (hyperlinks) có sẵn trong bài viết.
- Từ **Apple** bạn nhấp link sang **England** (nước Anh trồng nhiều táo).
- Từ **England** bạn nhấp link sang **English literature** (văn học Anh).
- Từ **English literature** bạn thấy ngay đường link dẫn tới **William Shakespeare**!

Đó chính là trò chơi **Wikispeedia**, một dự án nghiên cứu nổi tiếng của Đại học Stanford (SNAP). Người ta thu thập lại:
1. Mạng lưới các bài viết và đường link nối giữa các bài viết đó.
2. Nội dung văn bản của từng bài viết.
3. Phân loại danh mục bài viết (Khoa học, Lịch sử, Địa lý,...).

Bộ dữ liệu trong bài tập của chúng ta là phiên bản rút gọn chính thức gồm **4,604 bài viết** được chọn lọc.

---

## 2. Các khái niệm đồ thị (Graph) cơ bản cần biết

Trước khi đụng vào code hay công thức, bạn chỉ cần nắm 4 khái niệm hình học cực kỳ trực quan sau:

```
[ Bài A ] -------- link --------> [ Bài B ]
(Đỉnh nguồn / Source)         (Đỉnh đích / Target)
```

1. **Đỉnh (Node / Vertex):** Đại diện cho một bài viết Wikipedia (Ví dụ: `Apple`, `Computer`, `Albert_Einstein`). Tổng cộng có **4,604 đỉnh**.
2. **Cạnh có hướng (Directed Edge):** Đại diện cho một đường link dẫn từ bài viết $A$ sang bài viết $B$. Nếu bài $A$ có link tới $B$, không có nghĩa là bài $B$ sẽ có link quay ngược lại $A$. Tổng cộng có **119,882 cạnh phân biệt**.
3. **Bậc ra (Out-degree $d_j$):** Số lượng đường link từ bài viết $j$ trỏ ra các bài viết khác.
   - Ví dụ: Bài viết `Vietnam` có link tới `Hanoi`, `Asia`, `ASEAN` thì bậc ra $d = 3$.
4. **Nút cụt (Dead-end / Dangling node):** Là bài viết **không có bất kỳ đường link nào trỏ ra ngoài** ($d_j = 0$). Người lướt web đi vào đây sẽ bị kẹt lại! Trong dữ liệu Wikispeedia có đúng **17 nút cụt**.
5. **Cạnh tự nối (Self-loop):** Một bài viết tự đặt link trỏ về chính nó ($A \to A$). Trong dữ liệu có đúng **110 cạnh tự nối** (Đề bài yêu cầu phải giữ nguyên, không được xóa).

---

## 3. Cấu trúc các tệp dữ liệu trong thư mục `data/`

Sau khi bạn giải nén hai tệp nén `.tar.gz`, thư mục `data/` sẽ có:

### 3.1. Thư mục `wikispeedia_paths-and-graph/`
Chứa thông tin cấu trúc mạng lưới:
- **`articles.tsv`:** Danh sách toàn bộ tên bài viết (mỗi dòng 1 bài).
  - *Đặc điểm:* Các ký tự đặc biệt được mã hóa theo chuẩn URL (gọi là Percent-encoding). Ví dụ ký tự `É` được viết là `%C3%89`. **Quy ước:** Giữ nguyên chuỗi này, KHÔNG giải mã (unquote) thành chữ `É`.
- **`links.tsv`:** Danh sách các liên kết giữa các bài viết.
  - Mỗi dòng có dạng: `Tên_bài_nguồn <TAB> Tên_bài_đích`.
  - Có thể có những dòng lặp trùng $\implies$ Phải gộp lại thành 1 cạnh duy nhất.
- **`categories.tsv`:** Phân loại chủ đề của từng bài viết.
  - Mỗi dòng có dạng: `Tên_bài <TAB> Tên_danh_mục`.
  - Ví dụ: `Albert_Einstein <TAB> subject.Science.Physics.Physicists`.

### 3.2. Thư mục `plaintext_articles/`
Chứa **4,604 tệp văn bản thô `.txt`**, mỗi tệp chứa nội dung của một bài viết Wikipedia (tên tệp tương ứng với ID bài viết trong `articles.tsv`).

### 3.3. Tệp `pairs.csv`
Gồm **100 cặp bài viết** được chọn sẵn để kiểm tra thuật toán đo độ tương đồng văn bản ở Phần C.

---

## 4. Quy ước đọc dữ liệu và cài đặt hàm `load_graph`

Hàm đầu tiên trong bài tập là `load_graph(articles_path, links_path)` trong `submission.py`.

### 4.1. Quy tắc bắt buộc của đề bài:
1. Bỏ qua các dòng trống và dòng bắt đầu bằng ký tự `#` (dòng chú thích).
2. Tập đỉnh phải lấy từ `articles.tsv`, sắp xếp theo **thứ tự byte UTF-8** (`article.encode('utf-8')`).
3. Gộp cạnh trùng nhau, giữ cạnh tự nối.
4. Báo lỗi `ValueError` nếu:
   - Đồ thị không có đỉnh nào (tệp rỗng).
   - Trong `links.tsv` xuất hiện bài viết nào không có tên trong `articles.tsv`.

### 4.2. Giải thích code thực tế trong `submission.py`:

```python
def load_graph(articles_path: Path, links_path: Path) -> tuple[list[str], list[list[int]]]:
    # Bước 1: Đọc tất cả các đỉnh và sắp xếp theo byte UTF-8
    with articles_path.open(encoding="utf-8") as handle:
        ids = sorted(
            {line.rstrip("\r\n") for line in handle
             if line.strip() and not line.startswith("#")},
            key=lambda article: article.encode("utf-8"), # Sắp xếp theo byte UTF-8
        )
    if not ids:
        raise ValueError("Đồ thị không có đỉnh")

    # Bước 2: Tạo từ điển đánh chỉ số từ 0 đến N-1 để tra cứu nhanh O(1)
    index = {article: i for i, article in enumerate(ids)}
    neighbors = [set() for _ in ids] # Dùng set để tự động gộp các cạnh trùng lặp

    # Bước 3: Đọc các liên kết trong links.tsv
    with links_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            source, target = line.rstrip("\r\n").split("\t")
            # Kiểm tra xem 2 đầu mút có nằm trong tập đỉnh không
            if source not in index or target not in index:
                raise ValueError(f"Cạnh chứa ID không tồn tại: {source}, {target}")
            # Thêm đỉnh đích vào danh sách kề của đỉnh nguồn
            neighbors[index[source]].add(index[target])

    # Bước 4: Chuyển set thành list đã sắp xếp
    return ids, [sorted(destinations) for destinations in neighbors]
```

- **Kết quả trả về:**
  - `ids`: Danh sách 4,604 tên bài viết đã sắp xếp chuẩn.
  - `out_neighbors`: Danh sách kề. `out_neighbors[j]` chứa danh sách các chỉ số đỉnh mà bài viết thứ `j` trỏ tới. Nếu bài `j` là nút cụt, `out_neighbors[j] = []`.
