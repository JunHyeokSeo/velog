<h3 id="71-이벤트-루프-전-준비-단계">7.1 이벤트 루프 전 준비 단계</h3>
<ul>
<li><code>aeProcessEvents()</code>는 내부적으로 다음 이벤트 루프 전후로 훅(hook) 함수들을 호출함</li>
<li>그중 <code>beforeSleep()</code>은 커널의 I/O multiplexing 호출 전 마지막으로 실행되는 함수임</li>
<li>이벤트 처리 중 발생한 부가 작업을 마무리하거나 사전 정리함</li>
</ul>
<h3 id="72-주요-처리-목록">7.2 주요 처리 목록</h3>
<ul>
<li><code>processIOThreadsReadDone()</code>: IO 스레드에서 비동기적으로 처리한 읽기 작업을 메인 스레드에서 수합</li>
<li><code>connTypeProcessPendingData()</code>: TLS 또는 기타 특수 커넥션에서 아직 처리되지 않은 데이터를 처리</li>
<li><code>handleClientsWithPendingWrites()</code>: <code>client-&gt;reply</code> 큐에 전송할 데이터가 있는 클라이언트 대상으로 <code>sendReplyToClient()</code>를 호출</li>
<li><code>flushAppendOnlyFile()</code>: AOF 기능이 활성화되어 있다면, 메모리에 쌓인 로그를 디스크에 반영</li>
<li><code>activeExpireCycle()</code>: 만료 시간 도래한 키들을 제거</li>
<li><code>blockedBeforeSleep()</code>: blocking operation에서 복귀한 클라이언트 처리</li>
<li><code>updateFailoverStatus()</code>, <code>trackingBroadcastInvalidationMessages()</code> 등 부가 시스템 유지</li>
</ul>
<h3 id="73-ae_writable-보완-처리">7.3 AE_WRITABLE 보완 처리</h3>
<ul>
<li>beforeSleep에서 직접적으로 클라이언트에 대한 응답을 전송하기도 함</li>
<li>이는 AE_WRITABLE 이벤트가 감지되기 전에도, 응답 가능할 경우 즉시 처리함으로써 대기 시간 최소화</li>
<li>처리 불가능할 경우 AE_WRITABLE 유지</li>
</ul>
<h3 id="74-io-thread와의-동기화">7.4 I/O thread와의 동기화</h3>
<ul>
<li>IO thread가 사용하는 데이터셋에 접근 전/후로 GIL(Global Interpreter Lock) 비슷한 보호 구조 운영</li>
<li>beforeSleep → GIL release, afterSleep → GIL acquire 로 연결됨</li>
</ul>
<h3 id="75-흐름-요약">7.5 흐름 요약</h3>
<pre><code class="language-text">aeProcessEvents()
    ↓
beforeSleep() 호출
    ↓
- 비동기 IO 완료 처리
- AOF flush
- 키 만료 처리
- 응답 전송 시도 (handleClientsWithPendingWrites)
    ↓
aeApiPoll() 진입</code></pre>
<h3 id="정리">정리</h3>
<ul>
<li><code>beforeSleep()</code>은 이벤트 루프 주기 마지막의 housekeeping 역할</li>
<li>다음 이벤트 대기 전, 가능한 응답 및 유지 작업을 미리 처리하여 응답 지연을 줄이고 처리 효율을 높임</li>
<li>AE_WRITABLE 이벤트와 별도로, 명시적 flush 경로를 제공하여 지연 시간 최소화와 동기화 효율을 동시에 확보</li>
</ul>