---
title: "Handsontable 6.2.2"
excerpt: "web에서 excel grid 기능 구현시 다양한 도구들이 존재합니다. 그 중 MIT라이센스로 무료로 사용하기 괜찮은 도구가 Handsontable이 아닐까 생각합니다.  
물론 Handsontable도 상업적 이용시 라이센스 제약이 따릅니다만. 6.2.2(2018.12.19) 버젼까지는 MIT 라이센스가 적용되어 애용하는 편입니다.  
물론 릴리즈 날짜가 2018년 12월인 만큼 기능적으로 최신버전에 비해 미흡한 부분들이 있는건 어쩔 수 없네요.  "
categories:
    - javascript
tags:
    - Handsontable
last_modified_at: 2021-10-6T00:00:00+09:00
toc: true
toc_sticky: true
---


# Handsontable 소개
web에서 excel grid 기능 구현시 다양한 도구들이 존재합니다. 그 중 MIT라이센스로 무료로 사용하기 괜찮은 도구가 Handsontable이 아닐까 생각합니다.  
물론 Handsontable도 상업적 이용시 라이센스 제약이 따릅니다만. 6.2.2(2018.12.19) 버젼까지는 MIT 라이센스가 적용되어 애용하는 편입니다.  
물론 릴리즈 날짜가 2018년 12월에 PRO가 아닌 Community Edition 버전인 만큼 기능적으로 최신버전에 비해 미흡한 부분들이 있는건 어쩔 수 없습니다.  
  
Link : [Handsontable 6.2.2 doc](https://handsontable.com/docs/6.2.2/tutorial-introduction.html)

---

# 기본 사용법

## Quick start

```js
const data = [
  ["Year", "Ford", "Tesla", "Hyundai", "KIA"],
  ["2017", 10, 11, 12, 13],
  ["2018", 20, 11, 14, 13],
  ["2019", 30, 15, 12, 13]
];

const container = document.getElementById('example');
const hot = new Handsontable(container, {
  data: data,
  rowHeaders: true,
  colHeaders: true
});
```
Link : <https://jsfiddle.net/moregorenine/4Lnpqdfw/7/>

## Data binding
Data를 binding 할 경우 그리드에 입력된 모든 데이터는 원래 데이터 소스를 변경합니다.  
복잡한 응용 프로그램에서는 Handsontable 외부에서 데이터 소스를 변경할 수 있습니다.  
다만 `render() 메서드`를 사용하여 화면의 그리드를 갱신하지 않으면 변경 사항이 화면에 표시되지 않습니다.  
```js
const data = [
    ['Year', 'Tesla', 'Nissan', 'Hyundai', 'KIA', 'Audi', 'Ford'],
    ['2017', 10, 11, 12, 13, 15, 16],
    ['2018', 10, 11, 12, 13, 15, 16],
    ['2019', 10, 11, 12, 13, 15, 16],
    ['2020', 10, 11, 12, 13, 15, 16],
    ['2021', 10, 11, 12, 13, 15, 16]
  ],
  container = document.getElementById('example'),
  settings = {
    data: data
  };

const hot = new Handsontable(container, settings);
data[0][1] = 'BMW'; // change "Tesla" to "BMW" programmatically
hot.render();
```

Link : <https://jsfiddle.net/moregorenine/vbm2qxt0/2/>

## Data sources
Data를 바인딩 시키는 방법은 여러가지가 존재합니다. 그 중 실무에서 많이 쓰이는 방법은 JSON 행태의 list 자료형이 아닐까 합니다.  
```js
const nestedObjects = [
    {id: 1, name: {first: "Ted", last: "Right"}, address: ""},
    {id: 2, address: ""}, // HOT will create missing properties on demand
    {id: 3, name: {first: "Joan", last: "Well"}, address: ""}
  ];
const container = document.getElementById('example');

const hot = new Handsontable(container, {
  data: nestedObjects,
  colHeaders: true,
  columns: [
    {data: 'id'},
    {data: 'name.first'},
    {data: 'name.last'},
    {data: 'address'}
  ],
  minSpareRows: 1
});

nestedObjects[0].name.first = 'moregorenine'
hot.render();
```

Link : <https://jsfiddle.net/moregorenine/17osza2h/4/>

## Load and save
`afterChange` hook을 사용해 evenet 발생시 추적하여 save 로직과 연결하는 방법 및 `로컬 persistentState`를 이용하는 방법등을 소개합니다만  
전 엑셀 붙여 넣기 등 실무에서 editing event 발생시 마다 캐치하여 save하는 로직을 별로 선호하지 않고 biding된 data를 그냥 server에 통채로 던져서 처리하는 방식을 좋아해서 pass

Link : <https://handsontable.com/docs/6.2.2/tutorial-load-and-save.html>

## Setting options
`cell`은 특정 셀(행/열 조합)에 대해 생성자 또는 열 옵션을 덮어쓸 수 있습니다 . 
```js
var hot = new Handsontable(document.getElementById('example'), {
  cell: [
    {row: 0, col: 0, readOnly: true}
  ]
});
```
`cells`은 cell 갯수만큼 row, col, prop 인자를 넘겨받는 함수가 호출되고 cell해 해당하는 cellProperties 객체를 넘겨받습니다.
```js
var hot = new Handsontable(document.getElementById('example'), {
  cells: function (row, col, prop) {
    var cellProperties = {}

    if (row === 0 && col === 0) {
      cellProperties.readOnly = true;
    }

    return cellProperties;
  }
})
```
`columns`은 각 열 전체에 해당하는 설정이 가능합니다. 아래는 코드는 계단식 구성을 보여줍니다.  
우선 gird 전체는 `readOnly`에서 읽기 전용으로 설정했습니다.  
다음 `columns`에서 첫번째 열을 수정 가능으로 설정했습니다.  
다음 `cells`에서 첫 행, 첫 열을 읽기 전용으로 설정했습니다.  
이렇게 위에서 아래로 계단식 설정이 가능합니다.
```js
var hot = new Handsontable(document.getElementById('example'), {
  readOnly: true,
  columns: [
    {readOnly: false},
    {},
    {}
  ],
  cells: function (row, col, prop) {
    var cellProperties = {}

    if (row === 0 && col === 0) {
      cellProperties.readOnly = true;
    }

    return cellProperties;
  }
});
```

## Using callbacks
handsontable에 존재하는 event에 대한 callback함수들을 보여줍니다.  
  
Link : <https://handsontable.com/docs/6.2.2/tutorial-using-callbacks.html>

# Developer guide

## Cell types
handsontable cell에서 데이타 표현에 지원되는 type은 아래와 같습니다.   
`autocomplete, checkbox, date, dropdown, handsontable, numeric, password, text, time`  
이중 selectbox 형태의 데이타는 `autocomplete, dropdown`을 사용하는데 문제는 6.2.2 버전에서는 보통 selectbox에서 제공하는 key, value 형태를 지원하지 않습니다.  
npm에서 제공되는 plugin이 존재하지만 6.2.2 version에는 호환이 되지않아 degrade하여 npm에 등록했습니다. npm 주소는 아래 링크입니다.  
  
Link : [handsontable 6.2.2 cell type autocomplete key, value](https://www.npmjs.com/package/handsontable-6.2.2-celltype-key-value)

## Cell editor
handsontable은 셀 값을 표시하는 과정과 값을 변경하는 과정을 구분합니다.  
`Renderers`는 데이터를 HTML 코드로 반환하여 grid에 표현하고  
`Editors`는 사용자 입력(마우스 및 키보드 이벤트)을 처리하고 데이터를 검증하고 검증 결과에 따라 데이터를 변경합니다.