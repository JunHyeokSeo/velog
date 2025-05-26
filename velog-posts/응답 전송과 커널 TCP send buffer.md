<h3 id="61-write-시스템-콜의-동작">6.1 write() 시스템 콜의 동작</h3>
<ul>
<li><code>write(fd, buf, len)</code>은 유저 공간 데이터를 커널의 send buffer로 복사함</li>
<li>즉, write()는 실제 네트워크 전송을 보장하지 않음</li>
<li>send buffer가 가득 찬 경우, write()는 전송 가능한 만큼만 처리하고 나머지는 이후로 미룸</li>
</ul>
<h3 id="62-커널-tcp-스택의-역할">6.2 커널 TCP 스택의 역할</h3>
<ul>
<li><p>복사된 데이터는 커널의 TCP 계층에서 관리되며, 아래 조건에 따라 전송됨:</p>
<ul>
<li>TCP congestion window에 따라 네트워크로 내보냄</li>
<li>수신자의 ACK 수신 여부</li>
<li>네트워크 MTU와 NIC 상태</li>
<li>OS의 TCP 설정 (예: tcp_nodelay, tcp_cork)</li>
</ul>
</li>
<li><p>이 전송 로직은 유저 공간에서 제어할 수 없음</p>
</li>
</ul>
<h3 id="63-flush-타이밍과-버퍼-상태">6.3 flush 타이밍과 버퍼 상태</h3>
<ul>
<li>커널은 내부적으로 send queue를 관리하며, 일정량의 버퍼가 비거나 전송 조건이 만족될 때만 실제 전송을 수행함</li>
<li>TCP_NODELAY 설정 시 Nagle’s algorithm을 비활성화해 전송 지연을 줄일 수 있음</li>
</ul>
<h3 id="64-write-실패-대응">6.4 write 실패 대응</h3>
<ul>
<li><p>write() 결과가 -1 또는 전송 길이보다 작을 경우:</p>
<ul>
<li>errno가 EAGAIN 또는 EWOULDBLOCK이면 비동기 처리를 위한 신호</li>
<li>이 경우 AE_WRITABLE 이벤트를 유지하여 다음 루프에서 재시도</li>
</ul>
</li>
</ul>
<h3 id="65-유저-공간의-흐름-제어-방식">6.5 유저 공간의 흐름 제어 방식</h3>
<ul>
<li><p>Valkey는 커널 상태를 직접 알 수 없기 때문에 다음 기준으로 흐름 제어를 수행함:</p>
<ul>
<li><code>client-&gt;reply</code> 큐의 크기가 일정 이상 증가 → 클라이언트 일시 차단</li>
<li><code>server.client_max_querybuf_len</code> 등의 설정으로 메모리 사용 제한</li>
</ul>
</li>
</ul>
<h3 id="66-관련-syscall-및-커널-흐름">6.6 관련 syscall 및 커널 흐름</h3>
<ul>
<li><p>유관 시스템 콜:</p>
<ul>
<li><code>write()</code>, <code>send()</code>, <code>sendmsg()</code></li>
<li>socket buffer 상태 조회는 <code>getsockopt()</code>로 제한적 가능</li>
</ul>
</li>
<li><p>리눅스 커널의 net/ipv4/tcp_output.c에서 실제 TCP 전송 로직 처리됨</p>
</li>
</ul>
<h3 id="67-흐름-요약">6.7 흐름 요약</h3>
<pre><code class="language-text">addReply() → client-&gt;reply[] 저장
        ↓
sendReplyToClient() → write() 호출
        ↓
커널 send buffer로 복사
        ↓
실제 전송은 TCP 스택에 의해 조건 만족 시 수행</code></pre>
<h3 id="정리">정리</h3>
<ul>
<li>write()는 단지 유저 공간 데이터를 커널로 넘기는 역할이며, 네트워크 전송 자체는 TCP 스택이 담당</li>
<li>send buffer가 포화되면 Valkey는 이를 감지하고 AE_WRITABLE을 유지하며 다음 루프에서 재시도</li>
<li>커널 TCP 계층의 flush 조건은 유저 공간에서 직접 통제할 수 없으며, 이는 네트워크 성능에 중요한 영향을 미침</li>
</ul>