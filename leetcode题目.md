## 没时间做的每日一题

#### [面试题 16.18. 模式匹配](https://leetcode-cn.com/problems/pattern-matching-lcci/)

```java
class Solution {
        public boolean patternMatching(String pattern, String value) {
            int count_a = 0, count_b = 0;
            for (char ch: pattern.toCharArray()) {
                if (ch == 'a') {
                    ++count_a;
                } else {
                    ++count_b;
                }
            }
            if (count_a < count_b) {
                int temp = count_a;
                count_a = count_b;
                count_b = temp;
                char[] array = pattern.toCharArray();
                for (int i = 0; i < array.length; i++) {
                    array[i] = array[i] == 'a' ? 'b' : 'a';
                }
                pattern = new String(array);
            }
            if (value.length() == 0) {
                return count_b == 0;
            }
            if (pattern.length() == 0) {
                return false;
            }
            for (int len_a = 0; count_a * len_a <= value.length(); ++len_a) {
                int rest = value.length() - count_a * len_a;
                if ((count_b == 0 && rest == 0) || (count_b != 0 && rest % count_b == 0)) {
                    int len_b = (count_b == 0 ? 0 : rest / count_b);
                    int pos = 0;
                    boolean correct = true;
                    String value_a = "", value_b = "";
                    for (char ch: pattern.toCharArray()) {
                        if (ch == 'a') {
                            String sub = value.substring(pos, pos + len_a);
                            if (value_a.length() == 0) {
                                value_a = sub;
                            } else if (!value_a.equals(sub)) {
                                correct = false;
                                break;
                            }
                            pos += len_a;
                        } else {
                            String sub = value.substring(pos, pos + len_b);
                            if (value_b.length() == 0) {
                                value_b = sub;
                            } else if (!value_b.equals(sub)) {
                                correct = false;
                                break;
                            }
                            pos += len_b;
                        }
                    }
                    if (correct && !value_a.equals(value_b)) {
                        return true;
                    }
                }
            }
            return false;
        }
    } 
```





## 剑指offer系列

### 扔骰子：核心状态转换

![image-20200721132843642](/Users/wangkai/Library/Application Support/typora-user-images/image-20200721132843642.png)

```java
class Solution {
    public double[] twoSum(int n) {
        int[][] dp = new int[n+1][6*n+1];
        for(int i = 1;i <= 6;i++){
            dp[1][i] = 1;
        }

        int tot = 6;
        for(int i = 2;i <= n;++i){
            tot*=6;
            for(int j = i;j <= i*6;j++){
                for(int cur = 1;cur <= 6;cur++){
                    if(j - cur <= 0) continue;
                    dp[i][j] += dp[i-1][j-cur];
                }
            }
        }
        double []res = new double[5*n+1];
        for(int j = n;j <= 6*n;j++)
            res[j-n] = dp[n][j]*1.0/tot;

        
        return res;
    }
}
```





### *最长上升路径

```java
class Solution {

    private int[] row = {-1,1,0,0};
    private int[] col = {0,0,-1,1};

    public int longestIncreasingPath(int[][] matrix) {
        if(matrix.length ==0 || matrix[0].length == 0)
            return 0;
        boolean[][] visited = new boolean[matrix.length][matrix[0].length];
        int[][] len = new int[matrix.length][matrix[0].length];
        int max = 0;

        for(int i=0;i<matrix.length;i++){
            for(int j=0;j<matrix[0].length;j++){
                max = Math.max(max,find(matrix,visited,len,i,j));
            }
        }
        return max;
    }
    private int find(int[][] matrix,boolean[][] visited,int[][] len,int x,int y){
        if(visited[x][y])
            return len[x][y];
        len[x][y] = 1;
        for(int i=0;i<4;i++){
            int curX = x + row[i];
            int curY = y + col[i];
            if(curX >=0 && curX < matrix.length && curY >=0 && curY<matrix[0].length && matrix[curX][curY] < matrix[x][y]){
                len[x][y] = Math.max(len[x][y],find(matrix,visited,len,curX,curY)+1);
            }
        }
        visited[x][y] = true;
        return len[x][y];
    }
}
```





## 杂题

### 最长有效括号

https://leetcode-cn.com/problems/longest-valid-parentheses/

- 括号匹配充要条件（规律）
  - 规律1:在一段长度以内，左右括号数量是相同的
  - 规律2:直到最后一个字符为止，每一个前缀的左括号数量都大于右括号的数量

- 题解框架
  - 将字符串分割为一系列字串，确保每一段都不会破坏规则
- 题解思路
  - 根据规律2，确保当前子串的前缀是左括号更多。我们可以一个个字符串向后遍历，使用一个计数器`cnt`统计左右括号的差值，当遇到左边括号计数器就+1，右边就-1，`cnt<0`的时候说明当前子串再向右就一定不合法了，因为至少有一个前缀是不符合规律2的。
  - 通过规律2，子串可以划分成若干段，并且段与段之间是不能跨越的（注意，最后一个字符一定是右括号，除非没有右括号）
  - 再将这些段结合规律1来进行匹配
  - 规律1对应的是括号匹配题目的常见算法，我们可以利用一个栈，遇到做括号，就装入做括号，遇到右括号，就将栈顶的做括号抛弃，再次注意，这个匹配机制是针对每一段而言的
- 实现：栈

