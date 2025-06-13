<h2 id="1-핵심-개념">1. 핵심 개념</h2>
<table>
<thead>
<tr>
<th>용어</th>
<th>정의</th>
</tr>
</thead>
<tbody><tr>
<td><strong>공개키 (Public Key)</strong></td>
<td>비대칭 키 쌍 중 공개된 절반. 서명 검증·암호화에 사용.</td>
</tr>
<tr>
<td><strong>개인키 (Private Key)</strong></td>
<td>비밀 키. 소유자만이 보유하며 서명 생성·복호화에 사용.</td>
</tr>
<tr>
<td><strong>해시 함수</strong></td>
<td>입력 데이터를 고정 길이로 압축하는 일방향 함수. 서명 생성·무결성 검증에 필수.</td>
</tr>
<tr>
<td><strong>전자서명</strong></td>
<td>메시지 해시값을 개인키로 암호화한 값. 무결성·인증·부인방지 제공.</td>
</tr>
<tr>
<td><strong>인증서 (X.509)</strong></td>
<td>공개키 + 사용자 정보에 대해 <strong>CA</strong> 가 서명한 디지털 문서.</td>
</tr>
<tr>
<td><strong>인증기관 (CA)</strong></td>
<td>인증서에 전자서명을 수행해 공개키의 소유자를 보증하는 신뢰된 제3자.</td>
</tr>
<tr>
<td><strong>신뢰 저장소 (Trust Store)</strong></td>
<td>OS·브라우저에 내장된 루트/중간 CA 인증서 집합.</td>
</tr>
<tr>
<td><strong>PKCS#12 (.pfx/.p12)</strong></td>
<td>인증서·개인키·체인 인증서를 암호화해 담는 컨테이너. 반드시 비밀번호로 보호.</td>
</tr>
</tbody></table>
<hr />
<h2 id="2-공개키-기반-구조pki와-ca">2. 공개키 기반 구조(PKI)와 CA</h2>
<p><strong>PKI</strong>: 공개키·인증서·신뢰 사슬을 관리하는 제도적·기술적 프레임워크<br /><strong>CA(인증기관)</strong>: 사용자 공개키 ↔ 실제 신원을 연결해 <strong>전자서명</strong>으로 보증<br /><strong>Trust Store</strong>: OS·브라우저에 내장된 신뢰 가능한 루트·중간 CA 인증서 목록<br /><strong>신뢰 체인</strong>  </p>
<pre><code>Root CA
  └─ Intermediate CA
        └─ End‑Entity Certificate (User / Server)</code></pre><hr />
<h2 id="3-x509-인증서-구조와-생성-과정">3. X.509 인증서 구조와 생성 과정</h2>
<h3 id="31-구조">3‑1 구조</h3>
<pre><code>Certificate
 ├─ Version
 ├─ Serial Number
 ├─ Signature Algorithm
 ├─ Issuer              ← CA 정보
 ├─ Validity            ← Not Before / Not After
 ├─ Subject             ← 사용자/서버 정보
 ├─ Subject Public Key
 ├─ Extensions          ← KeyUsage, SAN 등
 └─ Signature           ← Issuer(=CA)의 전자서명</code></pre><h3 id="32-생성-단계">3‑2 생성 단계</h3>
