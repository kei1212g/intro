# 開発中の クラスライブラリ の紹介
- [DataProcessor クラス](#dataprocessor-クラス)
  - [簡易説明](#簡易説明)
  - [今後の新機能追加予定](#今後の新機能追加予定)
  - [メソッド: constructor(inputJsonData)](#メソッド-constructorinputjsondata)
  - [メソッド: run()](#メソッド-run)
  - [メソッド: getContent(inputJsonData)](#メソッド-getcontentinputjsondata)
  - [メソッド: getMultipleElements(targetName, targetJson)](#メソッド-getmultipleelementstargetname-targetjson)
  - [メソッド: getLinePropertyData(targetElement, necessaryTargetInputJson)](#メソッド-getlinepropertydatatargetelement-necessarytargetinputjson)
  - [メソッド: resetHTML()](#メソッド-resethtml)
  - [メソッド: putData(outputJsonData)](#メソッド-putdataoutputjsondata)
  - [メソッド: getCircularReplacer()](#メソッド-getcircularreplacer)
  - [このクラスの全体の定義](#このクラスの全体の定義)
  - [例を実行した、出力例](#例を実行した出力例)
  - [実行例](#実行例)

## 簡易説明
- 指定されたターゲットのデータを処理するためのデータ処理プログラムを提供します。  
- WEBスクレイピングパターンを最適化できるのフレームワーク
以下に各メソッドの詳細を説明します。

## 今後の新機能追加予定
- 7月11日 option追加で、対象のプロパティから取得したデータの前に文字列データをデータを追加できるようにする
→次につながるようにしっかり加工をする
- 7月12日 option追加機能をさらに、後ろへの文字列追加、置き換え機能の追加


## メソッド: constructor(inputJsonData)

DataProcessor クラスの新しいインスタンスを初期化します。
```javascript
 //コンストラクタ
  constructor(inputJsonData) {
    this.inputJsonData = inputJsonData;
    this.outputJsonData = {};
  }
```

引数:
- inputJsonData: ターゲットのデータを指定した JSON 形式のオブジェクト。
```javascript
const inputJsonData = {
  "video_list": {
    "selector_all": "ytd-rich-grid-row",
    "inner": {
      "title": "h3",
      "link": "a"
    }
  },
  "title":"h3"
};
```

selector_allとinnerキーを検出したとき、selector_allプロパティですべての要素取得し、innerプロパティのテンプレートの沿って出力できるように設計されている。

## メソッド: run()

データ処理プログラムを実行し、処理結果の JSON データを返します。  
```javascript
run() {
    this.outputJsonData = this.getContent(this.inputJsonData);
    this.resetHTML();
    this.putData(this.outputJsonData);
    return this.outputJsonData;
  }
```

パロメータ

- なし。

戻り値:

- 処理結果の JSON データ。

## メソッド: getContent(inputJsonData)

ターゲットのデータのコンテンツを取得します。指定されたターゲットのデータを処理し、コンテンツの JSON データを返します。

```javascript
  getContent(inputJsonData) {
    const outputJsonData = this.getLinePropertyData(document, inputJsonData);
    return outputJsonData;
  }
```

引数

- inputJsonData: ターゲットのデータを指定した JSON 形式のオブジェクト。

戻り値:

- 取得したコンテンツの JSON データ。

## メソッド: getMultipleElements(targetName, targetJson)

複数の要素を取得します。指定されたターゲットのプロパティに基づいて、複数の要素を取得し、それぞれの要素のプロパティを含む JSON データを返します。

```javascript
  getMultipleElements(targetName, targetJson) {
    const selectorAllList = document.querySelectorAll(targetJson[targetName].selector_all);
    const multipleOutputJsonData = Array.from(selectorAllList).map((targetElement, count) => {
      const multipleLayerJsonData = {};
      const lineJsonData = this.getLinePropertyData(targetElement, targetJson[targetName].inner);
      multipleLayerJsonData[targetName] = lineJsonData;
      return multipleLayerJsonData;
    });
    return multipleOutputJsonData;
  }
```

引数:

- targetName: ターゲットのプロパティ名。
- targetJson: ターゲットのプロパティを含んだ JSON 形式のオブジェクト。

戻り値:

- 取得した複数の要素の JSON データ。

## メソッド: getLinePropertyData(targetElement, necessaryTargetInputJson)

指定された要素と必要なプロパティの情報を使用して、1 層分の処理を行います。指定された要素のプロパティを取得し、それぞれのプロパティの値を含む JSON データを返します。

```javascript

  getLinePropertyData(targetElement, necessaryTargetInputJson) {
    const linePropertyOutputData = {};

    Object.entries(necessaryTargetInputJson).forEach(([keyName, valueOfKey]) => {
      if (typeof valueOfKey === "object") {
        const res = this.getMultipleElements(keyName, necessaryTargetInputJson);
        linePropertyOutputData[keyName] = res;
        return;
      }

      if (valueOfKey === "a") {
        linePropertyOutputData[keyName] = targetElement.querySelector(valueOfKey)?.getAttribute("href");
      } else {
        linePropertyOutputData[keyName] = targetElement.querySelector(valueOfKey)?.textContent;
      }
    });

    return linePropertyOutputData;
  }

```

引数:

- targetElement: 処理対象の要素。
- necessaryTargetInputJson: 処理に必要なプロパティを含んだ JSON 形式のオブジェクト。

戻り値:

- 1 層分の処理結果の JSON データ。

## メソッド: resetHTML()

HTML をリセットし、要素を削除します。

```javascript
resetHTML() {
    const html = document.querySelector("html");
    while (html.firstChild) {
      html.firstChild.remove();
    }
  }
```

引数:

- なし。

戻り値:

- なし。

## メソッド: putData(outputJsonData)

指定された処理結果の JSON データを表示します。
```javascript
 putData(outputJsonData) {
    const html = document.querySelector("html");
    html.textContent = JSON.stringify(outputJsonData, this.getCircularReplacer());
    return JSON.stringify(outputJsonData, this.getCircularReplacer());
  }

```

引数:

- outputJsonData: 処理結果の JSON データ。

戻り値:

- 処理結果の JSON データを文字列に変換したもの。

## メソッド: getCircularReplacer()

JSON の循環参照を回避するための関数を返します。

```javascript

  getCircularReplacer() {
    const seen = new WeakSet();
    return (key, value) => {
      if (typeof value === "object" && value !== null) {
        if (seen.has(value)) {
          return;
        }
        seen.add(value);
      }
      return value;
    };
  }
```
引数:

- なし。

戻り値:

- JSON の循環参照を回避するための関数。

## このクラスの全体の定義

使い方の例:
## 例を実行した、出力例
```json
{
  "video_list": [
    {
      "video_list": {
        "title": "Video Title 1",
        "link": "https://example.com/video1"
      }
    },
    {
      "video_list": {
        "title": "Video Title 2",
        "link": "https://example.com/video2"
      }
    },
    ...
  ],
  "title": "Main Title"
}
```
## 実行例
``` javascript 
//実行
const dataProcessor = new DataProcessor(inputJsonData);
dataProcessor.run();
```

上記の例では、以下の手順で DataProcessor クラスを使用してデータ処理プログラムを実行します。

1. `inputJsonData` にターゲットのデータを指定します。この例では、動画リストの取得に関する情報を指定しています。
2. `youtubeVideoChAbout` にチャンネル情報の取得に関する情報を指定します。
3. DataProcessor クラスの新しいインスタンスを作成し、`youtubeVideoChAbout` をコンストラクタに渡します。
4. `run` メソッドを呼び出してデータ処理プログラムを実行し、処理結果を `result` 変数に格納します。
5. `result` をコンソールに出力します。

これにより、DataProcessor クラスを使用して指定したターゲットのデータを処理し、結果を取得することができます。