```java
class Solution {
    public int longestValidParentheses(String s) {
        int res = 0, n = s.length();
        Stack<Integer> stack = new Stack<>();
        int start = -1;
        for(int i =  0;i < n;i++){
            if(s.charAt(i)=='('){
                stack.push(i);
            }else{
                if(!stack.isEmpty()){
                    stack.pop();
                    int tmp = 0;
                    if(!stack.isEmpty())            
                        tmp = i - stack.peek();
                    else                                //整段都满足情况
                        tmp = i - start;    
                    res = res > tmp?res:tmp;
                }
                else{
                    start = i;
                }
                
            }
        }
        return res;
    }
}
```



实现：左右扫描法（更快，且逻辑更好记）

- 细节：左右括号指针相互追逐，相遇的时候计算一下长度。

```java
public class Solution {
    public int longestValidParentheses(String s) {
        int left = 0, right = 0, maxlength = 0;
        for (int i = 0; i < s.length(); i++) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * right);
            } else if (right > left) {
                left = right = 0;
            }
        }
        left = right = 0;
        for (int i = s.length() - 1; i >= 0; i--) {
            if (s.charAt(i) == '(') {
                left++;
            } else {
                right++;
            }
            if (left == right) {
                maxlength = Math.max(maxlength, 2 * left);
            } else if (left > right) {
                left = right = 0;
            }
        }
        return maxlength;
    }
}
```



## 最长交叉路径

- 题意：从树的**任意一个节点出发**，每次向下遍历都必须交叉进行，求最长距离。
- 题解：设置一个全局最大值，每次保存最佳状态即可，要注意的是，这题是不能对每一个节点都进行深搜的，这种暴力做法会导致超时，对此我们可以剪枝，每次遇到没有‘交替’子节点的时候，就把路径长度设为1，相当于新的一次遍历，这样的话，每个节点其实只会被遍历一次。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    int max = 0;

    void dfs(TreeNode root, int depth, int state){
        max = Math.max(max, depth);
        if(root == null) return;
      
        if(state == 0 && root.right == null) dfs(root.left, 1, state);
        else if(state == 1 && root.left == null) dfs(root.right, 1, state);
        
        else if(state == 0) {
            dfs(root.right, depth+1, 1);
            dfs(root.left, 1, 0);
        }
        else {
            dfs(root.left, depth+1, 0);
            dfs(root.right, 1, 1);
        }
    }
    public int longestZigZag(TreeNode root) { 
        if(root == null || root.left == null && root.right==null) return 0;
        dfs(root, 0, 0);
        dfs(root, 0, 1);         
        return max;
    }
}
```



## 重建二叉树之前后序找到随便一个二叉树

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public TreeNode constructFromPrePost(int[] pre, int[] post) {
        return helper(pre,post,0,pre.length-1,0,post.length-1);
    }
    public TreeNode helper(int[] pre,int[] post,int prestart,int preend,int poststart,int postend){
        if(prestart > preend || poststart > postend) return null;

        TreeNode root = new TreeNode(pre[prestart]);
        if(prestart == preend) return root;
        
        int index = 0;
        while(post[index] != pre[prestart + 1]){
            index++;
        } 
        root.left = helper(pre, post, prestart+1, prestart+index+1-poststart, poststart, index);
        root.right = helper(pre, post, prestart+index+2-poststart, preend, index+1, postend);
        return root;
    }
}
```



## 前序中序重建

```java
 public TreeNode dfs(int[]preorder, int[]inorder, int prel, int prer, int inl, int inr){
        if(prel > prer) return null;
        TreeNode root = new TreeNode(preorder[prel]);
        if(prel == prer) return root;
        
        int pos = inl;
        while(inorder[pos] != pre[prel]) pos++;
        root.left = dfs(preorder, inorder, prel+1, prel+pos-inl, inl, pos-1);
        root.right = dfs(preorder, inorder, prel+1+pos-inl, prer, pos+1, inr);
        return root;
    }
```



## 最多参与的会议次数

https://leetcode-cn.com/problems/maximum-number-of-events-that-can-be-attended/

- 题目意思：给定一个列表，里面是[[start0, end0], [start1, end1], ...]，代表每个会议的起止日期，规定一天只能参与一个会议，求在min(start)～max(end)这些天内，能够参与的会议的最大值
- 选取规则
  -  对于相同起始时间的会议而言，越早结束的会议应该越早进行，但是一天只能进行一次，因此是贪心算法。
  - 对于截止时间已经过了，却没有进行的会议，应该被淘汰

```java
class Solution { 
   public int maxEvents(int[][] events) { 
        Arrays.sort(events, (o1,o2)->o1[0]-o2[0]);
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        int i = 0, cur = 1, res = 0;
        int n = events.length;
        while(i < n || !pq.isEmpty()){
            while(i < n&& events[i][0] == cur){
                pq.add(events[i][1]);
                i++;
            }
            while(!pq.isEmpty() && pq.peek() < cur){
                pq.poll();
            }
            if(!pq.isEmpty()){
                pq.poll();
                res++;
            }
            cur++;
        }
        return res;
    } 
}s
```

- 实现：以起始时间排序，然后暴力遍历，可以通过挖坑填补的方式，但是时间复杂度是 o(n*events.length)

