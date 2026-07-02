# VideoTarjimoni — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

VideoTarjimoni foydalanuvchilarga istalgan xorijiy tildagi videoni (maksimal 15 daqiqa) o‘zbek tiliga subtitr yoki ovozli tarjima qilib beradi. Videoni yuborishdan keyin foydalanuvchi qisqacha mazmuni bilan tanishadi va tarjima qilishni tasdiqlash uchun 'Ha/Yo'q' tugmalaridan birini bosadi. Tarjima qilingan video (yoki video + o‘zbekcha audio/subtitr) foydalanuvchiga qaytariladi.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- O‘zbek tilida kontent iste’mol qiluvchilar (tadqiqotchilar, talabalar, kasbiy mutaxassislar)
- Non-technical end users who interact via Telegram

## Success criteria

- Foydalanuvchi videoni yuborishdan keyin 100% mazmuni bilan tanishadi
- Foydalanuvchi tarjima qilishni 100% tasdiqlaydi yoki bekor qiladi
- Tarjima qilingan video foydalanuvchiga 100% to'liq qaytariladi

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Bot ishga tushirish va asosiy menyu ochish
- **Video yuborish** (button, actor: user) — Xorijiy tildagi video yuborish (maksimal 15 daqiqa)
  - inputs: video_file
  - outputs: summary, Ha/Yo'q buttons
- **Ha** (button, actor: user, callback: confirm:yes) — Tarjima qilishni tasdiqlash
  - inputs: confirmation
  - outputs: translation_start
- **Yo'q** (button, actor: user, callback: confirm:no) — Tarjima qilishni bekor qilish
  - inputs: confirmation
  - outputs: upload_another_or_cancel

## Flows

### Video tarjima
_Trigger:_ video upload

1. Video qabul qilish
2. Mazmun yaratish
3. Ha/Yo'q tasdiqlash
4. Tarjima qilish
5. Natija qaytarish

_Data touched:_ Uploaded Video, Summary, Translation Output, Job/Task

### Bekor qilish
_Trigger:_ Yo'q button

1. Bekor qilishni so'rash
2. Boshqa video yuborish yoki bekor qilish

_Data touched:_ Job/Task

### Xatoliklar
_Trigger:_ video >15 minutes

1. Xatolik xabari yuborish
2. Qisqaroq video yuborishni so'rash

_Data touched:_ Uploaded Video

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: persistent)_ — Telegram foydalanuvchisi
  - fields: user_id, username
- **Uploaded Video** _(retention: session)_ — Xorijiy tildagi video fayl (maksimal 15 daqiqa)
  - fields: video_id, file_path, duration, language
- **Summary** _(retention: session)_ — Video mazmuni (1-2 jumlalik o'zbek tilidagi qisqacha tavsif)
  - fields: summary_text
- **Translation Output** _(retention: session)_ — Tarjima qilingan video (o'zbek tilidagi audio yoki subtitr)
  - fields: output_type, file_path, duration
- **Job/Task** _(retention: persistent)_ — Har bir yuklangan video uchun ish jarayonini hisoblash
  - fields: job_id, user_id, video_id, created_at, status, output_type

## Integrations

- **Telegram** (required) — Bot API messaging
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Video uzunligi cheklovi (15 daqiqa)
- Tarjima chiqish formati (audio yoki subtitr)
- Mazmun yaratish algoritmi
- Xatolik xabarlari va qayta yuborish so'rovi

## Notifications

- Tarjima boshlanmoqda — iltimos kuting
- Tarjima yakunlandi
- Xatolik: video uzunligi cheklovini o'tdi

## Permissions & privacy

- Foydalanuvchi ma'lumotlari (user_id, username) saqlanadi
- Yuklangan video fayllari 30 kun saqlanadi
- Tarjima natijalari foydalanuvchiga qaytariladi

## Edge cases

- Video uzunligi 15 daqiqa dan oshsa
- Tarjima qilish jarayonida xatolik yuz berdi
- Foydalanuvchi bekor qilishni tanladi

## Required tests

- Video yuklash va mazmun hosil qilish
- Ha/Yo'q tasdiqlash jarayoni
- Tarjima qilish va natija qaytarish
- Xatoliklar va bekor qilish jarayoni

## Assumptions

- Bot videolarni 15 daqiqa cheklovini qo'llaydi
- Ha/Yo'q tasdiqlash uchun inline buttons ishlatiladi
- Tarjima chiqish formati o'zbek tilidagi audio bo'ladi
- Xatoliklar uchun qayta yuborish so'rovi yuboriladi
