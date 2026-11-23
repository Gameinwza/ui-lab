# UI lab results

Tool parts that I chose: Part 3 Robot Framework and Part 4 Cypress

## Part 0
- Step 0.1, Chrome version: 157.0.8103.79 (Official Build) (arm64)
- Step 0.3, ChromeDriver version: ChromeDriver 157.0.8103.79
- Step 0.4, locator of Pizza, Please!: `button[name="pizza-please"]`
- Step 0.4, locator of Excluded tools: `select`
- Step 0.4, locator of Must be vegetarian: `#default-checkbox`
- Step 0.4, locator of Custom Pizza Name: `input#pizza-name`
- Step 0.5, pizza name, tool and calories of the run by hand: Lab Pizza 20261123-174213, Pizza cutter, 475 calories per slice

| Step | Actual result | Pass or fail |
|------|---------------|--------------|
| 1    | The heading reads "Looking to break out of your pizza routine?" | Pass |
| 2    | Max Calories per Slice (1000), Excluded tools, Must be vegetarian and Custom Pizza Name appear | Pass |
| 3    | Max Calories shows 600 (replaced 1000), Knife selected, vegetarian ticked, name Lab Pizza 20261123-174213 | Pass |
| 4    | Name: Lab Pizza 20261123-174213, Tool: Pizza cutter, Calories per slice: 475 | Pass |
| 5    | The page says "Please log in first." | Pass |
| 6    | Both boxes already held admin; after Sign in the page shows "Latest pizza recommendations" | Pass |
| 7    | Lab Pizza 20261123-174213 (tool=Pizza cutter, ingredients_number=7), same tool as step 4 (second line, another user's pizza was newest) | Pass |

## Part 3: Robot Framework
- robot --version: Robot Framework 7.5.1 (Python 3.13.11 on darwin)
- Step 3.2, failure message for the wrong title: `Title should have been 'Web Form' but was 'Web form'.`
- Step 3.8, run 1, result, elapsed time and pizza name: PASS, 00:00:07.098, Lab Pizza 20261123-173831 (tool Scissors, 425 calories per slice)
- Step 3.8, run 2, result, elapsed time and pizza name: PASS, 00:00:06.917, Lab Pizza 20261123-173838 (tool Scissors, 475 calories per slice)

## Part 4: Cypress
- node --version, npm --version and npx cypress version: v24.12.0, 11.10.0, Cypress 16.2.1
- Step 4.3, failure message for the wrong title: `Timed out retrying after 10000ms`, with `-'Web form'` (actual) and `+'Web Form'` (expected)
- Step 4.6, tool and calories in the answer of POST /api/pizza: Pizza cutter, 475 (run 1)
- Step 4.7, the line of the admin list that shows your pizza: Lab Pizza 20261123-173912 (tool=Pizza cutter, ingredients_number=7)
- Step 4.8, run 1, result, duration and pizza name: All specs passed, 00:09 (QP-01 00:07), Lab Pizza 20261123-173912
- Step 4.8, run 2, result, duration and pizza name: All specs passed, 00:08 (QP-01 00:07), Lab Pizza 20261123-173926
- Step 4.9 (optional), browser and result: not done

## Part B
1. ช่องตัวเลขทั้งสามช่อง (Max Calories per Slice, Min และ Max Number of Toppings) ใช้ id เดียวกันคือ `floating_filled` เมื่อลอง `$$("#floating_filled").length` ใน Console ได้ผลเป็น 3 จึงใช้ id แยกช่องไม่ได้ เทสต์จึงหาช่องจากข้อความของ label แทน คือ `//label[contains(.,'Max Calories')]/../input` ใน Robot และ `cy.contains('label', 'Max Calories per Slice').siblings('input')` ใน Cypress ถ้า QuickPizza ให้แต่ละช่องมี id ไม่ซ้ำกัน เช่น `max-calories` เทสต์ก็ใช้ `#max-calories` ได้เลย ซึ่งช่วยผู้ทดสอบ เพราะ id ที่ไม่ซ้ำจะไม่เปลี่ยนเมื่อข้อความ label การแปลภาษา หรือ layout เปลี่ยน และเป็น locator ที่สั้นและชัดเจนที่สุด

2. การเช็กนี้เป็น consistency oracle คือเทสต์ไม่รู้ว่า tool ที่ถูกต้องคืออะไร แต่เช็กว่าผลลัพธ์สองที่ของระบบเดียวกันตรงกัน ได้แก่ คำแนะนำในหน้าแรกกับรายการในหน้า admin มันจะจับ fault ได้ในกรณีที่เซิร์ฟเวอร์บันทึกพิซซ่าด้วย tool ที่ต่างจากที่ส่งให้ลูกค้า เช่น หน้าแรกแสดง `Tool: Pizza cutter` แต่หน้า admin แสดง `tool=Knife` ในการรันของผม ทั้งสองที่ตรงกัน เช่น Lab Pizza 20261123-173912 เป็น Pizza cutter ทั้งสองที่

3. หน้าเว็บแสดงชื่อ dough ส่วนผสม tool และแคลอรี่ แต่ไม่ได้บอกว่าเป็นมังสวิรัติหรือไม่ อย่างไรก็ตาม response ของ `POST /api/pizza` มี `"vegetarian": true` ดังนั้นใน Cypress เพิ่ม `expect(response.body.vegetarian).to.be.true` ในบล็อก `cy.get('@pizza')` ได้ ถ้าจะเช็กจากหน้าเว็บเอง เทสต์ต้องมีข้อมูลทดสอบจากแหล่งอื่น คือรายชื่อส่วนผสมที่ไม่ใช่มังสวิรัติ เช่น เนื้อสัตว์และอาหารทะเล แล้วเช็กว่าไม่มีรายการใดใน `#recommendations li` ตรงกับรายชื่อนั้น

4. Robot Framework ต้องเขียนการรอเองทั้งหมด ได้แก่ `Wait Until Element Is Visible css:h1` รอหัวข้อ, `Wait Until Page Contains Element` รอตัวเลือก Knife และรอ `svelte-announcer` ก่อน login หน้า admin ส่วน Cypress รอให้เอง เพราะทุก `cy.get` และ `should` จะลองซ้ำได้นานถึง 10 วินาที เหลือแค่ตอน login admin ที่ยังต้องเช็ก `#svelte-announcer` การเขียน QP-01 ใน Cypress เร็วกว่าเล็กน้อย และ `cy.intercept` ยังช่วยเช็ก request และคำตอบของเซิร์ฟเวอร์ได้ด้วย ด้านรายงาน log.html ของ Robot อ่านง่ายที่สุด เพราะแต่ละ keyword กางดูขั้นตอนย่อยได้พร้อมค่า เช่น `${tool} = Scissors` และ screenshot อยู่ตรง step ที่ถ่าย ส่วน Cypress ให้วิดีโอและ Command Log ที่ดูหน้าเว็บ ณ แต่ละคำสั่งได้ QP-01 ใช้เวลาประมาณ 7 วินาทีในทั้งสองเครื่องมือ

5. ผมจะเลือก Cypress สำหรับ Project 2 เหตุผลแรก Cypress ไม่ต้องใช้ ChromeDriver ขณะที่ Robot ต้องใช้ driver ที่ตรงกับ Chrome 157 พอดีและต้องตั้งตัวแปร CHROMEDRIVER ซึ่งจะใช้ไม่ได้ทุกครั้งที่ Chrome อัปเดต แต่ Cypress รันได้ทันทีหลัง `npm install` เหตุผลที่สอง การลองซ้ำอัตโนมัติทำให้ QP-01 ผ่านทั้งสองรอบโดยไม่ต้องเขียนการรอเอง และ `cy.intercept` กับวิดีโออัตโนมัติให้การเช็กที่แข็งแรงกว่าและหลักฐานที่พร้อมใช้

6. (Part 3) locator ของช่องแคลอรี่อยู่ที่เดียวคือ `robot/resources/quickpizza.resource` ในตัวแปร `${MAX CALORIES}` ถ้า QuickPizza เปลี่ยน locator ผมจะแก้แค่บรรทัดนั้นในส่วน Variables ส่วน `pizza_scenario.robot` เรียกแค่ keyword เช่น `Fill In The Pizza Request    600    Knife    ${name}` และไม่มี locator อยู่เลย ไฟล์เทสต์จึงไม่ต้องแก้ และทุกเทสต์ที่ใช้ resource นี้จะกลับมาทำงานได้พร้อมกัน

7. (Part 4) Cypress ล้าง cookie, local storage และ session storage ก่อนทุกเทสต์ QuickPizza รันอยู่บนหลายเซิร์ฟเวอร์ โดย cookie ทำให้เบราว์เซอร์อยู่กับเซิร์ฟเวอร์เดิม และหน้า admin แสดงเฉพาะพิซซ่าของเซิร์ฟเวอร์นั้น ถ้า step 6 และ 7 เป็นเทสต์ที่สอง มันจะเริ่มโดยไม่มี cookie อาจไปเจอเซิร์ฟเวอร์อื่น และ `cy.contains('li', pizzaName)` จะ time out เพราะพิซซ่าไม่อยู่ในรายการนั้น นอกจากนี้เทสต์ที่สองจะไม่มี `pizzaName` และ alias `@tool` เพราะ alias ถูกรีเซ็ตระหว่างเทสต์
