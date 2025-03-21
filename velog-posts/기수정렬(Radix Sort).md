<h3 id="개념">개념</h3>
<p><strong>기수 정렬(Radix Sort)은 비교 기반 정렬이 아닌(non-comparative sorting algorithm) 자릿수(digit)를 기준으로 데이터를 정렬하는 알고리즘</strong>입니다. 주로 정수(integer)나 문자열(string) 정렬에 사용됩니다.</p>
<p>기수정렬의 시간복잡도는 <strong>O(k ∗ n)</strong>으로, 여기서 k는 자릿수를 의미합니다. 각각의 데이터에 대해 매 자릿수마다 분류작업을 하기 때문에 분류작업이 k번 반복된다고 볼 수 있어 다음과 같은 시간복잡도가 나오는 것입니다.</p>
<h3 id="문제">문제</h3>
<p>n개의 원소가 주어졌을 때, 기수 정렬을 이용해 n개의 숫자를 오름차순으로 정렬하는 프로그램을 작성해보세요.</p>
<h3 id="입력">입력</h3>
<ul>
<li>첫 번째 줄에는 n이 주어집니다.</li>
<li>두 번째 줄부터 n개의 원소값이 공백을 사이에 두고 주어집니다.</li>
<li>1 ≤ n ≤ 100,000</li>
<li>1 ≤ 원소 값 ≤ 100,000</li>
</ul>
<h3 id="코드">코드</h3>
<pre><code># 입력의 최대 자리수에 따라 K 미리 설정
# 자리수가 다른 수끼리도 비교할 수 있다 (작은 수는 계속되는 나눗셈 속에서 나머지가 0으로 나올 것이기 때문)
MAX_K = 6
MAX_DIGIT = 10

# 변수 선언 및 입력
n = int(input())
arr = list(map(int, input().split()))


def radix_sort():
    global arr

    p = 1
    for pos in range(MAX_K):
        arr_new = [[] for _ in range(MAX_DIGIT)]
        for elem in arr:
            digit = (elem // p) % 10
            arr_new[digit].append(elem)

        arr = []
        for digit in range(MAX_DIGIT):
            for elem in arr_new[digit]:
                arr.append(elem)

        p *= 10


radix_sort()

for elem in arr:
    print(elem, end=&quot; &quot;)</code></pre>