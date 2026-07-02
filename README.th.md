# Kubernetes 101: From Zero to Hero - คู่มือการสาธิต (Demo Guide)

[Read in English](file:///Users/thapanutlurejunda/work/kubectl/README.md)

คู่มือนี้ออกแบบมาเพื่อให้ผู้สอนทำตามในระหว่างการสาธิตแบบสด (Live Demonstration)

## สิ่งที่ต้องเตรียม (Prerequisites)
- ติดตั้งและรัน Docker Desktop เรียบร้อยแล้ว
- ติดตั้ง `kubectl` CLI เรียบร้อยแล้ว
- ติดตั้ง `helm` CLI เรียบร้อยแล้ว
- ติดตั้ง `kind` CLI เรียบร้อยแล้ว (`brew install kind` สำหรับ macOS)

---

## ส่วนที่ 1 (PART 01): พื้นฐานของ Kubernetes (Kubernetes Fundamentals)

เราจะเริ่มต้นด้วยการติดตั้ง Nginx web server แบบพื้นฐานโดยใช้ทรัพยากรหลักของ Kubernetes
*ตรวจสอบให้แน่ใจว่าเปิดใช้งาน Kubernetes ที่มาพร้อมกับ Docker Desktop แล้ว หรือสามารถใช้ kind แทนได้*

### 1. Pod
Pod คือหน่วยที่เล็กที่สุดที่สามารถติดตั้ง (deploy) ได้ใน Kubernetes ลองสร้างกันเลย:
```bash
kubectl apply -f part1-fundamentals/01-pod.yaml
kubectl get pods
```

### 2. Deployment
Pod เป็นสิ่งที่ไม่ถาวร (ephemeral) ส่วน Deployment จะคอยจัดการ Pod และช่วยในการกู้คืนตัวเอง (self-healing) รวมถึงการปรับขนาด (scaling)
```bash
# ก่อนอื่น มาลบ pod เดี่ยวที่เคยสร้างไว้เพื่อป้องกันความสับสน
kubectl delete pod nginx-pod

# จากนั้นใช้คำสั่งสร้าง Deployment
kubectl apply -f part1-fundamentals/02-deployment.yaml
kubectl get deployments
kubectl get pods
```

**สาธิตการกู้คืนตัวเอง (Self-healing):**
```bash
# ลบ Pod ตัวหนึ่งออก และคอยดู Deployment สร้าง Pod ใหม่ขึ้นมาแทนที่ทันที
kubectl delete pod <ชื่อ-pod-จากขั้นตอนก่อนหน้า>
kubectl get pods
```

### 3. Service
Deployment ของเรากำลังทำงานอยู่ แต่ยังไม่สามารถเข้าถึงได้จากเว็บเบราว์เซอร์ของเรา เราจึงจำเป็นต้องใช้ Service
```bash
kubectl apply -f part1-fundamentals/03-service.yaml
kubectl get services
```
*หมายเหตุ: หากใช้ Docker Desktop ตัว Service ประเภท `LoadBalancer` จะเปิดให้เข้าถึงได้โดยตรงที่ `localhost:80` ลองเปิด http://localhost ในเบราว์เซอร์ของคุณเพื่อดูหน้าเว็บ Nginx!*

---

## ส่วนที่ 2 (PART 02): การจัดการแพ็กเกจขั้นสูงด้วย Helm (Advanced Package Management)

การจัดการไฟล์ YAML จำนวนมากอาจมีความซับซ้อน Helm จึงเข้ามาทำหน้าที่เป็นตัวจัดการแพ็กเกจ (Package Manager) สำหรับ Kubernetes

```bash
cd part2-helm
```

### 1. สำรวจ Chart
แสดงโครงสร้างไดเรกทอรี `my-web-app` ที่สร้างขึ้นโดย `helm create` ให้ผู้เรียนดู โดยชี้ให้เห็น `Chart.yaml`, `values.yaml` และโฟลเดอร์ `templates/`

### 2. ติดตั้ง Chart (ค่าเริ่มต้น)
ติดตั้ง Nginx chart ตามค่าเริ่มต้น
```bash
helm install my-release ./my-web-app
kubectl get pods
```

### 3. อัปเกรดด้วยการกำหนดค่าเอง (จำลองสถานการณ์จริงบน Production)
เราต้องการ replicas เพิ่มขึ้นและต้องการใช้ LoadBalancer สำหรับสภาพแวดล้อม "production" เราจึงใช้ไฟล์ `values-prod.yaml` เพื่อแก้ไขค่าเริ่มต้น
```bash
helm upgrade my-release ./my-web-app -f values-prod.yaml
kubectl get pods  # คุณควรจะเห็น replicas จำนวน 3 ตัวในขณะนี้
kubectl get svc   # คุณควรจะเห็น LoadBalancer
```

### 4. การเข้าถึงแอปพลิเคชัน (Port Forwarding)
เนื่องจากเรากำลังใช้ Local Cluster (`kind`) ดังนั้น `EXTERNAL-IP` ของ LoadBalancer จะยังคงสถานะเป็น `<pending>` เราจึงต้องใช้การทำ Port Forwarding เพื่อเข้าถึงแอปพลิเคชันจากเครื่องของเรา
```bash
# รันคำสั่งนี้ค้างเอาไว้ใน Terminal
kubectl port-forward svc/my-release-my-web-app 8080:80
```
*เปิดเบราว์เซอร์ไปที่ http://localhost:8080 เพื่อดูหน้าเว็บแอปพลิเคชัน (กด `Ctrl + C` ใน Terminal เพื่อหยุดการทำงาน)*

```bash
# ล้างข้อมูล (Cleanup)
helm uninstall my-release
cd ..
```

---

## ส่วนที่ 3 (PART 03): เวิร์กช็อป - การกำหนดตารางเวลาทำงานแบบหลายโหนดขั้นสูง (Lab - Advanced Multi-Node Scheduling)

Docker Desktop มีโหนดให้ใช้เพียงโหนดเดียวเท่านั้น ในการสาธิตฟีเจอร์แบบหลายโหนด (Multi-Node) เช่น Anti-Affinity เราจะใช้ `kind` เพื่อสร้างคลัสเตอร์แบบหลายโหนดในเครื่องคอมพิวเตอร์ของเรา

### 1. สร้างคลัสเตอร์แบบหลายโหนด (Multi-Node Cluster)
เราจะสร้างคลัสเตอร์ที่มี 1 Control Plane และ 2 Worker nodes
```bash
kind create cluster --name k8s-101-demo --config part3-multinode/kind-config.yaml
```

ตรวจสอบว่าโหนดต่าง ๆ พร้อมใช้งานแล้ว:
```bash
kubectl get nodes
```
*คุณควรจะเห็น `k8s-101-demo-control-plane`, `k8s-101-demo-worker` และ `k8s-101-demo-worker2`*

### 2. สาธิตการทำงานของ Pod Anti-Affinity
เราต้องการให้แน่ใจว่า Pod ของ Nginx ที่ตั้งค่าให้มีความพร้อมใช้งานสูง (High-Availability) จะไม่ถูกกำหนดให้ทำงานบนโหนดผู้ใช้บริการ (Worker node) เดียวกัน เพื่อที่ว่าหากโหนดใดโหนดหนึ่งขัดข้อง (down) แอปพลิเคชันจะยังคงทำงานต่อไปได้

```bash
kubectl apply -f part3-multinode/01-deployment-anti-affinity.yaml
```

ตรวจสอบว่า Pod ต่าง ๆ ทำงานอยู่คนละโหนดกัน:
```bash
kubectl get pods -o wide
```
*สังเกตที่คอลัมน์ `NODE` คุณควรจะเห็น Pod หนึ่งทำงานบน `worker` และอีก Pod หนึ่งทำงานบน `worker2`*

### 3. ล้างข้อมูล (Cleanup)
เมื่อทำการสาธิตเสร็จสิ้นแล้ว คุณสามารถลบคลัสเตอร์ kind เพื่อคืนทรัพยากรให้กับระบบได้
```bash
kind delete cluster --name k8s-101-demo
```
