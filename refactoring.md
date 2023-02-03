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
// step1 呼び出し方を変えたい関数の中身を分離する
function inNewEngland(aCustomer) {
  const stateCode = aCustomer.address.state;
  return xxNEWinNewEngland(stateCode);
}
function xxNEWinNewEngland(stateCode) {
  return ["MA", "CT", "ME", "VT", "NH", "RI"].includes(stateCode);  
}

// step2 変数のインライン化
function inNewEngland(aCustomer) {
  return xxNEWinNewEngland(aCustomer.address.state);
}

// step3 呼び出し箇所を順次置き換え
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
## 変数のカプセル化
https://memberservices.informit.com/my_account/webedition/9780135425664/html/encapsulatevariable.html
### before
```js
let defaultOwner = {firstName: "Martin", lastName: "Fowler"};
```
### after
```js
class Person {
    constructor(data) {
      this._lastName = data.lastName;
      this._firstName = data.firstName
    }
    get lastName() {return this._lastName;}
    get firstName() {return this._firstName;}
```
### 動機
- 関数は新たな名前をつけたり移動したりが簡単だが、データはそうはいかない
- データをカプセル化してメソッド経由でgetとsetを行うことで、意図しない変更や可視性を制御する
### 手順
- 変数を参照・更新するためのカプセル化用関数を作る
- 変数への参照を置き換えながらテストする
- 変数の可視性を制限する
### ポイント
- 元のデータが他で参照されていて変更したくない時は、setterやgetterでコピーを返すこともある
  - コピーの深さに注意
## 変数名の変更
https://memberservices.informit.com/my_account/webedition/9780135425664/html/renamevariable.html
### before
```js
let tpHd = "untitled";
result += `<h1>${tpHd}</h1>`;
tpHd = obj['articleTitle'];
```
### after
```js
// step1 変数のカプセル化を行う
result += `<h1>${title()}</h1>`;
setTitle(obj['articleTitle']);
function title()       {return tpHd;}
function setTitle(arg) {tpHd = arg;}

// step2 変数名を変更する
let _title = "untitled";
function title()       {return _title;}
function setTitle(arg) {_title = arg;}
```
### 動機
- 良い名付けがしたい
  - スコープが極端に狭い場合などは短い変数名も許容
### 手順
- 変数の使用範囲が広い場合は「変数のカプセル化」を検討
- 名前の変更
### ポイント
## パラメータオブジェクトの導入
https://memberservices.informit.com/my_account/webedition/9780135425664/html/introduceparameterobject.html
### before
```js
function amountInvoiced(startDate, endDate) {...}
function amountReceived(startDate, endDate) {...}
function amountOverdue(startDate, endDate) {...}
```
### after
```js
function amountInvoiced(aDateRange) {...}
function amountReceived(aDateRange) {...}
function amountOverdue(aDateRange) {...}
```
### 動機
- データの群れのデータ同士の関連性を示したい
- 関数の引数を少なくしたい
### 手順
- ふさわしい構造体(多くの場合クラス)を作成する
  - 構造体があたいオブジェクトなのか確認すると良い
- パラメータとしてデータの群を渡している関数を順次置き換えていく
### ポイント
- 新たなクラスとして定義して、更新系のメソッドは生やさない
  - 多くの場合値オブジェクトになる
  - 便利な振る舞いをメソッドに生やしていける(同値判定メソッドなど)
