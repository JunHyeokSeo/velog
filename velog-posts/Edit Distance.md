<h1 id="문자열-편집-거리-edit-distance">문자열 편집 거리 (Edit Distance)</h1>
<h2 id="1-문제-설명"><strong>1. 문제 설명</strong></h2>
<p>두 개의 문자열 <code>A</code>, <code>B</code>가 주어졌을 때, <code>A</code>를 <code>B</code>로 변환하는 데 필요한 <strong>최소 연산 횟수</strong>(편집 거리)를 구하는 문제이다.</p>
<p>사용할 수 있는 연산은 <strong>세 가지</strong>이다.</p>
<ol>
<li><strong>삽입 (Insertion)</strong> → 원하는 위치에 문자 추가  </li>
<li><strong>삭제 (Deletion)</strong> → 특정 위치의 문자 제거  </li>
<li><strong>수정 (Replacement)</strong> → 특정 위치의 문자를 다른 문자로 변경  </li>
</ol>
<h3 id="예제">예제</h3>
<pre><code>A = &quot;SABSBA&quot;
B = &quot;ABABSA&quot;</code></pre><p>가능한 연산 방법:</p>
<pre><code>&quot;SABSBA&quot; → &quot;ABSBSA&quot; → &quot;ABABSA&quot;  (총 2번 수정)</code></pre><p>편집 거리는 <code>2</code>가 된다.</p>
<hr />
<h2 id="2-해결-방법-dp"><strong>2. 해결 방법 (DP)</strong></h2>
<h3 id="-1-dp-정의">** 1) DP 정의**</h3>
<ul>
<li><code>dp[i][j]</code> = <code>A</code>의 처음 <code>i</code>개의 문자와 <code>B</code>의 처음 <code>j</code>개의 문자를 일치시키는 데 필요한 <strong>최소 편집 거리</strong></li>
</ul>
<hr />
<h3 id="-2-점화식">** 2) 점화식**</h3>
<p>각 <code>dp[i][j]</code>는 <strong>마지막 문자(A[i-1], B[j-1])를 기준으로 결정</strong>된다.</p>
<ol>
<li><p><strong>문자가 같은 경우 (<code>A[i-1] == B[j-1]</code>)</strong></p>
<pre><code class="language-java">dp[i][j] = dp[i-1][j-1];  // 변환 없이 유지</code></pre>
</li>
<li><p><strong>문자가 다른 경우 (<code>A[i-1] != B[j-1]</code>)</strong></p>
<ul>
<li><strong>삽입 (Insertion)</strong> → <code>dp[i][j-1] + 1</code></li>
<li><strong>삭제 (Deletion)</strong> → <code>dp[i-1][j] + 1</code></li>
<li><strong>수정 (Replacement)</strong> → <code>dp[i-1][j-1] + 1</code><pre><code class="language-java">dp[i][j] = Math.min(
  dp[i][j-1] + 1,  // 삽입 (Insertion)
  dp[i-1][j] + 1,  // 삭제 (Deletion)
  dp[i-1][j-1] + 1 // 수정 (Replacement)
);</code></pre>
</li>
</ul>
</li>
</ol>
<hr />
<h3 id="-3-초기값-설정">** 3) 초기값 설정**</h3>
<pre><code class="language-java">for (int i = 0; i &lt;= n; i++) dp[i][0] = i; // A를 B(&quot;&quot;)로 만들기 위해 삭제 연산 i번 필요
for (int j = 0; j &lt;= m; j++) dp[0][j] = j; // &quot;&quot;(빈 문자열)을 B로 만들기 위해 삽입 연산 j번 필요</code></pre>
<hr />
<h2 id="3-코드-구현-java"><strong>3. 코드 구현 (Java)</strong></h2>
<pre><code class="language-java">import java.util.*;

public class EditDistance {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        String A = sc.next();
        String B = sc.next();

        int n = A.length();
        int m = B.length();

        int[][] dp = new int[n + 1][m + 1];

