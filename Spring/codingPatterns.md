### 코딩테스트 유형별 문제 정리 (1/20)

## ✅ 유형 1: 두 수의 합 (Two Sum)

### 💡 문제 설명

정수 배열 `nums`와 정수 `target`이 주어졌을 때, 합이 `target`이 되는 두 수의 **인덱스**를 반환하라.

* 단, 정확히 한 쌍만 존재한다고 가정한다.

### 📥 입력 예시

```java
int[] nums = {2, 7, 11, 15};
int target = 9;
```

출력: `[0, 1]` → 2 + 7 = 9

### ✅ 풀이 1 (최선): HashMap 활용 (O(n))

```java
public int[] twoSum(int[] nums, int target) {
    Map<Integer, Integer> map = new HashMap<>();
    for (int i = 0; i < nums.length; i++) {
        int complement = target - nums[i];
        if (map.containsKey(complement)) {
            return new int[]{map.get(complement), i};
        }
        map.put(nums[i], i);
    }
    return new int[0];
}
```

### ⚠️ 풀이 2 (차선): 완전탐색 (이중 for문, O(n^2))

```java
public int[] twoSum(int[] nums, int target) {
    for (int i = 0; i < nums.length; i++) {
        for (int j = i + 1; j < nums.length; j++) {
            if (nums[i] + nums[j] == target) {
                return new int[]{i, j};
            }
        }
    }
    return new int[0];
}
```

### 🔄 풀이 3 (대안): Java Stream API 활용

```java
public int[] twoSum(int[] nums, int target) {
    return IntStream.range(0, nums.length)
            .boxed()
            .flatMap(i -> IntStream.range(i + 1, nums.length)
                    .filter(j -> nums[i] + nums[j] == target)
                    .mapToObj(j -> new int[]{i, j}))
            .findFirst()
            .orElse(new int[0]);
}
```

---

## ✅ 유형 2: 괄호 유효성 검사 (Valid Parentheses)

### 💡 문제 설명

주어진 문자열이 올바른 괄호 구조인지 판별하라. 괄호는 (, ), {, }, \[, ] 만 포함한다.

### 📥 입력 예시

```java
String s = "()[]{}";
```

출력: true

### ✅ 풀이 1 (최선): Stack + 예상 닫힘 괄호 저장

```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(') stack.push(')');
        else if (c == '{') stack.push('}');
        else if (c == '[') stack.push(']');
        else if (stack.isEmpty() || stack.pop() != c) return false;
    }
    return stack.isEmpty();
}
```

### ⚠️ 풀이 2 (차선): Stack + 비교 조건문

```java
public boolean isValid(String s) {
    Stack<Character> stack = new Stack<>();
    for (char c : s.toCharArray()) {
        if (c == '(' || c == '{' || c == '[') {
            stack.push(c);
        } else {
            if (stack.isEmpty()) return false;
            char top = stack.pop();
            if ((c == ')' && top != '(') ||
                (c == '}' && top != '{') ||
                (c == ']' && top != '[')) return false;
        }
    }
    return stack.isEmpty();
}
```

### 🔄 풀이 3 (대안): Map + Stack (람다식 활용 가능)

```java
public boolean isValid(String s) {
    Deque<Character> stack = new ArrayDeque<>();
    Map<Character, Character> map = Map.of(')', '(', ']', '[', '}', '{');
    for (char c : s.toCharArray()) {
        if (map.containsValue(c)) stack.push(c);
        else if (stack.isEmpty() || stack.pop() != map.get(c)) return false;
    }
    return stack.isEmpty();
}
```

---

