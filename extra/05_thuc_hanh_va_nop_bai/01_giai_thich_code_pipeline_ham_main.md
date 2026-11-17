# GIẢI THÍCH CHI TIẾT HÀM `main` VÀ QUY TRÌNH XUẤT 4 TỆP CSV
> **Phần Điều phối hệ thống (Pipeline Orchestration & Execution)**  
> **Tệp:** `submission.py` (Dòng 261–374)

---

## 1. YÊU CẦU ĐỀ BÀI VÀ QUY CÁCH ĐẦU RA

Trong mục **5. Dùng template, tự chấm và nộp bài** của [bai-tap-wikispeedia.md](file:///C:/Users/HP/Downloads/HKI_2026-2027/GI%E1%BA%A2I%20THU%E1%BA%ACT%20KHDL/wikispeedia-student-package/bai-tap-wikispeedia.md):
- **Vai trò của hàm `main()`:** Là "nhạc trưởng" điều phối toàn bộ luồng xử lý từ đầu đến cuối: Nạp đồ thị $\to$ Chạy PageRank toàn cục $\to$ Chạy PageRank theo chủ đề $\to$ Tổ hợp tuyến tính $\to$ Băm shingle và MinHash $\to$ So sánh Jaccard $\to$ Xuất dữ liệu ra 4 tệp CSV chuẩn.
- **Yêu cầu kỹ thuật bắt buộc:**
  1. Hỗ trợ giao diện dòng lệnh CLI linh hoạt thông qua thư viện `argparse`.
  2. Định dạng số thực xuất ra CSV phải dùng `format(val, ".17g")` (chuẩn 17 chữ số có nghĩa của IEEE 754 float64, dùng dấu chấm thập phân).
  3. Xuất đúng 4 tệp kết quả vào thư mục `--output-dir` (mặc định là `output/`).
  4. Đặt trong khối bảo vệ `if __name__ == "__main__": main()` để khi bộ tự chấm `autograder.py` nạp module chấm từng hàm lẻ thì không tự ý chạy lại toàn bộ pipeline gây lãng phí thời gian.

---

## 2. BẢNG THAM SỐ DÒNG LỆNH CLI (ARGPARSE)

```python
def main() -> None:
```

Khi chạy từ terminal: `python submission.py [các_cờ_tùy_chọn]`

| Cờ tham số (Flag) | Kiểu dữ liệu | Giá trị mặc định | Ý nghĩa thực tế |
|---|---|---|---|
| **`--data-dir`** | `Path` | `Path("data")` | Thư mục chứa dữ liệu thô giải nén của Wikispeedia (`wikispeedia_paths-and-graph` và `plaintext_articles`). |
| **`--output-dir`** | `Path` | `Path("output")` | Thư mục đích dùng để lưu 4 tệp kết quả CSV xuất ra. |
| **`--hash-params`** | `Path` | `Path("assignment/minhash_params.json")` | Đường dẫn đến tệp JSON chứa số nguyên tố $p$ và 128 hàm băm MinHash. |
| **`--pairs`** | `Path` | `Path("data/pairs.csv")` | Tệp CSV chứa danh sách 100 cặp bài viết cần so sánh độ tương đồng Jaccard. |
| **`--beta`** | `float` | `0.85` | Hệ số ngẫu nhiên lướt web (damping factor) của PageRank. |
| **`--tol`** | `float` | `1e-10` | Ngưỡng sai số hội tụ dừng theo chuẩn khoảng cách $L_1$. |
| **`--max-iter`** | `int` | `1000` | Số vòng lặp tối đa cho phép thuật toán PageRank thực thi. |

---

## 3. TOÀN BỘ MÃ NGUỒN HÀM `main`

```python
def main() -> None:
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--data-dir", type=Path, default=Path("data"))
    parser.add_argument("--output-dir", type=Path, default=Path("output"))
    parser.add_argument(
        "--hash-params", type=Path, default=Path("assignment/minhash_params.json")
    )
    parser.add_argument(
        "--pairs", type=Path, default=Path("data/pairs.csv"),
        help="CSV có header article_id_a,article_id_b; mặc định dùng bộ cặp Wikispeedia chính thức.",
    )
    parser.add_argument("--beta", type=float, default=0.85)
    parser.add_argument("--tol", type=float, default=1e-10)
    parser.add_argument("--max-iter", type=int, default=1000)
    args = parser.parse_args()

    graph_dir = args.data_dir / "wikispeedia_paths-and-graph"
    text_dir = args.data_dir / "plaintext_articles"
    with args.hash_params.open(encoding="utf-8") as handle:
        hash_params = json.load(handle)
    with args.pairs.open(encoding="utf-8", newline="") as handle:
        reader = csv.DictReader(handle)
        if reader.fieldnames != ["article_id_a", "article_id_b"]:
            raise ValueError("Tệp cặp cần header article_id_a,article_id_b")
        pairs = [(row["article_id_a"], row["article_id_b"]) for row in reader]

    ids, out_neighbors = load_graph(
        graph_dir / "articles.tsv", graph_dir / "links.tsv"
    )
    required_ids = {article for pair in pairs for article in pair}
    if not required_ids.issubset(set(ids)):
        raise ValueError("Tệp cặp chứa ID không có trong articles.tsv")

    # A. Điểm PageRank toàn cục.
    scores = pagerank(ids, out_neighbors, args.beta, args.tol, args.max_iter)
    args.output_dir.mkdir(parents=True, exist_ok=True)
    with (args.output_dir / "pagerank.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        writer.writerow(["article_id", "score"])
        for i, article in enumerate(ids):
            writer.writerow([article, format(scores[i], ".17g")])

    # B. Ba chủ đề và r_mix trong cùng một tệp.
    topics = ["Science", "History", "Geography"]
    topic_scores = []
    for topic in topics:
        teleport = topic_distribution(
            ids, graph_dir / "categories.tsv", "subject." + topic
        )
        topic_scores.append(
            pagerank(ids, out_neighbors, args.beta, args.tol, args.max_iter, teleport)
        )
    mixed_scores = mix_pageranks(topic_scores, [0.5, 0.3, 0.2])
    with (args.output_dir / "topic_pagerank.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        writer.writerow(["topic", "article_id", "score"])
        for topic, vector in zip(topics + ["r_mix"], topic_scores + [mixed_scores]):
            for i, article in enumerate(ids):
                writer.writerow([topic, article, format(vector[i], ".17g")])

    # C. Chữ ký cho mọi bài; chỉ giữ tập chuỗi cho các cặp cần so sánh.
    signatures = {}
    pair_shingles = {}
    with (args.output_dir / "minhash.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        length = len(hash_params["hash_functions"])
        writer.writerow(
            ["article_id", "shingle_count"] + [f"h{j:03d}" for j in range(length)]
        )
        for article in ids:
            text = (text_dir / (article + ".txt")).read_text(encoding="utf-8")
            parts = shingles(normalize_text(text), hash_params["shingle_length"])
            codes = {shingle_id(part, hash_params["p"]) for part in parts}
            signature = minhash(codes, hash_params)
            signatures[article] = signature
            if article in required_ids:
                pair_shingles[article] = parts
            writer.writerow([article, len(parts), *signature])

    with (args.output_dir / "similarities.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        writer.writerow(
            ["article_id_a", "article_id_b", "jaccard_exact", "jaccard_estimate"]
        )
        for article_a, article_b in pairs:
            set_a, set_b = pair_shingles[article_a], pair_shingles[article_b]
            exact = jaccard(set_a, set_b)
            estimate = signature_similarity(
                signatures[article_a], signatures[article_b], not set_a, not set_b
            )
            writer.writerow(
                [article_a, article_b, format(exact, ".17g"), format(estimate, ".17g")]
            )

    print(f"Đã ghi bốn tệp kết quả vào {args.output_dir.resolve()}")


if __name__ == "__main__":
    main()
```

---

## 4. TỪ ĐIỂN BIẾN NỘI BỘ (LOCAL VARIABLES)

| Tên biến | Kiểu dữ liệu | Ý nghĩa & Vai trò trong thuật toán |
|---|---|---|
| **`parser`** | `ArgumentParser` | Đối tượng phân tích cú pháp tham số dòng lệnh từ terminal. |
| **`args`** | `Namespace` | Đối tượng lưu trữ các giá trị tham số cấu hình sau khi parse xong. |
| **`graph_dir`** | `Path` | Thư mục chứa các tệp đồ thị (`articles.tsv`, `links.tsv`, `categories.tsv`). |
| **`text_dir`** | `Path` | Thư mục chứa 4,604 tệp văn bản bài viết thô (`.txt`). |
| **`hash_params`** | `dict` | Cấu hình tham số băm MinHash đọc từ `minhash_params.json`. |
| **`pairs`** | `list[tuple[str, str]]` | Danh sách chứa 100 cặp bài viết cần so sánh `(article_id_a, article_id_b)`. |
| **`ids`** | `list[str]` | Danh sách tên định danh của 4,604 đỉnh sau khi đọc và làm sạch đồ thị. |
| **`out_neighbors`**| `list[list[int]]` | Danh sách kề biểu diễn cung liên kết ra của đồ thị. |
| **`required_ids`** | `set[str]` | Tập hợp chứa tên các bài viết xuất hiện trong bộ 100 cặp so sánh. Dùng làm bộ lọc nhớ để tiết kiệm RAM. |
| **`scores`** | `list[float]` | Vector điểm PageRank toàn cục (Phần A), gồm 4,604 số thực. |
| **`topics`** | `list[str]` | Danh sách 3 chủ đề cần tính toán: `["Science", "History", "Geography"]`. |
| **`topic_scores`** | `list[list[float]]` | Danh sách chứa 3 vector điểm PageRank ứng với 3 chủ đề trên. |
| **`mixed_scores`** | `list[float]` | Vector điểm PageRank hỗn hợp sau khi tổ hợp theo tỷ lệ $50\% - 30\% - 20\%$. |
| **`signatures`** | `dict[str, list[int]]`| Từ điển ánh xạ tên bài viết $\to$ Chữ ký MinHash 128 số nguyên của bài viết đó. |
| **`pair_shingles`**| `dict[str, set[str]]` | Bộ đệm RAM chỉ lưu tập chuỗi shingle của những bài viết có trong `required_ids`. |
| **`exact`** | `float` | Điểm tương đồng Jaccard tính toán chính xác trên tập shingle. |
| **`estimate`** | `float` | Điểm tương đồng Jaccard ước lượng qua chữ ký MinHash. |

---

## 5. GIẢI THÍCH CHI TIẾT TỪNG PHÂN ĐOẠN XỬ LÝ

### Phân đoạn 1: Khởi tạo CLI và đọc cấu hình

```python
    parser = argparse.ArgumentParser(description=__doc__)
    parser.add_argument("--data-dir", type=Path, default=Path("data"))
    ...
    args = parser.parse_args()
```
- Khởi tạo bộ đọc tham số dòng lệnh. Giúp mã nguồn có tính linh hoạt cao, sinh viên hoặc bộ chấm có thể tùy ý trỏ đến thư mục dữ liệu khác mà không cần sửa mã nguồn.

```python
    with args.hash_params.open(encoding="utf-8") as handle:
        hash_params = json.load(handle)
    with args.pairs.open(encoding="utf-8", newline="") as handle:
        reader = csv.DictReader(handle)
        if reader.fieldnames != ["article_id_a", "article_id_b"]:
            raise ValueError("Tệp cặp cần header article_id_a,article_id_b")
        pairs = [(row["article_id_a"], row["article_id_b"]) for row in reader]
```
- Mở và nạp 128 hàm băm từ tệp JSON.
- Đọc 100 cặp bài viết từ `pairs.csv`, kiểm tra đúng cấu trúc tiêu đề cột `article_id_a,article_id_b`.

```python
    ids, out_neighbors = load_graph(
        graph_dir / "articles.tsv", graph_dir / "links.tsv"
    )
    required_ids = {article for pair in pairs for article in pair}
    if not required_ids.issubset(set(ids)):
        raise ValueError("Tệp cặp chứa ID không có trong articles.tsv")
```
- Gọi hàm `load_graph` để nạp đồ thị Wikispeedia.
- Dùng cấu trúc Set Comprehension trích xuất toàn bộ các bài viết có mặt trong 100 cặp (`required_ids`).
- Kiểm tra tính hợp lệ: Tất cả các bài viết trong tập so sánh bắt buộc phải tồn tại trong đồ thị `ids` (kiểm tra tập con `issubset`).

---

### Phân đoạn 2: Tính và ghi tệp PageRank toàn cục (`pagerank.csv`)

```python
    scores = pagerank(ids, out_neighbors, args.beta, args.tol, args.max_iter)
    args.output_dir.mkdir(parents=True, exist_ok=True)
    with (args.output_dir / "pagerank.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        writer.writerow(["article_id", "score"])
        for i, article in enumerate(ids):
            writer.writerow([article, format(scores[i], ".17g")])
```
- `args.output_dir.mkdir(parents=True, exist_ok=True)`: Tự động tạo thư mục `output/` nếu chưa tồn tại.
- Mở tệp `pagerank.csv` với `newline=""` để chuẩn hóa ngắt dòng trên mọi hệ điều hành.
- Ghi dòng tiêu đề `article_id,score`.
- Duyệt qua từng bài viết theo đúng thứ tự `ids` và ghi định dạng `format(scores[i], ".17g")`.
- Tệp sinh ra gồm **4,604 dòng dữ liệu** (+ 1 dòng header = 4,605 dòng).

---

### Phân đoạn 3: Tính và ghi tệp PageRank theo chủ đề (`topic_pagerank.csv`)

```python
    topics = ["Science", "History", "Geography"]
    topic_scores = []
    for topic in topics:
        teleport = topic_distribution(
            ids, graph_dir / "categories.tsv", "subject." + topic
        )
        topic_scores.append(
            pagerank(ids, out_neighbors, args.beta, args.tol, args.max_iter, teleport)
        )
    mixed_scores = mix_pageranks(topic_scores, [0.5, 0.3, 0.2])
```
- Lần lượt sinh vector teleport cho 3 chủ đề Science, History, Geography bằng hàm `topic_distribution`.
- Chạy thuật toán `pagerank` với từng vector teleport tương ứng để thu được 3 vector điểm riêng biệt.
- Gọi hàm `mix_pageranks` để ghép theo tỷ lệ $[0.5, 0.3, 0.2]$ tạo thành vector hỗn hợp `mixed_scores`.

```python
    with (args.output_dir / "topic_pagerank.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        writer.writerow(["topic", "article_id", "score"])
        for topic, vector in zip(topics + ["r_mix"], topic_scores + [mixed_scores]):
            for i, article in enumerate(ids):
                writer.writerow([topic, article, format(vector[i], ".17g")])
```
- Ghi lần lượt 4 khối dữ liệu vào `topic_pagerank.csv`:
  - Khối 1: 4,604 dòng cho chủ đề `Science`.
  - Khối 2: 4,604 dòng cho chủ đề `History`.
  - Khối 3: 4,604 dòng cho chủ đề `Geography`.
  - Khối 4: 4,604 dòng cho `r_mix`.
- Tổng cộng gồm $4 \times 4604 = 18,416$ dòng dữ liệu (+ 1 header = **18,417 dòng**).

---

### Phân đoạn 4: Tính chữ ký MinHash và so sánh Jaccard (`minhash.csv` & `similarities.csv`)

```python
    signatures = {}
    pair_shingles = {}
    with (args.output_dir / "minhash.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        length = len(hash_params["hash_functions"])
        writer.writerow(
            ["article_id", "shingle_count"] + [f"h{j:03d}" for j in range(length)]
        )
        for article in ids:
            text = (text_dir / (article + ".txt")).read_text(encoding="utf-8")
            parts = shingles(normalize_text(text), hash_params["shingle_length"])
            codes = {shingle_id(part, hash_params["p"]) for part in parts}
            signature = minhash(codes, hash_params)
            signatures[article] = signature
            if article in required_ids:
                pair_shingles[article] = parts
            writer.writerow([article, len(parts), *signature])
```

- **Chiến lược tiết kiệm RAM đột phá:**
  - Nếu lưu trữ toàn bộ các tập chuỗi shingle của cả 4,604 bài viết trong bộ nhớ, RAM sẽ bị quá tải (tốn hơn 8 GB).
  - Vì vậy, với mỗi bài viết:
    1. Đọc văn bản và tính shingle.
    2. Băm sang mã số nguyên và tính ngay chữ ký MinHash 128 số nguyên.
    3. Ghi ngay vào tệp `minhash.csv` và lưu chữ ký vào từ điển `signatures` (chỉ tốn khoảng 500 byte mỗi bài).
    4. **Chỉ khi bài viết nằm trong `required_ids`**, ta mới lưu tập chuỗi shingle vào `pair_shingles` để phục vụ bước so sánh sau đó.
    5. Các shingle của những bài viết khác được hệ điều hành giải phóng bộ nhớ ngay lập tức!
- Tệp `minhash.csv` gồm: `article_id`, `shingle_count`, và 128 cột từ `h000` đến `h127`. Gồm **4,604 dòng dữ liệu** (+ 1 header = 4,605 dòng).

```python
    with (args.output_dir / "similarities.csv").open(
        "w", encoding="utf-8", newline=""
    ) as handle:
        writer = csv.writer(handle)
        writer.writerow(
            ["article_id_a", "article_id_b", "jaccard_exact", "jaccard_estimate"]
        )
        for article_a, article_b in pairs:
            set_a, set_b = pair_shingles[article_a], pair_shingles[article_b]
            exact = jaccard(set_a, set_b)
            estimate = signature_similarity(
                signatures[article_a], signatures[article_b], not set_a, not set_b
            )
            writer.writerow(
                [article_a, article_b, format(exact, ".17g"), format(estimate, ".17g")]
            )
```
- Lần lượt duyệt qua 100 cặp trong `pairs.csv`:
  - Lấy tập shingle tương ứng từ bộ nhớ đệm `pair_shingles`.
  - Tính Jaccard chính xác `jaccard(set_a, set_b)`.
  - Tính Jaccard ước lượng qua chữ ký MinHash `signature_similarity(...)`.
  - Ghi vào `similarities.csv` định dạng chuẩn 17 chữ số có nghĩa.
- Tệp gồm đúng **100 dòng dữ liệu** (+ 1 header = 101 dòng).

---

### Phân đoạn 5: Khối bảo vệ thực thi

```python
if __name__ == "__main__":
    main()
```
- **Ý nghĩa sống còn:**
  - Khi người dùng chạy lệnh `python submission.py`, biến nội tại `__name__` mang giá trị `"__main__"`, hàm `main()` được kích hoạt và xuất 4 file CSV.
  - Khi bộ chấm tự động thực hiện lệnh `import submission` trong Python, `__name__` mang giá trị `"submission"`, điều kiện `if` sai $\implies$ Hàm `main()` **hoàn toàn không chạy**. Bộ chấm có thể gọi trực tiếp từng hàm nhỏ (`pagerank`, `shingles`, `minhash`) để chấm điểm mà không bị nghẽn hệ thống.

---

## 6. TẠI SAO LẠI LÀM NHƯ VẬY? (THIẾT KẾ KỸ THUẬT & LÝ DO CỐT LÕI)

### 1. Tại sao dùng thư viện chuẩn `argparse`?
- Thay vì gán cứng các đường dẫn như `"data/articles.tsv"`, việc dùng `argparse` cho phép linh hoạt cấu hình tham số từ giao diện dòng lệnh (CLI).
- Giảng viên hoặc bộ chấm tự động có thể kiểm tra bài nộp trên các bộ dữ liệu nhỏ hoặc thư mục kiểm thử khác nhau (ví dụ: `--data-dir test_data/`) mà không cần phải can thiệp sửa một dòng code nào trong `submission.py`.

### 2. Tại sao mở file CSV luôn phải có `newline=""`?
- Trên hệ điều hành Windows, cơ chế đọc/ghi tệp mặc định sẽ tự động chuyển đổi ký tự xuống dòng `\n` thành cặp ký tự `\r\n`.
- Thư viện `csv.writer` của Python cũng tự động thêm `\r\n` theo chuẩn CSV quốc tế (RFC 4180).
- Nếu không có tham số `newline=""`, trên Windows sẽ xảy ra hiện tượng **"double carriage return"** (`\r\r\n`), dẫn đến việc tệp CSV xuất ra bị xen kẽ các dòng trống vô nghĩa làm bộ chấm báo lỗi sai định dạng!

### 3. Tại sao bắt buộc dùng `format(val, ".17g")`?
- Trong chuẩn số thực IEEE 754 (Double precision float64), phần định trị (mantissa) có 53 bit nhị phân, tương đương chính xác khoảng 15 đến 17 chữ số thập phân có nghĩa.
- Nếu bạn dùng hàm `round(val, 6)` hoặc `f"{val:.6f}"`, bạn đã vô tình cắt vứt đi phần đuôi số thực cực kỳ quý giá, khiến khoảng cách $L_1$ và sai số kiểm thử không thể đạt ngưỡng khắt khe $10^{-10}$ của đề bài.
- Định dạng `".17g"` đảm bảo lưu trọn vẹn từng bit giá trị của số thực mà không bị hao hụt độ chính xác.

### 4. Tại sao chỉ lưu `pair_shingles` cho các bài viết trong `required_ids`?
- Toàn bộ 4,604 bài viết tạo ra hàng triệu mảnh shingle dạng chuỗi văn bản. Nếu lưu tất cả vào RAM, chương trình sẽ tiêu tốn từ 6 GB đến 8 GB bộ nhớ, gây hiện tượng tràn RAM (Out of Memory) trên máy cấu hình thông thường.
- Danh sách `pairs.csv` chỉ gồm 100 cặp bài viết (tổng số bài viết độc nhất chỉ khoảng 80–120 bài).
- Bộ lọc `if article in required_ids:` giúp giảm tải bộ nhớ hơn **98%**, chương trình chạy mượt mà chỉ với vài trăm Megabyte RAM.

---

## 7. BẢNG TỔNG KẾT 4 TỆP CSV KẾT QUẢ ĐẦU RA

| Tên tệp CSV | Vị trí lưu | Cấu trúc các cột (Header) | Số dòng chính xác | Dung lượng ước tính |
|---|---|---|:---:|:---:|
| **`pagerank.csv`** | `output/` | `article_id,score` | **4,605** dòng *(4,604 dữ liệu + 1 header)* | $\approx 115$ KB |
| **`topic_pagerank.csv`** | `output/` | `topic,article_id,score` | **18,417** dòng *(18,416 dữ liệu + 1 header)* | $\approx 550$ KB |
| **`minhash.csv`** | `output/` | `article_id,shingle_count,h000,...,h127` | **4,605** dòng *(4,604 dữ liệu + 1 header)* | $\approx 4.8$ MB |
| **`similarities.csv`** | `output/` | `article_id_a,article_id_b,jaccard_exact,jaccard_estimate` | **101** dòng *(100 dữ liệu + 1 header)* | $\approx 5$ KB |
