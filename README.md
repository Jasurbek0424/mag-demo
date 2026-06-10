# MAG CRM — Max Arch Group

Qurilish kompaniyasi uchun CRM + avtomatizatsiya tizimi.

Bu papkada:
- **`MAG-CRM-demo.html`** — to'liq ishlaydigan interaktiv demo (mock data). Klientga ko'rsatish uchun. Brauzerда ochib ishlatish mumkin, server kerak emas.
- **`README.md`** — real (production) build uchun arxitektura va reja. Buni terminalда `claude code` orqali davom ettiramiz.

---

## 1. Demo nimani ko'rsatadi

- **Login + 5 rol**: Bosh administrator, Administrator, Loyiha menejeri (PM), Sotuv menejeri, Moliyachi.
- **Rolga qarab UI** — har rol faqat o'ziga ruxsat berilgan bo'limlarni ko'radi (RBAC). Tepadagi "Rol" almashtirgichdan tekshirish mumkin.
- **Boshqaruv paneli** — KPI kartalar, pul oqimi grafigi, xarajat donut-diagrammasi, sotuv konveyeri (har rol uchun alohida).
- **Lidlar (LID abrabotka)** — drag-and-drop kanban: `LID tushdi → Sifatli LID → Filialga keldi → Taklif berildi → Sotuv`. Har lidda: kimdan kelgani, kontakt, shartnoma raqami, summa, status (o'ylayapti / otmen / success). Yangi lid formasida **validatsiya** (majburiy maydonlar, telefon to'g'riligini tekshirish).
- **Loyihalar** — konveyer pipeline, jadval, CRUD, har loyihaning bosqichi/mas'uli/byudjeti/bajarilishi.
- **Moliya** — kirim→kassa→chiqim oqimi vizualizatsiyasi, naqd/bank ajratish, xarajat turlari, har loyihaning o'z puli.
- **Vazifalar (JIRA-style)** — `+` bilan task yaratish, drag-and-drop: Bajarish kerak / Jarayonda / Tekshiruvda / Bajarildi, assigned to, muhimlik. **"Faqat mening vazifalarim"** filtri (joriy foydalanuvchi tasklari).
- **Hodimlar va rollar** — hodim ustiga bosilganda **info + tahrirlash formasi** (rol tanlash, holat), **yangi rol yaratish** (nom, rang, ruxsat beriladigan bo'limlar). Rol biriktirilsa UI avtomatik moslanadi.
- **Audit jurnali** — kim, qachon, nimani o'zgartirgani.

> Demoda barcha o'zgarishlar (drag, yangi lid/task/tranzaksiya/hodim/rol) audit jurnaliga yoziladi va xotirada saqlanadi (sahifa yangilanmaguncha).

### Yangilanishlar — v1.1
- Lid qo'shish formasiga validatsiya (majburiy maydon, telefon/summa tekshiruvi, xato matnlari).
- Kanban ustun nuqtasi (status indikatori) endi to'g'ri dumaloq.
- Vazifalarda "Faqat mening vazifalarim" tugmasi.
- Hodim kartasi: ma'lumot ko'rish + tahrirlash formasi (rol/holat tanlash).
- Rol yaratish va tanlash (rang + bo'limlar bo'yicha ruxsat).
### Yangilanishlar — v1.2
- **UZ / RU til almashtirgichi** — tepa o'ng burchakda. Butun interfeys (menyu, sarlavhalar, jadvallar, kanban, modallar, toast xabarlar) ikki tilga o'tadi. Mijoz ismlari va loyiha nomlari kabi dinamik ma'lumotlar o'zgarmaydi. Lug'atda ~300 ibora.

---

## 2. Real build — tavsiya etilgan stack

| Qatlam | Texnologiya | Sabab |
|---|---|---|
| Frontend | **Next.js 15 (React 19) + TypeScript + Tailwind CSS** | Tez, SSR, zamonaviy; demo dizayni to'g'ridan-to'g'ri ko'chiriladi |
| UI komponentlar | **shadcn/ui** | Sifatli, AI-template ko'rinmaydigan komponentlar |
| Grafiklar | **Recharts** yoki qo'lda SVG (demodagidek) | Donut, bar, line |
| Backend | **NestJS** (Node + TS) yoki **Next.js API routes** | RBAC, audit, biznes-logika |
| DB | **PostgreSQL + Prisma ORM** | Munosabatli ma'lumotlar (lid→loyiha→tranzaksiya) |
| Auth | **NextAuth / Auth.js + JWT** yoki **Clerk** | Login, sessiya, 2FA |
| Realtime (ixtiyoriy) | **Socket.io** | Kanban'da jonli yangilanish |
| Deploy | **Docker + VPS** yoki **Vercel + Neon/Supabase** | — |

---

## 3. Ma'lumotlar modeli (asosiy jadvallar)

```
User(id, name, email, passwordHash, roleId, status, createdAt)
Role(id, key, name, permissions[])            // super_admin, admin, pm, sales, finance
Lead(id, name, source, phone, contractNo, amount, stage, mood, ownerId, note, createdAt)
Project(id, code, name, clientId, stage, pmId, budget, deadline, progress, respDept)
Transaction(id, date, type[in|out], category, projectId?, amount, method[cash|bank], createdById)
Task(id, title, projectId?, assigneeId, status, priority, dueDate, createdById)
AuditLog(id, userId, action, target, meta, createdAt)
Client(id, name, phone, ...)
```

## 4. Rollar va ruxsatlar matritsasi (RBAC)

| Bo'lim | Super Admin | Admin | PM | Sotuv | Moliyachi |
|---|:--:|:--:|:--:|:--:|:--:|
| Dashboard | ✓ | ✓ | ✓ | ✓ | ✓ |
| Lidlar | ✓ (CRUD) | ✓ (CRUD) | – | ✓ (CRUD) | – |
| Loyihalar | ✓ (CRUD) | ✓ | ✓ (CRUD) | – | ko'rish |
| Moliya | ✓ (to'liq) | ko'rish | qisman | – | ✓ (to'liq) |
| Vazifalar | ✓ | ✓ | ✓ | ✓ | – |
| Hodimlar/Rollar | ✓ | ko'rish | – | – | – |
| Audit | ✓ | ✓ | – | – | – |
| Sozlamalar | ✓ | – | – | – | – |

## 5. Xavfsizlik talablari

- Parollar: bcrypt/argon2 hash.
- RBAC: har API endpoint rol/permission bilan himoyalangan (server tomonda, faqat UI emas).
- Audit log: barcha yozish operatsiyalari (kim, qachon, eski→yangi qiymat).
- Ma'lumot shifrlash: TLS 1.3 (transport), maxfiy maydonlar uchun shifrlash.
- 2FA (TOTP) — admin/moliyachi uchun majburiy.
- Rate limiting, input validatsiya (zod), SQL injection himoyasi (Prisma).

## 6. Claude Code'da davom ettirish (terminal)

```bash
cd ~/Desktop/"MARS FORGE"/MAG
claude
```

Keyin Claude Code'ga taxminan shunday topshiriq bering:

> README.md dagi arxitektura bo'yicha Next.js 15 + TypeScript + Tailwind + shadcn/ui + Prisma + PostgreSQL stack'da MAG CRM ni qur. Demo dizayni (MAG-CRM-demo.html) va brend ranglari (#1C2B26, #2B3E38, #DCE2D8) saqlansin. Avval `prisma/schema.prisma`, auth va RBAC middleware, keyin modullarni ketma-ket: Lidlar → Loyihalar → Moliya → Vazifalar → Audit.

## 7. Brend ranglari (logodan)

`#1C2B26` asosiy to'q · `#2B3E38` brend yashil · `#365049` o'rta · `#DCE2D8` sage · `#EEF1EC` och fon

---
© 2026 Max Arch Group
