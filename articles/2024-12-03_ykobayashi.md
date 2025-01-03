---
title: "自分たちのアジャイル開発の紆余曲折を振り返る"
emoji: "🦔"
type: "idea"
topics: ["agile", "scrum"]
published: true
published_at: 2024-12-03 10:00 # 公開する未来の日時を指定する
publication_name: micin
---

MiROHAでエンジニアをしている小林です。
この記事は [MICIN Advent Calendar 2024](https://adventar.org/calendars/10022) の3日目の記事です。
https://adventar.org/calendars/10022
前回はRiku Takeuchiさんの『[TypeScriptのエラー制御のベストプラクティスを考える](https://zenn.dev/micin/articles/2024-12-02_rikson_error-handling-best-practices)』でした。

この記事では、過去2年間のMiROHAのアジャイル開発の紆余曲折を紹介していきます。
アジャイル開発を採用しているが、なんとなくうまくいっていないという方の何かのヒントになれば嬉しいです。

## MiROHAのアジャイル開発

具体的にどんな取り組みをしてきたかに入る前に、軽くMiROHAのアジャイル開発について触れていきます。

MiROHAではスクラム開発を採用しています。
チームの人数は増減はありますが概ね10人程度で、内エンジニアは3-5人です。

スプリント期間は2週間であり、行っている会議は一般的なスクラム開発と変わらず、スプリントごとに行うプランニング、スプリントレビュー、レトロスペクティブと、毎朝15分程度のデイリースクラムです。
そのほか次作る機能の仕様を議論する場として、毎週バックログリファインメントを行う会議も実施しています。
この2年間、スプリントイベントの曜日が変更になったり、スプリント期間を1週間にしてみたりと多少の変更はありましたが、概ね変わっていません。

## スクラム開発を行う動機とは

顧客に価値あるプロダクトを早く提供したい、というのはプロダクト開発する人々が共通して目指すところだと思います。スクラム開発は、その目的を達成するためのプラクティスの1つになります。スクラム開発が具体的には何なのかについては、ここでは詳細を省きます。詳しく知りたい方は、[SCRUM BOOT CAMP THE BOOK](https://www.shoeisha.co.jp/book/detail/9784798167282)などの書籍を参考されると良いと思います。

ざっくりいうと、スクラム開発を行う動機として大事な点は、「価値ある」と「早く」の2つです。ここでいう「価値」とは顧客にとっての価値です。それを提供するには、プロダクトを開発する私たちは、何が顧客にとって価値があることなのかを知っている必要があるということになります。「早く」は文字通りの意味で、開発着手からリリースまでの期間が短いことを指します。

## 2年前当時の状況や課題感

2年前当時からMiROHAのスクラムチームのよかった点を挙げると、以下になると思います。

- 機能開発を素早く行うことができていた
- チーム内のコミュニケーションが活発だった
- 自主的に動けるメンバーが揃っていた

一方課題を感じていた点でいうと

- プロダクト戦略の方針、方向性がチームに浸透しておらず、機能開発の優先順位がわからない
- 目の前の機能開発を行うことに注目がいき、将来的な方向を見据えた設計ができていない

があったと思います。プロダクト戦略の方針、方向性をチームで共有できれば、より「顧客への価値」にフォーカスして開発できることになりますし、将来的な方向を見据えた設計ができれば無駄な開発が減り、開発速度はより向上します。
まさに「価値ある」と「早く」のどちらも改善の余地ありという状況でした。

この状況は2022年9,10月ごろのレトロスペクティブのログからも読み取れます。
> Aエンジニア) チームの規模に見合った選択と集中ができてない気がする。何を作るのか、なんで作るのか、という議論が足りていないのではないか
> 
> Bエンジニア) 事業戦略とバックログが一致していない感じがする?
> 
> Cエンジニア) バックログの優先順位付け・誰が次のバックログをつくるのか、しっかり決めた状態でプランニングしたい。
> 
>   Dエンジニア) ↑自分がサボってます。というのもプロダクト方針が自分としてもあやふやで決めれず困ってます…

