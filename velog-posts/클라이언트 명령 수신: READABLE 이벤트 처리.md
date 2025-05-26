<h3 id="21-클라이언트로부터의-입력-흐름-개요">2.1 클라이언트로부터의 입력 흐름 개요</h3>
<ul>
<li>클라이언트는 TCP 소켓을 통해 명령어를 전송</li>
<li>Valkey는 해당 소켓 fd에 대해 AE_READABLE 이벤트를 등록한 상태</li>
<li>커널은 소켓에 데이터가 수신되었을 때 이를 감지하고 이벤트 루프는 <code>rfileProc</code> 콜백을 호출함</li>
</ul>
<h3 id="22-rfileproc과-connsocketeventhandler">2.2 rfileProc과 connSocketEventHandler</h3>
<ul>
<li><p>rfileProc은 일반적으로 <code>connSocketEventHandler()</code>로 설정되어 있음</p>
</li>
<li><p>이 함수는 내부적으로 connection 구조체에 등록된 read 콜백을 호출 (예: <code>readQueryFromClient()</code>)</p>
</li>
<li><p>처리 흐름:</p>
<ol>
<li>커널이 fd에 대해 readable 이벤트 감지</li>
<li>이벤트 루프에서 <code>rfileProc(eventLoop, fd, clientData, mask)</code> 호출</li>
<li>내부적으로 연결된 <code>readQueryFromClient()</code> 함수가 실행됨</li>
</ol>
</li>
</ul>
<h3 id="23-readqueryfromclient">2.3 readQueryFromClient()</h3>
<ul>
<li><p>클라이언트의 소켓으로부터 데이터를 읽고 <code>client-&gt;querybuf</code>에 저장</p>
</li>
<li><p>버퍼는 growable한 sds 구조를 사용하여 동적으로 확장됨</p>
</li>
<li><p>데이터를 모두 읽은 뒤에는 파싱을 위해 <code>processInputBuffer()</code> 호출</p>
</li>
<li><p>중요한 포인트:</p>
<ul>
<li>이 시점에서는 단순히 수신만 담당함</li>
<li>실행은 아직 일어나지 않음</li>
</ul>
</li>
</ul>
<h3 id="24-processinputbuffer">2.4 processInputBuffer()</h3>
<ul>
<li><p>입력 버퍼에서 명령어가 완전하게 수신되었는지 확인</p>
</li>
<li><p>RESP 프로토콜 구조에 따라 파싱 방식 결정:</p>
<ul>
<li>첫 문자가 <code>*</code> → <code>processMultibulkBuffer()</code> (Multibulk 방식)</li>
<li>그 외 → <code>processInlineBuffer()</code> (Inline 방식)</li>
</ul>
</li>
<li><p>파싱 완료 후 명령어와 인자를 <code>client-&gt;argv[]</code>에 저장하고 <code>client-&gt;argc</code> 갱신</p>
</li>
</ul>
<h3 id="25-연결의-논블로킹-특성">2.5 연결의 논블로킹 특성</h3>
<ul>
<li><code>read()</code>는 non-blocking으로 설정되어 있음</li>
<li>한 번의 호출로 모든 데이터를 읽지 못할 수 있음</li>
<li>이 경우 다음 이벤트 루프에서 추가 수신이 계속됨</li>
</ul>
<h3 id="26-핵심-구조-요약">2.6 핵심 구조 요약</h3>
<pre><code class="language-text">[클라이언트 입력]
   ↓
커널에서 READABLE 이벤트 감지
   ↓
rfileProc → connSocketEventHandler
   ↓
readQueryFromClient
   ↓
client-&gt;querybuf에 저장
   ↓
processInputBuffer()</code></pre>
<h3 id="27-보안-및-연결-관리">2.7 보안 및 연결 관리</h3>
<ul>
<li>명령 수신 시, 악성 클라이언트 또는 비정상 연결은 이 단계에서 필터링 가능</li>
<li>querybuf 오버플로 방지, 인증 미수행 클라이언트 차단, TIMEOUT 적용 등</li>
</ul>
<h3 id="정리">정리</h3>
<ul>
<li>READABLE 이벤트는 클라이언트 명령 수신의 진입점으로, 전체 처리 파이프라인의 시작</li>
<li><code>readQueryFromClient()</code>와 <code>processInputBuffer()</code>는 입력 수신과 파싱을 분리하여 명확한 역할을 수행</li>
<li>이 과정은 모든 명령 실행 이전에 반드시 거치는 필수 단계</li>
</ul>