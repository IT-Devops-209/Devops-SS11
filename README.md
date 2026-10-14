# DevOps Session 11 – GitHub Actions CI/CD

Repository Spring Boot (Gradle) kèm GitHub Actions Workflow tự động chạy Unit Test.

## Bài tập
- **Bài 1:** Khởi tạo Workflow chạy Unit Test tự động cho Java Spring Boot với Gradle
  - Thư mục: `BaiTap1/`
  - Workflow: `.github/workflows/ci.yml`

## Trigger
- Push lên nhánh `main` → tự động chạy `./gradlew test`
