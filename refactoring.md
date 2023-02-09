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
https://memberservices.informit.com/my_account/webedition/9780135425664/html/replacetempwithquery.html
### before
```js
const basePrice = this._quantity * this._itemPrice;
if (basePrice > 1000)
  return basePrice * 0.95;
else
  return basePrice * 0.98;
```
### after
```js
get basePrice() {this._quantity * this._itemPrice;}

...

if (this.basePrice > 1000)
  return this.basePrice * 0.95;
else
  return this.basePrice * 0.98;
```
### 動機
- クラスの中で、あらかじめ計算された結果を後から参照するような変数を関数に置き換えたい
### 手順
- 置き換えようとしている変数が不変であることを確認(constにしてみて確認)
- 一時変数への代入を関数として抽出する
### ポイント
- 結果が同じになることが重要
## クラスの抽出
https://memberservices.informit.com/my_account/webedition/9780135425664/html/extractclass.html
### before
```js
class Person {
  get officeAreaCode() {return this._officeAreaCode;}
  get officeNumber()   {return this._officeNumber;}
```
### after
```js
class Person {
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  get officeNumber()   {return this._telephoneNumber.number;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  get number()   {return this._number;}
}
```
### 動機
- 巨大になったクラスを分割したい
### 手順
- クラスの中で分割できそうな単位を探す(データの一部が同時に更新されたり、互いに強く依存していたりする部分)
- 新たなクラスを作り、元のクラスのnew時にattributeとして新たなクラスのインスタンスを持つようにする
- 新たな子クラスにメソッドを移していく
### ポイント
- 子クラスは値オブジェクトになることが多い
  - 値オブジェクトとして他のクラスからも使うなら「参照から値への変更」を検討する
## クラスのインライン化
https://memberservices.informit.com/my_account/webedition/9780135425664/html/inlineclass.html
### before
```js
class Person {
  get officeAreaCode() {return this._telephoneNumber.areaCode;}
  get officeNumber()   {return this._telephoneNumber.number;}
}
class TelephoneNumber {
  get areaCode() {return this._areaCode;}
  get number()   {return this._number;}
}
```
### after
```js
class Person {
  get officeAreaCode() {return this._officeAreaCode;}
  get officeNumber()   {return this._officeNumber;}
```
### 動機
- 不要になったクラスを別のクラスの一部にしたい
### 手順
- 畳み込み先のクラスに、不要になったクラスのpublicメソッドを実装し、順次テストしながら移行していく
### ポイント
- 絡み合ったクラスを正しく分割するために、一旦統合して再分割する時にも有効
## 委譲の隠蔽
https://memberservices.informit.com/my_account/webedition/9780135425664/html/hidedelegate.html
### before
```js
manager = aPerson.department.manager;
```
### after
```js
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;}
```
### 動機
- 他のクラスの中身を知りすぎない(メソッドチェーンを長くしない)ために、委譲を隠蔽するラッパークラスのようなものを作りたい
### 手順
- 委譲用メソッドを作ったクラスを呼び出すように、テストしながら順次置換していく
### ポイント
- 他のクラスの中身を知りすぎている(依存している)と、そのクラスが変更された時の被害が甚大
## 仲介人の除去
https://memberservices.informit.com/my_account/webedition/9780135425664/html/removemiddleman.html
### before
```js
manager = aPerson.manager;

class Person {
  get manager() {return this.department.manager;}
```
### after
```js
manager = aPerson.department.manager;
```
### 動機
- 「委譲の隠蔽」の逆
- 全てのメソッドの転送用メソッドを実装するのが煩わしくなった場合
### 手順
- 委譲先のオブジェクトを取得するgetterを作り、隠蔽していた委譲を復活させていく
### ポイント
- 委譲の隠蔽と仲介人の除去が混在していても良い
- 開発者の匙加減
## アルゴリズムの置き換え
https://memberservices.informit.com/my_account/webedition/9780135425664/html/substitutealgorithm.html
### before
```js
function foundPerson(people) {
  for(let i = 0; i < people.length; i++) {
    if (people[i] === "Don") {
      return "Don";
    }
    if (people[i] === "John") {
      return "John";
    }
    if (people[i] === "Kent") {
      return "Kent";
    }
  }
  return "";
}
```
### after
```js
function foundPerson(people) {
  const candidates = ["Don", "John", "Kent"];
  return people.find(p => candidates.includes(p)) || '';
}

```
### 動機
- もっと単純なやり方がある時
### 手順
### ポイント
- メソッドをなるべく分割しておいたほうがやりやすい
# 第８章 特性の移動
## 関数の移動
https://memberservices.informit.com/my_account/webedition/9780135425664/html/movefunction.html
### before
```js
class Account {
  get overdraftCharge() {...}
```
### after
```js
class AccountType {
  get overdraftCharge() {...}
```
### 動機
- 関数をより適したコンテクストの場所(クラス)に移動したい
### 手順
- 移動したい関数のそのコンテキストでの依存対象を洗い出す
- 選択した関数がポリモーフィックかどうか確認する
- 関数を別の名前で新設し、元の関数内から呼び出し、必要であれば元の関数を削除してインライン化する
### ポイント
- 新たなクラスを抽出するところから始まる場合もある
- どこに置けばいいのかの判断が難しい場合は移動する必要性も薄いかも
- 新たな関数に渡すパラメータが多い場合はオブジェクトごと渡す
## フィールドの移動
https://memberservices.informit.com/my_account/webedition/9780135425664/html/movefield.html
### before
```js
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this._discountRate;}
```
### after
```js
class Customer {
  get plan() {return this._plan;}
  get discountRate() {return this.plan.discountRate;}
```
### 動機
- データ構造を是正したい
  - あるレコードを関数に渡す時に必ず別レコードを渡す必要がある
  - レコードを更新するたびに別レコードのあるフィールドも更新している
