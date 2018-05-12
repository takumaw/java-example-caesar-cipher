# Java サンプルアプリ: シーザー暗号

このサンプルでは、Javaにてシーザー暗号を取り扱っています。
シーザー暗号については、後述します。


## サンプルのつかいかた

まず、本サンプルを Tomcat にデプロイしたら、以下のURLにアクセスします。

http://localhost:8080/CaesarCipher/ (手元の設定によっては、URLが変わる場合もあります。)

URLにアクセスすると、シーザー暗号アプリのトップページが表示されます。
ページ中には、 Encrypter, Decrypter, Cracker があります。

  * Encrypter
    * メッセージとシフト数を入力し、Encryptを押すと、暗号文が表示されます。
  * Decrypter
    * 暗号文とシフト数を入力し、Decryptを押すと、元のメッセージが表示されます。
  * Cracker
    * 暗号文だけを入力し、Crackを押すと、うまく暗号を破ることができた場合は元のメッセージが表示されます。


## サンプルの内容

### Java クラス (src/main/java フォルダ内)

  * `com.example.caesar.util.CaesarCipher` シーザー暗号を行うクラス
  * `com.example.caesar.util.CaesarCipherCracker` シーザー暗号破りを行うクラス

### Java Servlet クラス (src/main/java フォルダ内)

  * `com.example.caesar.web.EncryptionServlet` シーザー暗号で文章を暗号化するサーブレット
  * `com.example.caesar.web.EncryptionServlet` シーザー暗号化された文章を復号するサーブレット
  * `com.example.caesar.web.CrackServlet` シーザー暗号化された文章の暗号を破るサーブレット

### Java テストクラス (src/test/java フォルダ内)

  * `com.example.caesar.util.CaesarCipherTest` CaesarCipher のテストクラス
  * `com.example.caesar.util.CaesarCipherCrackerTest` CaesarCipherCrackerTest のテストクラス

### JSP ファイル (WebContent フォルダ内)

  * `index.jsp` トップページ
  * `result.jsp` 結果表示ページ (全サーブレットで共用)


## 付録: シーザー暗号とは
### シーザー暗号のしくみ

シーザー暗号は、とても単純な暗号です。

  * 暗号化: 英文を受け取り、指定された数だけ英字をシフトする(ずらす)
    * 例: 'A' を5つシフトすると、 [A] → B → C → D → E → [F] なので、 'F' となる。
  * 復号化: 暗号文を受け取り、指定された数だけ英字を逆にシフトする
    * 例: 'F' を5つ逆にシフトすると、[A] ← B ← C ← D ← E ← [F] なので、 'A' となる。

例:

  * 元の文章: "Hello, world!"
  * シフト数: 10
  * 暗号化された文章: "Rovvy, gybvn!"

シーザー暗号では、シフト数が『パスワード』となります。

今回のサンプルでは、A-Z, a-z 以外の文字は無視して動作するように作ってあります。

### 暗号の破り方

シーザー暗号を破るには、どうすれば良いのでしょうか？
実は、ある条件下では、シーザー暗号を破る方法が存在します。

前提条件1: 元のメッセージは、英語(などの自然言語)であること。
前提条件2: 元のメッセージは、ある程度長いこと。

英語で書かれた文章中のアルファベットは、おおまかな出現頻度(Letter frequency)がわかっています。
具体的には、以下のようになっています。

  * 'E': 12.702%	
  * 'A': 8.167%	
  * 'O': 7.507%	
  * ...(中略)...
  * 'X': 0.150%	
  * 'Q': 0.095%	
  * 'Z': 0.074%	

なので、暗号文中のアルファベットの出現頻度を分析(Frequency analysis)して、
シフトをi=0, 1, ..., 26とずらしていって、いちばん出現頻度パターンが一致するシフト数を求めることで、
シフト数を推定し、暗号文を破ることができます。


### 暗号の破り方 (より具体的に)

はじめに、以下のベクトルを定義しておきます。

V = (v0, v1, v2, ..., v26)
ここで、 vi = i番目の文字出現頻度 (例: v0 = 'A'の出現頻度, )

英語の場合、以下のようになります。
V = (0.08167, 0.01492, 0.02782, ..., 0.00074)


つぎに、暗号文を受け取ったら、その暗号文中の各文字の出現頻度を計算し、以下のベクトルを計算します。

Vt = (vt0, vt1, ..., vt26)

このとき、Vtは、Vを「シフト」したものとなっているはずです。
例えば、暗号文のシフト数が5だった場合、5つ逆方向に「シフト」したベクトル

Vt5 = (vt5, vt6, vt7, ..., vt25, vt26, vt0, vt1, vt2, vt3, vt4)

は、

Vt5 ≃ V

となっているはずです。
ですから、シフト数は、いちばんVと近くなる (Vti ≃ V となる) シフト数 i を計算することで、求められます。

具体的な計算方法には、内積を使います。

Vti (i = 0, 1, ..., 26) は、V と(ほぼ)大きさが同じで、向きだけが異なったベクトルとなります。
(26次元空間の中で、それぞれ違った向きを向いている。)

V・Vti = ||V|| ||Vti|| cosθ = v0 * vti0 + v1 * v (ここで、θは V と Vti のあいだの角度を指す)

ここで ||V|| (Vの長さ), ||Vti|| (Vtiの長さ) は変わらないので、
変わるのはθとなります。

VとVtiが一番近いときにθが最小になる、つまり cosθ が最大となります。

ですから、シフト数の求め方は、

V・Vti が最大となる i 、つまり

i = argmax[0 ≤ i ≤ 26] V . Vti

となります。


## 参照文献

  * シーザー暗号 - Wikipedia : https://ja.wikipedia.org/wiki/%E3%82%B7%E3%83%BC%E3%82%B6%E3%83%BC%E6%9A%97%E5%8F%B7