- 优化实现：可以使用优先队列优化，比较麻烦，要用一个last来记录当前会议的起始日期

  - 用一个last变量作为当前日期的指示器
  - 在一天之内，如果当前会议的开始日期等于last，说明last找到了截止日期，就将当前会议的截止日期加入到队列中
  - 如果存放在队列中的截止日期<当前日期，说明该会议已经不会再被访问，此时应该把这些pop出来

  - 终止条件：已经没有会议可以加入队列了，并且队列中也没有未处理的会议



## 颜色分类

```java
class Solution {
    public void swap(int []nums, int l, int r){
        int tmp = nums[l];
        nums[l] = nums[r];
        nums[r] = tmp;
    }
    public void sortColors(int[] nums) {
        int n = nums.length;
        int p0 = 0, p2 = n-1;
        int cur = 0;
        while(cur <= p2){
            if(nums[cur] == 0){
                swap(nums, cur, p0);
                p0++;
                cur++;
            }else if(nums[cur] == 2){
                swap(nums, cur, p2);
                p2--;
            }else{
                cur++;
            }
        }

    }
}
```



## 字典序第k小的数字

- trie树应用
- 从前往后以某个固定前缀对数据范围进行查询，范围比triek大，说明这个位置到

```java
class Solution {
    public int findKthNumber(int n, int k) {
       int l = 1, r = 9;
       while(k > 0){
           for(int i = l;i <= r;i++){
                int num = getRange(i, n); 
                if(k > num){
                    k -= num;
                }
                else{
                    k--;
                    if(k == 0) return i;
                    l = i*10;
                    r = i*10+9;
                    break;
                }
            }
       }
       return 0;
    }
    int getRange(int num, int max ){
        long left = (long)num*10 ;
        long right = (long)num*10 + 9;
        long reduce = 1;
        while(max >= left){
            if(right <= max){
                reduce += (right-left+1);
            }else{
                reduce += (max-left+1); 
            } 
            right = right*10+9;
            left = left*10;
        } 
        return (int)reduce;
    }
}
```



## 首尾拼接的字符串判断回文

- 给定字符串长度为n，判断这个字符串首尾拼接之后能否形成一个回文

- 环形数组和条状数组的区别：环形数组的其实位置可以是任意一个位置，比如 [0, (n-1)],  [1, (n-1), 0], [2, (n-1), 0, 1]...

- 这个可以通过将前n-1个字符拼接到原字符串的后面，形成(2n-1)长度的字符串，然后直接判断 [0, n-1], [1, n],[2,n+1]...

  

```java
class Solution{
  boolean judge(String sb){
    int mid = sb.length()/2;
    for(int i = 0, j = sb.length()-1;i < j; i++, j-- ){
      if(sb.charAt(i) != sb.charAt(j)) return false;
    }
    return true;
  }
  boolean isPerm(String s){
    StringBuilder sb = new StringBuilder(s);
    sb.append(s.substr(0, s.length()-1));
    for(int i = 0;i < s.length();i++){
      if(judge(sb.substr(i, i+s.length() ) ) ) return true;
    }
    return false;
  }
}
```

## 过河问题通解

- 一个手电筒，一个独木桥，过桥必须要手电筒，独木桥最多两个人走，给定n个人的过桥时间，求过桥最短时间
- 两种方案，考虑送两个最慢的人到对岸的情况：
  - 最快的人来回跑，每次接送最慢人：t =2* s[0]+s[n-1]+s[n-2]
    - 缺点：每次只能接送一个人，多了很多次加法
    - 优点：回来速度快，适用于一个飞毛腿和一群乌龟
  - 由最快和次快充当跑腿，最快的送次快的过去，然后最快的回来，将手电筒递给最慢的两个人，两人过桥之后，把手电交给次快的人: t = s[0]+2*s[1]+s[n-1]
    - 缺点：有一半的回程时次快的人完成的
    - 优点：减少很多浪费的时间，每次都让时间最长的两个人过桥，只用了一个人的时间，比较合理

```java
public int minTime(int []sp){
  Arrays.sort(sp);
  int n = sp.length;
  int res = 0;
  for(int i = 0;i < n;i++){
    res += min(2*sp[0]+sp[n-1]+sp[n-2], sp[0]+2*sp[1]+sp[n-1]);
  }
  return res;
}
```



## 字典树动态规划：恢复空格

```java
class Solution {
    class TreeNode{
        TreeNode []next;
        boolean isEnd ;
        TreeNode(){
            next = new TreeNode[26];
            isEnd = false;
        }
        void insert(String s){
            TreeNode cur = this;
            for(int i = s.length()-1;i >= 0;--i){
                int t = s.charAt(i) - 'a';
                if(cur.next[t] == null) //特别注意这里 
                    cur.next[t] = new TreeNode();            
                cur = cur.next[t];
            }
            cur.isEnd = true;
        }
    }
    public int respace(String[] dictionary, String sentence) {
        TreeNode root = new TreeNode();
        for(String word:dictionary){
            root.insert(word);
        }
        int n = sentence.length();
        int[] dp = new int[n+1];
        for(int i = 1;i <= n;i++){
            dp[i] = dp[i-1]+1;
            TreeNode cur = root;
            for(int j = i;j >= 1;--j){
                int t = sentence.charAt(j-1) - 'a';
                if(cur.next[t] == null){
                    break;
                }else if(cur.next[t].isEnd){
                    dp[i] = Math.min(dp[i], dp[j-1]);
                }
                if(dp[i]==0) break;
                cur = cur.next[t];
            }
        }
        return dp[n];
    }
}
```



