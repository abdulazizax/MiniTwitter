# MiniTwitter Backend

## Loyiha haqida qisqacha

**MiniTwitter** platformasi foydalanuvchilarga tvitlar yozish, ularni baholash, izoh qoldirish va boshqa foydalanuvchilar bilan o'zaro aloqada bo'lish imkonini beradi. Ushbu loyiha mikroservis arxitekturasi asosida ishlab chiqiladi, bu esa har bir servis ma'lum bir funktsional bo'lim uchun mas'ul bo'lishini ta'minlaydi.

## Texnologiyalar

- **Golang**: Backend dasturlash tili.
- **gRPC**: Mikroservislar o'rtasida samarali aloqa uchun.
- **Protocol Buffers**: Ma'lumotlarni seriyalash va deserializatsiya qilish uchun.
- **PostgreSQL**: Foydalanuvchi ma'lumotlarini saqlash uchun.
- **MongoDB**: Tvitlar va boshqa dinamik ma'lumotlarni saqlash uchun.
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

### PostgreSQL (Auth va User Service uchun)

**Users jadvali:**
- `id` (UUID, asosiy kalit)
- `email` (string, noyob)
- `password_hash` (string)
- `username` (string, noyob)
- `first_name` (string)
- `last_name` (string)
- `bio` (string)
- `created_at` (timestamp)
- `updated_at` (timestamp)

### MongoDB (Tweet, Comment, Like va Notification Service uchun)

**Tweets to'plami:**
- `_id` (ObjectId)
- `user_id` (UUID, Users jadvaliga havola)
- `content` (string)
- `created_at` (timestamp)
- `updated_at` (timestamp)
- `likes_count` (int)

**Comments to'plami:**
- `_id` (ObjectId)
- `tweet_id` (ObjectId, Tweets to'plamiga havola)
- `user_id` (UUID, Users jadvaliga havola)
- `content` (string)
- `created_at` (timestamp)
- `updated_at` (timestamp)
- `likes_count` (int)

**Likes to'plami:**
- `_id` (ObjectId)
- `user_id` (UUID, Users jadvaliga havola)
- `target_id` (ObjectId, Tweets yoki Comments to'plamiga havola)
- `target_type` (enum: tweet, comment)
- `created_at` (timestamp)

**Notifications to'plami:**
- `_id` (ObjectId)
- `user_id` (UUID, Users jadvaliga havola)
- `type` (enum: like, comment, follow, mention)
- `message` (string)
- `read` (bool)
- `created_at` (timestamp)

## Qo'shimcha Talablar

- **Xatoliklarni boshqarish**: Har bir servis uchun to'g'ri xatoliklarni boshqarish va jurnallash tizimini joriy etish.
- **gRPC xizmatlari**: Servislar o'rtasida gRPC orqali aloqa o'rnatish.
- **JWT autentifikatsiyasi**: API Gateway uchun JWT autentifikatsiyasini amalga oshirish.
- **Casbin orqali kirish nazorati**: Rolga asoslangan kirish nazorati uchun Casbin'dan foydalanish.
- **Ma'lumotlar bazasi migratsiyalari**: PostgreSQL uchun golang-migrate orqali migratsiyalarni amalga oshirish.
- **Keshirlash**: Redis orqali tez-tez murojaat qilinadigan ma'lumotlarni keshirlash.
- **Docker**: Barcha servislarni Docker yordamida konteynerlash va mahalliy joylashtirish uchun docker-compose faylini yaratish.
- **Unit testlar**: Har bir servis uchun birlamchi testlar yozish.

Ushbu spesifikatsiyalar loyihani samarali boshqarish, yuqori sifatli va barqaror tizim yaratish uchun yo'riqnoma sifatida xizmat qiladi.