### 手順
- 移動元のフィールドをカプセル化し、移動しやすくする
- 移行する
### ポイント
- 外部仕様が変わらないよう注意
## ステートメントの関数内への移動
https://memberservices.informit.com/my_account/webedition/9780135425664/html/movestatementsintofunction.html
### before
```js
result.push(`<p>title: ${person.photo.title}</p>`);
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [
    `<p>location: ${aPhoto.location}</p>`,
    `<p>date: ${aPhoto.date.toDateString()}</p>`,
  ];
}
```
### after
```js
result.concat(photoData(person.photo));

function photoData(aPhoto) {
  return [
    `<p>title: ${aPhoto.title}</p>`,
    `<p>location: ${aPhoto.location}</p>`,
    `<p>date: ${aPhoto.date.toDateString()}</p>`,
  ];
}
```
### 動機
- 関数に含めた方が良いステートメントを関数内に移動(することによる重複の削除)
### 手順
- ステートメントを関数内に含めた新たな関数を作る
- 元の関数とステートメントを新たな関数に置き換えながらテスト
- 必要があれば新たな関数の名前を改善
### ポイント
- 共通化した部分に差異が生じたら「ステートメントの呼び出し側への移動」
## ステートメントの呼び出し側への移動
https://memberservices.informit.com/my_account/webedition/9780135425664/html/movestatementstocallers.html
### before
```js
emitPhotoData(outStream, person.photo);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>\n`);
  outStream.write(`<p>location: ${photo.location}</p>\n`);
}
```
### after
```js
emitPhotoData(outStream, person.photo);
outStream.write(`<p>location: ${person.photo.location}</p>\n`);