## 最大异或：字典树灵活运用

```java
class Solution {

    static final int MAX_BIT = 31;
    static final int MIN_BIT = 0;

    class Node {
        Node[] children = new Node[2];
        void insert(int val){
            Node cur = this;
            for(int i = MAX_BIT;i >= MIN_BIT;i--){
                int bit = (val>>i) & 1;
                if(cur.children[bit] == null){
                    Node node = new Node();                
                    cur.children[bit] = node;
                }
                cur = cur.children[bit];
            }
        }
        int xor(int val){
            Node cur = this;
            int sum = 0; 
            for(int i = MAX_BIT;i >= MIN_BIT;i--){
                int bit = (val>>i) & 1;
                if(cur.children[bit^1] != null){
                    sum += 1<<i;
                    cur = cur.children[bit^1];
                }else{
                    cur = cur.children[bit];
                } 
            }
            return sum;
        }
    }

    public int findMaximumXOR(int[] nums) {
        Node node = new Node();
        for(int num:nums){
            node.insert(num);
        }
        int max = 0;
        for(int num:nums){
            max = Math.max(max, node.xor(num));
        }
        return max;
    } 

}
```



## 区间重叠问题

- 最少需申请的会议室

  ```java
  class Solution {
      
      public int[][] merge(int[][] intervals) {  
          if(intervals.length == 0 || intervals[0].length == 0) return intervals;
          Arrays.sort(intervals, Comparator.comparingInt(o -> o[0]));
          /* new Comparator<int[]>{ 
          		@Override public int comparable(int[] a, int[] b) {
          			if(a[0] > b[0]) return -1;
          			else if(b[0] > a[0]) return 1;
          			else if(a[1] > b[1]) return -1;
          			else if(a[1] < b[1]) return 1;
          			else return 0;
          		}
          }
        	*/
          Map<Integer, Integer> m = new HashMap<>();
          
          int cur = intervals[0][0];
          int pre = cur;
          int pos = 0;
          m.put(intervals[0][0], intervals[0][1]);
          for(int i = 1;i < intervals.length;i++){
              int maxEnd = m.get(cur);
              if(maxEnd < intervals[i][0]){
                  m.put(intervals[i][0], intervals[i][1]);
                  cur = intervals[i][0];
              }
              else{
                  if(maxEnd < intervals[i][1])
                      m.replace(cur, intervals[i][1]);
              }
          }
          int n = m.size();
          int [][]res = new int[n][2];
          int i =0 ;
          for(Map.Entry<Integer, Integer> entry:m.entrySet()){
                  res[i][0] = entry.getKey();
                  res[i][1] = entry.getValue();
                  ++i;
          }
          return res;
      }
  }
  ```

  

- 最多可参与的会议数：上下车

```java
class Solution { 
   public int maxEvents(int[][] events) { 
        Arrays.sort(events, (o1,o2)->o1[0]-o2[0]);
        PriorityQueue<Integer> pq = new PriorityQueue<>();
        int i = 0, cur = 1, res = 0;
        int n = events.length;
        while(i < n || !pq.isEmpty()){
            while(i < n&& events[i][0] == cur){
                pq.add(events[i][1]);
                i++;
            }
            while(!pq.isEmpty() && pq.peek() < cur){
                pq.poll();
            }
            if(!pq.isEmpty()){
                pq.poll();
                res++;
            }
            cur++;
        }
        return res;
    } 
}
```



## 勇者斗恶龙

```java
/*
通过结果问初始状态最小值的问题都要考虑从后往前进行

dp[i][j] 表示在(i,j)坐标需要的最小体力，走完一格之后，体力至少剩下1
dp[m-1][n-1] = 1:1-dungen[m-1][n-1];
状态转换
dp[i-1][j]>0? <= 1:dp[i][j]-dungenon[i][j];
dp[i][j-1]>0? <= 1:1-dp[i][j]-dungenon[i][j];
*/
```



## LRU缓存，可以考虑泛型，线程安全。

```java
class DoubleLink{
    DoubleLink prev;
    DoubleLink next;
    int key;
    int value;
    DoubleLink(){};
    DoubleLink(int key, int value){this.key = key; this.value = value;}
}
class <T1, T2>LRUCache {
    DoubleLink head;
    DoubleLink tail;
    int capacity;
    int len;
    Map<T1, T2> m;

    public LRUCache(int capacity) {
        this.capacity = capacity;
        m = new HashMap<>();
        head = new DoubleLink();
        tail = new DoubleLink();
        head.prev = null;
        head.next = tail;
        tail.prev = head;
        tail.next = null;
    }
    
    public int get(int key) {
        if(!m.containsKey(key)) return -1;
        DoubleLink cur = m.get(key); 
        remove(cur);
        addToHead(cur);
        return cur.value;
    }
    
    public void put(int key, int value) {
        if(get(key) != -1){
            DoubleLink node = m.get(key);
            node.value = value;
            m.replace(key, node);
        }else{
            len++;
            
            if(len > capacity){
                int k = tail.prev.key;
                m.remove(k);
                removeTail();
                len--;
            } 
            DoubleLink node = new DoubleLink(key, value);
            m.put(key, node);
            addToHead(node);
        }
    }
    void addToHead(DoubleLink node){
        node.prev = head;
        node.next = head.next; 
        head.next.prev = node;
        head.next = node;
    }
    void remove(DoubleLink node){ 
        node.prev.next = node.next;
        node.next.prev = node.prev;
    }
    void removeTail(){
        DoubleLink node = tail.prev ;
        remove(node);
    }
}
```