## 関数群のクラスへの集約
https://memberservices.informit.com/my_account/webedition/9780135425664/html/combinefunctionsintoclass.html
### before
```js
function base(aReading) {...}
function taxableCharge(aReading) {...}
function calculateBaseCharge(aReading) {...}
```
### after
```js
class Reading {
  base() {...}
  taxableCharge() {...}
  calculateBaseCharge() {...}
}
```
### 動機
- 共通のデータに対して互いに関わりの深い処理を行う一群の関数をまとめたい
### 手順
- 関数間で共有しているデータをまとめるレコードを作り(パラメータオブジェクトの導入)、新しく作ったクラスのインスタンス変数にする
- 共通レコードを扱う関数群をそのクラスのメソッドにする
### ポイント
- もう一つの方法として「関数群への変換への集約」がある
## 関数群の変換への集約
https://memberservices.informit.com/my_account/webedition/9780135425664/html/combinefunctionsintotransform.html
### before
```js
function base(aReading) {...}
function taxableCharge(aReading) {...}
```
### after
```js
function enrichReading(argReading) {
  const aReading = _.cloneDeep(argReading);
  aReading.baseCharge = base(aReading);
  aReading.taxableCharge = taxableCharge(aReading);
  return aReading;
}
```
### 動機
- なんらかの値の変換を一箇所にまとめたい
### 手順
- 引数(レコード)のディープコピーを返す関数を作る
- レコードのプロパティを変換するロジックを、その関数のプロパティに追加する
### ポイント
- 元のレコードの持つ値が途中で変化する可能性があるなら「関数群のクラスへの集約」を行ったほうが良い
## フェーズの分離
https://memberservices.informit.com/my_account/webedition/9780135425664/html/splitphase.html
### before
```js
function priceOrder(product, quantity, shippingMethod) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0)
          * product.basePrice * product.discountRate;
  const shippingPerCase = (basePrice > shippingMethod.discountThreshold)
          ? shippingMethod.discountedFee : shippingMethod.feePerCase; 
  const shippingCost = quantity * shippingPerCase;
  const price =  basePrice - discount + shippingCost;
  return price;
}
```
### after
```js
function priceOrder(product, quantity, shippingMethod) {
  const priceData = calculatePricingData(product, quantity);
  return applyShipping(priceData, shippingMethod);
}
  
function calculatePricingData(product, quantity) {
  const basePrice = product.basePrice * quantity;
  const discount = Math.max(quantity - product.discountThreshold, 0)
          * product.basePrice * product.discountRate;
  return {basePrice: basePrice, quantity: quantity, discount:discount};
}
function applyShipping(priceData, shippingMethod) {
  const shippingPerCase = (priceData.basePrice > shippingMethod.discountThreshold)
          ? shippingMethod.discountedFee : shippingMethod.feePerCase; 
  const shippingCost = priceData.quantity * shippingPerCase;
  return priceData.basePrice - priceData.discount + shippingCost;
}
```
### 動機
- 一つの関数が二つの処理(データの整形と、整形済みデータを使った計算)を行っていて分離したい
### 手順
- 後半フェーズのコードを関数として抽出し、引数として空の中間データ構造を導入する
- 空の中間データ構造に必要な引数を追加していく
- 中間データ構造を返す関数を前半に実装する
### ポイント
- リンクに安全にリファクタリングするための方法が載っている
# 第７章 カプセル化
## レコードのカプセル化
https://memberservices.informit.com/my_account/webedition/9780135425664/html/encapsulaterecord.html
### before
```js
organization = {name: "Acme Gooseberries", country: "GB"};
```
### after
```js
class Organization {
  constructor(data) {
    this._name = data.name; // プロパティにアンスコつけるのは古い習慣
    this._country = data.country;
  }
  get name()    {return this._name;}
  set name(arg) {this._name = arg;}
  get country()    {return this._country;}
  set country(arg) {this._country = arg;}
}
```
### 動機
- レコード構造(値がそのまま入っている)のデータ自体は隠蔽し、取り出したい計算結果だけを公開したい
### 手順
- 移行的手段はリンク参照
### ポイント
- 入れ子になったjsonを扱う際にも有効
## コレクションのカプセル化
https://memberservices.informit.com/my_account/webedition/9780135425664/html/encapsulatecollection.html
### before
```js
class Person {              
  get courses() {return this._courses;}
  set courses(aList) {this._courses = aList;}
```
### after
```js
lass Person {
  get courses() {return this._courses.slice();}
  addCourse(aCourse)    { ... }
  removeCourse(aCourse) { ... }
```
### 動機
- カプセル化していても、getterでコレクション(配列)を返すと、そのクラスを経由せずにインスタンス変数が変更できてしまうのを防ぎたい
### 手順
- getterでは配列をそのまま返すのではなく、配列のコピーを返す
- 配列に要素を追加するメソッドと削除するメソッドを作成する
  - 上記でsetterは不要になるはずだが、必要であれば引数をコピーしてからセットするようにする
### ポイント
- 配列を操作するとコピーを返す言語もあるが、そうでない場合は注意が必要
## オブジェクトによるプリミティブの置き換え
https://memberservices.informit.com/my_account/webedition/9780135425664/html/replaceprimitivewithobject.html
### before
```js
orders.filter(o => "high" === o.priority
                || "rush" === o.priority);
```
### after
```js
orders.filter(o => o.priority.higherThan(new Priority("normal")))
```
### 動機
- プリミティブな値を値オブジェクトっぽく扱いたい
### 手順
- データのための単純な値クラスを作成する
### ポイント
- getterはtoStringとかにするのも良い
- 新たなオブジェクトが参照オブジェクトなのか値オブジェクトなのか検討(リンク参照)
## 問い合わせによる一時変数の置き換え

### before
```js

```
### after
```js

```
### 動機
### 手順
### ポイント