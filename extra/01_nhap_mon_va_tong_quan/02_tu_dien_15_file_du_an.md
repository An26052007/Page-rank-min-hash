# BÀI 06: GIẢI THÍCH CHI TIẾT TỪNG TỆP TRONG DỰ ÁN GỐC
> **Từ điển bóc tách toàn bộ các tệp tin trong `wikispeedia-student-package`**  
> **Cấu trúc:** Tác dụng gì? Sử dụng ở đâu? Dữ liệu nhập vào là gì? Xuất ra cái gì? Lưu ý kỹ thuật.

---

## 🗺️ TỔNG QUAN BẢN ĐỒ CÁC TỆP TRONG DỰ ÁN

```
wikispeedia-student-package/
├── 📄 README.md                            [Tài liệu] Hướng dẫn tóm tắt quy trình nộp bài
├── 📄 bai-tap-wikispeedia.md               [Tài liệu] Đặc tả chi tiết đề bài, công thức và thang điểm
├── 📄 SHA256SUMS.txt                       [Kiểm định] Bảng mã băm kiểm tra tính toàn vẹn gói bài tập
├── 🐍 autograder.py                        [Công cụ] Bộ tự chấm điểm tự động (13 tiêu chí, 100đ)
├── 🐍 submission.py                        [Mã nguồn] File làm bài chính thức của sinh viên
│
├── 📁 assignment/
│   ├── 🐍 submission_template.py          [Mẫu] Template ban đầu chứa các hàm TODO
│   ├── ⚙️ minhash_params.json              [Cấu hình] 128 hàm băm affine và số nguyên tố p
│   └── 📊 pairs_sample.csv                 [Kiểm thử] Bộ 4 cặp bài viết để chạy thử nghiệm nhanh
│
├── 📁 data/
│   ├── 📊 pairs.csv                        [Dữ liệu] Bộ 100 cặp bài viết chính thức đánh giá Jaccard
│   ├── 📦 wikispeedia_paths-and-graph.tar.gz  (Giải nén ra thư mục wikispeedia_paths-and-graph/)
│   │   ├── 📋 articles.tsv                 [Dữ liệu] Danh sách 4,604 đỉnh của đồ thị
│   │   ├── 📋 links.tsv                    [Dữ liệu] Danh sách các liên kết có hướng giữa các bài
│   │   ├── 📋 categories.tsv               [Dữ liệu] Danh mục phân loại chủ đề của từng bài viết
│   │   ├── 📋 paths_finished.tsv           [Dữ liệu phụ] Nhật ký các ván chơi tìm đường thành công
│   │   ├── 📋 paths_unfinished.tsv         [Dữ liệu phụ] Nhật ký các ván chơi người chơi bỏ cuộc
│   │   └── 📋 shortest-path-distance-matrix.txt [Dữ liệu phụ] Ma trận khoảng cách ngắn nhất
│   └── 📦 wikispeedia_articles_plaintext.tar.gz (Giải nén ra thư mục plaintext_articles/)
│       └── 📄 plaintext_articles/*.txt    [Dữ liệu] 4,604 file văn bản thô của từng bài viết
│
└── 📁 output/                              [Đầu ra] Thư mục chứa 4 tệp kết quả CSV nộp bài
    ├── 📊 pagerank.csv                     [Kết quả Phần A] Điểm PageRank toàn cục (4,604 dòng)
    ├── 📊 topic_pagerank.csv               [Kết quả Phần B] Điểm PageRank 3 chủ đề + r_mix (18,416 dòng)
    ├── 📊 minhash.csv                      [Kết quả Phần C] Chữ ký MinHash 128 số của 4,604 bài
    └── 📊 similarities.csv                 [Kết quả Phần C] So sánh Jaccard exact vs estimate (100 dòng)
```

---

# PHẦN 1: CÁC TỆP TÀI LIỆU VÀ QUY CHUẨN ĐỀ BÀI

---

### 1. `README.md`
- **Vị trí:** Thư mục gốc (`./README.md`).
- **Tác dụng:** Là tài liệu chào đón (Landing Document) hướng dẫn nhanh cho sinh viên khi vừa tải gói bài tập về.
- **Ai sử dụng:** Sinh viên đọc để nắm 4 bước cơ bản: Chuẩn bị môi trường $\to$ Giải nén dữ liệu $\to$ Chạy code $\to$ Tự chấm và nộp bài.
- **Dữ liệu bên trong:** Các lệnh shell mẫu (`tar -xzf`, `python3 submission.py`, `python3 autograder.py`) và quy định số dòng của 4 file CSV đầu ra.
- **Lưu ý:** `README.md` chỉ tóm tắt ngắn gọn; chi tiết công thức toán học và bảng điểm nằm ở `bai-tap-wikispeedia.md`.

