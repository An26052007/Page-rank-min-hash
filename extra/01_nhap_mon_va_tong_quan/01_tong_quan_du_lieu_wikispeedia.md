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

## 4. Quy ước đọc dữ liệu và giải thích chi tiết hàm `load_graph`

Hàm đầu tiên trong bài tập là `load_graph(articles_path, links_path)` trong `submission.py` (Dòng 25–58). Đây là "cửa ngõ" nạp toàn bộ cấu trúc đồ thị vào bộ nhớ để phục vụ cho các thuật toán PageRank tiếp theo.

### 4.1. Quy tắc bắt buộc của đề bài
1. **Bỏ qua dòng chú thích:** Bỏ các dòng trống và dòng bắt đầu bằng ký tự `#` trong cả hai tệp TSV.
2. **Tập đỉnh từ `articles.tsv`:** Phải chứa đủ tất cả các bài viết, kể cả những bài cô lập không có liên kết nào.
3. **Giữ nguyên mã hóa percent-encoding:** Tuyệt đối không giải mã URL (ví dụ: giữ nguyên `%C3%89douard_Manet`, không chuyển thành `Édouard_Manet`).
4. **Sắp xếp theo thứ tự byte UTF-8:** Sắp xếp ID theo thứ tự từ điển của các byte UTF-8 (`article.encode("utf-8")`).
5. **Cạnh có hướng từ `links.tsv`:** `source <TAB> target`. Phải gộp cạnh trùng, giữ cạnh tự nối (self-loop).
6. **Bắt lỗi `ValueError`:**
   - Nếu đồ thị không có đỉnh nào (tệp rỗng hoặc chỉ có chú thích).
   - Nếu trong `links.tsv` xuất hiện bài viết nào không có tên trong `articles.tsv`.

---

### 4.2. Toàn bộ mã nguồn hàm `load_graph`

```python
def load_graph(
    articles_path: Path, links_path: Path
) -> tuple[list[str], list[list[int]]]:
    # Bước 1: Đọc và sắp xếp tập đỉnh theo byte UTF-8
    with articles_path.open(encoding="utf-8") as handle:
        ids = sorted(
            {line.rstrip("\r\n") for line in handle
             if line.strip() and not line.startswith("#")},
            key=lambda article: article.encode("utf-8"),
        )
    if not ids:
        raise ValueError("Đồ thị không có đỉnh")

    # Bước 2: Tạo từ điển chỉ số và khởi tạo danh sách kề
    index = {article: i for i, article in enumerate(ids)}
    neighbors = [set() for _ in ids]

    # Bước 3: Đọc và kiểm tra từng cạnh có hướng
    with links_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            source, target = line.rstrip("\r\n").split("\t")
            if source not in index or target not in index:
                raise ValueError(f"Cạnh chứa ID không tồn tại: {source}, {target}")
            neighbors[index[source]].add(index[target])

    # Bước 4: Chuyển sang danh sách đã sắp xếp và trả về
    return ids, [sorted(destinations) for destinations in neighbors]
```

---

### 4.3. Bóc tách và giải thích chi tiết từng dòng code

#### 🟢 Bước 1: Đọc và sắp xếp tập đỉnh từ `articles.tsv`

```python
    with articles_path.open(encoding="utf-8") as handle:
        ids = sorted(
            {line.rstrip("\r\n") for line in handle
             if line.strip() and not line.startswith("#")},
            key=lambda article: article.encode("utf-8"),
        )
```
- **`with articles_path.open(encoding="utf-8") as handle:`**
  - Mở file an toàn với context manager `with`, tự động đóng file sau khi đọc xong để tránh rò rỉ tài nguyên hệ điều hành.
  - Chỉ định rõ bộ mã `encoding="utf-8"` để đọc đúng tên các bài viết chứa ký tự đa ngôn ngữ.
