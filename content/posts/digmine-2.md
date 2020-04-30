---
title: "Malware Diary: DigMine Part2 - Dynamic Analysis ฉบับคนขี้เกียจ"
date: 2019-06-03T02:00:53+07:00
draft: false
categories: ["Malware Analysis : DigMine"]
featured_image: "/images/malware-diary2/1.jpg"
featured_image_caption: "เมื่ออยากรู้ว่ามัลแวร์ทำงานยังไงแต่ Malware Sandbox กลับตรวจไม่พบ ทำไงดีน้าา?"
---


หลังจากที่ Part ที่แล้วเราได้ทำการวิเคราะห์แบบ Static Analysis ไปแล้ว แต่ก็ยังไม่ได้ข้อมูลอะไรเท่าไรนอกจาก Source Code ของมัลแวร์จริง ๆ ถ้า Source Code ไม่ได้มีการ Obfuscate ไว้เมื่อถึงจุดนี้เราควรที่จะเข้าใจการทำงานของมัลแวร์บ้างแล้ว แต่ครั้งนี้ไม่ไง เพราะถึงได้ Source Code มาแต่เราก็อ่านไม่ออกอยู่ดี -___- 

คำถามต่อมา...แล้วไงต่ออ่ะ 
ขั้นตอนต่อไปที่เราจะทำเค้าเรียกว่า Dynamic Analysis แต่เคสนี้คือเราขี้เกียจไง จะให้ไปใช้ Ollydbg หรือพวก Debugger ก็ขี้เกียจ(ความจริงก็คือใช้มั่ยเปนนนน) แต่การที่เราได้ Source Code  เราสามารถเอาไปใช้ต่อได้

จำได้ไหมครับจาก Part ที่แล้ว จาก Source Code เรารู้อะไรบ้าง?
เรารู้ว่ามันมีฟังก์ชั่นที่คาดว่าเป็นฟังก์ชั่นถอดรหัส 

```AutoIt
Func gmfnzmi($gzhkalbyyq)
	$omipkddl = ""
	For $dlcqoqsup = $rgnzkvwianz To kfwjwweupfy($gzhkalbyyq)
		$swfrwp = divznduthlwg($gzhkalbyyq, $dlcqoqsup, furhpb())
		$qkkocudrwel = ejncdqtyn($usurpagyyn, $swfrwp, iewdtkkvw())
		$omipkddl &= divznduthlwg($dxbvhjewu, $qkkocudrwel, $mpbuyk)
	Next
	Return $omipkddl
EndFunc
```

คำถามคือเราจะเอา Source Code เนี้ย ไปใช้ยังไง?
ถ้าถามคำถามนี้กับคนขี้เกียจ ๆ แบบเราก็คือในเมื่อเรารู้ว่าฟังก์ชั่นนี้เป็นฟังก์ชั่นถอดรหัสเราก็แก้มัลแวร์ให้มันแสดงค่าที่ทำการถอดรหัสออกมาสิ...

```AutoIt
Func gmfnzmi($gzhkalbyyq)
	$omipkddl = ""
	ConsoleWrite("Before: " & $gzhkalbyyq & @CRLF)
	For $dlcqoqsup = $rgnzkvwianz To kfwjwweupfy($gzhkalbyyq)
		$swfrwp = divznduthlwg($gzhkalbyyq, $dlcqoqsup, furhpb())
		$qkkocudrwel = ejncdqtyn($usurpagyyn, $swfrwp, iewdtkkvw())
		$omipkddl &= divznduthlwg($dxbvhjewu, $qkkocudrwel, $mpbuyk)
	Next
	ConsoleWrite("After: " & $omipkddl & @CRLF)
	Return $omipkddl
EndFunc
```

จำ VM ? ที่เรา Setting ไว้ใช้จาก Part ที่แล้วได้ป่ะ? 
เราก็ไปดาวน์โหลดคอมไฟล์เลอร์ของภาษา AutoIt มาลงเพื่อรันโค๊ดมัลแวร์ที่เราได้ทำการแก้ไปด้านบน