---

### 2. `bai-tap-wikispeedia.md`
- **Vị trí:** Thư mục gốc (`./bai-tap-wikispeedia.md`).
- **Tác dụng:** Là **văn bản pháp lý cao nhất của đề bài**. Chứa toàn bộ đặc tả toán học, công thức cập nhật, quy tắc tiền xử lý văn bản, quy ước xử lý ngoại lệ và barem bảng điểm chi tiết (100 điểm).
- **Ai sử dụng:** Sinh viên và người chấm dùng làm căn cứ đối soát tính đúng đắn của thuật toán.
- **Nội dung quan trọng cần nhớ từ tệp này:**
  - Quy ước đọc đồ thị: Giữ mã hóa percent-encoding, sắp xếp ID theo byte UTF-8, gộp cạnh trùng, giữ cạnh tự nối.
  - Công thức PageRank: Bù nút cụt $\delta_t$ đều trên $n$ đỉnh.
  - PageRank theo chủ đề: Bù nút cụt bằng phân phối đều $u$, teleport bằng phân phối chủ đề $v$.
  - Shingling & MinHash: 9 code points, SHA-256 4 byte big-endian $\bmod p$.
  - Bảng điểm 13 tiêu chí.

---

### 3. `SHA256SUMS.txt`
- **Vị trí:** Thư mục gốc (`./SHA256SUMS.txt`).
- **Tác dụng:** Cung cấp mã băm kiểm tra tính toàn vẹn (Checksum SHA-256) của 9 tệp gốc do giảng viên đóng gói.
- **Ai sử dụng:** Dùng khi sinh viên vừa tải gói ZIP về để kiểm tra xem quá trình tải tệp có bị lỗi, hỏng dữ liệu hoặc thiếu file hay không.
- **Dữ liệu bên trong:** Gồm 9 dòng, mỗi dòng là mã hex 64 ký tự của SHA-256 đi kèm tên tệp.
  *Ví dụ:* `97697096f5d2dcb77aa69e3992305c6c561de89edb9fb10b5ad9feaf8ba534d5 data/wikispeedia_paths-and-graph.tar.gz`.
- **Lưu ý:** Không tự ý sửa tệp này. Bạn có thể kiểm tra toàn vẹn trên PowerShell bằng lệnh:
  ```powershell
  Get-FileHash data/wikispeedia_paths-and-graph.tar.gz -Algorithm SHA256
  ```

---

# PHẦN 2: CÁC TỆP MÃ NGUỒN VÀ BỘ TỰ CHẤM

---

### 4. `assignment/submission_template.py`
- **Vị trí:** `assignment/submission_template.py`.
- **Tác dụng:** Là tệp khuôn mẫu gốc (Template) do đề bài cung cấp sẵn. Chứa khung sườn của 10 hàm cần cài đặt đi kèm **docstring đặc tả chi tiết** (đầu vào, đầu ra, quy ước ném lỗi `ValueError`), cùng với hàm `main()` đã viết sẵn phần đọc tham số dòng lệnh CLI và ghi 4 file CSV.
- **Ai sử dụng:** Sinh viên sao chép tệp này thành `submission.py` ở thư mục gốc để bắt đầu làm bài.
- **Trạng thái ban đầu:** Các hàm đều chứa dòng lệnh `raise NotImplementedError("Chưa cài đặt...")`.
- **Lưu ý:** Giữ nguyên tệp gốc trong thư mục `assignment/` làm bản sao lưu dự phòng; không sửa trực tiếp vào tệp template này.

---

### 5. `submission.py`
- **Vị trí:** Thư mục gốc (`./submission.py`).
- **Tác dụng:** **Đây là tệp quan trọng nhất của toàn bộ bài tập** — tệp mã nguồn duy nhất sinh viên nộp cho giảng viên.
- **Ai sử dụng:**
  - Sinh viên chạy tệp này để tạo ra 4 tệp kết quả CSV: `python submission.py`.
  - Bộ chấm `autograder.py` import tệp này để kiểm tra từng hàm độc lập.
- **Dữ liệu nhập vào:**
  - `data/wikispeedia_paths-and-graph/articles.tsv` và `links.tsv` (cho Phần A).
  - `data/wikispeedia_paths-and-graph/categories.tsv` (cho Phần B).
  - `assignment/minhash_params.json`, `data/plaintext_articles/*.txt`, `data/pairs.csv` (cho Phần C).
