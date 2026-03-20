# 🎰 Casino Simulator – Tài Xỉu & Nổ Hũ (Coin Ảo)

Dự án **Casino Simulator** là một nền tảng game cá cược trực tuyến sử dụng coin ảo, được xây dựng với kiến trúc full-stack production-level. Hệ thống bao gồm hai game chính: **Tài Xỉu** (realtime) và **Nổ Hũ** (slot machine). Người chơi có thể nạp coin thông qua yêu cầu gửi đến admin, tham gia đặt cược, và cạnh tranh trên bảng xếp hạng.

---

## 🚀 Tính năng chính

### Người dùng
- Đăng ký / Đăng nhập / Đăng xuất (JWT)
- Xem số dư coin ảo
- Gửi yêu cầu nạp coin (PENDING → admin duyệt)
- Chơi **Tài Xỉu**:
  - Đặt cược Tài / Xỉu với số coin bất kỳ
  - Countdown round 20s
  - Kết quả realtime qua Socket.IO
- Chơi **Nổ Hũ**:
  - Quay slot với animation
  - Hệ thống jackpot tích lũy
  - Kết quả dựa trên weighted random
- Xem lịch sử cược và bảng xếp hạng (top coin thắng, top số lần thắng)

### Quản trị viên
- Quản lý người dùng (xem danh sách, khóa/mở khóa, reset coin)
- Duyệt / từ chối yêu cầu nạp coin
- Xem lịch sử game (Tài Xỉu & Nổ Hũ)
- Điều chỉnh tỷ lệ thắng slot (mô phỏng)
- Thống kê tổng coin đặt / thắng

---

## 🛠 Công nghệ sử dụng

| Thành phần       | Công nghệ                                                                 |
|------------------|--------------------------------------------------------------------------|
| Frontend         | React + Vite, TailwindCSS, Framer Motion, Socket.IO client               |
| Backend          | Node.js + Express (hoặc NestJS)                                          |
| Database         | MySQL (via Sequelize/TypeORM)                                            |
| Cache / PubSub   | Redis (cho countdown, pub/sub game state)                                |
| Authentication   | JWT + bcrypt                                                             |
| Deployment       | Vercel (frontend), Railway / Render (backend)                            |
| Dev Tools        | ESLint, Prettier, Git, GitHub Actions, Postman                           |

---

## 🧱 Kiến trúc hệ thống

```
[Frontend React + Vite] <--Socket.IO / REST API--> [Backend Node.js]
                                                   |
                                -------------------------------
                                |                             |
                            MySQL                         Redis
                    (users, wallets, bets,        (cache, countdown, 
                     rounds, deposit_requests)      pub/sub)
```

---

## 📁 Cấu trúc thư mục

```
/casino-simulator
│
├─ backend
│   ├─ src
│   │   ├─ index.js
│   │   ├─ app.js
│   │   ├─ routes/
│   │   │   ├─ auth.js
│   │   │   ├─ user.js
│   │   │   ├─ wallet.js
│   │   │   ├─ deposit.js
│   │   │   ├─ tai-xiu.js
│   │   │   └─ slot.js
│   │   ├─ controllers/
│   │   ├─ services/
│   │   ├─ models/
│   │   └─ utils/
│   └─ package.json
│
├─ frontend
│   ├─ src
│   │   ├─ main.jsx
│   │   ├─ App.jsx
│   │   ├─ pages/
│   │   │   ├─ Login.jsx
│   │   │   ├─ Dashboard.jsx
│   │   │   ├─ TaiXiu.jsx
│   │   │   ├─ Slot.jsx
│   │   │   └─ Deposit.jsx
│   │   ├─ components/
│   │   │   ├─ DiceAnimation.jsx
│   │   │   ├─ SlotReel.jsx
│   │   │   ├─ Wallet.jsx
│   │   │   ├─ Leaderboard.jsx
│   │   │   └─ DepositForm.jsx
│   │   ├─ services/
│   │   │   ├─ api.js
│   │   │   └─ socket.js
│   │   └─ context/
│   │       └─ AuthContext.jsx
│   └─ package.json
│
└─ docker-compose.yml
```

---

## 🗄 Cơ sở dữ liệu (MySQL)

Các bảng chính:
- `users`: thông tin tài khoản
- `wallets`: số dư coin
- `deposit_requests`: yêu cầu nạp coin
- `game_rounds`: vòng chơi Tài Xỉu
- `bets`: lịch sử cược
- `slot_spins`: lịch sử quay slot
- `leaderboard`: bảng xếp hạng

Xem chi tiết tại file `database-schema.sql`.

---

## 🔁 Luồng nghiệp vụ chính

### Nạp coin
1. Người dùng gửi yêu cầu nạp (số coin, ghi chú)
2. Tạo `deposit_request` với status `PENDING`
3. Admin duyệt → cập nhật wallet (transaction-safe)

### Tài Xỉu
1. Server mở round → broadcast countdown 20s
2. Người chơi đặt cược → trừ coin
3. Kết thúc countdown → roll dice → xác định thắng/thua
4. Cập nhật wallet, leaderboard, broadcast kết quả

### Nổ Hũ
1. Người chơi spin → weighted random → kết quả
2. Tính thắng/thua → cập nhật wallet, leaderboard
3. Nếu trúng jackpot → reset jackpot

---

## 🧪 Cài đặt và chạy thử

### Yêu cầu
- Node.js (v18+)
- MySQL (v8+)
- Redis

### Backend
```bash
cd backend
npm install
cp .env.example .env   # cấu hình database, JWT, Redis
npm run dev
```

### Frontend
```bash
cd frontend
npm install
cp .env.example .env   # cấu hình API URL, Socket URL
npm run dev
```

---

## 🔐 Một số lưu ý bảo mật & best practices

- **Server-authoritative**: mọi logic game, cập nhật coin đều thực hiện trên server
- **RNG an toàn**: sử dụng `crypto.randomInt` cho dice và slot
- **Transaction-safe**: cập nhật wallet, bets, deposit request trong transaction
- **Realtime**: Socket.IO cho countdown và kết quả game
- **Logging & audit**: lưu lại toàn bộ round, bet, deposit request để debug và replay

---

## 📬 Liên hệ & đóng góp

Dự án được phát triển với mục đích học tập và thực hành kiến trúc full-stack. Mọi đóng góp vui lòng tạo pull request hoặc liên hệ qua email.

---

**Happy coding! 🎲🎰**
```

---
