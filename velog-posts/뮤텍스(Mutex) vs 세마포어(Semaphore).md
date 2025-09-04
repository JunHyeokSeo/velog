<h2 id="개념-비교">개념 비교</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>뮤텍스(Mutex)</th>
<th>세마포어(Semaphore)</th>
</tr>
</thead>
<tbody><tr>
<td>기본 개념</td>
<td>상호 배제 (Mutual Exclusion). 한 번에 하나의 스레드/프로세스만 접근 가능</td>
<td>동시 접근 개수 제어. 세마포어 값 N에 따라 최대 N개까지 동시에 접근 가능</td>
</tr>
<tr>
<td>소유권</td>
<td>잠금을 건(owner)만 해제 가능</td>
<td>누구든 V 연산(signal)로 해제 가능</td>
</tr>
<tr>
<td>제어 단위</td>
<td>1개의 공유 자원 보호</td>
<td>여러 개의 공유 자원 제어 가능</td>
</tr>
<tr>
<td>용도</td>
<td>단일 임계 구역 보호 (ex: 계좌, 파일 쓰기)</td>
<td>제한된 개수 자원 제어 (ex: 좌석, 버퍼)</td>
</tr>
<tr>
<td>비유</td>
<td>화장실 열쇠 1개</td>
<td>식당 의자 N개</td>
</tr>
</tbody></table>
<hr />
<h2 id="적용-범위">적용 범위</h2>
<ul>
<li>둘 다 <strong>멀티 프로세스 / 멀티 스레드 환경</strong>에서 사용 가능.</li>
<li>POSIX: <code>pthread_mutex</code>, <code>sem_t</code></li>
<li>System V IPC: <code>semget</code>, <code>semop</code></li>
</ul>
<hr />
<h2 id="동작-방식">동작 방식</h2>
<h3 id="뮤텍스">뮤텍스</h3>
<ul>
<li>Lock/Unlock으로 자원 보호.</li>
<li>한 번에 하나만 접근 가능.</li>
</ul>
<pre><code class="language-c">pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;

pthread_mutex_lock(&amp;lock);
// 임계 구역
balance += 50;
pthread_mutex_unlock(&amp;lock);</code></pre>
<h3 id="세마포어">세마포어</h3>
<ul>
<li>정수 값을 기반으로 P(Wait) / V(Signal) 연산 수행.</li>
<li>자원 요청 시 값 감소, 반납 시 값 증가.</li>
<li>값이 0이면 대기 큐에서 block.</li>
</ul>
<pre><code class="language-c">sem_t sem;
sem_init(&amp;sem, 0, 3);   // 초기값 3 → 동시 접근 3개 허용

sem_wait(&amp;sem);         // 자원 요청
// 임계 구역
sem_post(&amp;sem);         // 자원 반납</code></pre>
<hr />
<h2 id="시나리오-예시">시나리오 예시</h2>
<h3 id="뮤텍스-1">뮤텍스</h3>
<ul>
<li><strong>은행 계좌 입출금</strong>: balance 값이 동시에 수정되지 않도록 보호.</li>
</ul>
<h3 id="세마포어-1">세마포어</h3>
<ul>
<li><strong>식당 의자 문제</strong>: 의자가 3개뿐일 때 동시에 3명까지만 입장 가능.</li>
<li><strong>생산자–소비자 문제</strong>: 버퍼가 비었을 때/가득 찼을 때 대기 처리.</li>
</ul>
<hr />
<h2 id="세마포어에서-누가-깨어나느냐">세마포어에서 "누가 깨어나느냐?"</h2>
<ul>
<li><code>sem_wait</code>에서 대기 중인 프로세스/스레드는 커널이 관리하는 대기 큐에 들어감.</li>
<li><code>sem_post</code> 호출 시:<ul>
<li><strong>일반적으로</strong>: FIFO 순서로 깨움.</li>
<li><strong>운영체제 스케줄러 정책</strong>이 개입할 수 있음 → 우선순위가 높은 스레드가 먼저 깨일 수도 있음.</li>
</ul>
</li>
</ul>
<hr />
<h2 id="요약">요약</h2>
<ul>
<li>뮤텍스 = 단일 자원 보호 (한 명만 들어가는 화장실 열쇠).</li>
<li>세마포어 = 여러 자원 보호 (N개의 의자, 선착순으로 앉음).</li>
<li>둘 다 프로세스/스레드 환경에서 모두 사용 가능.</li>
</ul>