- **Dữ liệu xuất ra:**
  - Tạo thư mục `output/` chứa đúng 4 file: `pagerank.csv`, `topic_pagerank.csv`, `minhash.csv`, `similarities.csv`.
- **10 hàm cốt lõi bên trong:**
  1. `load_graph`: Đọc TSV, trả về `(ids, out_neighbors)`.
  2. `pagerank`: Tính điểm PageRank lặp Power Iteration.
  3. `topic_distribution`: Tạo vector phân phối $v$ cho từng chủ đề.
  4. `mix_pageranks`: Ghép các vector theo trọng số $0.5, 0.3, 0.2$.
  5. `normalize_text`: Chuẩn hóa chuỗi (chỉ hạ ASCII A-Z, gộp khoảng trắng).
  6. `shingles`: Cắt cửa sổ trượt 9 ký tự Unicode.
  7. `shingle_id`: Băm chuỗi shingle thành số nguyên bằng SHA-256 $\bmod p$.
  8. `minhash`: Tính vector 128 số nguyên nhỏ nhất.
  9. `jaccard`: Tính hệ số Jaccard chính xác trên 2 tập shingle.
  10. `signature_similarity`: Đếm tỷ lệ tọa độ trùng nhau giữa 2 chữ ký.

---

### 6. `autograder.py`
- **Vị trí:** Thư mục gốc (`./autograder.py`).
- **Tác dụng:** Là hệ thống tự chấm điểm độc lập tại máy cá nhân, mô phỏng đúng cách thức giảng viên chấm bài.
- **Ai sử dụng:** Sinh viên chạy lệnh:
  ```powershell
  python autograder.py submission.py --timeout 300 --report grading_reports/self_check.json
  ```
- **Cơ chế hoạt động:**
  - Import module `submission.py`.
  - Chạy 13 hàm kiểm thử độc lập (`test_a1`, `test_a2`,..., `test_c4`, `test_io`) trong các tiến trình riêng có giới hạn thời gian (timeout).
  - So sánh kết quả của sinh viên với các **Oracle toán học độc lập** (ví dụ: giải hệ phương trình tuyến tính bằng thuật toán khử Gauss-Jordan, toán tử thưa, băm độc lập).
- **Dữ liệu xuất ra:**
  - In điểm từng nhóm tiêu chí và tổng điểm trên màn hình terminal (tối đa 100 điểm).
  - Ghi báo cáo JSON chi tiết vào `grading_reports/self_check.json`.

---

# PHẦN 3: CÁC TỆP CẤU HÌNH VÀ BỘ CẶP KIỂM THỬ

---

### 7. `assignment/minhash_params.json`
- **Vị trí:** `assignment/minhash_params.json`.
- **Tác dụng:** Định nghĩa cấu hình cố định cho toàn bộ thuật toán MinHash của bài tập, đảm bảo mọi sinh viên và bộ chấm đều dùng chung một hệ hàm băm.
- **Ai sử dụng:** Hàm `main()` trong `submission.py` và bộ chấm `autograder.py` đọc tệp này để lấy tham số băm.
- **Cấu trúc dữ liệu bên trong:**
  ```json
  {
    "version": "wikispeedia-minhash-v1",
    "p": 2147483647,
    "signature_length": 128,
    "shingle_length": 9,
    "empty_value": 2147483647,
    "hash_functions": [
      {"a": 1660981080, "b": 1401832859},
      {"a": 1856361611, "b": 1395632257},
      ... (đủ 128 cặp a, b)
    ]
  }
  ```
  - `p`: Số nguyên tố Mersenne $2^{31} - 1 = 2147483647$.
  - `shingle_length`: Độ dài cửa sổ shingle $k = 9$.
  - `empty_value`: Giá trị gán cho tập rỗng (sentinel value $p$).
  - `hash_functions`: Danh sách 128 cặp hệ số $(a_j, b_j)$ dùng cho hàm băm affine $h_j(x) = (a_j \cdot x + b_j) \bmod p$.
- **Lưu ý:** Tuyệt đối không tự ý chỉnh sửa các số $a, b$ hoặc $p$ trong tệp này.

---

### 8. `assignment/pairs_sample.csv`
- **Vị trí:** `assignment/pairs_sample.csv`.
- **Tác dụng:** Bộ dữ liệu kiểm thử nhanh (Smoke test) chỉ gồm **4 cặp bài viết**.
- **Ai sử dụng:** Sinh viên dùng để chạy thử hàm `main()` xem chương trình có chạy thông suốt hay không mà không phải đợi băm 100 cặp bài viết lớn:
  ```powershell
  python submission.py --pairs assignment/pairs_sample.csv --output-dir test_out
  ```
