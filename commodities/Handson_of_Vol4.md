# Tokyo Quantopian User Group Vol 4
shinseitaro

---
## 自己紹介

[しんせいたろう(@shinseitaro)](https://twitter.com/shinseitaro)

#### 個人トレーダー

+ VIX ETF など

#### コミュニティ

+ [fin-py](https://fin-py.connpass.com/) / [Pythonアルゴリズム勉強会](https://python-algo.connpass.com/)
+ 主催：[Tokyo Quantopian User Group](https://quantopian-tokyo.connpass.com/) / [月刊フィントーク](https://fintalk.connpass.com/) / [モグモグDjango](https://mogumogu-django.connpass.com/)
+ スタッフ：[GTUG Girls](https://gtuggirls.connpass.com/) / オプション勉強会


---
## 今日の予定

1. Quantopian Algorithmの書き方

    アルゴリズムの基本的な書き方と注意など

2. バックテスト結果の見方

    + Build Algorithm
    + Full Backtest
    + Tear Sheet

3. Future Algorithm：

    + クラッシュスプレッド
    + カレンダースプレッド
    + その他

---
## 資料

### [Tokyo Quantopian User Group Vol4 Handson Algorithm](https://www.quantopian.com/posts/tokyo-quantopian-user-group-vol4-handson-algorithm)

---

## Quantopianとは

+ クラウド上で数量分析ツールを提供するボストンのFund
+ 成績の良いアルゴリズムをFundで運用．作者にReturnの数パーセントを支払う．
+ 定期的にコンテストを開催．[Getting Capital Allocations](https://www.quantopian.com/allocation)

参照： [Frequently Asked Questions](https://www.quantopian.com/faq)

---

## Quantopianがユーザに提供しているもの

+ ヒストリカルデータ
    + 株価，ファンダメンタル. 基本的に無料．一部有料のファンダメンタルデータあり
    + [Quantopian Data](https://www.quantopian.com/data)    
+ バックテスト機能
+ リサーチ機能
+ 分析ライブラリ


## 先物データ

+ [available-futures](https://www.quantopian.com/help#available-futures)
+ [Continuous Future Data Lifespans](https://goo.gl/KbFxjx)

---
## Quantopian Algorithmの書き方 基本編

+ [tokyo-quantopian-user-group-vol4-handson-algorithm](https://www.quantopian.com/posts/tokyo-quantopian-user-group-vol4-handson-algorithm)
+ 今回使った関数等のドキュメントは，この資料の最後の[DOCS](https://github.com/shinseitaro/TQUGVol4/blob/master/PITCHME.md#docs)にまとめていますのでご利用下さい．

---

## ストラテジー内容

+ 原油期近
+ 毎朝，前日のクローズと見比べて，前日比 +1.0% 以上であればロング，-1.0％以下であればショート
+ アルゴリズムの説明用に作成したので，ストラテジーに全く根拠無し．

---
### 必要なライブラリー

```python
import pandas as pd 
# 注文時使用
from quantopian.algorithm import order_optimal_portfolio
import quantopian.optimize as opt
```

---
### def initialize(context): 

```python
def initialize(context):
    ．．．
```
+ アルゴリズムの初期化や設定を行う必須関数.
+ 定義されていないとエラー `Algorithm must implement initialize(context) method.` 
+ 必ず `initialize(context)`
+ 引数は `context` 一つ

+ 関数の中には，
    + 銘柄設定
    + スケジュール設定
    + 取引に必要な属性を設定

---

#### `context`

**役割1．アルゴリズムのポートフォリオやアカウントの状態を保持**

+ `context.account`
+ `context.portfolio` 

**役割2．ユーザー定義の属性を追加**

+ `context.my_property` のように '.' で属性を追加出来る．

    こうすることで，他の関数に `context` を引数として渡し，他の関数がその属性にアクセス出来る．

    <font color=red>Quantopianでは，グローバル変数を使うのは避ける</font>

    その代わり，この `context` の属性に情報を設定して，他の関数に渡すようにする．

--- 

**OK例**

```python
def initialize(context):
    ・・・
    context.myname = "hogehoge"
    
def other_func(context):
    print context.myname 
    
```

**NG例**

```python
myname = hogehoge
def initialize(context):
    ・・・

def other_func():
    print myname 
    
```
---
#### context の説明コード

1. https://www.quantopian.com/algorithms
1. New Algorithm 
1. Create New Algorithm で適当な名前（testなど）で作成
1. 左側のコードは一旦全部削除
1. スタートストップを短い時間に変える
1.  

```python

def initialize(context):
    context.myname = "hogehoge" 
    schedule_function(say)
    
def say(context, data):
    
    log.info(context.myname)
    log.info(dir(context)) # オブジェクトの属性を確認
    
    order(sid(24),1)
    log.info(context.account)
    log.info(context.portfolio)
    
```
---

### 先物オブジェクト

+ `continuous_future()`: つなぎ足先物

+ `future_symbol()`: 一代足先物

---

### つなぎ足

```python
context.my_future = continuous_future(
    'CL', # 先物のrow symbol
    offset=0, # 限月．期近＝0，2番限＝1，3番限=2，・・・
    roll='calendar', # ロールするタイミング
    adjustment=None # アジャスト方法
    )
```

このインスタンスを使ってヒストリカルデータにアクセスしたり，
各バックテスト日のターゲットのコントラクトを取得したりする．

---

### 一代足

```python
## 原油２０１７年１月限 (10/01/2016~12/20/2016)
context.my_future = future_symbol("CLF17")
```

（今回の勉強会では使っていません）


---
### スケジューラー

```python
schedule_function(my_rebalance1, # 実行したい関数名
    date_rule=date_rules.every_day(), # 日付ルール
    time_rule=time_rules.market_open(hours=1)) # 時間ルール
```
market open is at 6:30AM ET / market close is at 5PM ET.

参照：[Scheduling Functions](https://www.quantopian.com/help#ide-schedulefunction)

---

### ヒストリカルデータ

#### data 

OHLCVなどのデータや，現在のコントラクトのデータ，次のコントラクトに関するデータ，アセットが取引可能かどうかのデータなど，データにアクセスするためのオブジェクト．

#### `data.history` 

ヒストリカルデータを取得．引数の内容で，pd.Seriese / pd.DataFrame / pd.Panel のいずれかを返す

```python
price = data.history(
    context.my_future, # 先物インスタンス
    fields ='price',  # フィールド
    bar_count = 2, # 何個取るか
    frequency = '1d' # 日足
    )
```

ユーザーは，**自分でつなぎ足をつくることなく**，この方法でヒストリカルデータを取得できる．

乗り換えの日のデータは，`continuous_future` で指定した `roll`と`adjustment` オプションに従って作成される．

注意：ヒストリカルデータの最新のデータは，data.historyを実行した時に取れる最新データ. 詳細は[こちら](https://github.com/shinseitaro/TQUGVol4/blob/master/PITCHME.md#datahistory-1)

---
### data.current

バックテストが走っているその日の情報を取得する．

取得出来るデータは

+ コントラクト
+ 価格
+ 出来高

---
#### contract

```python
current_contract = data.current(context.my_future, 'contract')
```

今日現在のコントラクトを取得する．

このコントラクトを使って，現在価格取得や注文を行う．

#### 現在価格

```python
data.current(context.my_future, 'price')
```
(基本編アルゴリズムでは使っていません）



---

### 注文

#### 先物を一枚注文する時：

```python
## def my_rebalance1 の中にある

if context.ratio < -0.01:
    order_target(current_contract, -1) # ショート
elif context.ratio > 0.01:
    order_target(current_contract, 1) # ロング
else:
    order_target(current_contract, 0) # クローズ
```

+ `order_target()` に コントラクトと枚数を渡す．
+ 既に指定の枚数を持っている場合は，追加注文はしない．



---
#### 現在のポートフォリオの評価額に対して〜％持つように注文

```python
## def my_rebalance2 の中にある
target_weights = dict()

if context.ratio < -0.01:
    target_weights[current_contract] = -1.0
elif context.ratio > -0.01:
    target_weights[current_contract] = 1.0
else:
    target_weights[current_contract] = 1.0

if target_weights:
    order_optimal_portfolio(
    opt.TargetWeights(target_weights),
    constraints=[])
```

+ `target_weights` 辞書（命名は自由）に，対象のコントラクトをポートフォリオに対して何％オーダーするという注文方法．
+ `opt.TargetWeights()`に `target_weights` を渡してオーダー
+ `constraints`は割愛．[参照](https://www.quantopian.com/help#constraints)

---

#### 注文方法について

+ https://www.quantopian.com/help#api-order-methods
+ Quantopianでは，`order_optimal_portfolio` が推奨されている．
+ [order_optimal_portfolio](https://www.quantopian.com/help#api-order-optimal-portfolio) / [Optimize API](https://www.quantopian.com/help#optimize-api) を参照．

---
### ロギング

```python
log.info("order short %s" % context.ratio )
```

+ `log.info(msg)`．その他レベルはDOCS参照
+ `print`文も使える．

---
### その他

`context.portfolio.positions ` ：ポジションを持っている場合，辞書型でポートフォリオ情報を返す．


---
## バックテスト結果の見方

### Build Algorithm

文法チェックと簡単な結果を表示

1. スタート，エンド，初期設定資金額
1. **US Futures** を指定

![Screenshot from 2018-05-16 14-30-59.png](https://qiita-image-store.s3.amazonaws.com/0/14019/5cb465ac-b6c9-aa01-a862-dec001a1cb6f.png)

1. Build Algorithm 押下

---
### Full Backtest

+ [Quantopian Contest](https://www.quantopian.com/contest)用に開発.
+ リターンやリスク等をオサレに表示してくれる．
+ 詳しくは，こちら→ [Improved Backtest Analysis](https://www.quantopian.com/posts/improved-backtest-analysis)

---
### Full Backtest 詳細

### Risk
+ Leverage 資本金に対してポジションの額（毎日ベース） End-of-day gross leverage. A measure comparing position value to capital base.
+ Turnover アセットに対して投資資金額の比率．（63日移動平均） Turnover represents the rate at which assets are being bought and sold within the portfolio. The value displayed is the rolling 63-day mean turnover.
+ Beta To SPY 自分のポートフォリオのリターンとSPYのリターンの相関関係（6ヶ月移動BETA)．6-month rolling beta to SPY. Trailing beta measures the correlation between the portfolio's overall return stream and the returns of SPY.
+ Position Concentration ポートフォリオの中で1番多く保有しているアセットは，ポートフォリオの中で何％保有しているか．（毎日ベース） End-of-day position concentration. The percentage of the algorithm's portfolio invested in its most-concentrated asset.
+ Net Dollar Exposure ポートフォリオ中のロングとショートのポジション比率

---
### Performance
+ Total Return バックテストスタートからエンドまでのリターン The total percentage return of the portfolio from the start to the end of the backtest.
+ Sharp リターンを標準偏差で割ったもの（6ヶ月移動）
+ Max Drawdown 
+ Volatility

---
### Activity

+ Position 毎日のポジション
+ Transaction トレードログ
+ Custom Data record() したデータ
+ Logs 取引中に出したログ
+ Code この full backtest で使用したコードのスナップショット．

---
### Notebook
+ jupyter notebook
+ 最初のセルを実行すると，一分くらいでTear Sheet が作成される
+ tear sheet を見て，アルゴリズムの改良を考える
+ 作った tear sheet は https://www.quantopian.com/research にストラテジー名と実行時刻とハッシュキーで格納されるので，いつでも確認できる．

---

## Future Algorithm クラッシュスプレッド

+ [Tokyo Quantopian User Group Vol4 Handson Algorithm](https://www.quantopian.com/posts/tokyo-quantopian-user-group-vol4-handson-algorithm#5b34cbb7f895fe719c19092c)
+ ここのNotebookはどりらんさんに作ってもらいました！ありがとうございます．

### Research 機能について

JupyterNnotebook を使って，自分のアイデアを検証出来るツール. 
Quantopianがヒストリカルな価格データやファンダメンタルデータを無料で用意しているのでそれを使って検証．


### 原油のクラックスプレッドを考える

+ 'CL'(Light Sweet Crude Oil), 'HO'(NY Harbor USLD Futures (Heating Oil)), 'XB'(RBOB Gasoline Futures), 'NG'(Natural Gas)のヒストリカルデータを取得
+ 各ペアごと，同じ限月の割合を算出


---
### 特徴を見てみる

+ offset 3 (4限月)やoffset 4 (5限月) の HO/XB や CL/XB あたりが何かアヤシイ
+ とくに HO/XB は1の周りをウロウロしているのでコードも簡単に書けそうな予感．
+ （コード説明，ビルドテスト）

### ストラテジーを書いてみる．

+ 投資対象．HOとxbの4限月もしくは5限月
+ 20日分のヒストリカルデータを取得し，HO/XB の比率を取得し，標準偏差を取得．
+ 直近の HO/XB に 標準偏差を足しあわせ，それが1.1より大きい場合は，HOをショート XBをロング
+ 比率が１に戻って来たところでポジションクローズ

---
## カレンダースプレッド

+ [Tokyo Quantopian User Group Vol4 Handson Algorithm](https://www.quantopian.com/posts/tokyo-quantopian-user-group-vol4-handson-algorithm#5b35e3c0a6d2980046eb1ed2)
+ 上記と同じ先物４商品のフォワードカーブを表示．

### 特徴を見てみる

+ がーすさんの資料にあった通り，両端と真ん中が逆の方向に動いている事がよくあるのがわかる

### ストラテジーを書いてみる．

+ 対象は CL
+ `context.f1`, `context.f2`, `context.f3` という属性を作り，３つの限月の先物オブジェクトを比べる
+ 凹凸があるかどうかは，両端の平均と，真ん中の価格がどの程度違うかで判断．


---
## DOCS

### schedule_function

`schedule_function(func,
                   date_rule=date_rule,
                   time_rule=time_rule,
                   calendar=calendar,
                   half_days=True)`

func: 指定した時間に実行したい関数名
date_rule: 日付の指定．
+ `date_rules.every_day()`：毎日（デフォルト）
+ `date_rules.week_start(days_offset=0)`：週初めからdays_offset日分オフセットした日．
+ `date_rules.week_end(days_offset=0)`：週終わりからdays_offset日分巻き戻した日．
+ `date_rules.month_start(days_offset=0)`：月初めからdays_offset日分オフセットした日．
+ `date_rules.month_end(days_offset=0)` ：月終わりからdays_offset日分巻き戻した日．

time_rule：時間の指定．
+ `time_rules.market_open(hours=0, minutes=1)`（デフォルト）：取引開始時から〜時間〜分後
+ `time_rules.market_close(hours=0, minutes=1)`：取引終了時から〜時間〜分前

calendar：カレンダーの指定．デフォルトは，バックテスト時に選んだカレンダー．
+ `calendars.US_EQUITIES`
+ `calendars.US_FUTURES`

half_days：マーケットが半日しか開いていない日（例；クリスマスの次の日）にスケジュールを実行するかどうか．デフォルトはTrue．

【メモ】： 先物の場合：market open 6:30AM ET 〜 market close 5PM ET.
[schedulefunction](https://www.quantopian.com/help#ide-schedulefunction)参照

---

### data.history

`data.history(symbols, fields, bar_count, frequency)`:

+ ヒストリカルデータ取得．データは引数によって， `pd.Series` `pd.DataFrame` `pd.Panel`のいずれか
+ symbols: 銘柄，もしくは銘柄リスト．
+ fields: フィールド，もしくはそのリスト．`price`, `open`, `high`, `low`, `close`, `volume`．複数選択可．
+ bar_count: int. `frequency` で`1d`をした場合は，過去n日分．`1m`をした場合は，過去n分分．
+ frequency:　日足 = `1d`, 分足 = `1m`　のいずれか

---
**注意**
+ data.historyで取得したヒストリカルデータの最新のデータは，データを取ったタイミングのデータ．
+ 2017-03-16 10:30AM に `frequency = '1d'`, `fields ='price'`,`bar_count = 4`, で取得した場合，

時刻|AAAA|BBBB|CCCC
---|---|---|---
2017-03-13 **16:00**|6.64|10.98|6.94
2017-03-14 **16:00**|10.45|7.62|8.57
2017-03-15 **16:00**|9.84|5.96|3.26
2017-03-16 **10:30**|7.42|2.40|4.26


---
### future_symbol

`future_symbol(symbol)`

+ 一代足先物オブジェクトを返す
+ symbol: 文字列．

symbolの書き方

+ Symbol+月コード+年コード
+ 例 CLF160 CL=CrudeOil F=January 16=2016

![](https://www.quantopian.com/assets/futures_getting_started1_l2_screenshot1-0bc90a54a7e14712bce267e0a7b0bb6e06de3c5ddf302c8102274f687628f68a.png)

---
### continuous_future

`continuous_future(root_symbol, offset=0, roll='volume', adjustment='mul')` [参照](https://www.quantopian.com/help#quantopian_research_experimental_continuous_future)

+ つなぎ足オブジェクトを返す．（このオブジェクトは．`specifier` と呼ばれている）
+ root_symbol: str. 原資産シンボル．利用できる先物は[こちら](https://www.quantopian.com/help#available-futures)
+ offset: int. 限月．期近＝0，2番限＝1，3番限=2，・・・
+ roll: ロールのタイミングを指定．
    + `volume` (default) : 次の限月の取引量が期近の取引量を超えた時ロールする．これは，マーケットのトレーダーが次の限月に移ったというアイデアがベースになっている．もし，auto_close_dateまでにこの現象が起きなければ，auto_close_dateがロール日になる．また，ロールはauto_close_dateの７営業日前まで起きない．
    + `calendar` : コントラクトがauto_close_dateに達した時にロールする

+ adjustment: アジャスト方法
    + `mul` (default): 連続したコントラクトの価格比を価格シリーズに掛けたデータ．
    + `add`: 連続したコントラクトの価格差を加算したデータ
    + None: アジャスト無し



---

### data.current

`data.current(assets, fields)`  [参照](https://www.quantopian.com/help#api-data-current)

+ assets の現在価格や，コントラクトを返す．価格は，実際の取引価格．
+ assets: `continuous_future` オブジェクト．もしくはそのリスト．
+ fields: 'price', 'last_traded', 'open', 'high', 'low', 'close', 'volume', 'contract' のいずれか．もしくは複数入ったリスト．



---
参照

[Quantopian Futures API Tutorial](https://www.quantopian.com/posts/quantopian-futures-api-tutorial)

[Continuous Future Data Lifespans](https://goo.gl/KbFxjx)

[available futures](https://www.quantopian.com/help#available-futures)







