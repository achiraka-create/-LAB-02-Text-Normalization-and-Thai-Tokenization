# **Overall Discussion**





This experiment compared three Thai word tokenization methods using 20,000 restaurant reviews and 3,000 golden-reference reviews.



Newmm achieved the highest boundary F1-score (94.01%) and the fastest processing time (5.58 seconds). Longest achieved an F1-score of 93.79%, while AttaCut achieved 93.68%.



AttaCut produced the most tokens and achieved the highest recall, but it also created more incorrect word boundaries. Error analysis showed that common problems involved compound words, proper names, and uncommon food-related terms.



Minimal normalization preserved important information, including negation and emotional expressions. Overall, newmm provided the best balance of accuracy and processing speed for this dataset.



For negation cases (1,254 reviews), newmm achieved a boundary F1-score of 94.13%. Minimal normalization preserved negation words such as "ไม่".



For mixed-language cases (321 reviews), newmm achieved a boundary F1-score of 94.39%. The longest tokenizer showed lower precision, partly reflecting difficulties in handling mixed-language text.



Overall, newmm achieved the highest F1-score in both groups.



# Google Colab Discussion





#### **Step 1 — Data Preparation (Cell 1–6.1)**

Cell 1 — Import Libraries



นำเข้าไลบรารีที่จำเป็นและกำหนด SEED = 42 ได้ Python 3.13.18 และ Pandas 2.2.3 การกำหนด Seed ทำให้สุ่มข้อมูลชุดเดิมได้เมื่อทดลองซ้ำ ช่วยให้ผลการทดลองตรวจสอบย้อนหลังได้



Cell 2 — Upload Dataset



อัปโหลดไฟล์รีวิวร้านอาหารและ Golden Reference เข้าสู่ Google Colab เพื่อเตรียมใช้ในการทดลอง Cell นี้ไม่ได้คำนวณคะแนน แต่เป็นขั้นตอนนำเข้าข้อมูล



Cell 3 — Load and Inspect Data



อ่านไฟล์ข้อมูลด้วย Pandas พบรีวิวทั้งหมด 100,000 ข้อความ และ Golden Reference 3,000 ข้อความ แสดงว่ามีทั้งข้อมูลสำหรับทดลองและชุดคำตอบอ้างอิงสำหรับวัดความถูกต้อง



Cell 4 — Sample 20,000 Reviews



สุ่มรีวิว 20,000 ข้อความจากทั้งหมด 100,000 ข้อความ โดยใช้ Seed 42 และเรียงตาม review\_id เพื่อให้ได้ชุดข้อมูลคงที่และสามารถเปรียบเทียบทุก Pipeline บนข้อมูลเดียวกัน



Cell 5 — Freeze Golden Evaluation Set



ตรวจสอบรหัสรีวิวและข้อความว่าตรงกับชุดที่สุ่มไว้ จากนั้นบันทึก Golden Reference จำนวน 3,000 ข้อความ เป็น frozen\_eval.csv พร้อมค่า SHA256 สำหรับตรวจสอบว่าไฟล์ไม่ถูกเปลี่ยนแปลง



Cell 6 — Save Sampled Dataset



บันทึกรีวิวที่สุ่มมา 20,000 ข้อความเป็น sampled\_20000.csv เพื่อเก็บชุดข้อมูลที่ใช้ทดลอง และทำให้สามารถนำกลับมาทดลองซ้ำได้



Cell 6.1 — Experiment Settings



แสดงการตั้งค่าการทดลอง ได้แก่ Dataset 100,000 รีวิว, Sample 20,000 รีวิว, Golden Set 3,000 รีวิว และตัวตัดคำ 3 วิธี เป็นหลักฐานว่าแต่ละ Pipeline ใช้เงื่อนไขเดียวกัน



#### **Step 2 — Text Normalization (Cell 7–9)**



Cell 7 — Create Normalization Function



สร้างฟังก์ชัน normalize\_minimal() เพื่อจัดรูปแบบ Unicode เป็น NFC ลบอักขระที่มองไม่เห็นบางชนิด และจัดช่องว่างให้สม่ำเสมอ โดยไม่ลบคำหรืออักษรที่มีผลต่อความหมาย



Cell 8 — Test Normalization



ทดลองกับข้อความตัวอย่าง พบว่าช่องว่างหลายตำแหน่งถูกปรับให้เหลือหนึ่งช่อง และอักขระ Zero-width ถูกลบออก ขณะที่คำปฏิเสธ ไม่ และคำว่า อร่อยยยยมากกก ยังคงเดิม แสดงว่า Normalization ไม่ทำลายข้อมูลสำคัญต่อความหมายและอารมณ์ของรีวิว



Cell 9 — Apply Normalization to Dataset



นำฟังก์ชัน Normalization ไปใช้กับรีวิว 20,000 ข้อความและ Golden Set 3,000 ข้อความ เพื่อให้ข้อความอยู่ในรูปแบบมาตรฐานเดียวกันก่อนตัดคำและประเมินผล