- **Nội dung bên trong:**
  ```csv
  article_id_a,article_id_b
  Glass,Glass
  Fossil,Fossil_record
  Geography,Glacier
  Babur,Bronze_Age
  ```
  (Gồm 1 cặp trùng nhau $100\%$, 1 cặp tương đồng cao, 1 cặp cùng chủ đề và 1 cặp khác chủ đề).

---

### 9. `data/pairs.csv`
- **Vị trí:** `data/pairs.csv`.
- **Tác dụng:** Là **bộ 100 cặp bài viết chính thức** dùng để xuất tệp `similarities.csv` khi nộp bài và dùng trong nhóm kiểm thử chấm điểm C3.
- **Ai sử dụng:** Được gọi mặc định khi chạy `python submission.py` và bởi `autograder.py`.
- **Cơ cấu 100 cặp bài viết:**
  - **10 cặp đầu tiên:** Một bài so sánh với chính nó ($A \equiv A \implies \text{Jaccard} = 1.0$).
  - **30 cặp tiếp theo:** Hai bài viết khác nhau nhưng có nội dung trùng lặp cao (đã khảo sát trước).
  - **30 cặp tiếp theo:** Hai bài viết có chung một nhãn chủ đề cấp cao nhất (ví dụ cùng là Khoa học).
  - **30 cặp cuối cùng:** Hai bài viết chọn ngẫu nhiên không cùng chủ đề (thường có Jaccard rất thấp hoặc bằng 0).
- **Lưu ý:** Không tự ý đổi thứ tự các dòng trong tệp này vì tệp đầu ra `similarities.csv` phải giữ nguyên đúng 100 dòng theo thứ tự tương ứng.

---

# PHẦN 4: CÁC TỆP DỮ LIỆU THÔ WIKISPEEDIA (TRONG 2 TỆP TAR.GZ)

---

### 10. `data/wikispeedia_paths-and-graph.tar.gz`
Tệp nén (kích thước 9.9 MB), khi giải nén bằng `tar -xzf` sẽ tạo ra thư mục `data/wikispeedia_paths-and-graph/` chứa 6 tệp:

#### 10.1. `articles.tsv` (Sử dụng trực tiếp)
- **Tác dụng:** Định nghĩa toàn bộ danh sách bài viết (tập đỉnh của đồ thị).
- **Cấu trúc:** Mỗi dòng là một ID bài viết.
  *Ví dụ:* `10th_century`, `%C3%89douard_Manet`, `Zebra`.
- **Thống kê:** Có đúng **4,604 bài viết** phân biệt.
- **Sử dụng ở đâu:** Hàm `load_graph` đọc tệp này để tạo danh sách đỉnh `ids`.

#### 10.2. `links.tsv` (Sử dụng trực tiếp)
- **Tác dụng:** Định nghĩa toàn bộ các liên kết giữa các bài viết (tập cạnh có hướng).
- **Cấu trúc:** Mỗi dòng có định dạng: `bài_nguồn <TAB> bài_đích`.
  *Ví dụ:* `10th_century <TAB> 11th_century`.
- **Thống kê:** Có **119,882 cạnh có hướng phân biệt** (sau khi gộp cạnh trùng), gồm 110 cạnh tự nối và tạo ra 17 nút cụt.
- **Sử dụng ở đâu:** Hàm `load_graph` đọc tệp này để dựng danh sách kề `out_neighbors`.

#### 10.3. `categories.tsv` (Sử dụng trực tiếp)
- **Tác dụng:** Định nghĩa cây phân loại chủ đề của từng bài viết.
- **Cấu trúc:** Mỗi dòng có định dạng: `bài_viết <TAB> nhãn_danh_mục`.
  *Ví dụ:* `Albert_Einstein <TAB> subject.Science.Physics.Physicists`.
- **Sử dụng ở đâu:** Hàm `topic_distribution` đọc tệp này để lọc ra các bài viết thuộc `subject.Science`, `subject.History`, `subject.Geography`.

#### 10.4. Các tệp phụ trong kho gốc Wikispeedia (KHÔNG sử dụng trong bài tập này)
- **`paths_finished.tsv` & `paths_unfinished.tsv`:** Nhật ký các ván chơi tìm đường của người dùng thực tế trên web Wikispeedia (ghi nhận đường đi qua các bài viết, thời gian nhấp chuột). Đề bài bài tập này chỉ tập trung vào phân tích cấu trúc liên kết và nội dung văn bản, không phân tích hành vi người chơi, nên không sử dụng 2 tệp này.
- **`shortest-path-distance-matrix.txt`:** Bảng ma trận khoảng cách ngắn nhất giữa mọi cặp bài viết do Stanford tính sẵn trước đây. Thuật toán PageRank của chúng ta tự tính toán sự lan truyền xác suất trên toàn đồ thị nên không cần đến ma trận này.