- **`line.rstrip("\r\n")` vs `line.strip()`:**
  - `line.rstrip("\r\n")`: Chỉ cắt bỏ ký tự xuống dòng ở cuối dòng (hỗ trợ cả Windows `\r\n` và Linux `\n`). Không dùng `.strip()` ở đây để tránh vô tình cắt mất khoảng trắng có chủ đích nếu có trong tên bài.
  - `if line.strip() and not line.startswith("#")`: Kiểm tra nhanh: nếu dòng không rỗng (`line.strip()`) và không phải dòng chú thích `#` thì mới lấy.
- **Cú pháp `{...}` (Set Comprehension):**
  - Gom các tên bài đọc được vào một tập hợp `set` để loại bỏ các dòng bị trùng lặp nếu có trong `articles.tsv`.
- **`key=lambda article: article.encode("utf-8")` trong `sorted(...)` (ĐIỂM THEN CHỐT):**
  - Trong Python, nếu bạn chỉ gọi `sorted(ids)`, Python sẽ so sánh theo **Unicode code point** (ký tự).
  - Đề bài yêu cầu: *Sắp xếp theo thứ tự từ điển của byte UTF-8*.
  - Hàm `article.encode("utf-8")` chuyển chuỗi văn bản thành mảng các byte nhị phân thô (`b'...'`). Khi đó hàm `sorted` sẽ so sánh trực tiếp giá trị số của từng byte. Điều này đảm bảo 100% khớp với nghiệm của bộ chấm chính thức đối với các tên bài có mã hóa percent URL như `%C3%89douard_Manet`.

```python
    if not ids:
        raise ValueError("Đồ thị không có đỉnh")
```
- Nếu sau khi lọc chú thích mà tập `ids` rỗng, ta ném ngoại lệ `ValueError` đúng theo yêu cầu bắt buộc của đề bài.

---

#### 🟢 Bước 2: Tạo từ điển ánh xạ và danh sách kề

```python
    index = {article: i for i, article in enumerate(ids)}
    neighbors = [set() for _ in ids]
```
- **`index = {article: i for i, article in enumerate(ids)}`:**
  - Tạo một bảng băm (Hash Map / Dictionary) ánh xạ: `Tên_bài_viết -> Số_thứ_tự_nguyên (0 đến n-1)`.
  - *Ví dụ:* `{"10th_century": 0, "11th_century": 1, ..., "Zebra": 4603}`.
  - **Tại sao cực kỳ quan trọng về hiệu năng?**
    - Khi đọc 120,000 cạnh, nếu ta dùng `ids.index(source)` để tìm chỉ số đỉnh, mỗi lần tìm sẽ mất $O(n)$ thời gian $\implies 120,000 \times 4604 \approx 5.5 \times 10^8$ phép tính (chương trình sẽ bị đơ vài chục giây!).
    - Nhờ từ điển `index[source]`, việc tra cứu chỉ số đỉnh diễn ra tức thì với độ phức tạp $O(1)$!
- **`neighbors = [set() for _ in ids]`:**
  - Khởi tạo danh sách gồm $n$ tập hợp rỗng (`set()`), mỗi tập hợp đại diện cho danh sách các đỉnh đích mà đỉnh đó trỏ tới.
  - Sử dụng `set` cho phép ta gọi lệnh `.add(target)`: nếu một liên kết bị lặp lại nhiều lần trong file `links.tsv`, `set` sẽ tự động chỉ giữ lại 1 bản sao $\implies$ **Tự động gộp cạnh trùng lặp**.

---

#### 🟢 Bước 3: Đọc và kiểm tra các liên kết từ `links.tsv`

```python
    with links_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            source, target = line.rstrip("\r\n").split("\t")
            if source not in index or target not in index:
                raise ValueError(f"Cạnh chứa ID không tồn tại: {source}, {target}")
            neighbors[index[source]].add(index[target])
```
- **`source, target = line.rstrip("\r\n").split("\t")`:**
  - Tách mỗi dòng của `links.tsv` theo ký tự tab `\t` thành đỉnh nguồn `source` và đỉnh đích `target`.
