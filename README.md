<div align="center">

<a href="https://shopaipro.tech"><img src="docs/assets/shopaipro-logo.svg" width="112" height="112" alt="Logo SHOP AI PRO"></a>

# SHOP AI PRO

**Nền tảng cung cấp dịch vụ AI, quản lý API key, credit và điều phối model.**

Storefront · API Gateway · Quản lý API key, credit & usage

[![Website](https://img.shields.io/badge/Website-shopaipro.tech-1689FF?style=flat-square)](https://shopaipro.tech)
[![Tài liệu](https://img.shields.io/badge/Docs-Hướng_dẫn-2677FF?style=flat-square)](https://shopaipro.tech/docs)
[![Repository](https://img.shields.io/badge/Repository-Overview_only-64748B?style=flat-square)](#về-repository-này)

![Go](https://img.shields.io/badge/Go-1.27-00ADD8?style=flat-square&logo=go&logoColor=white)
![Next.js](https://img.shields.io/badge/Next.js-16-000000?style=flat-square&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React-19-61DAFB?style=flat-square&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-5.9-3178C6?style=flat-square&logo=typescript&logoColor=white)
![PostgreSQL](https://img.shields.io/badge/PostgreSQL-17-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Compose-2496ED?style=flat-square&logo=docker&logoColor=white)

[**Website ↗**](https://shopaipro.tech) · [Giới thiệu](#giới-thiệu) · [Tính năng](#tính-năng) · [Kiến trúc](#kiến-trúc) · [Chạy local](#chạy-local) · [API](#api-và-xác-thực) · [Triển khai](#triển-khai-và-vận-hành) · [Tài liệu](#tài-liệu)

</div>

---

## Giới thiệu

> **Repository giới thiệu:** repo công khai này chỉ chứa README và logo thương hiệu, không kèm source code. Các đường dẫn file, cấu trúc thư mục và lệnh phát triển bên dưới mô tả mã nguồn nội bộ; chúng không có sẵn để tải từ repo này. Website: **[https://shopaipro.tech](https://shopaipro.tech)**.

**SHOP AI PRO** kết hợp storefront, tài khoản khách hàng, đơn hàng và quyền truy cập model AI trong một hệ thống. Khách hàng có thể xem sản phẩm, theo dõi đơn hàng, quản lý API key, credit và mức sử dụng; quản trị viên quản lý nội dung, sản phẩm, thanh toán và các hoạt động hỗ trợ.

Mã nguồn dự án được tổ chức thành **giao diện web**, **các dịch vụ Go độc lập**, công cụ chuyển đổi từ backend Python và cấu hình triển khai. 

Mục tiêu của kiến trúc là tách các miền nghiệp vụ, giữ giao dịch và số dư có thể đối chiếu, đồng thời cung cấp một cổng truy cập model thống nhất cho ứng dụng và công cụ lập trình.

> **Phạm vi README:** tài liệu giới thiệu mã nguồn và hướng dẫn phát triển. Hệ thống có nhiều tích hợp và cấu hình theo môi trường; chạy riêng frontend hoặc một dịch vụ Go không đồng nghĩa khởi động toàn bộ nền tảng. Trạng thái triển khai và các bước vận hành hiện hành được ghi trong tài liệu VPS (`deploy/azure/OWNER_VPS.md`).

## Tính năng

Các nhóm chức năng có mã nguồn trong dự án; khả năng sử dụng thực tế phụ thuộc cấu hình, feature flag, quyền tài khoản và dịch vụ tích hợp.

| Nhóm | Nội dung |
| --- | --- |
| **Storefront** | Danh mục sản phẩm, phân loại, thông tin gói dịch vụ, giá và nội dung công khai |
| **Tài khoản** | Đăng ký/đăng nhập, hồ sơ, xác thực Google/Firebase, phiên đăng nhập và xác thực hai bước theo cấu hình |
| **Thương mại** | Đơn hàng, giỏ hàng, coupon, ví, top-up, thanh toán PayOS/PayPal và đối soát |
| **API key & quyền sử dụng** | Cấp và quản lý key, entitlement, credit theo nhà cung cấp, usage và quota |
| **API Token** | Client key/secret key, phân quyền scope, kiểm tra trạng thái, hạn dùng và giới hạn sử dụng |
| **Model gateway** | Endpoint tương thích các giao thức model, chuyển tiếp request, streaming và điều phối tuyến |
| **Gói Limit** | Luồng tồn kho, lease, cấp quyền và đồng bộ tài khoản cho sản phẩm Limit |
| **AI Chat & media** | Giao diện trò chuyện, lịch sử và luồng media theo quyền/tích hợp được bật |
| **Hỗ trợ & tương tác** | Liên hệ, chat hỗ trợ, thông báo, hoạt động người dùng và nội dung hướng dẫn |
| **Quản trị** | Sản phẩm, nội dung, khách hàng, đơn hàng, thanh toán, key và tác vụ vận hành |
| **Client tooling** | Script cài đặt/cấu hình client AI, đồng bộ model và ShopAIPro Launcher |
| **Tác vụ nền** | Xử lý outbox, đồng bộ, khôi phục nghiệp vụ và gửi thông báo theo worker được chọn |

## Kiến trúc

```mermaid
flowchart TB
    U[Khách hàng / quản trị / API client] --> EDGE[Cloudflare / Caddy ingress]
    EDGE --> WEB[Next.js frontend]
    EDGE --> API[Go services theo miền]
    EDGE --> GW[Go model gateway]
    WEB --> API
    API --> PG[(PostgreSQL)]
    API --> RD[(Redis)]
    W[Go workers] --> PG
    W --> RD
    API --> NEW[Hệ thống: tài nguyên và quota]
    GW --> NEW
    GW --> ROUTER[Hệ thống: định tuyến provider]
    NEW --> ROUTER
    ROUTER --> PROVIDER[Nhà cung cấp model]
    API --> EXT[Firebase / PayOS / PayPal / email / lưu trữ]
```

Sơ đồ thể hiện quan hệ tổng quát. Tuyến cụ thể phụ thuộc model, loại key, entitlement và cấu hình upstream; không phải mọi request đều đi qua cùng một chuỗi.

### Dịch vụ

Backend dùng Fiber v3, pgx và go-redis. Mỗi dịch vụ chạy bằng executable riêng và có lifecycle, pool kết nối, dependency và route tương ứng.

| Dịch vụ | Cổng local mặc định | Trách nhiệm chính |
| --- | --- | --- |
| `identity` | `127.0.0.1:8082` | Danh tính, xác thực, phiên người dùng/quản trị |
| `catalog` | `127.0.0.1:8083` | Danh mục, sản phẩm và thông tin catalog |
| `content` | `127.0.0.1:8084` | Nội dung, hướng dẫn, FAQ, chính sách và thiết lập |
| `commerce` | `127.0.0.1:8085` | Đơn hàng, ví, thanh toán và nghiệp vụ thương mại |
| `provisioning` | `127.0.0.1:8086` | Key, entitlement, credit, usage và Limit |
| `gateway` | `127.0.0.1:8087` | Model gateway và AI Chat |
| `support` | `127.0.0.1:8088` | Liên hệ, hỗ trợ và tương tác |
| `adminops` | `127.0.0.1:8089` | Thao tác quản trị/vận hành |
| `worker` | Không có HTTP listener | Một loại công việc nền được cấu hình rõ ràng |
| `migrate` | Không có HTTP listener | Migration và các thao tác schema được chỉ định |
| `backupcheck` | Không có HTTP listener | Kiểm tra bản phục hồi ngoại tuyến |

Các cổng trên là mặc định phát triển trong mã nguồn, không phải danh sách cổng public của Production. `cmd/server` là entrypoint tương thích để chọn **một** miền bằng `SHOPAI_SERVICE`, không phải một server gộp toàn bộ dịch vụ.

### Dữ liệu và thành phần liên quan

- **PostgreSQL:** dữ liệu nghiệp vụ, sổ credit, trạng thái xử lý và các giao dịch liên quan.
- **Redis:** cache, giới hạn request và các dependency runtime tương ứng.
- **Hệ thống:** tài nguyên gateway, token/quota và điều phối tương ứng hợp đồng tích hợp.
- **Hệ thống:** định tuyến tài khoản/provider; có dữ liệu SQLite riêng theo cấu hình triển khai.
- **Dịch vụ ngoài:** Firebase, Cloudinary/R2, Turnstile, email và thanh toán chỉ hoạt động khi có cấu hình hợp lệ.

`backend/` giữ mã Python legacy cùng oracle phục vụ đối chiếu/chuyển đổi. Không xóa hoặc coi nó là dependency đã hết vai trò chỉ dựa vào việc một dịch vụ Go đã build được. Backend Go README (`backend-go/README.md`) chứa chi tiết triển khai và lịch sử kiểm chứng.

## Công nghệ

| Thành phần | Công nghệ / cấu hình trong source |
| --- | --- |
| Backend chính | Go, Fiber v3, pgx v5, go-redis v9 |
| Toolchain Go | `go.mod` khai báo Go 1.26.0; Docker/CI hiện chọn Go 1.27 |
| Frontend | Next.js 16.3.5, React 19.2.6, TypeScript 5.9, Tailwind CSS 4 |
| Node.js | `frontend/package.json` yêu cầu từ 20.9; nên dùng Node.js 24 thống nhất môi trường |
| Database | PostgreSQL; target Compose native dùng PostgreSQL 17 |
| Container & ingress | Docker Compose, Caddy, Cloudflare theo môi trường |
| Công cụ chuẩn bị/kiểm chứng | Python 3.12, công cụ Go/SQL và kiểm thử frontend |

Phiên bản chi tiết nằm trong go.mod (`backend-go/go.mod`), package.json (`frontend/package.json`), lockfile và Dockerfile. Không có huy hiệu build/coverage để khẳng định một lần CI chưa được kiểm tra trong phiên biên soạn tài liệu này.

## Chạy local

### Chọn đúng phạm vi

| Nhu cầu | Cách bắt đầu |
| --- | --- |
| Xem/chỉnh giao diện | Chạy frontend, trỏ vào backend phát triển của bạn |
| Phát triển một miền Go | Chạy executable tương ứng với PostgreSQL/Redis local |
| Kiểm tra toàn bộ stack | Dùng `deploy/go-native`, chuẩn bị các repo tích hợp và secrets riêng |
| Vận hành hệ thống hiện có | Đọc runbook trong `deploy/azure`, không dùng lệnh khởi tạo mới |

### 1. Giao diện Next.js

Từ thư mục gốc dự án:

```powershell
cd frontend
npm ci
Copy-Item .env.example .env.local
```

Trên macOS/Linux, thay lệnh sao chép bằng `cp .env.example .env.local`.

Kiểm tra cấu hình trước khi chạy:

| Biến | Mục đích |
| --- | --- |
| `NEXT_PUBLIC_BACKEND_URL` | URL backend/ingress dành cho môi trường phát triển |
| `SHOPAI_INTERNAL_BACKEND_URL` | Tùy chọn URL backend nội bộ cho rewrite phía server |
| `MODEL_GATEWAY_URL` | Origin gateway cho các tuyến model tương ứng |
| `NEXT_PUBLIC_SHOP_TIMEZONE` | Múi giờ giao diện, ví dụ `Asia/Ho_Chi_Minh` |
| `NEXT_PUBLIC_FIREBASE_*` | Cấu hình Firebase client của project bạn dùng |
| `NEXT_PUBLIC_TURNSTILE_SITE_KEY` | Site key Turnstile khi bật luồng cần challenge |

Ví dụ có sẵn dùng backend `http://localhost:8080`; đây không phải cổng mặc định của một executable Go cụ thể. Khi dùng stack Go, hãy trỏ vào ingress đã cấu hình hoặc backend phát triển đúng phạm vi. Không sử dụng dịch vụ Production để thử thao tác ghi.

```bash
npm run dev
```

Frontend mặc định mở tại `http://localhost:3000`. Nếu backend hoặc tích hợp chưa chạy, các chức năng cần dữ liệu/xác thực sẽ chưa hoạt động đầy đủ.

### 2. Một dịch vụ Go độc lập

Chuẩn bị **database phát triển riêng**, Redis và schema phù hợp. Executable Go đọc biến môi trường tiến trình; không giả định có tự động nạp một file `.env` chung ở root.

Ví dụ cấu hình PowerShell cho dịch vụ catalog:

```powershell
cd backend-go
$env:DATABASE_URL='postgres://DEV_USER:DEV_PASSWORD@127.0.0.1:5432/shopai_dev?sslmode=disable'
$env:REDIS_URL='redis://127.0.0.1:6379/0'
$env:LISTEN_ADDRESS='127.0.0.1:8083'
$env:IDENTITY_INTERNAL_URL='http://127.0.0.1:8082'
$env:IDENTITY_SERVICE_TOKEN = node -e "process.stdout.write(require('node:crypto').randomBytes(32).toString('hex'))"
go mod download
# Chỉ với database phát triển MỚI, trống:
go run ./cmd/migrate
# Khởi động dịch vụ cần phát triển:
go run ./cmd/catalog
```

Các giá trị `DEV_USER`/`DEV_PASSWORD` là placeholder, cần thay bằng tài khoản local của bạn. Ví dụ tạo service token ngẫu nhiên cho môi trường mới; khi chạy thêm identity hoặc dịch vụ liên quan, cấu hình cùng token ở đúng các bên cần giao tiếp, không sinh lại mỗi bên một giá trị khác nhau. `sslmode=disable` trong ví dụ chỉ dành cho loopback phát triển; dùng TLS phù hợp cho kết nối qua mạng.

- `GET /health`: kiểm tra tiến trình/danh tính dịch vụ.
- `GET /ready`: kiểm tra dependency theo dịch vụ, gồm database và Redis. Không expose endpoint nội bộ này qua ingress public.
- Database đã có dữ liệu/legacy history cần quy trình riêng; **không chạy fresh migration lên Production hoặc database đang dùng**.
- Chức năng xác thực/quản trị/cấp key cần capability token, dependency và feature flag tương ứng. Catalog khởi động không có nghĩa commerce/gateway đã hoạt động.

Tham khảo cấu hình backend Go (`backend-go/README.md`) và hướng dẫn adoption legacy (`backend-go/LEGACY_ADOPTION.md`) trước khi chọn thao tác schema.

### 3. Stack Docker native đầy đủ

Cần Linux Docker, Compose hỗ trợ `env_file format: raw` (từ 2.30), Python 3.12 cùng các gói trong tools/requirements.txt (`backend-go/tools/requirements.txt`), và source tích hợp Hệ thống đúng cấu hình build.

```powershell
python -m pip install -r backend-go/tools/requirements.txt
cd deploy/go-native
python prepare.py init
python prepare.py bootstrap
docker compose --profile bootstrap config --quiet
```

Đây là các bước chuẩn bị/kiểm tra cấu hình. `init` chỉ dùng cho môi trường mới và tạo secrets riêng; không dùng để thay mật mã gốc của dữ liệu đã tồn tại. Sau đó thiết lập provider dashboards, credentials và integrations theo README native deployment (`deploy/go-native/README.md`) trước khi render/chạy profile `native`.

Không có một file `.env` mẫu duy nhất đủ khởi động mọi thành phần; secrets runtime được tạo/render theo service. Đọc đường dẫn build và yêu cầu repo tích hợp trong cấu hình Compose trước khi build.

## API và xác thực

Hệ thống có nhiều loại credential, dùng cho các bề mặt khác nhau:

| Bề mặt | Xác thực / mục đích |
| --- | --- |
| Website khách hàng/quản trị | Phiên đăng nhập, cookie hoặc bearer và kiểm tra quyền theo route |
| `/api/v1/token/*` | API Token client key/secret key, scope và entitlement |
| `/v1/*` | Model API key/Bearer và quyền truy cập model theo gateway |
| `/internal/*` | Capability/service token nội bộ; không công khai qua ingress |

### API Token

```http
X-ShopAI-Client-Key: <CLIENT_KEY>
X-ShopAI-Secret-Key: <SECRET_KEY>
```

Ví dụ đọc usage từ môi trường của bạn:

```bash
curl https://YOUR_SHOP_DOMAIN/api/v1/token/account/usage \
  -H "X-ShopAI-Client-Key: $SHOPAI_CLIENT_KEY" \
  -H "X-ShopAI-Secret-Key: $SHOPAI_SECRET_KEY"
```

Token bị kiểm tra trạng thái tài khoản, hạn dùng, scope, quota và rate limit. Các model route và API Token không dùng thay nhau một cách tự động; không lấy secret của API Token để đoán rằng đó là model key. Xem API Token Guide (`docs/API_TOKEN_GUIDE.md`) và Model API Examples (`docs/MODEL_API_EXAMPLES.md`).

### Model gateway

Client model sử dụng public base URL dạng `https://YOUR_SHOP_DOMAIN/v1`, key và model được cấp cho tài khoản. Protocol có thể là Responses, Chat Completions hoặc Messages tùy route/model; kiểm tra quyền và danh sách model trước khi cấu hình client.

Hướng dẫn client trong repo:

- ShopAIPro Launcher (`docs/SHOPAIPRO_LAUNCHER_GUIDE.md`).
- Đồng bộ model (`docs/SHOPAIPRO_MODEL_SYNC_GUIDE.md`).
- Điều phối API và credit (`docs/INFERENCE_ROUTING_AND_CREDIT.md`).

## Kiểm thử và kiểm chứng

README này được biên soạn bằng việc đọc mã nguồn và cấu hình; **không chạy lại toàn bộ test, không khởi động stack và không xác nhận Production trong lần cập nhật này**.

Các lệnh phát triển thường dùng:

```bash
# Từ frontend/
npm run lint
npm run test:api-token-scopes
npm run build

# Từ backend-go/, trong môi trường test đã chuẩn bị
go mod verify
go vet ./...
go test ./...
go build ./...
```

Một số kiểm tra cần PostgreSQL, Redis, oracle Python hoặc các service tích hợp; chỉ nhìn exit code của một selection không đủ để kết luận toàn bộ hệ thống đã đạt parity. Không chạy integration test vào database thật. Chi tiết prerequisite, phạm vi selection và CI gate nằm trong backend-go/CI.md (`backend-go/CI.md`).

Các workflow nằm tại .github/workflows (`.github/workflows`), gồm kiểm chứng Go native, container bootstrap, schema compatibility và các pipeline triển khai liên quan. Không suy ra một workflow đã pass chỉ từ việc file workflow tồn tại.

## Triển khai và vận hành

Stack native tách frontend, tám miền HTTP Go, worker, gateway dependencies và data services. Ingress public và listener quản trị nội bộ có ranh giới riêng; PostgreSQL, Redis, dashboard provider và service nội bộ không cần cổng public.

Tài liệu hiện hành mô tả runtime production trên Azure qua Cloudflare/Caddy. README không sao chép subscription ID, địa chỉ mạng riêng, đường dẫn private key hoặc thông tin phục hồi của chủ hệ thống. Dùng đúng runbook và quyền vận hành của môi trường đích.

| Tài liệu | Sử dụng khi |
| --- | --- |
| Native Go deployment (`deploy/go-native/README.md`) | Dựng cấu hình dịch vụ/ingress và chuẩn bị môi trường native |
| Owner VPS runbook (`deploy/azure/OWNER_VPS.md`) | Vận hành hệ thống đang triển khai, đọc quy trình delivery/rollback |
| VPS rebuild (`deploy/azure/VPS_REBUILD.md`) | Dựng/phục hồi môi trường theo biên bản và cấu hình của chủ hệ thống |
| Recovery (`backend-go/RECOVERY.md`) | Kiểm chứng phục hồi dữ liệu và ranh giới service/database |
| Legacy adoption (`backend-go/LEGACY_ADOPTION.md`) | Chuyển đổi schema/dữ liệu legacy có quy trình và điều kiện ràng buộc |

`deploy/single-vps` là tài liệu nền tảng chuyển tiếp cũ có Python. Không trộn hướng dẫn của nó với stack native hiện tại hoặc coi các checkpoint cũ trong tài liệu dài là trạng thái triển khai mới nhất.

## Bảo mật và quản lý cấu hình

- Phân biệt Firebase client config với service-account private key; bí mật server không dùng tiền tố `NEXT_PUBLIC_`.
- Không commit `.env`, provider credentials, API key, service token, database dump hoặc dữ liệu khách hàng.
- Dùng secret riêng cho các vai trò/signing/encryption/capability mà cấu hình yêu cầu; không thay tất cả bằng cùng một giá trị để vượt validation.
- Giữ `/internal/*`, `/ready` và dashboard provider ở ranh giới nội bộ theo ingress.
- Các luồng thanh toán dựa vào webhook/chứng cứ server đã xác minh; không coi browser return URL là bằng chứng đã thanh toán.
- Giữ nguyên cryptographic roots khi phục hồi dữ liệu đã mã hóa; không chạy bộ sinh credentials mới như một cách sửa dữ liệu cũ.

## Cấu trúc repository

```text
shopaipro-go/
├── frontend/                 # Next.js storefront, console, admin và AI Chat
│   ├── app/                  # App Router và các trang/tuyến frontend
│   ├── public/               # Tài nguyên và client installer/launcher
│   └── scripts/              # Công cụ phát triển và kiểm tra frontend
├── backend-go/
│   ├── cmd/                  # Entrypoint dịch vụ, worker và công cụ vận hành
│   ├── internal/             # Miền nghiệp vụ, persistence, HTTP và integration
│   ├── tools/                # Công cụ kiểm chứng, oracle và fixture
│   ├── README.md             # Chi tiết backend và checkpoint triển khai
│   ├── CI.md                 # Phạm vi kiểm chứng/CI
│   └── RECOVERY.md            # Kiểm chứng phục hồi
├── backend/                  # Python legacy và nguồn đối chiếu chuyển đổi
├── deploy/
│   ├── go-native/            # Compose, ingress và render cấu hình native
│   ├── azure/                # Delivery, vận hành, backup và rebuild
│   └── single-vps/           # Foundation chuyển tiếp cũ
├── docs/                     # API Token, model, client và tài liệu kiến trúc
├── .github/workflows/        # Pipeline kiểm chứng/triển khai
└── README.md                 # Giới thiệu và điểm bắt đầu đọc dự án
```

## Tài liệu

- Backend Go (`backend-go/README.md`) — triển khai, feature flag, lệnh local và lịch sử kiểm chứng.
- API Token (`docs/API_TOKEN_GUIDE.md`) — credential, scope và quyền truy cập.
- Model examples (`docs/MODEL_API_EXAMPLES.md`) — ví dụ các giao thức model.
- Routing & Credit (`docs/INFERENCE_ROUTING_AND_CREDIT.md`) — mô hình credit, cấp quyền và điều phối.
- Launcher (`docs/SHOPAIPRO_LAUNCHER_GUIDE.md`) — công cụ cấu hình client.
- Model sync (`docs/SHOPAIPRO_MODEL_SYNC_GUIDE.md`) — đồng bộ danh sách model.
- CI (`backend-go/CI.md`) — điều kiện kiểm thử và cách diễn giải kết quả.
- Native deployment (`deploy/go-native/README.md`) — cấu hình và ranh giới stack.

## Liên kết chính thức

| Kênh | Đường dẫn |
| --- | --- |
| Website | [shopaipro.tech](https://shopaipro.tech) |
| Tài liệu sử dụng | [Hướng dẫn API và công cụ AI](https://shopaipro.tech/docs) |
| Danh sách model | [Mô hình AI](https://shopaipro.tech/models) |
| Bảng giá | [Giá dịch vụ](https://shopaipro.tech/pricing) |
| Hỗ trợ | [Câu hỏi thường gặp](https://shopaipro.tech/faq) |
| Email liên hệ | [contact@shopaipro.tech](mailto:contact@shopaipro.tech) |
| Điều khoản | [Điều khoản dịch vụ](https://shopaipro.tech/terms) |
| Quyền riêng tư | [Chính sách](https://shopaipro.tech/privacy) |

Thông tin model, giá và gói dịch vụ có thể thay đổi; xem website để có thông tin hiện hành.

## Về repository này

- **Dự án:** SHOP AI PRO.
- **Mục đích:** giới thiệu sản phẩm, chức năng và kiến trúc tổng quát.
- **Website chính thức:** [https://shopaipro.tech](https://shopaipro.tech).
- **Đơn vị quản lý repository:** [levantri10](https://github.com/levantri10).
- **Phạm vi công khai:** README và logo lấy từ website chính thức; không công khai source code, secrets hoặc dữ liệu khách hàng.
- **Tài liệu nội bộ:** các đường dẫn trong phần phát triển là tham chiếu mô tả, không phải file có thể tải từ repository này.
- **Cập nhật tài liệu:** 05/11/2026.

## Quyền sử dụng

README này không cấp mới hoặc thay đổi giấy phép mã nguồn. Trước khi tái sử dụng hoặc phân phối, xác nhận điều kiện với chủ dự án và kiểm tra riêng giấy phép của Hệ thống cùng các dependency. Không gắn nhãn MIT/AGPL hoặc tuyên bố open source cho repository khi chưa có quyết định tương ứng của chủ sở hữu.

---

<div align="center">

**SHOP AI PRO** · Storefront, API access và model orchestration trên cùng một nền tảng.

[Website](https://shopaipro.tech) · Tài liệu backend (`backend-go/README.md`) · Hướng dẫn triển khai (`deploy/go-native/README.md`)

</div>
