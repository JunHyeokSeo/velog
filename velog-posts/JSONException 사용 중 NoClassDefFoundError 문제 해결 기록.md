<h2 id="문제-상황">문제 상황</h2>
<ul>
<li><code>@ExceptionHandler</code>에서 <code>JSONException</code>을 throws 하자 테스트 실행 중 다음 오류 발생:</li>
</ul>
<pre><code>java.lang.NoClassDefFoundError: org/springframework/boot/configurationprocessor/json/JSONException</code></pre><h2 id="원인-분석">원인 분석</h2>
<ul>
<li><code>JSONException</code> 클래스는 <code>spring-boot-configuration-processor</code> 모듈 안에 존재</li>
<li>이 모듈은 <code>compileOnly</code> 또는 <code>annotationProcessor</code> 용도로만 존재하며, <strong>런타임 classpath에 포함되지 않음</strong></li>
<li>따라서 실제 서버 실행 또는 테스트 중에는 해당 클래스가 존재하지 않아 <code>ClassNotFoundException</code> → <code>NoClassDefFoundError</code> 발생</li>
</ul>
<h2 id="해결-방법">해결 방법</h2>
<ul>
<li><code>JSONException</code>을 코드에서 제거</li>
<li>예외 메시지를 직접 <code>String</code>으로 처리하고, Jackson 기반 응답만 사용</li>
<li>이후 테스트 및 실행 모두 정상 작동 확인</li>
</ul>
<h2 id="결론">결론</h2>
<ul>
<li><code>configuration-processor</code> 패키지는 절대 직접 참조하면 안 됨</li>
<li>예외 처리는 표준 Java 예외 또는 외부 의존성(<code>org.json</code>)으로 명확하게 처리해야 함</li>
</ul>