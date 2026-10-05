# 🏨 SE100 — Hotel Booking Platform
### Nền Tảng Đặt Phòng Khách Sạn Trực Tuyến

<p align="left">
  <img src="https://img.shields.io/badge/NestJS-E0234E?style=for-the-badge&logo=nestjs&logoColor=white" alt="NestJS" />
  <img src="https://img.shields.io/badge/Vue.js-4FC08D?style=for-the-badge&logo=vue.js&logoColor=white" alt="Vue 3" />
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/PostgreSQL-4169E1?style=for-the-badge&logo=postgresql&logoColor=white" alt="PostgreSQL" />
  <img src="https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white" alt="Tailwind CSS" />
  <img src="https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/License-MIT-green.svg?style=for-the-badge" alt="License" />
</p>

---

### 📋 Thông Tin Dự Án

| Hạng mục | Chi tiết |
| :--- | :--- |
| **Môn học** | SE100 |
| **Chủ dự án** | **Thinh Phat Ho** ([@miyxotkem](https://github.com/miyxotkem)) |
| **Kiến trúc** | Monorepo (NestJS Backend + Vue 3 Frontend + PostgreSQL 16) |
| **Kho lưu trữ** | [miyxotkem/SE100_DatPhongKhachSan](https://github.com/miyxotkem/SE100_DatPhongKhachSan) |

---

## 📑 Mục Lục

1. [Kiến Trúc Công Nghệ (Tech Stack)](#1-kiến-trúc-công-nghệ-tech-stack)
2. [Cấu Trúc Thư Mục Monorepo](#2-cấu-trúc-thư-mục-monorepo)
3. [Hướng Dẫn Cài Đặt & Khởi Chạy (Local Development)](#3-hướng-dẫn-cài-đặt--khởi-chạy-local-development)
4. [Quy Chuẩn Làm Việc Git & GitHub](#4-quy-chuẩn-làm-việc-git--github)
   - [4.1. Ba Điều Cấm](#41-ba-điều-cấm)
   - [4.2. Quy Tắc Đặt Tên Nhánh](#42-quy-tắc-đặt-tên-nhánh)
   - [4.3. Quy Chuẩn Commit Message](#43-quy-chuẩn-commit-message)
   - [4.4. Quy Trình 6 Bước Làm Việc Hằng Ngày](#44-quy-trình-6-bước-làm-việc-hằng-ngày)
   - [4.5. Hướng Dẫn Xử Lý Xung Đột (Conflict)](#45-hướng-dẫn-xử-lý-xung-đột-conflict)
   - [4.6. Phân Vùng Quyền Hạn Trong Monorepo](#46-phân-vùng-quyền-hạn-trong-monorepo)

---

## 1. Kiến Trúc Công Nghệ (Tech Stack)

| Lĩnh vực | Công nghệ / Thư viện | Mô tả vai trò |
| :--- | :--- | :--- |
| **Backend** | [NestJS](https://nestjs.com/) (TypeScript) | Xây dựng RESTful API, Global Exception Filters, Transform Interceptors. |
| **Database & ORM** | [PostgreSQL 16](https://www.postgresql.org/) + [Prisma ORM](https://www.prisma.io/) | Cơ sở dữ liệu quan hệ chuẩn 3NF, quản lý schema và migrations. |
| **Authentication** | JWT, Passport-JWT, Bcrypt | Xác thực bảo mật người dùng, phân quyền truy cập. |
| **Media Storage** | [Cloudinary](https://cloudinary.com/) SDK | Lưu trữ đám mây và tối ưu hoá ảnh đại diện, ảnh phòng, ảnh khách sạn. |
| **Frontend** | [Vue 3](https://vuejs.org/) + [Vite](https://vitejs.dev/) | Giao diện hiện đại sử dụng Composition API (`<script setup lang="ts">`). |
| **Styling** | [Tailwind CSS](https://tailwindcss.com/) | Thiết kế giao diện trực quan, tối ưu hiển thị responsive đa màn hình. |
| **State & Route** | [Pinia](https://pinia.vuejs.org/) + [Vue Router 4](https://router.vuejs.org/) | Quản lý luồng trạng thái tập trung và định tuyến bảo vệ màn hình. |
| **Bản đồ** | [Mapbox GL JS](https://www.mapbox.com/) | Hiển thị toạ độ khách sạn, hỗ trợ tìm kiếm phòng theo vị trí địa lý. |
| **CI/CD & DevOps** | Docker Compose, GitHub Actions | Container hoá database môi trường local; CI tự động kiểm tra build BE/FE. |

---

## 2. Cấu Trúc Thư Mục Monorepo

```plaintext
SE100_DatPhongKhachSan/
├── .github/
│   ├── workflows/
│   │   └── ci.yml               # GitHub Actions CI tự động kiểm tra build BE & FE
│   └── pull_request_template.md # Mẫu Pull Request chuẩn dự án
├── backend/                     # Mã nguồn Backend (NestJS + Prisma)
│   ├── prisma/
│   │   └── schema.prisma        # Lược đồ CSDL chi tiết các bảng
│   └── src/
│       ├── common/              # Bộ lọc ngoại lệ, interceptor, upload Cloudinary
│       ├── modules/             # Các module nghiệp vụ (auth, hotels, rooms, bookings)
│       ├── prisma/              # PrismaService kết nối database
│       └── main.ts              # Entrypoint (CORS, ValidationPipe, Prefix api/v1)
├── frontend/                    # Mã nguồn Frontend (Vue 3 + Vite + Tailwind)
│   └── src/
│       ├── components/common/   # Các UI component tái sử dụng (BaseButton, BaseInput)
│       ├── layouts/             # Bố cục giao diện (MainLayout & BookingLayout)
│       ├── router/              # Thiết lập route và guard phân quyền
│       ├── services/            # Axios Client tích hợp Interceptor
│       └── views/               # Landing, Login, Register, Dashboard, Hotels, Booking
├── docker-compose.yml           # Cấu hình chạy PostgreSQL 16 (Port 5435)
├── .env.example                 # File mẫu cấu hình biến môi trường
├── .gitignore                   # Cấu hình bỏ qua tệp tin rác
└── README.md                    # Tài liệu hướng dẫn & quy chuẩn dự án
```

---

## 3. Hướng Dẫn Cài Đặt & Khởi Chạy (Local Development)

### 📌 Tổng quan các cổng dịch vụ (Ports)
| Dịch vụ | Cổng (Port) | Đường dẫn mặc định |
| :--- | :--- | :--- |
| **Frontend Web** | `5173` | `http://localhost:5173` |
| **Backend API** | `3000` | `http://localhost:3000/api/v1` |
| **PostgreSQL Database** | `5435` | Chạy nền qua Docker (tránh trùng cổng mặc định 5432 & 5434) |

---

### Các bước cài đặt chi tiết:

#### 🔹 Bước 1: Khởi tạo biến môi trường
Sao chép cấu hình mẫu `.env.example` sang file `.env` ở cả thư mục gốc và thư mục `backend/`:
```bash
cp .env.example .env
cp .env.example backend/.env
```

#### 🔹 Bước 2: Khởi động Database qua Docker
Đảm bảo **Docker Desktop** đã được mở, sau đó khởi chạy container PostgreSQL:
```bash
docker compose up -d
```

#### 🔹 Bước 3: Cài đặt và chạy Backend (NestJS)
Mở một cửa sổ Terminal và tiến hành cài đặt thư viện, generate Prisma schema và khởi động server:
```bash
cd backend
npm install
npx prisma generate
npx prisma migrate dev
npm run start:dev
```
> API Server sẽ sẵn sàng phục vụ tại: `http://localhost:3000/api/v1`

#### 🔹 Bước 4: Cài đặt và chạy Frontend (Vue 3)
Mở một cửa sổ Terminal riêng biệt cho Frontend:
```bash
cd frontend
npm install
npm run dev
```
> Truy cập giao diện ứng dụng tại: `http://localhost:5173`

---

## 4. Quy Chuẩn Làm Việc Git & GitHub

Nhằm đảm bảo an toàn cho mã nguồn và phòng tránh tối đa xung đột (conflict), tất cả thành viên **bắt buộc tuân thủ 100%** các quy định dưới đây.

### 4.1. Ba Điều Cấm

> [!CAUTION]
> 1. **TUYỆT ĐỐI KHÔNG** `git push` trực tiếp lên nhánh `main` hoặc nhánh tích hợp `feat/<module>`. Mọi thay đổi đều phải thông qua Pull Request (PR).
> 2. **TUYỆT ĐỐI KHÔNG** commit các file nhạy cảm (`.env`), file rác của hệ điều hành, hay thư mục phụ thuộc (`node_modules/`, `dist/`).
> 3. **TUYỆT ĐỐI KHÔNG** tự ý can thiệp vào mã nguồn module của người khác khi chưa có sự thống nhất trước.

---

### 4.2. Quy Tắc Đặt Tên & Phân Cấp Nhánh

Dự án áp dụng mô hình **Feature Integration Branching** với **`main`** là nhánh mặc định (production-ready). Mỗi tính năng lớn/module được phân cấp rõ ràng thành 2 tầng nhánh:

```mermaid
flowchart TD
    MAIN["main (Nhánh chính mặc định - Production)"]
    MOD["feat/&lt;tên-module&gt; (Nhánh tích hợp module)"]
    FE["feat/fe-&lt;tên-module&gt; (Frontend Dev)"]
    BE["feat/be-&lt;tên-module&gt; (Backend Dev)"]

    MAIN -- "1. Tách nhánh module" --> MOD
    MOD -- "2. Tách nhánh FE" --> FE
    MOD -- "2. Tách nhánh BE" --> BE
    FE -- "3. PR: Merge FE vào module" --> MOD
    BE -- "3. PR: Merge BE vào module" --> MOD
    MOD -- "4. Kiểm thử tích hợp OK -> PR merge vào main" --> MAIN
```

#### Bảng quy chuẩn đặt tên nhánh:

| Cấp nhánh | Định dạng đặt tên | Tách từ nhánh | Mục đích / Ví dụ |
| :--- | :--- | :--- | :--- |
| **Nhánh chính** | `main` | - | Nhánh mặc định, chứa mã nguồn ổn định nhất đã hoàn thiện tích hợp. |
| **Nhánh Module** | `feat/<tên-module>` | `main` | Nhánh gom và tích hợp chung của module (Ví dụ: `feat/hotels`, `feat/booking`, `feat/auth`). |
| **Backend Dev** | `feat/be-<tên-module>` | `feat/<tên-module>` | Backend triển khai API, DB migration (Ví dụ: `feat/be-hotels`, `feat/be-booking`). |
| **Frontend Dev** | `feat/fe-<tên-module>` | `feat/<tên-module>` | Frontend xây dựng UI, state, gọi API (Ví dụ: `feat/fe-hotels`, `feat/fe-booking`). |
| **Sửa lỗi Module** | `fix/fe-<lỗi>` / `fix/be-<lỗi>` | `feat/<tên-module>` | Sửa lỗi phát sinh trong quá trình tích hợp module. |
| **Hotfix khẩn cấp** | `hotfix/<tên-lỗi>` | `main` | Vá lỗi nghiêm trọng trực tiếp trên môi trường chạy thực tế. |

---

### 4.3. Quy Chuẩn Commit Message

Sử dụng định dạng **Conventional Commits** để thể hiện rõ ràng mục đích thay đổi:

| Tiền tố | Mô tả mục đích | Ví dụ minh họa |
| :--- | :--- | :--- |
| `feat(scope)` | Bổ sung tính năng mới | `feat(auth): thêm api đăng nhập bằng jwt` |
| `fix(scope)` | Vá lỗi hệ thống | `fix(booking): sửa lỗi tính toán ngày nhận/trả phòng` |
| `style(scope)` | Căn chỉnh CSS, giao diện (không đổi logic) | `style(hotel-card): chuẩn hóa khoảng cách hiển thị giá` |
| `refactor(scope)` | Tái cấu trúc mã nguồn, dọn dẹp logic | `refactor(rooms): tối ưu truy vấn lấy danh sách phòng trống` |
| `docs(scope)` | Bổ sung, điều chỉnh tài liệu | `docs(readme): làm mới hướng dẫn và quy chuẩn dự án` |

---

### 4.4. Quy Trình 6 Bước Làm Việc Hằng Ngày

Khi nhận nhiệm vụ cho một module mới, các thành viên thực hiện theo quy trình sau:

```mermaid
graph LR
    A["1. Đồng bộ feat/module"] --> B["2. Tạo nhánh FE / BE"]
    B --> C["3. Code & Atomic Commit"]
    C --> D["4. Đồng bộ nhánh module"]
    D --> E["5. PR vào feat/module"]
    E --> F["6. Tích hợp & PR vào main"]
```

#### 1️⃣ Bước 1: Khởi tạo hoặc kéo nhánh module chung (`feat/<tên-module>`)
- Nếu nhánh module chưa có trên GitHub, Tech Lead/đại diện tạo từ `main`:
  ```bash
  git checkout main
  git pull origin main
  git checkout -b feat/booking
  git push -u origin feat/booking
  ```
- Nếu nhánh module đã có sẵn trên GitHub:
  ```bash
  git checkout main
  git pull origin main
  git fetch origin
  git checkout feat/booking
  git pull origin feat/booking
  ```

#### 2️⃣ Bước 2: Tách nhánh làm việc con (`fe` hoặc `be`) từ nhánh module
```bash
# Đối với Frontend:
git checkout feat/booking
git checkout -b feat/fe-booking

# Đối với Backend:
git checkout feat/booking
git checkout -b feat/be-booking
```

#### 3️⃣ Bước 3: Code và Commit từng phần nhỏ (Atomic Commits)
Không dồn toàn bộ công việc vào một commit lớn. Hoàn tất từng hàm/component -> Kiểm tra chạy ổn định -> Commit ngay:
```bash
git status
git add src/views/HotelListView.vue
git commit -m "feat(hotels): hoàn thiện giao diện danh sách khách sạn"
```

#### 4️⃣ Bước 4: Kéo cập nhật từ nhánh module chung trước khi mở PR *(Tránh conflict)*
```bash
git checkout feat/booking
git pull origin feat/booking
git checkout feat/fe-booking
git merge feat/booking
```
* **Không xung đột:** Git sẽ tự động hợp nhất an toàn.
* **Có xung đột:** Tham khảo ngay mục [4.5](#45-hướng-dẫn-xử-lý-xung-đột-conflict) bên dưới.

#### 5️⃣ Bước 5: Đẩy nhánh lên GitHub và tạo Pull Request vào nhánh Module
```bash
git push -u origin feat/fe-booking
```
1. Truy cập repo: [miyxotkem/SE100_DatPhongKhachSan](https://github.com/miyxotkem/SE100_DatPhongKhachSan).
2. Nhấn nút **Compare & pull request**.
3. **⚠️ Rất quan trọng - Cấu hình nhánh đích:**
   - `base: feat/booking` ⬅️ `compare: feat/fe-booking` (hoặc `feat/be-booking`).
4. Điền mô tả PR và đợi CI kiểm tra hoàn tất -> Tiến hành Review & Merge vào nhánh `feat/booking`.

#### 6️⃣ Bước 6: Kiểm thử tích hợp toàn diện & Merge vào `main`
Khi cả **Frontend** và **Backend** đều đã được merge vào `feat/booking`:
1. Hai bên phối hợp chạy kiểm thử tích hợp (End-to-End): giao diện gọi API mượt mà, không phát sinh lỗi.
2. Tạo Pull Request tổng để đưa module vào nhánh chính:
   - `base: main` ⬅️ `compare: feat/booking`.
3. Sau khi Tech Lead review và approve, thực hiện **Squash and merge** vào `main`.
4. Dọn dẹp các nhánh tính năng đã hoàn thành trên local:
   ```bash
   git checkout main
   git pull origin main
   git branch -d feat/fe-booking
   git branch -d feat/booking
   ```

---

### 4.5. Hướng Dẫn Xử Lý Xung Đột (Conflict)

Khi chạy `git merge feat/<tên-module>` gặp thông báo `CONFLICT (content): Merge conflict in ...`:

1. **Giữ bình tĩnh:** Mở VS Code, các file có xung đột sẽ được đánh dấu màu cam/đỏ.
2. **Kiểm tra vùng xung đột:**
   - `Current Change`: Code trên nhánh bạn đang viết.
   - `Incoming Change`: Code mới từ nhánh module chung do thành viên khác vừa cập nhật.
3. **Lựa chọn xử lý:** Sử dụng công cụ của VS Code (`Accept Current`, `Accept Incoming`, hoặc `Accept Both`).
4. **Nguyên tắc phối hợp:** Nếu xung đột liên quan đến logic của người khác, hãy trao đổi trực tiếp với tác giả đoạn code đó; tuyệt đối không tự ý xoá code của đồng đội.
5. **Hoàn tất merge:**
   ```bash
   npm run build   # Kiểm tra tính toàn vẹn cú pháp sau khi giải quyết conflict
   git add .
   git commit -m "merge: giải quyết conflict với nhánh feat/booking"
   git push origin feat/fe-booking
   ```

---

### 4.6. Phân Vùng Quyền Hạn Trong Monorepo

* **Backend Dev:** Chỉ thực hiện thay đổi trong phạm vi thư mục `backend/`.
* **Frontend Dev:** Chỉ thực hiện thay đổi trong phạm vi thư mục `frontend/`.
* **Quản lý Dependencies:** Khi cài thêm gói NPM mới, phải thông báo lên nhóm để các thành viên khác chạy lại `npm install` khi đồng bộ code.
* **Cấu hình dùng chung:** Mọi thay đổi ở root (`docker-compose.yml`, `.github/`, `.env.example`) phải được thống nhất qua **Tech Lead**.
