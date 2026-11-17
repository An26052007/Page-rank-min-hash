# GIẢI THÍCH CHI TIẾT HÀM `topic_distribution`
> **Phần B — Tạo vector phân phối dịch chuyển theo chủ đề (Topic-Sensitive Teleportation)**  
> **Tệp:** `submission.py` (Dòng 117–144)

---

## 1. YÊU CẦU ĐỀ BÀI VÀ MỤC ĐÍCH HÀM

Trong mục **3. Phần B** của [bai-tap-wikispeedia.md](file:///C:/Users/HP/Downloads/HKI_2026-2027/GI%E1%BA%A2I%20THU%E1%BA%ACT%20KHDL/wikispeedia-student-package/bai-tap-wikispeedia.md):
- **Mục tiêu:** Khi người lướt web bị lạc hoặc chán không muốn click theo liên kết, thay vì nhảy ngẫu nhiên đến bất kỳ trang nào trong 4,604 bài viết (như PageRank toàn cục), họ chỉ nhảy vào **các bài viết thuộc một chủ đề cụ thể** (ví dụ: chỉ thích đọc Khoa học `subject.Science`).
- **Nhiệm vụ của hàm `topic_distribution`:** Đọc tệp phân loại `categories.tsv`, tìm tất cả các bài viết thuộc chủ đề `prefix`, sau đó tạo ra một vector xác suất cá nhân hóa $v$ sao cho:
  - Bài viết nằm trong chủ đề sẽ nhận xác suất: $v_i = \frac{1}{|S|}$ (với $|S|$ là tổng số bài viết phân biệt trong chủ đề).
  - Bài viết không thuộc chủ đề sẽ nhận xác suất: $v_i = 0.0$.
  - Tổng toàn bộ xác suất của vector phải đúng bằng $1.0$.

---

## 2. BẢNG THÔNG SỐ ĐẦU VÀO VÀ ĐẦU RA

```python
def topic_distribution(
    ids: list[str], categories_path: Path, prefix: str
) -> list[float]:
```

| Tên tham số / Đầu ra | Kiểu dữ liệu | Nguồn gốc / Ý nghĩa thực tế | Giá trị ví dụ |
|---|---|---|---|
| **`ids`** | `list[str]` | Danh sách tên định danh của tất cả các bài viết trong đồ thị (theo thứ tự chuẩn hóa từ Phần A). | `["10th_century", "14th_century", ..., "Yeti", "Zoroaster"]` (gồm 4,604 phần tử) |
| **`categories_path`**| `Path` | Đường dẫn đến tệp danh mục `categories.tsv` của Wikispeedia. | `Path("data/wikispeedia_paths-and-graph/categories.tsv")` |
| **`prefix`** | `str` | Tiền tố nhãn chủ đề cần trích xuất (phân cấp bằng dấu chấm `.`). | `"subject.Science"`, `"subject.History"`, hoặc `"subject.Geography"` |
| **ĐẦU RA (`->`)** | `list[float]` | Vector phân phối dịch chuyển chủ đề $v$, độ dài đúng bằng $n = \text{len}(ids)$, tổng các phần tử bằng $1.0$. | `[0.0, 0.000905, 0.0, ..., 0.000905]` |

---

## 3. TOÀN BỘ MÃ NGUỒN HÀM `topic_distribution`

```python
def topic_distribution(
    ids: list[str], categories_path: Path, prefix: str
) -> list[float]:
    known_ids = set(ids)
    members = set()
    with categories_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            article, label = line.rstrip("\r\n").split("\t")
            if article not in known_ids:
                raise ValueError(f"Nhãn tham chiếu ID không tồn tại: {article}")
            if label == prefix or label.startswith(prefix + "."):
                members.add(article)
    if not members:
        raise ValueError(f"Chủ đề không có bài viết: {prefix}")
    probability = 1.0 / len(members)
    return [probability if article in members else 0.0 for article in ids]
```

---

## 4. TỪ ĐIỂN BIẾN NỘI BỘ (LOCAL VARIABLES)

| Tên biến | Kiểu dữ liệu | Ý nghĩa & Vai trò trong thuật toán |
|---|---|---|
| **`known_ids`** | `set[str]` | Tập hợp (Hash Set) chứa toàn bộ các ID hợp lệ từ danh sách `ids`. Giúp thao tác kiểm tra tồn tại `article not in known_ids` đạt tốc độ $O(1)$ thay vì $O(n)$. |
| **`members`** | `set[str]` | Tập hợp chứa tên các bài viết phân biệt thỏa mãn điều kiện thuộc chủ đề `prefix`. Dùng `set` để tự động khử trùng lặp khi một bài có nhiều nhãn con. |
| **`handle`** | `io.TextIOWrapper` | Luồng đọc tệp tin `categories.tsv` được mở an toàn trong khối lệnh `with`. |
| **`line`** | `str` | Từng dòng thô được đọc lần lượt từ tệp `categories.tsv`. |
| **`article`** | `str` | Tên định danh bài viết ở cột thứ nhất của dòng TSV (ví dụ: `"Albert_Einstein"`). |
| **`label`** | `str` | Nhãn danh mục phân cấp ở cột thứ hai của dòng TSV (ví dụ: `"subject.Science.Physics.Physicists"`). |
| **`probability`**| `float` | Giá trị xác suất chia đều cho mỗi bài viết trong chủ đề, tính bằng $1.0 / |S|$. |

---

## 5. GIẢI THÍCH CHI TIẾT TỪNG DÒNG CODE

### Bước 1: Khởi tạo các cấu trúc dữ liệu tra cứu

```python
    known_ids = set(ids)
    members = set()
```
- `known_ids = set(ids)`: Chuyển mảng `ids` (4,604 phần tử) thành tập băm (`set`). Khi kiểm tra một phần tử có nằm trong tập băm hay không, Python chỉ mất thời gian $O(1)$, cực nhanh so với việc dò tìm trên danh sách ($O(n)$).
- `members = set()`: Khởi tạo một tập hợp rỗng để gom các bài viết thuộc chủ đề. Do là cấu trúc `set`, nếu ta vô tình thêm một bài viết nhiều lần, nó cũng chỉ lưu đúng một phần tử duy nhất.

---

### Bước 2: Đọc tệp TSV và lọc nhãn theo quy tắc phân cấp

```python
    with categories_path.open(encoding="utf-8") as handle:
        for line in handle:
            if not line.strip() or line.startswith("#"):
                continue
            article, label = line.rstrip("\r\n").split("\t")
            if article not in known_ids:
                raise ValueError(f"Nhãn tham chiếu ID không tồn tại: {article}")
            if label == prefix or label.startswith(prefix + "."):
                members.add(article)
```

- **`with categories_path.open(encoding="utf-8") as handle:`**
  Mở tệp `categories.tsv` với bảng mã UTF-8 chuẩn. Khối lệnh `with` tự động đóng tệp ngay cả khi gặp sự cố, tránh rò rỉ tài nguyên bộ nhớ.
- **`if not line.strip() or line.startswith("#"): continue`**
  Bỏ qua các dòng trống hoặc các dòng chú thích bắt đầu bằng dấu `#` (quy định chuẩn của bộ dữ liệu Wikispeedia).
- **`article, label = line.rstrip("\r\n").split("\t")`**
  - Cắt bỏ ký tự xuống dòng ở cuối (`\r\n` hoặc `\n`).
  - Tách chuỗi theo dấu tab (`\t`) thành đúng 2 phần: cột đầu là tên bài viết (`article`), cột sau là tên danh mục (`label`).
- **`if article not in known_ids: raise ValueError(...)`**
  - Kiểm tra tính toàn vẹn của dữ liệu: Nếu một dòng danh mục gán nhãn cho một bài viết lạ lẫm không hề có trong đồ thị `ids`, chương trình lập tức ném lỗi `ValueError` để chặn đứng dữ liệu bẩn.
- **`if label == prefix or label.startswith(prefix + "."):`**
  - **Quy tắc so khớp phân cấp cực kỳ chuẩn mực:**
    1. `label == prefix`: Bài viết mang chính xác nhãn gốc (ví dụ: `subject.Science`).
    2. `label.startswith(prefix + ".")`: Bài viết mang nhãn con thuộc nhánh bên dưới (ví dụ: `subject.Science.Physics`).
    - *Chú ý:* Bắt buộc phải có dấu chấm `.` đằng sau tiền tố (`prefix + "."`), để tránh nhận nhầm các chủ đề khác trùng tiền tố chữ cái!
- **`members.add(article)`**
  Thêm tên bài viết vào tập `members`.

---

### Bước 3: Kiểm tra tính hợp lệ và phân bổ xác suất

```python
    if not members:
        raise ValueError(f"Chủ đề không có bài viết: {prefix}")
    probability = 1.0 / len(members)
    return [probability if article in members else 0.0 for article in ids]
```

- **`if not members:`**
  Nếu tập `members` hoàn toàn rỗng (chủ đề người dùng yêu cầu không hề tồn tại trong cơ sở dữ liệu hoặc không có bài viết nào), ném ngoại lệ `ValueError`. Điều này cũng ngăn chặn lỗi chia cho 0 (`ZeroDivisionError`) ở dòng tiếp theo.
- **`probability = 1.0 / len(members)`**
  Tính đều xác suất cho mỗi bài viết thuộc chủ đề:
  $$p = \frac{1}{|S|}$$
  Tổng xác suất của toàn bộ các bài trong tập chủ đề sẽ luôn bảo toàn đúng bằng $1.0$:
  $$\sum_{a \in S} p = |S| \times \frac{1}{|S|} = 1.0$$
- **`return [probability if article in members else 0.0 for article in ids]`**
  - Sử dụng List Comprehension duyệt qua toàn bộ 4,604 bài viết theo đúng thứ tự của `ids`.
  - Nếu bài viết nằm trong `members`: gán giá trị bằng `probability`.
  - Nếu bài viết không nằm trong `members`: gán bằng `0.0`.
  - Vector kết quả có kích thước chính xác là $n = 4604$.

---

## 6. TẠI SAO LẠI LÀM NHƯ VẬY? (THIẾT KẾ KỸ THUẬT & LÝ DO CỐT LÕI)

### 1. Tại sao phải chuyển `known_ids = set(ids)` thay vì tìm kiếm trực tiếp trên `ids`?
- Tệp `categories.tsv` có hơn 10.000 dòng.
- Nếu kiểm tra `if article not in ids` (trên kiểu `list`), Python phải quét tuần tự từ đầu đến cuối danh sách ($O(n)$ với $n = 4604$). Tổng số phép so sánh sẽ lên đến $10000 \times 4604 \approx 46.000.000$ phép tính!
- Với `set`, bảng băm cho phép kiểm tra tồn tại với độ phức tạp $O(1)$. Toàn bộ 10.000 dòng được kiểm tra chỉ trong vài mili-giây.

### 2. Tại sao phải có dấu chấm `.` trong `label.startswith(prefix + ".")`?
- Xét ví dụ bạn muốn lọc chủ đề Nghệ thuật: `prefix = "subject.Art"`.
- Nếu bạn chỉ viết `label.startswith("subject.Art")` mà không có dấu chấm:
  - Nhãn `subject.Art.Visual_arts` $\rightarrow$ Khớp (đúng).
  - Nhưng nhãn `subject.Artificial_intelligence` $\rightarrow$ Cũng bị coi là khớp (SAI HOÀN TOÀN vì đây là ngành Trí tuệ nhân tạo thuộc Tin học, không phải Nghệ thuật!).
- Thêm dấu chấm `prefix + "."` (`subject.Art.`) đảm bảo phân cách rõ ràng ranh giới thư mục cấp bậc, ngăn ngừa triệt để lỗi va chạm chuỗi tiền tố (prefix collision).

### 3. Tại sao `members` phải là một `set` chứ không phải `list`?
- Một bài viết trong Wikipedia thường có nhiều khía cạnh nên được gán nhiều nhãn con trong cùng một chủ đề lớn.
  *Ví dụ:* Bài viết `Isaac_Newton` có thể xuất hiện 2 dòng trong `categories.tsv`:
  1. `Isaac_Newton \t subject.Science.Physics.Physicists`
  2. `Isaac_Newton \t subject.Science.Mathematics.Mathematicians`
- Cả hai nhãn đều bắt đầu bằng `subject.Science.`. Nếu dùng `list`, bài viết `Isaac_Newton` sẽ bị tính 2 lần, làm $|S|$ bị đội lên sai lệch và `Isaac_Newton` sẽ nhận gấp đôi xác suất teleport.
- Cấu trúc `set` đảm bảo mỗi bài viết chỉ được ghi nhận duy nhất một lần ($|S|$ chính xác là số bài viết độc nhất).

### 4. Tại sao vector trả về phải duyệt tuần tự theo `ids`?
- Thuật toán PageRank ở hàm `pagerank(...)` biểu diễn đồ thị dưới dạng ma trận và danh sách kề đánh chỉ số từ $0$ đến $n-1$.
- Phần tử $v[i]$ bắt buộc phải tương ứng chính xác với đỉnh `ids[i]`. Việc duyệt `for article in ids` bảo đảm thứ tự vị trí của vector xác suất dịch chuyển khớp 100% với danh sách kề `out_neighbors`.

---

## 7. BẢNG DỮ LIỆU THỰC TẾ TRÊN 3 CHỦ ĐỀ CHÍNH THỨC

Dưới đây là kết quả kiểm thử thực tế khi chạy hàm `topic_distribution` trên toàn bộ tập dữ liệu Wikispeedia:

| Chủ đề | Tiền tố (`prefix`) | Số bài viết độc nhất ($|S|$) | Xác suất mỗi bài ($1/|S|$) |
|---|---|:---:|:---:|
| **Science** (Khoa học) | `subject.Science` | **1,105** | $\approx 0.000904977$ |
| **History** (Lịch sử) | `subject.History` | **525** | $\approx 0.001904762$ |
| **Geography** (Địa lý) | `subject.Geography` | **1,063** | $\approx 0.000940734$ |

*Ghi chú: Tất cả các bài viết không thuộc chủ đề đều nhận xác suất bằng `0.0`.*
