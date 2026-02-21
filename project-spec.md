# Mnemonic-VI - Project Specification

## Tổng quan

Thư viện Python tạo mnemonic (cụm từ khôi phục ví) bằng tiếng Việt cho các dự án blockchain, tuân thủ chuẩn BIP-39.

## Mục tiêu

- Tạo danh sách 2048 từ tiếng Việt theo chuẩn BIP-39
- Sinh mnemonic phrase (12, 15, 18, 21, 24 từ) từ entropy ngẫu nhiên
- Chuyển đổi mnemonic sang seed để tạo ví blockchain
- Hỗ trợ validate mnemonic phrase
- Tương thích với các ví HD (Hierarchical Deterministic) theo BIP-32/BIP-44

## Yêu cầu kỹ thuật

### Ngôn ngữ & phiên bản

- Python >= 3.10

### Chuẩn tuân thủ

| Chuẩn   | Mô tả                                |
| ------- | ------------------------------------- |
| BIP-39  | Mnemonic code cho HD wallets          |
| BIP-32  | Hierarchical Deterministic Wallets    |
| BIP-44  | Multi-account HD wallet structure     |

### Wordlist tiếng Việt

- Đúng 2048 từ ghép, không trùng lặp
- Mỗi mục trong wordlist là **từ ghép gồm 2 từ đơn**, phân tách bằng dấu cách
  - Ví dụ: `hoa sen`, `mặt trời`, `biển xanh`, `cánh đồng`
- Trong file `vietnamese.txt`, mỗi dòng chứa một từ ghép
- Phân tách giữa các từ ghép trong mnemonic phrase bằng dấu gạch ngang `-`
  - Ví dụ mnemonic 12 từ: `hoa-sen biển-xanh cánh-đồng mặt-trời ...`
- Sắp xếp theo thứ tự UTF-8
- Tiêu chí chọn từ ghép:
  - **Phổ biến**: Từ thông dụng trong đời sống, ai cũng biết
  - **Dễ nhớ**: Gợi hình ảnh cụ thể, trực quan (ví dụ: `hoa sen`, `mặt trời`)
  - **Không nhập nhằng chính tả**: Tránh từ dễ nhầm lẫn cách viết
    - Tránh: `giành/dành`, `giả/dả`, `sắp xếp/sắp sếp`
    - Tránh các từ hay viết sai dấu: `dùi/duì`, `nghỉ/nghĩ`
  - **Không nhập nhằng cách đọc**: Tránh từ có nhiều cách phát âm vùng miền
    - Tránh: `vui vẻ` (v/d miền Nam), `rừng rậm` (r/g miền Nam)
    - Tránh: `trưa nắng` (tr/ch), `xoài sống` (s/x tùy vùng)
  - **Rõ nghĩa**: Mỗi từ ghép chỉ có một nghĩa duy nhất, không đa nghĩa gây hiểu lầm
  - **Dễ viết**: Tránh từ có cấu trúc phụ âm phức tạp hoặc ít gặp
  - **Trung tính về nội dung**: Tránh từ liên quan đến:
    - Chính trị, tôn giáo, sắc tộc
    - Bạo lực, vũ khí, ma túy
    - Tục tĩu, nhạy cảm giới tính
    - Từ mang nghĩa tiêu cực hoặc xúc phạm

## Tính năng chính

### 1. Sinh mnemonic

```python
from mnemonic_vi import Mnemonic

m = Mnemonic("vietnamese")
words = m.generate(strength=128)  # 12 từ ghép
# "hoa-sen biển-xanh cánh-đồng mặt-trời ..."
```

### 2. Validate mnemonic

```python
is_valid = m.validate(words)
```

### 3. Mnemonic sang seed

```python
seed = m.to_seed(words, passphrase="")
```

### 4. Mnemonic sang entropy và ngược lại

```python
entropy = m.to_entropy(words)
words = m.from_entropy(entropy)
```

## Cấu trúc dự án

Dự án gồm 2 phần trong cùng một repo. Thư mục `dev/` được thêm vào `.gitignore` để không public.

```
mnemonic-vi/
├── .gitignore                   # Chứa dev/ để không public
├── README.md
├── LICENSE
├── project-spec.md
├── working-job.md
├── log.md
├── wordlists/
│   └── vietnamese.txt           # 2048 từ ghép tiếng Việt
├── docs/
│   ├── wordlist-criteria.md     # Tiêu chí chọn từ
│   └── bip39-vietnamese.md      # Đặc tả BIP-39 cho tiếng Việt
└── dev/                         # PRIVATE - không public
    ├── pyproject.toml
    ├── src/
    │   └── mnemonic_vi/
    │       ├── __init__.py
    │       ├── mnemonic.py      # Logic chính sinh/validate mnemonic
    │       └── wordlist.py      # Load và quản lý wordlist
    ├── scripts/
    │   └── generate_wordlist.py # Script sinh/lọc/sắp xếp wordlist
    ├── data/
    │   ├── raw_words.txt        # Nguồn từ thô trước khi lọc
    │   └── excluded_words.txt   # Từ bị loại và lý do
    ├── tests/
    │   ├── test_mnemonic.py
    │   ├── test_wordlist.py
    │   └── test_vectors.py
    └── examples/
        └── basic_usage.py
```

## Bảng ánh xạ Strength - Số từ

| Entropy (bits) | Checksum (bits) | Tổng (bits) | Số từ ghép |
| -------------- | --------------- | ----------- | ---------- |
| 128            | 4               | 132         | 12         |
| 160            | 5               | 165         | 15         |
| 192            | 6               | 198         | 18         |
| 224            | 7               | 231         | 21         |
| 256            | 8               | 264         | 24         |

## Dependencies

- Không phụ thuộc thư viện bên ngoài cho logic core (chỉ dùng stdlib)
- `hashlib` (stdlib) - SHA-256, PBKDF2
- `secrets` (stdlib) - sinh entropy an toàn
- Dev dependencies: `pytest`, `ruff`

## Bảo mật

- Sử dụng `secrets` thay vì `random` để sinh entropy
- PBKDF2-HMAC-SHA512 với 2048 iterations để tạo seed từ mnemonic
- Không log hoặc cache mnemonic/seed ra disk
- Xóa dữ liệu nhạy cảm khỏi bộ nhớ khi không còn sử dụng
