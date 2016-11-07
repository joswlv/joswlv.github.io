---
layout: post
title: jquery :last 선택자
date: 2016-11-07 08:04
categories: js
---

```html
<table id = "table1"> ---</table>
<table id = "table2"> ---</table>
<table id = "table3"> ---</table>
<table id = "table4"> 
	<tr>---</tr>
	<tr>
		<td>asd<td>
		<td><input class="inputs"></td>
	</tr>
</table>
```

위와 같은 환경이 있고 `#table4`는 파일을 추가 할때마다 `tr`이 추가되는 상황이라고 하자!

(추가 될때는 모든 속성이 같다.)

#### 이때 가장 마지막에 생성된 `#inputs`에 접근할려면... 

`$('#table4 tr:last .inputs').val();`

이런 식으로 `:last`선택자를 사용하면 된다.


### Reference

* [jqueryDoc](http://api.jquery.com/last-selector/)