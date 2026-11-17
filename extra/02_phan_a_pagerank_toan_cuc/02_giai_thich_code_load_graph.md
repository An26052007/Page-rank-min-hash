# GIẢI THÍCH CHI TIẾT HÀM `load_graph`
> **Phần A — Nạp và tiền xử lý đồ thị liên kết**  
> **Tệp mã nguồn:** `submission.py` (Dòng 25–58)

---

## 1. BẢNG THAM SỐ ĐẦU VÀO VÀ ĐẦU RA (INPUT & OUTPUT)

```python
def load_graph(
    articles_path: Path, links_path: Path
) -> tuple[list[str], list[list[int]]]
```

| Tên tham số | Kiểu dữ liệu | Nguồn truyền vào | Ý nghĩa thực tế | Ví dụ giá trị |
|---|---|---|---|---|
| **`articles_path`** | `Path` (từ thư viện `pathlib`) | Hàm `main()` truyền vào từ đường dẫn `data/wikispeedia_paths-and-graph/articles.tsv` | Đường dẫn tệp chứa danh sách toàn bộ 4,604 bài viết Wikipedia. | `Path("data/wikispeedia_paths-and-graph/articles.tsv")` |
| **`links_path`** | `Path` (từ thư viện `pathlib`) | Hàm `main()` truyền vào từ đường dẫn `data/wikispeedia_paths-and-graph/links.tsv` | Đường dẫn tệp chứa danh sách tất cả các liên kết có hướng giữa các bài. | `Path("data/wikispeedia_paths-and-graph/links.tsv")` |

* **ĐẦU RA (Return):** Trả về một Tuple gồm đúng 2 phần tử `(ids, out_neighbors)`:
  1. **`ids` (`list[str]`):** Danh sách gồm đúng 4,604 chuỗi tên bài viết, đã sắp xếp theo thứ tự từ điển của byte UTF-8.
  2. **`out_neighbors` (`list[list[int]]`):** Danh sách 2 chiều gồm 4,604 danh sách con. Phần tử thứ $j$ chứa các chỉ số nguyên của những bài viết mà bài viết `ids[j]` đặt link trỏ tới.

---

## 2. BẢNG TỪ ĐIỂN CÁC BIẾN NỘI BỘ (VARIABLE DICTIONARY)

| Tên biến | Kiểu dữ liệu | Ý nghĩa và vai trò trong thuật toán |
|---|---|---|
| **`handle`** | `TextIOWrapper` | Đối tượng file được mở bằng context manager `with`, đại diện cho luồng đọc dữ liệu từ ổ cứng. |
| **`ids`** | `list[str]` | Danh sách chứa toàn bộ tên các bài viết duy nhất (đã loại bỏ trùng lặp và sắp xếp chuẩn UTF-8). |
| **`index`** | `dict[str, int]` | Bảng băm ánh xạ từ `tên_bài_viết -> chỉ_số_nguyên (0 đến 4603)` để tra cứu cực nhanh trong thời gian $O(1)$. |
| **`neighbors`** | `list[set[int]]` | Mảng gồm 4,604 tập hợp `set()`. Mỗi tập hợp chứa các chỉ số đỉnh đích của một bài viết (dùng `set` để tự động gộp cạnh trùng). |
| **`line`** | `str` | Chuỗi ký tự của từng dòng đọc được từ tệp TSV. |
| **`source`** | `str` | Tên bài viết nguồn (nơi đặt liên kết) đọc từ cột 1 của `links.tsv`. |
| **`target`** | `str` | Tên bài viết đích (nơi được liên kết trỏ tới) đọc từ cột 2 của `links.tsv`. |
| **`destinations`** | `set[int]` | Tập hợp các đỉnh đích của một bài viết, sau đó được hàm `sorted()` chuyển thành `list[int]`. |

---

## 3. TOÀN BỘ MÃ NGUỒN

```python
def load_graph(
    articles_path: Path, links_path: Path
) -> tuple[list[str], list[list[int]]]:
    with articles_path.open(encoding="utf-8") as handle:
        ids = sorted(
            {line.rstrip("\r\n") for line in handle
             if line.strip() and not line.startswith("#")},
            key=lambda article: article.encode("utf-8"),
        )
    if not ids:
        raise ValueError("Đồ thị không có đỉnh")

    index = {article: i for i, article in enumerate(ids)}
    neighbors = [set() for _ in ids]
    with links_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            source, target = line.rstrip("\r\n").split("\t")
            if source not in index or target not in index:
                raise ValueError(f"Cạnh chứa ID không tồn tại: {source}, {target}")
            neighbors[index[source]].add(index[target])

    return ids, [sorted(destinations) for destinations in neighbors]
```

---

## 4. BÓC TÁCH TỪNG BƯỚC NHỎ VÀ GIẢI THÍCH "TẠI SAO LẠI LÀM NHƯ VẬY?"

### 🟢 Bước 1: Đọc và sắp xếp tập đỉnh từ `articles.tsv`

```python
    with articles_path.open(encoding="utf-8") as handle:
        ids = sorted(
            {line.rstrip("\r\n") for line in handle
             if line.strip() and not line.startswith("#")},
            key=lambda article: article.encode("utf-8"),
        )
```

* **Dòng lệnh này làm gì?**
  Mở tệp `articles.tsv`, duyệt qua từng dòng, bỏ các dòng trống hoặc dòng bắt đầu bằng dấu `#`, gom tên bài viết vào một tập hợp `set` để loại bỏ tên trùng lặp, sau đó sắp xếp theo thứ tự byte UTF-8.
