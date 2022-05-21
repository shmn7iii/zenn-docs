---
title: "はじめてのTapyrus"
emoji: "📑"
type: "tech"
topics: ["Tapyrus"]
published: false
---

# はじめての Tapyrus

初めて Tapryus を学ぶ方へ向けた、「Tapyrus とはなんたるか」「Tapyrus は何がいいのか」を解説する記事になります。記事中ではブロックチェーン、特に Bitcoin の知識を前提としています。基礎的な内容については触れておりませんのでご注意ください。

記事の内容は [tapyrus-core/doc/tapyrus/](https://github.com/chaintope/tapyrus-core/tree/master/doc/tapyrus) 配下のドキュメント、特に [getting_started.md](https://github.com/chaintope/tapyrus-core/blob/master/doc/tapyrus/getting_started.md) と [technical_overview.md](https://github.com/chaintope/tapyrus-core/blob/master/doc/tapyrus/Tapyrus_Technical_Overview_ja.pdf) を参考にしております。より正確かつ最新の情報は[公式ソース](https://www.chaintope.com/chaintope-blockchain-protocol/)を参照してください。

# Tapyrus とは

任意のビジネスケース毎に独立したネットワークを構成・運用することができるブロックチェーン。各ネットワークは管理主体となるフェデレーション（連邦）をもち、ジェネシスブロック・集約公開鍵・ネットワーク ID で識別される。また、一つのネットワークは「**Signer Network**」と「**Tapyrus Network**」の二つの層（[後述](http://localhost:3000/hello-tapyrus#%E3%83%95%E3%82%A1%E3%82%A4%E3%83%8A%E3%83%AA%E3%83%86%E3%82%A3%E3%81%AE%E6%AC%A0%E5%A6%82)）で構成される。

## ブロックチェーンの分類

※公式の見解ではなく、個人の解釈であることに注意してください。

### NIST による分類

**「[NISTIR 8202](https://nvlpubs.nist.gov/nistpubs/ir/2018/NIST.IR.8202.pdf)」**
ブロックの発行は Signer Network にのみ許されるため、「許可型ブロックチェーン」に分類される。

### Vitalik による分類

**「[On Public and Private Blockchains](https://blog.ethereum.org/2015/08/07/on-public-and-private-blockchains/) 」**
同様の理由により、「コンソーシアムブロックチェーン」に分類される。中でもブロックのルートハッシュと API を公開しており、一定の参照が可能であることから「ハイブリッド」な手法であると言える。
また、Tapyrus Network が完全にオープンである点、つまり参照権限がパブリックであることに留意することが必要であり、「不完全分権型（partially decentralized）」であると言える。

### chaintope での表現

chaintope のサイトおよびレポジトリではしばしば「ハイブリッド型ブロックチェーン」という表現がされる。「単にプライベートとパブリックな特徴を併せ持つチェーン」をハイブリッドブロックチェーンと呼び、中でも「複数の利害関係者からなるコンソーシアム、ないしフェデレーションにより運営されるもの」をコンソーシアムチェーンと呼ぶ事例をを見かけることがあるが、筆者には明確で一般的な定義は確認できなかった。

### コンセンサスアルゴリズム

Bitcoin における PoW では後述の問題があるとし、Tapyrus では PoA に近い、フェデレーションによる検証可能な閾値署名方式を用いたコンセンサスをとっている。公式では明確に「PoA」と行った表現はしていない。

# 既存チェーンの課題

Tapyrus の特徴を説明するために既存チェーンにおける課題点をおさらいする。

## ファイナリティの欠如

一般に、ファイナリティとは「現金通貨の支払いにより債権債務問題が解決され、決済が完了する」といった、法的性質をもった「完了性」として使われる。

> “**ファイナリティ**（settlement finality）”とは、一般に「決済が無条件かつ取消不能となり、最終的に完了した状態」と定義され、我が国においては「決済完了性」と呼ばれる。 - **[金融庁](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjIiom1h4v3AhU7w4sBHZHPA_cQFnoECAsQAw&url=https%3A%2F%2Fwww.fsa.go.jp%2Ffrtc%2Fnenpou%2F2006a%2F11.pdf&usg=AOvVaw3onP5VNoY_VjLtFAIFjG7D)**

しかし、ブロックチェーンにおいては、チェーンの「不可逆性」としてファイナリティという言葉が使われる。これら二つの概念は全くもって別のものであるという認識が重要である。
従来の Bitcoin などによる PoW ではファイナリティは存在せず、ブロックが徐々に繋がることにより再編成やチェーン分岐の可能性が小さくなり、確率的にファイナリティがもたらされる仕組みになっている。つまり既存チェーンにおいては分岐可能性は常に存在し、絶対的なファイナリティがもたらされることはなく、時間経過により確率的に分岐可能性が収束することによる「疑似的なファイナリティ」がもたらされるにすぎない。
このような「確率的ファイナリティ」においては以下の課題が挙げられる。

**決済速度の低下**
前述の通り、時間経過による確率の収束を待たなければ決済が成立したと見なすことができず、即時決済などに向かない。

**トランザクションの不確定性**
ブロックの再編成・チェーンの分岐が発生した場合、破棄されたブロック内に含まれるトランザクションは無効となり、決済は失敗に終わる。これは「時間経過により決済成功とみなした後」にも起こり得る事象であり、ファイナリティ欠如の最たるデメリットとも言える。

## ボラリティの低さ

ボラリティとは「価値変動の度合い」を示す経済学用語である。ここでは各チェーンにおけるネイティブトークン（BTC や ETH など）のマーケット価格の揺れを指す。
多くのブロックチェーンでは大量のトランザクション処理要求などによる Dos 攻撃を防ぐため、トランザクションの実行に手数料を課している。この手数料は各チェーンのネイティブトークンで支払われることになるが、現在それらのマーケット価格の変動は非常に激しいものになっている。
例えばエンタープライズ向けなどにサービスを構築する際、手数料価格を含む実コストの予想が非常に困難となり、サービスの運用に支障をきたす可能性がある。また、外的要因によりネットワークを流れるトランザクション数が増加し、手数料が高騰（[Ethereum におけるガス代高騰問題](https://www.coindeskjapan.com/137693/)など）することも珍しくない。

## チェーンのガバナンス

パブリックチェーンにおいて、ガバナンス主体としての管理者は存在しない。これはブロックチェーンにおける大きなメリットでもあるが、同時にデメリットにもなりうる。例えばコンセンサスに影響をもたらすような大きな機能拡張、または致命的な欠陥の修正において、コミュニティ内でうまく意見が合わず、結果ハードフォークなどの結果に繋がる例も過去には発生している。2017 年には Bitcoin において、BIP49（SegWit）の導入に一部マイナーが反対し、実際に「Bitcoin Cash」としてハードフォークした事例がある。また、Ethereum における The DAO 事件では Ethereum コミュニティにより救済的なハードフォークが行われたが、これは当該事件に対する「特別な処置」でもあり、いわば「コミュニティからの神の手」である対に反対する一部派閥により、新たな分岐チェーン「Ethereum Classic」が誕生した。
このように明確な管理者が存在しないチェーンにおいて、その不在がしばしば問題となっている。

## 参照権限の有無

上記のような問題を解決するため、許可型ブロックチェーン「Hyperledger Fabric」が登場した。Hyperledger Fabric は参加者を限定した許可型のブロックチェーンであり、コンセンサスアルゴリズムには PBFT が採用され、許可を得た参加者であるかを API 層で任意に実装するなどで、トランザクション手数料の撤廃に成功している。またチェーン参加が許可型であることから許可主体によるガバナンスが取られ、上記のような課題は解決されている。
しかし、チェーン参加が許可型であると同時に、チェーン上のデータの参照権限も許可されたノードに限られる。つまり、オープンな台帳アクセスが提供されておらず、第三者による「データの正しさ」の証明が不可能である。

# Tapyrus による解決「ブロックチェーン層の二層分割」

これら既存チェーンの課題に対する Tapyrus の解決を紹介する。

Tapyrus ではネットワークを Signer Network と Tapyrus Network の二層に分割することを提案・実装している。
![[https://www.chaintope.com/chaintope-blockchain-protocol/](https://www.chaintope.com/chaintope-blockchain-protocol/)](https://www.chaintope.com/wp-content/uploads/2019/11/tapyrus-network-diagram.png)
[https://www.chaintope.com/chaintope-blockchain-protocol/](https://www.chaintope.com/chaintope-blockchain-protocol/)

**Signer Netwotk**
ネットワークの利害関係者からなるフェデレーションにより運営される。Signer Network 内の各ノードはブロックの署名（Sign）を担当し、承認されたブロックを Tapyrus Network へ伝播する。これにより Tapyrus では確率的なファイナリティではなく、ブロック作成の時点で各トランザクションにファイナリティが与えられる。

**Tapyrus Network**
フェデレーション関係者のみで構成される Signer Network とは異なり、Tapyrus Network はオープンな台帳ネットワークとなる。ジェネシスブロックおよびネットワーク ID により誰でも Tapyrus Network に参加することが可能であり、Tapyrus Network 内の各ノードは自由にチェーン上のデータを参照することが可能である。

これら二層へ分割することで、各課題は以下のように解決される。

**ファイナリティ** → Signer Network によるファイナリティの実現
**ガバナンス** → フェデレーションによる Signer Network の運営
**参照権限** → 自由に参加可能な Tapyrus Network によるオープンな台帳ネットワークの実現

# 技術仕様

Tapyrus の技術的な仕様を紹介する。

## ネットワーク構成

ネットワークは三種類のノードから構成される。

| ノード         | 数               | 目的                        |
| -------------- | ---------------- | --------------------------- |
| Tapyrus Signer | 最低 3, 上限なし | Signer Network を構築       |
| Tapyrus Core   | 最低 1, 上限なし | Tapyrus Network を構築      |
| Tapyrus Seeder | 最低 1, 上限なし | ネットワークの DNS シーダー |

また、ネットワークは以下のパラメータを持つ。

**Network ID**
ネットワーク ID。ネットワークを識別するものであり、独自である必要がある。

**Network magic bytes**
ネットワークマジックバイト。 マジックバイトはノード間通信時、送受信メッセージの接頭辞に付与され、ネットワークの識別子として機能する。マジックバイトはネットワーク ID から計算され、16 進数で表示される。

**Genesis block**
ジェネシスブロック。最初のブロック。Bitcoin とは異なりソースコード内には記載がなく、tapyrus-genesis というユーティリティを用いて生成される。ジェネシスブロックは Signer Network による署名が必要であり、各ノードはこれを外部ファイルとして保持する。ネットワークで独自である必要がある。

**Aggregate public key**
集約公開鍵。ジェネシスブロックに含まれる。各ノードがブロックを検証する際に用いる。

## Signer Network

Signer Network は構築時に設定されたフェデレーションのメンバーで構築される。このフェデレーションメンバーは後からでも編集することが可能である。その際には新たな集約公開鍵が生成される。

ノードの役割は主に

- Tapyrus Network で生成されたトランザクションを収集し検証する
- 有効なトランザクションからブロックを生成作成し署名する
- Tapyrus Nerwork へブロードキャストする

ことである。

### ブロックの検証

Signer Network の作成したブロックの検証は、ブロックの proof が有効な署名かどうかを検証することで確認ができる。検証に用いるデータは以下。

- ブロックの proof にセットされている署名データ
- 署名対象のメッセージ `m`（ブロックヘッダーから proof を除外したデータの double-SHA256 ハッシュ値）
- 集約公開鍵（全 Signer の公開鍵を加算し、集約したもの）

上二つは検証対象のブロックから取得でき、また集約公開鍵は公開鍵の集約にすぎないため、Tapyrus Network の参加者全てがこれらについて参照可能である。つまり、誰でもブロックの正しさを独立して検証することができる。

### 閾値署名方式

先述の集約公開鍵に対応するものとして集約秘密鍵がある。しかし、集約秘密鍵の生成には全ての Signer の秘密鍵を知る必要があり、これはセキュリティ上望ましい手法とは言えない。そこで Tapyrus では分散 Schnorr 署名と(t, n)閾値署名方式を基に、「集約秘密鍵を生成することなく集約公開鍵に対して有効な Schnorr 署名を作成する」方式をとっている。この方式では内部で「検証可能な秘密分散（Verifiable Secret Sharing: VSS）」を使用し、秘密分散法と Schnorr 署名を組み合わせ、n 人の Signer 中、閾値 t 個の部分署名を集めると有効な Schnorr 署名を完成させることができる閾値署名を提供する。これにより、全ての Signer の署名を収集する必要はなく、t 個集めれば良いため、部分的な Signer の故障に対しても堅牢である。（ただし閾値を満たす必要はある）

また、閾値 t は任意に設定することができるが、分母に対し過半数を超える値、もしくはビザンチン耐性を考慮した値が望ましい。
これにより生成される Schnorr 署名は、ネットワーク参加者から見ると単一の Schnorr 署名であるため、署名検証コストは一定となる。

**秘密分散**
秘密分散とは、秘密情報を複数のメンバーに分散する方法。

**Schnorr 署名**
Schnorr 署名は、1990 年にシュノアが提唱した離散対数問題の困難性に基づくデジタル署名であり、非対話ゼロ知識証明プロトコルである Schnoor プロトコルをデジタル署名に応用したもの。2021 年には Taproot により Bitcoin へ正式に導入された。

**ビザンチン耐性・ビザンチン将軍問題**
不特定のノードが一時的に故障しその後復旧することがある、あるいは故意に嘘の情報を他のノードに伝える可能性があるという前提の上での合意問題。全ノード数を `n` , 結託した嘘つきノード数を `f` とした時、 `3f≧n` の場合には、ビザンチン将軍問題を解くいかなるアルゴリズムも存在しないことが証明されている。

### Signer Network の処理フロー

各 Signer ノードはそれぞれの公開鍵で識別され、辞書順にインデックスが割り当てられる。
また、Signer がメッセージを交換し、一つのブロックを作成するまでの期間を「ラウンド」と呼び、各ラウンド毎にラウンドマスターが選出される。このラウンドマスターは Signer ノードのインデックスを基に決定される。
各ラウンドの流れを以下に示す。（ラウンドマスターは「全 Signer」に含まれる）

1. ラウンドマスターが選出される
2. ラウンドマスターは「候補ブロック（candidateblock）」を全 Signer に送信する
3. 全 Signer は「ブロックへ対する VSS（blockVSS）」をお互いに送信する
4. ラウンドマスターは全 Signer に「ブロック参加（blockparticipants）」を送信する
5. 全 Signer は「ブロックへの部分署名（blocksig）」をお互いに送信する
6. ラウンドマスターは受け取った blocksig を基に「最終的な署名」を作成し Tapyrus Network にブロードキャストする
7. ラウンドマスターは「完成したブロック（completedblock）」を全 Signer に送信する

https://camo.githubusercontent.com/d172ef8de9290ca64221c4884797d765b50b6a711ff12653637aca2161fcd586/68747470733a2f2f6d65726d6169642e696e6b2f696d672f65794a6a6232526c496a6f696332567864575675593256456157466e636d46745847354262476c6a5a53684e59584e305a5849704943302d5069424262476c6a5a53684e59584e305a5849704f69427a6247566c63434279623356755a43426b64584a6864476c76626c7875546d39305a5342735a575a304947396d4945467361574e6c4b4531686333526c63696b36494531686333526c63694269636d39685a474e6863335138596e49765069426849474e68626d52705a4746305a5342696247396a61317875515778705932556f5457467a644756794b534174506a3467516d39694f69426a5957356b61575268644756696247396a61317875515778705932556f5457467a644756794b534174506a3467513246796232773649474e68626d52705a4746305a574a7362324e7258473563626b3576644755676247566d644342765a69424262476c6a5a53684e59584e305a5849704f6942426247776763326c6e626d56796379427a5a57356b49485a7a637a78696369382d4948527649475668593267676233526f5a5849755847354262476c6a5a53684e59584e305a5849704943302d506942436232493649474a7362324e72646e4e7a5847354262476c6a5a53684e59584e305a5849704943302d5069424459584a7662446f67596d78765932743263334e63626b4a7659694174506a3467515778705932556f5457467a644756794b546f67596d78765932743263334e63626b4a7659694174506a3467513246796232773649474a7362324e72646e4e7a5847354459584a7662434174506a3467515778705932556f5457467a644756794b546f67596d78765932743263334e63626b4e68636d39734943302d506942436232493649474a7362324e72646e4e7a58473563626b3576644755676247566d644342765a69424262476c6a5a53684e59584e305a5849704f69424e59584e305a584967596e4a765957526a59584e3050474a794c7a34676347467964476c6a61584268626e527a49475a766369423061475538596e4976506942685a6e526c6369426d624739334c6a78696369382d4945686c636d55676432556759584e7a6457316c4948526f5a534138596e4976506e4268636e527059326c775957353063794268636d55675157787059325538596e497650694268626d5167516d39694c6c7875515778705932556f5457467a644756794b534174506a3467516d39694f6942696247396a61334268636e527059326c775957353063317875515778705932556f5457467a644756794b534174506a3467513246796232773649474a7362324e726347467964476c6a61584268626e527a58473563626b3576644755676247566d644342765a69424262476c6a5a53684e59584e305a5849704f69424659574e6f49484268636e527059326c775957353050474a794c7a3467596e4a765957526a59584e3049474a7362324e7263326c6e50474a794c7a34676257567a6332466e5a533563626b467361574e6c4b4531686333526c63696b674c54342d49454a76596a6f67596d78765932747a61576463626b467361574e6c4b4531686333526c63696b674c54342d49454e68636d39734f6942696247396a61334e705a317875516d39694943302d5069424262476c6a5a53684e59584e305a5849704f6942696247396a61334e705a317875516d39694943302d5069424459584a7662446f67596d78765932747a61576463626c7875546d39305a5342735a575a304947396d4945467361574e6c4b4531686333526c63696b36494531686333526c636942795a574e76626e4e30636e566a644478696369382d49475a70626d467349484e705a323568644856795a534268626d5138596e49765069427a64574a7461585167644738676447686c4946526863486c7964584d38596e4976506942755a58523362334a724c6a78696369382d4946526f5a57347349474a796232466b5932467a644478696369382d49474e76625842735a58526c5a474a7362324e7250474a794c7a34676257567a6332466e5a533563626b467361574e6c4b4531686333526c63696b674c54342d49454a76596a6f67593239746347786c6447566b596d787659327463626b467361574e6c4b4531686333526c63696b674c54342d49454e68636d39734f69426a62323177624756305a5752696247396a61794973496d316c636d3168615751694f6e73696447686c625755694f694a6b5a575a6864577830496e3139

## Tapyrus Network

Tapyrus Network は誰もが参加可能なブロックチェーンネットワークである。ネットワーク ID とジェネシスブロックさえ分かれば、tapyrusd を利用し接続可能。

ノードの役割は主に

- 全てのオンチェーンデータの提供
- トランザクションの発行、伝搬

Tapyrus のフルノード実装である Tapyrus Core は、Bitcoin Core をベースに作られている。

## トークン

多くのトークンプロトコルは、オーバーレイプロトコルである Layer2 で実装されている。しかし、これら実装は「任意データをブロックチェーンに記録する機能」を用いており、これは Layer1 におけるコンセンサスとして正しさを検証されない。
例えば Bitcoin におけるトークン「カラードコイン」を表現するプロトコルは複数存在し、Open Assets Protocol が主流である。しかし、Open Assets Protocol を通じて発行されたカラードコインは Bitcoin 単体ではカラードコインとして解釈することはできず、あくまで Open Assets Protocol を通して見た時に初めてカラードコインであると認識できる。これはウォレットの実装においても、仮に Open Assets Protocol を解釈しないウォレットにカラードコインが送られた場合、ウォレットは受信トークンをトークンと認識できず、ネイティブトークンである BTC として解釈する。またこの説明において、 “Bitcoin” がすなわち “Layer1” であり、”Open Assets Protocol” が “Layer2 プロトコル” である。
対して Tapyrus では Layer1 において、つまり外部のライブラリやプロトコル等に依存せず Tapyrus 単体での任意のトークンの発行・転送をサポートしている。

### トークン仕様

Tapyrus では以下の 3 種類のトークンタイプが発行可能である。

- Reissuable Token（再発行可能なファンジブルトークン）
- Non-Reissuable Token（再発行不可能なファンジブルトークン）（一部ドキュメントで Unreissuable Token と表記される場合もある）
- Non-Fungible Token（NFT）

上二つについては Fungible Token であり、複数枚の発行が可能である。
NFT については、ERC721 のような「コントラクトを基にトークンを発行し、Metadata を付与する」といった考え方ではなく、特定のアドレスに対し一意な ID を持ったトークンが発行されるものである。
トークンを識別するために、Tapyrus Script にて OP_COLOR opcode が追加されている。OP_COLOR は Script 内に一つのみ含めることができ、OP_COLOR が出現するとスタックの一番上の要素が “COLOR 識別子” として解釈され、その UTXO のコインは任意のトークンを示す。OP_COLOR のない Script はネイティブトークンとして解釈される。
つまり単に「OP_COLOR の有無」のみでネイティブトークンか否かを判断することができ、また後述の「COLOR 識別子」によりトークンの識別が可能であるため、ノード/ウォレットは UTXO 単体でトークンを解釈することが可能である。それにより、経由した使用済み TXO の全てを保持する必要がなくなり、軽量ノードなどにおいても容易にトークンを扱うことが可能である。

**COLOR 識別子**
CCOLOR 識別子は 1 バイトの「タイプ（TYPE）」と 32 バイトの「ペイロード（PAYLOAD）」で構成される。COLOR 識別子の内容は以下表に示す通り。

| タイプ | 定義                 | ペイロード                                       |
| ------ | -------------------- | ------------------------------------------------ |
| 0xC1   | Reissuable Token     | 発行 TX のインプットの scriptPubkey の SHA256 値 |
| 0xC2   | Non-Reissuable Token | 発行 TX のインプットの OutPoint の SHA256 値     |
| 0xC3   | Non-Fungible Token   | 発行 TX のインプットの OutPoint の SHA256 値     |

「発行 TX のインプットの OutPoint」
OutPoint とは特定の TXO を指定するために利用されるデータ構造であり、TXID とインデックス番号から成る。つまり NRT, NFT におけるペイロードは発行時に利用した「TXID とインデックス番号」すなわち UTXO を基に造られることになり、UTXO はチェーンで一意であるため COLOR 識別子にも一意性がもたらされる。

また、COLOR 識別子は、[tapyrus-cli にて token を発行](https://zenn.dev/shmn7iii/articles/65318fa90432f0#issue)した際には `color`、Tapyrus の Ruby 実装である[tapyrusrb においては](https://github.com/chaintope/tapyrusrb/blob/c1d1d538f8fa3d9066dd081df881ab70eefed401/lib/tapyrus/tx_builder.rb#L71) `color_id` と表記される。

以降では tapyrus-cli を用いた testnet での NFT 操作を交えて進める。

### トークンの発行

| インプット   | 発行に使用する UTXO                                                       |
| ------------ | ------------------------------------------------------------------------- |
| アウトプット | COLOR 識別子と OP_COLOR を使った scriptPubkey を含む UTXO（トークン本体） |
|              | お釣り UTXO                                                               |

トークン新規発行のトランザクションは、UTXO をインプットにセットし COLOR 識別子を導出、COLOR 識別子と OP_COLOR を使った scriptPubkey をアウトプットにセットすることで作成される。またトークンの発行量はアウトプットにおいて value としてセットすることが可能であり、インプットに指定された UTXO で発生するお釣りは非トークンアウトプットが作成され、従来通り TPC で返却される。

```bash
# 手持ち
$ tapyrus-cli listunspent
[
  {
    "txid": "3bc7c60867a3fe2e4c0b1be12cf930ee31f20e96c46028aa08071889e1f83c57",
    "vout": 0,
    "address": "1DZeRgPHQepFJrHiJLwvHibKY9XUtvTrKF",
    "token": "TPC",
    "amount": 1.00000000,
    "label": "",
    "scriptPubKey": "76a91489ce0fb90a30f9141cc2d2896fdd8e2252c1a61688ac",
    "confirmations": 219,
    "spendable": true,
    "solvable": true,
    "safe": true
  }
]

# 発行
$ tapyrus-cli -conf=/etc/tapyrus/tapyrus.conf issuetoken 3 1 3bc7c60867a3fe2e4c0b1be12cf930ee31f20e96c46028aa08071889e1f83c57 0
{
  "color": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22",
  "txid": "153dfc29ce70d7c249fc382055a44deefa72a4c48ee20ec1e3fa8e0c3f9f3120"
}

# txの内容
{
  "txid": "153dfc29ce70d7c249fc382055a44deefa72a4c48ee20ec1e3fa8e0c3f9f3120",
  "hash": "36a0e2a144bca76365bc7717e32002a5f9c767ae9d084527e38c8b9d7a234e4e",
  "features": 1,
  "size": 257,
  "locktime": 187790,
  "vin": [
    {
      "txid": "3bc7c60867a3fe2e4c0b1be12cf930ee31f20e96c46028aa08071889e1f83c57",
      "vout": 0,
      "scriptSig": {
        "asm": "304302206efe41c9968bf7edeac3c5c636806b82198626358cece9f4998841e693fcca49021f0096c08d46dcda64a2390fc6d95de0d6a3aa67225ded1744fbc3f856938bb1[ALL] 03f960211ba2f99001cfe168b2190c9eadc93f1e9504c629800228d38ce1b3cf78",
        "hex": "46304302206efe41c9968bf7edeac3c5c636806b82198626358cece9f4998841e693fcca49021f0096c08d46dcda64a2390fc6d95de0d6a3aa67225ded1744fbc3f856938bb1012103f960211ba2f99001cfe168b2190c9eadc93f1e9504c629800228d38ce1b3cf78"
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "token": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22",
      "value": 1,
      "n": 0,
      "scriptPubKey": {
        "asm": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22 OP_COLOR OP_HASH160 29dc588b58a35145d7281c6ccdcf1d38794fcaf7 OP_EQUAL",
        "hex": "21c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22bca91429dc588b58a35145d7281c6ccdcf1d38794fcaf787",
        "reqSigs": 1,
        "type": "coloredscripthash",
        "addresses": [
          "4ZyqgapFfb4Hz6HhgkXjFYMKUZoKjnc2RrsQ4gzQnpfafK4etwTsKeTNPAxKatXJ3whEEkPzKwjUeh3"
        ]
      }
    },
    {
      "token": "TPC",
      "value": 0.99999742,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 62cfafe2c7062cb73929b0fb0b8ce44259dbfa04 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a91462cfafe2c7062cb73929b0fb0b8ce44259dbfa0488ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1A1U29dqTfqvuUfTCMGZGiaZFhFCzCf6o7"
        ]
      }
    }
  ]
}
```

vout[0] の scriptPubkey asm において OP_COLOR opcode が Script 内に出現したことにより最上位要素が COLOR 識別子として解釈されている。

```bash
c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22 OP_COLOR ...
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                             これ
```

### トークンの転送

| インプット   | 移転対象のトークンを持つ UTXO                                                                                                           |
| ------------ | --------------------------------------------------------------------------------------------------------------------------------------- |
|              | 手数料用 UTXO（FeeProvider が自分の場合）                                                                                               |
| アウトプット | 送信先のアドレスに対して移転対象のトークンと同じ COLOR 識別子と OP_COLOR opcode を付与した UTXO（アドレスが書き換えられたトークン本体） |
|              | お釣り UTXO                                                                                                                             |

トークン移転のトランザクションは、トークンを持つ UTXO をインプットにセットし、「送信先のアドレスに対してインプットのトークンと同じ COLOR 識別子と OP_COLOR を付与」したアウトプットをセットする。インプット/アウトプット数やトークンの種類を複数指定することもできるが、トランザクション内で各トークンの総量が保存されている必要がある。

```bash
# 手持ち
user@nodeA:$ tapyrus-cli -conf=/etc/tapyrus/tapyrus.conf listunspent
[
  {
    "txid": "153dfc29ce70d7c249fc382055a44deefa72a4c48ee20ec1e3fa8e0c3f9f3120",
    "vout": 0,
    "address": "4ZyqgapFfb4Hz6HhgkXjFYMKUZoKjnc2RrsQ4gzQnpfafK4etwTsKeTNPAxKatXJ3whEEkPzKwjUeh3",
    "token": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22",
    "amount": 1,
    "scriptPubKey": "21c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22bca91429dc588b58a35145d7281c6ccdcf1d38794fcaf787",
    "confirmations": 1,
    "spendable": true,
    "solvable": true,
    "safe": true
  },
  {
    "txid": "153dfc29ce70d7c249fc382055a44deefa72a4c48ee20ec1e3fa8e0c3f9f3120",
    "vout": 1,
    "address": "1A1U29dqTfqvuUfTCMGZGiaZFhFCzCf6o7",
    "token": "TPC",
    "amount": 0.99999742,
    "scriptPubKey": "76a91462cfafe2c7062cb73929b0fb0b8ce44259dbfa0488ac",
    "confirmations": 1,
    "spendable": true,
    "solvable": true,
    "safe": true
  }
]

# 受け取り先アドレスを生成
# getnewaddressコマンドの第三引数に受け取るTokenのCOLOR識別子を指定する必要がある
# 第二引数はlabel。任意。
user@nodeB:$ tapyrus-cli -conf=/etc/tapyrus/tapyrus.conf getnewaddress "" c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22
vypPXVBfqAuVqVt6unNWvoMXdUpb6BsWqK1aL62GkAjtGxP3AY5QRGaXr5b7ZkNy2Z2zvFS9qcCRth

# アドレスを確認
# tokenが指定したCOLOR識別子となっていることがわかる。
user@nodeA:$ tapyrus-cli -conf=/etc/tapyrus/tapyrus.conf getaddressinfo vypPXVBfqAuVqVt6unNWvoMXdUpb6BsWqK1aL62GkAjtGxP3AY5QRGaXr5b7ZkNy2Z2zvFS9qcCRth
{
  "address": "vypPXVBfqAuVqVt6unNWvoMXdUpb6BsWqK1aL62GkAjtGxP3AY5QRGaXr5b7ZkNy2Z2zvFS9qcCRth",
  "token": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22",
  "amount": 0,
  "scriptPubKey": "21c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22bc76a914156279703d67559ecf434c10443c6b2551a94a4588ac",
  "ismine": false,
  "iswatchonly": false,
  "isscript": false,
  "istoken": true,
  "color": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22",
  "labels": [
  ]
}

# 転送
user@nodeA:$ tapyrus-cli -conf=/etc/tapyrus/tapyrus.conf transfertoken vypPXVBfqAuVqVt6unNWvoMXdUpb6BsWqK1aL62GkAjtGxP3AY5QRGaXr5b7ZkNy2Z2zvFS9qcCRth 1
3b49b5afb9027dd910007424d59c6cd1ce457ab08edb3b521c75163e6eaef500

# txの内容
{
  "txid": "3b49b5afb9027dd910007424d59c6cd1ce457ab08edb3b521c75163e6eaef500",
  "hash": "2d6159b377ff26a9b7e711be17ead9ef87228eeb6da0ce870c0b69516a49a64c",
  "features": 1,
  "size": 327,
  "locktime": 187795,
  "vin": [
    {
      "txid": "153dfc29ce70d7c249fc382055a44deefa72a4c48ee20ec1e3fa8e0c3f9f3120",
      "vout": 1,
      "scriptSig": {
        "asm": "304402200dc914aef786e52d69782ae910e7d2100df89d3bd79dd3d306820d64870b53c5022011e9152407d84df08170521c87cd3d14140ff975e4d161e63b6276ffffba720e[ALL] 02ccdaebb6da9df094d158b279e419fa1c485021ef2cf17fbc3959455c645ad03c",
        "hex": "47304402200dc914aef786e52d69782ae910e7d2100df89d3bd79dd3d306820d64870b53c5022011e9152407d84df08170521c87cd3d14140ff975e4d161e63b6276ffffba720e012102ccdaebb6da9df094d158b279e419fa1c485021ef2cf17fbc3959455c645ad03c"
      },
      "sequence": 4294967294
    },
    {
      "txid": "153dfc29ce70d7c249fc382055a44deefa72a4c48ee20ec1e3fa8e0c3f9f3120",
      "vout": 0,
      "scriptSig": {
        "asm": "76a914c560ac838ad1434d6178dd5da43ddf1afe698f6588ac",
        "hex": "1976a914c560ac838ad1434d6178dd5da43ddf1afe698f6588ac"
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "token": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22",
      "value": 1,
      "n": 0,
      "scriptPubKey": {
        "asm": "c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22 OP_COLOR OP_DUP OP_HASH160 156279703d67559ecf434c10443c6b2551a94a45 OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "21c397cad1a9664d9b99696f4c83badf8b3e0cdf54803a17066519aa859cdedd2a22bc76a914156279703d67559ecf434c10443c6b2551a94a4588ac",
        "reqSigs": 1,
        "type": "coloredpubkeyhash",
        "addresses": [
          "vypPXVBfqAuVqVt6unNWvoMXdUpb6BsWqK1aL62GkAjtGxP3AY5QRGaXr5b7ZkNy2Z2zvFS9qcCRth"
        ]
      }
    },
    {
      "token": "TPC",
      "value": 0.99999415,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 0b7b790f099e87ea17828b64fc433134ffaa253d OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a9140b7b790f099e87ea17828b64fc433134ffaa253d88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "123iMA5vDJgo89NkTosfKqhQXArL3p1Mcm"
        ]
      }
    }
  ]
}
```

### トークンの焼却

| インプット   | 焼却対象のトークンを持つ UTXO |
| ------------ | ----------------------------- |
|              | 手数料用の TPC を持つ UTXO    |
| アウトプット | お釣り UTXO                   |

トークン焼却のトランザクションは、トークンを持つ UTXO、および手数料用の UTXO をインプットにセットし、お釣り UTXO がアウトプットとして返却される。手数料は TPC で支払う必要がある。

```bash
# 手持ち
$ tapyrus-cli listunspent
[
	{
    "txid": "c5d3c6f8d29322834bcba84f71dc3624100d43be62ba906287cd306785241a82",
    "vout": 0,
    "address": "vy1vD2goKSGxz6RrAETJ2GfPC25SdNhtde1KMspx2oBRYDPsYWf815mjxLmzFAidPJvks3fLzDTET5",
    "token": "c37a34d9f3602badf8110d65ee06cc7647539e885c29a981d02b6ee36daf84e4ec",
    "amount": 1,
    "label": "",
    "scriptPubKey": "21c37a34d9f3602badf8110d65ee06cc7647539e885c29a981d02b6ee36daf84e4ecbc76a9141e09d72d4620a62aa27081afb0b1a590cc4f85ee88ac",
    "confirmations": 1,
    "spendable": true,
    "solvable": true,
    "safe": true
  },
	{
    "txid": "f91ad0bc723e62b8ec69928faec77706b70ebfef265a19c8ff17ee8713bbc0a5",
    "vout": 1,
    "address": "12dFZkyEo3XbPQmnNhGBn6ZWXpUgmKpvTy",
    "token": "TPC",
    "amount": 0.07932965,
    "scriptPubKey": "76a91411d3422c2d6164d16167c04b2195e135bc00302988ac",
    "confirmations": 2,
    "spendable": true,
    "solvable": true,
    "safe": true
  },
	...

# burn
$ tapyrus-cli c37a34d9f3602badf8110d65ee06cc7647539e885c29a981d02b6ee36daf84e4ec 1
6eb2dd62fb164590b3377d34111785e9ecfe4342b8417b41c0eeb94235e29170

# txの内容
{
  "txid": "6eb2dd62fb164590b3377d34111785e9ecfe4342b8417b41c0eeb94235e29170",
  "hash": "471018aa8ed5402d1a6ba70a2cc5d8b0d3b46e0bfd3fe353819c4ecbb870457d",
  "features": 1,
  "size": 338,
  "locktime": 188132,
  "vin": [
    {
      "txid": "f91ad0bc723e62b8ec69928faec77706b70ebfef265a19c8ff17ee8713bbc0a5",
      "vout": 1,
      "scriptSig": {
        "asm": "3044022056ddf6cd953602d9bac639902770c8803c86f6e4258d64afd090ff7a3ed5b2d802200d77b96e93c65d2b83e67eb53721d838a5f18634869fd14a22786dc1539c5649[ALL] 02c85ca219273c2e122519f6b7f97bd08e392bceae338e9d76ed59718d54038ea3",
        "hex": "473044022056ddf6cd953602d9bac639902770c8803c86f6e4258d64afd090ff7a3ed5b2d802200d77b96e93c65d2b83e67eb53721d838a5f18634869fd14a22786dc1539c5649012102c85ca219273c2e122519f6b7f97bd08e392bceae338e9d76ed59718d54038ea3"
      },
      "sequence": 4294967294
    },
    {
      "txid": "c5d3c6f8d29322834bcba84f71dc3624100d43be62ba906287cd306785241a82",
      "vout": 0,
      "scriptSig": {
        "asm": "304402205872ebe121b23831721220d81556d9e6e43611710f6ec7e98edd9ef8aae17c720220345b7defb538fbff85a3f53c96938c024c3164590e4768def8f65e25c275f642[ALL] 03b6f816de05e8748387ef07d6398c6ede9b2198c924af50f74194b9b7fc6287da",
        "hex": "47304402205872ebe121b23831721220d81556d9e6e43611710f6ec7e98edd9ef8aae17c720220345b7defb538fbff85a3f53c96938c024c3164590e4768def8f65e25c275f642012103b6f816de05e8748387ef07d6398c6ede9b2198c924af50f74194b9b7fc6287da"
      },
      "sequence": 4294967294
    }
  ],
  "vout": [
    {
      "token": "TPC",
      "value": 0.07926205,
      "n": 0,
      "scriptPubKey": {
        "asm": "OP_DUP OP_HASH160 da51263c611741c7085ea56a648e21e924d6268d OP_EQUALVERIFY OP_CHECKSIG",
        "hex": "76a914da51263c611741c7085ea56a648e21e924d6268d88ac",
        "reqSigs": 1,
        "type": "pubkeyhash",
        "addresses": [
          "1LuMXBAon9ry3cUwJFcPNX8JADBBu8GPK8"
        ]
      }
    }
  ]
}
```

これら 3 つの処理について、一つのトランザクションにまとめることが可能である。有効な組み合わせについては以下の資料を参照。

[https://docs.google.com/spreadsheets/d/1hYEe5YVz5NiMzBD2cTYLWdOEPVUkTmVIRp8ytypdr2g/edit#gid=0](https://docs.google.com/spreadsheets/d/1hYEe5YVz5NiMzBD2cTYLWdOEPVUkTmVIRp8ytypdr2g/edit#gid=0)

# 関連プロジェクト

## Tapyrus Core

https://github.com/chaintope/tapyrus-core
Tapyrus のフルノード実装。

## Docker image: tapyrus/tapyrusd

https://hub.docker.com/r/tapyrus/tapyrusd
Tapyrus Core の Docker イメージ。

## Tapyrus SPV

https://github.com/chaintope/tapyrus-spv
Tapyrus の軽量ノード。

## Tapyrus Signer

https://github.com/chaintope/tapyrus-signer
Signer Network を構成する Signer ノード実装。

## Tapyrus Faucet

https://testnet-faucet.tapyrus.dev.chaintope.com/
https://github.com/chaintope/tapyrus-faucet
chaintope が提供する testnet 用の公式 Faucet。Rails で実装されている。

## tapyrusrb

https://github.com/chaintope/tapyrusrb
Tapyrus の Ruby 実装。[JS](https://github.com/chaintope/tapyrusjs-lib), [Rust](https://github.com/chaintope/rust-tapyrus) ライブラリも存在するが、Ruby が一番開発が活発。

## Glueby

https://github.com/chaintope/glueby
「専門知識の必要なしに」をテーマに作られた Ruby ライブラリ。Gem で配布。Rails をサポート。
最近（2022 年 04 月 07 日）[v1.0.0 がリリース](https://github.com/chaintope/glueby/releases/tag/v1.0.0)された。

## Tapyrus API

利用方法: [https://github.com/chaintope/tapyrus-api-client-java#tapyrus-api-の利用法](https://github.com/chaintope/tapyrus-api-client-java#tapyrus-api-%E3%81%AE%E5%88%A9%E7%94%A8%E6%B3%95)
「Glueby を言語依存なく幅広く利用可能に」をテーマに作られた REST API。利用には OpenID による認証が必要。

**クライアントのサンプル**
https://github.com/chaintope/tapyrus-api-client-examples

## Tapyrus Guid

https://site.tapyrus.chaintope.com/
いつの間にか公式ガイドができてました。

## workshop

https://github.com/chaintope/workshop202107
ブロックチェーン勉強会で使われたサンプルコード。Glueby の使い方はここから体系的に学ぶことができる。

**参考**
https://github.com/chaintope/tapyrus-core/blob/master/doc/tapyrus/getting_started.md
https://github.com/chaintope/tapyrus-core/blob/master/doc/tapyrus/Tapyrus_Technical_Overview_ja.pdf
https://www.chaintope.com/chaintope-blockchain-protocol/
**「[ブロックチェーン技術概論 理論と実践](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=&ved=2ahUKEwjHn4ywm4v3AhXlyosBHWSiDSUQFnoECBkQAQ&url=https%3A%2F%2Fbookclub.kodansha.co.jp%2Fproduct%3Fitem%3D0000353684&usg=AOvVaw0Ze_PRLr6UVd6Rm75mLmTt)」** - 著：山崎 重一郎, 安土 茂亨, 金子 雄介, 長田 繁幸 - 講談社
