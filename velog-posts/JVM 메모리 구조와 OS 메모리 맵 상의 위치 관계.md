<p>JVM의 메모리 구조와 이를 OS 메모리 상의 어느 영역에 위치시키는가에 대한 궁금증이 들어서 정리했다.<br />단순히 JVM 내부 메모리 구조만 보는 것이 아니라, OS 프로세스 관점에서 어떤 영역에 어떤 방식으로 매핑되는지까지 확인하고 싶었다.</p>
<hr />
<h2 id="jvm은-os의-프로세스다">JVM은 OS의 프로세스다</h2>
<p>JVM은 결국 운영체제 위에서 실행되는 하나의 사용자 프로세스다.<br />따라서 JVM이 사용하는 메모리도 OS가 관리하는 프로세스 메모리 공간 안에서 할당되고 운영된다.</p>
<hr />
<h2 id="jvm의-메모리-구조-java-17-이상-기준">JVM의 메모리 구조 (Java 17 이상 기준)</h2>
<p>JVM은 내부적으로 다음과 같은 주요 메모리 영역을 가진다.</p>
<table>
<thead>
<tr>
<th>JVM 메모리 영역</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>Heap</td>
<td>new로 생성된 객체 저장, GC 대상 영역</td>
</tr>
<tr>
<td>Stack</td>
<td>스레드마다 생성되는 호출 스택, 지역 변수 저장</td>
</tr>
<tr>
<td>Metaspace</td>
<td>클래스 로딩 정보, static 메서드 정의 등</td>
</tr>
<tr>
<td>Code Cache</td>
<td>JIT 컴파일된 네이티브 코드 저장</td>
</tr>
<tr>
<td>Native Memory</td>
<td>JNI 등 네이티브 코드에서 사용하는 메모리</td>
</tr>
<tr>
<td>Direct ByteBuffer</td>
<td>Direct 메모리 (네이티브 I/O용)</td>
</tr>
<tr>
<td>Constant Pool</td>
<td>클래스별 상수 풀 (문자열, 메서드 참조 등)</td>
</tr>
</tbody></table>
<hr />
<h2 id="이-jvm-메모리들이-os의-어디에-위치하나">이 JVM 메모리들이 OS의 어디에 위치하나?</h2>
<p>OS 입장에서 보면, JVM의 각 메모리 영역은 다음과 같이 매핑된다.</p>
<table>
<thead>
<tr>
<th>JVM 메모리 구조</th>
<th>OS 메모리 상 위치</th>
<th>시스템 호출</th>
</tr>
</thead>
<tbody><tr>
<td>Heap</td>
<td>mmap 영역</td>
<td><code>mmap()</code></td>
</tr>
<tr>
<td>Stack</td>
<td>stack segment 또는 mmap</td>
<td><code>pthread_create</code>, <code>mmap()</code></td>
</tr>
<tr>
<td>Metaspace</td>
<td>mmap 영역</td>
<td><code>mmap()</code></td>
</tr>
<tr>
<td>Code Cache</td>
<td>mmap 영역</td>
<td><code>mmap()</code></td>
</tr>
<tr>
<td>Native Memory</td>
<td>mmap 또는 heap segment</td>
<td><code>mmap()</code>, <code>brk()</code></td>
</tr>
<tr>
<td>Constant Pool</td>
<td>Metaspace 또는 Heap</td>
<td>내부 관리</td>
</tr>
</tbody></table>
<hr />
<h2 id="mmap-영역이란">mmap 영역이란?</h2>
<p><code>mmap()</code>은 운영체제의 시스템 호출로, 파일이나 메모리 블록을 프로세스의 가상 주소 공간에 매핑하는 함수다.<br />JVM은 <code>malloc()</code> 대신 <code>mmap()</code>을 통해 대용량 메모리를 확보하고, 그 내부에서 Heap, Metaspace 등을 구성한다.</p>
<p>즉, JVM의 주요 메모리는 전통적인 OS heap segment가 아니라, <code>mmap()</code>을 통해 얻은 별도의 메모리 공간에 올라간다.</p>
<hr />
<h2 id="os-프로세스-메모리-맵과-jvm-메모리의-위치-관계">OS 프로세스 메모리 맵과 JVM 메모리의 위치 관계</h2>
<p>아래는 가상 메모리 상에서 JVM이 사용하는 영역들이 실제로 어떻게 배치되는지를 도식화한 것이다.</p>
<pre><code>↓ 낮은 주소
┌────────────────────────────────────────────┐
│ 코드 영역 (Text Segment)                     │ ← JVM 실행 바이너리 등
├────────────────────────────────────────────┤
│ 데이터 영역 (Data Segment)                    │ ← 초기화된 static 변수 등
├────────────────────────────────────────────┤
│ 힙 세그먼트 (Heap Segment - brk)              │ ← C의 malloc 등
├────────────────────────────────────────────┤
│ mmap 영역                                   │
│   ├─ JVM Heap                              │
│   ├─ Metaspace                             │
│   ├─ CodeCache                             │
│   ├─ DirectBuffer, Native Memory           │
│   └─ Shared Libraries                      │
├────────────────────────────────────────────┤
│ 스택 영역 (Stack Segment)                    │ ← 각 스레드별 Stack
└────────────────────────────────────────────┘
↑ 높은 주소</code></pre><hr />
<h2 id="낮은-주소-vs-높은-주소">낮은 주소 vs 높은 주소</h2>
<p>운영체제는 각 프로세스에 대해 고유한 가상 주소 공간을 할당한다.<br />이 주소는 낮은 주소(0x00000000)에서 시작하여 높은 주소(0xFFFFFFFF...)까지 증가한다.</p>
<ul>
<li>코드, 데이터, 힙 세그먼트는 <strong>낮은 주소부터 위로 확장</strong>된다.</li>
<li>스택은 <strong>높은 주소부터 아래로 확장</strong>된다.</li>
<li>mmap 영역은 중간에서 양쪽 방향으로 유동적으로 할당된다.</li>
</ul>
<p>이 구조 때문에 Heap과 Stack이 충돌하면 <code>StackOverflow</code>, <code>OutOfMemory</code> 같은 문제가 발생할 수 있다.</p>
<hr />
<h2 id="결론">결론</h2>
<ul>
<li>JVM의 모든 메모리는 결국 OS가 할당한 프로세스 가상 주소 공간 위에 존재한다.</li>
<li>Heap, Metaspace, Stack 등은 전통적인 OS 세그먼트와 다르게 대부분 mmap 영역에 위치한다.</li>
<li>따라서 JVM Heap은 OS의 &quot;heap segment&quot;가 아니며, mmap을 통해 별도로 관리되는 공간이다.</li>
</ul>