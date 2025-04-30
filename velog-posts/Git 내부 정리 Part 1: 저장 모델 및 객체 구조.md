<h3 id="1-git-저장-모델-핵심-개념">1. Git 저장 모델 핵심 개념</h3>
<p>Git은 <strong>스냅샷 기반의 분산 버전 관리 시스템</strong>이다. Git은 파일의 변경 사항을 저장하는 것이 아니라, <strong>전체 디렉토리 구조의 스냅샷을 트리 객체로 저장</strong>하고, 이를 커밋 객체가 참조하는 방식으로 동작한다.</p>
<blockquote>
<p>Git의 핵심은 &quot;내용 기반 주소 지정(content-addressable storage)&quot;에 있다. 즉, 저장된 내용은 SHA-1 해시를 주소로 삼아 고정되고, 동일한 내용은 절대 중복 저장되지 않는다.</p>
</blockquote>
<h3 id="2-git-객체-4종">2. Git 객체 4종</h3>
<table>
<thead>
<tr>
<th>객체</th>
<th>설명</th>
<th>역할</th>
</tr>
</thead>
<tbody><tr>
<td>blob</td>
<td>파일 내용 자체</td>
<td>파일의 스냅샷 저장</td>
</tr>
<tr>
<td>tree</td>
<td>디렉토리 구조</td>
<td>파일명 → blob/tree 해시 매핑</td>
</tr>
<tr>
<td>commit</td>
<td>스냅샷의 포인터</td>
<td>tree를 참조하고 메타데이터 포함</td>
</tr>
<tr>
<td>tag</td>
<td>커밋에 붙이는 레이블</td>
<td>특정 커밋(릴리즈 등)에 이름 부여</td>
</tr>
</tbody></table>
<h4 id="▶-관련-질문-요약">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: tree를 구성할 때, blob에 부모 정보가 포함되나?</strong></p>
</blockquote>
<p>❌ 아니다. Git은 단방향 구조로, blob은 자신이 포함된 tree나 commit을 전혀 모른다. tree가 blob을 참조하며 경로를 부여한다.</p>
<h3 id="3-blob-객체">3. Blob 객체</h3>
<ul>
<li>실제 파일의 내용만 저장</li>
<li>동일한 내용은 동일한 SHA-1 해시</li>
<li>저장 형식: <code>blob &lt;size&gt;\0&lt;내용&gt;</code></li>
</ul>
<h4 id="▶-관련-질문-요약-1">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: 동일한 파일을 수정하면 blob은 덮어씌워지는가?</strong></p>
</blockquote>
<p>❌ 아니다. 수정되면 새로운 blob이 생성된다. 이전 blob은 사라지지 않고 <code>.git/objects</code>에 계속 존재한다.</p>
<blockquote>
<p><strong>Q: blob 해시만 알고 있으면 복구 가능한가?</strong></p>
</blockquote>
<p>✅ 예. blob 객체가 Git에 남아 있다면 <code>git cat-file -p &lt;해시&gt;</code>로 언제든 복구 가능하다.</p>
<blockquote>
<p><strong>Q: blob 해시만 있고 객체가 없으면?</strong></p>
</blockquote>
<p>❌ 불가능. SHA-1은 단방향 해시이기 때문에 해시만으로 내용을 역산할 수 없다.</p>
<h3 id="4-tree-객체">4. Tree 객체</h3>
<ul>
<li>디렉토리 한 단계를 표현함</li>
<li>각 항목은 <code>권한 blob|tree 해시  파일명</code> 형태로 구성됨</li>
<li>tree는 재귀적으로 하위 디렉토리(tree)를 포함할 수 있음</li>
</ul>
<pre><code class="language-bash">git cat-file -p &lt;tree-hash&gt;
# 예시 출력:
100644 blob &lt;hash&gt; a.txt
040000 tree &lt;hash&gt; src</code></pre>
<h4 id="▶-관련-질문-요약-2">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: 특정 커밋의 전체 파일 상태를 어떻게 복원하나?</strong></p>
</blockquote>
<p>커밋 → tree → 하위 tree/blob들을 재귀적으로 따라가며 모든 파일을 복원할 수 있다. Git은 이를 통해 워킹 디렉토리를 체크아웃함.</p>
<h3 id="5-commit-객체">5. Commit 객체</h3>
<ul>
<li>하나의 커밋은 하나의 tree를 가리킨다</li>
<li>메타데이터(작성자, 시간, 메시지) 포함</li>
<li>부모 커밋 해시 포함 (최초 커밋은 없음)</li>
</ul>
<pre><code class="language-bash">git cat-file -p &lt;commit-hash&gt;
# 출력 예시:
tree &lt;tree-hash&gt;
parent &lt;commit-hash&gt;
author ...
committer ...

&lt;메시지&gt;</code></pre>
<h4 id="▶-관련-질문-요약-3">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: commit은 실제로 무엇을 저장하나?</strong></p>
</blockquote>
<p>commit은 tree를 가리키는 포인터일 뿐이다. 파일 자체를 담고 있지 않다.</p>
<blockquote>
<p><strong>Q: commit만 있으면 디렉토리 전체 복원 가능한가?</strong></p>
</blockquote>
<p>✅ 그렇다. tree → blob을 따라가며 전체 파일 복원 가능.</p>
<h3 id="6-tag-객체">6. Tag 객체</h3>
<ul>
<li>특정 커밋에 의미 있는 이름을 붙인 객체</li>
<li>lightweight: 단순히 해시를 참조</li>
<li>annotated: 별도 객체가 생성되고, 작성자/메시지 포함</li>
</ul>
<pre><code class="language-bash">git tag v1.0              # lightweight
git tag -a v2.0 -m &quot;msg&quot;   # annotated</code></pre>
<h4 id="▶-관련-질문-요약-4">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: tag도 Git 객체인가?</strong></p>
</blockquote>
<p>✅ annotated tag는 <code>.git/objects</code>에 저장되는 별도의 tag 객체다. lightweight는 단순히 참조 포인터.</p>
<hr />