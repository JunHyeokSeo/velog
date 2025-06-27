<h2 id="1번-알고리즘-포함비포함-분기-방식">1번 알고리즘: 포함/비포함 분기 방식</h2>
<pre><code class="language-java">public static void f(int depth, int cnt) {
    if (depth == n + 1) {
        if (cnt == m) print();
        return;
    }

    list.add(depth);
    f(depth + 1, cnt + 1);
    list.remove(list.size() - 1);

    f(depth + 1, cnt);
}</code></pre>
<ul>
<li>깊이 <code>depth</code>에 대해, 해당 값을 포함하거나 포함하지 않는 두 가지 분기를 수행</li>
<li>모든 부분집합을 탐색하며 <code>cnt == m</code>인 경우만 출력</li>
<li>시간 복잡도: O(2ⁿ)</li>
<li>공간 복잡도: O(n)</li>
<li>불필요한 분기 많음</li>
</ul>
<h2 id="2번-알고리즘-조합형-dfs-방식">2번 알고리즘: 조합형 DFS 방식</h2>
<pre><code class="language-java">public static void f(int depth, int prevNum) {
    if (depth == m) {
        print();
        return;
    }

    for (int i = prevNum + 1; i &lt;= n; i++) {
        arr[depth] = i;
        f(depth + 1, i);
    }
}</code></pre>
<ul>
<li>조합 조건을 만족하는 원소만 탐색</li>
<li>depth는 현재까지 선택한 원소 수, prevNum은 마지막으로 선택한 수</li>
<li>시간 복잡도: O(nCm × m)</li>
<li>공간 복잡도: O(m)</li>
<li>불필요한 연산 없음</li>
</ul>
<h2 id="성능-비교-요약">성능 비교 요약</h2>
<table>
<thead>
<tr>
<th>항목</th>
<th>1번 (포함/비포함)</th>
<th>2번 (조합 DFS)</th>
</tr>
</thead>
<tbody><tr>
<td>시간 복잡도</td>
<td>O(2ⁿ)</td>
<td>O(nCm × m)</td>
</tr>
<tr>
<td>공간 복잡도</td>
<td>O(n)</td>
<td>O(m)</td>
</tr>
<tr>
<td>불필요한 호출</td>
<td>있음</td>
<td>없음</td>
</tr>
<tr>
<td>실행 속도</td>
<td>느림</td>
<td>빠름</td>
</tr>
</tbody></table>