function emitPhotoData(outStream, photo) {
  outStream.write(`<p>title: ${photo.title}</p>\n`);
}
```
### 動機
- 関数の中で場合分けが発生したり、異質なものが紛れ込んだ
### 手順
- 分離したい部分以外を抜き出して新たな関数を作り、元の関数内で呼ぶ
- 順次置き換え
### ポイント
## 関数呼び出しによるインラインコードの置き換え
https://memberservices.informit.com/my_account/webedition/9780135425664/html/replaceinlinecodewithfunctioncall.html
### before
```js
let appliesToMass = false;
for(const s of states) {
  if (s === "MA") appliesToMass = true;
}
```
### after
```js
appliesToMass = states.includes("MA");
```
### 動機
- この処理もう関数化されてるよって時
### 手順
- インラインコードを既存の関数で置き換え
### ポイント
- 置き換えた時に意味が通らなければ関数名が悪いかも
## ステートメントのスライド
https://memberservices.informit.com/my_account/webedition/9780135425664/html/slidestatements.html
### before
```js
const pricingPlan = retrievePricingPlan();
const order = retreiveOrder();
let charge;
const chargePerUnit = pricingPlan.unit;
```
### after
```js
const pricingPlan = retrievePricingPlan();
const chargePerUnit = pricingPlan.unit;
const order = retreiveOrder();
let charge;
```
### 動機
- 意味的にまとまってるものを場所的にも近くにおきたい
### 手順
- 影響の有無を考慮しながら場所を移動
### ポイント
- 「関数の抽出」などの前準備として行うことが多い
- 副作用のないコードを書いているとやりやすい
  - コマンドとクエリの分離原則(値を返す関数には副作用を与えない)
## ループの分離
https://memberservices.informit.com/my_account/webedition/9780135425664/html/splitloop.html
### before
```js
let averageAge = 0;
let totalSalary = 0;
for (const p of people) {
  averageAge += p.age;
  totalSalary += p.salary;
}
averageAge = averageAge / people.length;
```
### after
```js
let totalSalary = 0;
for (const p of people) {
  totalSalary += p.salary;
}

let averageAge = 0;
for (const p of people) {
  averageAge += p.age;
}
averageAge = averageAge / people.length;
```
### 動機
- 一つのループの中で複数の作業が行われているとメンテが辛い
### 手順
- ループをコピーする
- 重複による副作用を特定し、排除する
### ポイント
- 関数の抽出を続けて行うことが多い
- 性能上のボトルネックになることはほとんどない
## パイプラインによるループの置き換え
https://memberservices.informit.com/my_account/webedition/9780135425664/html/replaceloopwithpipeline.html
### before
```js
const names = [];
for (const i of input) {
  if (i.job === "programmer")
    names.push(i.name);
}
```
### after
```js
const names = input
  .filter(i => i.job === "programmer")
  .map(i => i.name)
;
```
### 動機
- ループよりパイプラインの方が何をしているかがわかりやすい
### 手順
- ループ処理を移し替えていく用の変数を用意し、ループに与える
- ループ処理の中の一つ一つの処理を、順次変数に対するパイプライン処理に置き換えていく
### ポイント
- csv処理とかに使える
- インデントやスペースなどの見た目も整えるとよりきれい
## デッドコードの削除
https://memberservices.informit.com/my_account/webedition/9780135425664/html/removedeadcode.html
### before
```js
if(false) {
  doSomethingThatUsedToMatter();
}
```
### after
```js
 
