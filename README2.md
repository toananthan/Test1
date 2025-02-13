
Here’s a list of useful algorithms in Java that are applicable across various domains. These algorithms are foundational and can help you improve problem-solving skills and efficiency in software development.


# Sorting Algorithms
## 1. Bubble Sort

Simple algorithm for sorting small datasets.
Time Complexity: O(n^2)

```java
void bubbleSort(int[] arr) {
    int n = arr.length;
    for (int i = 0; i < n - 1; i++) {
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

```

## 2. Merge Sort

Divide and conquer-based sorting for large datasets.
Time Complexity: O(nlogn)

```java
void mergeSort(int[] arr, int left, int right) {
    if (left < right) {
        int mid = (left + right) / 2;
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, right);
    }
}

void merge(int[] arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    int[] L = new int[n1];
    int[] R = new int[n2];
    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];
    int i = 0, j = 0, k = left;
    while (i < n1 && j < n2) arr[k++] = (L[i] <= R[j]) ? L[i++] : R[j++];
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}
```
## 3. Quick Sort

Divide and conquer-based algorithm; faster for small-to-medium datasets.
Time Complexity:O(nlogn)

```java
void quickSort(int[] arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}

int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = (low - 1);
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
    }
    int temp = arr[i + 1];
    arr[i + 1] = arr[high];
    arr[high] = temp;
    return i + 1;
}

```

# Search Algorithms
## 4. Binary Search
For sorted arrays only.
Time Complexity: O(logn)

```java
int binarySearch(int[] arr, int left, int right, int x) {
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == x) return mid;
        if (arr[mid] < x) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

```

# Graph Algorithms
## 5. Depth-First Search (DFS)

Traverses graph deeply before backtracking.
Time Complexity: O(V+E)

```java
void dfs(int v, boolean[] visited, List<List<Integer>> adjList) {
    visited[v] = true;
    System.out.print(v + " ");
    for (int u : adjList.get(v)) {
        if (!visited[u]) dfs(u, visited, adjList);
    }
}
```

## 6. Breadth-First Search (BFS)

Traverses graph level by level.
Time Complexity: O(V+E)

```java
void bfs(int start, List<List<Integer>> adjList) {
    boolean[] visited = new boolean[adjList.size()];
    Queue<Integer> queue = new LinkedList<>();
    queue.add(start);
    visited[start] = true;
    while (!queue.isEmpty()) {
        int v = queue.poll();
        System.out.print(v + " ");
        for (int u : adjList.get(v)) {
            if (!visited[u]) {
                visited[u] = true;
                queue.add(u);
            }
        }
    }
}

```

# Dynamic Programming
## 7. Fibonacci Sequence

Efficient calculation using memoization.

```Java
int fibonacci(int n, int[] memo) {
    if (n <= 1) return n;
    if (memo[n] == 0) memo[n] = fibonacci(n - 1, memo) + fibonacci(n - 2, memo);
    return memo[n];
}

```

## 8. Knapsack Problem

Finds the maximum value that can fit in a knapsack of given capacity.
```java
int knapsack(int[] weights, int[] values, int capacity, int n) {
    if (n == 0 || capacity == 0) return 0;
    if (weights[n - 1] > capacity) return knapsack(weights, values, capacity, n - 1);
    return Math.max(values[n - 1] + knapsack(weights, values, capacity - weights[n - 1], n - 1),
                    knapsack(weights, values, capacity, n - 1));
}
```
# String Algorithms
## 9. Longest Common Subsequence

Finds the longest subsequence common to two strings.
```java
int lcs(String X, String Y, int m, int n) {
    if (m == 0 || n == 0) return 0;
    if (X.charAt(m - 1) == Y.charAt(n - 1)) return 1 + lcs(X, Y, m - 1, n - 1);
    return Math.max(lcs(X, Y, m, n - 1), lcs(X, Y, m - 1, n));
}
```
## 10. KMP Pattern Matching

