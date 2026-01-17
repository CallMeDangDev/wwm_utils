# WWM Utils
Công cụ xử lý file game Where Winds Meet (WWM) - Binary parser và packer cho localization files.

## 🎯 Tính năng chính

### 1. 🔧 GitHub Workflow - Patch DB_MP (Khuyến nghị)

**Workflow này tự động patch file `db_mp` với hotfix translation mới nhất từ GitHub.**

#### 📋 Yêu cầu
- Fork repo này về tài khoản GitHub của bạn
- File `db_mp` từ game (nằm trong `LocalData/LocalDB/`)

#### 🚀 Cách sử dụng

1. **Fork repo này**
   ```
   Click nút "Fork" ở góc trên bên phải GitHub
   ```

2. **Vào tab Actions**
   ```
   Vào repo đã fork → Actions → "Patch DB_MP"
   ```

3. **Chạy workflow**
   - Click **"Run workflow"** (nút màu xanh)
   - Nhập **URL download file db_mp** của bạn
     - Có thể upload db_mp lên Mediafire, Dropbox, hoặc bất kỳ nơi nào có direct download link
     - Hoặc upload lên GitHub Release của repo fork
   - Click **"Run workflow"** để bắt đầu

4. **Lấy kết quả**
   - Workflow sẽ chạy khoảng 1-2 phút
   - Khi hoàn thành, vào **Releases** → tải file `db_mp` hoặc `db_mp.zip` mới nhất

### 2. Unpack/Pack Localize Map Files
- **Cách dùng**: Kéo thả file/folder vào executable
  - Kéo file map → Unpack ra JSON + raw tables
  - Kéa folder `output/{filename}` → Pack lại thành file binary
  - Kết quả được tạo trong thư mục `merged/`
  
- **⚠️ Lưu ý**: Một số map có file patch `_diff` hợp lệ. Tool chưa hỗ trợ merge patch, nên chỉ nên edit các map có patch file mặc định (1KB).

#### ⚙️ Workflow làm gì?
1. ✅ Tải file `db_mp` từ URL bạn cung cấp
2. ✅ Tải hotfix translation mới nhất từ [CallMeDangDev/WWM_VH](https://github.com/CallMeDangDev/WWM_VH)
3. ✅ Parse translations từ CSV
4. ✅ Patch vào bảng `translate_words_map_en` trong db_mp
5. ✅ Upload file đã patch lên GitHub Release của repo bạn

#### 📝 Lưu ý quan trọng
- File `db_mp` phải đến từ game (sau khi vào game ít nhất 1 lần)
- Không commit file `db_mp` của bạn vào repo (file lớn ~200MB)
- Kết quả có thể dùng ngay, copy vào `LocalData/LocalDB/` của game

## 🛠️ Build từ source

```bash
cargo build --release
```

Binary sẽ nằm trong `target/release/wwm_utils`

---

## 📖 Chi tiết kỹ thuật

### File Format Structure
- **MapHeader** (12 bytes): Magic `0xDEADBEEF`, version, entry count
- **Seek table**: Array of offsets to compressed blocks
- **Blocks**: BlockHeader + zstd-compressed table data
- **Tables**: Table 0 = raw data; tables 1+ = localization strings với hash-based lookup

### Core Workflow
1. **Unpack**: Read binary → extract zstd blocks → parse string tables → output JSON + raw tables
2. **Pack**: Read modified JSON + original tables → recompress → write binary

---

## 📄 License

MIT License - Xem file [LICENSE](LICENSE)