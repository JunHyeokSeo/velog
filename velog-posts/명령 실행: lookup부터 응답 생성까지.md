<h3 id="41-processcommand의-역할">4.1 processCommand()의 역할</h3>
<ul>
<li><p>파싱된 명령어(<code>client-&gt;argv[0]</code>)를 기반으로 실제 실행할 명령 구조체를 식별하고 실행을 수행</p>
</li>
<li><p>내부 단계:</p>
<ol>
<li>명령어 조회: <code>lookupCommand()</code> 호출</li>
<li>권한 및 인증 체크 (ex. ACL, 레플리카 읽기 제한)</li>
<li>명령 실행 함수 호출: <code>cmd-&gt;proc(client *c)</code></li>
</ol>
</li>
</ul>
<h3 id="42-lookupcommand">4.2 lookupCommand()</h3>
<ul>
<li><p>전역 명령어 해시 테이블에서 <code>argv[0]</code> 값과 일치하는 <code>redisCommand</code> 구조체를 조회</p>
</li>
<li><p>내부적으로는 문자열 → 커맨드 포인터 매핑이 해시 기반으로 구성되어 있음</p>
</li>
<li><p>이 구조체에는 다음과 같은 정보가 포함됨:</p>
<ul>
<li>이름, 인자 수, flag, key 위치, 실행 함수 포인터(proc)</li>
</ul>
</li>
</ul>
<h3 id="43-subcommand-처리">4.3 subcommand 처리</h3>
<ul>
<li>일부 명령어는 subcommand 구조를 가짐 (ex. <code>CONFIG GET</code>)</li>
<li><code>cmd-&gt;subcommands</code>를 통해 하위 명령 테이블 존재 여부를 확인</li>
<li>있을 경우, <code>lookupSubcommand()</code>를 통해 하위 명령 구조체 조회</li>
</ul>
<h3 id="44-명령어-실행">4.4 명령어 실행</h3>
<ul>
<li><code>cmd-&gt;proc()</code> 함수 포인터 호출로 실제 동작 수행</li>
<li>예시: <code>echoCommand(client *c)</code>는 <code>addReply(c, c-&gt;argv[1])</code> 호출로 응답 준비</li>
<li>실행 중 오류가 발생하면 <code>addReplyError()</code> 또는 관련 에러 처리 루틴이 호출됨</li>
</ul>
<h3 id="45-응답-버퍼-구성">4.5 응답 버퍼 구성</h3>
<ul>
<li>실행 결과는 클라이언트 구조체 내 <code>client-&gt;reply</code> 큐에 저장</li>
<li>이 큐는 sds 구조체를 기반으로 하며, write 전까지 대기</li>
<li>이 과정은 명령어가 직접 데이터를 반환하는 대신, 출력을 비동기로 준비시키는 방식</li>
</ul>
<h3 id="46-ae_writable-이벤트-등록">4.6 AE_WRITABLE 이벤트 등록</h3>
<ul>
<li>명령 실행 후 응답이 있을 경우, <code>prepareClientToWrite()</code>가 호출됨</li>
<li>이 함수는 클라이언트 fd에 <code>AE_WRITABLE</code> 이벤트를 등록하고, <code>wfileProc</code>을 <code>sendReplyToClient()</code>로 설정</li>
<li>이로 인해 다음 이벤트 루프에서 해당 fd가 쓰기 가능할 때 응답 전송이 가능해짐</li>
</ul>
<h3 id="47-명령-실행-흐름-요약">4.7 명령 실행 흐름 요약</h3>
<pre><code class="language-text">client-&gt;argv[0] → lookupCommand()
                 ↓
            redisCommand 구조체
                 ↓
           cmd-&gt;proc(client *c)
                 ↓
     결과 → client-&gt;reply[]에 저장
                 ↓
   prepareClientToWrite() → AE_WRITABLE 등록</code></pre>
<h3 id="정리">정리</h3>
<ul>
<li><code>processCommand()</code>는 파싱된 명령을 실행하는 중심 허브</li>
<li>lookup 단계에서의 검증, 명령어 함수 실행, 응답 큐 구성, writable 이벤트 등록까지 일괄 처리</li>
<li>이 구조는 명령 실행의 side effect를 명확히 분리하고, 응답 처리를 비동기로 분할함으로써 고성능 처리를 가능하게 한다</li>
</ul>