## N皇后

- 做法和普通dfs是一样的，只是方向有八个，最主要的区别在于对角线的性质
- 每个N*N的矩阵中，都会有2N条主对角线，以及2N条反对角线
- 如果要遍历每一条对角线，很明显是复杂度爆炸的，因此只能优化
- **这一步是最关键的**：我们不难发现，一条对角线就会对应一个直线方程，因此我们可以通过设置两个2N大小的数组来记录每个主对角和从对角线是否出现过一个点，不过要注意一个点，就是在左下对角线的时候，b < 0，因此我们需要进行一个位移
  - 主对角线：y = x+b =>  y-x = b => y-x+N = b+N
  - 从对角线：y = -x+c => y+x = c
- 然后就是dfs回溯的过程

```java
class Solution {
    int []positive;
    int []negative;
    int []coordinX;
    int []coordinY;
  
    List<StringBuilder> graph;
    int N;
  
    public List<StringBuilder> build(int n){
        List<StringBuilder> list = new ArrayList<>();
        for(int i = 0;i < n;i++){
            StringBuilder sb = new StringBuilder("");
            for(int j = 0;j < n;j++){
                sb.append('.');
            }  
            list.add(sb); 
        }
        return list;
    }
    boolean check(int x, int y){
        if(positive[y-x+N] == 1 || negative[y+x] == 1 ||coordinX[x]==1|| coordinY[y] == 1)
            return false;
        return true;
    }
    void setPos(int pos, int neg, int x, int y, int flag){
        coordinX[x] = flag;
        coordinY[y] = flag;
        positive[pos] = flag;
        negative[neg] = flag;
    }
    int res;
    public void backtrack(int n, int row, List<String> sol){
        if(row == n){
            res++;
            return;
        }
        for(int col = 0;col < n;col++){
            if(check(row, col)){
                setPos(col-row+n, col+row, row, col, 1); 
                backtrack(n, row+1, sol); 
                setPos(col-row+n, col+row, row, col, 0);
            }
        }
    }
    public int totalNQueens(int n) {
        N = n; 
        List<String> sol = new ArrayList<>();
        graph = build(n); 
        positive = new int[2*n+1];
        negative = new int[2*n+1];
        coordinX = new int[n+1];
        coordinY = new int[n+1];
        backtrack(n,0,sol);
        return res;
    }
}
```

### 最大岛屿面积

- 四面加和

```java
int dfs(int [][]grid, int x, int y){
    if(x >= n || x < 0 || y >= m || y < 0 || grid[x][y] == 0) return 0;
    int tmp = 1;
    for(int i=0;i<4;i++)
    {
      grid[x][y] = 0;
      tmp += maxSize(grid, x+dx[i], y+dy[i]);
    }
    return tmp;
}
int maxSize(int [][]grid){
  int ans = 0;
	for(int i = 0;i < n;i++){
    for(int j = 0;j < m;j++){
      if(grid[i][j] == 1)	
	      ans = Math.max(ans, dfs(grid, i, j));
    }
  }
  return ans;
}

```



## 最小跳跃次数

```c++
class Solution {
private:
    int f[10000000 + 7];
    int maxdis[10000000 + 7];
public:
    int minJump(vector<int>& jump) {
        int n = jump.size();
        int w = 0;
        int ans = 1000000000;

        for (int i=1; i<=n; ++i) {
            f[i] = 1000000000;
            maxdis[i] = 0;
        }
        f[1] = 0;
        maxdis[0] = 1;

        for (int i=1; i<=n; ++i) {
            if (i > maxdis[w]) { // 更新单调指针
                ++w;
            }
            f[i] = min(f[i], w+1); // 用 maxdis[w] 更新 f[i]
            int next = i + jump[i-1]; // 第一步跳跃更新

            if (next > n) {
                ans = min(ans, f[i] + 1);
            } else if (f[next] > f[i] + 1) {
                f[next] = f[i] + 1;
                maxdis[f[next]] = max(maxdis[f[next]], next);
            }
        }

        return ans;
    }
};
 
```





### kmp算法

- 可以解决
  - 最大重复前缀
  - 统一字符串模式匹配

```java
void kmp(String s, String p){
        char[] sc = (" "+s).toCharArray();
        char[] pc = (" "+p).toCharArray();
        int len1 = sc.length, len2 = pc.length;
        int[]next = new int[len2+1];
        int i = 0, j = 0;
        while(i < len2){
            if(j == 0||pc[i] == pc[j]){
                next[i] = j;
                i++;j++;
            }
            else j = 0;
        }
        int cnt = 0;
        j = 0;
        i = 0;
        while(i < sc.length){
            if(j == 0||sc[i] == pc[j]){
                i++;
                j++;
            }else{
                j = next[j];
            }
            if(j == len2-1){
                cnt++;
            }
        }
    }
```



## 洗牌算法

