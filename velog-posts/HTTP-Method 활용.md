<h1 id="클라이언트-→-서버-데이터-전송">클라이언트 → 서버 데이터 전송</h1>
<h2 id="데이터-전달-방식">데이터 전달 방식</h2>
<ol>
<li>쿼리 파라미터를 통한 데이터 전송
 <code>GET</code>
 주로 정렬 필터(검색어)</li>
<li>메시지 바디를 통한 데이터 전송
 <code>POST, PUT, PATCH</code><br /> 회원 가입, 상품 주문, 리소스 등록/변경</li>
</ol>
<h2 id="클라이언트→서버-데이터-전송">클라이언트→서버 데이터 전송</h2>
<ol>
<li>정적 데이터 조회(<code>GET</code>)</li>
<li>동적 데이터 조회(<code>GET + Query</code>)</li>
<li>HTML Form을 통한 데이터 전송(<code>POST, GET</code>)</li>
<li>HTTP API를 통한 데이터 전송(<code>XML, JSON</code>)</li>
</ol>
<h2 id="정적-데이터-조회">정적 데이터 조회</h2>
<p>정적 데이터는 일반적으로 쿼리 파라미터 없이 리소스 경로로 단순하게 조회 가능</p>
<h2 id="동적-데이터-조회">동적 데이터 조회</h2>
<p>주로 검색, 게시판 목록에서 정렬 필터(검색어)</p>
<p>조회 조건을 줄여주는 필터, 조회 결과를 정렬하는 정렬 조건에 주로 사용</p>
<p>GET을 사용하여 조회하기 때문에 쿼리 파라미터를 사용하여 데이터 전달</p>
<p>⇒ Body 사용도 가능하나 지원하는 서버가 거의 없음</p>
<h2 id="html-form-데이터-전송">HTML Form 데이터 전송</h2>
<h3 id="method">Method</h3>
<p><strong>POST</strong></p>
<p>기본적으로 바디에 key-value 형태의 쿼리 파라미터 형태로 데이터가 존재한다</p>
<p><strong>GET</strong></p>
<p>URL 경로에 쿼리를 넣어서 데이터를 전달한다</p>
<p>GET은 데이터 변경이 발생하는 곳에 사용하면 안되므로 주의가 필요하다</p>
<h3 id="content-type">Content-Type</h3>
<p><strong><code>application/x-www-form-urlencoded</code></strong></p>
<p>기본값</p>
<p>form의 내용을 메시지 바디를 통해서 전송(key-value 쿼리 파라미터 형식)</p>
<p>전송 데이터를 url encoding 처리(<code>abc김 → abc%EA%B9%80</code>)</p>
<p><strong><code>multipart/form-data</code></strong></p>
<p>파일 업로드 같은 바이너리 데이터 전송 시 사용</p>
<p>다른 종류의 여러 파일과 폼의 내용 함께 전송 가능</p>
<h2 id="http-api-데이터-전송">HTTP API 데이터 전송</h2>
<aside>
💡 Android App ↔ Server 처럼 데이터를 바로 전송하는 경우

</aside>

<h3 id="예시">예시</h3>
<p><strong>서버 TO 서버(백엔드 시스템 통신)</strong></p>
<p><strong>앱 클라이언트(아이폰, 안드로이드)</strong></p>
<p><strong>웹 클라이언트</strong></p>
<p>HTML에서 Form 전송 대신 자바 스크립트를 통한 통신에 사용(AJAX)</p>
<p>예) React, VueJs같은 웹 클라이언트와 API 통신</p>
<h3 id="content-type-1">Content-Type</h3>
<p><code>application/json</code>을 주로 사용 (사실상 표준)</p>
<p>Json 형태로 데이터를 전달하겠다는 의미</p>
<p>다른 타입도 물론 가능하다</p>
<h1 id="http-api-설계-예시">HTTP API 설계 예시</h1>
<ol>
<li>HTTP API - 컬렉션</li>
<li>HTTP API - 스토어</li>
<li>HTML FORM 사용</li>
</ol>
<h2 id="http-api-설계---post-기반-등록">HTTP API 설계 - POST 기반 등록</h2>
<blockquote>
<p>회원 목록 /members -&gt; GET
회원 등록 /members -&gt; POST
회원 조회 /members/{id} -&gt; GET
회원 수정 /members/{id} -&gt; <code>PATCH(부분 수정, 빈도 높음)</code>, PUT(덮어쓰기), POST(애매한 상황)
회원 삭제 /members/{id} -&gt; DELETE </p>
</blockquote>
<p>클라이언트는 등록될 리소스의 URI를 모른다</p>
<p>서버가 새로 등록된 리소스 URI를 생성해준다</p>
<h3 id="컬렉션">컬렉션</h3>
<p>서버가 관리하는 리소스 디렉터리</p>
<p>서버가 리소스의 URI를 생성하고 관리</p>
<p>여기서 컬렉션은 <code>/members</code></p>
<h2 id="http-api-설계---put-기반-등록">HTTP API 설계 - PUT 기반 등록</h2>
<blockquote>
<p>파일 업로드 등 특수한 경우만 사용
파일이름을 알고 있으므로 URI 지정 가능.
<strong>만약 동일한 이름의 파일이 있다면?</strong> 덮어씌우면 된다 <code>(PUT 적합)</code></p>
</blockquote>
<blockquote>
<p>파일 목록 /files -&gt; GET
파일 조회 /files/{filename} -&gt; GET
파일 등록 /files/{filename} -&gt; <code>PUT</code> 
파일 삭제 /files/{filename} -&gt; DELETE 
파일 대량 등록 /files -&gt; POST</p>
</blockquote>
<p>클라이언트가 리소스 URI를 알고 있어야 한다</p>
<p>클라이언트가 직접 리소스의 URI를 지정한다</p>
<h3 id="스토어">스토어</h3>
<p>클라이언트가 관리하는 리소스 저장소</p>
<p>클라이언트가 리소스의 URI를 알고 관리</p>
<p>여기서 스토어는 <code>/files</code></p>
<h2 id="html-form-사용">HTML Form 사용</h2>
<p>HTML FORM은 GET, POST만 지원</p>
<p>AJAX 같은 기술을 사용해서 해결 가능 -&gt; 회원 API 참고 </p>
<p>여기서는 순수 HTML, HTML FORM 이야기</p>
<p>GET, POST만 지원하므로 제약이 있음</p>
<blockquote>
<p>회원 목록 /members -&gt; GET</p>
<p>회원 등록 폼 /members/<code>new</code> -&gt; GET</p>
<p>회원 등록 /members/<code>new</code>, /members -&gt; POST</p>
<p>회원 조회 /members/{id} -&gt; GET</p>
<p>회원 수정 폼 /members/{id}/<code>edit</code> -&gt; GET</p>
<p>회원 수정 /members/{id}/<code>edit</code>, /members/{id} -&gt; POST</p>
<p>회원 삭제  /members/{id}/<code>delete</code> -&gt; POST</p>
</blockquote>
<h3 id="controll-url">Controll URL</h3>
<p>API를 설계할 때, 모두 같은 URI에 Method만 변경하는 방식을 권장한다.</p>
<p>그러나 위 방법으로 모든 동작을 표현할 수 없다는 한계가 존재하고 <code>Control URL</code>을 사용하여 해결한다 </p>
<blockquote>
<p>/members/{id}/<code>delete</code>
/members/{id}/<code>edit</code>
Control URI = 리소스 + 동사 형태의 URI</p>
</blockquote>