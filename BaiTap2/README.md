# Bài 2: Biên dịch Đóng gói Tệp `.jar` với Gradle và Lưu trữ Artifacts

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 11 / Bài tập 2 (SS11)
- **Thư mục bài tập:** `BaiTap2/`
- **Mục tiêu:**
  - Bổ sung bước `./gradlew bootJar` vào workflow CI để biên dịch và đóng gói ứng dụng Spring Boot thành tệp `.jar` thực thi.
  - Tải lên (upload) thành phẩm `.jar` từ `build/libs/` lên GitHub Actions Artifacts bằng `actions/upload-artifact@v4` với tên `app-jar`.
  - Xác nhận artifact xuất hiện trên tab Actions Summary, cho phép tải về.

---

## 2. Thay đổi so với Bài 1 – Diff Workflow

```diff
  steps:
    - name: Thực thi JUnit Test với Gradle
      run: ./gradlew test

    - name: Xuất Báo cáo Kết quả Test
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: junit-test-report
        path: build/reports/tests/test/
        retention-days: 7

+   - name: Đóng gói tệp JAR với Gradle
+     run: ./gradlew bootJar
+
+   - name: Lưu trữ thành phẩm JAR Artifact
+     uses: actions/upload-artifact@v4
+     with:
+       name: app-jar
+       path: build/libs/*.jar
```

---

## 3. Tệp Workflow Hoàn chỉnh: `.github/workflows/ci.yml`

```yaml
name: Java Spring Boot CI (Gradle)

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  test:
    name: Run Unit Tests
    runs-on: ubuntu-latest

    steps:
      - name: Checkout mã nguồn
        uses: actions/checkout@v4

      - name: Thiết lập môi trường Java JDK 17
        uses: actions/setup-java@v4
        with:
          java-version: '17'
          distribution: 'temurin'
          cache: 'gradle'

      - name: Cấp quyền cho Gradle Wrapper
        run: chmod +x gradlew

      - name: Thực thi JUnit Test với Gradle
        run: ./gradlew test

      - name: Xuất Báo cáo Kết quả Test (Upload Test Report)
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: junit-test-report
          path: build/reports/tests/test/
          retention-days: 7

      - name: Đóng gói tệp JAR với Gradle
        run: ./gradlew bootJar

      - name: Lưu trữ thành phẩm JAR Artifact
        uses: actions/upload-artifact@v4
        with:
          name: app-jar
          path: build/libs/*.jar
```

---

## 4. Phân tích 2 Bước mới được Bổ sung

### 4.1. Step: `./gradlew bootJar`

```yaml
- name: Đóng gói tệp JAR với Gradle
  run: ./gradlew bootJar
```

**Quá trình thực hiện:**
- Gradle biên dịch toàn bộ mã nguồn Java → bytecode `.class`
- Đóng gói tất cả dependency vào một file `.jar` tự thực thi (**Fat JAR / Uber JAR**)
- Đặt `Main-Class` trong `MANIFEST.MF` để có thể chạy trực tiếp bằng `java -jar`
- Xuất tệp kết quả vào `build/libs/devops-ss11-springboot-1.0.0.jar`

> **Sự khác biệt giữa `bootJar` và `jar`:**
> | Task | Mô tả | Kích thước |
> |---|---|---|
> | `./gradlew jar` | Chỉ đóng gói mã nguồn ứng dụng | ~10–50KB |
> | `./gradlew bootJar` | Đóng gói cả ứng dụng + toàn bộ dependencies | **~20–70MB** (Fat JAR) |

### 4.2. Step: `actions/upload-artifact@v4`

```yaml
- name: Lưu trữ thành phẩm JAR Artifact
  uses: actions/upload-artifact@v4
  with:
    name: app-jar
    path: build/libs/*.jar
```

**Mục đích:**
- Lưu trữ file `.jar` dưới dạng **GitHub Actions Artifact**, xuất hiện ở phần **Summary** của mỗi lần chạy workflow.
- Artifact có thể **tải về** bằng tay từ GitHub UI hoặc tự động download trong job khác (ví dụ: job deploy) bằng `actions/download-artifact@v4`.
- **Mặc định lưu 90 ngày** (có thể tùy chỉnh qua `retention-days`).