```java
int mix(int num){
  int []rnd = new int[num+1];
  Random random = new Random();
  for(int i = num+1;i >= 0;i++){
    int x = random.nextInt();
    rnd[x] = i;
    rnd[i] = x; 
  }
  
}
```



## 寻宝问题

```java
class Solution {
    int[] dx = {1, -1, 0, 0};
    int[] dy = {0, 0, 1, -1};
    int n, m;

    public int minimalSteps(String[] maze) {
        n = maze.length;
        m = maze[0].length();
        // 机关 & 石头
        List<int[]> buttons = new ArrayList<int[]>();
        List<int[]> stones = new ArrayList<int[]>();
        // 起点 & 终点
        int sx = -1, sy = -1, tx = -1, ty = -1;
        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                if (maze[i].charAt(j) == 'M') {
                    buttons.add(new int[]{i, j});
                }
                if (maze[i].charAt(j) == 'O') {
                    stones.add(new int[]{i, j});
                }
                if (maze[i].charAt(j) == 'S') {
                    sx = i;
                    sy = j;
                }
                if (maze[i].charAt(j) == 'T') {
                    tx = i;
                    ty = j;
                }
            }
        }
        int nb = buttons.size();
        int ns = stones.size();
        int[][] startDist = bfs(sx, sy, maze);

        // 边界情况：没有机关
        if (nb == 0) {
            return startDist[tx][ty];
        }
        // 从某个机关到其他机关 / 起点与终点的最短距离。
        int[][] dist = new int[nb][nb + 2];
        for (int i = 0; i < nb; i++) {
            Arrays.fill(dist[i], -1);
        }
        // 中间结果
        int[][][] dd = new int[nb][][];
        for (int i = 0; i < nb; i++) {
            int[][] d = bfs(buttons.get(i)[0], buttons.get(i)[1], maze);
            dd[i] = d;
            // 从某个点到终点不需要拿石头
            dist[i][nb + 1] = d[tx][ty];
        }

        for (int i = 0; i < nb; i++) {
            int tmp = -1;
            for (int k = 0; k < ns; k++) {
                int midX = stones.get(k)[0], midY = stones.get(k)[1];
                if (dd[i][midX][midY] != -1 && startDist[midX][midY] != -1) {
                    if (tmp == -1 || tmp > dd[i][midX][midY] + startDist[midX][midY]) {
                        tmp = dd[i][midX][midY] + startDist[midX][midY];
                    }
                }
            }
            dist[i][nb] = tmp;
            for (int j = i + 1; j < nb; j++) {
                int mn = -1;
                for (int k = 0; k < ns; k++) {
                    int midX = stones.get(k)[0], midY = stones.get(k)[1];
                    if (dd[i][midX][midY] != -1 && dd[j][midX][midY] != -1) {
                        if (mn == -1 || mn > dd[i][midX][midY] + dd[j][midX][midY]) {
                            mn = dd[i][midX][midY] + dd[j][midX][midY];
                        }
                    }
                }
                dist[i][j] = mn;
                dist[j][i] = mn;
            }
        }

        // 无法达成的情形
        for (int i = 0; i < nb; i++) {
            if (dist[i][nb] == -1 || dist[i][nb + 1] == -1) {
                return -1;
            }
        }
        
        // dp 数组， -1 代表没有遍历到
        int[][] dp = new int[1 << nb][nb];
        for (int i = 0; i < 1 << nb; i++) {
            Arrays.fill(dp[i], -1);
        }
        for (int i = 0; i < nb; i++) {
            dp[1 << i][i] = dist[i][nb];
        }
        
        // 由于更新的状态都比未更新的大，所以直接从小到大遍历即可
        for (int mask = 1; mask < (1 << nb); mask++) {
            for (int i = 0; i < nb; i++) {
                // 当前 dp 是合法的
                if ((mask & (1 << i)) != 0) {
                    for (int j = 0; j < nb; j++) {
                        // j 不在 mask 里
                        if ((mask & (1 << j)) == 0) {
                            int next = mask | (1 << j);
                            if (dp[next][j] == -1 || dp[next][j] > dp[mask][i] + dist[i][j]) {
                                dp[next][j] = dp[mask][i] + dist[i][j];
                            }
                        }
                    }
                }
            }
        }

        int ret = -1;
        int finalMask = (1 << nb) - 1;
        for (int i = 0; i < nb; i++) {
            if (ret == -1 || ret > dp[finalMask][i] + dist[i][nb + 1]) {
                ret = dp[finalMask][i] + dist[i][nb + 1];
            }
        }

        return ret;
    }

    public int[][] bfs(int x, int y, String[] maze) {
        int[][] ret = new int[n][m];
        for (int i = 0; i < n; i++) {
            Arrays.fill(ret[i], -1);
        }
        ret[x][y] = 0;
        Queue<int[]> queue = new LinkedList<int[]>();
        queue.offer(new int[]{x, y});
        while (!queue.isEmpty()) {
            int[] p = queue.poll();
            int curx = p[0], cury = p[1];
            for (int k = 0; k < 4; k++) {
                int nx = curx + dx[k], ny = cury + dy[k];
                if (inBound(nx, ny) && maze[nx].charAt(ny) != '#' && ret[nx][ny] == -1) {
                    ret[nx][ny] = ret[curx][cury] + 1;
                    queue.offer(new int[]{nx, ny});
                }
            }
        }
        return ret;
    }

    public boolean inBound(int x, int y) {
        return x >= 0 && x < n && y >= 0 && y < m;
    }
}
```



