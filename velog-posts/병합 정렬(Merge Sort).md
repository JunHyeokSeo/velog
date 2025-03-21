<h2 id="병합정렬-코드">병합정렬 코드</h2>
<pre><code class="language-java">import java.util.Arrays;
import java.util.Scanner;

public class Main {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);

        int n = Integer.parseInt(scanner.nextLine()); // 첫 번째 줄에서 n 읽기
        int[] arr = Arrays.stream(scanner.nextLine().split(&quot; &quot;))
                            .mapToInt(Integer::parseInt)
                            .toArray();

        mergeSort(arr, new int[n], 0, n - 1); // 정렬 실행

        System.out.println(Arrays.toString(arr).replaceAll(&quot;[\\[\\],]&quot;, &quot;&quot;)); // 출력 포맷 정리
        scanner.close();
    }

    private static void mergeSort(int[] arr, int[] temp, int low, int high) {
        if (low &gt;= high) return;

        int mid = (low + high) / 2;
        mergeSort(arr, temp, low, mid);
        mergeSort(arr, temp, mid + 1, high);
        merge(arr, temp, low, mid, high);
    }

    private static void merge(int[] arr, int[] temp, int low, int mid, int high) {
        System.arraycopy(arr, low, temp, low, high - low + 1); // 원본 배열을 temp에 복사

        int left = low, right = mid + 1, index = low;

        while (left &lt;= mid &amp;&amp; right &lt;= high) {
            arr[index++] = (temp[left] &lt;= temp[right]) ? temp[left++] : temp[right++];
        }

        while (left &lt;= mid) arr[index++] = temp[left++]; // 남은 왼쪽 배열 복사
    }
}
</code></pre>
<hr />
<h2 id="왼쪽-배열만-따로-복사하는-이유">왼쪽 배열만 따로 복사하는 이유</h2>
<pre><code class="language-java">while (left &lt;= mid) arr[index++] = temp[left++]; // 남은 왼쪽 배열 복사</code></pre>
<p>이 부분은 <strong>병합 과정(merge step)</strong>에서 오른쪽 배열의 요소가 먼저 모두 복사된 경우에만 실행된다.</p>
<h3 id="병합-과정에서-어떤-일이-발생하는가">병합 과정에서 어떤 일이 발생하는가?</h3>
<p>병합(merge) 과정에서 왼쪽 배열(left ~ mid)과 오른쪽 배열(mid+1 ~ high)을 비교하면서 정렬하여 arr에 복사합니다.
하지만 왼쪽과 오른쪽 배열의 크기가 다를 수 있고, 비교 과정에서 오른쪽 배열의 요소가 먼저 다 사용될 수도 있음.</p>
<h3 id="오른쪽-배열이-먼저-소진될-경우">오른쪽 배열이 먼저 소진될 경우</h3>
<p>병합 과정에서 작은 값부터 arr에 채우다 보면 오른쪽 배열(mid+1 ~ high)이 먼저 소진될 수 있음.
이 경우, 왼쪽 배열에 남은 요소들이 이미 정렬된 상태로 존재하기 때문에, 비교 없이 그대로 복사해도 됨.</p>
<h3 id="왼쪽-배열만-따로-복사하는-이유-1">왼쪽 배열만 따로 복사하는 이유</h3>
<p>오른쪽 배열이 먼저 소진되면 왼쪽 배열의 남은 값들은 그대로 순서대로 복사하면 됨.
하지만 오른쪽 배열이 남는 경우는 따로 처리하지 않아도 됨 → System.arraycopy(temp, low, arr, low, high - low + 1)에서 이미 복사되었기 때문.</p>
<hr />
<h2 id="예제로-이해하기">예제로 이해하기</h2>
<h3 id="예제-1-오른쪽-배열이-먼저-소진되는-경우">예제 1: 오른쪽 배열이 먼저 소진되는 경우</h3>
<pre><code>초기 배열: [3, 8, 12] [5, 7]
temp 배열 복사: [3, 8, 12, 5, 7]

병합 과정:
  비교 → 3 &lt; 5 → arr[0] = 3
  비교 → 8 &gt; 5 → arr[1] = 5
  비교 → 8 &gt; 7 → arr[2] = 7
  → 오른쪽 배열 소진됨! (5, 7 다 사용됨)
  → 남은 왼쪽 배열(8, 12)을 그대로 복사</code></pre><p><code>while (left &lt;= mid) 실행 → arr[3] = 8, arr[4] = 12 복사.</code></p>
<h3 id="예제-2-왼쪽-배열이-먼저-소진되는-경우">예제 2: 왼쪽 배열이 먼저 소진되는 경우</h3>
<pre><code>초기 배열: [2, 6] [4, 9, 11]
temp 배열 복사: [2, 6, 4, 9, 11]

병합 과정:
  비교 → 2 &lt; 4 → arr[0] = 2
  비교 → 6 &gt; 4 → arr[1] = 4
  비교 → 6 &lt; 9 → arr[2] = 6
  → 왼쪽 배열 소진됨! (2, 6 다 사용됨)</code></pre><p><code>오른쪽 배열(9, 11)은 그대로 남아있음. 하지만 따로 복사하지 않아도 System.arraycopy로 유지됨!</code></p>
<hr />
<h2 id="결론">결론</h2>
<p>오른쪽 배열이 먼저 소진되면 남은 왼쪽 배열을 복사해야 한다.
왼쪽 배열이 먼저 소진되면, 오른쪽 배열은 원래 위치에 있으므로 따로 복사할 필요가 없다.
따라서 while (left &lt;= mid)만 사용하면 충분하다.</p>