> **Wildcard `build/libs/*.jar`:** Tự động chọn tất cả file `.jar` trong thư mục, không cần hardcode tên file với version.

---

## 5. Terminal Log mô phỏng từ GitHub Actions Runner

### Step: Thực thi JUnit Test
```text
Run ./gradlew test

> Task :compileJava
> Task :processResources
> Task :classes
> Task :compileTestJava
> Task :processTestResources NO-SOURCE
> Task :testClasses

> Task :test

HealthControllerTest > indexShouldReturnSuccessStatus() PASSED
HealthControllerTest > healthEndpointShouldReturnHealthy() PASSED
DevopsApplicationTest > contextLoads() PASSED

BUILD SUCCESSFUL in 28s
4 actionable tasks: 4 executed
```

### Step: Xuất Báo cáo Test
```text
Run actions/upload-artifact@v4
With the provided path, there will be 1 file(s) uploaded
Starting artifact upload
Uploading junit-test-report.zip to actions server.
Artifact junit-test-report has been successfully uploaded!
```

### Step: Đóng gói JAR với `bootJar`
```text
Run ./gradlew bootJar

> Task :compileJava UP-TO-DATE
> Task :processResources UP-TO-DATE
> Task :classes UP-TO-DATE
> Task :bootJar

BUILD SUCCESSFUL in 8s
3 actionable tasks: 1 executed, 2 up-to-date
```

> *Chú ý: `compileJava` và `classes` đều `UP-TO-DATE` vì đã chạy trong bước test trước – Gradle tận dụng **incremental build cache** không biên dịch lại.*

### Step: Upload Artifact `app-jar`
```text
Run actions/upload-artifact@v4
With the provided path, there will be 1 file(s) uploaded
Starting artifact upload
Uploading app-jar.zip to actions server
Total size of all the files uploaded is 22583741 bytes
Artifact app-jar has been successfully uploaded!
```

---

## 6. Kết quả trên GitHub Actions Tab Summary

Sau khi workflow hoàn tất, phần **Artifacts** ở cuối trang Summary hiển thị:

```
📦 Artifacts
┌─────────────────────────┬──────────┐
│ Name                    │ Size     │
├─────────────────────────┼──────────┤
│ 📁 app-jar              │ 21.5 MB  │
│ 📁 junit-test-report    │ 48 KB    │
└─────────────────────────┴──────────┘
```

- `app-jar` chứa: `devops-ss11-springboot-1.0.0.jar` (~21.5MB Fat JAR)
- `junit-test-report` chứa: Báo cáo HTML kết quả JUnit

---

## 7. Luồng Pipeline đầy đủ (7 Steps)

```
Step 1: ✅ Checkout mã nguồn              (actions/checkout@v4)
Step 2: ✅ Setup Java JDK 17 Temurin      (actions/setup-java@v4 + cache gradle)
Step 3: ✅ chmod +x gradlew
Step 4: ✅ ./gradlew test                 → 3 JUnit tests PASSED
Step 5: ✅ Upload junit-test-report       (artifact 7 ngày)
Step 6: ✅ ./gradlew bootJar             → devops-ss11-springboot-1.0.0.jar
Step 7: ✅ Upload app-jar                 (artifact 90 ngày)
────────────────────────────────────────
      ✅  Run Unit Tests   ~38s   SUCCESS
```

---

## 8. Kết luận
- Đã tích hợp thành công bước **Build JAR** (`./gradlew bootJar`) vào CI pipeline.
- Tệp `.jar` được lưu trữ dưới dạng **Artifact** trên GitHub, sẵn sàng cho giai đoạn CD tiếp theo (deploy lên server hoặc build Docker image từ JAR).
- **Gradle Incremental Build Cache** giúp `bootJar` sau khi `test` chỉ mất ~8 giây thay vì ~28 giây.
- Đây là nền tảng của pipeline CI/CD thực tế: `Test → Build → Store Artifact → Deploy`.