```java
//转链表，头插法
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    TreeNode pre; 
    public void flatten(TreeNode root) {
        if(root == null) return;
        flatten(root.right);
        flatten(root.left);
        root.right = pre;
        root.left = null;
        pre = root;
    }
}
```





## 二分图判断

- 着色法+bfs

## 考过的题目

- 字典序第k小
- k个一组翻转
- N皇后
- 单词拆分
- nim游戏
- 搜索旋转数组（目标值，最小值，中位数）
- 岛屿最大面积
- 验证回文字符串
- 最小覆盖子串
- 最小跳跃次数



## 奇偶问题

- 华为：让奇数位于奇数位置，偶数位于偶数位置，对数据顺序不作要求，O(n)时间，O(1)空间
  - 用一个奇数指针遍历奇数位置，用一个偶数指针遍历偶数位置，看到不合规范的就交换。
- 腾讯PCG：给定数组，让奇数统一位于左边，偶数统一位于右边，保持奇数和偶数内部相对顺序不变
  - o(n)时间+o(n)空间：新建数组按奇偶顺序放入即可
  - o(n^2)时间+o(1)空间：插入排序，如果当前位置nums[i]是奇数，那么就从i到0遍历，然后将元素往后挪，直到遇到奇数就往后插入
- 字节：给定链表，奇数位置是升序，偶数位置是降序，请用原地操作对链表进行重新排序
  - 将奇数节点和偶数节点相连，分为两个链表
  - 将偶数链表翻转
  - 利用双指针将奇偶链表进行合并
  - 拓展：给定链表和数字k，链表中子序列 i+k，i+2*k... 对应的节点都是有序的，秋



## 字符串dp问题，一般使用图表法会好解决

### leetcode 交错字符串



![image-20200718144833128](/Users/wangkai/Library/Application Support/typora-user-images/image-20200718144833128.png)

- 状态表示：`dp[i][j]`代表`s[0-(i-1)]+p[0-(j-1)]==s3.substr[0-(i+j-1)]` 匹配成功

- 状态转换：

  - 如图所示，`dp[i][j]`所对应的字符串只能由`上面`或`左边`的字符拼接，只要上面或者左边是不能匹配，说明从那个状态开始的状态都是不可以的，因此当前状态依赖于上一个状态是否匹配成功, 以及当前位置的字符是否有办法匹配。
    - `if(s1[i-1] == s3[i+j-1]) dp[i][j] = dp[i][j-1]`
    - `if(s1[j-1] == s3[i+j-1]) dp[i][j] |= dp[i-1][j]` 

  ```java
  class Solution {
      public boolean isInterleave(String s1, String s2, String s3) {
          if(s1 == null || s2 == null || s3 == null) return false;
          int n = s1.length();
          int m = s2.length(); 
          if(n + m != s3.length()) return false; 
          boolean[][] dp = new boolean[n+1][m+1];
          dp[0][0] = true;
  
          for(int i = 1;i <= n;i++) {
              if(s1.charAt(i-1) != s3.charAt(i-1)) break;
              dp[i][0] = true;
          }
          for(int i = 1;i <= m;i++) {
              if(s2.charAt(i-1) != s3.charAt(i-1)) break;
              dp[0][i] = true;
          }
  
          for(int i = 1;i <= n;i++){
              for(int j = 1;j <= m;j++){
                  if(s1.charAt(i-1) == s3.charAt(i+j-1)) dp[i][j] = dp[i-1][j];
                  if(s2.charAt(j-1) == s3.charAt(i+j-1)) dp[i][j] |= dp[i][j-1];
              }
          }   
          return dp[n][m];
      } 
  }
  ```

  

## 戳气球类型（区间dp)

```java
// dp[i][j] := 取走(i,j)的最大值，其中i,j是取不到的（开区间）
// 这类题目，取的时候会受到两边的影响，因此换个角度: 假设要取k
// 假设左边区间(i,k)已经取完，右边区间(k,j)也已经取完就可以快速计算出整个区间(i,j)的值
// 所以先枚举小区间再计算大区间

arr = {1, nums, 1};
d = arr.size();

for (w: 2 to d - 1): // 枚举区间大小，从2开始
    for (i: 0 to (i + w) < d): // 枚举起始位置
       j = i + w;
       for (k: i + 1 to j - 1): // 枚举中间位置
            .... // 进行处理
```



## 搜索二叉树删除节点

```java
TreeNode dfs(TreeNode root, int key){
	if(root.val < key){
    return dfs(root.right, key);
  }else if(root.val > key){
    return dfs(root.left, key);
  }else{
    if(root.left == null) return root.right;
    else if(root.right == null) return root.left;
    else{
      TreeNode newRoot = findroot(root.right);
      newRoot.right = deleteOldpos(newRoot);
      newRoot.left = root.left;
    }
  }
  TreeNode findroot(TreeNode oldRoot){
    while(oldRoot.left!=null){
      oldRoot = oldRoot.left;
    }
    return root;
  }
  TreeNode deleteOldpos(TreeNode node){
    if(node.left == null) return node.right;
    node.left = deleteOldpos(node.left);
    return node;
  }
}
```



## n数之和

