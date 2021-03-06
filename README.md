# Web-dev by using jsp
<br>
2019-12-20
<br>

## To Do

1. Dept, Emp 웹페이지 구현해보기
2. ~~Crawling and Save bitcoin historical data in DB.~~
~~select option을 통해 날짜 범위를 받으면 그 해당하는 부분만 테이블에 뿌리기~~    
3. 뿌려진 데이터 차트로 표현해보기  
4. 코인 별로 만들어보기  
5. DB저장방법(여러번 크롤링을 돌리도록 메서드화 할건지), Update방법(최신데이터를 전체크롤링없이 부분만 하도록 해보자)
6. 기타 세부사항 수정하기(디자인적 요소, 날짜 에러잡기)
<br>

## Report

<b>1) <%@ include ... %> 유의하면서 코드작성하기</b>  
include를 통해 header, footer를 분리했을시 jQuery 연동은 footer에 있기 때문에
script를 통해 jQuery를 이용할시 footer include한 곳보다 아래에서 작업해야 한다. (script작업은 무조건 맨 밑에서 하는 것이 좋다.)

<b>2) Crawling 데이터 개수 제한</b>  
[(크롤링 대상: 비트코인 과거데이터)](https://coinmarketcap.com/currencies/bitcoin/historical-data/)  
Crawling을 하는데 있어서 DB에 저장까지 문제없이 잘되다가 어느순간 IndexOutOfBoundsException 예외상황이 발생  
많은 자료를 한꺼번에 크롤링해서 그런건지 아니면 무슨 오류가 있는 건지 (웹사이트에서 막아둔 건지는 모르나 대용량으로 많은 데이터를 한꺼번에 크롤링하는 것이 막힘, 길어봐야 2년정도의 데이터정도 가능한듯)  
<b> → 해결: 2년씩 크롤링 하면 전체 데이터 크롤링 가능</b>  
```
int cYear = 2019; // 2019년 기준(원하는 날짜로 해서 삽입가능)
while (cYear >= 2013) {
	StringBuffer startDate = new StringBuffer();
	StringBuffer endDate = new StringBuffer();

	endDate.append(cYear);
	endDate.append("12");
	endDate.append("19");
	cYear -= 2;
	startDate.append(cYear);
	startDate.append("12");
	startDate.append("19");

	String url = "https://coinmarketcap.com/currencies/bitcoin/historical-data/?start="
		+ startDate.toString()
		+ "&end=" 
		+ endDate.toString();

	Document doc = null;

	try {
		// 대상 데이터 크롤링하기
		}

	} catch (IOException e) {
		e.printStackTrace();
	}
}
```

<b>3) 새로고침시 alert 알림표시(아직 해결 못함)</b>  

<b>4) url parameter 줄이기</b>  
localhost/crawling/list.jsp?page=1&start=20140711&end=20191219 이런식으로 parameter를 축약함. 원래는 start의 연,월,일따로 end의 연,월,일 따로 parameter지정해서 넘기려고 하니 url이 지저분해짐 → 하나로 묶어서 parameter로 보냄  
(select option으로 넘겼기 때문에 parameter로 start, end의 연, 월, 일 따로 지정돼있음. post방식으로 넘겨서 url에는 보이지 않음.)  

<b>5) date형 변수 받아서 형식 바꾸기</b>  
크롤링 대상 사이트는 거래날짜가 "DEC 20, 2019" 이런식으로 나옴. 이걸 크롤링으로 받게 되면 String형식 되어버림  
**"DEC 20, 2019" > "2019-12-20"** 날짜형식 바꿔서 DB에 저장하기  
**[java]SimpleDateFormat Class**를 이용하자  
```
String changeDate = null;
SimpleDateFormat in = new SimpleDateFormat("MMM dd, yyyy", Locale.US);
SimpleDateFormat out = new SimpleDateFormat("yyyy-MM-dd");
try {
	// String => Date
	// 지정한 Date Form대로 y,m,d 나눠서 Date로 parsing하겠다.
	Date usDate = in.parse(us);
			
	// Date => String
	changeDate = out.format(usDate);
} catch (ParseException e) {
	// TODO Auto-generated catch block
	e.printStackTrace();
}
```

<b>6) Google LineChart를 이용한 그래프 그리기</b>  
가로축 데이터를 입력하는데 Date 형식으로 받고자 하면 new Date(year, month, day)를 이용해야한다.  
```
data.addRows([ 
[ new Date(<%=year%>, <%=month%>, <%=day%>), <%=dto.getClose()%> ] 
]);

var options = {
	hAxis : {
		title : 'Date',
		format : 'yyyy-MM'
	}
}
```
※ [반응형으로 그래프 그리기](https://codepen.io/flopreynat/pen/BfLkA)  
→ 그래프가 반응형이 아니면 graph.jsp에 접근하는순간의 브라우저 창크기를 기준으로 그래프가 그려짐  
→ 접속했을 때 무슨 크기인지 알 수 없기 때문에 반응형으로 그 때 그 때 동적으로 사이즈를 할당해야 한다.  
(특히 horizontal 크기) 
```
<div id="chart_div" style="width: 100%; min-height:500px"></div>

<script>
$(window).resize(function(){
	drawTrendlines();
</script>
});
```
