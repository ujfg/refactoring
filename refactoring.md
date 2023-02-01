# 第６章 リファクタリングはじめの一歩
## 関数の抽出
https://memberservices.informit.com/my_account/webedition/9780135425664/html/extractfunction.html
### before
```js
function printOwing(invoice) {
  printBanner();
  let outstanding = calculateOutstanding();

  // 明細の印字
  console.log(`name: ${(invoice.customer)}`);
  console.log(`amount: ${outstanding}`)
}
```
### after
```js
function printOwing(invoice) {
  printBanner();
  printDetails(outstanding);

  function printDetails(outstanding) {
    console.log(`name: ${(invoice.customer)}`);
    console.log(`amount: ${outstanding}`)
  }
}
```
### 動機
適切な粒度で切り出して分離することで、何をしているかのみを意識すれば良くなり、どうやってしているのかを意識しなくて良くなる
### 手順
- 関数として切り出し、適切に命名する
- スコープの外に出てしまう変数を特定して引数として渡す
  - 関数が入れ子定義できるなら上記が必要なくなる
- 該当箇所を関数呼び出しに置き換える
### ポイント
- 変数が多すぎるようなら「変数の分離」や「問い合わせによる一時変数の置き換え」を検討
## 関数のインライン化
https://memberservices.informit.com/my_account/webedition/9780135425664/html/inlinefunction.html
### before
```js
function getRating(driver) {
  return moreThanFiveLateDeliveries(driver) ? 2 : 1;
}

function moreThanFiveLateDeliveries(driver) {
  return driver.numberOfLateDeliveries > 5;
}
```
### after
```js
function getRating(driver) {
  return (driver.numberOfLateDeliveries > 5) ? 2 : 1;
}
```
### 動機
関数の本体が関数の名前と同じくらいわかりやすい時に、間接化のしすぎを防ぐ
### 手順
- ポリモーフィックなメソッドでないことを確認
- 関数をインライン化
### ポイント
手強い時は「ステートメントの呼び出し側への移動」
## 変数の抽出
https://memberservices.informit.com/my_account/webedition/9780135425664/html/extractvariable.html
### before
```js
return order.quantity * order.itemPrice -
  Math.max(0, order.quantity - 500) * order.itemPrice * 0.05 +
  Math.min(order.quantity * order.itemPrice * 0.1, 100);
```
### after
```js
const basePrice = order.quantity * order.itemPrice;
const quantityDiscount = Math.max(0, order.quantity - 500) * order.itemPrice * 0.05;
const shipping = Math.min(basePrice * 0.1, 100);
return basePrice - quantityDiscount + shipping;
```
### 動機
コード内の指揮に名前をつけたいが、関数として抽出するまでもない
### 手順
- 抽出する式に副作用がないことを確認
- 変更不可な変数を定義し、式の結果を代入
- 元の式を新しい変数で置き換え
### ポイント
- クラス内部ならメソッド化することも検討
## 変数のインライン化
https://memberservices.informit.com/my_account/webedition/9780135425664/html/inlinevariable.html
### before
```js
let basePrice = anOrder.basePrice;
return (basePrice > 1000);
```
### after
```js
return anOrder.basePrice > 1000;
```
### 動機
名前が式以上の意味を持たない
### 手順
### ポイント
## 関数宣言の変更
https://memberservices.informit.com/my_account/webedition/9780135425664/html/changefunctiondeclaration.html
### before
```js
function inNewEngland(aCustomer) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(aCustomer.address.state);
}
```
### after
```js
// phase1 呼び出し方を変えたい関数の中身を分離する
function inNewEngland(aCustomer) {
  const stateCode = aCustomer.address.state;
  return xxNEWinNewEngland(stateCode);
}
function xxNEWinNewEngland(stateCode) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(stateCode);  
}

// phase2 変数のインライン化
function inNewEngland(aCustomer) {
  return xxNEWinNewEngland(aCustomer.address.state);
}

// phase3 呼び出し箇所を順次置き換え
const newEnglanders = someCustomers.filter(c => inNewEngland(c));

const newEnglanders = someCustomers.filter(c => xxNEWinNewEngland(c.address.state));

// phase4 古い関数を削除、新たな関数名を元の名前に戻し、新たな関数の使用箇所を一括置換

```
### 動機
関数の名前を変えたい、パラメータの渡し方を変えたい
### 手順
- 簡易な手順
  - 関数を変更し、呼ばれている全ての箇所で呼び方を変更する
- 移行的手順
  - 例を参照
### ポイント
- エディタの力を借りてもできないこともある(移行的手順)