<h1 id="http-상태코드">HTTP 상태코드</h1>
<aside>
💡 클라이언트가 보낸 요청의 처리 상태를 응답에서 알려주는 기능

</aside>

<blockquote>
<p>1xx (Informational): 요청이 수신되어 처리중</p>
<p>2xx (Successful): 요청 정상 처리</p>
<p>3xx (Redirection): 요청을 완료하려면 추가 행동이 필요</p>
<p>4xx (Client Error): 클라이언트 오류, 잘못된 문법등으로 서버가 요청을 수행할 수 없음</p>
<p>5xx (Server Error): 서버 오류, 서버가 정상 요청을 처리하지 못함</p>
</blockquote>
<h2 id="모르는-상태-코드가-나타나면">모르는 상태 코드가 나타나면?</h2>
<aside>
💡 299 (클라이언트가 인식 불가) ⇒ 2XX 형태이므로 정상처리에 대한 상태코드로 이해함

</aside>

<p>클라이언트가 인식할 수 없는 상태코드를 서버가 반환하면 상위 상태 코드로 해석해서 처리</p>
<p>새로운 상태 코드가 추가되더라도 클라이언트를 변경하지 않아도 됨</p>
<h1 id="2xx-successful">2xx (Successful)</h1>
<aside>
💡 클라이언트의 요청을 성공적으로 처리

</aside>

<h3 id="200-ok">200 OK</h3>
<p>요청 성공</p>
<h3 id="201-created">201 Created</h3>
<p>요청 성공해서 새로운 리소스가 생성됨</p>
<p>생성된 리소스는 응답의 Location 헤더 필드로 식별</p>
<h3 id="202-accepted">202 Accepted</h3>
<p>요청이 접수되었으나 처리가 완료되지 않음</p>
<p>배치 처리 등에 사용</p>
<h3 id="204-no-content">204 No Content</h3>
<aside>
💡 요청에 대한 성공 여부만 알고 싶을 때

</aside>

<p>요청을 성공적으로 처리했지만, 응답 페이로드 본문에 보낼 데이터가 없음</p>
<p>예) 웹 문서 편집기의 save 버튼</p>
<ul>
<li>save 버튼의 결과로 아무 내용이 없어도된다</li>
<li>save 버튼을 눌러도 같은 화면을 유지해야 한다</li>
</ul>
<h1 id="3xx-redirection">3xx (Redirection)</h1>
<aside>
💡 요청을 완료하기 위해 유저 에이전트`(Client 프로그램, 주로 웹 브라우저)`의 추가 조치 필요

</aside>

<aside>
💡 웹 브라우저는 3xx 응답의 결과에 Location 헤더가 있으면 해당 위치로 자동 이동(Redirect)

</aside>

<h3 id="리다이렉션-종류">리다이렉션 종류</h3>
<ol>
<li><p>영구 리다이렉션</p>
</li>
<li><p>일시 리다이렉션</p>
<p> 주문 완료 후 주문 내역 화면으로 이동</p>
<p> <code>PRG : Post/Redirect/Get</code></p>
</li>
<li><p>특수 리다이렉션</p>
<p> 캐시 사용 여부를 서버에게 물어볼 때, 그 응답으로 사용되는 리다이렉션</p>
<p> 캐시에서 다시 조회하도록 하는 리다이렉션</p>
</li>
</ol>
<h2 id="영구-리다이렉션">영구 리다이렉션</h2>
<p>리소스의 URI가 영구적으로 이동
원래의 URL를 사용X, 검색 엔진 등에서도 변경 인지</p>
<h3 id="301-moved-permanently">301 Moved Permanently</h3>
<p>리다이렉트시 요청 메서드가 GET으로 변하고<code>(POST → GET)</code>, 본문이 제거될 수 있음<code>(MAY)</code></p>
<h3 id="308-permanent-redirect">308 Permanent Redirect</h3>
<p>301과 기능은 같음</p>
<p>리다이렉트시 요청 메서드와 본문 유지<code>(처음 POST를 보내면 리다이렉트도 POST 유지)</code></p>
<h2 id="일시-리다이렉션">일시 리다이렉션</h2>
<aside>
💡 많은 애플리케이션 라이브러리들이 302를 기본값으로 사용

<p>자동 리다이렉션 시, GET으로 변해도 되면 302를 사용해도 문제 없음</p>
</aside>