(## ✅ 유형 3: 부분 수열 / 조합 (Subsets / Combinations)

### 💡 문제 설명

정수 배열이 주어졌을 때, 가능한 모든 부분 수열(부분 집합)을 구하라.

### 📥 입력 예시

```java
int[] nums = {1, 2, 3};
```

출력 예시:

```
[
  [], [1], [2], [3], [1, 2], [1, 3], [2, 3], [1, 2, 3]
]
```

### ✅ 풀이 1 (최선): 백트래킹

```java
public List<List<Integer>> subsets(int[] nums) {
    List<List<Integer>> result = new ArrayList<>();
    backtrack(result, new ArrayList<>(), nums, 0);
    return result;
}
private void backtrack(List<List<Integer>> result, List<Integer> temp, int[] nums, int start) {
    result.add(new ArrayList<>(temp));
    for (int i = start; i < nums.length; i++) {
        temp.add(nums[i]);
        backtrack(result, temp, nums, i + 1);
        temp.remove(temp.size() - 1);
    }
}
```

### ⚠️ 풀이 2 (차선): 비트마스크

```java
public List<List<Integer>> subsets(int[] nums) {
    int n = nums.length;
    List<List<Integer>> result = new ArrayList<>();
    int total = 1 << n;
    for (int mask = 0; mask < total; mask++) {
        List<Integer> subset = new ArrayList<>();
        for (int i = 0; i < n; i++) {
            if ((mask & (1 << i)) != 0) {
                subset.add(nums[i]);
            }
        }
        result.add(subset);
    }
    return result;
}
```

### 🔄 풀이 3 (대안): Stream + 비트마스크

```java
public List<List<Integer>> subsets(int[] nums) {
    return IntStream.range(0, 1 << nums.length)
        .mapToObj(mask -> IntStream.range(0, nums.length)
            .filter(i -> (mask & (1 << i)) != 0)
            .mapToObj(i -> nums[i])
            .collect(Collectors.toList()))
        .collect(Collectors.toList());
}
```

---

(## ✅ 유형 4: 최대 연속 부분합 (Maximum Subarray, Kadane’s Algorithm)

### 💡 문제 설명

정수 배열 nums가 주어질 때, 연속된 부분 배열 중 합이 가장 큰 값을 반환하라.

### 📥 입력 예시

```java
int[] nums = {-2, 1, -3, 4, -1, 2, 1, -5, 4};
```

출력: 6 → \[4, -1, 2, 1]의 합

### ✅ 풀이 1 (최선): Kadane 알고리즘 (O(n))

```java
public int maxSubArray(int[] nums) {
    int maxSum = nums[0];
    int current = nums[0];
    for (int i = 1; i < nums.length; i++) {
        current = Math.max(nums[i], current + nums[i]);
        maxSum = Math.max(maxSum, current);
    }
    return maxSum;
}
```

### ⚠️ 풀이 2 (차선): 모든 부분합 시도 (O(n^2))

```java
public int maxSubArray(int[] nums) {
    int maxSum = Integer.MIN_VALUE;
    for (int i = 0; i < nums.length; i++) {
        int sum = 0;
        for (int j = i; j < nums.length; j++) {
            sum += nums[j];
            maxSum = Math.max(maxSum, sum);
        }
    }
    return maxSum;
}
```

### 🔄 풀이 3 (대안): 스트림 + DP 배열 (O(n))

```java
public int maxSubArray(int[] nums) {
    int[] max = new int[nums.length];
    max[0] = nums[0];
    IntStream.range(1, nums.length)
        .forEach(i -> max[i] = Math.max(nums[i], max[i - 1] + nums[i]));
    return Arrays.stream(max).max().orElse(Integer.MIN_VALUE);
}
```

---

(## ✅ 유형 5: 이진 탐색 (Binary Search)

### 💡 문제 설명

정렬된 정수 배열 `nums`와 정수 `target`이 주어질 때, `target`의 인덱스를 반환하라. 없으면 -1을 반환.

### 📥 입력 예시

```java
int[] nums = {-1, 0, 3, 5, 9, 12};
int target = 9;
```

출력: 4

### ✅ 풀이 1 (최선): 반복문 기반 이진 탐색 (O(log n))

```java
public int binarySearch(int[] nums, int target) {
    int left = 0, right = nums.length - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (nums[mid] == target) return mid;
        else if (nums[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

### ⚠️ 풀이 2 (차선): 재귀 기반 이진 탐색

```java
public int binarySearch(int[] nums, int target) {
    return binarySearchRecursive(nums, 0, nums.length - 1, target);
}
private int binarySearchRecursive(int[] nums, int left, int right, int target) {
    if (left > right) return -1;
    int mid = left + (right - left) / 2;
    if (nums[mid] == target) return mid;
    else if (nums[mid] < target)
        return binarySearchRecursive(nums, mid + 1, right, target);
    else
        return binarySearchRecursive(nums, left, mid - 1, target);
}
```

### 🔄 풀이 3 (대안): Java Stream + IntStream (비추천, 탐색 불가)

```java
public int binarySearch(int[] nums, int target) {
    return IntStream.range(0, nums.length)
        .filter(i -> nums[i] == target)
        .findFirst()
        .orElse(-1);
}
```

* 이진 탐색이 아님
* 단순 탐색에 불과함 → 성능 손해

---

(## ✅ 유형 6: K번째 수 / 배열 정렬 (K-th Number / Array Sorting)

### 💡 문제 설명

배열에서 i번째부터 j번째까지 자른 후 정렬했을 때, k번째에 있는 수를 구하라. (프로그래머스 대표 문제)

### 📥 입력 예시

```java
int[] array = {1, 5, 2, 6, 3, 7, 4};
int[][] commands = {{2, 5, 3}, {4, 4, 1}, {1, 7, 3}};
```

출력: `[5, 6, 3]`

### ✅ 풀이 1 (최선): 자르기 + 정렬 + 인덱싱

```java
public int[] solution(int[] array, int[][] commands) {
    return Arrays.stream(commands)
        .mapToInt(cmd -> {
            int[] sliced = Arrays.copyOfRange(array, cmd[0] - 1, cmd[1]);
            Arrays.sort(sliced);
            return sliced[cmd[2] - 1];
        })
        .toArray();
}
```

### ⚠️ 풀이 2 (차선): 반복문으로 구현

```java
public int[] solution(int[] array, int[][] commands) {
    int[] result = new int[commands.length];
    for (int i = 0; i < commands.length; i++) {
        int start = commands[i][0] - 1;
        int end = commands[i][1];
        int k = commands[i][2] - 1;
        int[] sliced = new int[end - start];
        for (int j = start; j < end; j++) {
            sliced[j - start] = array[j];
        }
        Arrays.sort(sliced);
        result[i] = sliced[k];
    }
    return result;
}
```

### 🔄 풀이 3 (대안): Java Stream + List 변환 후 처리

```java
public int[] solution(int[] array, int[][] commands) {
    return Arrays.stream(commands)
        .mapToInt(cmd -> IntStream.range(cmd[0] - 1, cmd[1])
            .mapToObj(i -> array[i])
            .sorted()
            .collect(Collectors.toList())
            .get(cmd[2] - 1))
        .toArray();
}
```

---

(## ✅ 유형 7: 문자열 압축 (String Compression)

### 💡 문제 설명

문자열이 주어졌을 때, 같은 문자가 반복될 경우 반복 횟수를 문자 뒤에 붙여서 압축한 문자열의 최소 길이를 구하라.

### 📥 입력 예시

```java
String s = "aabbaccc";
```

출력: 7 → 압축 결과 "2a2ba3c"

### ✅ 풀이 1 (최선): 문자열 길이 k로 자르며 압축 (O(n^2))

```java
public int solution(String s) {
    int minLen = s.length();
    for (int len = 1; len <= s.length() / 2; len++) {
        StringBuilder compressed = new StringBuilder();
        String prev = s.substring(0, len);
        int count = 1;
        for (int j = len; j <= s.length(); j += len) {
            String next = s.substring(j, Math.min(j + len, s.length()));
            if (prev.equals(next)) {
                count++;
            } else {
                compressed.append(count > 1 ? count + prev : prev);
                prev = next;
                count = 1;
            }
        }
        minLen = Math.min(minLen, compressed.length());
    }
    return minLen;
}
```

### ⚠️ 풀이 2 (차선): 문자열 반복 검출만 적용

```java
public int simpleCompress(String s) {
    StringBuilder sb = new StringBuilder();
    char prev = s.charAt(0);
    int count = 1;
    for (int i = 1; i < s.length(); i++) {
        char curr = s.charAt(i);
        if (curr == prev) {
            count++;
        } else {
            sb.append(count > 1 ? count + "" + prev : prev);
            prev = curr;
            count = 1;
        }
    }
    sb.append(count > 1 ? count + "" + prev : prev);
    return sb.length();
}
```

### 🔄 풀이 3 (대안): Stream 활용 불가 → 문자열은 반복문 필수 처리

* 해당 유형은 Stream API로 효율적으로 처리하기 어렵기 때문에 반복문이 최선입니다.

---

(## ✅ 유형 8: 슬라이딩 윈도우 (Sliding Window)

### 💡 문제 설명

배열 `nums`와 정수 `k`가 주어졌을 때, 길이 `k`인 모든 연속 부분 배열의 최대합을 구하라.

### 📥 입력 예시

```java
int[] nums = {1, 4, 2, 10, 23, 3, 1, 0, 20};
int k = 4;
```

출력: 39 → 부분 배열 \[4, 2, 10, 23]

### ✅ 풀이 1 (최선): 슬라이딩 윈도우 기법

```java
public int maxSum(int[] arr, int k) {
    int windowSum = 0;
    for (int i = 0; i < k; i++) windowSum += arr[i];
    int maxSum = windowSum;
    for (int i = k; i < arr.length; i++) {
        windowSum += arr[i] - arr[i - k];
        maxSum = Math.max(maxSum, windowSum);
    }
    return maxSum;
}
```

### ⚠️ 풀이 2 (차선): 모든 구간합 비교 (O(n\*k))

```java
public int maxSum(int[] arr, int k) {
    int maxSum = 0;
    for (int i = 0; i <= arr.length - k; i++) {
        int sum = 0;
        for (int j = 0; j < k; j++) {
            sum += arr[i + j];
        }
        maxSum = Math.max(maxSum, sum);
    }
    return maxSum;
}
```

### 🔄 풀이 3 (대안): Stream 활용 (성능은 다소 떨어짐)

```java
public int maxSum(int[] arr, int k) {
    return IntStream.rangeClosed(0, arr.length - k)
        .map(i -> IntStream.range(i, i + k).map(j -> arr[j]).sum())
        .max().orElse(0);
}
```

---

(## ✅ 유형 9: 중복 문자 제거 (Remove Duplicate Letters)

### 💡 문제 설명

중복된 문자를 제거하고, 사전 순으로 가장 앞서는 결과를 구하라 (단, 문자 순서를 바꾸지 말 것)

### 📥 입력 예시

```java
String s = "cbacdcbc";
```

출력: "acdb"

### ✅ 풀이 1 (최선): 스택 + 방문처리 (O(n))

```java
public String removeDuplicateLetters(String s) {
    Stack<Character> stack = new Stack<>();
    boolean[] visited = new boolean[26];
    int[] lastIndex = new int[26];

    for (int i = 0; i < s.length(); i++)
        lastIndex[s.charAt(i) - 'a'] = i;

    for (int i = 0; i < s.length(); i++) {
        char c = s.charAt(i);
        if (visited[c - 'a']) continue;

        while (!stack.isEmpty() && c < stack.peek() && lastIndex[stack.peek() - 'a'] > i) {
            visited[stack.pop() - 'a'] = false;
        }
        stack.push(c);
        visited[c - 'a'] = true;
    }

    StringBuilder result = new StringBuilder();
    for (char ch : stack) result.append(ch);
    return result.toString();
}
```

### ⚠️ 풀이 2 (차선): Set + LinkedHashSet 유지 (순서 보존, 사전 순 보장 X)

```java
public String removeDuplicateLettersSimple(String s) {
    Set<Character> set = new LinkedHashSet<>();
    for (char c : s.toCharArray()) {
        if (set.contains(c)) set.remove(c);
        set.add(c);
    }
    StringBuilder sb = new StringBuilder();
    for (char c : set) sb.append(c);
    return sb.toString();
}
```

### 🔄 풀이 3 (대안): Stream + distinct (정렬 X, 순서 보장 X)

```java
public String removeDuplicateLettersStream(String s) {
    return s.chars()
            .mapToObj(c -> String.valueOf((char) c))
            .distinct()
            .collect(Collectors.joining());
}
```

---

(## ✅ 유형 10: 주식 최대 수익 (Best Time to Buy and Sell Stock)

### 💡 문제 설명

하루 단위의 주식 가격이 담긴 배열이 주어질 때, 가장 낮은 가격에 사고 가장 높은 가격에 팔아 얻을 수 있는 최대 이익을 구하라.

* 단, 구매 후 판매해야 하며 한 번만 거래할 수 있다.

### 📥 입력 예시

```java
int[] prices = {7, 1, 5, 3, 6, 4};
```

출력: 5 → 1에 사서 6에 팔기

### ✅ 풀이 1 (최선): 최소값 갱신 + 이익 최대화 (O(n))

```java
public int maxProfit(int[] prices) {
    int minPrice = Integer.MAX_VALUE;
    int maxProfit = 0;
    for (int price : prices) {
        minPrice = Math.min(minPrice, price);
        maxProfit = Math.max(maxProfit, price - minPrice);
    }
    return maxProfit;
}
```

### ⚠️ 풀이 2 (차선): 이중 반복문으로 모든 경우 탐색 (O(n^2))

```java
public int maxProfit(int[] prices) {
    int maxProfit = 0;
    for (int i = 0; i < prices.length - 1; i++) {
        for (int j = i + 1; j < prices.length; j++) {
            maxProfit = Math.max(maxProfit, prices[j] - prices[i]);
        }
    }
    return maxProfit;
}
```

### 🔄 풀이 3 (대안): 스트림으로 최소값 추적 (성능 미세 저하)

```java
public int maxProfit(int[] prices) {
    int[] min = {Integer.MAX_VALUE};
    return IntStream.range(0, prices.length)
        .map(i -> {
            min[0] = Math.min(min[0], prices[i]);
            return prices[i] - min[0];
        })
        .max().orElse(0);
}
```

---

(## ✅ 유형 11: 가장 긴 증가하는 부분 수열 (LIS - Longest Increasing Subsequence)

### 💡 문제 설명

정수 배열이 주어졌을 때, 가장 길게 증가하는 부분 수열의 길이를 구하라.

### 📥 입력 예시

```java
int[] nums = {10, 9, 2, 5, 3, 7, 101, 18};
```

출력: 4 → 부분 수열 \[2, 3, 7, 101]

### ✅ 풀이 1 (최선): DP + 이진 탐색 (O(n log n))

```java
public int lengthOfLIS(int[] nums) {
    List<Integer> list = new ArrayList<>();
    for (int num : nums) {
        int idx = Collections.binarySearch(list, num);
        if (idx < 0) idx = -idx - 1;
        if (idx == list.size()) list.add(num);
        else list.set(idx, num);
    }
    return list.size();
}
```

### ⚠️ 풀이 2 (차선): DP 기반 완전탐색 (O(n^2))

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
    }
    return Arrays.stream(dp).max().getAsInt();
}
```

### 🔄 풀이 3 (대안): Java Stream 으로 DP 시각화

```java
public int lengthOfLIS(int[] nums) {
    int[] dp = new int[nums.length];
    Arrays.fill(dp, 1);
    IntStream.range(0, nums.length).forEach(i ->
        IntStream.range(0, i).forEach(j -> {
            if (nums[i] > nums[j]) dp[i] = Math.max(dp[i], dp[j] + 1);
        })
    );
    return Arrays.stream(dp).max().orElse(1);
}
```

---

(## ✅ 유형 12: 집 도둑 문제 (House Robber)

### 💡 문제 설명

인접한 집은 털 수 없을 때, 도둑이 훔칠 수 있는 최대 금액을 구하라.

### 📥 입력 예시

```java
int[] nums = {2, 7, 9, 3, 1};
```

출력: 12 → 2 + 9 + 1

### ✅ 풀이 1 (최선): DP 사용 (O(n))

```java
public int rob(int[] nums) {
    if (nums.length == 0) return 0;
    if (nums.length == 1) return nums[0];

    int[] dp = new int[nums.length];
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0], nums[1]);

    for (int i = 2; i < nums.length; i++) {
        dp[i] = Math.max(dp[i - 1], dp[i - 2] + nums[i]);
    }
    return dp[nums.length - 1];
}
```

### ⚠️ 풀이 2 (차선): 재귀 + 메모이제이션 없이 (비효율적)

```java
public int rob(int[] nums) {
    return robHelper(nums, nums.length - 1);
}
private int robHelper(int[] nums, int i) {
    if (i < 0) return 0;
    return Math.max(robHelper(nums, i - 2) + nums[i], robHelper(nums, i - 1));
}
```

### 🔄 풀이 3 (대안): 변수만으로 공간 최적화 (O(1) 공간)

```java
public int rob(int[] nums) {
    int prev1 = 0, prev2 = 0;
    for (int num : nums) {
        int temp = prev1;
        prev1 = Math.max(prev2 + num, prev1);
        prev2 = temp;
    }
    return prev1;
}
```

---

(## ✅ 유형 13: 계단 오르기 / 점프 문제 (Climbing Stairs / Jump Game)

### 💡 문제 설명

계단을 1칸 또는 2칸씩 오를 수 있을 때, `n`개의 계단을 오르는 방법의 수를 구하라.

### 📥 입력 예시

```java
int n = 5;
```

출력: 8 → 방법: 1+1+1+1+1, 1+1+1+2, ..., 2+2+1 등 총 8가지

### ✅ 풀이 1 (최선): DP 점화식 (O(n))

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int[] dp = new int[n + 1];
    dp[1] = 1;
    dp[2] = 2;
    for (int i = 3; i <= n; i++) {
        dp[i] = dp[i - 1] + dp[i - 2];
    }
    return dp[n];
}
```

### ⚠️ 풀이 2 (차선): 재귀 방식 (O(2^n)) → 비효율적

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    return climbStairs(n - 1) + climbStairs(n - 2);
}
```

### 🔄 풀이 3 (대안): 변수 2개로 공간 최적화 (O(1))

```java
public int climbStairs(int n) {
    if (n <= 2) return n;
    int prev1 = 1, prev2 = 2;
    for (int i = 3; i <= n; i++) {
        int curr = prev1 + prev2;
        prev1 = prev2;
        prev2 = curr;
    }
    return prev2;
}
```

---

(## ✅ 유형 14: 프린터 / 프로세스 순서 (Printer Queue / Process Scheduling)

### 💡 문제 설명

문서마다 중요도가 부여된 인쇄 큐에서, 특정 문서가 몇 번째로 인쇄되는지 구하라.

### 📥 입력 예시

```java
int[] priorities = {2, 1, 3, 2};
int location = 2;
```

출력: 1 → 문서 2(우선순위 3)가 가장 먼저 출력됨

### ✅ 풀이 1 (최선): Queue + 정렬 비교

```java
public int solution(int[] priorities, int location) {
    Queue<int[]> queue = new LinkedList<>();
    for (int i = 0; i < priorities.length; i++) {
        queue.add(new int[]{i, priorities[i]});
    }
    int order = 0;
    while (!queue.isEmpty()) {
        int[] current = queue.poll();
        boolean hasHigher = queue.stream().anyMatch(x -> x[1] > current[1]);
        if (hasHigher) {
            queue.add(current);
        } else {
            order++;
            if (current[0] == location) return order;
        }
    }
    return -1;
}
```

### ⚠️ 풀이 2 (차선): 우선순위 기준으로 정렬만 사용

```java
public int solution(int[] priorities, int location) {
    List<Integer> sorted = Arrays.stream(priorities)
                                 .boxed()
                                 .sorted(Comparator.reverseOrder())
                                 .collect(Collectors.toList());
    Queue<Integer> queue = new LinkedList<>();
    for (int p : priorities) queue.add(p);

    int idx = 0;
    while (!queue.isEmpty()) {
        int current = queue.poll();
        if (current == sorted.get(0)) {
            sorted.remove(0);
            if (location == 0) return ++idx;
            idx++;
        } else {
            queue.add(current);
        }
        location = (location - 1 + queue.size()) % queue.size();
    }
    return -1;
}
```

### 🔄 풀이 3 (대안): PriorityQueue는 적합하지 않음 (위치 추적 어려움)

* `PriorityQueue`는 우선순위만 고려할 뿐, 인덱스 정보가 없어 해당 문제에 적합하지 않음
* 큐에 인덱스를 함께 저장하는 방식이 안정적임

---

(## ✅ 유형 15: 미로 탐색 (Maze Search - BFS)

### 💡 문제 설명

N×M 크기의 미로가 주어지고, (1,1)에서 (N,M)까지 이동할 때 최단 경로의 길이를 구하라. (1은 이동 가능, 0은 벽)

### 📥 입력 예시

```java
int[][] maze = {
    {1, 0, 1, 1, 1},
    {1, 0, 1, 0, 1},
    {1, 0, 1, 0, 1},
    {1, 1, 1, 0, 1},
    {0, 0, 0, 0, 1}
};
```

출력: 9 (최단 거리)

### ✅ 풀이 1 (최선): BFS (O(N\*M))

```java
public int bfsMaze(int[][] maze) {
    int n = maze.length, m = maze[0].length;
    int[][] dir = {{-1,0},{1,0},{0,-1},{0,1}};
    boolean[][] visited = new boolean[n][m];
    Queue<int[]> queue = new LinkedList<>();
    queue.add(new int[]{0, 0, 1}); // x, y, distance
    visited[0][0] = true;

    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int x = curr[0], y = curr[1], dist = curr[2];
        if (x == n - 1 && y == m - 1) return dist;
        for (int[] d : dir) {
            int nx = x + d[0], ny = y + d[1];
            if (nx >= 0 && ny >= 0 && nx < n && ny < m && maze[nx][ny] == 1 && !visited[nx][ny]) {
                visited[nx][ny] = true;
                queue.add(new int[]{nx, ny, dist + 1});
            }
        }
    }
    return -1;
}
```

### ⚠️ 풀이 2 (차선): DFS (모든 경로 탐색, 비효율적)

```java
public int minSteps = Integer.MAX_VALUE;
public void dfsMaze(int[][] maze, boolean[][] visited, int x, int y, int steps) {
    int n = maze.length, m = maze[0].length;
    if (x == n - 1 && y == m - 1) {
        minSteps = Math.min(minSteps, steps);
        return;
    }
    int[][] dir = {{-1,0},{1,0},{0,-1},{0,1}};
    for (int[] d : dir) {
        int nx = x + d[0], ny = y + d[1];
        if (nx >= 0 && ny >= 0 && nx < n && ny < m && maze[nx][ny] == 1 && !visited[nx][ny]) {
            visited[nx][ny] = true;
            dfsMaze(maze, visited, nx, ny, steps + 1);
            visited[nx][ny] = false;
        }
    }
}
```

### 🔄 풀이 3 (대안): BFS + 거리 누적 배열 활용

```java
public int bfsWithDistance(int[][] maze) {
    int n = maze.length, m = maze[0].length;
    int[][] dist = new int[n][m];
    Queue<int[]> queue = new LinkedList<>();
    queue.add(new int[]{0, 0});
    dist[0][0] = 1;

    int[][] dir = {{-1,0},{1,0},{0,-1},{0,1}};
    while (!queue.isEmpty()) {
        int[] curr = queue.poll();
        int x = curr[0], y = curr[1];
        for (int[] d : dir) {
            int nx = x + d[0], ny = y + d[1];
            if (nx >= 0 && ny >= 0 && nx < n && ny < m && maze[nx][ny] == 1 && dist[nx][ny] == 0) {
                dist[nx][ny] = dist[x][y] + 1;
                queue.add(new int[]{nx, ny});
            }
        }
    }
    return dist[n - 1][m - 1];
}
```

---

(## ✅ 유형 16: DFS/BFS 순회 (Graph Traversal)

### 💡 문제 설명

그래프가 주어졌을 때, 특정 노드에서 시작하여 모든 노드를 방문하는 순서를 DFS 또는 BFS 방식으로 출력하라.

### 📥 입력 예시

```java
int[][] graph = {
    {},         // 0 (사용 안 함)
    {2, 3, 8},  // 1
    {1, 7},     // 2
    {1, 4, 5},  // 3
    {3, 5},     // 4
    {3, 4},     // 5
    {7},        // 6
    {2, 6, 8},  // 7
    {1, 7}      // 8
};
```

출력 (DFS): 1 2 7 6 8 3 4 5

### ✅ 풀이 1 (최선): DFS (재귀 방식)

```java
public void dfs(int v, boolean[] visited, int[][] graph) {
    visited[v] = true;
    System.out.print(v + " ");
    for (int next : graph[v]) {
        if (!visited[next]) {
            dfs(next, visited, graph);
        }
    }
}
```

### ⚠️ 풀이 2 (차선): BFS (Queue 방식)

```java
public void bfs(int start, int[][] graph) {
    boolean[] visited = new boolean[graph.length];
    Queue<Integer> queue = new LinkedList<>();
    queue.offer(start);
    visited[start] = true;

    while (!queue.isEmpty()) {
        int v = queue.poll();
        System.out.print(v + " ");
        for (int next : graph[v]) {
            if (!visited[next]) {
                queue.offer(next);
                visited[next] = true;
            }
        }
    }
}
```

### 🔄 풀이 3 (대안): 인접 행렬 기반 DFS

```java
public void dfsMatrix(int v, boolean[] visited, int[][] adjMatrix) {
    visited[v] = true;
    System.out.print(v + " ");
    for (int i = 1; i < adjMatrix.length; i++) {
        if (adjMatrix[v][i] == 1 && !visited[i]) {
            dfsMatrix(i, visited, adjMatrix);
        }
    }
}
```

---

(## ✅ 유형 17: 네트워크 연결 수 (Connected Components)

### 💡 문제 설명

그래프가 주어졌을 때, 서로 연결된 네트워크의 개수를 구하라. (서로 연결된 노드 집합 수)

### 📥 입력 예시

```java
int n = 3;
int[][] computers = {
    {1, 1, 0},
    {1, 1, 0},
    {0, 0, 1}
};
```

출력: 2

### ✅ 풀이 1 (최선): DFS로 연결 요소 탐색

```java
public int solution(int n, int[][] computers) {
    boolean[] visited = new boolean[n];
    int count = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            dfs(computers, visited, i);
            count++;
        }
    }
    return count;
}
private void dfs(int[][] computers, boolean[] visited, int i) {
    visited[i] = true;
    for (int j = 0; j < computers.length; j++) {
        if (computers[i][j] == 1 && !visited[j]) {
            dfs(computers, visited, j);
        }
    }
}
```

### ⚠️ 풀이 2 (차선): BFS 방식

```java
public int solution(int n, int[][] computers) {
    boolean[] visited = new boolean[n];
    int count = 0;
    for (int i = 0; i < n; i++) {
        if (!visited[i]) {
            bfs(computers, visited, i);
            count++;
        }
    }
    return count;
}
private void bfs(int[][] computers, boolean[] visited, int start) {
    Queue<Integer> queue = new LinkedList<>();
    queue.add(start);
    visited[start] = true;
    while (!queue.isEmpty()) {
        int cur = queue.poll();
        for (int j = 0; j < computers.length; j++) {
            if (computers[cur][j] == 1 && !visited[j]) {
                queue.add(j);
                visited[j] = true;
            }
        }
    }
}
```

### 🔄 풀이 3 (대안): Union-Find 활용 (고급 방식)

```java
public int solution(int n, int[][] computers) {
    int[] parent = new int[n];
    for (int i = 0; i < n; i++) parent[i] = i;

    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            if (i != j && computers[i][j] == 1) {
                union(parent, i, j);
            }
        }
    }
    return (int) IntStream.range(0, n).map(i -> find(parent, i)).distinct().count();
}
private int find(int[] parent, int x) {
    if (parent[x] == x) return x;
    return parent[x] = find(parent, parent[x]);
}
private void union(int[] parent, int a, int b) {
    int rootA = find(parent, a);
    int rootB = find(parent, b);
    if (rootA != rootB) parent[rootB] = rootA;
}
```

---

(## ✅ 유형 18: 전화번호 목록 / 접두어 탐색 (Prefix Search)

### 💡 문제 설명

전화번호 목록이 주어졌을 때, 어떤 번호가 다른 번호의 접두어인 경우가 있는지 확인하라.

### 📥 입력 예시

```java
String[] phone_book = {"119", "97674223", "1195524421"};
```

출력: false → "119"가 다른 번호의 접두어

### ✅ 풀이 1 (최선): 정렬 후 접두어 비교 (O(n log n))

```java
public boolean solution(String[] phone_book) {
    Arrays.sort(phone_book);
    for (int i = 0; i < phone_book.length - 1; i++) {
        if (phone_book[i + 1].startsWith(phone_book[i])) {
            return false;
        }
    }
    return true;
}
```

### ⚠️ 풀이 2 (차선): 이중 반복문으로 비교 (O(n^2))

```java
public boolean solution(String[] phone_book) {
    for (int i = 0; i < phone_book.length; i++) {
        for (int j = 0; j < phone_book.length; j++) {
            if (i == j) continue;
            if (phone_book[j].startsWith(phone_book[i])) return false;
        }
    }
    return true;
}
```

### 🔄 풀이 3 (대안): HashMap 활용 (선형 탐색)

```java
public boolean solution(String[] phone_book) {
    Map<String, Boolean> map = new HashMap<>();
    for (String number : phone_book) map.put(number, true);
    for (String number : phone_book) {
        for (int i = 1; i < number.length(); i++) {
            if (map.containsKey(number.substring(0, i))) return false;
        }
    }
    return true;
}
```

---

(## ✅ 유형 19: 조이스틱 문제 (Greedy Joystick)

### 💡 문제 설명

문자열이 주어졌을 때, 알파벳 A부터 Z까지 조이스틱으로 바꾸는 최소 조작 횟수를 구하라.

* 위/아래는 문자 변경 (A ↔ Z)
* 좌/우는 커서 이동

### 📥 입력 예시

```java
String name = "JEROEN";
```

출력: 56

### ✅ 풀이 1 (최선): Greedy + 최소 좌우 이동 판단

```java
public int solution(String name) {
    int answer = 0;
    int len = name.length();
    int move = len - 1;

    for (int i = 0; i < len; i++) {
        char c = name.charAt(i);
        answer += Math.min(c - 'A', 'Z' - c + 1);

        int next = i + 1;
        while (next < len && name.charAt(next) == 'A') next++;
        move = Math.min(move, i + len - next + Math.min(i, len - next));
    }
    return answer + move;
}
```

### ⚠️ 풀이 2 (차선): 오른쪽으로만 이동 (A 고려 안함)

```java
public int solution(String name) {
    int answer = 0;
    for (int i = 0; i < name.length(); i++) {
        char c = name.charAt(i);
        answer += Math.min(c - 'A', 'Z' - c + 1);
    }
    answer += name.length() - 1;
    return answer;
}
```

### 🔄 풀이 3 (대안): 문자별 위치 기록 후 최소 이동 시뮬레이션

* 복잡성 증가로 인해 실전에서는 최선 풀이 사용 권장
* A 연속 구간을 제외한 최소 이동 경로 직접 계산 필요

---

(## ✅ 유형 20: 숫자 게임 (Number Game - 투 포인터 전략)

### 💡 문제 설명

두 배열 A와 B가 주어졌을 때, B에서 A의 숫자보다 큰 수를 선택하여 최대 승점을 얻는 경우의 수를 구하라.
(프로그래머스 예시 문제)

### 📥 입력 예시

```java
int[] A = {5, 1, 3, 7};
int[] B = {2, 2, 6, 8};
```

출력: 3 → B는 \[6, 8]로 A의 \[5, 3, 7]을 이김

### ✅ 풀이 1 (최선): 정렬 후 투 포인터

```java
public int solution(int[] A, int[] B) {
    Arrays.sort(A);
    Arrays.sort(B);
    int answer = 0, aIdx = 0, bIdx = 0;

    while (aIdx < A.length && bIdx < B.length) {
        if (B[bIdx] > A[aIdx]) {
            answer++;
            aIdx++;
        }
        bIdx++;
    }
    return answer;
}
```

### ⚠️ 풀이 2 (차선): 완전탐색 (모든 경우의 수 시도)

```java
public int solution(int[] A, int[] B) {
    boolean[] used = new boolean[B.length];
    int answer = 0;
    for (int a : A) {
        int maxIdx = -1;
        for (int i = 0; i < B.length; i++) {
            if (!used[i] && B[i] > a) {
                if (maxIdx == -1 || B[i] < B[maxIdx]) maxIdx = i;
            }
        }
        if (maxIdx != -1) {
            used[maxIdx] = true;
            answer++;
        }
    }
    return answer;
}
```

### 🔄 풀이 3 (대안): PriorityQueue 사용 (투 포인터 응용)

* 두 배열을 우선순위 큐에 넣고 가장 작은 수부터 비교하며 처리 가능
* 투 포인터가 더 직관적이고 간단하므로 실전에서는 정렬 방식이 효율적

---
