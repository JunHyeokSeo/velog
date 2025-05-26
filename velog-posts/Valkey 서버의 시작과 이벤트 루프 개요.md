<h3 id="main-함수">main() 함수</h3>
<ul>
<li><p>Valkey 서버의 진입점으로, 프로세스가 시작되면 제일 먼저 호출됨.</p>
</li>
<li><p>기본적인 초기화 작업을 수행:</p>
<ul>
<li>시간 설정 (<code>tzset()</code>), 메모리 할당기 설정 (<code>zmalloc_set_oom_handler()</code>), 난수 시드 설정 (<code>srandom()</code>, <code>init_genrand64()</code>), 서버 설정 구조 초기화 (<code>initServerConfig()</code> 등).</li>
<li>서버 PID 저장, ACL 및 모듈 시스템 초기화.</li>
</ul>
</li>
<li><p>이후 서버 실행 모드에 따라 sentinel 모드, RDB 체크, AOF 체크 등 진입 분기.</p>
</li>
<li><p>최종적으로 <code>aeMain()</code> 호출하여 이벤트 루프 진입.</p>
</li>
</ul>
<h3 id="aemain">aeMain()</h3>
<ul>
<li>이벤트 루프의 시작점으로, <code>aeProcessEvents()</code>를 반복 호출하며 모든 이벤트를 처리하는 루프.</li>
<li><code>aeEventLoop</code> 구조체를 중심으로 동작하며, 이 안에 등록된 file 이벤트(read/write)와 time 이벤트(timer)들을 반복적으로 확인 및 실행함.</li>
</ul>
<h3 id="aeprocesseventsaeeventloop-eventloop-int-flags">aeProcessEvents(aeEventLoop *eventLoop, int flags)</h3>
<ul>
<li><p>이벤트 루프의 핵심 함수.</p>
</li>
<li><p>file 이벤트(AE_READABLE, AE_WRITABLE) 및 time 이벤트(AE_TIME_EVENTS)를 확인하고 처리함.</p>
</li>
<li><p>내부 구조:</p>
<ol>
<li><code>beforesleep</code> 콜백 실행 (ex: AOF flush, 만료 처리 등)</li>
<li><code>aeApiPoll()</code> 호출로 커널로부터 이벤트 대기 (epoll, kqueue 등 플랫폼 의존적 I/O multiplexing)</li>
<li>감지된 이벤트 수만큼 반복하며 해당 fd의 콜백(rfileProc/wfileProc) 실행</li>
<li>필요시 <code>processTimeEvents()</code> 호출하여 타이머 기반 작업 처리</li>
</ol>
</li>
</ul>
<h3 id="aeapipoll">aeApiPoll()</h3>
<ul>
<li>커널 이벤트 감지 함수.</li>
<li>epoll, kqueue 등 시스템에 따라 구현된 low-level 함수로부터 입력/출력 준비 상태인 fd 목록을 가져옴.</li>
<li>감지된 이벤트를 <code>eventLoop-&gt;fired[]</code>에 저장하여 이후 루프에서 처리되도록 함.</li>
</ul>
<h3 id="fired">fired[]</h3>
<ul>
<li>이번 루프에서 실제 I/O 이벤트가 발생한 fd 및 해당 mask 정보를 담는 배열.</li>
<li><code>aeProcessEvents()</code>는 이 배열을 순회하면서 해당 fd에 등록된 콜백 함수들을 호출함.</li>
</ul>
<h3 id="beforesleep-콜백">beforesleep 콜백</h3>
<ul>
<li><p>실제 커널 I/O 감지 직전에 실행되는 후처리 hook 함수.</p>
</li>
<li><p>주요 작업:</p>
<ul>
<li><code>handleClientsWithPendingWrites()</code>로 쓰기 대기 중인 클라이언트 처리</li>
<li><code>flushAppendOnlyFile()</code>로 AOF 버퍼 처리</li>
<li>만료 키 제거(<code>activeExpireCycle()</code>), 클러스터 상태 점검, 모듈 GIL 해제 등</li>
</ul>
</li>
</ul>
<h3 id="time-이벤트와-aeprocessevents의-통합">time 이벤트와 aeProcessEvents의 통합</h3>
<ul>
<li><code>aeProcessEvents()</code>는 I/O 이벤트뿐 아니라 time 이벤트도 함께 처리함.</li>
<li>sleep 전 <code>usUntilEarliestTimer()</code>를 통해 다음 실행될 타이머까지의 남은 시간을 확인하고, 그만큼만 대기하도록 <code>aeApiPoll()</code>에 timeout 전달.</li>
<li><code>processTimeEvents()</code>는 시간 조건이 충족된 타이머 콜백 함수들을 실행함.</li>
</ul>
<h3 id="중요-개념-non-blocking과-이벤트-중심-구조">중요 개념: Non-blocking과 이벤트 중심 구조</h3>
<ul>
<li>Valkey는 단일 스레드 이벤트 기반 구조를 사용함.</li>
<li>모든 fd에 대해 non-blocking I/O 설정이 되어 있으며, I/O는 언제든지 처리 가능한 시점에서만 실행됨.</li>
<li>이를 통해 하나의 스레드에서도 수천~수만 개의 클라이언트를 처리 가능함.</li>
</ul>
<h3 id="정리">정리</h3>
<ul>
<li>Valkey 서버는 <code>main()</code>에서 초기화된 후 <code>aeMain()</code>을 통해 이벤트 루프에 진입한다.</li>
<li><code>aeProcessEvents()</code>는 이벤트 중심 구조의 핵심으로, 클라이언트 명령, 타이머, 내부 작업 등 거의 모든 로직이 이 함수 내에서 처리된다.</li>
<li>이벤트 루프는 non-blocking I/O + 최소 슬립 타이머 조합으로 고성능 동시성을 실현한다.</li>
</ul>