<h3 id="1-git-디렉토리-구조-개요">1. <code>.git</code> 디렉토리 구조 개요</h3>
<p><code>git init</code>을 실행하면 <code>.git/</code> 내부에 다양한 파일과 디렉토리가 생성된다. Git은 이 내부 구조를 기반으로 버전 관리, 브랜치, 커밋 이력 등을 추적한다.</p>
<pre><code class="language-bash">.git/
├── HEAD
├── config
├── description
├── hooks/
├── info/
├── objects/
├── refs/</code></pre>
<hr />
<h3 id="2-주요-구성-요소-설명">2. 주요 구성 요소 설명</h3>
<table>
<thead>
<tr>
<th>경로</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><code>.git/HEAD</code></td>
<td>현재 체크아웃된 브랜치를 가리킴 (<code>ref: refs/heads/main</code>)</td>
</tr>
<tr>
<td><code>.git/config</code></td>
<td>저장소 단위 설정 (user, core, remote 등)</td>
</tr>
<tr>
<td><code>.git/description</code></td>
<td>bare 저장소용 설명 (로컬에서는 무시됨)</td>
</tr>
<tr>
<td><code>.git/hooks/</code></td>
<td>Git 이벤트 발생 시 실행되는 스크립트 모음</td>
</tr>
<tr>
<td><code>.git/info/exclude</code></td>
<td>개인용 <code>.gitignore</code> 설정 파일</td>
</tr>
<tr>
<td><code>.git/objects/</code></td>
<td>실제 Git 객체(blob, tree, commit, tag 등) 저장 공간</td>
</tr>
<tr>
<td><code>.git/refs/</code></td>
<td>브랜치와 태그의 참조 정보 저장</td>
</tr>
</tbody></table>
<hr />
<h3 id="3-head의-역할">3. HEAD의 역할</h3>
<ul>
<li>HEAD는 현재 작업 중인 커밋 혹은 브랜치를 가리키는 포인터</li>
<li>보통 <code>ref: refs/heads/&lt;브랜치명&gt;</code> 형태</li>
<li><code>Detached HEAD</code> 상태일 경우 직접 커밋 해시를 가리킴</li>
</ul>
<h4 id="▶-관련-질문-요약">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: HEAD가 무엇을 의미하나?</strong></p>
</blockquote>
<p>HEAD는 현재 작업 중인 브랜치 또는 커밋을 참조하는 포인터다. 대부분 브랜치를 참조하지만, checkout으로 특정 커밋을 가리킬 수도 있다.</p>
<hr />
<h3 id="4-refs-시스템-브랜치와-태그의-실체">4. refs 시스템: 브랜치와 태그의 실체</h3>
<p>Git에서 브랜치나 태그는 단순한 <strong>텍스트 파일</strong>이며, 내부에는 특정 커밋의 SHA-1 해시가 들어 있다.</p>
<ul>
<li><code>refs/heads/</code> → 로컬 브랜치</li>
<li><code>refs/remotes/origin/</code> → 원격 브랜치</li>
<li><code>refs/tags/</code> → 태그 참조</li>
</ul>
<h4 id="예시">예시:</h4>
<pre><code class="language-bash">cat .git/refs/heads/main
5198337ce609698b6783fc67c22c22e2fbcf1887</code></pre>
<h4 id="▶-관련-질문-요약-1">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: <code>refs/heads/main</code>과 <code>refs/remotes/origin/main</code>의 차이는?</strong></p>
</blockquote>
<ul>
<li><code>refs/heads/main</code>: 내 로컬 브랜치</li>
<li><code>refs/remotes/origin/main</code>: fetch 시 받아온 원격 저장소 상태 (읽기 전용)</li>
</ul>
<blockquote>
<p><strong>Q: <code>logs/refs/heads/main</code>과 <code>logs/HEAD</code> 차이는?</strong></p>
</blockquote>
<ul>
<li><code>logs/HEAD</code>: HEAD의 이동 기록</li>
<li><code>logs/refs/heads/main</code>: main 브랜치가 어떤 커밋을 가리켰는지의 이력</li>
</ul>
<blockquote>
<p><strong>Q: origin 외 다른 remote가 생길 수 있나?</strong></p>
</blockquote>
<p>✅ 가능하다. <code>git remote add &lt;name&gt; &lt;url&gt;</code> 혹은 <code>git clone -o &lt;name&gt; &lt;url&gt;</code>로 추가하면 <code>refs/remotes/&lt;name&gt;/</code> 디렉토리가 생성된다.</p>
<hr />
<h3 id="5-gitindex-스테이징-영역">5. <code>.git/index</code>: 스테이징 영역</h3>
<ul>
<li>Git에서 커밋 전 파일을 잠시 보관하는 공간 (staging area)</li>
<li><code>git add</code> 하면 파일의 blob 해시가 index에 기록됨</li>
<li><code>git commit</code>은 index의 상태를 기반으로 tree 객체를 생성함</li>
</ul>
<pre><code class="language-bash">git ls-files --stage</code></pre>
<h4 id="▶-관련-질문-요약-2">▶ 관련 질문 요약</h4>
<blockquote>
<p><strong>Q: index는 언제 생기고, 어떻게 동작하나?</strong></p>
</blockquote>
<ul>
<li>저장소 생성 시 파일로 존재하지만, 실질적인 데이터는 <code>git add</code> 시 기록됨</li>
<li>index는 파일 경로 → blob 해시 매핑을 담고 있음</li>
</ul>
<blockquote>
<p><strong>Q: commit하면 index는 초기화되나?</strong></p>
</blockquote>
<p>❌ 아니다. index는 그대로 유지되며 HEAD와 동기화된 상태가 된다.</p>
<blockquote>
<p><strong>Q: reset 명령은 index에도 영향을 미치나?</strong></p>
</blockquote>
<p>✅ <code>git reset --mixed</code>, <code>--hard</code>는 index를 과거 커밋 상태로 되돌린다. <code>--soft</code>는 HEAD만 이동시키고 index는 그대로다.</p>
<hr />