---
name: L3 Control Center
description: OBS Lower Third Controller with Excel data import and dynamic card management
---

# L3 Control Center - OBS Dock Controller

## Overview

แอปพลิเคชันสำหรับควบคุม Lower Third (L3) ใน OBS Studio ผ่าน WebSocket
- นำเข้าข้อมูลจาก Excel
- จัดการ Cards แบบ Single (1 คน) และ Double (2 คน)
- สร้าง OBS Sources อัตโนมัติ
- ควบคุม visibility ของ L3

---

## Project Structure

```
L3/
├── L3_Control_Center.html    # Main HTML (load in OBS Dock)
├── css/
│   └── style.css             # All styles
└── js/
    └── app.js                # All JavaScript logic
```

---

## Key Components

### 1. OBS Connection
- ใช้ Native WebSocket (ไม่ใช้ obs-websocket-js)
- เชื่อมต่อที่ `ws://localhost:4455`
- Auto-reconnect ทุก 5 วินาทีเมื่อหลุด

### 2. Card Types
| Type | Description | OBS Sources |
|------|-------------|-------------|
| Single | แสดง 1 คน | `T_Name_N`, `T_Label1_N`, `T_Label2_N`, `T_Label3_N`, `I_Avatar_N` |
| Double | แสดง 2 คน (Home/Away) | `T_Name_Home_N`, `T_Name_Away_N`, etc. |

### 3. Excel Columns
```
ID | name | LabelName1 | LabelName2 | LabelName3 | image
```

---

## OBS Source Naming Convention

### Single Card (Card ID = 1)
- **Group**: `G_LowerThird_1`
- **Text Name**: `T_Name_1`
- **Text Labels**: `T_Label1_1`, `T_Label2_1`, `T_Label3_1`
- **Image Avatar**: `I_Avatar_1`

### Double Card (Card ID = 2)
- **Home Group**: `G_LowerThird_Home_2`
- **Home Sources**: `T_Name_Home_2`, `T_Label1_Home_2`, `I_Avatar_Home_2`
- **Away Group**: `G_LowerThird_Away_2`
- **Away Sources**: `T_Name_Away_2`, `T_Label1_Away_2`, `I_Avatar_Away_2`

---

## Key Functions in app.js

| Function | Description |
|----------|-------------|
| `connectOBS()` | เชื่อมต่อ OBS WebSocket |
| `sendOBSRequest()` | ส่ง request ไปยัง OBS |
| `sendOBSBatch()` | ส่ง batch requests |
| `processFile()` | อ่านไฟล์ Excel |
| `renderCards()` | แสดง Cards ทั้งหมด |
| `applySingleL3()` | อัปเดต Single Card → OBS |
| `applyDoubleL3()` | อัปเดต Double Card → OBS |
| `toggleSingleVisibility()` | เปิด/ปิด Single L3 |
| `toggleDoubleVisibility()` | เปิด/ปิด Double L3 |
| `createGroupWithSources()` | สร้าง OBS Sources อัตโนมัติ |

---

## LocalStorage Keys

| Key | Description |
|-----|-------------|
| `l3_cards` | เก็บ Card configuration (JSON array) |

---

## How to Use in OBS

1. เปิด OBS Studio
2. ไปที่ **Docks → Custom Browser Docks**
3. ตั้งชื่อ Dock: `L3 Control`
4. ใส่ URL: `file:///PATH/TO/L3_Control_Center.html`
5. กด OK

---

## Modification Tips

### เพิ่ม Label ใหม่
1. แก้ไข `renderSingleCard()` / `renderDoubleCard()` ใน `app.js`
2. เพิ่ม input ใน HTML ถ้าต้องการ
3. แก้ไข `applySingleL3()` / `applyDoubleL3()` เพื่อส่งไป OBS

### เปลี่ยนสี Theme
- แก้ไขใน `css/style.css`
- สี Primary: `#00aaff`
- สี Single Card: `#4ade80`
- สี Double Card: `#fbbf24`
- Background: `#1e1e1e`

### เพิ่ม OBS Actions ใหม่
- ใช้ `sendOBSRequest(requestType, requestData)` 
- ดู OBS WebSocket Protocol: https://github.com/obsproject/obs-websocket/blob/master/docs/generated/protocol.md
