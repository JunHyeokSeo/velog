<p>Git 명령어 실행 시 <code>.git/</code> 내부가 어떻게 변하는지를 따라가며 설명합니다.</p>
<hr />
<h2 id="git-init"><code>git init</code></h2>
<pre><code class="language-bash">git init</code></pre>
<h3 id="변화"><strong>변화</strong></h3>
<ul>
<li><code>.git/</code> 디렉토리 생성</li>
<li><code>.git/HEAD</code> → 기본적으로 <code>ref: refs/heads/main</code></li>
<li><code>.git/objects/</code>, <code>.git/refs/</code>, <code>.git/config</code> 등이 초기화됨</li>
</ul>
<h3 id="확인"><strong>확인:</strong></h3>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/e6fbeee4-6060-4bc6-8f39-8d6c02e9964f/image.png" /></p>
<hr />
<h2 id="git-add-atxt"><code>git add a.txt</code></h2>
<pre><code class="language-bash">echo &quot;a&quot; &gt; a.txt
git add a.txt</code></pre>
<h3 id="변화-1"><strong>변화:</strong></h3>
<ul>
<li><code>a.txt</code> 내용이 blob 객체로 저장 → <code>.git/objects/</code>에 새 디렉토리 생성</li>
<li><code>.git/index</code>에 a.txt의 경로와 blob 해시 등록</li>
</ul>
<h3 id="확인-1"><strong>확인:</strong></h3>
<h4 id="atxt-생성-및-git-status-확인">a.txt 생성 및 git status 확인</h4>
<ul>
<li><code>a.txt</code> 생성 및 <code>git status</code> 확인</li>
<li><code>a.txt</code>는 <code>untracked</code> 상태로, 아직 <code>Git</code>의 관리 대상이 아님</li>
<li><code>.git/index</code>에 등록되지 않은 파일은 <strong>스테이징되지 않은 것으로 간주됨</strong></li>
<li>Git에서 커밋하려면 먼저 <code>git add</code>로 <code>index(스테이징 영역)</code> 에 등록해야 함
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/d0773c85-ec24-4eaa-95ba-8e65665120c3/image.png" /><h4 id="git-add-이후-내부-변화"><code>git add</code> 이후 내부 변화</h4>
</li>
<li><code>git add a.txt</code> 실행 후 <code>.git/index</code> 파일이 생성되고, <code>a.txt</code>의 <code>blob</code> 객체가 <code>.git/objects/78/...</code> 경로에 저장됨</li>
<li>이때 <code>blob</code> 객체의 해시는 <code>a.txt</code>의 파일 내용에 기반한 <strong>SHA-1</strong> 값으로 결정됨</li>
<li>Git은 이 시점부터 a.txt를 <strong>추적 중인 파일(tracked file)</strong>로 인식하며, <strong>커밋 가능 상태(staged)</strong>로 간주함
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/7d2e853b-4c23-45aa-b852-dd3085536ba4/image.png" /></li>
</ul>
<h4 id="blob-객체-및-index-내용-확인">blob 객체 및 index 내용 확인</h4>
<ul>
<li><code>git ls-files --stage</code> 명령으로 <code>a.txt</code>의 <code>blob 해시</code> 및 <code>모드(권한 정보)</code>가 <code>.git/index</code>에 등록되었음을 확인 가능</li>
<li><code>git cat-file -p &lt;해시&gt;</code> 명령을 통해 해당 blob 객체의 실제 파일 내용 확인 가능</li>
<li><code>git cat-file -t &lt;해시&gt;</code> 결과가 blob으로 출력되며, 이는 해당 해시가 파일 내용 객체임을 의미함
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/24a355b2-0b10-4a9e-b415-62c7d0e40dc2/image.png" /></li>
</ul>
<h3 id="▶-관련-질문-요약">▶ 관련 질문 요약</h3>
<blockquote>
<p>Q: <code>index</code>는 언제 생기고, 어떤 역할을 하나?</p>
</blockquote>
<p>✅ git init 시 파일로 존재하며, git add 실행 시 실질적인 blob 경로 ↔ 파일명 매핑 정보가 등록됨</p>
<hr />
<h2 id="git-commit"><code>git commit</code></h2>
<pre><code class="language-bash">git commit -m &quot;test commit&quot;</code></pre>
<h3 id="변화-2"><strong>변화:</strong></h3>
<ul>
<li><code>.git/objects/</code>에 tree + commit 객체 추가</li>
<li><code>.git/refs/heads/main</code> 생성 → 커밋 해시 저장</li>
</ul>
<h3 id="확인-2"><strong>확인:</strong></h3>
<h4 id="커밋-전-상태-브랜치-참조-없음">커밋 전 상태: 브랜치 참조 없음</h4>
<ul>
<li><code>.git/refs/heads</code> 디렉토리가 <code>비어 있음</code> → <code>아직 커밋이 없다는 뜻</code></li>
<li><code>git status</code> 상에서 <code>a.txt</code>는 <code>staged</code> 상태로, <strong>커밋 대상에 포함됨</strong></li>
<li>이 시점에서 <code>HEAD</code>는 여전히 <code>refs/heads/main</code>을 가리키고 있지만, <strong>해당 파일은 존재하지 않음</strong>
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/bd6e0b79-b847-4e6a-92cf-fde8805b6a13/image.png" /><h4 id="최초-커밋-생성-및-브랜치-포인터-등록">최초 커밋 생성 및 브랜치 포인터 등록</h4>
</li>
<li><code>git commit</code> 실행 시 <code>.git/objects/</code>에 <code>tree</code>와 <code>commit</code> 객체가 생성됨</li>
<li><code>.git/refs/heads/main</code> 파일이 생기고, <code>main</code> 브랜치가 <code>새 커밋 해시</code>를 가리킴</li>
<li>커밋 메시지, 변경 파일, 파일 모드 등은 commit 객체에 저장됨
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/b99c1d9b-f9f1-4219-aeae-6b19fa970748/image.png" /><h4 id="커밋-객체-및-tree-구조-확인">커밋 객체 및 tree 구조 확인</h4>
</li>
<li><code>git cat-file -p &lt;commit&gt;</code> → commit 객체의 내부 구조 확인 가능<ul>
<li>참조하는 <code>tree 해시</code></li>
<li>작성자, 커밋 시간, 메시지 등 포함</li>
</ul>
</li>
<li><code>git cat-file -p &lt;tree&gt;</code> → 커밋 당시 파일 구조 확인 가능<ul>
<li>파일 경로와 연결된 <code>blob 해시</code> 포함</li>
</ul>
</li>
<li><code>tree</code>가 <code>blob(a.txt)</code>을 참조하고 있음을 확인할 수 있음
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/9ee47224-aa7b-4a64-a91e-70b7e848e00f/image.png" /></li>
</ul>
<hr />
<h2 id="git-checkout--b-featuretest1"><code>git checkout -b feature/test1</code></h2>
<pre><code class="language-bash">git checkout -b feature/test1</code></pre>
<h3 id="변화-3"><strong>변화:</strong></h3>
<ul>
<li><code>.git/refs/heads/feature/test1</code> 생성</li>
<li><code>.git/HEAD</code>가 해당 브랜치를 참조하도록 변경됨</li>
</ul>
<h3 id="확인-3"><strong>확인:</strong></h3>
<h4 id="브랜치-생성-및-head-이동-확인">브랜치 생성 및 HEAD 이동 확인</h4>
<ul>
<li><code>git checkout -b feature/test1</code> 실행 시 새로운 브랜치가 생성되고 HEAD가 해당 브랜치를 가리키게 됨</li>
<li><code>.git/HEAD</code>가 <code>refs/heads/feature/test1</code>을 가리키고 있으며, <code>main</code> 브랜치를 그대로 복사한 상태로 시작
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/430ff8a4-7821-49d0-8dd9-5812aed66114/image.png" /></li>
</ul>
<hr />
<h2 id="feature-브랜치에서-새-커밋-생성">feature 브랜치에서 새 커밋 생성</h2>
<pre><code class="language-bash">echo &quot;b&quot; &gt; b.txt
git add b.txt
git commit -m &quot;add b.txt&quot;</code></pre>
<h3 id="변화-4"><strong>변화:</strong></h3>
<ul>
<li>b.txt의 blob 객체 생성</li>
<li>새로운 tree 및 commit 객체 추가</li>
<li><code>feature/test1</code> 브랜치가 새 커밋을 가리킴</li>
</ul>
<h3 id="확인-4">확인</h3>
<h4 id="featuretest1-브랜치에서-btxt-추가-및-커밋">feature/test1 브랜치에서 b.txt 추가 및 커밋</h4>
<ul>
<li><code>echo &quot;b&quot; &gt; b.txt</code>로 새로운 파일 생성 후 <code>git add</code>, <code>git commit</code> 수행</li>
<li>커밋 해시 <code>5733ce...</code>가 생성되며 feature/test1 브랜치가 해당 커밋을 가리키게 됨</li>
<li><code>.git/refs/heads/feature/test1</code>가 새로운 커밋 해시를 포함하고, <code>main</code>은 이전 상태 유지
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/e54db938-b024-41c1-b1d8-03b6cc9dba05/image.png" /></li>
</ul>
<h4 id="브랜치-간-커밋-분기-상태-확인">브랜치 간 커밋 분기 상태 확인</h4>
<ul>
<li><code>git log --oneline --graph</code> 실행으로 main과 feature/test1이 서로 다른 커밋을 가리키고 있음을 확인</li>
<li>분기 이후 커밋이 독립적으로 생성되어 병합 대상이 될 수 있는 상태
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/024c2862-50b3-4f8e-9a53-e59a05a52bea/image.png" />
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/6a4fbd4c-ebac-42b6-948b-4a23639b3a9f/image.png" /></li>
</ul>
<hr />
<h2 id="git-switch-main--git-merge-featuretest1"><code>git switch main &amp;&amp; git merge feature/test1</code></h2>
<pre><code class="language-bash">git switch main
git merge feature/test1</code></pre>
<h3 id="상황-a-fast-forward-병합-발생"><strong>상황 A: Fast-forward 병합 발생</strong></h3>
<ul>
<li>main이 feature의 조상이므로, 새 커밋 없이 main → feature 커밋으로 포인터만 이동</li>
</ul>
<h3 id="변화-5"><strong>변화:</strong></h3>
<ul>
<li><code>.git/refs/heads/main</code>이 feature의 커밋 해시로 덮어씀</li>
</ul>
<h3 id="확인-5"><strong>확인:</strong></h3>
<h4 id="fast-forward-병합-결과-확인">Fast-forward 병합 결과 확인</h4>
<ul>
<li><code>git merge feature/test1</code> 실행 시 main 브랜치가 해당 브랜치의 조상이기 때문에 <code>fast-forward</code> 방식으로 병합됨</li>
<li>별도의 병합 커밋 없이 <code>main</code> 브랜치의 포인터가 <code>feature/test1</code>의 최신 커밋으로 이동</li>
<li><code>b.txt</code>의 변경 사항이 그대로 반영되며 Git은 병합 과정을 <strong>단순 포인터 이동</strong>으로 처리함
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/ee11dec8-aa70-460e-a248-dd9739d3a755/image.png" /><h4 id="커밋-그래프-확인-fast-forward-병합">커밋 그래프 확인 (fast-forward 병합)</h4>
</li>
<li><code>git log --oneline --graph</code> 결과, <code>main</code>과 <code>feature/test1</code> 브랜치가 동일한 커밋 <code>5733ce1</code>을 가리킴</li>
<li><code>HEAD</code>도 해당 커밋을 가리키며 분기 흔적 없이 <code>선형 히스토리로 유지</code>됨
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/1b3411ab-679f-4dd1-b13e-a3f61641287a/image.png" /></li>
</ul>
<h3 id="▶-관련-질문-요약-1">▶ 관련 질문 요약</h3>
<blockquote>
<p>Q: merge 시 왜 새 커밋이 생기지 않고 포인터만 이동했나?</p>
</blockquote>
<p>✅ Fast-forward merge 조건이 만족됨: main이 feature 브랜치의 조상이므로 별도 병합 커밋 없이 포인터만 이동함</p>
<hr />
<h2 id="병합-커밋-강제-생성---no-ff">병합 커밋 강제 생성 (<code>--no-ff</code>)</h2>
<pre><code class="language-bash">git reset --hard &lt;merge 전 커밋&gt;
git merge --no-ff feature/test1 -m &quot;merge feature/test1 into main&quot;</code></pre>
<h3 id="변화-6"><strong>변화:</strong></h3>
<ul>
<li>새로운 병합 <code>commit(C3)</code> 생성</li>
<li>부모는 <code>main 이전 커밋(C1)</code>과 <code>feature 커밋(C2)</code></li>
<li><code>.git/objects/</code>에 새로운 commit + tree 객체 저장됨</li>
</ul>
<h3 id="확인-6"><strong>확인:</strong></h3>
<h4 id="병합-커밋-생성---no-ff-옵션-사용">병합 커밋 생성 (<code>--no-ff</code> 옵션 사용)</h4>
<ul>
<li><code>git merge --no-ff feature/test1 -m &quot;merge feature/test1 into main&quot;</code> 명령은 fast-forward 병합을 방지하고 병합 커밋을 강제로 생성함</li>
<li>커밋 해시 <code>d04d532</code>가 새로 생성되며 두 부모를 가진 merge commit이 <code>.git/refs/heads/main</code>을 업데이트함</li>
<li>병합 전략은 기본적으로 <code>ort</code>가 사용되며, 내용 충돌이 없으므로 자동 병합됨
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/e302b1c7-0727-4120-8f64-71a5c1f7ce5b/image.png" /><h4 id="병합-커밋-구조-및-이력-확인">병합 커밋 구조 및 이력 확인</h4>
</li>
<li><code>git cat-file -p</code>으로 확인한 merge commit은 두 개의 부모 커밋을 가짐: <code>5198337</code>(main), <code>5733ce1</code>(feature/test1)</li>
<li><code>git log --oneline --graph</code> 결과에서 브랜치 병합 구조가 시각적으로 표현됨</li>
<li>병합 커밋은 분기 및 병합 이력을 명확히 남기며, 협업 시 변경 흐름 추적이 용이해짐
<img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/82abad34-e184-4754-b745-c35362a6bf59/image.png" /></li>
</ul>
<h3 id="▶-관련-질문-요약-2">▶ 관련 질문 요약</h3>
<blockquote>
<p>Q: merge 커밋을 강제로 생성하는 이유는?</p>
</blockquote>
<p>✅ 기록을 명확히 남기기 위함. 병합이 언제, 어떤 브랜치에서 발생했는지 명확하게 추적 가능</p>
<hr />