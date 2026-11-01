# BẢNG ĐỐI CHIẾU YÊU CẦU DỰ ÁN & HƯỚNG DẪN KIỂM TRA BẰNG CMD

Tài liệu này tổng hợp:
1. Chi tiết từng yêu cầu đề bài đã làm ở file nào, dòng code nào.
2. Hướng dẫn chạy dự án và dùng CMD (Command Prompt / PowerShell / curl / DevTools) để xem port, xem lỗi và theo dõi các gói tin HTTP (Request/Response Headers, Body).

---

## PHẦN 1: BẢNG ĐỐI CHIẾU CHI TIẾT TỪNG YÊU CẦU ĐỀ BÀI

### A. Yêu cầu Backend (Node.js + Express)

Thư mục gốc Backend: `C:\Users\hao14\.gemini\antigravity\scratch\jwt-mini-project\server`

| STT | Yêu cầu đề bài | File thực hiện | Dòng code | Nội dung / Giải thích |
|---|---|---|---|---|
| **1** | **Cài đặt express, cors, jsonwebtoken** | `server/package.json` | Dòng 10 - 14 | Khai báo trong `dependencies`: `"cors"`, `"express"`, `"jsonwebtoken"` |
| **2** | **Cấu hình CORS cho Client React (port 5173)** | `server/server.js` | Dòng 11 | `app.use(cors({ origin: "http://localhost:5173" }));` |
| **3** | **Sử dụng `app.use(express.json())` đọc body JSON** | `server/server.js` | Dòng 13 | `app.use(express.json());` phân tích dữ liệu JSON gửi lên vào `req.body` |
| **4** | **API Đăng nhập: Nhận `email`, `matKhau` từ client** | `server/server.js` | Dòng 16 - 17 | `app.post("/login", (req, res) => { const { email, matKhau } = req.body;` |
| **5** | **Hardcode kiểm tra đúng `admin@gmail.com` / `123456` → trả JWT token** | `server/server.js` | Dòng 20 - 25 | `if (email === "admin@gmail.com" && matKhau === "123456") { const token = jwt.sign(...); return res.status(200).json({ token }); }` |
| **6** | **Đăng nhập sai → trả về lỗi 401** | `server/server.js` | Dòng 27 | `res.status(401).json({ message: "Email hoặc mật khẩu không đúng" });` |
| **7** | **Middleware lấy token từ `req.headers.authorization`** | `server/middleware.js` | Dòng 9 - 16 | `const authHeader = req.headers.authorization;` và kiểm tra cấu trúc `Bearer <token>` |
| **8** | **Không có header hoặc sai định dạng → trả 401** | `server/middleware.js` | Dòng 12 - 14 | `if (!authHeader \|\| !authHeader.startsWith("Bearer ")) { return res.status(401).json(...); }` |
| **9** | **Dùng `jwt.verify()` kiểm tra: lỗi trả 403, thành công gọi `next()`** | `server/middleware.js` | Dòng 18 - 27 | `jwt.verify(token, SECRET_KEY, (err, payload) => { if (err) return res.status(403)...; req.user = payload; next(); })` |
| **10** | **Route `GET /api/products` được bảo vệ bởi middleware `xacThuc`** | `server/server.js` | Dòng 31 | `app.get("/api/products", xacThuc, (req, res) => { ... });` |
| **11** | **Xác thực thành công → trả về JSON danh sách sản phẩm ảo** | `server/server.js` | Dòng 32 - 39 | Trả về mảng 4 sản phẩm: Bút bi, Vở kẻ ngang, Thước kẻ, Tẩy với mã HTTP `200` |

---

### B. Yêu cầu Frontend (React)

Thư mục gốc Frontend: `C:\Users\hao14\.gemini\antigravity\scratch\jwt-mini-project\client-app`

