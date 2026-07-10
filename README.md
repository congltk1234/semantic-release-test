# Semantic Release — Demo triển khai với GitHub Actions

## Các file trong bộ này
- `.releaserc.json` — cấu hình semantic-release (branch, plugin nào chạy, thứ tự chạy)
- `.github/workflows/release.yml` — workflow tự động chạy khi push lên `main`
- `package.json` — khai báo các plugin cần cài (devDependencies)

## Các bước triển khai vào repo thật

### 1. Copy file vào repo
Copy `.releaserc.json` và thư mục `.github/workflows/` vào repo của bạn.
Thêm các `devDependencies` trong `package.json` mẫu vào `package.json` hiện tại của bạn, rồi:

```bash
npm install
```

### 2. Không cần tạo secret gì thêm cho GitHub Release
`secrets.GITHUB_TOKEN` được GitHub tự cấp cho mỗi workflow run, chỉ cần đảm bảo
**Settings → Actions → General → Workflow permissions** đặt là
**"Read and write permissions"** (hoặc dùng `permissions:` như trong file mẫu, cách này ưu tiên hơn).

### 3. Nếu muốn publish lên npm
- Đổi `"npmPublish": false` → `true` trong `.releaserc.json`
- Tạo secret `NPM_TOKEN` trong repo (Settings → Secrets → Actions)
- Bỏ comment dòng `NPM_TOKEN` trong `release.yml`

### 4. Dùng Conventional Commits khi commit
```bash
git commit -m "feat: thêm chức năng đăng nhập"     # → minor bump
git commit -m "fix: sửa lỗi validate email"         # → patch bump
git commit -m "feat!: đổi cấu trúc API response

BREAKING CHANGE: response trả về object thay vì array" # → major bump
```

Gợi ý: cài thêm `commitlint` + `husky` để chặn commit sai format ngay từ local.

### 5. Push lên main và kiểm tra
Push commit theo chuẩn trên lên nhánh `main` → tab **Actions** sẽ chạy job `release`
→ nếu có commit đủ điều kiện release, sẽ tự động:
- Tính version mới
- Cập nhật `CHANGELOG.md`
- Tạo Git tag + GitHub Release
- (tuỳ chọn) publish npm

Nếu không có commit nào đủ điều kiện (ví dụ chỉ có `docs:`, `chore:`), semantic-release
sẽ log "no release" và không làm gì cả — đây là hành vi bình thường.

## Test cục bộ (dry-run) trước khi đẩy lên CI
```bash
npx semantic-release --dry-run --no-ci
```
Lệnh này cho biết version tiếp theo sẽ là gì mà không thực sự publish/tag.

## Lỗi thường gặp
| Lỗi | Nguyên nhân |
|---|---|
| `ENOGITHEAD` / không tính được version | Thiếu `fetch-depth: 0` khi checkout |
| `EGITNOPERMISSION` | Chưa cấp `contents: write` cho `GITHUB_TOKEN` |
| Không có release nào được tạo | Commit không theo Conventional Commits, hoặc chỉ có `chore/docs/style` |
| Chạy release nhưng bị loop vô hạn | Thiếu `[skip ci]` trong message của `@semantic-release/git` |
