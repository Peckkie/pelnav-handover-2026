# ติดตั้ง PelNav บนเครื่องแล็บที่เป็น Linux (แทนคู่มือ Windows)

คู่มือติดตั้ง.pdf เขียนไว้สำหรับ Windows แต่เครื่องแล็บที่มี CVAT เป็น Ubuntu 20.04 จึงทำแบบนี้แทน

ต่างจากคู่มือ 2 อย่าง

- **พอร์ต 8091 ไม่ใช่ 8090** — บนเครื่องนี้ traefik ของ CVAT จอง 8090 ไว้แล้ว (ไม่แตะ CVAT) ลิงก์ของหมอจึงเป็น `http://<IP เครื่องแล็บ>:8091/`
- **รันใน docker** — เครื่องมีแค่ Python 3.8 แต่โค้ดต้องใช้ 3.11 ขึ้นไป (`tomllib`) และไม่มี sudo จึงใช้ image `python:3.12-slim`

โค้ดเว็บใช้ commit เดียวกับคู่มือ: `witsarut17/landmark-marker` @ `47d598b25f62ed6f7e1d53761395894797c586c7`

## โครงสร้างบนเครื่อง

```
<ROOT>/            เช่น /media/hdd3/<user>/PelNav  (chmod 700, อย่าใช้ดิสก์ / ที่ใกล้เต็ม)
  app/             โค้ดจาก GitHub (มี server/ และ marker/)
  data/            config.toml, db/, backups/, logs/
  packages/        orig_package/ (มีชื่อ/HN ผู้ป่วย) และ test_package/
  docker/          Dockerfile + requirements.txt
  m                ไฟล์ m ในโฟลเดอร์นี้
  run.sh           ไฟล์ run.sh ในโฟลเดอร์นี้
```

## ขั้นตอน

1. สร้าง `<ROOT>` แล้ว `chmod 700 <ROOT>`
2. แตก zip ของโค้ดไว้ที่ `<ROOT>/app` (ต้องมี `app/server/manage.py` และ `app/marker/index.html`)
3. คัดลอก zip แพ็กเกจภาพขึ้นเครื่อง เช็ก `sha256sum` ให้ตรงกับคู่มือ แตกไว้ที่ `<ROOT>/packages` แล้ว**ลบ zip ทิ้งทุกที่** (ทั้งบนเครื่องแล็บและเครื่องที่ดาวน์โหลดมา)
4. build image
   ```
   cp Dockerfile <ROOT>/docker/ && cp <ROOT>/app/server/requirements.txt <ROOT>/docker/
   docker build -t pelnav-py312:47d598b <ROOT>/docker
   ```
5. วาง `m` กับ `run.sh` ไว้ที่ `<ROOT>` แล้ว `chmod 700 <ROOT>/m <ROOT>/run.sh`
6. `<ROOT>/m init` แล้วแก้ `<ROOT>/data/config.toml` ส่วน `[server]`
   ```
   host = "0.0.0.0"
   port = 8091
   public_url = "http://<IP เครื่องแล็บ>:8091/"
   ```
7. `<ROOT>/run.sh` → ตรวจ `curl http://127.0.0.1:8091/api/health` ต้องได้ `"ok": true`
8. ทำต่อตามคู่มือหน้า 2 (ขั้นที่ 5–7) โดยใช้ `<ROOT>/m ...` แทน `m ...` และใช้พอร์ต 8091 ใน `--base`

## ดูแลประจำ

- ดู log การเปิด/ปิด: `docker logs pelnav` · log การใช้งาน: `<ROOT>/data/logs/app.log`
- รีสตาร์ท: `docker restart pelnav`
- อัปเดตโค้ด: แทนที่ `<ROOT>/app` ด้วย zip ใหม่ → ถ้า `requirements.txt` เปลี่ยนให้ build image ใหม่ (เปลี่ยน tag) แล้ว `docker rm -f pelnav && <ROOT>/run.sh` → ถ้าไม่เปลี่ยน แค่ `docker restart pelnav`
- ไม่มี sudo จึงไม่ได้ตั้งไฟร์วอลล์ของเครื่อง ใช้การกรองเครือข่ายในตัวแอป (`allowed_networks`) แทน

## ห้าม

- ห้าม commit ภาพผู้ป่วย, `config.toml`, ฐานข้อมูล, รหัสผ่าน หรือลิงก์ของหมอ ขึ้น repo นี้ (repo เป็น public)
