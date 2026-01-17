---
marp: true
theme: haruyuki-16278
paginate: true
image: https://haruyuki-16278.github.io/slides/2026-01-17-jaws-ug-fukui-reboot-01/index.png
title: aws cli と soracom cli を Deno でうまくくっつけてIoT開発を楽にしたかった話
description: 
keywords: aws cli, soracom cli, Deno, AWS IoT
---

<!-- 
プロット

IoT開発の話でAWS IoTを活用したかった
→ 結局REST APIにしちゃったよね 😇 という話
→ 普段はAPI Gateway + Lambda + DynamoDBをよく使ってる

1. 作りたかったもの 🏢
   1. wifiがない環境でもデータをAWSに送りたい
2. AWS IoT との出会い 🤝
   1. 遡ることn年前、AWSコンソールでIoTの文字を見た
   2. 軽くAWS IoTの説明
3. M5Stack CAT-M/NB-IoT+GNSS ユニット ❤️ SORACOM Air
   1. この組み合わせ、行けるらしい
   2. 買っちった
4. SIM7080G 💔 AWS IoT ...?
   1. 動かしてみた (m5 wifi AWSIoT)
      1. Thing以外は基本terraformで作る
      2. ThingはDenoで登録しやすく
      3. mqtt動く！
   2. 動かしてみた (m5 LTE 適当なwebサイト)
      1. soracom cliで接続の確認
      2. HTTPリクエストできた！
   3. 動かしてみた (m5 LTE AWS IoT)
      1. だめで草
      2. ATコマンドで証明書をCAT-M/NB-IoTに入れるらしい
      3. 😇
5. ---------- ㊙️ ----------
   1. CAT-M/NB-IoT ❤️‍🩹 API Gateway
6. まとめ
   1. 諦めも肝心だよね
   2. とはいえ IoT Core 使えますって言いたいので誰か助けてください

 -->

<!-- _class: lead -->

# ==aws cli=={.aws-color} と ==soracom cli=={.soracom-color} を ==Deno=={.black} で<br/>うまくくっつけて<br/>IoT開発を楽にしたかった話

2026年1月17日
はるゆき (@haruyuki_16278)

<!-- _paginate: false -->

---

![bg right:27% vertical contain](images/icon.jpg)
![bg right:27% vertical contain](images/twitter.png)

## 自己紹介

- **はるゆき** ==（山地 駿徹）=={.text-sm}
  - 香川県出身 → 就職で福井へ
  - 社会人5年目 **エンジニア**
  - 3年くらいWeb → 何でも屋
- **主な活動**
  - NTふくい (旧NT鯖江)
  - 技術書典

---

## 話の流れ

1. 作りたかったもの 🏢
2. AWS IoT との出会い 🤝
3. M5Stack CAT-M/NB-IoT+GNSS ユニット ❤️ SORACOM Air
4. SIM7080G 💔 AWS IoT ...?
5. ---------- ㊙️ ----------
6. まとめ

---

## 話の流れ

1. **作りたかったもの 🏢**{.text-lg}
2. AWS IoT との出会い 🤝
3. M5Stack CAT-M/NB-IoT+GNSS ユニット ❤️ SORACOM Air
4. SIM7080G 💔 AWS IoT ...?
5. ---------- ㊙️ ----------
6. まとめ

---

<!-- header: 1. 作りたかったもの 🏢 -->

## 作りたかったもの 🏢

==マイコンで<br/>センシングしたくて...=={.text-xl5}

---

## 作りたかったもの 🏢

==取ったデータを<br/> ==AWS=={.aws-color} で使いたくて...=={.text-xl5}

---

## 作りたかったもの 🏢

==WiFiがない環境にも<br/>デバイスを置きたくて...=={.text-xl5}

---

:::_ {.text-xl5 .center .pt-1}
IoT やろう
:::

---

## 話の流れ

1. 作りたかったもの 🏢
2. **AWS IoT との出会い 🤝**{.text-lg}
3. M5Stack CAT-M/NB-IoT+GNSS ユニット ❤️ SORACOM Air
4. SIM7080G 💔 AWS IoT ...?
5. ---------- ㊙️ ----------
6. まとめ