#### **Step 3 — Thai Tokenization (Cell 10–16)**



Cell 10 — Install Libraries



ติดตั้ง PyThaiNLP 5.4.0 และ AttaCut 1.0.6 เพื่อใช้ทดลองตัดคำภาษาไทย การระบุเวอร์ชันช่วยให้สามารถทำการทดลองซ้ำในสภาพแวดล้อมใกล้เคียงกันได้



Cell 11 — Test Three Tokenizers



ทดลองตัดประโยค อาหารอร่อยมากแต่บริการไม่ดี ด้วย newmm, longest และ attacut พบว่าทั้งสามวิธีแบ่งเป็น 7 คำเหมือนกัน แสดงว่าตัวตัดคำทั้งสามทำงานได้ แต่ยังไม่สามารถสรุปความแตกต่างด้านคุณภาพจากประโยคเดียวได้



Cell 12 — Create Three Tokenization Pipelines



สร้างฟังก์ชันตัดคำร่วมกันและกำหนด Pipeline A = newmm, B = longest และ C = attacut โดยใช้ keep\_whitespace=False เพื่อให้ทุกวิธีทดลองภายใต้เงื่อนไขเดียวกัน



Cell 13 — Run Three Tokenizers on 20,000 Reviews



ทดลองตัดคำทั้งหมด 20,000 รีวิว ได้ผลดังนี้

Pipeline	เวลา	จำนวน Token

A — newmm	5.58 วินาที	477,094

B — longest	57.22 วินาที	482,651

C — attacut	85.48 วินาที	502,928



newmm ประมวลผลเร็วที่สุด ส่วน AttaCut สร้าง Token มากที่สุด แสดงว่าตัวตัดคำแต่ละวิธีมีลักษณะการแบ่งคำและความเร็วแตกต่างกัน แต่จำนวน Token มากไม่ได้หมายถึงความถูกต้องสูงกว่า



Cell 14 — Compare Tokenization Outputs



แสดงผลตัดคำของทั้งสาม Pipeline เปรียบเทียบกัน เช่น ดีจริง ถูกแบ่งเป็นคำเดียวใน newmm และ longest แต่ AttaCut แบ่งเป็น ดี | จริง แสดงว่าแต่ละวิธีสามารถกำหนดขอบเขตคำต่างกัน แม้ใช้ข้อความเดียวกัน



Cell 15 — Inspect Golden Reference Tokens



ตรวจสอบว่าเมื่อนำคำจาก Golden Reference มาต่อกันจะตรงกับข้อความต้นฉบับหรือไม่ พบตรงกันเพียง 60 ข้อความ และไม่ตรง 2,940 ข้อความ เนื่องจากการต่อคำอ้างอิงไม่ได้รักษาช่องว่างเหมือนต้นฉบับ จึงต้องปรับวิธีตรวจสอบ



Cell 16 — Check Golden References Without Spaces



ตรวจสอบอีกครั้งโดยไม่นับช่องว่าง พบว่าตรงกันครบ 3,000 ข้อความ และไม่ตรง 0 ข้อความ แสดงว่าสามารถเปรียบเทียบตำแหน่งขอบเขตคำโดยใช้กติกาไม่นับช่องว่างได้



#### **Step 4 — Boundary Evaluation (Cell 17–19)**



Cell 17 — Create Boundary Evaluation Functions



สร้างฟังก์ชันหาตำแหน่งขอบเขตคำและคำนวณ Precision, Recall และ F1 โดยไม่นับช่องว่างและขอบเขตสุดท้ายของข้อความ



ผลทดสอบได้ Precision = 1.00, Recall = 0.67 และ F1 = 0.80 หมายความว่าขอบเขตที่ทำนายมาถูกทั้งหมด แต่ยังตรวจพบขอบเขตคำไม่ครบ



Cell 18 — Evaluate Three Pipelines on Golden Set



ประเมินตัวตัดคำทั้งสามด้วย Golden Reference 3,000 ข้อความ ได้ผลดังนี้

Pipeline	Precision	Recall	F1

newmm	95.33%	92.72%	94.01%

longest	94.56%	93.02%	93.79%

attacut	92.32%	95.07%	93.68%



newmm มี F1 สูงที่สุด แสดงว่ามีสมดุลระหว่าง Precision และ Recall ดีที่สุดในชุดข้อมูลนี้ ส่วน AttaCut มี Recall สูงที่สุด แต่ตรวจพบขอบเขตคำเกินจาก Golden Reference มากกว่า



Cell 18.1 — Debug Alignment Error



ตรวจสอบรีวิว R000929 หลังพบปัญหา Alignment พบว่า longest เปลี่ยนคำว่า Juicy เป็น juicy ทำให้ข้อความไม่ตรงกันแบบตัวอักษร แต่จำนวนและตำแหน่งอักขระยังเท่าเดิม



Cell 18.2 — Check Alignment for All Pipelines