<ol>
<li><strong>Key pair 생성</strong>: 개인키·공개키</li>
<li><strong>CSR</strong>: 공개키 + Distinguished Name → <code>req.csr</code></li>
<li><strong>CA 검증</strong>: 신청자 신원 확인</li>
<li><strong>CA 서명</strong>: CSR 해시값을 <strong>CA 개인키</strong>로 암호화 → <code>cert.crt</code></li>
<li>(선택) <strong>PKCS#12 패키징</strong>: 인증서 + 개인키 + 체인 → <code>user.pfx</code></li>
</ol>
<blockquote>
<p><strong>전자서명 대상</strong>은 <code>Signature</code> 필드를 제외한 <strong>본문 전체</strong>이며, 변조 시 무효화됩니다.</p>
</blockquote>
<hr />
<h2 id="4-인증서·키-파일-형식-비교">4. 인증서·키 파일 형식 비교</h2>
<table>
<thead>
<tr>
<th>확장자</th>
<th>인코딩</th>
<th>포함 내용</th>
<th>암호화</th>
<th>주 용도</th>
</tr>
</thead>
<tbody><tr>
<td><code>.crt</code> <code>.cer</code></td>
<td>PEM/DER</td>
<td>인증서</td>
<td>×</td>
<td>TLS, 클라이언트 인증서 배포</td>
</tr>
<tr>
<td><code>.pem</code></td>
<td>PEM</td>
<td>인증서·키 혼합 가능</td>
<td>선택</td>
<td>유연한 구성</td>
</tr>
<tr>
<td><code>.der</code></td>
<td>DER</td>
<td>인증서(바이너리)</td>
<td>×</td>
<td>Java 환경</td>
</tr>
<tr>
<td><code>.key</code></td>
<td>PEM</td>
<td>개인키</td>
<td>선택</td>
<td>서버 비밀키 저장</td>
</tr>
<tr>
<td><code>.csr</code></td>
<td>PEM</td>
<td>인증서 서명 요청</td>
<td>×</td>
<td>발급 신청</td>
</tr>
<tr>
<td><code>.pfx</code> <code>.p12</code></td>
<td>PKCS#12</td>
<td><strong>인증서 + 개인키 + 체인</strong></td>
<td><strong>필수</strong></td>
<td>클라이언트 인증, Windows, 홈택스</td>
</tr>
</tbody></table>
<hr />
<h2 id="5-pkcs12pfxp12-패키지-상세">5. PKCS#12(.pfx/.p12) 패키지 상세</h2>
<table>
<thead>
<tr>
<th>내부 객체</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong>개인키</strong></td>
<td>challenge 서명·문서 서명에 사용</td>
</tr>
<tr>
<td><strong>최종 인증서</strong></td>
<td>사용자 공개키·Subject·CA 서명 포함</td>
</tr>
<tr>
<td><strong>체인 인증서</strong></td>
<td>중간/루트 CA 인증서 (필요 시)</td>
</tr>
<tr>
<td><strong>메타데이터</strong></td>
<td>friendlyName, localKeyID 등</td>
</tr>
</tbody></table>
<h3 id="51-추출·분리">5‑1 추출·분리</h3>
<pre><code class="language-bash"># 인증서만 추출
openssl pkcs12 -in user.pfx -clcerts -nokeys -out cert.crt
# 개인키만 추출
openssl pkcs12 -in user.pfx -nocerts -nodes -out private.key</code></pre>
<h3 id="5-2-pfx-복호화-후-인증서만-추출·전송-가능-여부">5-2. pfx 복호화 후 인증서만 추출·전송 가능 여부</h3>
<blockquote>
<p>인증서만 서버로 전송 가능하나 <strong>인증 행위</strong>에는 개인키 기반 <strong>전자서명 데이터</strong>가 추가로 필요.</p>
</blockquote>
<ul>
<li><strong>가능</strong>하다. pfx는 컨테이너이므로 OpenSSL 등으로 <strong>개인키를 분리</strong>하고,<br />공개용 인증서(<code>.crt</code> 또는 <code>.pem</code>)만 서버로 전송하거나 공개 저장소에 게시할 수 있다.</li>
<li>단, 서버가 클라이언트를 인증하려면 <strong>개인키로 서명한 값</strong>도 함께 받아야 하므로,<br />인증서만 단독으로는 <strong>‘신분증’</strong> 역할만 하고 <strong>‘서명 도구’</strong> 역할은 하지 못한다.</li>
</ul>
<hr />
<h2 id="6-클라이언트서버-인증-시퀀스">6. 클라이언트–서버 인증 시퀀스</h2>
<table>
<thead>
<tr>
<th>단계</th>
<th>설명</th>
</tr>
</thead>
<tbody><tr>
<td><strong>1. 서버 → 클라이언트</strong></td>
<td>랜덤 challenge 전송</td>
</tr>
<tr>
<td><strong>2. 클라이언트</strong></td>
<td>pfx 선택 → 비밀번호 입력 → 개인키 복호화</td>
</tr>
<tr>
<td><strong>3. 클라이언트</strong></td>
<td>개인키로 challenge 서명</td>
</tr>
<tr>
<td><strong>4. 클라이언트 → 서버</strong></td>
<td><code>{signature, certificate}</code> 전송</td>
</tr>
<tr>
<td><strong>5. 서버</strong></td>
<td>인증서 안의 <code>Issuer</code> 정보 확인</td>
</tr>
<tr>
<td><strong>6. 서버</strong></td>
<td>CA 공개키(Trust Store)로 인증서 서명 검증 (CA의 공개키로 cert 서명 검증)</td>
</tr>
<tr>
<td><strong>7. 서버</strong></td>
<td>인증서 안의 <strong>공개키</strong>로 <code>signature</code> 검증</td>
</tr>
<tr>
<td><strong>8. 서버</strong></td>
<td>검증 성공 → 사용자 인증 완료</td>
</tr>
</tbody></table>
<hr />
<h2 id="7-키-사용-구조">7. 키 사용 구조</h2>
<table>
<thead>
<tr>
<th>사용 키</th>
<th>주체</th>
<th>시점</th>
<th>목적</th>
</tr>
</thead>
<tbody><tr>
<td><strong>개인키 (pfx 내부)</strong></td>
<td>클라이언트</td>
<td>challenge 전자서명 시</td>
<td>본인임을 증명하는 서명 생성</td>
</tr>
<tr>
<td><strong>공개키 (인증서 포함)</strong></td>
<td>서버</td>
<td>서명 검증 시</td>
<td>서명이 개인키로 생성된 것인지 확인</td>
</tr>
<tr>
<td><strong>CA 공개키 (신뢰 저장소)</strong></td>
<td>서버</td>
<td>인증서 검증 시</td>
<td>인증서 위·변조 여부 및 CA 신뢰 확인</td>
</tr>
</tbody></table>
<hr />
<h2 id="8-faq">8. FAQ</h2>
<table>
<thead>
<tr>
<th>질문</th>
<th>답변 요약</th>
</tr>
</thead>
<tbody><tr>
<td>pfx 암호를 모르면?</td>
<td>컨테이너 복호화·서명 불가 → 인증 불가</td>
</tr>
<tr>
<td>인증서 내용 수정 가능?</td>
<td>서명 무효화 → CA 재서명 없이는 불가능</td>
</tr>
<tr>
<td>CA의 개인키는 언제 사용?</td>
<td><strong>발급 시 1회</strong> 서명용. 인증 과정엔 관여하지 않음</td>
</tr>
<tr>
<td>인증서만 서버에 주면?</td>
<td>신분 증명은 되지만 서명 없으므로 로그인 성립 안 됨</td>
</tr>
<tr>
<td>CA 공개키는 어디?</td>
<td>OS·브라우저 Trust Store에 내장</td>
</tr>
</tbody></table>
<hr />
<h2 id="9-핵심-요약">9. 핵심 요약</h2>
<ol>
<li><strong>X.509 인증서</strong>는 “본문 해시 + CA 개인키 서명”으로 완성되는 디지털 신분증이다.  </li>
<li><strong>PKCS#12(.pfx)</strong> 는 인증서·개인키를 암호화해 담는 컨테이너이며, 비밀번호 없이는 사용 불가.  </li>
<li>클라이언트‑서버 인증은 “challenge → 개인키 서명 → 공개키 검증” 절차로 동작한다.</li>
<li>CA는 발급 시점에만 서명하며, 실시간 인증에서는 <strong>CA 공개키</strong>로 인증서 진위를 검사한다.  </li>
<li>신뢰는 <strong>운영체제·브라우저 Trust Store</strong>의 루트 CA 목록에 기반한다.</li>
<li>pfx 복호화 후 <strong>인증서만 추출</strong>해 서버로 보내는 것은 가능하나, <strong>실제 인증</strong>에는 <strong>개인키로 생성한 서명 데이터</strong>가 반드시 필요하다.  </li>
</ol>