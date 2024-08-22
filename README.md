# MiniTwitter Backend

## Loyiha haqida qisqacha

**MiniTwitter** platformasi foydalanuvchilarga tvitlar yozish, ularni baholash, izoh qoldirish va boshqa foydalanuvchilar bilan o'zaro aloqada bo'lish imkonini beradi. Ushbu loyiha mikroservis arxitekturasi asosida ishlab chiqiladi, bu esa har bir servis ma'lum bir funktsional bo'lim uchun mas'ul bo'lishini ta'minlaydi.

## Texnologiyalar

- **Golang**: Backend dasturlash tili.
- **gRPC**: Mikroservislar o'rtasida samarali aloqa uchun.
- **Protocol Buffers**: Ma'lumotlarni seriyalash va deserializatsiya qilish uchun.
- **PostgreSQL**: Barcha ma'lumotlarni saqlash uchun asosiy ma'lumotlar bazasi.
- **Gin framework**: API Gateway uchun RESTful xizmatlarni yaratish.
- **Golang-Migrate**: Ma'lumotlar bazasi migratsiyalari uchun.
- **Swagger**: API hujjatlarini yaratish va boshqarish uchun.
- **JWT**: Autentifikatsiya va avtorizatsiya uchun.
- **Redis**: Keshirlash va seanslarni boshqarish uchun.
- **Docker va Docker-compose**: Servislarni konteynerlash va lokal joylashtirish uchun.
- **Kafka yoki RabbitMQ**: Asinxron xabarlar uchun.
- **Casbin**: Rolga asoslangan kirish nazorati uchun.

## Mikroservislar

- **API Gateway**: Foydalanuvchi bilan bevosita aloqa qiluvchi RESTful API'larni taqdim etadi va boshqa mikroservislar bilan gRPC orqali aloqa qiladi.
- **Auth Service**: Foydalanuvchilarni ro'yxatdan o'tkazish, login qilish, tokenlarni tekshirish va boshqarish xizmatlarini taqdim etadi.
- **User Service**: Foydalanuvchilar profilini boshqarish va tahrirlash uchun.
- **Tweet Service**: Tvitlarni yaratish, o'chirish, qidirish va boshqarish uchun.
- **Comment Service**: Tvitlarga izoh qoldirish va ularni boshqarish uchun.
- **Like Service**: Tvitlar va izohlarga "like" qo'yish va ularni boshqarish uchun.
- **Follow Service**: Foydalanuvchilar o'rtasida "follow" va "unfollow" qilish imkoniyatini boshqarish uchun.
- **Notification Service**: Foydalanuvchilarga yangiliklar va hodisalar haqida bildirishnomalar yuborish uchun.

## API Spetsifikatsiyalari

### Auth API

- `POST /api/v1/auth/register`: Foydalanuvchini ro'yxatdan o'tkazish.
- `POST /api/v1/auth/login`: Foydalanuvchini tizimga kirishini ta'minlash.
- `POST /api/v1/auth/logout`: Foydalanuvchini tizimdan chiqarish.
- `POST /api/v1/auth/refresh`: JWT tokenni yangilash.

### User API

- `GET /api/v1/users/profile`: Foydalanuvchi profilini olish.
- `PUT /api/v1/users/profile`: Foydalanuvchi profilini yangilash.
- `GET /api/v1/users/{id}`: Boshqa foydalanuvchilarning profillarini ko'rish.
- `POST /api/v1/users/follow`: Boshqa foydalanuvchini "follow" qilish.
- `DELETE /api/v1/users/unfollow`: Foydalanuvchini "unfollow" qilish.

### Tweet API

- `POST /api/v1/tweets`: Yangi tvit yaratish.
- `GET /api/v1/tweets`: Barcha tvitlarni ko'rish.
- `GET /api/v1/tweets/{id}`: Muayyan tvitni ko'rish.
- `DELETE /api/v1/tweets/{id}`: Tvitni o'chirish.

### Comment API