```java
// 背包问题

//解1: 全排列（全子集），复杂度为2^n，只能解决有限数量的难题
List<List<Integer>> res = new ArrayList<>();
for(int i = 1;i < 1<<n;i++){
  int tmp = i;
  int sum = 0;
  List list = new ArrayList<>();
	for(int j = i, k = 1;j != 0;j >>= 1, k++){
		 if(j == 1)
     		sum+=k;
     list.add(k);
  }
  if(sum == target){
    res.add(list);
  }
}
  
```



## 排序算法

- n^2：都会形成遍历到第i个数字就确保前面i个数据已经排好序。
  - 冒泡：必须进行n^2次遍历，每次比较当前位置元素和前一个位置元素的大小情况，如果小了，就前面交换，否则位置不变

    ```java
    void bubbleSort(int []a){
      int n = a.length;
      	for(int i = 0;i < n;i++){
          for(int j = i+1;j < n;j++){
            if(a[i] > a[j]){
              int tmp = a[i];
              a[i] = a[j];
              a[j] = tmp;
            }
          }
        }
    }
    ```

    

  - 插入：到第i个位置的时候，倒序遍历前面所有元素，看看有没有更小的，稳定

    ```java
    void insertSort(int []a){
      int n = a.length;
      for(int i = 1;i < n;i++){
        int tmp = a[i];
        for(int j = i-1;j>=0;j--){
          if(tmp < a[j]){
            a[j+1] = a[j];
          }else{
            a[j+1] = tmp;
            break;
          }
          
        }
      }
    }
    ```

    

  - 选择：第i个位置选择第i大的元素，不稳定

    ```java
    void selectSort(int []a){
      int n = a.length;
      for(int i = 0;i < n;i++){
        for(int j = i+1;j< n;j++){
          if(a[j] < a[i]){
            int tmp = a[j];
            a[j] = a[i];
            a[i] = tmp;
          }
        }
      }
      
    }
    ```

- n^1.1
  - 希尔排序，从大区间到小区间。
    - 先排 (1, n/2, n)
    - 接着排(1, n/4, n/2, 3n/4, n)
    - 以此类推
- nlogn
  - 快排（最坏n^2，最好nlogn，平均nlogn）：不稳定，最坏就是给定数组排已经有顺序，最好就是每次枢纽都位于二分位置
    - 快速选择(最坏kn，最好n，平均n)：最坏就是每次只能淘汰一个数字，这样需要k轮，最好就是一次选中，平均就是每次排除一半的元素，按照等比数列可以计算出平均也是n
  - 归并（都是nlogn）：稳定，空间占用大，用于逆序对情况多一些，（链表归并可以原地操作，断开和链接的细节麻烦）。
  - 堆排序(都是nlogn，但是不稳定，小顶堆为例子)：
    - 主要是up和down操作，在插入的时候，插入到最后一个节点，然后使用up和父节点对比，在淘汰的时候，先把最后一个节点交换到头部，然后使用down，选出最大的头部。
    - 注意，大顶堆是最大值在上面，小顶堆是最小值在上面，因此，正序要用小顶堆，倒序要用大顶堆
    - 在topk问题中，**我们要选择小顶堆**，因为每次替换掉的是最小值



## 链表排序

- 快排

  ```java
  class Solution {
      public ListNode quickSortList(ListNode head) {
          if(head==null||head.next==null) return head;
          ListNode bnode, pnode;
          bnode = null;
          pnode = null;
          while(head.next!=null){
              ListNode node = head.next;
              head.next = node.next;
              if(node.val<head.val){
                  if(pnode==null){
                      pnode = node;
                      pnode.next = null;
                  } else{
                      node.next = pnode;
                      pnode = node;
                  }
  
              }
              if(node.val>=head.val){
                  if(bnode==null){
                      bnode = node; 
                      bnode.next = null;
                  }else{
                      node.next = bnode;
                      bnode = node; 
                  }
              }
          }
          ListNode n1 = quickSortList(pnode);
          ListNode n2 = quickSortList(bnode);
          head.next = n2;
          if(n1!=null)getTail(n1).next = head;
          else return head;
          return n1;
  
      }
  
      public ListNode getTail(ListNode node){
          if(node==null||node.next==null) return node;
          return getTail(node.next);
      }
  } 
  ```

  





## 数学算法

- 线性筛素数, 将0-size的素数都筛选掉
- 当前数字是`n=p1^a * p2^b * p3^c`(p1<p2<p3且均为素数)，一次循环筛除*小于等于p1的素数乘以n得到的*数。比如p1之前有pi,pj和pk三个素数，则此次循环筛掉`pi*n,pj*n,pk*n`和`p1*n` ，实现见代码的**标注一**，`prime` 里的素数都是升序排列的，`break`时的`prime[j]` 就是这里的`p1`。 

```c++

#define SIZE 1000000
int prime()
{
    int check[SIZE] = {0}; 
    int prime[SIZE] = {0};
    int pos=0;
    int flag;
    for (int i = 2 ; i < SIZE ; i++)
    {
        if (!check[i]) 
            prime[pos++] = i;

        for (int j = 0 ; j < pos && i*prime[j] < SIZE ; j++)
        {
            check[i*prime[j]] = 1;
          
            if (i % prime[j] == 0)
                break;
        }
    }

    return 0;
} 
```





