<h2 id="배열-복사">배열 복사</h2>
<h3 id="아래-두-코드의-동작은-동일한가">아래 두 코드의 동작은 동일한가?</h3>
<pre><code class="language-java">-- System.arraycopy()
if (high + 1 - low &gt;= 0) 
    System.arraycopy(mergedArr, low, arr, low, high + 1 - low);

-- 루프
for (int idx = low; idx &lt;= high; idx++)
    arr[idx] = mergedArr[idx];</code></pre>
<p>동일한 동작을 수행하지만, 방식이 다름.
System.arraycopy()는 네이티브 최적화가 적용된 고속 메모리 복사.
for 루프는 자바 코드로 반복문을 통해 하나씩 복사.</p>
<h3 id="arraycopy">arraycopy</h3>
<p><strong>System.arraycopy()</strong>는 네이티브 최적화가 적용된 고속 메모리 복사.
<strong>JNI(Java Native Interface) 기반</strong>으로 네이티브 코드에서 실행되며, 성능 최적화가 되어 있음
Java의 System.arraycopy()는 내부적으로 <strong>C 또는 어셈블리 수준에서 최적화된 메모리 복사 연산을 수행</strong>하기 때문에 for 루프보다 훨씬 빠름
내부적으로 JVM의 <strong>Unsafe.copyMemory()</strong>와 유사한 방식으로 동작</p>
<h3 id="결론">결론</h3>
<p>두 코드의 동작은 동일하지만, <strong>System.arraycopy()</strong>가 <strong>JNI</strong> 기반으로 더 최적화된 방식임.</p>
<h2 id="얕은-복사shallow-copy와-깊은-복사deep-copy">얕은 복사(Shallow Copy)와 깊은 복사(Deep Copy)</h2>
<blockquote>
<p><strong>arraycopy</strong>와 <strong>for 루프</strong> 모두 얕은 복사(Shallow Copy)</p>
</blockquote>
<h3 id="얕은-복사shallow-copy">얕은 복사(Shallow Copy)</h3>
<p>객체의 <strong>참조(메모리 주소)만 복사</strong>하는 방식.
복사된 객체와 원본 객체가 <strong>같은 메모리 주소</strong>를 공유.
하나의 객체를 수정하면 원본에도 영향을 미침.
<strong>System.arraycopy(), Arrays.copyOf(), clone()</strong> 등으로 수행 가능.</p>
<h3 id="깊은-복사deep-copy">깊은 복사(Deep Copy)</h3>
<p>객체 자체를 <strong>새로운 메모리 공간에 완전히 복사</strong>하는 방식.
복사된 객체와 원본 객체가 <strong>독립적인 메모리 주소</strong>를 가짐
복사된 객체를 수정해도 원본에는 영향을 주지 않음.
직접 새 객체를 생성하여 필드를 개별적으로 복사하거나, clone()을 오버라이딩하여 수행.</p>
<h3 id="왜-systemarraycopy나-for-루프-복사는-얕은-복사인가">왜 System.arraycopy나 for 루프 복사는 얕은 복사인가?</h3>
<p>정수형 배열을 복사할 때 위 두 방법을 사용했다. 단순히 값을 복사하니까 깊은 복사라고 이해했지만 이는 얕은 복사였다.
얕은 복사(Shallow Copy)와 깊은 복사(Deep Copy)의 개념은 참조 방식에 따라 달라진다. </p>
<h3 id="기본형-배열-int-double-복사">기본형 배열 (int[], double[]) 복사</h3>
<pre><code class="language-java">int[] arr = {1, 2, 3, 4, 5};
int[] tmp = Arrays.copyOf(arr, arr.length); // 복사

tmp[0] = 100; // tmp 변경

