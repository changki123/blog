+++
title = "Active Directory Always On Failover"
date = 2026-03-01
[taxonomies]
tags = ["AD", "Widnows", "AWS"]
+++

## TL;DR
<img width="780" height="555" alt="Image" src="https://github.com/user-attachments/assets/5bfbe04d-4fa5-4bf5-87ab-6c8cc4a31e89" />

<!-- more -->

<img width="781" height="556" alt="Image" src="https://github.com/user-attachments/assets/e7eeb343-c9d1-4563-95b6-594bae612d4d" />
<img width="782" height="556" alt="Image" src="https://github.com/user-attachments/assets/9eddc868-f357-4dc9-b583-412ec36ee42a" />
<img width="781" height="555" alt="Image" src="https://github.com/user-attachments/assets/b2ab30fa-0446-48ba-a834-34f49c3ea413" />
<img width="779" height="555" alt="Image" src="https://github.com/user-attachments/assets/98fa86a2-ad85-454c-9e59-57efe4e5c9f7" />
<img width="781" height="552" alt="Image" src="https://github.com/user-attachments/assets/985fd175-4a41-426b-a0da-bbd7e35a680d" />
<img width="781" height="557" alt="Image" src="https://github.com/user-attachments/assets/acfe1234-6fc8-40bf-ac99-eba94ac8f48c" />
<img width="780" height="557" alt="Image" src="https://github.com/user-attachments/assets/2cca8e18-d50f-41ea-b131-9e7315c98424" />
<img width="781" height="556" alt="Image" src="https://github.com/user-attachments/assets/c4b61224-a957-4301-b9a8-d316d56db7ff" />
<img width="414" height="339" alt="Image" src="https://github.com/user-attachments/assets/5319ce2d-a7bd-40ba-a4a1-20a608bef641" />
<img width="757" height="556" alt="Image" src="https://github.com/user-attachments/assets/ebdd1bb6-a3aa-4165-b5e2-48e447dbff3d" />
<img width="757" height="550" alt="Image" src="https://github.com/user-attachments/assets/91d244b0-d145-463c-8700-bd9bfa42f278" />
<img width="754" height="552" alt="Image" src="https://github.com/user-attachments/assets/aa4fd3d2-020e-4e58-9690-c176e54bab1e" />
<img width="757" height="556" alt="Image" src="https://github.com/user-attachments/assets/56f3437a-0c36-4982-84d4-5ac7d51f88d2" />
<img width="755" height="554" alt="Image" src="https://github.com/user-attachments/assets/fc6d5033-1353-4451-9063-49bc9f9f2394" />
<img width="757" height="555" alt="Image" src="https://github.com/user-attachments/assets/a4c058f5-d0c1-4e42-b35a-967224913820" />
<img width="755" height="554" alt="Image" src="https://github.com/user-attachments/assets/ce2fe73f-506e-4922-84cf-5656a9dae363" />
<img width="674" height="155" alt="Image" src="https://github.com/user-attachments/assets/795125b4-4f6d-45b1-b8c6-101651d25c5f" />
<img width="316" height="189" alt="Image" src="https://github.com/user-attachments/assets/6df31a73-dc1b-4a73-b573-8b4c69bb6cec" />
<img width="438" height="564" alt="Image" src="https://github.com/user-attachments/assets/b796e9d0-7b36-4453-816a-60d8a0cb3def" />
<img width="507" height="740" alt="Image" src="https://github.com/user-attachments/assets/202b9df5-61d6-498a-94b9-7a2abd14abbb" />
<img width="763" height="573" alt="Image" src="https://github.com/user-attachments/assets/d753720d-2452-4d6a-b35f-8375e929d721" />
<img width="433" height="373" alt="Image" src="https://github.com/user-attachments/assets/68659c28-1177-4b5e-a007-5e19ac6dc324" />
<img width="433" height="375" alt="Image" src="https://github.com/user-attachments/assets/eefbe586-a991-4c97-b511-12153c31c208" />
<img width="436" height="376" alt="Image" src="https://github.com/user-attachments/assets/004a328e-ebdf-49c2-9e17-8b7b05c956ee" />
<img width="300" height="295" alt="Image" src="https://github.com/user-attachments/assets/ec36b323-df9a-4215-be27-0b54804353cb" />
<img width="403" height="535" alt="Image" src="https://github.com/user-attachments/assets/a6da3ee6-f57a-4ebd-8144-e69e93326d19" />
<img width="455" height="250" alt="Image" src="https://github.com/user-attachments/assets/877e3cc7-81fe-4623-8c86-616f18fd4c8c" />
<img width="508" height="570" alt="Image" src="https://github.com/user-attachments/assets/e05de936-4475-49f1-9541-234ff0c9912f" />
<img width="454" height="246" alt="Image" src="https://github.com/user-attachments/assets/79b5433e-2f39-433b-a899-676dce58aca9" />
<img width="410" height="536" alt="Image" src="https://github.com/user-attachments/assets/3e5c9d59-a803-4a75-9cae-71887ad83faa" />
<img width="1340" height="562" alt="Image" src="https://github.com/user-attachments/assets/c825b179-b48f-4880-80c5-93c1c0ad44fc" />
<img width="945" height="467" alt="Image" src="https://github.com/user-attachments/assets/635aae6b-b4e9-40e4-bf9e-ba8799806c91" />
<img width="451" height="300" alt="Image" src="https://github.com/user-attachments/assets/384b9158-9611-4d97-b5cc-157135e28904" />
<img width="260" height="146" alt="Image" src="https://github.com/user-attachments/assets/6474b339-a17a-47b2-945d-68e5cb179fc1" />
<img width="350" height="180" alt="Image" src="https://github.com/user-attachments/assets/ce9ed03f-a66f-4675-9372-0086df35965c" />
<img width="347" height="166" alt="Image" src="https://github.com/user-attachments/assets/edfbb367-4706-46e1-9c2c-9f4a6aae1682" />
<img width="783" height="558" alt="Image" src="https://github.com/user-attachments/assets/22362006-e6e0-4c09-b4af-1f19812cf543" />
<img width="780" height="551" alt="Image" src="https://github.com/user-attachments/assets/5207fe94-e1f9-4ec3-b3a6-e522baf01c5a" />
<img width="783" height="556" alt="Image" src="https://github.com/user-attachments/assets/7f439dcf-1e06-40d3-83c4-f89128bea9de" />
<img width="781" height="555" alt="Image" src="https://github.com/user-attachments/assets/d070572b-fdd3-497b-99ad-e9b99430898a" />
<img width="778" height="554" alt="Image" src="https://github.com/user-attachments/assets/c6431484-def7-4196-b591-c2cf0c860a7b" />
<img width="779" height="555" alt="Image" src="https://github.com/user-attachments/assets/a5606706-562c-4541-a494-4017250b6060" />
<img width="491" height="598" alt="Image" src="https://github.com/user-attachments/assets/2e45bc3e-a6bf-402f-8c10-3a454e9b0d74" />
<img width="1042" height="804" alt="Image" src="https://github.com/user-attachments/assets/66e95d84-b3e5-4aab-9fed-d893bf01075b" />
<img width="662" height="450" alt="Image" src="https://github.com/user-attachments/assets/8897a955-2782-478d-a09c-b2ea0c9671be" />
<img width="663" height="447" alt="Image" src="https://github.com/user-attachments/assets/12915041-9ba1-44c5-b229-2ebd352ce119" />
<img width="456" height="247" alt="Image" src="https://github.com/user-attachments/assets/329e408e-cf16-4dfc-bf81-3ff0e98d9a21" />
<img width="510" height="570" alt="Image" src="https://github.com/user-attachments/assets/0c96ff3b-b08d-4302-975e-1fcf24c2cd7a" />
<!-- Failed to upload "image 49.png" -->
<img width="663" height="447" alt="Image" src="https://github.com/user-attachments/assets/ba532e63-97ee-4fe4-a850-2feea480cc35" />
<img width="662" height="457" alt="Image" src="https://github.com/user-attachments/assets/5baba11e-1714-483d-99b9-a02aa9395866" />
<img width="660" height="458" alt="Image" src="https://github.com/user-attachments/assets/22bf524b-0551-42c3-a002-b1c09cf9eab7" />
<img width="666" height="458" alt="Image" src="https://github.com/user-attachments/assets/60559238-bf0f-4ded-8194-584dca654206" />
<img width="664" height="457" alt="Image" src="https://github.com/user-attachments/assets/76d62bb1-e9f8-4b42-ae52-514fbcaf339c" />
<img width="660" height="457" alt="Image" src="https://github.com/user-attachments/assets/4235e71f-0687-4e79-b6a2-7edb2266f631" />
<img width="662" height="451" alt="Image" src="https://github.com/user-attachments/assets/1466a993-89ec-42f5-ab42-eb0956fd50a8" />
<img width="666" height="452" alt="Image" src="https://github.com/user-attachments/assets/c3598b34-507e-4ffd-8870-e6b66e5416c4" />
<img width="662" height="449" alt="Image" src="https://github.com/user-attachments/assets/ac2ad9a7-78db-422f-8ccf-8c4eb3c9e0b5" />
<img width="1039" height="801" alt="Image" src="https://github.com/user-attachments/assets/55d29f0d-da4b-44d5-9af0-f43075b07a03" />
<img width="1040" height="861" alt="Image" src="https://github.com/user-attachments/assets/870ad327-eeec-435d-97dd-abd7743f6a7d" />
<img width="394" height="485" alt="Image" src="https://github.com/user-attachments/assets/b1cdd01e-dc2b-4020-b61f-33ef53acd3f5" />
<img width="396" height="482" alt="Image" src="https://github.com/user-attachments/assets/ec35322f-d3ad-4ddb-a5f2-cdcf20b1c9d0" />
<img width="396" height="484" alt="Image" src="https://github.com/user-attachments/assets/04b4e820-919a-45e4-afd1-3dec24a61d7d" />
<img width="456" height="273" alt="Image" src="https://github.com/user-attachments/assets/6d545966-e02e-44a4-9ef3-0a287ae2090e" />
<img width="435" height="95" alt="Image" src="https://github.com/user-attachments/assets/7d79576c-5d22-4136-83dd-4593b86d703e" />
<img width="1038" height="802" alt="Image" src="https://github.com/user-attachments/assets/c50e3966-3ad8-4c2d-b953-6c5441744822" />
<img width="942" height="435" alt="Image" src="https://github.com/user-attachments/assets/0cd3b41b-49f4-44bf-bb9d-30d57a1af8a6" />
<!-- Failed to upload "image 68.png" -->
<!-- Failed to upload "image 69.png" -->
<img width="453" height="246" alt="Image" src="https://github.com/user-attachments/assets/c53ba045-a28a-4b3d-a9ba-4df936be5e5b" />
<!-- Failed to upload "image 71.png" -->
<img width="324" height="275" alt="Image" src="https://github.com/user-attachments/assets/ccf76c07-82a9-4240-8325-1043ca2b7be2" />
<img width="479" height="585" alt="Image" src="https://github.com/user-attachments/assets/0298513d-4b04-4423-8678-c705c2c22df1" />
<!-- Failed to upload "image 74.png" -->
<img width="735" height="832" alt="Image" src="https://github.com/user-attachments/assets/e45c4f13-c817-468e-9580-ec3a43d27a76" />
<!-- Failed to upload "image 76.png" -->
<!-- Failed to upload "image 77.png" -->
<!-- Failed to upload "image 78.png" -->
<img width="369" height="356" alt="Image" src="https://github.com/user-attachments/assets/49b695ff-7125-4292-9c7a-8452cea0240c" />
<!-- Failed to upload "image 80.png" -->
<!-- Failed to upload "image 81.png" -->
<!-- Failed to upload "image 82.png" -->
<!-- Failed to upload "image 83.png" -->
<!-- Failed to upload "image 84.png" -->
<!-- Failed to upload "image 85.png" -->
<!-- Failed to upload "image 86.png" -->
<!-- Failed to upload "image 87.png" -->
<!-- Failed to upload "image 88.png" -->
<!-- Failed to upload "image 89.png" -->
<!-- Failed to upload "image 90.png" -->
<!-- Failed to upload "image 91.png" -->
<!-- Failed to upload "image 92.png" -->
<!-- Failed to upload "image 93.png" -->
<!-- Failed to upload "image 94.png" -->
<!-- Failed to upload "image 95.png" -->
<!-- Failed to upload "image 96.png" -->
<!-- Failed to upload "image 97.png" -->
<!-- Failed to upload "image 98.png" -->
<!-- Failed to upload "image 99.png" -->
<img width="816" height="746" alt="Image" src="https://github.com/user-attachments/assets/72936092-c550-4ea5-aa29-00876f74090d" />
<img width="818" height="745" alt="Image" src="https://github.com/user-attachments/assets/ac31993f-52f6-43a4-b104-2a89332de83c" />
<img width="819" height="745" alt="Image" src="https://github.com/user-attachments/assets/32169884-96c8-4dc4-a849-106e85738ca5" />
<img width="819" height="744" alt="Image" src="https://github.com/user-attachments/assets/a8efe544-6944-4083-8bcf-225c6972afa4" />
<img width="817" height="743" alt="Image" src="https://github.com/user-attachments/assets/dbb6d5cb-1d72-4333-9b37-3bf6469bf14c" />
<img width="818" height="743" alt="Image" src="https://github.com/user-attachments/assets/00fb62f9-8c45-4a3a-a418-c37cd4e01ba8" />
<img width="817" height="744" alt="Image" src="https://github.com/user-attachments/assets/d201d80f-3607-4fe2-aff0-41b656e32007" />
<img width="1197" height="514" alt="Image" src="https://github.com/user-attachments/assets/65f8cd64-b6d1-4176-8810-0a5914f001a7" />
<img width="819" height="743" alt="Image" src="https://github.com/user-attachments/assets/591dd809-6641-4dff-9026-ac989d20caa3" />
<img width="814" height="743" alt="Image" src="https://github.com/user-attachments/assets/6759a3be-73ee-4392-b854-46ed928ca605" />
<img width="817" height="746" alt="Image" src="https://github.com/user-attachments/assets/7727e845-5ff4-43f0-bc1a-9ca74c740e42" />
<img width="815" height="744" alt="Image" src="https://github.com/user-attachments/assets/ed14b67e-c8b5-475a-bf49-a742a95d7eb4" />
<img width="331" height="272" alt="Image" src="https://github.com/user-attachments/assets/09af33cc-39a4-42de-89bb-339371c49021" />
<img width="845" height="599" alt="Image" src="https://github.com/user-attachments/assets/644abe4a-2064-498e-985e-c2cbbc597213" />
<img width="631" height="325" alt="Image" src="https://github.com/user-attachments/assets/dd81a257-f06e-4443-a3e4-48490de19716" />
<img width="756" height="614" alt="Image" src="https://github.com/user-attachments/assets/69cd9a22-6a59-4ffd-ac0c-7421ad8ec4b3" />
<img width="580" height="531" alt="Image" src="https://github.com/user-attachments/assets/4217e5c8-b7b5-40b3-9247-a1d4efd33607" />
<img width="400" height="346" alt="Image" src="https://github.com/user-attachments/assets/c87a1bbf-b003-4264-b38b-35379a655f60" />
<img width="969" height="298" alt="Image" src="https://github.com/user-attachments/assets/6aa13f80-1c07-4111-9056-232827b55361" />
<img width="1075" height="365" alt="Image" src="https://github.com/user-attachments/assets/f53d3ca3-f75f-4888-9325-44cba1ea6aa1" />
<img width="1058" height="360" alt="Image" src="https://github.com/user-attachments/assets/9d639047-3fce-48a1-adc6-760dee12a05d" />
<img width="1613" height="251" alt="Image" src="https://github.com/user-attachments/assets/bd10f912-5e11-4986-90ac-df7599c9e1f3" />


## 123
