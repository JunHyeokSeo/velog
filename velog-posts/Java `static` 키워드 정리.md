<h2 id="1-static이란">1. static이란?</h2>
<ul>
<li>클래스 로딩 시 메모리에 올라가는 <strong>클래스 레벨 멤버</strong></li>
<li>객체 인스턴스 없이도 접근 가능</li>
<li>대표적인 사용처: 유틸리티 메소드, 공용 상수, 싱글톤 인스턴스</li>
</ul>
<hr />
<h2 id="2-static-변수">2. static 변수</h2>
<ul>
<li>모든 인스턴스가 <strong>공유</strong>함</li>
<li>JVM의 <strong>Method Area</strong>(또는 Class 영역)에 저장됨</li>
<li>멀티스레드 환경에서는 <strong>동기화 주의</strong></li>
</ul>
<pre><code class="language-java">public class Example {
    static int counter = 0;

    public static void increment() {
        counter++;
    }
}</code></pre>
<hr />
<h2 id="3-static-메소드">3. static 메소드</h2>
<ul>
<li>클래스명으로 직접 호출 가능 (<code>Example.increment()</code>)</li>
<li><strong>this 사용 불가</strong></li>
<li><strong>인스턴스 변수/메소드 직접 접근 불가</strong></li>
<li>static 변수에는 직접 접근 가능</li>
</ul>
<pre><code class="language-java">public class Example {
    static int value = 10;

    public static void printValue() {
        System.out.println(value); // 가능
        // System.out.println(this.value); // 불가능
    }
}</code></pre>
<hr />
<h2 id="4-static-메소드의-제약-사항">4. static 메소드의 제약 사항</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td>인스턴스 멤버 접근</td>
<td>❌ 불가 (<code>this</code> 없음)</td>
</tr>
<tr>
<td>오버라이딩</td>
<td>❌ 불가 (메소드 숨김 발생)</td>
</tr>
<tr>
<td>상태 공유 문제</td>
<td>⚠ 동기화 없이 접근 시 Race Condition 발생 가능</td>
</tr>
<tr>
<td>테스트 용이성</td>
<td>❌ 낮음 (DI/Mock 어려움)</td>
</tr>
</tbody></table>
<hr />
<h2 id="5-메소드-숨김-method-hiding">5. 메소드 숨김 (Method Hiding)</h2>
<ul>
<li>static 메소드는 오버라이딩되지 않고 <strong>숨겨짐</strong></li>
<li>호출은 <strong>참조 변수 타입 기준</strong>으로 결정됨</li>
</ul>
<pre><code class="language-java">class Parent {
    static void greet() { System.out.println(&quot;Parent&quot;); }
}
class Child extends Parent {
    static void greet() { System.out.println(&quot;Child&quot;); }
}

Parent obj = new Child();
obj.greet(); // 출력: Parent (참조 타입 기준)</code></pre>
<hr />
<h2 id="6-static과-지역변수">6. static과 지역변수</h2>
<ul>
<li>static 메소드 내에서도 <strong>지역 변수 사용 가능</strong></li>
<li>지역 변수는 <strong>스택에 저장</strong>, static 여부와 무관하게 정상 동작</li>
</ul>
<pre><code class="language-java">public static void example() {
    int temp = 10; // OK
}</code></pre>
<hr />
<h2 id="7-race-condition">7. Race Condition</h2>
<ul>
<li>static 변수에 여러 스레드가 동시에 접근하면 발생</li>
<li>증상: 값이 덮어쓰기 되거나 불일치</li>
<li>해결 방법: <code>synchronized</code>, <code>Lock</code>, <code>AtomicInteger</code> 등 사용</li>
</ul>
<pre><code class="language-java">class Counter {
    static int count = 0;

    public static void increment() {
        count++; // 동기화 안 하면 Race Condition 발생 가능
    }
}</code></pre>
<hr />
<h2 id="8-사용-시-권장-사항">8. 사용 시 권장 사항</h2>
<ul>
<li>상태 없는 유틸 메소드에만 static 사용</li>
<li>상태를 갖거나 확장 가능성이 있는 경우 인스턴스 방식 권장</li>
<li>동기화, 테스트 용이성, 유지보수 관점에서 신중하게 사용</li>
</ul>
<hr />
<h2 id="✅-요약">✅ 요약</h2>
<table>
<thead>
<tr>
<th>구분</th>
<th>특징</th>
</tr>
</thead>
<tbody><tr>
<td>static 변수</td>
<td>클래스 로딩 시 공유, 모든 인스턴스에서 동일한 값</td>
</tr>
<tr>
<td>static 메소드</td>
<td>객체 없이 호출 가능, 인스턴스 멤버 접근 불가</td>
</tr>
<tr>
<td>메소드 숨김</td>
<td>다형성 미지원, 참조 타입 기준 호출</td>
</tr>
<tr>
<td>지역 변수</td>
<td>static 메소드 내에서도 사용 가능 (스택에 저장)</td>
</tr>
<tr>
<td>Race Condition</td>
<td>static 변수 접근 시 동기화 없으면 발생</td>
</tr>
</tbody></table>