| STT | Yêu cầu đề bài | File thực hiện | Dòng code | Nội dung / Giải thích |
|---|---|---|---|---|
| **1** | **Tạo form nhập email và mật khẩu** | `client-app/src/Login.jsx` | Dòng 72 - 95 | Thẻ `<form>` với `<input id="email">` và `<input id="matKhau">` |
| **2** | **Dùng axios gửi POST request tới `/login`** | `client-app/src/Login.jsx` | Dòng 17 - 20 | `const res = await axios.post("http://localhost:3000/login", { email, matKhau });` |
| **3** | **Đăng nhập thành công → lưu token vào `localStorage`** | `client-app/src/Login.jsx` | Dòng 22 - 23 | `localStorage.setItem("token", res.data.token); onDangNhap(res.data.token);` |
| **4** | **Lấy token từ `localStorage` khi mount (useEffect)** | `client-app/src/ProductList.jsx` | Dòng 21 - 23 | `useEffect(() => { const savedToken = localStorage.getItem("token"); ... }, []);` |
| **5** | **Gửi GET `/api/products` kèm header `Authorization: Bearer <token>`** | `client-app/src/ProductList.jsx` | Dòng 33 - 35 | `axios.get("http://localhost:3000/api/products", { headers: savedToken ? { Authorization: \`Bearer ${savedToken}\` } : {} })` |
| **6** | **Khai báo 3 state: `data`, `loading`, `error`** | `client-app/src/ProductList.jsx` | Dòng 16 - 18 | `const [data, setData] = useState([]);`<br>`const [loading, setLoading] = useState(true);`<br>`const [error, setError] = useState(null);` |
| **7** | **Bắt đầu gọi API -> set `loading = true`** | `client-app/src/ProductList.jsx` | Dòng 24 - 25 | `setLoading(true); setError(null);` |
| **8** | **Gọi thành công -> cập nhật `data`, tắt loading** | `client-app/src/ProductList.jsx` | Dòng 36 - 39 & 46 | `.then((res) => { setData(res.data); ... })` và `.finally(() => setLoading(false));` |
| **9** | **Thất bại -> cập nhật `error` báo lỗi, tắt loading** | `client-app/src/ProductList.jsx` | Dòng 40 - 46 | `.catch((err) => setError(...))` và `.finally(() => setLoading(false));` |
| **10** | **Hiển thị giao diện theo trạng thái ("Đang tải...", "Lỗi...", danh sách)** | `client-app/src/ProductList.jsx` | Dòng 106 - 171 | • `loading && (...)`: hiển thị Spinner "Đang gọi API..."<br>• `!loading && error && (...)`: hiển thị khung đỏ báo lỗi và nút Retry<br>• `!loading && !error && (...)`: render bảng sản phẩm |

---

## PHẦN 2: HƯỚNG DẪN CHẠY VÀ XEM CÁC GÓI TIN / PORT TRÊN CMD

Bạn có thể mở cửa sổ **Command Prompt (CMD)** hoặc **PowerShell** trên Windows để thực hiện các bước sau:

### 1. Cách khởi động Server và Client trên 2 cửa sổ CMD

#### Cửa sổ CMD 1 (Chạy Backend Server Express):
```cmd
cd /d C:\Users\hao14\.gemini\antigravity\scratch\jwt-mini-project\server
node server.js
```
> Khi chạy thành công, CMD 1 sẽ in:
> `Server đang chạy tại http://localhost:3000`

#### Cửa sổ CMD 2 (Chạy Frontend React Vite):
```cmd
cd /d C:\Users\hao14\.gemini\antigravity\scratch\jwt-mini-project\client-app
npx vite --host
```
> Khi chạy thành công, CMD 2 sẽ in:
> `➜ Local: http://localhost:5173/`

---

### 2. Cách xem các Port (3000 & 5173) có đang hoạt động trên CMD không

Mở thêm một cửa sổ CMD mới và gõ lệnh sau để kiểm tra cổng mạng đang lắng nghe (LISTENING):

```cmd
netstat -ano | findstr "3000 5173"
```

**Kết quả mẫu hiển thị:**
```text
  TCP    0.0.0.0:3000           0.0.0.0:0              LISTENING       12344
  TCP    0.0.0.0:5173           0.0.0.0:0              LISTENING       15820
```
- Cột số cuối cùng (`12344`, `15820`) chính là mã tiến trình (PID).
- Muốn biết tiến trình nào đang chiếm cổng, gõ tiếp:
  ```cmd
  tasklist | findstr "12344"
  ```
  *(thay `12344` bằng số PID thực tế của bạn)*

---

### 3. Cách dùng CMD (curl) để "soi" từng gói tin Request, Response và Mã lỗi

Dùng lệnh `curl -v` (Verbose mode) trên CMD để xem chi tiết từng dòng header, HTTP Status code (200, 401, 403), và Body của gói tin.