* **Tại sao lại làm như vậy?**
  1. *Tại sao dùng `line.rstrip("\r\n")` thay vì `line.strip()`?*  
     Vì hàm `.strip()` sẽ cắt bỏ cả khoảng trắng ở đầu và cuối tên bài nếu có. Dùng `rstrip("\r\n")` chỉ cắt bỏ ký tự xuống dòng của Windows/Linux mà giữ nguyên vẹn 100% tên bài viết.
  2. *Tại sao bắt buộc phải có `key=lambda article: article.encode("utf-8")`?*  
     Trong Python, hàm `sorted()` mặc định so sánh theo ký tự Unicode. Nhưng đề bài đặc tả: **Sắp xếp theo thứ tự từ điển của byte UTF-8**. Ví dụ ký tự mã hóa percent URL như `%C3%89` (`É`) nếu so sánh theo ký tự sẽ cho thứ tự khác với so sánh theo từng byte nhị phân thô `0x25, 0xC3, 0x89`. Câu lệnh `article.encode("utf-8")` ép so sánh mảng byte, đảm bảo khớp 100% với barem của trường.

```python
    if not ids:
        raise ValueError("Đồ thị không có đỉnh")
```
* **Dòng lệnh này làm gì?** Kiểm tra nếu danh sách đỉnh rỗng thì lập tức ném lỗi `ValueError`.
* **Tại sao lại làm như vậy?** Để ngăn chặn việc chạy thuật toán trên một đồ thị rỗng không có nút nào, bảo vệ chương trình không bị lỗi chia cho 0 ở các bước sau.

---

### 🟢 Bước 2: Tạo từ điển ánh xạ chỉ số và cấu trúc danh sách kề

```python
    index = {article: i for i, article in enumerate(ids)}
    neighbors = [set() for _ in ids]
```

* **Dòng lệnh này làm gì?**
  - Tạo từ điển `index` biến tên bài viết thành số nguyên: `{"10th_century": 0, "11th_century": 1, ..., "Zebra": 4603}`.
  - Tạo mảng `neighbors` gồm 4,604 tập hợp rỗng `set()`.
* **Tại sao lại làm như vậy? (Tối ưu hóa sống còn):**
  - *Tại sao không dùng `ids.index(source)`?*  
    Trong Python, `ids.index()` là phép quét tuần tự từ đầu đến cuối danh sách mất thời gian $O(n)$. Với gần 120,000 cạnh, nếu mỗi cạnh đều gọi `ids.index()`, máy tính phải thực hiện $120,000 \times 4604 \approx 5.5 \times 10^8$ phép tính (bị đơ mất vài chục giây!). Dùng từ điển `index[source]` tra cứu bằng bảng băm chỉ tốn đúng thời gian $O(1)$ (tức thì!).
  - *Tại sao dùng `set()` cho từng đỉnh đích?*  
    Trong tệp `links.tsv`, nhiều bài viết có thể chứa nhiều liên kết trùng nhau trỏ tới cùng một bài đích ($A \to B$ xuất hiện 2 lần). Bản chất của cấu trúc dữ liệu `set` trong Python là không bao giờ chứa phần tử trùng lặp, nên lệnh `.add()` sẽ **tự động gộp cạnh trùng lặp** mà không tốn công kiểm tra thủ công.

---

### 🟢 Bước 3: Đọc và kiểm tra các liên kết từ `links.tsv`

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

* **Dòng lệnh này làm gì?**
  - Duyệt từng dòng của `links.tsv`, bỏ qua dòng chú thích `#`.
  - Tách theo ký tự tab `\t` thành đỉnh nguồn `source` và đỉnh đích `target`.
  - Kiểm tra xem cả 2 đỉnh có nằm trong `index` (tập đỉnh hợp lệ) không. Nếu có đỉnh lạ, ném ngay lỗi `ValueError`.
  - Lấy chỉ số số nguyên và đưa đỉnh đích vào tập kề của đỉnh nguồn: `neighbors[index[source]].add(index[target])`.
* **Tại sao lại làm như vậy?**
  - *Xử lý cạnh tự nối (Self-loops):* Nếu một bài viết tự trỏ vào chính nó (`source == target`), lệnh `add` vẫn thêm chỉ số đó vào bình thường $\implies$ Giữ nguyên cạnh tự nối đúng theo quy ước đề bài.

---

### 🟢 Bước 4: Sắp xếp danh sách kề và trả về kết quả

```python
    return ids, [sorted(destinations) for destinations in neighbors]
```

* **Dòng lệnh này làm gì?**
  Chuyển từng tập hợp `set` thành danh sách `list[int]` đã sắp xếp tăng dần, rồi trả về bộ đôi `(ids, out_neighbors)`.
* **Tại sao lại làm như vậy?**
  - Bộ chấm kiểm tra thứ tự của các đỉnh đích trong danh sách kề phải được sắp xếp tăng dần.
  - *Xử lý nút cụt (Dead ends):* Nếu bài viết thứ $j$ không có bất kỳ liên kết ra nào, `neighbors[j]` là tập rỗng `set()` $\implies$ Khi chuyển đổi thành `sorted(destinations)` sẽ là danh sách rỗng `[]`. Đây chính là cách nhận diện nút cụt trong thuật toán PageRank tiếp theo.