- **`if source not in index or target not in index:`**
  - Kiểm tra xem 2 bài viết này có nằm trong danh sách bài viết hợp lệ từ `articles.tsv` không. Nếu phát hiện một bài viết lạ chưa từng được khai báo, ném ngay ngoại lệ `ValueError`.
- **`neighbors[index[source]].add(index[target])`:**
  - Lấy chỉ số số nguyên của `source` thông qua `index[source]`.
  - Thêm chỉ số số nguyên của `target` vào tập hợp kề của đỉnh nguồn.
  - *Xử lý cạnh tự nối (Self-loops):* Nếu `source == target`, chỉ số đỉnh đích chính là chỉ số đỉnh nguồn $\implies$ Cạnh tự nối này vẫn được thêm vào tập hợp bình thường, không bị loại bỏ, hoàn toàn đúng quy ước đề bài.

---

#### 🟢 Bước 4: Chuyển đổi và trả về kết quả

```python
    return ids, [sorted(destinations) for destinations in neighbors]
```
- **`[sorted(destinations) for destinations in neighbors]`:**
  - Chuyển từng tập hợp `set` thành danh sách `list[int]` đã được sắp xếp tăng dần theo chỉ số đỉnh đích.
  - *Xử lý nút cụt (Dead ends):* Nếu đỉnh $j$ không có đường link nào trỏ ra ngoài, tập `neighbors[j]` vẫn là rỗng `set()` $\implies$ Khi chuyển đổi sẽ trở thành danh sách rỗng `[]`.
- **Giá trị trả về:**
  - `ids`: `list[str]` chứa đúng 4,604 bài viết đã sắp xếp chuẩn UTF-8.
  - `out_neighbors`: `list[list[int]]` chứa đúng 4,604 danh sách con biểu diễn các cạnh ra.

---

### 4.4. Ví dụ minh họa quá trình chạy thực tế

Giả sử ta có một đồ thị thu nhỏ gồm 4 bài viết:
1. `articles.tsv`:
   ```tsv
   Zebra
   Apple
   Cat
   Dog
   ```
2. `links.tsv`:
   ```tsv
   # Cạnh có hướng
   Apple	Cat
   Cat	Apple
   Apple	Cat       # Cạnh lặp trùng!
   Cat	Cat         # Cạnh tự nối (Self-loop)!
   Apple	Zebra
   # Chú ý: Bài "Dog" không có link nào trỏ ra ngoài (Nút cụt!)
   ```

**Diễn biến xử lý của hàm `load_graph`:**
1. **Sắp xếp `ids` theo byte UTF-8:**
   `ids = ["Apple", "Cat", "Dog", "Zebra"]`
2. **Từ điển chỉ số `index`:**
   `{"Apple": 0, "Cat": 1, "Dog": 2, "Zebra": 3}`
3. **Quá trình nạp các cạnh vào `neighbors`:**
   - Đọc `Apple -> Cat` (0 -> 1) $\implies$ `neighbors[0].add(1)`.
   - Đọc `Cat -> Apple` (1 -> 0) $\implies$ `neighbors[1].add(0)`.
   - Đọc `Apple -> Cat` (0 -> 1) $\implies$ 1 đã có trong `neighbors[0]`, tự động bỏ qua!
   - Đọc `Cat -> Cat` (1 -> 1) $\implies$ `neighbors[1].add(1)` (Giữ cạnh tự nối!).
   - Đọc `Apple -> Zebra` (0 -> 3) $\implies$ `neighbors[0].add(3)`.
   - Bài `Dog` (chỉ số 2) không có cạnh nào $\implies$ `neighbors[2]` vẫn là `set()`.
4. **Kết quả trả về:**
   - `ids = ["Apple", "Cat", "Dog", "Zebra"]`
   - `out_neighbors = [[1, 3], [0, 1], [], []]` (Đỉnh 2 và 3 là các nút cụt).