Efficient substring search algorithm.
```java
void KMPSearch(String pat, String txt) {
    int m = pat.length();
    int n = txt.length();
    int[] lps = computeLPSArray(pat);
    int i = 0, j = 0;
    while (i < n) {
        if (pat.charAt(j) == txt.charAt(i)) {
            j++;
            i++;
        }
        if (j == m) {
            System.out.println("Found pattern at index " + (i - j));
            j = lps[j - 1];
        } else if (i < n && pat.charAt(j) != txt.charAt(i)) {
            if (j != 0) j = lps[j - 1];
            else i++;
        }
    }
}

int[] computeLPSArray(String pat) {
    int m = pat.length();
    int[] lps = new int[m];
    int len = 0, i = 1;
    lps[0] = 0;
    while (i < m) {
        if (pat.charAt(i) == pat.charAt(len)) lps[i++] = ++len;
        else if (len != 0) len = lps[len - 1];
        else lps[i++] = 0;
    }
    return lps;
}
```
These algorithms are frequently used in competitive programming, software development, and interviews. Let me know if you'd like examples of specific applications or advanced algorithms!

---

Here are some advanced algorithms widely used in solving complex problems, competitive programming, and various domains like artificial intelligence, cryptography, and optimization.

# Graph Algorithms
## 1. Dijkstra's Algorithm

Finds the shortest path from a source to all vertices in a graph.
Time Complexity: O((V+E)logV) using a priority queue.

```java
void dijkstra(int[][] graph, int src) {
    int V = graph.length;
    int[] dist = new int[V];
    boolean[] visited = new boolean[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
    pq.add(new int[]{src, 0});
    while (!pq.isEmpty()) {
        int[] current = pq.poll();
        int u = current[0];
        if (visited[u]) continue;
        visited[u] = true;
        for (int v = 0; v < V; v++) {
            if (graph[u][v] != 0 && !visited[v] && dist[u] + graph[u][v] < dist[v]) {
                dist[v] = dist[u] + graph[u][v];
                pq.add(new int[]{v, dist[v]});
            }
        }
    }
    System.out.println("Shortest distances: " + Arrays.toString(dist));
}
```
## 2. Bellman-Ford Algorithm

Handles graphs with negative weights; detects negative weight cycles.
Time Complexity: O(V⋅E).

```java
void bellmanFord(int[][] edges, int V, int src) {
    int[] dist = new int[V];
    Arrays.fill(dist, Integer.MAX_VALUE);
    dist[src] = 0;
    for (int i = 1; i < V; i++) {
        for (int[] edge : edges) {
            int u = edge[0], v = edge[1], weight = edge[2];
            if (dist[u] != Integer.MAX_VALUE && dist[u] + weight < dist[v]) {
                dist[v] = dist[u] + weight;
            }
        }
    }
    for (int[] edge : edges) {
        int u = edge[0], v = edge[1], weight = edge[2];
        if (dist[u] != Integer.MAX_VALUE && dist[u] + weight < dist[v]) {
            System.out.println("Graph contains negative weight cycle");
            return;
        }
    }
    System.out.println("Shortest distances: " + Arrays.toString(dist));
}
```

## 3. Floyd-Warshall Algorithm

Finds shortest paths between all pairs of vertices.
Time Complexity: O(V^3).

```java

void floydWarshall(int[][] graph) {
    int V = graph.length;
    int[][] dist = new int[V][V];
    for (int i = 0; i < V; i++) {
        System.arraycopy(graph[i], 0, dist[i], 0, V);
    }
    for (int k = 0; k < V; k++) {
        for (int i = 0; i < V; i++) {
            for (int j = 0; j < V; j++) {
                if (dist[i][k] != Integer.MAX_VALUE && dist[k][j] != Integer.MAX_VALUE &&
                    dist[i][k] + dist[k][j] < dist[i][j]) {
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    for (int[] row : dist) System.out.println(Arrays.toString(row));
}
```

## 4. Prim’s Algorithm

Finds the Minimum Spanning Tree (MST) of a graph.
Time Complexity: O((V+E)logV) using a priority queue.

