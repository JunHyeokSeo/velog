<h3 id="51-응답-대기-큐-client-reply">5.1 응답 대기 큐: client-&gt;reply</h3>
<ul>
<li>명령어 실행 결과는 즉시 네트워크로 전송되지 않음</li>
<li>대신 <code>client-&gt;reply</code>라는 큐에 sds 버퍼 형태로 임시 저장됨</li>
<li>이 큐는 FIFO 방식으로 동작하며, flush 타이밍은 서버의 쓰기 가능 상태에 따라 조절됨</li>
<li>이 구조는 I/O 병목을 피하고, 비동기 처리로 throughput을 높이기 위한 전략임</li>
</ul>
<h3 id="52-prepareclienttowrite">5.2 prepareClientToWrite()</h3>
<ul>
<li><code>client-&gt;reply</code>에 데이터가 추가되면, Valkey는 해당 클라이언트의 fd에 대해 <code>AE_WRITABLE</code> 이벤트를 등록</li>
<li>이 작업은 <code>prepareClientToWrite()</code> 함수에서 수행됨</li>
<li>내부적으로는 <code>aeCreateFileEvent()</code>를 호출하여 이벤트 루프에 쓰기 이벤트 추가</li>
</ul>
<h3 id="53-ae_writable의-의미">5.3 AE_WRITABLE의 의미</h3>
<ul>
<li>커널이 해당 fd에 대해 write-ready 상태임을 알리는 신호</li>
<li>send buffer가 비어있거나 여유 공간이 있음을 의미</li>
<li>이 이벤트를 감지하면 Valkey는 등록된 <code>wfileProc</code>, 즉 <code>sendReplyToClient()</code>를 호출</li>
</ul>
<h3 id="54-sendreplytoclient">5.4 sendReplyToClient()</h3>
<ul>
<li><p>클라이언트의 <code>reply</code> 큐에 있는 데이터를 하나씩 꺼내 <code>write()</code> 시스템 콜을 통해 TCP send buffer에 복사</p>
</li>
<li><p>전체 데이터를 한 번에 전송하지 못하면:</p>
<ul>
<li>전송되지 않은 데이터는 큐에 그대로 남김</li>
<li>AE_WRITABLE 이벤트 유지</li>
<li>다음 루프에서 다시 전송 시도</li>
</ul>
</li>
</ul>
<h3 id="55-send-buffer와-커널의-역할">5.5 send buffer와 커널의 역할</h3>
<ul>
<li><code>write()</code>는 데이터를 커널의 send buffer로 복사만 할 뿐, 실제 전송은 커널 TCP 스택이 담당</li>
<li>실제 flush 타이밍은 OS의 TCP 혼잡 제어, ACK 수신 여부, MTU, 소켓 상태 등에 따라 결정됨</li>
<li>Valkey는 이를 직접 제어하지 않으며, 오직 <code>write()</code> 호출만 반복함</li>
</ul>
<h3 id="56-클라이언트-처리-종료">5.6 클라이언트 처리 종료</h3>
<ul>
<li>모든 응답이 전송되면 <code>client-&gt;reply</code>는 비워지고, AE_WRITABLE 이벤트는 해제됨</li>
<li><code>client-&gt;flags</code>에 따라 일부는 종료 요청(CLIENT_CLOSE_AFTER_REPLY)으로 연결 종료됨</li>
</ul>
<h3 id="57-흐름-요약">5.7 흐름 요약</h3>
<pre><code class="language-text">cmd 실행 결과 → client-&gt;reply[]에 저장
              ↓
prepareClientToWrite() → AE_WRITABLE 등록
              ↓
sendReplyToClient() → write() 호출
              ↓
kernel send buffer로 복사 (flush는 OS 결정)
              ↓
모든 응답 전송 시 AE_WRITABLE 해제</code></pre>
<h3 id="정리">정리</h3>
<ul>
<li>AE_WRITABLE 이벤트는 비동기 응답 전송을 위한 핵심 트리거</li>
<li><code>reply</code> 큐는 비동기 write를 위한 staging 영역이며, 이를 통해 네트워크 지연과 처리 병목을 완화함</li>
<li>이 설계는 높은 연결 수와 동시성 하에서 효율적인 응답 처리를 가능케 한다</li>
</ul>