แต่! อย่าพึ่งรันเลยนะครับ ลง Google Chrome ก่อน เพราะมัลแวร์ตัวนี้มี Mechanism บางอย่างเกี่ยวกับ Google Chrome ถ้าเราอยากเข้าใจการทำงานจำเป็นต้องติดตั้งด้วย(ถามว่ารู้ได้ไง ตอนแรกเราไม่ได้ลงอ่ะ แล้วมันก็หาอะไรไม่เจอ มาเจอทีหลังว่า DigMine มันมี Mechanism บางอย่างกับ Google Chrome ด้วย)

แล้วถ้าลง Google Chrome แล้วรันเลยมั้ย?
ก็ไม่อยู่ดี เพราะถ้าอย่างนั้นเราอาจจะรู้แค่ว่าค่าที่ถูกนำว่าถอดรหัส มีอะไรบ้าง อาจพลาดรายละเอียดบางส่วน 
โดยเฉพาะในส่วนที่เป็น Network เราคิดว่าเราไม่น่าเห็น Flow การทำงานของ Network จาก Source Code หรอกถูกมะ?
ฉนั้นเราต้องใช้สิ่งที่เรียกว่า **[Wireshark](https://www.wireshark.org/)** ครับ

ถ้าจะใช้แค่ Wireshark ในการดักจับ Packet ที่รับส่งในเครื่องเรา แต่แค่นั้นผมคิดว่ายังไม่น่าจะเพียงพอนะ เพราะถ้ามันมีการแก้ไขค่าต่าง ๆ เช่น Registry ในเครื่อง Wireshark เองไม่ได้ดักจับในส่วนนี้ด้วย เราต้องใช้โปรแกรมที่เรียกว่า **[Process Monitor](https://docs.microsoft.com/en-us/sysinternals/downloads/procmon)**  หรือชื่อเล่นคือ ProcMon (จากลูกพี่ Microsoft เองเบย)

ซึ่งเจ้า ProcMon  เนี่ย จริง ๆ แล้วมันคล้าย ๆ กับ Wireshark แต่สิ่งที่มันดักจับเป็น Event ที่เกิดขึ้นบน Windows  
พอติดตั้งเสร็จทั้งสามตัวเนี่ยกดอย่าลืมกด Snapshot ด้วยนะครับ เผื่ออยากกลับมาทำใหม่

เปิด Wireshark กับ ProcMon ขึ้นมา จากนั้นก็กดแคปเชอเลยครับ...

![ทำไมกดแล้วไม่เห็นได้เลยอ่ะ...](/images/malware-diary2/3.jpg)

 อ้าว คนละแคปหรอ เค ๆๆๆๆ
กด Capture ของทั้งสองโปรแกรม จากนั้นก็รัน AutoIt เลยจ้าม... (ผลลัพท์ที่ได้จิ้ม[ตรงนี้]([https://github.com/jusmistic/Malware-Analysis-DigMine/blob/master/Source%20Code/malware_output.txt](https://github.com/jusmistic/Malware-Analysis-DigMine/blob/master/Source Code/malware_output.txt))จ้า)

![ผลลัพท์ของมัลแวร์](/images/malware-diary2/69.png)

แต่แบบนี้ผมว่ายังมีข้อเสียอยู่คือ… **โปรแกรมในส่วนที่ไม่ได้เข้า Control Flow เราจะไม่รู้เลยว่ามันมีการทำงานยังไงบ้าง**

  

แต่เท่าที่ผมสังเกตุดู Flow นีัยคือ Flow การทำงานที่มัลแวร์มันทำการ Infect อ่ะ เลยคิดว่าเราน่าจะได้อะไรบ้างอยู่ เห้ย เกือบลืมเราไม่ได้มีแค่ String ชุดนี้นี่หว่า เรายังมี Pcap ที่เรากดแคปเชอ(ปราง)บน Wireshark กับ Log ของ ProcMon อยู่นี่หว่า.. น่าจะเจออะไรเพิ่ม

  

วิธีที่ผมทำคือ ผมจะเริ่มดูจาก String ที่เราได้มาจากมัลแวร์ถ้าเจออะไรที่น่าสนใจผมจะกลับไปดูที่ pcap กับ Log นะ หลังจากไล่ ๆ อ่าน String ไปเจอบรรทัดที่ 47

![www.google.com 🤔](/images/malware-diary2/4.jpg)

 เป็นโดเมนแฮะ งั้นลองไปดูใน Wireshark ดีกว่า..

  ![Paket เรียกขอ www.google.com จาก DNS Server](/images/malware-diary2/5.jpg)

มีการเรียก DNS ไปที่ Google จริง ๆ แฮะ เลยไล่ดูต่อคือมัน Ping ไปที่ Google คำถามคือ..ทำไม? เลยมานั่งคิดนอนคิด ผมคิดว่ามันน่าจะใช้เป็นเงื่อนไขในการเช็คว่ามันต่ออินเตอร์เน็ตได้ไหม ถ้าเชื่อมต่อได้มันคงทำอะไรซักอย่างนึกที่เราไม่รู้ (อันนี้ผมคาดเดาเอาจากบริบทนะ ยังไงใครว่าง ๆ ลองเอาไปทำแบบไม่ต่ออินเตอร์เน็ตที ได้ความยังไงบอกผมด้วย 5555)

 โอเค เรากลับมาดูที่ String ต่อเผื่อเจออะไรน่าสนใจ

![เหมือนเป็น HTTP Request เบย!](/images/malware-diary2/6.png)

กลับไปดูที่ Wireshark เผื่อมันมีการเรียก HTTP Request จริง ๆ ก่อนอื่นขอตัดมา Network 101 อธิบายเรื่อง DNS ก่อน

![์Network 101](/images/malware-diary2/7.png)  

คือจะเล่าย่อ ๆ DNS คือ Service ที่แปลงจากชื่อเว็บไซต์ไปเป็น  IP ถูกมะ ด้านบน(สีแดง)มันคือ Query ไปขอ fusu[.]icu โดยใช้ Type A คือ Type A เนี่ยเป็นการบอกว่าให้ DNS Server มันตอบมาเป็น IP หน่อย(สีเขียว) มันจะได้เอา IP ไปใช้ต่อ

  **จบแล้วครับอธิบายสั้น ๆ ก็ประมาณนี้ ถ้างงตรงนี้อย่าถามผมนะครับ เพราะผมก็งงเหมือนกัน**

  ถามว่าทำไมต้องอธิบาย? เพราะเราจะเอาไปหา Flow การเชื่อมต่อไปยัง Host นี้ใน Wireshark ไง แล้วตอน Filter มันต้องใช้ IP! ซึ่งในที่นี้คือ 104.27.136.252 และ 104.27.137.252 นะ

  ![มันมี HTTP Request จริง ๆ ด้วยโว้ยยย](/images/malware-diary2/8.png)

​    เอาตรง ๆ ป่ะ ตอนแรกที่ผมเห็น Packet นี้ผมโคตรว้าวเลยอ่ะ คือผมไม่เคยเห็นใครใช้ HEAD Method มาก่อน ถ้าไม่นับ CTF อันนี้น่าจะเป็นครั้งแรก อีกอย่างคือการใช้งานมันโคตร Creative คือปกติเนี่ย HEAD Method เนี่ยมันจะส่งไปได้แต่ส่วนที่เป็น HTTP Header แล้ว Server ก็ตอบมาได้แต่ส่วนที่เป็น Header เหมือนกัน ซึ่งความ Creative มันอยู่ตรงที่คนทำมัลแวร์ตัวนี้เค้าเอาค่าต่าง ๆ ที่ต้องการส่งมันทำเป็น Custom Header แล้วส่งไป แล้ว Server ก็ตอบโดยแปะค่ามาเป็น Custom Header เหมือนกัน เอ้ะ หรือปกติเค้าใช้แบบนี้กันอยู่แล้ว 55555

​    จาก Packet ด้านบนจะเห็นว่ามันส่งรายละเอียดต่าง ๆ ของคอมเรากลับไปที่ Server แล้ว Server ก็ Reply กลับมาโดยมี Custom Header ที่น่าสนใจมาด้วยคือ Unzip กับ Zip เพราะภายในมี Url ที่น่าสนใจอยู่

  เลยมาไล่ดู Flow Packet ต่อ

![HTTP Request Method ที่น่าสนใจ](/images/malware-diary2/9)

จะเห็นว่ามันดาวน์โหลดไฟล์มาทั้งสองไฟล์ที่ url อยู่ในค่า unzip และ zip ด้วย แต่…ดาวน์โหลดไปไว้ที่ไหนล่ะ

![ส่อง Log ของ ProcMon แปป](/images/malware-diary2/10.jpg)

จะเห็นว่ามันทำการดาวน์โหลดไฟล์มาไว้ที่ %appdata%/ ซึ่งถ้าลองเข้าไปดูเราจะเจอทั้ง 7za.exe และ files.7z แต่… มันไม่ได้เจอแค่นั้นอ่ะดิ

![WTF มาจากไหนฟะะ](/images/malware-diary2/10-fix1.png)

เลยลองไล่ดู Wireshark ก็ไม่เห็นแฮะว่ามันไปดาวน์ไฟล์พวกนี้มาตอนไหน เลยลองกลับไปหาใน String ของมัลแวร์ดู

> After: 7za.exe e files.7z -aoa -p(พาสเวิร์ด)

พอเห็นบรรทัดนี้พอเข้าใจละ แต่ยังไม่มันใจเลยลองไปหาใน Google ต่อ… พบว่า 7za.exe คือโปรแกรม 7z แบบ Standalone (ทุกอย่างยัดไว้ใน Binary ตัวเดียว) และทำการแตกไฟล์ files.7z โดยมี -p[password] เราสามารถลองเช็คได้โดยการลองแตกไฟล์ดูโดยใช้ Password นี้

 ![ลองแตกไฟล์โดยใช้ Password ที่ได้มัลแวร์](/images/malware-diary2/16.png)

ลองดูเทียบกับภาพบนจะเห็นว่า มันมีบางไฟล์ที่หายไปนั่นคือไฟล์ app.exe แล้วไฟล์ app.exe มาจากไหน?

เลยกลับไปส่อง String ที่ได้มาจากมัลแวร์…

  ![ขี้เกียจคิดแคปชั่น](/images/malware-diary2/11.png)

เหมือนมันมีการเรียกคำสั่ง Copy ไฟล์แฮะ…คำถามต่อมาก็อปปี้มาจากไฟล์ไหน? คำตอบคือ Copy มาจากไฟล์ Execute ที่เรารันไปครั้งแรกนั่นแหละ(เอา hash ไฟล์มาเทียบดู) แต่มัน Copy มาทำไมฟะะะ? เลยวนกลับไปดูที่ String อีกรอบ…ว่ามีอะไรให้คำตอบเราได้บ้าง

มันมีการแก้ Registry แฮะ ไปเช็ด Log ด้วยจะได้ชัวร์ ๆ

![1559501244370](/images/malware-diary2/9-1.png)

ซึ่งการแก้ไขต่าง ๆ ผมสรุปมาได้ประมาณนี้ครับ

| ค่า Registry ที่มีการเปลี่ยนแปลง                                   | ค่าก่อนเปลี่ยน        | ค่าหลังเปลี่ยน               |
| ------------------------------------------------------------ | ----------------- | ------------------------ |
| HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Associations\LowRiskFileTypes | ไม่มีการกำหนดค่า     | .exe                     |
| HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Associations\LowRiskFileTypes | ไม่มีการกำหนดค่า     | .exe                     |
| HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\EnableLUA | โดยปกติควรมีค่าเป็น 1 | 0                        |
| HKLM\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Run\Google Updater | ไม่มี Registry นี้    | %appdata%/(user)/app.exe |
| HKCU\Software\Unzip\Installed                                | ไม่มี Registry นี้    | Yes                      |

1. Registry ตัวนี้เป็นการกำหนดสกุลไฟล์ที่จะไม่ให้แจ้งเตือนเวลาทำการรัน โดยในที่นี้ตัวมัลแวร์เลือกเป็นนามสกุล .exe อ่านคำอธิบายเพิ่ม[ลิงค์นี้](https://blog.malwarebytes.com/detections/pum-optional-lowriskfiletypes/)เบย
2. กลับไปอ่านข้อหนึ่ง ลองดูว่ามันต่างกันตรงไหนและมันจะส่งผลแตกต่างกันอย่างไร
3. Registry ตัวนี้ถ้าเราเปลี่ยนเป็น 0 (False) มันจะไม่ทำการแจ้งเตือนเวลาที่โปรแกรมทำการเปลี่ยนแปลงกับเครื่องคอมพิวตเอร์ [อ่านเพิ่ม](https://docs.microsoft.com/en-us/windows-hardware/customize/desktop/unattend/microsoft-windows-lua-settings-enablelua)
4. **ข้อนี้ดูเหมือนจะเป็นคำตอบของเราเลยนะฮะ** เพราะจากค่า Value ด้านในมันเป็น Path ที่ไปที่ %appdata%/(user)/app.exe  และการตั้งค่า Registry ตรงนี้คือการสั่งให้ Windows ทำการรันไฟล์นี้เมื่อทำการเปิดเครื่อง ขั้นตอนนี้เป็นขั้นตอนที่มัลแวร์ทำการฝังตัวบมเครื่องนั่นเอง..
5. ตัวนี้ผมไม่ค่อยแน่ใจเท่าไหร เพราลองไล่ดู Flow มันเหมือนตั้งต่า Registry นี้ตอนที่ทำการโหลดไฟล์ 7za.exe เสร็จ เหมือนเป็นสถานะการติดตั้ง อะไรประมาณนี้

 แต่…มันจะรันตัวเองซ้ำไปทำไมวะ 555    ตอนนี้เหมือนเราเห็นแค่ Mechanism ที่มัลแวร์ทำการฝังตัวเอง แต่ยังไม่ค่อยเห็น Mechanism ที่มัลแวร์มันทำอะไรที่ส่งผลกระทบกับเครื่องคอมเลย

จริง ๆ ProcMon มีอีก Feature ที่น่าสนใจคือ Process Tree

![app.exe มีการเรียก update-x64.exe ด้วยแฮะ](/images/malware-diary2/11-1.png)

คือเราสามารถดูได้ว่า Process ไหนมันเรียก Process ไหนคือจาก Process Tree เนี่ย เราจะเห็นว่า app.exe มันมีการเรียกทั้ง chrome.exe(Google Chrome) และ update-x64.exe(Monero Miner) 

ในส่วนของ Google Chrome เนี่ยผมขอข้ามไปก่อนนะ เดี้ยวเรามาดูกัน Part หน้า ซึ่งผมว่าส่วนนี้ก็น่าสนใจไม่แพ้กัน

![](/images/malware-diary2/12.png)

จากชุดข้อความที่ได้จากมัลแวร์เหมือนมีการเช็ค CPU Architecture ก่อนที่จะทำการ Mining เหรียญ Monero ด้วย

ซึ่งนี่น่าจะเป็นเหตุผลที่มัลแวร์มันการแก้ไข Registry ที่จะทำการรันโปรแกรมเมื่อ StartUp ไปที่ %appdata%/[user]/app.exe(ตัวมัลแวร์เอง) แล้วตัวมัลแวร์จึงมารัน Update-x64.exe ต่อเพื่อทำการ Mining Monero ต่อ
โดยใช้ไฟล์ Config ที่อยู่ใน Path เดียวกันนั่นเอง

**ตอนนี้เริ่มขี้เกียจแล้วครับ** Part นี้ผมขอเบรคไว้ที่ตรงนี้ก่อน เดี้ยว Part หน้าเรามาดูกันว่ามัลแวร์ตัวนี้มันทำอะไรกับ Google Chrome ของเหยื่อบ้าง ผมว่าน่าจะสนุกไม่แพ้ Part นี้เลยครับ

ถ้าอ่านมาถึงตรงนี้ก็ขอบคุณทุกคนที่ติดตามมาจนถึง Part 2 และยังอ่านจนจบอีก หากมีอะไรผิดพลาด/อยากติชมอะไรเมนชั่นมาบอกผมใน Twitter เลยก็ได้ครับ พร้อมรับคำติชมไปปรับเสมอครับ 😄

ปล. ต้องขอโทษจริง ๆ ครับที่ Part นี้มาช้ามาก  เพราะติดค่าย [ITCAMP | KMITL](https://www.facebook.com/itcampKMITL/)  ได้ข่าวว่าค่าย Netherine มีเล่น CTF กันด้วย ไม่ไปก็กลัวจะเสียดายทีหลังเลยต้องแว๊ปไปหน่อย 😅