```java
void primsAlgorithm(int[][] graph) {
    int V = graph.length;
    int[] key = new int[V];
    boolean[] mstSet = new boolean[V];
    int[] parent = new int[V];
    Arrays.fill(key, Integer.MAX_VALUE);
    key[0] = 0;
    parent[0] = -1;
    PriorityQueue<int[]> pq = new PriorityQueue<>(Comparator.comparingInt(a -> a[1]));
    pq.add(new int[]{0, 0});
    while (!pq.isEmpty()) {
        int u = pq.poll()[0];
        mstSet[u] = true;
        for (int v = 0; v < V; v++) {
            if (graph[u][v] != 0 && !mstSet[v] && graph[u][v] < key[v]) {
                parent[v] = u;
                key[v] = graph[u][v];
                pq.add(new int[]{v, key[v]});
            }
        }
    }
    for (int i = 1; i < V; i++) System.out.println(parent[i] + " - " + i + ": " + graph[i][parent[i]]);
}
```

# Dynamic Programming
## 5. Longest Increasing Subsequence (LIS)

Finds the length of the longest subsequence where all elements are sorted in increasing order.
Time Complexity: O(n^2) OR O(n log n) with Binary Search.
```java
int lis(int[] arr) {
    int n = arr.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (arr[i] > arr[j]) dp[i] = Math.max(dp[i], dp[j] + 1);
        }
    }
    return Arrays.stream(dp).max().orElse(1);
}
```

## 6. Matrix Chain Multiplication

Determines the most efficient way to multiply a series of matrices.
Time Complexity:O(n^3)
```java
int matrixChainOrder(int[] p) {
    int n = p.length;
    int[][] dp = new int[n][n];
    for (int l = 2; l < n; l++) {
        for (int i = 1; i < n - l + 1; i++) {
            int j = i + l - 1;
            dp[i][j] = Integer.MAX_VALUE;
            for (int k = i; k < j; k++) {
                dp[i][j] = Math.min(dp[i][j], dp[i][k] + dp[k + 1][j] + p[i - 1] * p[k] * p[j]);
            }
        }
    }
    return dp[1][n - 1];
}

```

# String Algorithms
## 7. Trie (Prefix Tree)
Efficient for string search and prefix queries.
```java
class Trie {
    class Node {
        Node[] children = new Node[26];
        boolean isEnd;
    }
    private Node root = new Node();
    
    void insert(String word) {
        Node current = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (current.children[index] == null) current.children[index] = new Node();
            current = current.children[index];
        }
        current.isEnd = true;
    }
    
    boolean search(String word) {
        Node current = root;
        for (char c : word.toCharArray()) {
            int index = c - 'a';
            if (current.children[index] == null) return false;
            current = current.children[index];
        }
        return current.isEnd;
    }
}

```

These advanced algorithms are essential for optimizing solutions to complex problems. 

---

```java
import java.util.regex.*;

public class DecimalValidator {
    public static void main(String[] args) {
        String[] testCases = {
            "LOCAMT=2;257.28; bb; DHFG KDF",  // valid
            "LOCAMT=2;237.28 DHFG KDF",       // valid
            "LOCAMT=2;237.284 DHFG KDF",      // invalid
            "LOCAMT=4;33367.00",              // valid
            "LOCAMT=4;4567",                  // invalid
            "LOCAMT=4;4H67",                  // invalid
            "LOCAMT=4;4H67.88"                 // invalid
        };
        
        for (String attributeVal : testCases) {
            validateDecimal(attributeVal);
        }
    }

    public static void validateDecimal(String attributeVal) {
        // Extracting substring after first semicolon
        String[] parts = attributeVal.split(";", 2);
        if (parts.length < 2) {
            System.out.println("Invalid: No semicolon found - " + attributeVal);
            return;
        }
        
        String valuePart = parts[1].trim();
        
        // Regex pattern explanation:
        // ^(\d+) - Starts with digits before decimal
        // \\.(\d{2}) - Ensures exactly two decimal places
        // (?!\d) - Ensures the third character after decimal is not a digit
        Pattern pattern = Pattern.compile("^(\\d+)\\.(\\d{2})(?!\\d).*");
        Matcher matcher = pattern.matcher(valuePart);
        
        if (matcher.find()) {
            System.out.println("Valid amount: " + matcher.group());
        } else {
            System.out.println("Invalid: " + attributeVal);
        }
    }
}
```
