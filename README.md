# SM-Team-Zibbix

โปรเจกต์นี้ใช้ Docker Compose สำหรับรันระบบ Zabbix บนเครื่อง local หรือเครื่อง UAT โดยอิงจากโครงสร้าง `zabbix-docker` และปรับค่าไว้สำหรับงานของทีม

## ภาพรวมระบบ

service หลักที่ใช้ในชุดนี้:

- `mysql-server` สำหรับฐานข้อมูล
- `zabbix-server` สำหรับประมวลผลข้อมูล monitoring
- `zabbix-web-nginx-mysql` สำหรับหน้าเว็บ Zabbix
- `server-db-init` สำหรับ initialize schema ของฐานข้อมูล

service เพิ่มเติมที่มีใน compose:

- `zabbix-agent`
- `zabbix-proxy-sqlite3`
- `zabbix-proxy-mysql`
- `zabbix-java-gateway`
- `zabbix-snmptraps`
- `zabbix-web-service`

## โครงสร้างไฟล์สำคัญ

- `compose.yaml` จุดเริ่มต้นของ Docker Compose
- `compose_zabbix_components.yaml` กำหนดค่าของ Zabbix services
- `compose_databases.yaml` กำหนดค่าฐานข้อมูล
- `.env` เก็บค่า image tag, port, network และ path หลักของโปรเจกต์
- `env_vars/` เก็บ environment files และ secret files ที่ container ใช้งาน
- `zbx_env/` เก็บ data, scripts และไฟล์ mount ที่ใช้ระหว่างรันระบบ

## สิ่งที่ต้องมี

- Docker Desktop
- Docker Compose v2
- Git

ตรวจสอบเวอร์ชันด้วยคำสั่ง:

```powershell
docker --version
docker compose version
git --version
```

## การตั้งค่าก่อนรัน

1. clone repo นี้ลงเครื่อง

```powershell
git clone https://github.com/tonenyS/SM-Team-Zibbix.git
cd SM-Team-Zibbix
```

2. ตรวจสอบไฟล์ `.env`

ค่าเริ่มต้นที่สำคัญ:

- `OS=alpine`
- `ZBX_VERSION=7.4`
- `DATA_DIRECTORY=./zbx_env`
- `ENV_VARS_DIRECTORY=./env_vars`
- `ZABBIX_SERVER_PORT=10051`
- `ZABBIX_WEB_NGINX_HTTP_PORT=80`
- `ZABBIX_WEB_APACHE_HTTP_PORT=8081`

3. ตรวจสอบไฟล์ในโฟลเดอร์ `env_vars/`

ไฟล์ที่ควรมีอย่างน้อย:

- `env_vars/.env_srv`
- `env_vars/.env_web`
- `env_vars/.env_db_mysql`
- `env_vars/.MYSQL_USER`
- `env_vars/.MYSQL_PASSWORD`
- `env_vars/.MYSQL_ROOT_USER`
- `env_vars/.MYSQL_ROOT_PASSWORD`

4. ถ้าต้องการแก้ port หรือ image tag ให้แก้ที่ไฟล์ `.env` ก่อนสั่ง run

## วิธี build และ start project

กรณีต้องการดึง image ตามค่าใน compose และ start ระบบ:

```powershell
docker compose pull
docker compose up -d
```

กรณีต้องการ build ใหม่จาก Dockerfile ในโปรเจกต์:

```powershell
docker compose build
docker compose up -d
```

หมายเหตุ:

- ถ้าไม่ได้ระบุ profile ระบบจะรันชุดหลักตามค่า default
- ถ้าต้องการ service เพิ่มเติม เช่น agent, snmptrap หรือ proxy ให้ใช้ profile

ตัวอย่าง:

```powershell
docker compose --profile full up -d
```

หรือ

```powershell
docker compose --profile all up -d
```

## ตรวจสอบหลัง start

ดูสถานะ container:

```powershell
docker compose ps
```

ดู log:

```powershell
docker compose logs -f
```

ดู log เฉพาะ service:

```powershell
docker compose logs -f zabbix-server
docker compose logs -f zabbix-web-nginx-mysql
docker compose logs -f mysql-server
```

## การเข้าใช้งานระบบ

เมื่อ container ขึ้นครบแล้ว ให้เปิดผ่าน browser:

- Zabbix Web Nginx: `http://localhost`
- ถ้าใช้ Apache image: `http://localhost:8081`

port ที่สำคัญจากค่า default:

- `80` สำหรับ Zabbix Web Nginx
- `443` สำหรับ Zabbix Web Nginx HTTPS
- `10051` สำหรับ Zabbix Server
- `10050` สำหรับ Zabbix Agent
- `10060` สำหรับ Zabbix Agent2
- `31999` สำหรับ Agent2 status port

## คำสั่งที่ใช้บ่อย

start:

```powershell
docker compose up -d
```

restart:

```powershell
docker compose restart
```

stop:

```powershell
docker compose stop
```

stop และลบ container/network:

```powershell
docker compose down
```

stop และลบ volume ด้วย:

```powershell
docker compose down -v
```

## ปัญหาที่พบบ่อย

1. container ไม่ขึ้นเพราะ secret file ไม่ครบ

ให้เช็กไฟล์ใน `env_vars/` โดยเฉพาะ `.MYSQL_USER`, `.MYSQL_PASSWORD`, `.MYSQL_ROOT_USER`, `.MYSQL_ROOT_PASSWORD`

2. เปิดเว็บไม่ได้

ให้เช็กว่าพอร์ต `80` หรือ `8081` ชนกับ service อื่นในเครื่องหรือไม่ และดู `docker compose ps`

3. Zabbix server รอ database นาน

ให้ดู log ของ `mysql-server` และ `server-db-init` ว่าการ initialize database สำเร็จหรือไม่

4. ต้องการล้างข้อมูลเพื่อเริ่มใหม่

ใช้คำสั่งนี้ด้วยความระวัง เพราะจะลบ volume และข้อมูลที่รันอยู่:

```powershell
docker compose down -v
```

## Git Workflow เบื้องต้น

ดึงงานล่าสุด:

```powershell
git pull origin main
```

ดูสถานะไฟล์:

```powershell
git status
```

commit และ push:

```powershell
git add .
git commit -m "update readme"
git push origin main
```

## หมายเหตุเพิ่มเติม

- โฟลเดอร์ `zbx_env/` ใช้เก็บไฟล์ที่ถูก mount เข้า container
- ไฟล์ log runtime ไม่ควร commit ขึ้น repo
- ถ้ามีการปรับค่าระบบเฉพาะเครื่อง ให้จดไว้ใน README หรือแยกเป็นไฟล์ config ของทีมเพื่อให้คนอื่นรันต่อได้ง่าย