---

### 11. `data/wikispeedia_articles_plaintext.tar.gz`
Tệp nén (kích thước 35.8 MB), khi giải nén bằng `tar -xzf` sẽ tạo ra thư mục `data/plaintext_articles/`:
- **Chứa:** Đúng **4,604 tệp văn bản thô `.txt`**, mỗi tệp chứa toàn bộ nội dung của một bài viết.
  *Ví dụ:* `data/plaintext_articles/Apple.txt`, `data/plaintext_articles/%C3%89ire.txt`.
- **Sử dụng ở đâu:** Vòng lặp trong hàm `main()` của Phần C đọc từng tệp `.txt`, đưa qua hàm `normalize_text` $\to$ `shingles` $\to$ `shingle_id` $\to$ `minhash`.
- **Đầu ra:** Mỗi tệp văn bản sinh ra số lượng shingle phân biệt và vector 128 số nguyên ghi vào `minhash.csv`.

---

# PHẦN 5: CÁC TỆP ĐẦU RA KẾT QUẢ TRONG THƯ MỤC `output/`

Sau khi chạy `python submission.py`, thư mục `output/` sẽ chứa đúng 4 tệp CSV sau để nộp bài:

---

### 12. `output/pagerank.csv`
- **Tác dụng:** Lưu kết quả tính PageRank toàn cục (Phần A).
- **Cấu trúc:** Header `article_id,score`.
- **Số dòng:** Đúng **4,604 dòng dữ liệu** (+ 1 dòng header = 4,605 dòng).
- **Quy cách:** Mỗi bài viết nằm trên 1 dòng theo đúng thứ tự từ điển UTF-8 của `ids`. Điểm số ghi định dạng `.17g`. Tổng tất cả các điểm số trong tệp này phải bằng đúng `1.0`.

---

### 13. `output/topic_pagerank.csv`
- **Tác dụng:** Lưu kết quả tính PageRank cho 3 chủ đề và vector kết hợp (Phần B).
- **Cấu trúc:** Header `topic,article_id,score`.
- **Số dòng:** Đúng **18,416 dòng dữ liệu** (+ 1 dòng header = 18,417 dòng).
- **Quy cách:** Gồm 4 khối dữ liệu xếp chồng lên nhau theo thứ tự:
  1. 4,604 dòng có cột `topic` là `Science`.
  2. 4,604 dòng có cột `topic` là `History`.
  3. 4,604 dòng có cột `topic` là `Geography`.
  4. 4,604 dòng có cột `topic` là `r_mix` (ghép với trọng số $0.5, 0.3, 0.2$).

---

### 14. `output/minhash.csv`
- **Tác dụng:** Lưu chữ ký MinHash của toàn bộ kho tài liệu (Phần C).
- **Cấu trúc:** Header `article_id,shingle_count,h000,h001,...,h127` (gồm 130 cột).
- **Số dòng:** Đúng **4,604 dòng dữ liệu** (+ 1 dòng header = 4,605 dòng).
- **Quy cách:**
  - `article_id`: Tên bài viết theo đúng thứ tự UTF-8.
  - `shingle_count`: Số lượng shingle 9 ký tự phân biệt của bài viết trước bước băm.
  - `h000` đến `h127`: 128 số nguyên đại diện cho giá trị băm nhỏ nhất của 128 hàm băm affine.

---

### 15. `output/similarities.csv`
- **Tác dụng:** Lưu kết quả đánh giá độ tương đồng văn bản trên bộ cặp kiểm thử chính thức (Phần C).
- **Cấu trúc:** Header `article_id_a,article_id_b,jaccard_exact,jaccard_estimate`.
- **Số dòng:** Đúng **100 dòng dữ liệu** (+ 1 dòng header = 101 dòng).
- **Quy cách:**
  - Giữ nguyên đúng thứ tự 100 cặp từ `data/pairs.csv`.
  - `jaccard_exact`: Hệ số Jaccard chính xác tính trực tiếp từ phép giao/hợp tập shingle.
  - `jaccard_estimate`: Hệ số Jaccard ước lượng bằng tỷ lệ số tọa độ trùng nhau trong 128 con số của chữ ký MinHash.