System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5] (원본 영향 없음)
System.out.println(Arrays.toString(tmp)); // [100, 2, 3, 4, 5] (복사본만 변경됨)
</code></pre>
<p>tmp는 새로운 배열 메모리 공간을 가짐 → <code>arr과 독립적.</code>
이 경우 &quot;얕은 복사&quot;지만 깊은 복사와 동작 차이가 없음 → <code>&quot;값을 복사했기 때문&quot;.</code>
tmp를 수정해도 arr에는 영향 없음.
<strong>이 경우 &quot;얕은 복사&quot;지만 깊은 복사와 동작이 같아 헷갈릴 수 있음.</strong></p>
<h3 id="객체-배열string-object-복사">객체 배열(String[], Object[]) 복사</h3>
<pre><code class="language-java">String[] arr = {&quot;hello&quot;, &quot;world&quot;};
String[] tmp = Arrays.copyOf(arr, arr.length); // 얕은 복사

tmp[0] = &quot;changed&quot;;

System.out.println(Arrays.toString(arr)); // [&quot;hello&quot;, &quot;world&quot;] (원본 영향 없음)
System.out.println(Arrays.toString(tmp)); // [&quot;changed&quot;, &quot;world&quot;] (복사본만 변경됨)
</code></pre>
<p>String은 <strong>불변 객체(Immutable Object)</strong>이므로 동작이 기본형 배열과 비슷해 보임.
하지만, 만약 객체 내부의 속성을 수정할 경우, 원본에도 영향을 줄 수 있음.</p>
<h3 id="객체-배열의-속성-변경-얕은-복사의-문제점">객체 배열의 속성 변경 (얕은 복사의 문제점)</h3>
<pre><code class="language-java">class Person {
    String name;

    Person(String name) {
        this.name = name;
    }
}

public class Main {
    public static void main(String[] args) {
        Person[] arr = {new Person(&quot;Alice&quot;), new Person(&quot;Bob&quot;)};
        Person[] tmp = Arrays.copyOf(arr, arr.length); // 얕은 복사

        tmp[0].name = &quot;Changed&quot;; // tmp의 첫 번째 객체 변경

        System.out.println(arr[0].name); // &quot;Changed&quot; (원본도 변경됨!)
        System.out.println(tmp[0].name); // &quot;Changed&quot;
    }
}</code></pre>
<p>객체 배열에서 <strong>Arrays.copyOf()</strong> 또는 <strong>System.arraycopy()</strong>를 사용하면, 객체의 참조만 복사됨(즉, 같은 인스턴스를 가리킴).
<strong>tmp[0]</strong>의 <strong>name</strong>을 바꾸면, 원본 <strong>arr[0]</strong>의 <strong>name</strong>도 바뀜 → <code>이것이 얕은 복사의 문제점.</code></p>
<h2 id="arraycopy와-for-루프-성능비교">arraycopy와 for 루프 성능비교</h2>
<blockquote>
<p>14배의 속도 차이를 보인다.</p>
</blockquote>
<h3 id="arraycopy-1">arraycopy</h3>
<blockquote>
<p>System.arraycopy() 실행 시간: <code>175708 ns</code></p>
</blockquote>
<pre><code class="language-java">public class ArrayCopyBenchmark {
    public static void main(String[] args) {
        int size = 1000000; // 복사할 배열 크기
        int[] source = new int[size];
        int[] destination = new int[size];

        // 배열 초기화
        for (int i = 0; i &lt; size; i++) {
            source[i] = i;
        }

        long start = System.nanoTime();
        System.arraycopy(source, 0, destination, 0, size);
        long end = System.nanoTime();

        System.out.println(&quot;System.arraycopy() 실행 시간: &quot; + (end - start) + &quot; ns&quot;);
    }
}
</code></pre>
<h3 id="for-루프">for 루프</h3>
<blockquote>
<p>for 루프 실행 시간: <code>2480625 ns</code></p>
</blockquote>
<pre><code class="language-java">public class ForLoopCopyBenchmark {
    public static void main(String[] args) {
        int size = 1000000; // 복사할 배열 크기
        int[] source = new int[size];
        int[] destination = new int[size];

        // 배열 초기화
        for (int i = 0; i &lt; size; i++) {
            source[i] = i;
        }

        long start = System.nanoTime();
        for (int i = 0; i &lt; size; i++) {
            destination[i] = source[i];
        }
        long end = System.nanoTime();

        System.out.println(&quot;for 루프 실행 시간: &quot; + (end - start) + &quot; ns&quot;);
    }
}
</code></pre>