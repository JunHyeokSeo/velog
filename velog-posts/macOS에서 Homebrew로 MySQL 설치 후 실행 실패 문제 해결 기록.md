<p>이 문서는 macOS 환경에서 <code>Homebrew</code>를 이용해 MySQL 8.4를 설치하고 실행하는 과정에서 발생한 두 가지 주요 문제와 그 해결 과정을 설명합니다.</p>
<hr />
<h2 id="문제-1-plist와-launchctl을-이용한-서비스-등록-실패">문제 1: <code>.plist</code>와 <code>launchctl</code>을 이용한 서비스 등록 실패</h2>
<h3 id="문제가-발생한-상황">문제가 발생한 상황</h3>
<p>Homebrew를 통해 MySQL 8.4를 설치한 후, 다음 명령어로 MySQL을 백그라운드 서비스로 실행하려 했습니다:</p>
<pre><code class="language-bash">brew services start mysql@8.4</code></pre>
<p>처음에는 <code>&quot;Successfully started&quot;</code>라는 메시지가 출력되었지만, 상태를 확인하면 아래와 같이 <code>error</code>로 표시되었습니다:</p>
<pre><code class="language-bash">brew services list
# 결과:
# mysql@8.4    error   ...</code></pre>
<p>즉, <strong>MySQL 서비스가 실제로는 실행되지 않고 있었던 것</strong>입니다.</p>
<hr />
<h3 id="원인-plist와-launchctl에-대한-이해-부족">원인: <code>.plist</code>와 <code>launchctl</code>에 대한 이해 부족</h3>
<blockquote>
<p><code>.plist</code>는 macOS에서 서비스 실행에 필요한 설정 정보를 담고 있는 파일입니다.<br /><code>.plist</code>는 &quot;Property List&quot;의 줄임말로, XML 형식으로 구성되어 있으며 어떤 프로그램을 어떤 방식으로 언제 실행할지를 정의합니다.<br />예: MySQL이 어떤 경로의 실행 파일을 사용하고, 어떤 옵션을 붙여 백그라운드에서 실행될지를 명시합니다.</p>
</blockquote>
<blockquote>
<p><code>launchctl</code>은 macOS에서 <code>.plist</code> 파일을 등록하거나 제거하고, 백그라운드에서 서비스를 관리하는 명령어입니다.</p>
</blockquote>
<p>Homebrew는 내부적으로 <code>launchctl</code>을 이용해 이 <code>.plist</code> 파일을 자동으로 등록하고, 서비스를 실행합니다. 하지만 이 과정에서 다음과 같은 문제가 발생할 수 있습니다:</p>
<ul>
<li>기존에 실행 중인 MySQL 프로세스가 있는 경우</li>
<li><code>.plist</code> 파일이 손상되었거나 설정이 올바르지 않은 경우</li>
<li>접근 권한 문제가 있는 경우</li>
</ul>
<hr />
<h3 id="해결-방법">해결 방법</h3>
<ol>
<li><p>기존 <code>.plist</code> 등록 해제</p>
<pre><code class="language-bash">launchctl bootout gui/$(id -u) ~/Library/LaunchAgents/homebrew.mxcl.mysql@8.4.plist</code></pre>
</li>
<li><p>Homebrew 서비스 캐시 정리</p>
<pre><code class="language-bash">brew services cleanup</code></pre>
</li>
<li><p>다시 실행 시도</p>
<pre><code class="language-bash">brew services start mysql@8.4</code></pre>
</li>
<li><p><strong>그리고 반드시 확인할 것</strong>: MySQL이 수동으로 실행 중이라면 이 상태에서는 <code>brew services</code>가 실패합니다. 반드시 수동 실행 프로세스를 종료해야 합니다.</p>
<pre><code class="language-bash">ps aux | grep mysqld
sudo kill -9 [PID]</code></pre>
</li>
</ol>
<hr />
<h2 id="문제-2-디렉토리-퍼미션-문제로-서버-실행-및-로그-생성-실패">문제 2: 디렉토리 퍼미션 문제로 서버 실행 및 로그 생성 실패</h2>
<h3 id="문제가-발생한-상황-1">문제가 발생한 상황</h3>
<p>MySQL이 설치된 후 다음 명령어로 수동 실행을 시도했습니다:</p>
<pre><code class="language-bash">sudo /opt/homebrew/opt/mysql@8.4/bin/mysqld_safe --datadir=/opt/homebrew/var/mysql</code></pre>
<p>하지만 <code>.sock</code> 파일(서버 접속용 소켓 파일)이 생성되지 않았고, 로그 파일(<code>.err</code>)도 존재하지 않거나 비어 있었습니다.</p>
<blockquote>
<p><code>.sock</code> 파일은 MySQL 클라이언트가 서버와 통신하기 위해 사용합니다. 파일이 존재하지 않으면 클라이언트는 서버에 접속할 수 없습니다.</p>
<p><code>.err</code> 파일은 MySQL 실행 시 발생한 로그를 담고 있는 파일입니다. 실행 중 오류가 발생했다면 이 파일에 기록됩니다.</p>
</blockquote>
<hr />
<h3 id="원인-데이터-디렉토리의-소유권-문제">원인: 데이터 디렉토리의 소유권 문제</h3>
<p><code>/opt/homebrew/var/mysql</code> 디렉토리의 소유자가 현재 로그인한 사용자가 아닌 <code>_mysql</code> 유저로 되어 있어야 MySQL이 제대로 작동할 수 있습니다. MySQL은 보안을 위해 <code>_mysql</code>이라는 전용 시스템 계정을 사용하여 실행되기 때문입니다.</p>
<p>잘못된 퍼미션 때문에 다음과 같은 문제가 발생했습니다:</p>
<ul>
<li>MySQL이 <code>.sock</code> 파일을 생성하지 못함</li>
<li>로그 파일이 생성되지 않음 → 문제 원인을 파악할 수 없음</li>
<li>실행 직후 서버가 자동으로 종료됨</li>
</ul>
<hr />
<h3 id="해결-방법-1">해결 방법</h3>
<ol>
<li><p>데이터 디렉토리의 권한을 <code>_mysql</code>로 변경</p>
<pre><code class="language-bash">sudo chown -R _mysql:admin /opt/homebrew/var/mysql</code></pre>
</li>
<li><p>MySQL 데이터 디렉토리를 재초기화</p>
<pre><code class="language-bash">sudo /opt/homebrew/opt/mysql@8.4/bin/mysqld --initialize-insecure --datadir=/opt/homebrew/var/mysql</code></pre>
<blockquote>
<p><code>--initialize-insecure</code> 옵션은 root 비밀번호 없이 초기화하며, 이후 보안 설정을 별도로 해야 합니다.</p>
</blockquote>
</li>
<li><p>다시 실행</p>
<pre><code class="language-bash">sudo /opt/homebrew/opt/mysql@8.4/bin/mysqld_safe --datadir=/opt/homebrew/var/mysql</code></pre>
</li>
<li><p><code>.sock</code>, <code>.err</code>, <code>.pid</code> 파일이 정상적으로 생성되는지 확인</p>
<pre><code class="language-bash">ls -l /tmp/mysql.sock
ls -l /opt/homebrew/var/mysql/*.err</code></pre>
</li>
</ol>
<hr />
<h2 id="결론">결론</h2>
<ul>
<li>Homebrew의 <code>brew services</code> 명령은 내부적으로 <code>launchctl</code>과 <code>.plist</code>를 사용하며, 기존 수동 실행 중인 서버와 충돌할 수 있으므로 반드시 중지하고 실행해야 합니다.</li>
<li>데이터 디렉토리의 퍼미션 설정이 MySQL 서버 실행의 핵심 전제조건이며, 적절한 권한이 없으면 로그 파일도 남지 않아 문제를 파악하기 어렵습니다.</li>
<li>문제 해결 시 <code>ps</code>, <code>tail</code>, <code>ls</code>, <code>chown</code> 명령어를 활용한 상태 확인과 로그 확인이 매우 중요합니다.</li>
</ul>