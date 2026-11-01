# RSA & SHA-256

## 概要

基本の非対称鍵暗号であるRSA暗号と、ハッシュ値を求めるSHA-256を実装しました。

RSAは本来共通鍵暗号の鍵の送受信に使われ文字列には使われないようですが、今回は文字列の暗号化、復号化に使いました。

### RSA

桁数が大きい合成数の素因数分解が現実的な時間内で困難だという事を安全性の根拠に利用している公開鍵暗号の一つです。

任意の文字列を、任意の二つの素数から求められた公開鍵で暗号化し、秘密鍵で復号化します。

```mermaid
flowchart TD
    subgraph KEY["鍵生成 generateKey"]
        direction TB
        P["素数 p, q を用意"] --> N["N = p × q"]
        P --> L["L = lcm(p-1, q-1)"]
        L --> E["E = 65537<br/>gcd(E, L) ≠ 1 なら 3 から探す"]
        E --> D["D = E の逆元 mod L"]
        N --> PUB["公開鍵 (E, N)"]
        E --> PUB
        D --> PRI["秘密鍵 (D, N)"]
        N --> PRI
    end

    subgraph ENC["暗号化 encrypt"]
        direction TB
        T["平文の文字列"] --> B["UTF-8 のバイト列に変換"]
        B --> M["整数 m に変換"]
        M --> CHK{"m ≥ N ?"}
        CHK -- "はい" --> ERR["エラー: もっと大きな素数が必要"]
        CHK -- "いいえ" --> C["c = m^E mod N"]
    end

    subgraph DEC["復号 decrypt"]
        direction TB
        C2["暗号文 c"] --> M2["m = c^D mod N"]
        M2 --> B2["バイト列に変換"]
        B2 --> T2["UTF-8 でデコード → 元の文字列"]
    end

    PUB -. "使う" .-> C
    C --> C2
    PRI -. "使う" .-> M2
```

![RSAを実行している様子](assets/rsa.png)

### SHA-256

アメリカ国家安全保障局によって設計されました。

作成したハッシュ関数とライブラリのハッシュ関数の出力を比較し、正しい値が出力されているかを確認します。

```mermaid
flowchart TD
    IN["入力メッセージ bytes"] --> PAD["padding<br/>0x80 を付加 → 0 埋め → 末尾に長さ 64bit"]
    PAD --> SPLIT["64バイトごとのブロックに分割"]
    H0["初期ハッシュ値 H0〜H7"] --> LOOP

    SPLIT --> LOOP

    subgraph LOOP["各ブロックに対して繰り返す process_block"]
        direction TB
        S["build_scedule<br/>16語を 64語 W0〜W63 に拡張"] --> R["step_round を 64回<br/>作業変数 a〜h を更新<br/>定数 K と W を使用"]
        R --> ADD["元の H に a〜h を加算<br/>mask32 で 32bit に収める"]
    end

    LOOP --> NEXT{"次のブロックがある?"}
    NEXT -- "はい" --> LOOP
    NEXT -- "いいえ" --> OUT["H0〜H7 を連結<br/>64桁の16進数 = ハッシュ値"]
```

![SHA-256を実行している様子](assets/sha256.png)

## 制作者のコメント

pythonには符号無し32bit整数が存在しない為、オーバーフローの処理を用意しないといけないので、書いていて大変でした。ビット単位で行なう処理はpythonで行なう物ではないと感じました。

rsa暗号は必要となる公開鍵が大きくなってしまったため、大きい素数を探すのに少し苦労しました。

関数は出来る限り副作用が発生しないように作成し、型は明示する様にしました。型は無いより有る方が書きやすく、読み易いと感じました。今度はhaskellなどで実装してみたいと思います。

どのように動いているか知りたい時は、自分で実装するのが簡単で身に付く方法だと思います。次は楕円曲線暗号を実装したいです。

## LICENSE

Released under the [MIT LICENSE](LICENSE)
