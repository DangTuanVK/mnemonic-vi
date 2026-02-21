# Log - Mnemonic-VI

## 2026-02-21

### 09:00 - #1 - Tạo `project-spec.md`

- Tạo file đặc tả dự án ban đầu gồm: tổng quan, mục tiêu, yêu cầu kỹ thuật, tính năng, cấu trúc dự án, dependencies, bảo mật
- Chỉnh sửa:
  - Thay wordlist từ đơn sang **từ ghép 2 từ đơn**, phân tách bằng `-` trong mnemonic phrase
  - Thêm tiêu chí chọn từ: không nhập nhằng chính tả, không nhập nhằng cách đọc vùng miền, rõ nghĩa, dễ viết
  - Tách cấu trúc dự án thành **public** (`mnemonic-vi/`) và **private** (`mnemonic-vi-dev/`)
  - Thêm tiêu chí **trung tính về nội dung**: tránh từ liên quan chính trị, tôn giáo, bạo lực, tục tĩu

### 09:20 - #2 - Tạo `working-job.md`

- Tạo danh sách 16 công việc cần làm, chia theo public (9) và private (7)

### 09:25 - #3 - Tạo `log.md`

- Tạo file ghi lịch sử công việc

### 09:30 - #10 - Khởi tạo project Python

- Tạo cấu trúc thư mục: `src/mnemonic_vi/`, `scripts/`, `data/`, `tests/`, `examples/`
- Tạo `pyproject.toml`: Python >=3.10, build backend hatchling, dev deps pytest + ruff
- Tạo `src/mnemonic_vi/__init__.py`

### 09:46 - Chuyển cấu trúc thư mục

- Di chuyển từ `mnemonic-vi-dev/` (thư mục riêng) vào `mnemonic-vi/dev/` (trong project)
- Tạo `.gitignore` chứa `dev/` để không public
- Cập nhật `project-spec.md` cho phù hợp cấu trúc mới

### 09:50 - #11 - Viết `mnemonic.py`

- Tạo `dev/src/mnemonic_vi/mnemonic.py` - class `Mnemonic` với các method:
  - `generate(strength)`: Sinh mnemonic từ entropy ngẫu nhiên (secrets)
  - `from_entropy(entropy)`: Chuyển entropy bytes thành mnemonic phrase
  - `to_entropy(mnemonic)`: Chuyển mnemonic thành entropy, verify checksum
  - `validate(mnemonic)`: Kiểm tra tính hợp lệ
  - `to_seed(mnemonic, passphrase)`: PBKDF2-HMAC-SHA512, 2048 iterations
- Cập nhật `__init__.py` export class `Mnemonic`

### 09:52 - #12 - Viết `wordlist.py`

- Tạo `dev/src/mnemonic_vi/wordlist.py` với các function:
  - `load(language)`: Load wordlist từ file, validate đúng 2048 từ
  - `validate_wordlist(words)`: Kiểm tra trùng lặp, từ ghép 2 phần, thứ tự UTF-8
  - `find_index(words, compound_word)`: Tìm index của từ ghép
- Refactor `mnemonic.py`: bỏ `_load_wordlist()` nội bộ, import từ `wordlist` module

### 09:54 - #13 - Viết `generate_wordlist.py`

- Tạo `dev/scripts/generate_wordlist.py` - script CLI sinh/lọc wordlist:
  - Đọc từ thô từ `data/raw_words.txt`
  - Lọc: từ ghép 2 phần, không trùng, không nhạy cảm (danh sách SENSITIVE_KEYWORDS)
  - Sắp xếp UTF-8, xuất ra `wordlists/vietnamese.txt`
  - Lưu từ bị loại + lý do vào `data/excluded_words.txt`
  - Các mode: `--validate-only` (chỉ kiểm tra), `--stats` (thống kê)

### 10:02 - #14 - Chuẩn bị `raw_words.txt`

- Tạo `dev/data/raw_words.txt` với 2142 từ ghép duy nhất (2035 dòng gốc + bổ sung, 47 trùng)
- Tổ chức theo 20+ nhóm chủ đề: thiên nhiên, thực vật, động vật, thực phẩm, nhà cửa, con người, cơ thể, quần áo, nghề nghiệp, phương tiện, giáo dục, hoạt động, tính chất, địa điểm, thời gian, âm nhạc, thể thao, màu sắc, kinh tế, lễ hội, công cụ, vật liệu, thời tiết, v.v.
- Đủ buffer (~100 từ dư) cho quá trình lọc để đạt 2048 từ

### 10:08 - #15 - Viết tests

- Chạy `generate_wordlist.py` -> tạo `wordlists/vietnamese.txt` (2048 từ, 64 bị loại)
- Copy wordlist vào `dev/src/mnemonic_vi/wordlists/` để module đọc được
- Tạo 3 file test (48 test cases tổng cộng):
  - `test_wordlist.py` (10 tests): load, validate, find_index, duplicates, sort
  - `test_mnemonic.py` (17 tests): generate, from_entropy, to_entropy roundtrip, validate, to_seed, checksum
  - `test_vectors.py` (11 tests): entropy cố định (all zeros, all ones, sequential), passphrase, consistency
- Kết quả: **48/48 passed** trong 0.17s

### 10:17 - Sửa bộ lọc nhạy cảm

- **Vấn đề**: `contains_sensitive()` dùng substring match (`"đĩ" in "đĩa cơm"` -> True) gây loại nhầm 7 từ hợp lệ: `đĩa cơm`, `đĩa hoa`, `đĩa gốm`, `đánh cầu`, `đĩnh đạc`, `bát đĩa`, `lưới đánh`
- **Fix**: Tách `SENSITIVE_KEYWORDS` thành 2 tập:
  - `SENSITIVE_COMPOUNDS`: exact match cả cụm (`"chiến tranh"`, `"cộng sản"`, ...)
  - `SENSITIVE_WORDS`: exact match từng phần (`"đĩ"` chỉ match `"đĩ"`, không match `"đĩa"`)
- Kết quả: 57 từ bị loại (tất cả duplicate, 0 false positive), tests 48/48 passed

### 10:30 - Tạo nhiều phương án raw_words

- Rename `raw_words.txt` → `raw_words_v1_manual.txt` (2142 từ, nguồn: tự soạn theo chủ đề)
- Tạo `scripts/generate_v2_nlp.py` - script trích xuất từ ghép từ Vietnamese NLP datasets:
  - Nguồn: underthesea Viet74K (73,901 mục) + Viet39K (39,071 mục)
  - Trích xuất từ ghép 2 phần, lọc tên riêng, từ nhạy cảm, từ không thuần Việt
  - Ưu tiên từ có trong cả Viet39K (phổ biến hơn)
- Tách riêng 2 nguồn thay vì gộp:
  - `raw_words_v2-Viet74K.txt`: **48,510 từ ghép** từ Viet74K (từ điển lớn)
  - `raw_words_v3-Viet39K.txt`: **30,578 từ ghép** từ Viet39K (từ phổ biến hơn)
- Xóa `raw_words_v2_nlp.txt` (đã thay bằng 2 file tách riêng)
