# คู่มือการ Deployment โปรเจกต์ Full Stack

**(Express + Prisma + Supabase + Quasar/Vue)**

คู่มือนี้จัดทำขึ้นสำหรับนักศึกษาเพื่ออธิบายขั้นตอนการนำโปรเจกต์ขึ้นระบบ Cloud (**Vercel**) โดยใช้ฐานข้อมูล Supabase

---

## 🏗️ ภาพรวมของระบบ (Architecture)

ระบบของเราประกอบด้วย 3 ส่วนหลักที่ทำงานร่วมกัน:

1.  **Frontend (Quasar/Vue)**: ฝากไว้ที่ **Vercel** (ทำหน้าที่แสดงผล UI ให้ผู้ใช้)
2.  **Backend (Express/Node.js)**: ฝากไว้ที่ **Vercel** (ทำหน้าที่จัดการ Logic และ API)
3.  **Database (PostgreSQL)**: ฝากไว้ที่ **Supabase** (ทำหน้าที่เก็บข้อมูลจริง)

---

## 🛠️ สิ่งที่ต้องติดตั้งก่อนเริ่ม (Prerequisites)

ก่อนจะเริ่มทำการ Deploy นักศึกษาต้องตรวจสอบว่าเครื่องของตนมีเครื่องมือดังนี้:

- **Git**: สำหรับการส่งโค้ดขึ้น GitHub
- **Node.js (เวอร์ชัน 18 ขึ้นไป)**: สำหรับรันคำสั่งในเครื่อง
- **GitHub Account**: สำหรับเก็บ Source Code
- **Vercel Account**: สำหรับการ Deploy ทั้ง Frontend และ Backend
- **Supabase Account**: สำหรับฐานข้อมูล PostgreSQL

---

## 🚩 ขั้นตอนที่ 0: นำโค้ดขึ้น GitHub (Git Initialization)

ก่อนจะเริ่ม Deploy บน Cloud นักศึกษาต้องนำโค้ดขึ้น GitHub Repository ของตัวเองก่อน:

1.  เปิด Terminal ในโฟลเดอร์หลักของโปรเจกต์
2.  **Initialize Git**: `git init`
3.  **ตรวจสอบ .gitignore**: มั่นใจว่ามีไฟล์ชื่อ `.gitignore` และมีรายชื่อโฟลเดอร์ `node_modules`, `.env`, `dist` เพื่อไม่ให้ไฟล์หนักหรือความลับหลุดขึ้นไป
4.  **Add & Commit**:
    ```bash
    git add .
    git commit -m "Initial commit: My Fullstack Project"
    ```
5.  **สร้าง Repo ใน GitHub**: ไปที่ GitHub.com > New Repository
6.  **Push Code**:
    ```bash
    git remote add origin https://github.com/USERNAME/REPO-NAME.git
    git branch -M main
    git push -u origin main
    ```

---

## 🚩 ขั้นตอนที่ 1: เตรียมฐานข้อมูล [Supabase](https://supabase.com/)