- `POST /api/v1/tweets/{tweet_id}/comments`: Tvitga izoh qo'shish.
- `GET /api/v1/tweets/{tweet_id}/comments`: Tvitdagi barcha izohlarni ko'rish.
- `DELETE /api/v1/comments/{id}`: Izohni o'chirish.

### Like API

- `POST /api/v1/tweets/{tweet_id}/like`: Tvitni yoqtirish.
- `DELETE /api/v1/tweets/{tweet_id}/like`: Tvitni yoqtirishni bekor qilish.
- `POST /api/v1/comments/{comment_id}/like`: Izohni yoqtirish.
- `DELETE /api/v1/comments/{comment_id}/like`: Izohni yoqtirishni bekor qilish.

### Notification API

- `GET /api/v1/notifications`: Foydalanuvchi bildirishnomalarini ko'rish.
- `PUT /api/v1/notifications/{id}/read`: Bildirishnomani o'qilgan deb belgilash.

## Ma'lumotlar Bazasi Sxemalari

### PostgreSQL

```sql
-- Users jadvali
CREATE TABLE Users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    username VARCHAR(50) UNIQUE NOT NULL,
    first_name VARCHAR(50),
    last_name VARCHAR(50),
    bio TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL
);

-- Tweets jadvali
CREATE TABLE Tweets (
    id SERIAL PRIMARY KEY,
    user_id UUID REFERENCES Users(id) ON DELETE CASCADE,
    tweet_serial INT NOT NULL,
    content TEXT NOT NULL,
    likes_count INT DEFAULT 0,
    comments_count INT DEFAULT 0,
    views_count INT DEFAULT 0,
    repost_count INT DEFAULT 0,
    shares_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL,
    UNIQUE(user_id, tweet_serial)
);

-- Comments jadvali
CREATE TABLE Comments (
    id SERIAL PRIMARY KEY,
    tweet_id INT REFERENCES Tweets(id) ON DELETE CASCADE,
    user_id UUID REFERENCES Users(id) ON DELETE CASCADE,
    comment_serial INT NOT NULL,
    content TEXT NOT NULL,
    likes_count INT DEFAULT 0,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL,
    UNIQUE(user_id, comment_serial)
);

-- Likes jadvali
CREATE TABLE Likes (
    id SERIAL PRIMARY KEY,
    user_id UUID REFERENCES Users(id) ON DELETE CASCADE,
    target_id INT NOT NULL,
    target_type VARCHAR(10) CHECK (target_type IN ('tweet', 'comment')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL
);

-- Notifications jadvali
CREATE TABLE Notifications (
    id SERIAL PRIMARY KEY,
    user_id UUID REFERENCES Users(id) ON DELETE CASCADE,
    type VARCHAR(10) CHECK (type IN ('like', 'comment', 'follow', 'mention')),
    message TEXT NOT NULL,
    read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP DEFAULT NULL
);


```

## Qo'shimcha Talablar
- Xatoliklarni boshqarish: Har bir servis uchun to'g'ri xatoliklarni boshqarish va jurnallash tizimini joriy etish.
- gRPC xizmatlari: Servislar o'rtasida gRPC orqali aloqa o'rnatish.
- JWT autentifikatsiyasi: API Gateway uchun JWT autentifikatsiyasini amalga oshirish.
- Casbin orqali kirish nazorati: Rolga asoslangan kirish nazorati uchun Casbin'dan foydalanish.
- Ma'lumotlar bazasi migratsiyalari: PostgreSQL uchun golang-migrate orqali migratsiyalarni amalga oshirish.
- Keshirlash: Redis orqali tez-tez murojaat qilinadigan ma'lumotlarni keshirlash.
- Docker: Barcha servislarni Docker yordamida konteynerlash va mahalliy joylashtirish uchun docker-compose faylini yaratish.
- Unit testlar: Har bir servis uchun birlamchi testlar yozish.
- Ushbu spesifikatsiyalar loyihani samarali boshqarish, yuqori sifatli va barqaror tizim yaratish uchun yo'riqnoma sifatida xizmat qiladi.