#### Test 1: Xem gói tin ĐĂNG NHẬP THÀNH CÔNG (Nhận Token 200 OK)
Chạy trên CMD:
```cmd
curl -v -X POST http://localhost:3000/login -H "Content-Type: application/json" -d "{\"email\":\"admin@gmail.com\",\"matKhau\":\"123456\"}"
```
**Ý nghĩa các dòng xuất hiện trên màn hình CMD:**
- Dòng bắt đầu bằng `>`: Là **Request Header** gửi từ client lên (ví dụ: `> POST /login HTTP/1.1`).
- Dòng bắt đầu bằng `<`: Là **Response Header** máy chủ trả về:
  - `< HTTP/1.1 200 OK` (Mã trạng thái thành công).
  - `< access-control-allow-origin: http://localhost:5173` (Header CORS được cấu hình).
- Dòng cuối cùng: Body JSON chứa chuỗi JWT Token:
  `{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6..."}`

---

#### Test 2: Xem gói tin ĐĂNG NHẬP SAI (Báo lỗi 401 Unauthorized)
Chạy trên CMD:
```cmd
curl -v -X POST http://localhost:3000/login -H "Content-Type: application/json" -d "{\"email\":\"admin@gmail.com\",\"matKhau\":\"sai_mat_khau\"}"
```
**Ý nghĩa kết quả:**
- `< HTTP/1.1 401 Unauthorized`
- Body trả về: `{"message":"Email hoặc mật khẩu không đúng"}`

---

#### Test 3: Xem gói tin BỊ CHẶN do KHÔNG CÓ TOKEN (Lỗi 401)
Chạy trên CMD:
```cmd
curl -v http://localhost:3000/api/products
```
**Ý nghĩa kết quả:**
- Gửi GET không có Header `Authorization`.
- Middleware `xacThuc` phát hiện và trả về ngay:
  - `< HTTP/1.1 401 Unauthorized`
  - Body: `{"message":"Bạn chưa đăng nhập"}`

---

#### Test 4: Xem gói tin BỊ TỪ CHỐI do TOKEN GIẢ HOẶC HẾT HẠN (Lỗi 403 Forbidden)
Chạy trên CMD:
```cmd
curl -v http://localhost:3000/api/products -H "Authorization: Bearer token_gia_mao_123"
```
**Ý nghĩa kết quả:**
- Gửi kèm token nhưng sai chữ ký bí mật (`SECRET_KEY`).
- Hàm `jwt.verify()` báo lỗi và trả về:
  - `< HTTP/1.1 403 Forbidden`
  - Body: `{"message":"Token không hợp lệ hoặc đã hết hạn"}`

---

#### Test 5: Xem gói tin GỬI KÈM TOKEN HỢP LỆ (Thành công 200 OK)
Lấy chuỗi token nhận được từ Test 1 dán vào vị trí `<TOKEN>`:
```cmd
curl -v http://localhost:3000/api/products -H "Authorization: Bearer <TOKEN>"
```
**Ý nghĩa kết quả:**
- `< HTTP/1.1 200 OK`
- Body: `[{"id":1,"ten":"Bút bi","gia":5000},{"id":2,"ten":"Vở kẻ ngang","gia":12000},...]`

---

### 4. Cách theo dõi gói tin trực tiếp trên Giao diện Trình duyệt (F12 DevTools)

Ngoài CMD, cách trực quan nhất để xem các gói tin tương tác giữa React và Express:
1. Mở trình duyệt vào trang **http://localhost:5173**.
2. Nhấn phím **F12** (hoặc chuột phải chọn **Inspect**) -> Chọn tab **Network** (Mạng).
3. Đăng nhập:
   - Bạn sẽ thấy dòng **`login`** xuất hiện.
   - Nhấp vào **`login`** -> chọn tab **Payload** để thấy `email`, `matKhau` gửi đi.
   - Chọn tab **Preview** hoặc **Response** để thấy `token` máy chủ trả về.
4. Sau khi vào màn hình danh sách:
   - Thấy dòng **`products`**.
   - Chọn tab **Headers** -> cuộn xuống mục **Request Headers** sẽ thấy dòng:
     `Authorization: Bearer eyJhbGciOi...`
   - Chọn tab **Response** để thấy danh sách 4 sản phẩm JSON.

