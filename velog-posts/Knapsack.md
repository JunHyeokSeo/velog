<h1 id="보석-배낭-문제-knapsack-problem---정확한-무게-조합-찾기">보석 배낭 문제 (Knapsack Problem) - 정확한 무게 조합 찾기</h1>
<h2 id="문제">문제</h2>
<p>보석의 무게와 가격이 주어졌을 때, 제한된 무게 안에서 최대 가격이 되도록 보석을 담는 문제.<br />가방에 담을 수 있는 보석의 무게는 <strong>최대 20kg</strong>이며, <strong>각 보석은 한 개씩만 존재</strong>한다.</p>
<hr />
<h2 id="풀이">풀이</h2>
<h3 id="조건문-설명">조건문 설명</h3>
<h4 id="j--weightsi"><code>j &lt; weights[i]</code></h4>
<p>잔여 무게가 현재 보석의 무게보다 적어 보석을 담을 수 없음을 의미한다.<br />즉, 현재 보석을 고려하기 전에 <strong>가방에 넣을 공간이 부족한 경우</strong>를 필터링한다.</p>
<h4 id="dpi---1j---weightsi--0--j--weightsi--dpi---1j--0"><code>dp[i - 1][j - weights[i]] != 0 || j == weights[i] || dp[i - 1][j] != 0</code></h4>
<p>보석을 추가할 수 있는 경우를 체크하는 핵심 조건문이다.</p>
<ol>
<li><p><strong><code>dp[i - 1][j - weights[i]] != 0</code></strong>  </p>
<ul>
<li><strong>이전 단계에서 무게 <code>j - weights[i]</code>를 만들 수 있는 경우</strong>, 현재 보석을 추가할 수 있음을 의미한다.</li>
<li>즉, 현재 보석을 더했을 때 유효한 조합이 되는지를 확인하는 조건이다.</li>
</ul>
</li>
<li><p><strong><code>j == weights[i]</code></strong>  </p>
<ul>
<li>현재 보석 하나만을 넣었을 때 정확히 무게 <code>j</code>가 되는 경우를 허용하기 위한 조건이다.</li>
<li>예를 들어, <code>j = 6</code>이고 <code>weights[i] = 6</code>이라면, 해당 보석만으로 정확히 무게 <code>6</code>을 만들 수 있다.</li>
</ul>
</li>
<li><p><strong><code>dp[i - 1][j] != 0</code></strong>  </p>
<ul>
<li>이전 단계에서 이미 무게 <code>j</code>를 만들 수 있었다면, <strong>그 값을 유지해야 한다</strong>.</li>
<li>즉, 현재 보석을 추가하지 않아도 최대 가치를 유지할 수 있음을 의미한다.</li>
</ul>
</li>
</ol>
<h3 id="조건문의-역할">조건문의 역할</h3>
<p>이 조건문을 사용하면, <strong>정확한 무게 조합이 가능한 경우에만 보석을 추가하도록 제한</strong>할 수 있다.<br />이로 인해 <strong>무게 제한을 초과하지 않으면서도, 보석의 조합이 정확하게 일치하는 경우에만 업데이트되는 DP 테이블을 유지</strong>할 수 있다.</p>
<hr />
<h2 id="코드">코드</h2>
<p>다음은 해당 문제를 해결하는 <strong>Java 코드</strong>이다.</p>
<pre><code class="language-java">import java.util.*;

public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);

        int n = 10; // 보석 개수
        int maxWeight = 20; // 최대 무게
        int[] weights = {0, 2, 6, 2, 3, 4, 5, 4, 6, 7, 10}; // 보석 무게 (1-based index)
        int[] values = {0, 3, 5, 4, 2, 3, 3, 2, 6, 9, 8}; // 보석 가치

        int[][] dp = new int[n + 1][maxWeight + 1];

        for (int i = 1; i &lt;= n; i++) {
            for (int j = 0; j &lt;= maxWeight; j++) {
                if (j &lt; weights[i]) {
                    dp[i][j] = dp[i - 1][j]; // 보석을 담을 수 없음
                }
                else if (dp[i - 1][j - weights[i]] != 0 || j == weights[i] || dp[i - 1][j] != 0) {
                    dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - weights[i]] + values[i]);
                }
            }
        }

        System.out.print(&quot;DP\t&quot;);
        for (int j = 0; j &lt;= maxWeight; j++) {
            System.out.print(j + &quot;\t&quot;);
        }
        System.out.println();

        for (int i = 1; i &lt;= n; i++) {
            System.out.print(i + &quot;\t&quot;);
            for (int j = 0; j &lt;= maxWeight; j++) {
                if (dp[i][j] == 0)
                    System.out.print(&quot;-\t&quot;);
                else
                    System.out.print(dp[i][j] + &quot;\t&quot;);
            }
            System.out.println();
        }

        sc.close();
    }
}</code></pre>
<h2 id="결과">결과</h2>
<p>위 코드를 실행하면 다음과 같은 DP 테이블이 출력된다.
각 행은 1<del>10번째 보석까지 고려했을 때,
각 열은 현재 가방의 허용 무게(0</del>20kg)에서의 최대 가격을 의미한다.</p>
<pre><code>DP    0    1    2    3    4    5    6    7    8    9    10    11    12    13    14    15    16    17    18    19    20    
1    -    -    3    -    -    -    -    -    -    -    -    -    -    -    -    -    -    -    -    -    -    
2    -    -    3    -    -    -    5    -    8    -    -    -    -    -    -    -    -    -    -    -    -    
3    -    -    4    -    7    -    5    -    9    -    12    -    -    -    -    -    -    -    -    -    -    
4    -    -    4    2    7    6    5    9    9    7    12    11    -    14    -    -    -    -    -    -    -    
5    -    -    4    2    7    6    7    9    10    9    12    12    12    14    15    14    -    17    -    -    -    
6    -    -    4    2    7    6    7    9    10    10    12    12    12    14    15    15    15    17    17    18    17    
7    -    -    4    2    7    6    7    9    10    10    12    12    12    14    15    15    15    17    17    18    17    
8    -    -    4    2    7    6    7    9    10    10    13    12    13    15    16    16    18    18    18    20    21    
9    -    -    4    2    7    6    7    9    10    13    13    16    15    16    18    19    19    22    21    22    24    
10    -    -    4    2    7    6    7    9    10    13    13    16    15    16    18    19    19    22    21    22    24    </code></pre>