当時は、バックログの優先順位付けなどを行う専任のプロダクトオーナーを置けておらず、優先順位づけなどの一部のタスクをエンジニアで行なっていました。
また、プロダクトオーナーからプロダクト戦略について情報共有が全くなかったわけではなく、ちょこちょこ共有してもらっていました。
しかし、他メンバー(特にエンジニア)のドメイン理解が不十分だったことや情報共有方法が洗練されていなかったために、各バックログの価値をうまく理解できず優先順位づけのタスクを十分にこなせていませんでした。

このドメインとプロダクト戦略を十分理解できていない状況が、リファインメントで一部のメンバーしか機能要件の議論に参加できていない状況を生み出し、その状況の中で開発された機能は今後の拡張の方向性をうまく反映できたものにはなっていませんでした。

2つの課題の緊急度でいうと、前者の「プロダクト戦略が浸透していない」の方が強く、まずはそちらの解決を中心に立て直しが進んでいきました。

## プロダクト戦略やドメインへの理解を深める

### 社内のドメインエキスパートとの交流会(2022年10月から3ヶ月くらい)

漠然とドメインがわかるとプロダクト開発もやりやすくなるのではという思いもあり、幸いMiROHAのチームには治験の現場で働いていたドメインエキスパートが数名いたことから、その方々にいろいろ聞いてドメイン知識を身につけていこう！となり始まったのがこの会でした。

形式的には特に事前に議題は決めず、30分ほど集まって気になっていることなどを聞くというゆるいものでした。
実際やってみて気づいたことは、「何を聞いたらいいのか分からない」ということ、そして「ドメイン知識は幅広く、エンジニアが全て理解するのは不可能であり、その必要もない」ということでした。
どちらも大きな問題でエンジニアからはなかなか質問が出ず、ドメインエキスパートもどごまで答えればよいのか迷うという厳しい結果にはなりました。

ただ、意味がない取り組みだったというわけではなく、多少でも治験の現場の人たちがどんな働き方をしているのか理解する機会にはなりましたし、最近形を変えて復活している取り組みでもあります。

### アジャイルサムライの勉強会,KPIやインセブションデッキの作り直し(2022年11月~2023年2月)