ตรวจสอบข้อมูลทั้ง 3 Pipeline พบความแตกต่าง 63 กรณี และทั้งหมดเกิดจากตัวพิมพ์ใหญ่–เล็ก ไม่มีกรณีอื่น จึงใช้การตรวจสอบแบบไม่สนใจตัวพิมพ์ใหญ่–เล็ก โดยคงตำแหน่งอักขระสำหรับประเมิน Boundary



Cell 19 — Save Evaluation Metrics



บันทึกคะแนน Precision, Recall, F1 และจำนวน TP, FP, FN เป็น metrics.csv เพื่อใช้เปรียบเทียบประสิทธิภาพและนำผลไปเขียนรายงาน

#### 

#### **Step 5 — Error Analysis (Cell 20–23)**



Cell 20 — Save Paired Tokenization Outputs



บันทึกผลการตัดคำของทั้งสาม Pipeline เทียบกับ Golden Reference ครบ 3,000 รีวิว เป็น paired\_outputs.csv ช่วยให้ตรวจสอบได้ว่าตัวตัดคำแต่ละวิธีแบ่งคำแตกต่างจากคำตอบอ้างอิงตรงไหน



Cell 21 — Identify Tokenization Errors



ตรวจพบรายการเปรียบเทียบที่มีข้อผิดพลาดรวม 7,613 รายการ แบ่งเป็น

Pipeline	จำนวนรายการที่ผิด

newmm	2,522

longest	2,501

attacut	2,590



AttaCut มีจำนวนรายการผิดมากที่สุด และพบ Over-segmentation มากกว่าอีกสองวิธี ส่วนตัวเลขนี้เป็นจำนวนรายการเปรียบเทียบรีวิว–Pipeline ที่ผิด ไม่ใช่จำนวนขอบเขตคำที่ผิดทั้งหมด



Cell 22 — Select Five Error Examples



เลือก 5 กรณีที่มีจำนวนข้อผิดพลาดมากเพื่อศึกษา พบข้อผิดพลาดจากการรวมคำที่ควรแยก และการแยกคำที่ควรรวม เช่น ชื่อร้าน ชื่อบุคคล และคำศัพท์อาหาร อย่างไรก็ตาม ทั้ง 5 กรณีเป็นประเภท Mixed จึงยังไม่ครอบคลุมข้อผิดพลาดทุกประเภท



Cell 23 — Select Diverse Error Examples



ปรับการเลือกเป็น 6 กรณี เพื่อให้ครอบคลุม Under-segmentation, Over-segmentation และ Mixed เช่น การแบ่ง ซาชิมิ เป็น ซาชิ | มิ และการรวมคำ ครั้งแรก ซึ่งแตกต่างจาก Golden Reference ช่วยให้วิเคราะห์ลักษณะข้อผิดพลาดของแต่ละ Pipeline ได้ชัดเจนขึ้น



#### **Step 6 — Save and Reuse (Cell 24–26)**



Cell 24 — Create preprocess.py



รวบรวมฟังก์ชัน Normalization, Tokenization และ Boundary Extraction ไว้ในไฟล์ preprocess.py เพื่อให้สามารถนำไปใช้ซ้ำได้ โดยไม่ต้องเขียนโค้ดใหม่ทุกครั้ง



Cell 25 — Test preprocess.py



ทดสอบไฟล์ที่สร้างด้วยข้อความ อาหาร   อร่อยมาก พบว่าปรับเป็น อาหาร อร่อยมาก และตัดเป็น อาหาร | อร่อย | มาก พร้อมขอบเขตตำแหน่ง 5 และ 10 แสดงว่าฟังก์ชันที่บันทึกไว้ทำงานได้ถูกต้องตามที่กำหนด



Cell 26 — Download Lab Files



รวบรวมไฟล์ผลการทดลอง 6 ไฟล์ ได้แก่ Dataset ที่สุ่มไว้, Frozen Reference, Metrics, Paired Outputs, Error Analysis และ preprocess.py ลงใน Lab2\_1\_Results.zip เพื่อจัดเก็บและใช้เป็นหลักฐานประกอบการส่งงาน



Cell 27 — Evaluate Negation and Mixed-language Cases

Cell นี้ประเมินความแม่นยำของการตัดคำในข้อความ 2 ประเภท ได้แก่ Negation Case (1,254 รีวิว) และ Mixed-language Case (321 รีวิว) โดยคำนวณ Precision, Recall และ F1 ของทั้ง 3 Pipelines



ผลการทดลองพบว่า newmm มีค่า F1 สูงที่สุดทั้งสองกรณี คือ 94.13% สำหรับ Negation และ 94.39% สำหรับ Mixed-language ขณะที่ AttaCut มีค่า Recall สูงที่สุดทั้งสองกรณี



แสดงให้เห็นว่า newmm มีความสมดุลระหว่าง Precision และ Recall ดีที่สุด สำหรับข้อมูลทั้งสองกลุ่ม ส่วน AttaCut สามารถตรวจพบขอบเขตคำได้มากกว่า แต่มีแนวโน้มแบ่งคำเกินจาก Golden Reference มากกว่าเช่นกัน



