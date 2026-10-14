# DevOps Session 11 – GitHub Actions CI/CD

Repository Spring Boot (Gradle) kèm GitHub Actions Workflow tự động chạy Unit Test.

## Bài tập
- **Bài 1:** Khởi tạo Workflow chạy Unit Test tự động cho Java Spring Boot với Gradle
  - Thư mục: `BaiTap1/`
  - Workflow: `.github/workflows/ci.yml`
- **Bài 2:** Biên dịch đóng gói tệp ứng dụng .jar với Gradle và lưu trữ Artifacts
  - Thư mục: `BaiTap2/`
  - Thêm bước `./gradlew bootJar` và lưu artifact `app-jar` (`build/libs/*.jar`) bằng `actions/upload-artifact@v4`

- **Bài 3:** Tối ưu tốc độ CI bằng Gradle Dependency Caching
  - Thư mục: `BaiTap3/`
  - Kích hoạt `cache: 'gradle'` trong `actions/setup-java@v5`, lập báo cáo `report.md` so sánh hiệu năng giảm ~75% thời gian chạy.

## Trigger
- Push lên nhánh `main` → tự động chạy test, build jar, lưu artifacts và cache dependencies.