2022年の11月ごろから行われたのが、『[アジャイルサムライ](https://shop.ohmsha.co.jp/shopdetail/000000001901/)』の勉強会でした。スクラム開発に苦しんでいた時期でもあったので「アジャイル開発とは?」を改めて勉強しようというモチベーションから始まったものだっと記憶しています。
アジャイル開発について改めて知ることでできたのはもちろんのこと、後述の「インセプションデッキを見直し」も相まって、スクラム開発を基本からやり直す良い機会になりました。

### プロダクト戦略の再検討(2022年11月~2022年12月)

同時期にプロダクトオーナーを中心にプロダクト戦略の再検討を行いました。ビジネスチームなども含めた全員が参加する会議を2ヶ月間毎週のように行っており、チームの立て直しの緊急度が伝わる会議でもあります。
この場では、事業ドメインの共有から始まり、KPIやインセプションデッキの見直しも行いました。どちらも以前に作られたものはありましたが、作成当時のメンバーもすでに離れていたこともあり、作り直すことになりました。

この活動を通して、プロダクトオーナー以外の開発メンバーも各バックログの価値の理解も深まり、徐々に「優先順位がわからない」という状況は改善していきます。

「理解が深まった」とは言いましたが、プロダクトオーナーやビジネスチームの人たちの議論を聞いて「なるほど、確かにその方向が良さそうだ」と思える程度の荒い理解で、議論ができるレベルまで深まったわけではなかったです。議論できるレベルまで解像度が高めることは、簡単なことではなく時間がかかることで、2ヶ月でそのレベルまで達することはできませんでした。

また、戦略は状況に応じて変わるものですし、2ヶ月みっちり話したらもう話さなくてOKというものでもありません。事業戦略について考える機会は、現在まで月1回のマンスリーミーティングという形式で続いており、営業などMiROHAのチーム全員でワークショップ等を通して議論を深める場として残っています。

## 対面の議論は大事

スクラム開発とは直接は関係ないかもしれませんが大切なことだと認識しているので、対面のコミュニケーションを大事にしていることにも触れておきたいと思います。

DCTチームはリモートで働くことを基本としていますが、適宜対面のコミュニケーションを活用することも大事にしています。体感の話ですが、ビデオ通話に比べて対面の方が、密に会話ができて互いの理解が進むのが速い気がしますし、ホワイトボードなどのツールが使えるのも良い点です。

前述のプロダクト戦略の再検討も対面で行われましたし、マンスリーミーティングは今でも対面です。エンジニア間での詳細設計の検討会や、複雑な問題について検討する場合にも対面で行うようにしています。
対面のコミュニケーションの大切さを再認識して、日々の会議の中で活用し出したのも、おそらく2023年初めごろからだったのではと記憶しています。

## 将来的な方向を見据えた設計をどう試行錯誤する(2023年後半~)

ドメインや事業戦略について考え理解する機会が増え、このあたりから「バックログの優先順位づけができない」などの状況が改善すると、どう効率よく良いプロダクトを作っていくかによりフォーカスされるようになっていきました。
一度作った機能を拡張する機会が増えたことも、拡張性高い設計や効率の良い開発への関心が強まった理由だったと思います。

また、この頃も以前よりは議論は活発になってきたものの、リファインメントでの議論はまだまだ一部のメンバーが話すだけの状況はそれほど変わっていませんでした。

余談ですが、チームとしてはQAも含めた開発効率を考えるようになっており、その理由等は[こちらの記事](https://zenn.dev/micin/articles/why-qa-bottleneck-is-your-matter)に記載されており、良かったらご覧ください。

### 将来を見据えた設計とは

システムの設計を考える時、事業的な側面を考慮することは重要です。事業が成長する中で、当然プロダクトも成長していきますし、その2つの成長の方向性は一致します。あらゆる拡張に対して対応できる設計は、過剰に複雑になりやすく開発生産性が低くなる傾向にあります。それならば、事業拡大の方向を見据えて、拡張性を保つことができれば良いという話になります。
また、どこくらい先まで見据えていくかを考えておくことが重要です。数年先まで考えた設計したところで解像度が荒いため、いざ手を加えるとなった時結局作り直しになりやすく、そんなに遠くのことまで想定したところで旨みがないケースがほとんどだからです。
MiROHAでは半年から1年先くらいで行われる機能改修までを想定して設計を進めることにしています。

### 現在の状態と将来像の認識をそろえる(2024年~)

一般的にプロダクトの現在像を一番よく知っているのはエンジニアです。一方、その将来像を最もよく把握しているのは要件定義を多なっているプロダクトオーナーになります。また実際の作業に着目してみますと、プロダクトオーナーは現在像を踏まえつつ要件定義を作っていきますし、エンジニアはその要件定義に従って、その将来像を実際に動くプロダクトへ変換していきます。つまり、プロダクトオーナーは現在像を把握することも、エンジニアが将来像を認識していることも、正しい方向にプロダクトを進化させていくには必要なことと言えます。

この現在の状態と将来像の２つを把握するために、MiROHAでは2つのドキュメントを用意しました。2つともドメイン駆動設計に関連する書籍などから着想を得ています。

1つめは、ドメインモデル図です。ここでいうドメインモデルとは、プロダクトのドメインの主要な概念のことであり、クラス図はER図に比べると抽象度の高いものを指しています。こちらには現在の状態、ないしは次の機能開発後のドメインモデルの関係図を作成しています。機能開発ごとに、実装方針を検討する中で適宜プロダクトオーナーとイメージをすり合わせています。

![ドメインモデル図の一部抜粋](/images/domain_model.png)

2つめは、コンテキストマップで、こちらはドメインモデル図よりもさらに抽象度が高いものになります。Bounded Contextごとに主要なドメインモデル数個が記載されているくらいのもので、こちらをベースに1-2年先くらいの将来像を開発チームとして認識をそろえるために使用しています。見直しの期間は半年に1回くらいで今は運用しています。

![コンテキストマップ](/images/context_map.png)

上記2つのドキュメントを作成して一番気づいたことは、エンジニアとプロダクトオーナーの間で想像しているシステムの内部構造にずいぶん差があることでした。ドメインモデル図で言えば「モデルAとモデルBは紐づいている」とプロダクトオーナーが思っていたものが実は紐づいていないというは度々ありましたし、一度擦り合わせても実装のやりやすさから影響が少ないと判断して思って他の形に変えた結果プロダクトオーナーのイメージと合わないモデルになってしまうこともありました。同じシステムを一緒に作り、同じ用語でできるだけ会話している開発チーム内でもイメージに差があるのは驚きもありつつ、図を使って認識をそろえていく必要性を感じた場面でもありました。

要件定義と詳細設計の中間ポイントとして、一度ドメインモデル図を使って実装方針のイメージをする合わせられるようになり、それまでの要件定義→詳細設計の流れに比べて、実装と将来像のギャップは小さくできていると感じています。

## 今後の課題

ただ、まだまだ課題もあります。たとえば、ドメインモデル図だけでは要件定義の詳細まですり合わせできず、見落としていた要件により開発途中で実装方針に大きな変更が入ることがあります。解決方法として、要件定義と実装方針を細かく擦り合わせながら同時に作成していく、などを模索しているところですが、未だ明確な解決には至っていません。

他にも、スケールするチーム体制になっているかも今後課題になってくると考えています。この2年間開発のコアなメンバーがほとんど変わらず阿吽の呼吸で開発が進んでいる一面があり、新しいメンバーにもスムーズにチームに参加してもらえるようドキュメントの整備に力を入れ始めているところです。

## まとめ

色々試行錯誤しましたが劇的に変わる手立てはなく、徐々に変わっていきました。2年前時点で今と同じ開発フローを実践したところで今と同じパフォーマンスが出ないと思っています。その時々で、困ったことを相談し合い、色々試したことが今の状態まで改善した要因だと思います。また、今回紹介した内容はあくまで、私たちのチームの場合の話であり、メンバーが異なれば他の解決方法が良いこともあると思います。

振り返ってみると『[アジャイルソフトウェア宣言](https://agilemanifesto.org/iso/ja/manifesto.html)』の「プロセスやツールよりも個人と対話を」という言葉の通り「対話」が鍵だったと考えています。ある程度人数がいる事業になれば、大きな戦略方針を立てる人、何を作るか考える人、手を動かして作る人はそれぞれ別の人になります。その間でのイメージのズレをドキュメントだけで解消することは難しく、対面でのコミュニケーションが必要になります。チームとして、それを大切にしてこられたことがよかったと思っています。

もちろんここに記載した話だけが成熟していく要因になったわけではありません。たとえば、途中から専属のプロダクトオーナーを置けるようなったことや、定期的に外部のドメインエキスパートにヒアリングできる機会を持てるようになったことはとても大きかったと思います。今回は紹介していませんが、他にもロールプレイや、日々自分たちのシステムを触ってみる仕組みを作ったりといろんな活動を通じて、ドメインへの理解を深め、それをプロダクトへ反映できるよう開発を進めてきました。

私たちも先ほどあげた通り、まだまだ課題は残っており道半ばです。半年後には全く違う取り組みを始めているかもしれません。それもまた何かの機会があれば共有できればと思います。

以上、特に何かのベストプラクティスを紹介する記事ではありませんが参考になれば嬉しいです。
