---
title: "DataTables"
excerpt: "DataTables"
categories:
    - javascript
tags:
    - DataTables
last_modified_at: 2021-9-2T00:00:00+09:00
toc: true
toc_sticky: true
---

## excel download button
DataTables에서 버튼 확장 library를 `import`하여 Excel Download 버튼을 추가할 경우. 제대로 동작하지 않았습니다.  
`jszip` library를 import만으로 해결되지 않았습니다. `buttons.html5.js`에서 찾을 수 있게 `$.fn.dataTable.Buttons.jszip(JSZip);` 설정 해줘서 정상 처리 되었습니다.
```js
import 'datatables.net-dt/css/jquery.dataTables.min.css'
import 'datatables.net-scroller-dt/css/scroller.dataTables.min.css'
import 'datatables.net-buttons-dt/css/buttons.dataTables.min.css'

import $ from 'jquery'
import 'datatables.net'
import 'datatables.net/js/jquery.dataTables.min'
import * as JSZip from 'jszip'
import 'datatables.net-scroller/js/dataTables.scroller.min'
import 'datatables.net-buttons/js/dataTables.buttons.min'
import 'datatables.net-buttons/js/buttons.html5.min'

$.fn.dataTable.Buttons.jszip(JSZip);
```

## button completed event
위에서 `import 'datatables.net-scroller/js/dataTables.scroller.min'` 했던 이유는 7만건이 넘는 데이타를 불러오기 때문에 스크롤을 써야했습니다.  
그런데 7만건이나 되는 데이타를 Excel 다운로드시 오랜 시간이 걸려 사용자에게 알려주기 위한 loading 화면이 필요했습니다.  
엑셀 버튼 클릭시 `buttons.action`으로 이벤트는 잡을 수 있었으나 엑셀 다운로드 완료시 이벤트를 잡을 수 없어 아래 링크로 해결했습니다.  
https://datatables.net/reference/event/buttons-processing
```js
var table = $('#example').DataTable();
var overlay = $('<div class="ui-blocker">Please wait...</div>'
 
table.on( 'buttons-processing', function ( e, indicator ) {
    if ( indicator ) {
        overlay.appendTo('body');
    }
    else {
        overlay.remove();
    }
} );
```
