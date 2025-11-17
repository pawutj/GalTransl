# การตั้งค่า GalTransl สำหรับแปล JP→TH ด้วย QWEN 3 + XLSX

คู่มือนี้อธิบายวิธีการตั้งค่า GalTransl เพื่อแปลจากภาษาญี่ปุ่นเป็นภาษาไทยโดยใช้ QWEN 3 และไฟล์ XLSX (T++ format)

## ✅ สิ่งที่แก้ไขแล้ว

### 1. การเพิ่มภาษาไทย (Thai Language Support)

**ไฟล์ที่แก้ไข:**
- ✅ `GalTransl/__init__.py` - เพิ่ม `"th": "Thai"` และ `"th": "ไทย"`
- ✅ `GalTransl/i18n.py` - เพิ่ม `"th"` ใน `AVAILABLE_LANGUAGES`
- ✅ `GalTransl/Problem.py` - เพิ่มการตรวจสอบอักษรไทย (U+0E00-U+0E7F)

### 2. ไฟล์ Configuration ตัวอย่าง

- ✅ `sampleProject/config_qwen_thai.yaml` - ตัวอย่าง config สำหรับ QWEN 3 + Thai

---

## 🚀 ขั้นตอนการใช้งาน

### ขั้นตอนที่ 1: เตรียม QWEN API Key

