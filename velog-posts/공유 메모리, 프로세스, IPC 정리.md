<h2 id="프로세스와-fork">프로세스와 fork</h2>
<ul>
<li>프로세스는 독립된 메모리 공간을 가진 실행 단위이다.</li>
<li><code>fork()</code> 호출 시 부모 프로세스를 복제하여 자식 프로세스를 생성한다.<ul>
<li>부모 프로세스: <code>fork()</code>가 자식의 PID(양수)를 반환.</li>
<li>자식 프로세스: <code>fork()</code>가 0을 반환.</li>
<li>실패: -1 반환.</li>
</ul>
</li>
<li>부모와 자식은 <code>fork()</code> 이후의 코드를 동시에 실행한다.</li>
<li>동작 순서는 운영체제의 스케줄러가 결정한다.</li>
</ul>
<h2 id="공유-메모리-shared-memory">공유 메모리 (Shared Memory)</h2>
<ul>
<li>IPC(Inter-Process Communication)의 한 방식으로, 여러 프로세스가 동일한 메모리 영역을 매핑하여 사용할 수 있다.</li>
<li>가장 빠른 IPC 방식이지만, 동시에 접근 시 동기화 문제가 발생할 수 있으므로 세마포어/뮤텍스와 함께 사용된다.</li>
</ul>
<h3 id="주요-함수">주요 함수</h3>
<ul>
<li><code>shmget(key_t key, size_t size, int shmflg)</code><ul>
<li>공유 메모리 세그먼트를 생성하거나 획득한다.</li>
</ul>
</li>
<li><code>shmat(int shmid, const void *shmaddr, int shmflg)</code><ul>
<li>공유 메모리를 프로세스 주소 공간에 attach한다.</li>
</ul>
</li>
<li><code>shmdt(const void *shmaddr)</code><ul>
<li>공유 메모리를 detach한다.</li>
</ul>
</li>
<li><code>shmctl(int shmid, int cmd, struct shmid_ds *buf)</code><ul>
<li>공유 메모리 제어 함수. <code>IPC_RMID</code>로 세그먼트 삭제 가능.</li>
</ul>
</li>
</ul>
<h3 id="데이터-타입">데이터 타입</h3>
<ul>
<li><code>key_t</code>: IPC 키 값.</li>
<li><code>pid_t</code>: 프로세스 식별자.</li>
</ul>
<h2 id="ipc-inter-process-communication">IPC (Inter-Process Communication)</h2>
<ul>
<li>프로세스 간 통신을 위한 메커니즘.</li>
<li>각 프로세스는 독립적인 주소 공간을 가지므로 직접적인 데이터 공유가 불가능하다. 따라서 IPC 기법이 필요하다.</li>
</ul>
<h3 id="주요-ipc-방식">주요 IPC 방식</h3>
<ol>
<li><p>파이프 (Pipe)</p>
<ul>
<li>부모-자식 간 단방향 통신.</li>
<li>예시: 쉘 파이프라인 (<code>ls | grep txt</code>).</li>
</ul>
</li>
<li><p>FIFO (Named Pipe)</p>
<ul>
<li>파일 시스템에 특수 파일로 존재.</li>
<li>관계 없는 프로세스 간 통신 가능.</li>
</ul>
</li>
<li><p>메시지 큐 (Message Queue)</p>
<ul>
<li>커널이 관리하는 메시지 단위 큐.</li>
<li>메시지 단위로 송수신 가능.</li>
</ul>
</li>
<li><p>공유 메모리 (Shared Memory)</p>
<ul>
<li>동일한 메모리 영역을 여러 프로세스가 공유.</li>
<li>속도가 빠르지만 동기화 필요.</li>
</ul>
</li>
<li><p>세마포어 (Semaphore)</p>
<ul>
<li>공유 자원 접근을 제어하기 위한 동기화 기법.</li>
<li>자체적으로 데이터 전달 기능은 없으며 주로 공유 메모리와 함께 사용된다.</li>
</ul>
</li>
<li><p>소켓 (Socket)</p>
<ul>
<li>네트워크 기반 통신.</li>
<li>동일 호스트 내부 또는 원격 호스트 간 통신 가능.</li>
<li>가장 범용적인 IPC 방식.</li>
</ul>
</li>
</ol>
<h2 id="ipc-플래그">IPC 플래그</h2>
<ul>
<li><code>IPC_CREAT</code>: 존재하지 않으면 생성.</li>
<li><code>IPC_EXCL</code>: <code>IPC_CREAT</code>와 함께 사용. 이미 존재하면 에러 반환.</li>
</ul>
<h2 id="권한-비트">권한 비트</h2>
<ul>
<li>IPC 자원의 접근 권한은 파일과 동일하게 8진수 권한 비트를 사용한다.</li>
<li>예: <code>0666</code> → <code>rw-rw-rw-</code></li>
<li>앞의 0은 8진수 리터럴임을 의미한다.</li>
</ul>