<h1 id="최소-시간으로-거리-이동하기-문제-요약">최소 시간으로 거리 이동하기 문제 요약</h1>
<h2 id="문제-설명">문제 설명</h2>
<ul>
<li>한 지점에서 다른 지점까지 거리 x만큼 이동해야 한다.</li>
<li>매초 이동 속도를 1만큼 증가하거나 감소시킬 수 있다.</li>
<li>초기 속도는 0이다.</li>
<li>도착 지점에서는 속도가 반드시 1이어야 한다.</li>
<li>가장 빠르게 도착할 수 있는 시간을 구해야 한다.</li>
</ul>
<hr />
<h2 id="내가-했던-고민들">내가 했던 고민들</h2>
<h3 id="1-기본-전략-속도-증가-→-감속-→-도착">1. 기본 전략: 속도 증가 → 감속 → 도착</h3>
<ul>
<li>최단 시간으로 도달하기 위해서는 중간에 속도를 계속 증가시키고, 적절한 시점부터 속도를 줄여야 한다.</li>
<li>도착 시 속도가 1이 되도록 설계되어야 한다.</li>
</ul>
<h3 id="2-감속-거리-계산에-대한-오해">2. 감속 거리 계산에 대한 오해</h3>
<ul>
<li><code>(speed * (speed + 1)) / 2</code>는 <code>0 → speed</code>까지 가속하는 거리로도, <code>speed → 1</code>까지 감속하는 거리로도 해석 가능하다.</li>
<li>등차수열 합 공식을 감속에 그대로 적용해도 수치상 맞는 결과가 나온다.</li>
<li>수식 자체가 잘못된 것은 아니었지만, 적용 <strong>시점</strong>의 판단이 중요했다.</li>
</ul>
<h3 id="3-감속-판단-로직의-핵심-오류">3. 감속 판단 로직의 핵심 오류</h3>
<ul>
<li>내가 사용한 수식은 <code>dist + speed + a &gt; x</code> 형태였고, 여기서 <code>a = (speed * (speed + 1)) / 2</code>였다.</li>
<li>이 수식은 &quot;속도를 한 번 더 유지하고 그 이후 감속해도 x를 초과하는가?&quot;를 판단하고 있었다.</li>
<li>하지만 실제로는 &quot;지금 감속을 시작했을 때 x에 도달 가능한가?&quot;를 판단해야 한다.</li>
<li><code>dist + speed + a &gt; x</code>와 같은 조건으로 속도 유지가 필요한 시점에도 감속을 하여 x에 도착하는 시간이 길어지는 문제가 발생했다.</li>
</ul>
<hr />
<h2 id="중요하게-고려해야-했던-쟁점">중요하게 고려해야 했던 쟁점</h2>
<ol>
<li><p><strong>감속은 즉시 적용된다는 점</strong></p>
<ul>
<li>감속을 한다고 결정한 순간부터 감속된 속도로 이동하게 되므로, 현재 감속을 시작했을 때의 이동 가능 거리로 판단해야 한다.</li>
</ul>
</li>
<li><p><strong>감속 거리 계산 자체는 맞지만, 적용 방식이 잘못되면 판단을 흐릴 수 있음</strong></p>
<ul>
<li>감속 거리 = <code>(speed * (speed + 1)) / 2</code>는 올바른 수식이다.</li>
<li>그러나 &quot;지금 감속해도 되는가?&quot;를 판단할 때는 <code>dist + a &gt; x</code>로 비교해야 정확하다.</li>
</ul>
</li>
<li><p><strong>가속 조건 역시 다음 속도로 감속이 가능한지를 기준으로 판단해야 함</strong></p>
<ul>
<li>가속 후 감속이 가능한지를 판단하기 위해 <code>(speed + 1) * (speed + 2) / 2</code> 수식을 사용하는 것이 올바른 기준이 된다.</li>
</ul>
</li>
</ol>
<hr />
<h2 id="핵심-아이디어-요약">핵심 아이디어 요약</h2>
<ol>
<li><strong>속도는 최대한 빠르게 증가시키되, 도착 지점에 정확히 맞추기 위해서는 감속을 정확한 시점에 시작해야 한다.</strong></li>
<li><strong>감속 조건을 판단할 때는 현재 감속을 시작했을 경우의 거리만을 기준으로 삼아야 하며, 속도를 유지한 후 감속하는 형태는 실제 흐름과 맞지 않는다.</strong></li>
<li><strong>가속은 &quot;지금 속도를 1 올리고 다음 순간부터 감속해도 되는가?&quot;로 판단해야 하고, 이를 위해 (speed + 1)(speed + 2)/2를 기준 거리로 본다.</strong></li>
</ol>
<hr />
<h2 id="최종-정답-코드-java">최종 정답 코드 (Java)</h2>
<pre><code class="language-java">import java.util.Scanner;
public class Main {
    public static void main(String[] args) {
        Scanner sc = new Scanner(System.in);
        int x = sc.nextInt();

        int dist = 0;
        int speed = 0;
        int time = 0;
        while (dist &lt; x) {
            int diff = x - dist;

            if (diff &gt;= ((speed + 1) * (speed + 2)) / 2 )
                speed++;
            else if (diff &lt; (speed * (speed + 1)) / 2)
                speed--;

            dist += speed;

            time++;
        }

        System.out.println(time);
    }
}</code></pre>