<p>리소스의 URI가 일시적으로 변경
따라서 검색 엔진 등에서 URL을 변경하면 안됨</p>
<h3 id="302-found">302 Found</h3>
<p>리다이렉트시 요청 메서드가 <code>GET</code>으로 변하고, 본문이 제거될 수 있음<code>(MAY)</code></p>
<h3 id="307-temporary-redirect">307 Temporary Redirect</h3>
<p>302와 기능은 같음</p>
<p>리다이렉트시 요청 <code>메서드와 본문 유지</code>(요청 메서드를 변경하면 안된다. <code>MUST NOT</code>)</p>
<h3 id="303-see-other">303 See Other</h3>
<p>302와 기능은 같음</p>
<p>리다이렉트시 <code>요청 메서드가 GET으로 변경</code></p>
<p>302는 대부분의 브라우저에서 요청 메서드가 GET으로 변경되나, 구현에 따라 변경되지 않는 경우도 있음</p>
<p>⇒ 명확하게 GET으로 변경하기 위해 303 사용</p>
<h3 id="prg-postredirectget">PRG: Post/Redirect/Get</h3>
<aside>
💡 Post로 주문 후에 웹 브라우저를 새로고침한다면? `중복 주문`이 될 수 있다

</aside>

<p>POST로 주문 후에 주문 결과 화면을 GET 메서드로 리다이렉트한다</p>
<p>새로고침 해도 결과 화면을 GET으로 조회하므로 중복 주문을 방지할 수 있다</p>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/5a7d2e55-2cca-459b-a93a-30d73a293495/image.png" /></p>
<h2 id="특수-리다이렉션">특수 리다이렉션</h2>
<h3 id="300-multiple-choices">300 Multiple Choices</h3>
<p>안쓴다. </p>
<h3 id="304-not-modified">304 Not Modified</h3>
<p>캐시를 목적으로 사용</p>
<p>클라이언트에게 리소스가 수정되지 않았음을 알려준다. 따라서 클라이언트는 로컬PC에 저장된 캐시를 재사용한다. (캐시로 리다이렉트 한다.) ⇒ <code>네트워크 사용량 감소</code></p>
<p>304 응답은 응답에 메시지 바디를 포함하면 안된다. (로컬 캐시를 사용해야 하므로) </p>
<p>조건부 GET, HEAD 요청시 사용</p>
<h1 id="4xx-client-error">4xx (Client Error)</h1>
<aside>
💡 복구 불가능, 똑같은 요청 재시도? `오류`

</aside>

<p>클라이언트의 요청에 잘못된 문법등으로 서버가 요청을 수행할 수 없음</p>
<p>오류의 원인이 클라이언트에 있음</p>
<p><strong>클라이언트가 이미 잘못된 요청/데이터를 보내고 있기 때문에, 똑같은 재시도가 실패함</strong></p>
<h2 id="400-bad-request">400 Bad Request</h2>
<p>클라리언트가 잘못된 요청을 해서 서버가 요청을 처리할 수 없음</p>
<p>요청 파라미터가 잘못되거나, API 스펙이 맞지 않는 경우</p>
<h2 id="401-unauthorized">401 Unauthorized</h2>
<p>클라이언트가 해당 리소스에 대한 인증이 필요함</p>
<p>인증(Authentication) 되지 않은 경우</p>
<p>응답에 <code>WWW-Authenticate</code> 헤더와 함께 인증 방법을 설명</p>
<p>오류 메시지가 Unauthorized(비인가)이지만 인증에 대한 오류</p>
<blockquote>
<p>인증(Authentication): 본인이 누구인지 확인(로그인)</p>
<p>인가(Authorization): 로그인 O, 권한 여부 확인 </p>
</blockquote>
<h2 id="403-forbidden">403 Forbidden</h2>
<p>서버가 요청을 이해했지만 승인을 거부함</p>
<p>주로 인증 자격 증명은 있지만, 접근 권한이 불충분한 경우</p>
<h2 id="404-not-found">404 Not Found</h2>
<p>요청 리소스가 서버에 없음</p>
<p>또는 클라이언트가 권한이 부족한 리소스에 접근하는 경우 해당 리소스를 숨기고 싶을 때 사용</p>
<h1 id="5xx-server-error">5xx (Server Error)</h1>
<aside>
💡 복구 가능, 똑같은 요청 재시도? `서버 상황에 따라 정상 처리 가능`

</aside>

<h2 id="500-internal-server-error">500 Internal Server Error</h2>
<p>서버 내부 문제로 오류가 발생</p>
<p>애매하면 500 오류를 사용한다</p>
<h2 id="503-service-unavailable">503 Service Unavailable</h2>
<aside>
💡 사용 빈도 낮음

</aside>

<p>서비스 이용 불가</p>
<p>일시적인 과부하 또는 예정된 작업으로 잠시 요청을 처리할 수 없는 경우</p>
<p>Retry-After 헤더 필드로 얼마 뒤에 복구되는지 보낼 수도 있음</p>