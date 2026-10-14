# Bài 4: Khai Báo GitHub Secrets và Cấu Hình Đăng Nhập Container Registry (GHCR)

## 1. Thông Tin Chung
- **Dự án:** Spring Boot 3.2.0 (Gradle)
- **Bài thực hành:** Bài 4 - Session 11 (SS11)
- **Thư mục:** `BaiTap4/`
- **Mục tiêu:**
  - Tạo Personal Access Token (classic) với quyền `write:packages` trên GitHub.
  - Khai báo Repository Secret an toàn mang tên `REGISTRY_TOKEN` trong GitHub Actions.
  - Bổ sung bước `docker/login-action@v3` vào workflow `ci.yml` để tự động xác thực đăng nhập vào GitHub Container Registry (`ghcr.io`).
  - Kiểm tra log runner xác nhận `Login Succeeded`.

---

## 2. Hướng Dẫn Từng Bước Thiết Lập

### Bước 1: Tạo GitHub Personal Access Token (classic)
1. Đăng nhập GitHub, nhấn vào ảnh đại diện góc trên cùng bên phải -> chọn **Settings**.
2. Cuộn xuống dưới cùng menu trái, chọn **Developer settings** -> **Personal access tokens** -> **Tokens (classic)**.
3. Chọn **Generate new token** -> **Generate new token (classic)**.
4. Điền các thông tin:
   - **Note:** `GHCR Deploy Token SS11`
   - **Expiration:** Chọn `90 days` hoặc tùy chọn theo nhu cầu.
   - **Scopes (Quyền hạn):** Tích chọn:
     - ✅ `write:packages` *(Tự động tích kèm `read:packages`)*: Cho phép upload container images lên GitHub Packages / GHCR.
     - ✅ `delete:packages` *(Tùy chọn)*.
5. Nhấn **Generate token** và **SAO CHÉP** chuỗi token (bắt đầu bằng `ghp_...`). *Lưu ý: Token chỉ hiển thị 1 lần duy nhất.*

---

### Bước 2: Khai Báo Repository Secret `REGISTRY_TOKEN`
1. Truy cập vào Repository bài tập: `https://github.com/IT-Devops-209/Devops-SS11`
2. Nhấn vào tab **Settings** của repository.
3. Ở menu bên trái, tìm mục **Security** -> chọn **Secrets and variables** -> **Actions**.
4. Nhấn nút xanh **New repository secret**.
5. Nhập:
   - **Name:** `REGISTRY_TOKEN`
   - **Secret:** Dán mã token `ghp_...` vừa tạo ở Bước 1.
6. Nhấn **Add secret** để hoàn tất.

> 🔒 **Tính an toàn của GitHub Secrets:** Sau khi lưu, giá trị token được mã hóa bảo mật bằng Libsodium sealing keys. Không một ai (kể cả chủ repository) có thể xem lại giá trị thô, chỉ có GitHub Actions runner mới có quyền giải mã trong quá trình chạy workflow.

---

## 3. Cấu Hình Workflow: `ci.yml`

Bước đăng nhập được tích hợp vào cuối Job kiểm thử và đóng gói:

```yaml
      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

### Giải thích các tham số:
| Tham số | Giá trị | Ý nghĩa |
|---|---|---|
| `registry` | `ghcr.io` | Địa chỉ GitHub Container Registry chính thức |
| `username` | `${{ github.actor }}` | Tự động lấy tên tài khoản GitHub đang kích hoạt workflow |
| `password` | `${{ secrets.REGISTRY_TOKEN }}` | Trích xuất token bảo mật từ Repository Secrets |

---

## 4. Tệp Workflow Hoàn Chỉnh (`BaiTap4/ci.yml` & `.github/workflows/ci.yml`)

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests & Build JAR & Login Registry
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v4

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v5
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew
        working-directory: BaiTap1

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test
        working-directory: BaiTap1

      - name: Xuất Báo cáo Kết quả Test (Upload Test Report)
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: junit-test-report
          path: BaiTap1/build/reports/tests/test/
          retention-days: 7

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar
        working-directory: BaiTap1

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: BaiTap1/build/libs/*.jar

      - name: Đăng nhập vào GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.REGISTRY_TOKEN }}
```

---

## 5. Kết Quả Kiểm Tra (Verification)

### Log thực tế từ GitHub Actions Runner:
```text
Run docker/login-action@v3
  with:
    registry: ghcr.io
    username: IT-Devops-209
    password: ***
    logout: true
Logging into ghcr.io...
Login Succeeded
```

> ✅ Chuỗi bí mật tự động được che dấu bằng dấu sao `***` để tránh lộ thông tin nhạy cảm trong console log. Dòng chữ `Login Succeeded` xác nhận runner đã xác thực thành công với GitHub Container Registry!

---

## 6. Hướng Dẫn Chụp Ảnh Màn Hình Nộp Bài

Để hoàn tất yêu cầu nộp bài, bạn chụp 2 ảnh màn hình:
1. **Ảnh 1: Mục Repository Secrets:**
   - Đường dẫn: `https://github.com/IT-Devops-209/Devops-SS11/settings/secrets/actions`
   - Hiển thị danh sách bí mật có mục: `REGISTRY_TOKEN` (Updated ... by you).
2. **Ảnh 2: Log thực thi trên Actions Tab:**
   - Đường dẫn: `https://github.com/IT-Devops-209/Devops-SS11/actions`
   - Nhấn vào run mới nhất -> Bước **"Đăng nhập vào GitHub Container Registry"** -> Hiển thị **`Login Succeeded`**.