---

<!-- header: 2. AWS IoT との出会い 🤝 -->

## AWS IoT との出会い 🤝

遡ること n年前...

---

## AWS IoT との出会い 🤝

遡ること n年前...

![w:800px](./images/aws-console-iot.png)

---

## AWS IoT との出会い 🤝

:::c

![w:300px](./images/icon.jpg)

< ==IoT系のサービスもあるんやなぁ...🤔=={.text-xl2}
:::

---

![bg right contain](./images/about-aws-iot-by-q.png)

## ![](./images/Arch-Category_Internet-of-Things_64.png) AWS IoT

IoTデバイスを**安全に**AWSと通信させたり、デバイスの管理や監視を行えたりする仲間

:::_ {.tip .mt-1}
AWS IoT系サービスは  
大阪リージョンでは  
(多分まだ)使えない
:::

---

### ![](./images/Arch_AWS-IoT-Core_64.png) AWS IoT Core

> IoTデバイスを**安全に**AWSと通信

↑この部分を担うサービス

- デバイスとAWS間で相互認証して安全に通信
- MQTT等で通信→必要に応じてほかサービスへ連携

---

### こんな感じ

:::_ {.center .pt-1}
![w:1000px](./images/aws-iot-architecture.png)
:::

---

<!-- header: 3. M5Stack CAT-M/NB-IoT+GNSS ユニット ❤️ SORACOM Air -->

## M5Stack CAT-M/NB-IoT+GNSS ユニット ❤️ SORACOM Air

<!-- TODO: この組み合わせ、行けるらしい -->

---

### SORACOM

> IoTデバイス向けの「通信」と「クラウド」を融合させ、誰でも手軽かつセキュアにIoTシステムを構築・運用できるグローバルなプラットフォーム
>
> by ==Gemini=={.blue}

---

### 使うとこんな感じ

M5StickC -(==**LTE**=={.red}/mqtt)-> AWS IoT Gateway -(rules)-> DynamoDB

---

### 買っちった 🛒

<!-- TODO: 購入した製品の写真や説明 -->

---

<!-- header: 4. SIM7080G 💔 AWS IoT ...? -->

## SIM7080G 💔 AWS IoT ...?

いざ動かしてみると...

---

### 動かしてみた① (M5 + WiFi + AWS IoT)

<!-- TODO: Thing以外は基本terraformで作る -->

---

### ThingはDenoで登録しやすく

## 実はArduinoにもCLIってあって...

M5StickCの開発にはArduinoを利用

ただ...

AWSで証明書を発行 → ソースコードにコピペ → 書き込み
↑面倒

↓

SORACOMのSIM IDを使ってAWS IoTのデバイス管理をやれば楽なのでは...?

---

Arduino CLI 

---

Deno + Dax で 3つのCLIをいい感じに組み合わせよう！

---

### mqtt動く！ ✅

<!-- TODO: WiFi経由でMQTT通信が成功した様子 -->

---

### 動かしてみた② (M5 + LTE + 適当なWebサイト)

<!-- TODO: soracom cliで接続の確認 -->

---

### HTTPリクエストできた！ ✅

<!-- TODO: LTE経由でHTTPリクエストが成功した様子 -->

---

### 動かしてみた③ (M5 + LTE + AWS IoT)

❌ **だめで草**

---

### 何がダメだったのか

ATコマンドで証明書をCAT-M/NB-IoTモジュール(SIM7080G)に入れる必要があった

😇😇😇

---

<!-- header: 5. ㊙️ -->

## ---------- ㊙️ ----------

### CAT-M/NB-IoT ❤️‍🩹 API Gateway

<!-- TODO: AWS IoT Core を諦めて、API Gateway 経由に変更した話 -->

---

<!-- header: 6. まとめ -->

## まとめ

- **諦めも肝心だよね** 😅
  <!-- TODO: AWS IoT Core は素晴らしいけど、ハードウェアの制約もある -->

- **とはいえ...**
  - IoT Core 使えますって言いたいので誰か助けてください 🙏

---

<!-- header: "" -->
<!-- _class: lead -->

## ご清聴ありがとうございました！

質問・助言お待ちしています 🙇
