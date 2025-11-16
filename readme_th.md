<div align=center><img width="150" height="150" src="./img/logo.png"/></div>

<h1><p align='center'>GalTransl</p></h1>
<div align=center><img src="https://img.shields.io/github/v/release/XD2333/GalTransl"/>   <img src="https://img.shields.io/github/license/XD2333/GalTransl"/>   <img src="https://img.shields.io/github/stars/XD2333/GalTransl"/></div>
<p align='center'>โซลูชันแปลอัตโนมัติสำหรับ Galgame ด้วย LLM (GPT-4/Claude/Deepseek/Sakura)</p>

[中文](https://github.com/XD2333/GalTransl/blob/main/README.md) | [English](https://github.com/XD2333/GalTransl/blob/main/README_EN.md)

## เกี่ยวกับ GalTransl

GalTransl คือเครื่องมือแปลอัตโนมัติสำหรับเกม Visual Novel (Galgame) ที่ใช้ประโยชน์จาก Large Language Models เพื่อสร้างแพตช์แปลแบบฝังตัว

**คุณสมบัติหลัก**:
- รองรับ **GPT-4, Claude, Deepseek, Sakura** และโมเดลอื่นๆ พร้อมการปรับแต่ง Prompt Engineering
- **GPT字典** (GPT Dictionary) - ให้ GPT เข้าใจบุคลิกตัวละคร แปลชื่อและสรรพนามได้แม่นยำ
- ระบบดิกชันนารีอัตโนมัติ (ดิกชันนารีก่อนแปล/หลังแปล/แบบมีเงื่อนไข)
- บันทึกแคชแบบเรียลไทม์ และต่อการแปลอัตโนมัติ
- รองรับการแกะไฟล์และฉีดแพตช์สำหรับหลายเอนจิน
- รองรับการแปลไฟล์ srt, lrc, vtt, mtool json, t++ excel, epub โดยตรง
- โมเดลท้องถิ่น: [GalTransl-7B-v3.5](https://huggingface.co/SakuraLLM/GalTransl-7B-v2), [GalTransl-14B-v3](https://huggingface.co/SakuraLLM/Sakura-GalTransl-14B-v3)

**⚠️ คำเตือน**: เมื่อเผยแพร่แพตช์ที่แปลด้วยเครื่องมือนี้โดยไม่ผ่านการพิสูจน์อักษรเต็มรูปแบบ กรุณาระบุว่า "GPT翻译/AI翻译补丁" ไม่ใช่ "个人汉化" หรือ "AI汉化"

## ขั้นตอนการทำแพตช์แปล

### 1. การเตรียมสภาพแวดล้อม

**ดาวน์โหลดและติดตั้ง**:
- ดาวน์โหลด [GalTransl Release](https://github.com/XD2333/GalTransl/releases/) (มีเวอร์ชัน WinEXE ไม่ต้องติดตั้ง Python)
- ติดตั้ง Python 3.11.9 ([ดาวน์โหลด](https://www.python.org/downloads/release/python-3119/)) - **ติ๊กเลือก "Add Python to PATH"**
- รัน `安装、更新依赖.bat` เพื่อติดตั้ง dependencies

### 2. การระบุเอนจินและแกะไฟล์

- ใช้ GARbro เปิดไฟล์แพ็คเกจของเกม → ดูชื่อเอนจินที่ด้านล่างซ้าย
- หาไฟล์สคริปต์ (มักอยู่ในโฟลเดอร์ scene/scenario/message/script)
- แกะไฟล์ออกมา

### 3. การแยกข้อความและแปล

**3.1 แยกสคริปต์เป็น JSON**:
- ใช้ GalTransl_DumpInjector หรือ VNTextPatch/SExtractor
- แยกสคริปต์เป็นรูปแบบ JSON (name-message format)
```json
[
  {
    "name": "咲來",
    "message": "「ってか、白鷺学園だったらあたしと一緒じゃん。\\r\\nセンパイだったんですねー」"
  }
]
```

**3.2 ตั้งค่า GalTransl**:
- คัดลอก `sampleProject` และเปลี่ยนชื่อเป็นชื่อเกม
- เปลี่ยนชื่อ `config.inc.yaml` → `config.yaml`
- วางไฟล์ JSON ที่แยกได้ใน `gt_input/`
- แก้ไข `config.yaml`:

```yaml
backendSpecific:
  OpenAI-Compatible:
    tokens:
      - token: sk-example-key1
        endpoint: https://api.deepseek.com
        modelName: deepseek-chat
```

**3.3 เริ่มแปล**:
- รัน `run.bat` (หรือ `GalTransl.exe` สำหรับเวอร์ชัน WinEXE)
- เลือกไฟล์ config และ translator (เช่น ForGal-json)
- **แนะนำ**: ตั้งค่า GPT Dictionary ก่อนแปล (ดูหัวข้อถัดไป)

**3.4 สร้างสคริปต์ภาษาจีน**:
- ใช้เครื่องมือเดียวกับที่แยกสคริปต์ (GalTransl_DumpInjector/SExtractor)
- เลือกโฟลเดอร์สคริปต์ญี่ปุ่น → โฟลเดอร์ JSON แปลแล้ว → โฟลเดอร์บันทึกสคริปต์จีน
- คลิก "ฉีด"

### 4. การบรรจุและรองรับการแสดงผล

**การบรรจุ/ไม่บรรจุ**:
- เอนจินส่วนใหญ่รองรับการอ่านไฟล์โดยไม่ต้องบรรจุ
- สำหรับ Krkr/Krkrz: ใช้ `version.dll` จาก [KirikiriTools](https://github.com/arcusmaximus/KirikiriTools) + สร้างโฟลเดอร์ `unencrypted` วางสคริปต์

**การรองรับการแสดงผลภาษาจีน**:
- **เอนจินที่รองรับ Unicode** (Krkr, Artemis): เล่นได้เลย
- **เอนจิน SJIS**: ใช้วิธีใดวิธีหนึ่ง
  - **เส้นทาง 1**: ฉีดแบบ GBK + แก้ไขเอนจินให้รองรับ GBK
  - **เส้นทาง 2 (แนะนำ)**: ใช้ SJIS Tunnel หรือ SJIS Replacement
    - **SJIS Tunnel**: [VNTextProxy](https://github.com/arcusmaximus/VNTranslationTools#vntextproxy) + sjis_ext.bin
    - **SJIS Replacement**: [UniversalInjectorFramework](https://github.com/AtomCrafty/UniversalInjectorFramework) + ตารางแทนที่

## ฟีเจอร์หลักของ GalTransl

### GPT Dictionary (GPT字典)

ระบบสำคัญที่ช่วยให้คุณภาพการแปลดีขึ้นอย่างมาก - กำหนดชื่อตัวละครและคำศัพท์

**รูปแบบ**: `ญี่ปุ่น[TAB]จีน[TAB]คำอธิบาย`

**ตัวอย่างการกำหนดตัวละคร**:
```
フラン	芙兰	name, lady, teacher
笠間	笠间	笠間 陽菜乃's lastname, girl
陽菜乃	阳菜乃	笠間 陽菜乃's firstname, girl
张三	张三	player's name, boy
```

**ตัวอย่างการกำหนดคำศัพท์**:
```
大家さん	房东
あたし	我/人家	use '人家' when being cute
```

**ไฟล์**:
- `Dict/通用GPT字典.txt` - ดิกชันนารีทั่วไป
- `项目GPT字典.txt` - ดิกชันนารีเฉพาะโปรเจค

### การจัดการแคชและการหาข้อผิดพลาด

**แคชการแปล** (`transl_cache/`):
- แก้ไข `pre_zh` หรือ `proofread_zh` เพื่อแก้ไขคำแปล
- ลบแถว `pre_zh` เพื่อแปลใหม่
- ลบไฟล์แคชเพื่อแปลทั้งไฟล์ใหม่

**การตรวจสอบอัตโนมัติ** (กำหนดใน `config.yaml`):
```yaml
problemAnalyze:
  problemList:
    - 词频过高      # คำซ้ำมากกว่า 20 ครั้ง
    - 标点错漏      # เครื่องหมายวรรคตอนผิด
    - 残留日文      # ยังมีภาษาญี่ปุ่นเหลืออยู่
    - 丢失换行      # ขาดการขึ้นบรรทัดใหม่
    - 多加换行      # ขึ้นบรรทัดใหม่มากเกินไป
    - 比日文长      # ยาวกว่าต้นฉบับ 1.3 เท่า
    - 字典使用      # ไม่ใช้ตาม GPT Dictionary
```

ใช้ EmEditor หรือ VSCode ค้นหา `problem` ในแคชเพื่อแก้ไขปัญหา

### ระบบดิกชันนารีแบบหลายชั้น

1. **ดิกชันนารีก่อนแปล** - ปรับแต่งข้อความก่อนส่งแปล
2. **GPT Dictionary** - ป้อนบริบทให้ LLM เมื่อพบคำ
3. **ดิกชันนารีหลังแปล** - แทนที่คำหลังแปลเสร็จ
   - รองรับเงื่อนไข: `pre_jp/post_jp[TAB]เงื่อนไข[TAB]คำค้นหา[TAB]คำแทนที่`

## การตั้งค่าเพิ่มเติม

**ไฟล์คอนฟิก**: อ่านคำอธิบายใน `config.yaml` ได้เลย (อัพเดต 2025.9 อธิบายครบแล้ว)

**เอกสารเพิ่มเติม**: [GalTransl Wiki](https://github.com/xd2333/GalTransl/wiki)

## เครื่องมือที่มีประโยชน์

| ชื่อ | คำอธิบาย |
| --- | --- |
| GARbro | แกะไฟล์แพ็คเกจ [ดาวน์โหลด](https://github.com/morkt/GARbro/releases) |
| [KirikiriTools](https://github.com/arcusmaximus/KirikiriTools) | แกะ/ฉีดสำหรับ Krkr |
| [UniversalInjectorFramework](https://github.com/AtomCrafty/UniversalInjectorFramework) | Framework ฉีดแบบ SJIS |
| [VNTextProxy](https://github.com/arcusmaximus/VNTranslationTools) | Framework แบบ SJIS Tunnel |
| GalTransl_DumpInjector | GUI สำหรับ VNTextPatch |
| [SExtractor](https://github.com/satan53x/SExtractor) | เครื่องมือแยก/ฉีดสคริปต์รวม |
| [EmEditor](https://www.ghxi.com/emeditor.html) | โปรแกรมแก้ไขข้อความ |
| [VSCode](https://code.visualstudio.com/) | โปรแกรมแก้ไขข้อความ |

## อัพเดทล่าสุด

- **2025.5**: อัพเดท v6 - เพิ่ม template ForGal, GalTransl-14B-v3
- **2024.5**: อัพเดท v5 - GalTransl-7B model, รองรับหลายประเภทไฟล์
- **2024.2**: อัพเดท v4 - ระบบปลั๊กอิน
- **2023.12**: อัพเดท v3 - multi-threading
