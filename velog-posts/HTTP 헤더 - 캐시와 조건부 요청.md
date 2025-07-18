<h1 id="캐시">캐시</h1>
<h2 id="캐시-미사용">캐시 미사용</h2>
<p>데이터가 변경되지 않아도 계속 네트워크를 통해서 데이터를 다운로드 받아야 한다.</p>
<p>인터넷 네트워크는 매우 느리고 비싸다.</p>
<p>브라우저 로딩 속도가 느리다.</p>
<p>느린 사용자 경험</p>
<h2 id="캐시-사용">캐시 사용</h2>
<p>캐시 덕분에 캐시 가능 시간동안 네트워크를 사용하지 않아도 된다.</p>
<p>비싼 네트워크 사용량을 줄일 수 있다.</p>
<p>브라우저 로딩 속도가 매우 빠르다.</p>
<p>빠른 사용자 경험</p>
<h2 id="캐시-제어-헤더">캐시 제어 헤더</h2>
<blockquote>
<p><code>Cache-Control:</code> 캐시 제어</p>
<p><code>Pragma:</code> 캐시 제어(하위 호환) </p>
<p><code>Expires:</code> 캐시 유효 기간(하위 호환)</p>
</blockquote>
<h3 id="cache-control">Cache-Control</h3>
<aside>
💡 Cache-Control 헤더만으로도 캐시를 제어할 수 있다

<p>다만, 하위 버전의 브라우저를 호환하기 위해 <code>Pragma, Expires</code>를 같이 사용한다</p>
</aside>

<blockquote>
<p>Cache-Control: max-age  <code>(캐시 유효 시간, 초단위)</code></p>
<p>Cache-Control: no-cache <code>(데이터는 캐시해도 되지만, 항상 원 서버에 검증하고 사용)</code></p>
<p>Cache-Control: no-store   <code>(저장하면 안됨, 메모리에서 사용하고 최대한 빨리 삭제)</code></p>
</blockquote>
<h2 id="캐시-시간-초과">캐시 시간 초과</h2>
<p>캐시 유효 시간이 초과하면, 서버를 통해 데이터를 다시 조회하고, 캐시를 갱신한다.</p>
<p>이때 다시 네트워크 다운로드가 발생한다.</p>
<p>캐시 유효 시간이 초과해서 서버에 다시 요청하면 두 가지 상황이 나타난다</p>
<ol>
<li>서버에서 기존 데이터를 변경함</li>
<li><code>서버에서 기존 데이터를 변경하지 않음</code></li>
</ol>
<h3 id="캐시-만료-후에도-서버에서-데이터를-변경하지-않았다면">캐시 만료 후에도 서버에서 데이터를 변경하지 않았다면?</h3>
<p>데이터를 새로 전송하는 대신에 저장해두었던 캐시를 재사용 할 수 있다</p>
<p>단, 클라이언트의 데이터와 서버의 데이터가 같다는 사실을 확인할 수 있는 방법 필요</p>
<p>⇒ <code>검증헤더</code></p>
<h1 id="검증-헤더">검증 헤더</h1>
<p>캐시를 재사용 하기 위한 <code>검증헤더</code>와 <code>조건부 요청</code>은 크게 두가지 형태가 존재한다</p>
<ol>
<li>Last-Modified, If-Modified-Since</li>
<li>ETag, If-None-Match</li>
</ol>
<h2 id="last-modified-if-modified-since">Last-Modified, if-Modified-Since</h2>
<blockquote>
<p><code>검증헤더:</code> Last-Modified</p>
<p><code>조건부 요청:</code> if-Modified-Since</p>
</blockquote>
<h3 id="시나리오">시나리오</h3>
<ol>
<li><p>클라이언트가 서버에 데이터 요청</p>
</li>
<li><p>서버측 응답에 캐시 관련 헤더와 <code>Last-Modified</code> 헤더 추가</p>
<p> 헤더의 값은 Data의 최종 수정일</p>
</li>
<li><p>브라우저 캐시에 응답 결과<code>(Data 및 최종 수정일 포함)</code>를 저장</p>
</li>
<li><p>브라우저 캐시에 Data 요청 및 기간 만료로 인한 오류 발생</p>
</li>
<li><p>서버에 데이터 요청(요청 패킷에 <code>if-modified-since</code>헤더 추가)</p>
<p> if-modified-since의 값으로 <code>Last-Modified</code> 값(캐시 데이터의 최종 수정일) 사용</p>
</li>
<li><p>서버는 <code>if-modified-since</code>와 Data의 최종 수정일을 비교</p>
</li>
<li><p>변경되지 않았다면 <code>304 Not Modified</code> 상태의 패킷을 보낸다<code>(변경되면 200 OK)</code></p>
</li>
<li><p>클라이언트는 브라우저 캐시에서 다시 조회<code>(3xx 패킷은 자동 리다이렉션)</code></p>
</li>
</ol>
<h3 id="정리">정리</h3>
<p>캐시 유효 시간이 초과해도, 서버의 데이터가 갱신되지 않으면
<code>304 Not Modified + 헤더 메타 정보만 응답 (바디X)</code></p>
<p>클라이언트는 서버가 보낸 <code>응답 헤더 정보로 캐시의 메타 정보를 갱신</code></p>
<p>클라이언트는 캐시에 저장되어 있는 데이터 재활용</p>
<p>결과적으로 네트워크 다운로드가 발생하지만 용량이 적은 헤더 정보만 다운로드 매우 실용적인 해결책</p>
<h2 id="etag-if-none-match">ETag, If-None-Match</h2>
<blockquote>
<p><code>검증헤더:</code> ETag</p>
<p><code>조건부 요청:</code> If-None-Match</p>
</blockquote>
<h3 id="last-modified-if-modified-since-단점">Last-Modified, if-Modified-Since 단점</h3>
<p>1초 미만 단위로 캐시 조정이 불가능</p>
<p>날짜 기반의 로직 사용 ⇒ 날짜가 다르지만 데이터가 똑같은 경우 동일한 데이터를 다시 전송</p>
<p>서버에서 별도의 캐시 매커니즘을 사용하고 싶은 경우 구현 불가능</p>
<p>⇒ <code>ETag, If-None-Match</code> 사용</p>
<h3 id="etag">ETag</h3>
<p>캐시용 데이터에 임의의 고유한 버전 이름을 달아둠</p>
<p>데이터가 변경되면 이름을 바꿔서 Hash를 다시 생성</p>
<p>단순하게, ETag만 보내서 같으면 유지, 다르면 다시 받기</p>
<h3 id="시나리오-1">시나리오</h3>
<ol>
<li><p>클라이언트가 서버에 데이터 요청</p>
</li>
<li><p>서버측 응답에 캐시 관련 헤더와 <code>ETag</code> 헤더 추가</p>
<p> ETag의 값은 해당 Data 고유의 버전 이름(매커니즘에 따라 달라짐)</p>
</li>
<li><p>브라우저 캐시에 응답 결과<code>(ETag 포함)</code>를 저장</p>
</li>
<li><p>브라우저 캐시에 Data 요청 및 기간 만료로 인한 오류 발생</p>
</li>
<li><p>서버에 데이터 요청(요청 패킷에 <code>if-none-match</code>헤더 추가)</p>
<p> if-none-match의 값으로 <code>ETag</code>(Data 고유의 버전 이름) 사용</p>
</li>
<li><p>서버는 <code>if-none-match</code>와 서버 내부의 <code>ETag</code>를 비교</p>
</li>
<li><p>변경되지 않았다면 <code>304 Not Modified</code> 상태의 패킷을 보낸다<code>(변경되면 200 OK)</code></p>
</li>
<li><p>클라이언트는 브라우저 캐시에서 다시 조회<code>(3xx 패킷은 자동 리다이렉션)</code></p>
</li>
</ol>
<h3 id="정리-1">정리</h3>
<p>진짜 단순하게 ETag만 서버에 보내서 같으면 유지, 다르면 다시 받기!</p>
<p><code>캐시 제어 로직을 서버에서 완전히 관리</code></p>
<p>클라이언트는 단순히 이 값을 서버에 제공<code>(클라이언트는 캐시 메커니즘을 모름)</code></p>
<h1 id="프록시-캐시">프록시 캐시</h1>
<aside>
💡 클라이언트가 매번 Origin 서버에 데이터를 요청하는 것은 비효율적 ⇒ 프록시 캐시 사용

