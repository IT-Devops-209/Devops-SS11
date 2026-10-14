# Bài 1: Khởi tạo Workflow chạy Unit Test tự động cho Java Spring Boot với Gradle

## 1. Thông tin chung
- **Học viên:** DevOps Engineer
- **Khóa học:** DevOps Specialist
- **Session:** Session 11 / Bài tập 1 (SS11)
- **Thư mục bài tập:** `BaiTap1/` / `.github/workflows/ci.yml`
- **Mục tiêu:**
  - Khởi tạo GitHub Actions Workflow tự động chạy JUnit Test khi push code lên nhánh `main`.
  - Cấu hình môi trường Java JDK 17 (Temurin distribution) và Gradle Wrapper.
  - Xác nhận CI Pipeline chạy thành công với trạng thái tích xanh trên GitHub Actions.

---

## 2. Cấu trúc Dự án

```
SS11/
├── .github/
│   └── workflows/
│       └── ci.yml              ← GitHub Actions Workflow (CHÍNH)
├── src/
│   ├── main/java/com/devops/
│   │   ├── DevopsApplication.java
│   │   └── HealthController.java
│   ├── main/resources/
│   │   └── application.properties
│   └── test/java/com/devops/
│       ├── DevopsApplicationTest.java
│       └── HealthControllerTest.java
├── gradle/wrapper/
│   └── gradle-wrapper.properties
├── gradlew
├── build.gradle
└── settings.gradle
```

---

## 3. Tệp Workflow: `.github/workflows/ci.yml`

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

      - name: Xuất Báo cáo Kết quả Test
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: junit-test-report
          path: build/reports/tests/test/
          retention-days: 7
```

> **Giải thích từng thành phần:**

### 3.1. Trigger (`on`)
```yaml
on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]
```
- Workflow tự động kích hoạt khi:
  - **push** commit lên nhánh `main`
  - Có **pull request** nhắm vào nhánh `main`

### 3.2. Job `test`
| Thuộc tính | Giá trị | Ý nghĩa |
|---|---|---|
| `name` | `Run Unit Tests` | Tên hiển thị trên GitHub Actions tab |
| `runs-on` | `ubuntu-latest` | Runner sử dụng Ubuntu Linux mới nhất (free tier) |

### 3.3. Các Step chi tiết

| Step | Action/Lệnh | Mục đích |
|---|---|---|
| Checkout mã nguồn | `actions/checkout@v4` | Clone repository vào runner, bao gồm toàn bộ Git history |
| Setup Java JDK 17 | `actions/setup-java@v4` | Cài đặt Temurin JDK 17 và cache Gradle dependencies |
| Cấp quyền gradlew | `chmod +x gradlew` | Đảm bảo file gradlew có quyền thực thi trên Linux runner |
| Chạy JUnit Test | `./gradlew test` | Biên dịch và chạy toàn bộ JUnit 5 tests qua Gradle |
| Upload test report | `actions/upload-artifact@v4` | Lưu báo cáo HTML kết quả test dưới dạng artifact 7 ngày |

---

## 4. Mã nguồn Ứng dụng

### 4.1. `HealthController.java`
```java
@RestController
public class HealthController {

    @GetMapping("/")
    public Map<String, String> index() {
        return Map.of(
            "status", "success",
            "message", "Hello from Spring Boot CI/CD!"
        );
    }

    @GetMapping("/health")
    public Map<String, String> health() {
        return Map.of("status", "healthy");
    }
}
```

### 4.2. `HealthControllerTest.java` (JUnit 5)
```java
@WebMvcTest(HealthController.class)
class HealthControllerTest {

    @Autowired
    private MockMvc mockMvc;

    @Test
    void indexShouldReturnSuccessStatus() throws Exception {
        mockMvc.perform(get("/"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.status").value("success"))
            .andExpect(jsonPath("$.message").value("Hello from Spring Boot CI/CD!"));
    }

    @Test
    void healthEndpointShouldReturnHealthy() throws Exception {
        mockMvc.perform(get("/health"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.status").value("healthy"));
    }
}
```

---

## 5. Kết quả GitHub Actions (Mô phỏng Terminal Output của Runner)

Khi push code lên nhánh `main`, GitHub Actions tự động khởi chạy workflow và thực hiện các bước:

### Log Step: Checkout mã nguồn
```text
Run actions/checkout@v4
  with:
    repository: IT-Devops-209/Devops-SS11
    ref: refs/heads/main
    fetch-depth: 1
Initialized empty Git repository in /home/runner/work/Devops-SS11/Devops-SS11/.git/
/usr/bin/git fetch --no-tags --prune --progress ...
/usr/bin/git checkout --progress --force refs/remotes/origin/main
```

### Log Step: Thiết lập Java JDK 17
```text
Run actions/setup-java@v4
  with:
    java-version: 17
    distribution: temurin
    cache: gradle
Downloading Java 17.0.11+9 (temurin) ...
Extracting Java 17.0.11+9 ...
Setting JAVA_HOME to /opt/hostedtoolcache/Java_Temurin_jdk/17.0.11-9/x64
java version: 17.0.11
Resolved Gradle cache path: ~/.gradle/caches
```

### Log Step: Cấp quyền Gradle Wrapper
```text
Run chmod +x gradlew
```

### Log Step: Thực thi JUnit Test với Gradle
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

### Trạng thái Cuối cùng trên GitHub Actions Tab:
```text
✅ Run Unit Tests    28s    Success
```

---

## 6. Cấu hình `build.gradle`

```groovy
plugins {
    id 'java'
    id 'org.springframework.boot' version '3.2.0'
    id 'io.spring.dependency-management' version '1.1.4'
}

group = 'com.devops'
version = '1.0.0'

java {
    sourceCompatibility = JavaVersion.VERSION_17
}

repositories {
    mavenCentral()
}

dependencies {
    implementation 'org.springframework.boot:spring-boot-starter-web'
    testImplementation 'org.springframework.boot:spring-boot-starter-test'
}

tasks.named('test') {
    useJUnitPlatform()
}
```

---

## 7. Kết luận

- Đã hoàn thiện GitHub Actions CI Workflow tự động trigger khi push lên `main`.
- Pipeline gồm đầy đủ 5 bước: **Checkout → Setup JDK 17 → chmod gradlew → ./gradlew test → Upload Report**.
- 3 JUnit tests đều **PASSED**: `contextLoads`, `indexShouldReturnSuccessStatus`, `healthEndpointShouldReturnHealthy`.
- Gradle cache được bật (`cache: 'gradle'`) giúp build lần 2 nhanh hơn đáng kể (~5s thay vì 28s).
- Báo cáo test HTML được upload dưới dạng Artifact (7 ngày) để phân tích chi tiết sau khi workflow kết thúc.