        // 초기값 설정
        for (int i = 0; i &lt;= n; i++) dp[i][0] = i;
        for (int j = 0; j &lt;= m; j++) dp[0][j] = j;

        // DP 점화식 적용
        for (int i = 1; i &lt;= n; i++) {
            for (int j = 1; j &lt;= m; j++) {
                if (A.charAt(i - 1) == B.charAt(j - 1)) {
                    dp[i][j] = dp[i - 1][j - 1];  // 문자가 같으면 그대로 유지
                } else {
                    dp[i][j] = Math.min(
                        dp[i][j - 1] + 1,  // 삽입 (Insertion)
                        Math.min(dp[i - 1][j] + 1,  // 삭제 (Deletion)
                                 dp[i - 1][j - 1] + 1) // 수정 (Replacement)
                    );
                }
            }
        }

        System.out.println(&quot;최소 편집 거리: &quot; + dp[n][m]);
        sc.close();
    }
}</code></pre>
<hr />
<h2 id="4-핵심-원리"><strong>4. 핵심 원리</strong></h2>
<h3 id="-1-왜-대각선-dpi-1j-1이-수정replacement인가">** 1) 왜 대각선 (<code>dp[i-1][j-1]</code>)이 수정(Replacement)인가?**</h3>
<ul>
<li><code>dp[i-1][j-1]</code>은 <strong>이전까지의 최적 상태</strong>를 의미.</li>
<li><strong>문자가 다르면</strong> <code>A[i-1]</code>을 <code>B[j-1]</code>로 바꿔야 하므로, <strong>수정(Replace) 연산을 추가 (<code>+1</code>)</strong>.</li>
<li>따라서 점화식에 <code>dp[i-1][j-1] + 1</code>이 포함됨.</li>
</ul>
<h3 id="-2-삽입insertion과-삭제deletion-공식">** 2) 삽입(Insertion)과 삭제(Deletion) 공식**</h3>
<ul>
<li><code>dp[i][j-1] + 1</code> → <strong>B의 <code>j</code>번째 문자를 추가해야 하므로, B의 <code>j-1</code>까지 변환한 후 삽입</strong></li>
<li><code>dp[i-1][j] + 1</code> → <strong>A의 <code>i</code>번째 문자를 삭제해야 하므로, A의 <code>i-1</code>까지 변환한 후 삭제</strong></li>
</ul>
<h3 id="-3-편집-거리에서-삽입insertion과-삭제deletion의-동작-방식">** 3) 편집 거리에서 삽입(Insertion)과 삭제(Deletion)의 동작 방식**</h3>
<blockquote>
<h4 id="1️⃣-삽입insertion-dpij-1--1"><strong>1️⃣ 삽입(Insertion): <code>dp[i][j-1] + 1</code></strong></h4>
</blockquote>
<ul>
<li><strong>정의:</strong> A를 B로 변환할 때, <strong>B의 <code>j</code>번째 문자를 추가하는 연산</strong>  </li>
<li><strong>즉, B의 <code>j-1</code>번째까지는 이미 변환 완료된 상태에서 새로운 문자를 삽입하는 것!</strong><h4 id="-예제-1-abc-→-abcd-삽입">** 예제 1: <code>&quot;abc&quot;</code> → <code>&quot;abcd&quot;</code> (삽입)**</h4>
<pre><code class="language-plaintext">A = &quot;abc&quot;
B = &quot;abcd&quot;</code></pre>
<h4 id="연산-과정"><strong>연산 과정</strong></h4>
</li>
</ul>
<ol>
<li><code>&quot;abc&quot;</code> → <code>&quot;abcd&quot;</code> (<code>'d'</code> 추가)<h4 id="dp-테이블"><strong>DP 테이블</strong></h4>
<pre><code class="language-plaintext">a b c d
a 0 1 2 3
b 1 0 1 2
c 2 1 0 1</code></pre>
<code>dp[i][j-1]</code>은 <strong>A가 B의 <code>j-1</code>번째까지 변환된 상태</strong>  </li>
</ol>
<p><strong>B의 <code>j</code>번째 문자를 삽입하면 <code>dp[i][j-1] + 1</code>이 된다!</strong>  </p>
<blockquote>
<h4 id="2️⃣-삭제deletion-dpi-1j--1"><strong>2️⃣ 삭제(Deletion): <code>dp[i-1][j] + 1</code></strong></h4>
</blockquote>
<ul>
<li><strong>정의:</strong> A를 B로 변환할 때, <strong>A의 <code>i</code>번째 문자를 삭제하는 연산</strong>  </li>
<li><strong>즉, A의 <code>i-1</code>번째까지는 이미 변환 완료된 상태에서 A의 <code>i</code>번째 문자를 삭제하는 것!</strong><h4 id="-예제-2-abcd-→-abc-삭제">** 예제 2: <code>&quot;abcd&quot;</code> → <code>&quot;abc&quot;</code> (삭제)**</h4>
<pre><code class="language-plaintext">A = &quot;abcd&quot;
B = &quot;abc&quot;</code></pre>
<h4 id="연산-과정-1"><strong>연산 과정</strong></h4>
</li>
</ul>
<ol>
<li><code>&quot;abcd&quot;</code> → <code>&quot;abc&quot;</code> (<code>'d'</code> 삭제)<h4 id="dp-테이블-1"><strong>DP 테이블</strong></h4>
<pre><code class="language-plaintext">a b c 
a 0 1 2 
b 1 0 1 
c 2 1 0
d 3 2 1   &lt;-- &quot;d&quot;를 삭제해야 하므로, dp[i-1][j] + 1</code></pre>
<code>dp[i-1][j]</code>은 <strong>A의 <code>i-1</code>번째까지 변환된 상태</strong><br /><strong>A의 <code>i</code>번째 문자를 삭제하면 <code>dp[i-1][j] + 1</code>이 된다!</strong></li>
</ol>
<h3 id="4--삽입과-삭제의-직관적인-이해">4) ** 삽입과 삭제의 직관적인 이해**</h3>
<ul>
<li><strong>삽입(Insertion)</strong> → &quot;B에 새로운 문자를 추가하는 연산&quot;  </li>
<li><strong>삭제(Deletion)</strong> → &quot;A의 특정 문자를 제거하는 연산&quot;  </li>
<li><strong>각각 기존 최적 상태에서 한 번의 연산(<code>+1</code>)을 추가하는 개념</strong>  </li>
</ul>
<hr />
<h2 id="5-예제-테스트"><strong>5. 예제 테스트</strong></h2>
<h3 id="입력"><strong>입력</strong></h3>
<pre><code>SABSBA
ABABSA</code></pre><h3 id="출력"><strong>출력</strong></h3>
<pre><code>최소 편집 거리: 3</code></pre><p>이 경우, <code>&quot;SABSBA&quot;</code>를 <code>&quot;ABABSA&quot;</code>로 변환하는 최소 연산 횟수는 <code>3</code>이다.</p>
<hr />
<h2 id="6-시간-복잡도"><strong>6. 시간 복잡도</strong></h2>
<ul>
<li><strong>O(n × m)</strong> (이중 루프 사용)</li>
<li>문자열 길이가 길어도 DP를 사용하면 효율적으로 해결 가능!</li>
</ul>
<hr />
<h2 id="7-결론"><strong>7. 결론</strong></h2>
<p>✔ <strong>문자열 편집 거리 문제는 동적 프로그래밍(DP)으로 해결 가능!</strong><br />✔ <strong>편집 거리 점화식을 올바르게 이해하면 최적의 알고리즘을 설계할 수 있다.</strong><br />✔ <strong>초기값 설정(<code>dp[i][0] = i</code>, <code>dp[0][j] = j</code>)을 통해 삭제/삽입 연산을 고려해야 한다.</strong><br />✔ <strong>시간 복잡도 O(n × m)으로 효율적으로 해결 가능하다.</strong>  </p>