```
### 動機
- 不要なものは削除
### 手順
### ポイント
- コメントアウトはバージョン管理システム以前の習慣
# 第９章 データの再編成
## 変数の分離
https://memberservices.informit.com/my_account/webedition/9780135425664/html/splitvariable.html
### before
```js
let temp = 2 * (height + width);
console.log(temp);
temp = height * width;
console.log(temp);
```
### after
```js
const perimeter = 2 * (height + width);
console.log(perimeter);
const area = height * width;
console.log(area);
```
### 動機
- 変数が複数の意味で使われていたり、不必要に再代入されたりしている
### 手順
- 宣言時と最初の代入時で変数名を変える
  - 可能であれば新しい変数は変更不可にする
- 変数名を変更していく
### ポイント
- 引数を関数内でミューテーションしている場合はこれを適用するのが良い
## フィールド名の変更
https://memberservices.informit.com/my_account/webedition/9780135425664/html/renamefield.html
### before
```js
class Organization {
  get name() {...}
}
```
### after
```js
class Organization {
  get title() {...}
}
```
### 動機
- 名前重要
### 手順
- レコードのスコープが狭い場合は一括置換で終了
- レコードがカプセル化していなければする
- 旧名称と新名称どちらでも呼べるようにし、順次変更していく
### ポイント
- 途中でテストが失敗するなら慎重な、順次変更可能な手法を検討する
## 問い合わせによる導出変数の置き換え
https://memberservices.informit.com/my_account/webedition/9780135425664/html/replacederivedvariablewithquery.html
### before
```js
get discountedTotal() {return this._discountedTotal;}
set discount(aNumber) {
  const old = this._discount;
  this._discount = aNumber;
  this._discountedTotal += old - aNumber; 
}
```
### after
```js
get discountedTotal() {return this._baseTotal - this._discount;}
set discount(aNumber) {this._discount = aNumber;}
```
### 動機
- 変更可能なデータをできるだけ無くしたい
  - バグの温床
- 導出変数(値が変わった変数を返す)より、新たな値を計算して返す問い合わせ(クエリ)のほうがバグが少ない
### 手順
- 変数の変更を計算する関数を作る
- 変数の参照箇所を関数呼び出しで置き換える
### ポイント
- 計算できるなら毎回計算する(関数型言語っぽく)
## 参照から値への変更
https://memberservices.informit.com/my_account/webedition/9780135425664/html/changereferencetovalue.html
### before
```js
class Product {
  applyDiscount(arg) {this._price.amount -= arg;}
```
### after
```js
class Product {
  applyDiscount(arg) {
    this._price = new Money(this._price.amount - arg, this._price.currency);
  }
```
### 動機
- フィールドが値オブジェクトを持つようにしたい
  - 値オブジェクトは変更されないので仕様の把握が容易
  - 分散システムや並行システムで重要
### 手順
- 各setterに対して「setterの削除」を行う
- 値オブジェクトとして正しく同値比較ができるようにする(rubyなら==メソッドをオーバーライドするなど)
### ポイント
- あるオブジェクトの変更を他の場所でも共有したいなら値オブジェクトは使わない
## 値から参照への変更
https://memberservices.informit.com/my_account/webedition/9780135425664/html/changevaluetoreference.html
### before
```js
let customer = new Customer(customerData);
```
### after
```js
let customer = customerRepository.get(customerData.id);
```
### 動機
- あるオブジェクトの変更を他の場所でも共有したい
### 手順
- 参照オブジェクトのインスタンス用のリポジトリを作成する
- リポジトリを使用して参照オブジェクトを取得するようコンストラクタを変更する
### ポイント
- 依存関係が心配ならリポジトリをコンストラクタのパラメータとして渡しても良い
# 第１０章 条件記述の単純化
## 条件記述の分解
https://memberservices.informit.com/my_account/webedition/9780135425664/html/decomposeconditional.html
### before
```js
if (!aDate.isBefore(plan.summerStart) && !aDate.isAfter(plan.summerEnd))
  charge = quantity * plan.summerRate;
else
  charge = quantity * plan.regularRate + plan.regularServiceCharge;
```
### after
```js
if (summer())
  charge = summerCharge();
else
  charge = regularCharge();
```
### 動機
- 条件分岐は「何」をしているかはわかっても「なぜ」そうしているかがわかりにくい
- 条件分岐を分解して関数に置き換え、名付けをすることで意図が明確になる
### 手順
- 各条件に「関数の抽出」を適用する
### ポイント
## 条件記述の統合
https://memberservices.informit.com/my_account/webedition/9780135425664/html/decomposeconditional.html
### before
```js
if (anEmployee.seniority < 2) return 0;
if (anEmployee.monthsDisabled > 12) return 0;
if (anEmployee.isPartTime) return 0;
```
### after
```js
if (isNotEligableForDisability()) return 0;

function isNotEligableForDisability() {
  return ((anEmployee.seniority < 2)
          || (anEmployee.monthsDisabled > 12)
          || (anEmployee.isPartTime));
}
```
### 動機
- 複数の条件分岐を行っている箇所で、何を判定しているのか明確にしたい
### 手順
- いずれの条件判定にも副作用がないことを確認する
  - 副作用がある場合は「問い合わせと更新の分離」を行う
- 条件判定を論理演算子を用いて結合していき、一つになった時点で関数として抽出する
### ポイント
- それぞれが独立した条件分岐であれば無理に統合する必要はない
## ガード節による入れ子の条件記述の置き換え
https://memberservices.informit.com/my_account/webedition/9780135425664/html/replacenestedconditionalwithguardclauses.html
### before
```js
function getPayAmount() {
  let result;
  if (isDead)
    result = deadAmount();
  else {
    if (isSeparated)
      result = separatedAmount();
    else {
      if (isRetired)
        result = retiredAmount();
      else
        result = normalPayAmount();
    }
  }
  return result;
}
```
### after
```js
function getPayAmount() {
  if (isDead) return deadAmount();
  if (isSeparated) return separatedAmount();
  if (isRetired) return retiredAmount();
  return normalPayAmount();
}
```
### 動機
- 主要な処理ではないことを明示して、条件分岐にウェイトの差をつける
### 手順
- 置き換えるべき条件で最も外側のものから順次ガード節に変更していく
### ポイント
- if と elseを使う時はそれぞれの条件が等価な時
- 例外処理と正常処理なら、例外処理の条件だけでアーリーリターンしてしまった方が良い
## ポリモーフィズムによる条件記述の置き換え

### before
```js
switch (bird.type) {
  case 'EuropeanSwallow':
    return "average";
  case 'AfricanSwallow':
    return (bird.numberOfCoconuts > 2) ? "tired" : "average";
  case 'NorwegianBlueParrot':
    return (bird.voltage > 100) ? "scorched" : "beautiful";
  default:
    return "unknown";
```
### after
```js
class EuropeanSwallow {
  get plumage() {
    return "average";
  }
class AfricanSwallow {
  get plumage() {
     return (this.numberOfCoconuts > 2) ? "tired" : "average";
  }
class NorwegianBlueParrot {
  get plumage() {
     return (this.voltage > 100) ? "scorched" : "beautiful";
  }
```
### 動機
- 難解な条件分岐を上位概念に変換したい
### 手順
- リンク参照
### ポイント
- スーパークラスに条件を抽出したりも可能
- 単純なifやswitchが悪というわけではない
## 特殊ケース(nullオブジェクト)の導入
https://memberservices.informit.com/my_account/webedition/9780135425664/html/introducespecialcase.html
### before
```js
if (aCustomer === "unknown") customerName = "occupant";
```
### after
```js
class UnknownCustomer {
    get name() {return "occupant";}
```
### 動機
- 特殊なケースに毎回対応しているコードがあちこちにある時に、特殊ケースとして分離してそこに振る舞いをまとめたい
- 典型的なケースで言うとnullオブジェクト
### 手順
- リンク参照
### ポイント
- `if customer === "unknown"`を一気に特殊オブジェクトへの問い合わせに置き換えるのはしんどい
- 「関数の抽出」で`if customer === "unknown"`を関数として抽出してしまえば順次変更できる
## アサーションの導入
https://memberservices.informit.com/my_account/webedition/9780135425664/html/introduceassertion.html
### before
```js
if (this.discountRate)
  base = base - (this.discountRate * base);
```
### after
```js
assert(this.discountRate >= 0);
if (this.discountRate)
  base = base - (this.discountRate * base);
```
### 動機
- このコードはこれを前提にしていますよ、と言うことをコードとして明示できる
- エラー発見にも役立つが、本来コミュニュケーションのためなので、理想としてはアサーションが一切なくても動く状態が好ましい
### 手順
- ある条件が真であることを前提にできる場合にアサーションを書く
### ポイント
- 書く必要があるのは「真であることすべて」ではなく「真である必要がある」こと
# 第１１章　APIのリファクタリング
## 問い合わせと更新の分離
https://memberservices.informit.com/my_account/webedition/9780135425664/html/separatequeryfrommodifier.html
### before
```js
function getTotalOutstandingAndSendBill() {
  const result = customer.invoices.reduce((total, each) => each.amount + total, 0);
  sendBill();
  return result;
}
```
### after
```js
function totalOutstanding() {
  return customer.invoices.reduce((total, each) => each.amount + total, 0);  
}
function sendBill() {
  emailGateway.send(formatBill(customer));
}
```
### 動機
- 副作用のある関数とない関数を分離したい
  - 冪等性、テスト容易性などメリット多数
  - 「コマンドとクエリの分離原則」=> 値を返す関数は観察可能な副作用を持ってはならない
### 手順
- 関数をコピーして問い合わせ用に名付け、副作用を除去する
- 元の関数の呼び出しについて戻り値を使っている場合は新たな関数呼び出しに置き換え、元の関数をその後ろで呼び出す
- 元の関数から戻り値を削除する
### ポイント
## パラメータによる関数の統合
https://memberservices.informit.com/my_account/webedition/9780135425664/html/parameterizefunction.html
### before
```js
function tenPercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.1);
}
function fivePercentRaise(aPerson) {
  aPerson.salary = aPerson.salary.multiply(1.05);
}
```
### after
```js
function raise(aPerson, factor) {
  aPerson.salary = aPerson.salary.multiply(1 + factor);
}
```
### 動機
- リテラル値が異なるだけのよく似たロジックを持つ関数の重複を削除したい
### 手順
- 関数のリテラル値をパラメータに変更する
- 類似する関数呼び出しを置き換える
### ポイント
- ロジックの一部(最大値と最小値をパラメータにとって間にあることを判定するなど)を切り出すのも有効
## フラグパターンの削除
https://memberservices.informit.com/my_account/webedition/9780135425664/html/removeflagargument.html
### before
```js
function setDimension(name, value) {
  if (name === "height") {
    this._height = value;
    return;
  }
  if (name === "width") {
    this._width = value;
    return;
  }
}
```
### after
```js
function setHeight(value) {this._height = value;}
function setWidth (value) {this._width = value;}
```
### 動機
- 関数内でのフラグの条件分岐は中身が読み取りにくい 
### 手順
- フラグで分岐している処理を別々の関数として切り出す
- フラグの使われ方が複雑な場合はラップ関数を作る
### ポイント
- 関数のパラメータデフラグをわたすデメリット
  - どの関数呼び出しが使えてどう呼び出せばいいかの理解が難しくなる
  - 利用可能な関数呼び出しの多様性を隠してしまう
- 複数のフラグを引数に取るならその組み合わせで処理を変更する関数の意味はある
  - 全ての組み合わせを関数化すると大量になるため
## オブジェクトそのものの呼び出し
https://memberservices.informit.com/my_account/webedition/9780135425664/html/preservewholeobject.html
### before
```js
const low = aRoom.daysTempRange.low;
const high = aRoom.daysTempRange.high;
if (aPlan.withinRange(low, high))
```
### after
```js
if (aPlan.withinRange(aRoom.daysTempRange))
```
### 動機
- 引数の数を減らしたい
### 手順
- 引数の少ない新しい関数を作って変更していく
- 最後に古い関数を削除して新しい関数の名前を戻す
### ポイント
- 依存関係を増やしたくない時(Moduleをまたいでオブジェクトを渡すなど)
  - あるオブジェクトからいくつか値を取り出すロジックをオブジェクト側に移す
  - 同じデータの群れが頻出するなら「パラメータオブジェクトの導入」を適用
  - あるオブジェクトの部分集合だけ扱うことがよくあるなら「クラスの抽出」