</aside>

<blockquote>
<p><code>Cache-Control: public</code> 응답이 public 캐시에 저장되어도 됨</p>
<p><code>Cache-Control: private</code> 응답이 해당 사용자만을 위한 것임, private 캐시에 저장해야 함(기본값)</p>
</blockquote>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/8e15d015-090a-41b4-a0c4-e9d83c57ff78/image.png" /></p>
<h1 id="캐시-무효화">캐시 무효화</h1>
<aside>
💡 요청이 없더라도 브라우저가 임의로 캐시하는 경우가 있다

<p>⇒ 캐시되지 않아야 하는 정보를 요청하는 경우 아래의 헤더를 사용하여 캐시를 무효화 한다</p>
</aside>

<blockquote>
<p><code>Cache-Control</code>: no-cache, no-store, must-revalidate</p>
<p><code>Pragma</code>: no-cache</p>
</blockquote>
<h2 id="no-cache-vs-must-revalidate">no-cache VS must-revalidate</h2>
<p><strong>no-cache, no-store를 사용하면 충분할 것 같은데 must-revalidate까지 사용하는 이유는?</strong></p>
<p>no-cache는 항상 Origin 서버에 캐시를 검증하고 사용한다.</p>
<p>Client와 Origin 서버 사이의 Proxy 서버에서 네트워크 단절 등의 이유로 Origin 서버에 접근할 수 없는 상황 발생 시, 오류가 발생하지 않고 오래된 캐시 데이터를 보내주는 경우가 있다.</p>
<p><code>(오류 보다는 오래된 데이터라도 전송하는 것이 낫다는 내부 매커니즘에 의해)</code></p>
<p>must-revalidate 헤더 사용 시, Origin 서버에 접근할 수 없는 경우 <code>항상 오류</code>가 발생하게 되므로 완전한 캐시 무효화를 구현할 수 있다.</p>