<h2 id="문제-정의">문제 정의</h2>
<p>자연수 1부터 N까지의 노드를 사용하여 만들 수 있는 <strong>서로 다른 이진 탐색 트리(BST)</strong> 의 개수를 구하는 문제이다.</p>
<hr />
<h2 id="핵심-개념">핵심 개념</h2>
<ul>
<li>BST는 <strong>왼쪽 서브트리 &lt; 루트 &lt; 오른쪽 서브트리</strong> 조건을 만족하는 이진 트리이다.</li>
<li>노드 값 자체가 아닌 <strong>노드 개수</strong>에 따라 구조의 경우의 수가 결정된다.</li>
<li>루트를 하나 고정하면, 그보다 작은 수들은 <strong>왼쪽 서브트리</strong>, 큰 수들은 <strong>오른쪽 서브트리</strong>에만 배치될 수 있다.</li>
</ul>
<hr />
<h2 id="점화식-유도">점화식 유도</h2>
<p><code>dp[n]</code>을 노드 n개로 만들 수 있는 BST의 개수라고 하자.<br />루트를 1부터 n까지 하나씩 선택해보면,</p>
<ul>
<li>루트를 i로 선택했을 때<ul>
<li>왼쪽 서브트리: 노드 개수 = i - 1 → 경우 수 = dp[i - 1]</li>
<li>오른쪽 서브트리: 노드 개수 = n - i → 경우 수 = dp[n - i]</li>
</ul>
</li>
</ul>
<p>그러므로 점화식은 다음과 같다:</p>
<pre><code>dp[n] = sum(dp[i - 1] * dp[n - i]) for i in 1..n</code></pre><hr />
<h2 id="초기-조건">초기 조건</h2>
<ul>
<li><code>dp[0] = 1</code>: 빈 트리도 하나의 경우로 간주</li>
<li><code>dp[1] = 1</code>: 하나의 노드로 만들 수 있는 트리는 하나</li>
</ul>
<hr />
<h2 id="코드-예시-java">코드 예시 (Java)</h2>
<pre><code class="language-java">public class UniqueBST {
    public int numTrees(int n) {
        int[] dp = new int[n + 1];
        dp[0] = 1;
        dp[1] = 1;

        for (int nodes = 2; nodes &lt;= n; nodes++) {
            for (int root = 1; root &lt;= nodes; root++) {
                int left = root - 1;
                int right = nodes - root;
                dp[nodes] += dp[left] * dp[right];
            }
        }

        return dp[n];
    }
}</code></pre>
<hr />
<h2 id="고찰">고찰</h2>
<ul>
<li><code>dp[n]</code>은 <strong>"n개의 노드로 만들 수 있는 BST 구조의 개수"</strong> 를 의미한다.</li>
<li>특정 루트를 기준으로 좌/우 서브트리의 <strong>노드 수만 고려</strong>하면 되며,
어떤 숫자가 들어가는지는 구조상 영향을 주지 않는다.</li>
<li>이 문제는 <strong>카탈란 수(Catalan Number)</strong> 와 동일한 구조를 가진다.</li>
</ul>