1. ไปที่ [Alibaba Cloud DashScope](https://dashscope.console.aliyun.com/)
2. สมัครบัญชีและสร้าง API Key
3. คัดลอก API Key ไว้

### ขั้นตอนที่ 2: เตรียมไฟล์ XLSX

ใช้ T++ หรือเครื่องมืออื่นแยกข้อความจากเกมเป็นไฟล์ XLSX โดยมีรูปแบบ:

| Column A | Column B | Column C |
|----------|----------|----------|
| Original Text (JP) | Name | Machine Translation (TH) |

- **Column A**: ข้อความภาษาญี่ปุ่นต้นฉบับ
- **Column B**: ชื่อตัวละคร (optional)
- **Column C**: จะถูกเติมคำแปลภาษาไทยโดย GalTransl

### ขั้นตอนที่ 3: สร้างโปรเจคใหม่

```bash
# คัดลอก sampleProject และเปลี่ยนชื่อ
cp -r sampleProject MyThaiProject
cd MyThaiProject
```

### ขั้นตอนที่ 4: ตั้งค่า config.yaml

คัดลอกไฟล์ตัวอย่างที่สร้างไว้:

```bash
cp config_qwen_thai.yaml config.yaml
```

แก้ไข `config.yaml`:

```yaml
backendSpecific:
  OpenAI-Compatible:
    tokens:
      - token: sk-your-actual-qwen-key  # ← ใส่ QWEN API key ที่นี่
        endpoint: https://dashscope.aliyuncs.com/compatible-mode/v1
        modelName: qwen-plus

plugin:
  filePlugin: file_translator++_xlsx  # ← ใช้ XLSX plugin

common:
  language: "th"  # ← ภาษาเป้าหมายคือไทย
```

### ขั้นตอนที่ 5: สร้าง GPT Dictionary (สำคัญมาก!)

สร้างไฟล์ `项目GPT字典.txt` ในโฟลเดอร์โปรเจค:

```
# รูปแบบ: ญี่ปุ่น[TAB]ไทย[TAB]คำอธิบาย

# ตัวอย่างชื่อตัวละคร
フラン	ฟลาน	name, lady, teacher, protagonist's mentor
主人公	ผู้เล่น	player, boy, high school student
咲來	ซากุระ	name, girl, classmate, cheerful personality

# ตัวอย่างคำศัพท์
先輩	รุ่นพี่	senior, used when speaking to upperclassmen
お姉ちゃん	พี่สาว	older sister, affectionate term
ごめんなさい	ขอโทษ	apology, polite form
```

**หมายเหตุ:**
- ใช้ **TAB** คั่นระหว่างคอลัมน์ (ไม่ใช่เว้นวรรค!)
- คำอธิบายช่วย QWEN เข้าใจบริบท (เพศ, อายุ, ความสัมพันธ์)
- แยกชื่อและนามสกุลเป็นคนละบรรทัด

### ขั้นตอนที่ 6: วางไฟล์ XLSX

```bash
# วางไฟล์ XLSX ในโฟลเดอร์ gt_input
mkdir -p gt_input
cp your_game.xlsx gt_input/
```

### ขั้นตอนที่ 7: เริ่มแปล

```bash
# Windows
python run_GalTransl.py

# หรือ Linux/Mac
python -m GalTransl -p ./MyThaiProject -t ForGal-tsv
```

เลือกตัวเลือก:
1. ใส่ path ของโปรเจค: `./MyThaiProject`
2. เลือก translator: **ForGal-tsv** (แนะนำ, ประหยัด token) หรือ **ForGal-json**

---

## 📊 การตรวจสอบผลลัพธ์

### ตรวจสอบไฟล์แปล

ผลลัพธ์จะอยู่ใน:
- `gt_output/` - ไฟล์ XLSX ที่แปลแล้ว
- `transl_cache/` - แคชการแปล (JSON format)

### ตรวจสอบปัญหาในแคช

เปิด `transl_cache/*.json` ด้วย VSCode หรือ EmEditor และค้นหา `"problem"`:

```json
{
  "index": 42,
  "pre_jp": "先輩、お疲れ様です！",
  "pre_zh": "รุ่นพี่ สู้ๆนะครับ!",
  "problem": "残留日文: 様"  // ← มีภาษาญี่ปุ่นเหลือ!
}
```

### ประเภทปัญหาที่ตรวจพบ

| ปัญหา | ความหมาย | วิธีแก้ |
|-------|----------|---------|
| `残留日文` | ยังมีภาษาญี่ปุ่นเหลืออยู่ | เพิ่มใน GPT Dictionary |
| `语言不通-非泰语输出` | แปลเป็นภาษาอื่น (ไม่ใช่ไทย) | ตรวจสอบ prompt/model |
| `词频过高` | ใช้คำซ้ำมากเกินไป (>20 ครั้ง) | ปรับ batch size |
| `字典使用` | ไม่ใช้ GPT Dictionary | เพิ่มคำใน dictionary |
| `比日文长` | ยาวกว่าต้นฉบับ 1.3 เท่า | ตรวจสอบคำแปล |

### แก้ไขปัญหาในแคช

1. เปิดไฟล์ `transl_cache/your_file.json`
2. แก้ไขฟิลด์ `pre_zh` (คำแปลหลัก)
3. **ลบบรรทัด** `pre_zh` ถ้าต้องการแปลใหม่ (ห้ามเว้นบรรทัดว่าง!)
4. รัน GalTransl อีกครั้ง → จะ rebuild ไฟล์ผลลัพธ์

---

## ⚙️ การปรับแต่งขั้นสูง

### 1. ปรับ Batch Size

```yaml
common:
  gpt:
    numPerRequestTranslate: 10  # ลดเป็น 5 ถ้าแปลผิดบ่อย
    contextNum: 8              # เพิ่มเป็น 10 ถ้าต้องการบริบทมากขึ้น
```

### 2. เพิ่ม Custom Prompt

```yaml
common:
  gpt:
    change_prompt: "AdditionalPrompt"
    prompt_content: |
      Please use appropriate Thai politeness levels:
      - Use ครับ/ค่ะ for polite/formal speech
      - Use นะ/จ้า for casual/friendly speech
      - Use จ๊ะ/จ้ะ for cute/childish speech
      - Adjust based on character age, gender, and relationship
```

### 3. เปิด/ปิดการตรวจสอบปัญหา

```yaml
problemAnalyze:
  problemList:
    - 词频过高
    - 残留日文
    - 语言不通      # ตรวจสอบว่าเป็นภาษาไทย
    # - 引入英文    # ปิดถ้าเกมมีคำอังกฤษอยู่แล้ว
```

### 4. เพิ่ม Token หลายตัว

```yaml
backendSpecific:
  OpenAI-Compatible:
    tokens:
      - token: sk-key-1
        endpoint: https://dashscope.aliyuncs.com/compatible-mode/v1
        modelName: qwen-plus
      - token: sk-key-2
        endpoint: https://dashscope.aliyuncs.com/compatible-mode/v1
        modelName: qwen-turbo  # ใช้โมเดลเล็กกว่าเพื่อประหยัด
    tokenStrategy: "random"  # สุ่มใช้ token → load balancing
```

---

## 🐛 แก้ปัญหาที่พบบ่อย

### ปัญหา: QWEN แปลเป็นภาษาจีนแทนภาษาไทย

**วิธีแก้:**
1. ตรวจสอบ `common.language: "th"` ใน config
2. ตรวจสอบว่า GPT Dictionary มีตัวอย่างภาษาไทย
3. เพิ่ม custom prompt บังคับให้แปลเป็นไทย:
   ```yaml
   gpt.prompt_content: "Always translate to Thai language (ภาษาไทย), never output Chinese or English."
   ```

### ปัญหา: แปลผิดบ่อย / JSON parse error

**วิธีแก้:**
1. ลด batch size: `gpt.numPerRequestTranslate: 5`
2. เปิด smart retry: `smartRetry: true`
3. เปลี่ยนเป็น `ForGal-json` (ช้ากว่าแต่แม่นยำกว่า)

### ปัญหา: API Error 429 (Too Many Requests)

**วิธีแก้:**
1. เพิ่มเวลารอ: `apiErrorWait: 120`
2. ลดจำนวน workers: `workersPerProject: 8`
3. ใช้หลาย API keys (ดูข้อ 4 ในการปรับแต่งขั้นสูง)

### ปัญหา: ไฟล์ XLSX ไม่โหลด

**วิธีแก้:**
1. ตรวจสอบว่ามี column A (Original Text)
2. ตรวจสอบว่าไฟล์เป็น `.xlsx` (ไม่ใช่ `.xls`)
3. ลองเปิดไฟล์ด้วย Excel และ Save As ใหม่

---

## 📚 ทรัพยากรเพิ่มเติม

- **QWEN API Docs**: https://help.aliyun.com/zh/dashscope/
- **GalTransl Wiki**: https://github.com/xd2333/GalTransl/wiki
- **T++ Tool**: ใช้แยกข้อความจากเกม Visual Novel

---

## 🎯 Checklist การใช้งานครั้งแรก

- [ ] สมัครและรับ QWEN API Key
- [ ] ติดตั้ง GalTransl และ dependencies (`安装、更新依赖.bat`)
- [ ] สร้างโปรเจคใหม่จาก sampleProject
- [ ] คัดลอก `config_qwen_thai.yaml` → `config.yaml`
- [ ] ใส่ QWEN API key ใน config
- [ ] สร้าง `项目GPT字典.txt` พร้อมชื่อตัวละคร
- [ ] วางไฟล์ XLSX ใน `gt_input/`
- [ ] **แปลแค่ 1 ไฟล์ก่อน** (ทดสอบ)
- [ ] ตรวจสอบผลลัพธ์ใน `gt_output/`
- [ ] ตรวจสอบปัญหาใน `transl_cache/`
- [ ] แก้ไขแคชถ้ามีปัญหา
- [ ] แปลไฟล์ทั้งหมด

---

## 💡 เคล็ดลับสำหรับคุณภาพการแปล

### 1. GPT Dictionary คือกุญแจสำคัญที่สุด
- ใส่ชื่อตัวละครทุกคน พร้อมเพศ/อายุ/ความสัมพันธ์
- ใส่คำศัพท์เฉพาะของเกม (อาวุธ, สถานที่, เทคนิค)
- ตัวอย่างดี: `主人公	ผู้เล่น	player, 18 years old boy, high school student`

### 2. เริ่มจากไฟล์เล็กๆ
- แปล 1 ไฟล์ก่อน (100-200 ประโยค)
- ตรวจสอบคุณภาพ
- ปรับ config ถ้าจำเป็น
- ค่อยแปลทั้งหมด

### 3. ใช้ Context
- `contextNum: 8` ให้ QWEN เห็นประโยคก่อนหน้า
- ช่วยให้คำแปลสอดคล้องและมีความต่อเนื่อง

### 4. ตรวจสอบและแก้ไขเสมอ
- ค้นหา `"problem"` ในแคช
- แก้ไขคำแปลที่ผิด
- เพิ่มคำใน GPT Dictionary ถ้าพบคำใหม่

### 5. ใช้ QWEN Model ที่เหมาะสม
- `qwen-turbo`: เร็ว, ถูก, เหมาะสำหรับการทดสอบ
- `qwen-plus`: สมดุล, แนะนำสำหรับงานปกติ
- `qwen-max`: แม่นยำที่สุด, แพง, ใช้สำหรับคำแปลสำคัญ

---

**สนุกกับการแปลเกม Visual Novel เป็นภาษาไทย! 🎮🇹🇭**
