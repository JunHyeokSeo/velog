<h3 id="31-redis-serialization-protocol-resp-개요">3.1 Redis Serialization Protocol (RESP) 개요</h3>
<ul>
<li><p>Redis 클라이언트와 서버 간 통신은 RESP 프로토콜을 기반으로 함</p>
</li>
<li><p>텍스트 기반이지만 구조화된 형식으로, 명령어 및 파라미터를 효율적으로 구분 가능</p>
</li>
<li><p>주요 유형:</p>
<ul>
<li>Simple Strings (<code>+OK\r\n</code>)</li>
<li>Errors (<code>-Error message\r\n</code>)</li>
<li>Integers (<code>:1000\r\n</code>)</li>
<li>Bulk Strings (<code>$6\r\nfoobar\r\n</code>)</li>
<li>Arrays (<code>*2\r\n$4\r\nLLEN\r\n$6\r\nmylist\r\n</code>)</li>
</ul>
</li>
</ul>
<h3 id="32-파싱-방식-분기">3.2 파싱 방식 분기</h3>
<ul>
<li><p><code>processInputBuffer()</code> 내부에서 querybuf의 첫 바이트 확인:</p>
<ul>
<li><code>*</code> 로 시작 → <code>processMultibulkBuffer()</code>로 분기</li>
<li>나머지 → <code>processInlineBuffer()</code>로 분기</li>
</ul>
</li>
<li><p>Multibulk는 일반적인 Redis 클라이언트가 사용하는 기본 방식이며, inline은 telnet 등 단순 입력용 디버깅용으로 주로 사용됨</p>
</li>
</ul>
<h3 id="33-multibulk-파싱-과정-processmultibulkbuffer">3.3 Multibulk 파싱 과정: processMultibulkBuffer()</h3>
<ul>
<li><p>첫 줄에서 전체 인자 개수 파악: <code>*&lt;argc&gt;\r\n</code></p>
</li>
<li><p>이후 각 인자마다 다음과 같은 형식으로 반복 파싱:</p>
<ul>
<li><code>$&lt;len&gt;\r\n&lt;실제 문자열&gt;\r\n</code></li>
</ul>
</li>
<li><p>파싱된 인자들은 <code>client-&gt;argv[]</code>에 순서대로 저장되고, <code>client-&gt;argc</code>에 총 개수가 저장됨</p>
</li>
<li><p>완전한 명령이 수신되면 C_OK 반환하여 다음 단계로 진행 가능하게 함</p>
</li>
</ul>
<h3 id="34-inline-파싱-과정-processinlinebuffer">3.4 Inline 파싱 과정: processInlineBuffer()</h3>
<ul>
<li>명령어가 <code>PING\r\n</code>과 같이 단순 공백으로 구분되어 들어오는 경우</li>
<li>전체 querybuf를 공백 기준으로 tokenize하여 argv[]에 저장</li>
<li>보안상 querybuf 길이 제한, token 수 제한 등의 방어 로직 포함</li>
</ul>
<h3 id="35-client-구조체의-argv-버퍼">3.5 client 구조체의 argv[] 버퍼</h3>
<ul>
<li><code>client-&gt;argv[]</code>는 Redis 객체 포인터(<code>robj*</code>) 배열로 구성됨</li>
<li>각 인자는 <code>createStringObject()</code>를 통해 heap에 동적으로 생성되며, 명령어 실행 후 해제됨</li>
<li><code>client-&gt;argc</code>는 파라미터 개수를 저장하고 이후 명령 처리에 사용됨</li>
</ul>
<h3 id="36-잘못된-파싱-대응">3.6 잘못된 파싱 대응</h3>
<ul>
<li>malformed한 요청이 감지되면 해당 클라이언트에 에러 메시지 전송 후 연결 종료 또는 재시도 유도</li>
<li><code>client-&gt;flags</code>에 <code>CLIENT_CLOSE_AFTER_REPLY</code> 플래그가 설정될 수 있음</li>
</ul>
<h3 id="37-파싱-흐름-요약">3.7 파싱 흐름 요약</h3>
<pre><code class="language-text">querybuf
  ↓
processInputBuffer()
  ↓
  ├── processMultibulkBuffer() → *형식 RESP 파싱
  └── processInlineBuffer() → 문자열 기반 파싱
  ↓
client-&gt;argv[] 및 argc 세팅 완료</code></pre>
<h3 id="정리">정리</h3>
<ul>
<li>RESP 파싱은 Redis 명령어 실행을 위한 핵심 진입점이며, 정확성과 효율성이 매우 중요함</li>
<li><code>processMultibulkBuffer()</code>는 대부분의 클라이언트 명령어 처리 루트이며, 구조화된 파싱이 가능함</li>
<li>파싱 실패는 보안과 안정성을 위해 즉시 감지 및 차단되어야 하며, 이 구조가 명령 실행 전 검증 계층으로 작동함</li>
</ul>