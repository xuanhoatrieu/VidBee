# 🔄 Hướng dẫn cập nhật từ Upstream (nexmoe/VidBee)

## Cấu trúc Git Remotes

| Remote | URL | Mục đích |
|--------|-----|----------|
| `origin` | `https://github.com/xuanhoatrieu/VidBee.git` | Fork của bạn |
| `upstream` | `https://github.com/nexmoe/VidBee` | Repo gốc của tác giả |

## Các branch

| Branch | Nội dung |
|--------|----------|
| `main` | Giữ nguyên bản gốc, sync 1:1 với upstream |
| `custom/my-changes` | Bản tùy chỉnh của bạn (mọi thay đổi nằm ở đây) |

---

## 📥 Khi tác giả cập nhật bản mới

### Bước 1: Fetch bản mới từ upstream

```bash
cd ~/VidBee
git fetch upstream
```

### Bước 2: Cập nhật branch `main`

```bash
git checkout main
git merge upstream/main --ff-only
git push origin main
```

### Bước 3: Rebase custom changes lên bản mới

```bash
git checkout custom/my-changes
git rebase main
```

### Bước 4: Xử lý xung đột (nếu có)

Nếu có conflict, Git sẽ báo file nào bị xung đột:

```bash
# Xem file conflict
git status

# Mở file, sửa conflict (phần giữa <<<< và >>>>)
# Sau khi sửa xong:
git add <tên-file-đã-sửa>
git rebase --continue
```

Nếu quá phức tạp, hủy và thử lại:
```bash
git rebase --abort
```

### Bước 5: Push lên fork

```bash
git push origin custom/my-changes --force-with-lease
```

> ⚠️ Dùng `--force-with-lease` vì rebase viết lại lịch sử. Chỉ an toàn vì đây là branch riêng của bạn.

### Bước 6: Rebuild Docker

```bash
docker compose up -d --build api web
```

---

## 🚀 Quy trình tóm gọn (copy-paste)

```bash
# Cập nhật từ upstream và rebase custom changes
cd ~/VidBee
git fetch upstream
git checkout main && git merge upstream/main --ff-only && git push origin main
git checkout custom/my-changes && git rebase main
# Nếu có conflict → sửa → git add . → git rebase --continue
git push origin custom/my-changes --force-with-lease
docker compose up -d --build api web
```

---

## ⚠️ Lưu ý quan trọng

1. **Luôn làm việc trên branch `custom/my-changes`**, KHÔNG commit trực tiếp vào `main`
2. **`main` luôn giữ sạch** = bản gốc upstream, để merge dễ dàng
3. **Các file bạn đã sửa** (có khả năng conflict cao khi upstream update):
   - `packages/downloader-core/src/yt-dlp-args.ts` (force MP4)
   - `packages/downloader-core/src/downloader-core.ts` (perEntryFormats)
   - `apps/web/src/components/download/playlist-download.tsx` (per-video quality)
   - `apps/web/src/components/download/download-dialog.tsx` (UI changes)
   - `apps/web/src/lib/web-settings.ts` (default 1080p)

4. **Nếu cần AI giúp merge**, gõ `/debug` và dán output của `git status` + nội dung conflict
