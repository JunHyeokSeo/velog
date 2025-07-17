<h1 id="api-uri-설계">API URI 설계</h1>
<aside>
💡 가장 중요한 것은 `리소스 식별`!

</aside>

<p><strong>리소스란 무엇인가?</strong> 회원 등록/수정에서 회원이라는 개념 자체가 리소스다</p>
<p><strong>리소스를 어떻게 식별하는게 좋을까?</strong> 회원이라는 리소스만 식별하면 된다 ⇒ 회원 리소스를 URI에 매핑</p>
<h2 id="리소스-식별-uri-계층-구조-활용">리소스 식별, URI 계층 구조 활용</h2>
<aside>
💡 계층 구조상 상위를 컬렉션으로 보고 복수단어 사용을 권장한다!

</aside>

<p>회원 목록 조회: <code>/members</code></p>
<p>회원 조회: <code>/members/{id}</code></p>
<p>회원 등록: <code>/members/{id}</code></p>
<aside>
💡 API 별 URI가 동일하다면 어떻게 구분하지? `HTTP 메서드 사용`

</aside>

<h1 id="http-method">HTTP Method</h1>
<h2 id="주요-메서드">주요 메서드</h2>
<p>GET: 리소스 조회</p>
<p>POST: 요청 데이터 처리, 주로 등록에 사용</p>
<p>PUT: 리소스를 대체, 해당 리소스가 없으면 생성</p>
<p>PATCH: 리소스 부분 변경</p>
<p>DELETE: 리소스 삭제</p>
<h2 id="get">GET</h2>
<p>리소스 조회</p>
<p>서버에 전달하고 싶은 데이터는 query를 통해서 전달</p>
<h2 id="post">POST</h2>
<p>요청 데이터 처리</p>
<p>메시지 바디를 통해 서버로 요청 데이터 전달 ⇒ 서버는 요청 데이터를 처리</p>
<p>메시지 바디를 통해 들어온 데이터를 처리하는 모든 기능을 수행한다</p>
<h3 id="사용-예시">사용 예시</h3>
<aside>
💡 리소스 URI에 POST 요청이 오면 데이터를 어떻게 처리할지 리소스마다 따로 정해야 함

<p>= 정해진 것이 없음</p>
</aside>

<ol>
<li><p>새 리소스 생성(등록)</p>
</li>
<li><p>요청 데이터 처리</p>
<p> 단순히 데이터를 생성/변경하는 것을 넘어서 프로세스를 처리해야 하는 경우</p>
<p> POST의 결과로 새로운 리소스가 생성되지 않을 수 있음</p>
<p> <code>POST /order/{orderID}/start-delivery(Control URI)</code></p>
</li>
<li><p>다른 매서드로 처리하기 애매한 경우</p>
</li>
</ol>
<h2 id="put">PUT</h2>
<aside>
💡 리소스가 있으면 대체, 없으면 생성

</aside>

<p>전송되는 필드만 수정되는 것이 아니라 데이터를 완전히 대체한다</p>
<p>username, age 필드가 있는 URI에 age 필드값만 전송하면, username 필드는 <code>삭제된다</code></p>
<p>수정에 어려움이 있다 ⇒ <code>Patch</code> 사용</p>
<h3 id="post와의-차이점">POST와의 차이점</h3>
<p>클라이언트가 리소스 위치를 알고 URI를 지정한다</p>
<h2 id="patch">PATCH</h2>
<p>리소스 부분 변경 </p>
<p>PATCH를 지원하지 않는 서버의 경우 POST를 사용하여 구현한다</p>
<h2 id="delete">DELETE</h2>
<p>리소스 제거</p>
<h1 id="http-메서드의-속성">HTTP 메서드의 속성</h1>
<h2 id="속성">속성</h2>
<ol>
<li>안전(Safe Methods)</li>
<li>멱등(Idempotent Methods)</li>
<li>캐시가능(Cacheable Methods)</li>
</ol>
<h2 id="표">표</h2>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/4f5d0bb6-d167-41f8-85f9-4e2d4c4d1c70/image.png" /></p>
<h2 id="안전safe">안전(Safe)</h2>
<p>호출해도 리소스를 변경하지 않는다</p>
<blockquote>
<p>Q. 계속 호출해서, 로그가 쌓여 장애가 발생하면?</p>
<p>A. 안전은 해당 리소스만 고려한다. 그런 부분까지 고려하지 않는다</p>
</blockquote>
<h2 id="멱등idempotent">멱등(Idempotent)</h2>
<p>몇 번을 호출하던 결과가 같다</p>
<blockquote>
<p>Q. 재요청 중간에 다른 곳에서 리소스를 변경해버리면?</p>
<p>A. 멱등은 외부 요인으로 중간에 리소스가 변경되는 것까지는 고려하지 않는다</p>
</blockquote>
<h3 id="멱등-메소드">멱등 메소드</h3>
<p>GET: 한 번 조회하든, 두 번 조회하든 같은 결과가 조회된다</p>
<p>PUT: 결과를 대체한다. 따라서 같은 요청을 여러번 해도 최종 결과는 같다.</p>
<p>DELETE: 결과를 삭제한다. 같은 요청을 여러번 해도 삭제된 결과는 똑같다. </p>
<p>POST: 멱등이 아니다! 두 번 호출하면 같은 결제가 중복해서 발생할 수 있다</p>
<h3 id="활용">활용</h3>
<p><strong>자동 복구 매커니즘</strong></p>
<p>클라이언트 요청을 완료하지 못하고 오류가 발생 ⇒ 자동 복구 판단의 기준</p>
<p>멱등 O: 재실행, 멱등 X : 별도처리</p>
<p>서버가 정상 응답을 못주었을 때, 클라이언트가 같은 요청을 다시 해도 되는지에 대한 판단 근거</p>
<h2 id="캐시가능cacheable">캐시가능(Cacheable)</h2>
<p>응답 결과 리소스를 캐시해서 사용해도 되는가?</p>
<p>GET, HEAD, POST, PATCH 캐시가능</p>
<p>실제로는 GET, HEAD 정도만 캐시로 사용 (POST, PATCH의 경우 구현이 어려움)</p>