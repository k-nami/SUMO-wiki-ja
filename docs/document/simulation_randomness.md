# ランダム性

偶然性はシミュレーションのシナリオ中で現実を再現する上で重要です。
シミュレーションに偶然性を与える方法はいくつかあり、以下で解説します。

## 乱数生成(Random Number Generation:RNG)

SUMOは乱数生成に[メルセンヌツイスター](https://ja.wikipedia.org/wiki/%E3%83%A1%E3%83%AB%E3%82%BB%E3%83%B3%E3%83%8C%E3%83%BB%E3%83%84%E3%82%A4%E3%82%B9%E3%82%BF)アルゴリズムを実装しています。
この乱数生成器(RNG)は(任意の)seed値で初期化され、デフォルトでは**23423**です。
固定シード値からは同じ乱数列が出現するので、この設定によって全てのアプリケーションは決定論的に動くことになります。
シード値は**--seed[`<INT>`][intlink]**によって変更することができます。
**--random**を使うとシード値は現在のシステム時刻を使うようになり、完全にランダムな結果になります。

シミュレーションの異なる側面を分離するために、複数のRNGインスタンスが使われています。

* 車両読みこみ時のランダム性(車両タイプの分布、速度の分散...)
* 確率的交通流
* 車両運転ダイナミクス
* 車両デバイス
  
分離は読みこまれた車両がその前の車両に影響を及ぼさないように行なわれます。
全てのRNGが同じシード値を使います。

## 車両タイプと経路分布

車両のふるまいを動的に変更する最も簡単な方法は、読みこみ時に経路を確率分布に従って決めることです。
車両タイプ/経路はそれぞれ決められた確率に従って陽に与えられます。
詳細は[経路と車両タイプの確率分布]を参照してください。
これはきめのこまかい結果の制御を生む一方で、例えば車の全長を5m～7mの一葉分布によって選ぶといったモデルを作ることを難しくします。

## [速度分布]

デフォルトでは、SUMOの車両は通行する車線に決められた最高速度と密接しています(その車両タイプの最高速度が許す限り)。
こうした振舞いは`<vType>`-属性の`speedFactor`で変更することができます。

## 車両数固定の交通流

[DUAROUTER]、[DFROUTER]、[JTRROUTER]、これらのアプリケーションは**--randomize-flows**オプションをサポートしています。
このオプションが使用されると、`<flow>`要素で定義された車両はフロー中のインターバルごとに等確率で出発時間が割り当てられます(デフォルトでは車両のフローは丁度よく時間を空けて生成されます)。

## 車両数のランダムな交通流

[DUAROUTER]と[SUMO]はどちらも`probability`属性を持つ`<flow>`要素の読みこみをサポートしています。
この属性が使用されると(`vehsPerHour,number`または`period`のかわりに)、毎秒ごとに決められた確率でランダムに車両が放出されます。
この結果は[二項分布](https://ja.wikipedia.org/wiki/%E4%BA%8C%E9%A0%85%E5%88%86%E5%B8%83)に従います(小さな確率のもとでは、これはポアソン分布の近似になります)。
こうした交通流を2車線以上の道路でモデル化する場合、個々の車線に対して`<flow>`を設定することが推奨されています。


## 出発と到着の属性

`<flow>`,`<trip>`,`<vehicle>`の要素は属性`departLane`,`departPos`,`departSpeed`,`arrivalPos`の値に"random"を入れることができます。
値は挿入試行時(出発(depart)属性の場合)ごとか、到着時の値を再検証する必要があるとき(例: 経路の再選択後)にランダムに選ばれます。

## ランダム性に関する情報

* [randomTrips.py]() を使うことで、交通流をランダムな道路上に生成できるようになります。これはまた、到着確率をランダムにする機能もサポートしています。
* [OD2TRIPS]() はO/D行列から個々の移動を表現するときにランダム性を付加します。
* [DUAROUTER]() は[動的ユーザー割り当て]()時にランダム性を付加します。
* [経路選択をランダムにできる]()ことは、代替経路が使用されることを保証します。

[intlink]: basics_notation.md#データタイプ