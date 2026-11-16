# BỘ TÀI LIỆU HƯỚNG DẪN CHI TIẾT BÀI TẬP WIKISPEEDIA
> **Dành cho người mới bắt đầu (Beginner-Friendly Guide)**  
> **Môn học:** Giải thuật Khoa học Dữ liệu (KHDL)

---

## 📌 Lời mở đầu

Chào bạn! Nếu bạn mới tiếp cận môn **Giải thuật Khoa học Dữ liệu** hoặc chưa từng học qua các thuật toán tìm kiếm liên kết (PageRank) và khai phá tương đồng văn bản (MinHash), tài liệu này được biên soạn riêng cho bạn.

Mục tiêu của bộ tài liệu này là:
1. **Giải thích bằng trực giác đời thực:** Không dùng các thuật ngữ toán học hàn lâm khó hiểu mà giải thích bản chất "tại sao lại làm như vậy".
2. **Bóc tách từng mục trong đề bài:** Đi sâu vào từng phần A, B, C và yêu cầu I/O của đề bài.
3. **Liên hệ trực tiếp với code:** Giúp bạn hiểu từng hàm trong [submission.py](file:///C:/Users/HP/Downloads/HKI_2026-2027/GI%E1%BA%A2I%20THU%E1%BA%ACT%20KHDL/wikispeedia-student-package/submission.py) hoạt động như thế nào và vì sao lại viết như vậy.

---

## 🗺️ Mục lục các bài hướng dẫn

| Tệp hướng dẫn | Nội dung chính |
|---|---|
| [00_TONG_QUAN_VA_BOI_CANH.md](00_TONG_QUAN_VA_BOI_CANH.md) | Bối cảnh trò chơi Wikispeedia, dữ liệu lớn là gì, cấu trúc các tệp TSV/tar.gz và quy ước đọc dữ liệu. |
| [01_PHAN_A_PAGERANK_TOAN_CUC.md](01_PHAN_A_PAGERANK_TOAN_CUC.md) | **Phần A (30đ):** Thuật toán Google PageRank, mô hình người lướt web ngẫu nhiên, cách xử lý nút cụt (Dead ends) và phương pháp lặp Power Iteration. |
| [02_PHAN_B_PAGERANK_THEO_CHU_DE.md](02_PHAN_B_PAGERANK_THEO_CHU_DE.md) | **Phần B (30đ):** PageRank theo chủ đề (Personalized PageRank), quy tắc bù nút cụt đặc biệt của bài toán và tính chất ghép vector tuyến tính ($r_{\text{mix}}$). |
| [03_PHAN_C_SHINGLING_VA_MINHASH.md](03_PHAN_C_SHINGLING_VA_MINHASH.md) | **Phần C (35đ):** So sánh văn bản quy mô lớn, $k$-shingling (cửa sổ trượt 9 ký tự), băm SHA-256 và chữ ký MinHash 128 tọa độ. |
| [04_HUONG_DAN_CHAY_CHAM_DIEM_VA_NOP_BAI.md](04_HUONG_DAN_CHAY_CHAM_DIEM_VA_NOP_BAI.md) | Cách chạy chương trình, đọc hiểu báo cáo 13 tiêu chí của `autograder.py` (100 điểm) và kiểm tra 4 tệp CSV nộp bài. |

---

## 🚀 Cách bắt đầu học

1. Đọc bài **00** để hiểu bài toán và dữ liệu.
2. Đọc lần lượt các bài **01**, **02**, **03** song song với việc đối chiếu code trong `submission.py`.
3. Đọc bài **04** để biết cách chạy tự chấm điểm và xuất file nộp bài.