1.  สมัครใช้งานและสร้างโปรเจกต์ใหม่ใน [Supabase Dashboard](https://supabase.com/dashboard)
2.  ไปที่เมนู **Project Settings > Database**
3.  หาหัวข้อ **Connection string** และเลือกแท็บ **Transaction**
4.  คัดลอก URL นั้นมา (พอร์ตจะลงท้ายด้วย `6543`)
5.  **สำคัญ**: ให้เติมคำว่า `?pgbouncer=true` ต่อท้าย URL เสมอ เช่น:
    `postgresql://postgres.xxx:pass@aws-xxx.pooler.supabase.com:6543/postgres?pgbouncer=true`

---

## 🚩 ขั้นตอนที่ 2: สร้างตารางและข้อมูลเริ่มต้น (Prisma CLI)

ในขั้นตอนนี้เราจะส่งโครงสร้างตารางจากเครื่องเราไปยัง Supabase โดยตรง เพื่อให้ฐานข้อมูลพร้อมใช้งานก่อนตั้งค่า Server

1.  เปิด Terminal ในโฟลเดอร์ `backend` ที่เครื่องของนักศึกษา
2.  สร้างไฟล์ชื่อ `.env` และใส่ค่าดังนี้:
    `DATABASE_URL="URL_ที่ได้จากขั้นตอนที่_1"`
3.  รันคำสั่งตามลำดับดังนี้:

    ```bash
    # 1. ติดตั้ง Library ทั้งหมด
    npm install

    # 2. สร้าง Client สำหรับเชื่อมต่อ
    npx prisma generate

    # 3. ส่งโครงสร้างตารางไปที่ Supabase
    npx prisma db push

    # 4. ใส่ข้อมูลตัวอย่าง (Seeding) เข้าไปในฐานข้อมูล
    npx prisma db seed
    ```

4.  **วิธีการตรวจสอบข้อมูลที่บันทึกเข้าไป**:
    - **วิธีที่ 1 (ผ่าน Dashboard)**: เข้าเว็บ [Supabase Table Editor](https://supabase.com/dashboard/project/_/editor) > เลือกตาราง `tasks` จะเห็นข้อมูลที่เพิ่ง Seed เข้าไป
    - **วิธีที่ 2 (ผ่าน GUI ของ Prisma)**: รันคำสั่ง `npx prisma studio` ใน Terminal ระบบจะเปิดหน้าเว็บในเครื่องให้คุณดูและแก้ไขข้อมูลใน Supabase ได้ทันที

---

## 🚩 ขั้นตอนที่ 3: เตรียม Backend ขึ้นระบบ [Vercel](https://vercel.com/)

1.  **การสมัครสมาชิกรวมถึง Login**:
    - เข้าหน้าเว็บ [Vercel Dashboard](https://vercel.com/dashboard) และเลือก **Continue with GitHub**
2.  **การสร้าง Project ใหม่**:
    - กดปุ่ม **Add New > Project**
    - เลือก GitHub Repository ของโปรเจกต์นี้ และกด **Import**
3.  **การตั้งค่า (Project Settings)**:
    - **Project Name**: ตั้งชื่อ เช่น `my-backend-api`
    - **Framework Preset**: เลือก **Other** (เนื่องจากเป็น Express)
    - **Root Directory**: เลือกโฟลเดอร์ `backend`
4.  **การตั้งค่า Environment Variables**:
    - ไปที่หัวข้อ **Environment Variables**
    - เพิ่มตัวแปรชื่อ `DATABASE_URL` และใส่ค่า URL จาก Supabase (ที่ได้จากขั้นตอนที่ 1)
5.  **การ Deploy**:
    - กดปุ่ม **Deploy** และรอจนระบบทำงานเสร็จ
6.  จด URL ที่ได้จาก Vercel ไว้ (เช่น `https://my-backend-api.vercel.app`)

---

## 🚩 ขั้นตอนที่ 4: เตรียม Frontend ขึ้นระบบ [Vercel](https://vercel.com/)

1.  **การสร้าง Project ใหม่ (แยกจาก Backend)**:
    - กลับไปที่ Dashboard และกด **Add New > Project** อีกครั้ง
    - เลือก Repository เดิม (ตัวเดียวกันกับ Backend)
2.  **การตั้งค่า (Project Settings)**:
    - **Project Name**: ตั้งชื่อ เช่น `my-frontend-app`
    - **Framework Preset**: เลือก **Other** หรือ Vercel อาจจะตรวจเจอว่าเป็น Vite/Vue
    - **Root Directory**: เลือกโฟลเดอร์ `frontend`
3.  **การตั้งค่า Build Command**:
    - **Build Command**: `npx quasar build`
    - **Output Directory**: `dist/spa`
4.  **การตั้งค่า Environment Variables**:
    - เพิ่มตัวแปรชื่อ `API_URL`
    - ใส่ URL ของ Backend ที่ได้จากขั้นตอนที่ 3 (ต้องเติม `/api` ต่อท้ายเสมอ) เช่น `https://my-backend-api.vercel.app/api`
5.  **การ Deploy**:
    - กดปุ่ม **Deploy** และรอจนเสร็จสิ้น

---

## 💡 คำแนะนำสำหรับการแก้ปัญหา (Troubleshooting)

- **หน้าเว็บโหลดข้อมูลไม่ได้**: เช็ค `API_URL` ใน Vercel (Frontend Project) ว่าสะกดถูกต้องและมี `/api` ต่อท้ายหรือไม่
- **API ติด CORS**: ตรวจสอบใน `backend/server.js` ว่ามีการใช้ `cors()` หรือยัง (Vercel Backend ส่วนใหญ่จะไม่มีปัญหานี้ถ้าตั้งค่าถูกต้อง)
- **ตรวจสอบข้อมูลผ่าน API**: เปิด Browser ไปที่ `URL_ของ_Vercel_Backend/api/demo` เพื่อดูว่า API ตอบกลับหรือไม่
- **Error P1000**: เช็ครหัสผ่านใน `DATABASE_URL` ใน Vercel (Backend Project) ว่าถูกต้องหรือไม่

---

_จัดทำโดย: ระบบ AI Assistant สำหรับผู้ช่วยสอน (Ta)_
