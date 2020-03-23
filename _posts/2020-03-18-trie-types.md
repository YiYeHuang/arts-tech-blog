---
layout: post
title:  "The evolution of Trie and problem it solves"
date:   2020-03-18 15:59:53 -0700
categories: algorithm
tag: [leetcode, trie, heap]
---

## Algorithm
Trie Tree 的用处在开发中非常大，LeetCode中的题反而不多。但是每题都很经典，需要对Trie Node进行大量的变换。

### Trie Tree
Trie Tree结构很简单，由一个空Root构成，底下构成26个child node, 分别对应26个字母  
每个node又有26个child node。可以由application不同扩展成ASCII table个child node。

Operation:
- insert:
  - 把每个单词拆分，每一个字符构成一层
- search:
  - 搜索单个单词是否存在时候用DFS
  - 个别应用可用BFS，搜索每一层
- delete: 应用不多

### Basic Trie Node
```java
public class TrieNode {
	public char val;
	public boolean wordEndHere;
	public TrieNode[] children = new TrieNode[26];

	public TrieNode() { }

	public TrieNode(char c){
		TrieNode node = new TrieNode();
		node.val = c;
	}
}
```
最简单的Trie Node, 能确认一个单词是否存在，解决不需要使用存储的单词的问题

#### 解决问题1
```text
676. Implement Magic Dictionary

Implement a magic directory with buildDict, and search methods.

For the method buildDict, you'll be given a list of non-repetitive words to build a dictionary.

For the method search, 
you'll be given a word, and judge whether if you modify exactly one character 
into another character in this word, the modified word is in the dictionary you just built.

Example 1:
Input: buildDict(["hello", "leetcode"]), Output: Null
Input: search("hello"), Output: False
Input: search("hhllo"), Output: True
Input: search("hell"), Output: False
Input: search("leetcoded"), Output: False
Note:
You may assume that all the inputs are consist of lowercase letters a-z.
For contest purpose, the test data is rather small by now. 
You could think about highly efficient algorithm after the contest.
Please remember to RESET your class variables declared in class MagicDictionary, 
as static/class variables are persisted across multiple test cases. Please see here for more details.
```

```java
public class MagicDictionary {

    TrieNode root;

    public MagicDictionary() {
        root = new TrieNode();
    }

    public void buildDict(String[] dict) {
        for(String s : dict){
            TrieNode curr = root;
            for(char c : s.toCharArray()){
                if(curr.children[c - 'a'] == null){
                    curr.children[c - 'a'] = new TrieNode(c);
                }
                curr = curr.children[c - 'a'];
            }
            curr.wordEndHere = true;
        }
    }

    public boolean search(String word) {
        // word, current Index, # of change, current node
        return dfs(word, 0, 0, root);
    }

    private boolean dfs(String word, int cur, int numOfChange, TrieNode node) {
        if (cur == word.length()) {
            return numOfChange == 1 && node.wordEndHere;
        }

        if (numOfChange > 1) {
            return false;
        }

        char c = word.charAt(cur);

        for (int i = 0; i < 26; i++) {
            if (node.children[i] != null &&
                    dfs(word, cur+1,
                        word.charAt(cur) - 'a' == i ? numOfChange : numOfChange+1,
                        node.children[i])) {
                return true;
            }
        }
        return false;
    }
}
```

去掉特殊条件就是一个简单的找单词的问题。  
利用递归DFS记录更改字母的次数。

#### 解决问题2
```text
 211. Add and Search Word - Data structure design

 Design a data structure that supports the following two operations:

 void addWord(word)
 bool search(word)
 search(word) can search a literal word or a regular expression string containing 
 only letters a-z or .. A . means it can represent any one letter.

 Example:

 addWord("bad")
 addWord("dad")
 addWord("mad")
 search("pad") -> false
 search("bad") -> true
 search(".ad") -> true
 search("b..") -> true
 ```

 ```java
 public class AddandSearchWord {

	private TrieNode root;

	/** Initialize your data structure here. */
	public AddandSearchWord() {
		root = new TrieNode();
		root.val = ' ';
	}

	/** Adds a word into the data structure. */
	public void addWord(String word) {
		TrieNode walker = root;
		for (int i = 0; i < word.length(); i++) {
			char insert = word.charAt(i);
			if (walker.children[insert - 'a'] == null) {
				walker.children[insert - 'a'] = new TrieNode(insert);
			}
			walker = walker.children[insert - 'a'];
		}

		walker.wordEndHere = true;
	}

	/** Returns if the word is in the data structure. A word could contain the dot character '.' to represent any one letter. */
	public boolean search(String word) {
		TrieNode walker = root;
		return findMatch(word, 0, walker);
	}

	public boolean findMatch(String word, int level, TrieNode currRoot) {
		if (word.length() == level) {
			if (currRoot.wordEndHere) {
				return true;
			} else {
				return false;
			}
		}

		char search = word.charAt(level);

		if ( search != '.' ) {
			if (currRoot.children[search-'a'] == null) {
				// mismatch
				return false;
			} else {
				// find
				return findMatch(word,level+1,currRoot.children[search-'a']);
			}
		} else {
			// KEY point: find . skip a level
			for (int i = 0; i < 26; i++) {
				if (currRoot.children[i] != null) {
					if (findMatch(word,level+1,currRoot.children[i])) {
						return true;
					}
				}
			}
		}

		// never get here
		return false;
	}
}
```
add没有任何变化，搜索更加复杂，需要handle 出现`.`的时候的skip问题，最好的办法还是利用递归DFS，分开两个case处理正常的搜索和skip的case。

#### 解决问题3
```text
648. Replace Words

In English, we have a concept called root, which can be followed by some other words to 
form another longer word - let's call this word successor. 
For example, the root an, followed by other, which can form another word another.

Now, given a dictionary consisting of many roots and a sentence. 
You need to replace all the successor in the sentence with the root forming it. 
If a successor has many roots can form it, replace it with the root with the shortest length.

You need to output the sentence after the replacement.

Example 1:

Input: dict = ["cat", "bat", "rat"]
sentence = "the cattle was rattled by the battery"
Output: "the cat was rat by the bat"
 

Note:

The input will only have lower-case letters.
1 <= dict words number <= 1000
1 <= sentence words number <= 1000
1 <= root length <= 100
1 <= sentence words length <= 1000
```
缩写风暴？？
```java
public class ReplaceWords {

    /**
     * pretty much the question tells you that you need to build a trie
     */
    public static String replaceWords(List<String> dict, String sentence) {

        TrieNode root = new TrieNode(' ');
        String[] list = sentence.split(" ");
        for (String word : dict) {
            insert(word, root);
        }

        StringBuilder stringBuilder = new StringBuilder();
        for (String token : list) {
            stringBuilder.append(replace(token, root));
            stringBuilder.append(" ");
        }
        return stringBuilder.substring(0, stringBuilder.length()-1);
    }

    public static String replace(String token, TrieNode root) {
        TrieNode walker = root;
        StringBuilder result = new StringBuilder();

        for (char c:token.toCharArray()) {
            result.append(c);
            if (walker.children[c - 'a'] != null) {
                // append the result until word end here
                if (walker.children[c - 'a'].wordEndHere) {
                    return result.toString();
                }
                walker = walker.children[c - 'a'];
            } else {
                // mis match, return the original words
                return token;
            }
        }
        return token;
    }

    public static void insert(String word, TrieNode root) {
        TrieNode node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNode();
            }
            node = node.children[idx];
        }
        node.wordEndHere = true;
    }
}
```
- build trie tree with dictionary
- 搜索每个词语，一边搜索一边build新词。
    - 找到word end here就replace
    - 没有找到就return原来的词语


### Basic Trie Node with extension
Node的children变成ASCII的全局，用来处理有大小写，或者别的字符的case

```java
public class TrieNodeFlex {
    public boolean wordEndHere;
    public Map<Character, TrieNodeFlex> children;

    public TrieNodeFlex() {
        children = new HashMap<>();
    }
}
```

#### 解决问题
```text
1023. Camelcase Matching

A query word matches a given pattern if we can insert lowercase 
letters to the pattern word so that it equals the query.
(We may insert each character at any position, and may insert 0 characters.)

Given a list of queries, and a pattern, return an answer list of booleans, 
where answer[i] is true if and only if queries[i] matches the pattern.

Example 1:

Input: queries = ["FooBar","FooBarTest","FootBall","FrameBuffer","ForceFeedBack"], pattern = "FB"
Output: [true,false,true,true,false]
Explanation:
"FooBar" can be generated like this "F" + "oo" + "B" + "ar".
"FootBall" can be generated like this "F" + "oot" + "B" + "all".
"FrameBuffer" can be generated like this "F" + "rame" + "B" + "uffer".
Example 2:

Input: queries = ["FooBar","FooBarTest","FootBall","FrameBuffer","ForceFeedBack"], pattern = "FoBa"
Output: [true,false,true,false,false]
Explanation:
"FooBar" can be generated like this "Fo" + "o" + "Ba" + "r".
"FootBall" can be generated like this "Fo" + "ot" + "Ba" + "ll".
Example 3:

Input: queries = ["FooBar","FooBarTest","FootBall","FrameBuffer","ForceFeedBack"], pattern = "FoBaT"
Output: [false,true,false,false,false]
Explanation:
"FooBarTest" can be generated like this "Fo" + "o" + "Ba" + "r" + "T" + "est".


Note:

1 <= queries.length <= 100
1 <= queries[i].length <= 100
1 <= pattern.length <= 100
All strings consists only of lower and upper case English letters.
```
审题完毕立刻就build了words的Trie Tree, 很美观很直观
![trie3](/public/img/trie3.png)
然而搜索写起来怎么都卡手，如果是比赛就跪了  
题目的size条件不是很苛刻
```
1 <= queries.length <= 100
1 <= queries[i].length <= 100
1 <= pattern.length <= 100
```
但是query的大小还是比Pattern大太多，所以改为build pattern trie tree

```java
public class CamelcaseMatching {

    public static List<Boolean> camelMatch(String[] queries, String pattern) {
        TrieNodeFlex root = new TrieNodeFlex();
        List<Boolean> res = new ArrayList<>();
        TrieNodeFlex node = root;

        // build Trie with pattern, since a is > A, creating child size 128
        for (int i = 0; i < pattern.length(); i++) {
            TrieNodeFlex temp = new TrieNodeFlex();
            node.children.put(pattern.charAt(i), temp);
            node = temp;
        }
        node.wordEndHere = true;

        for(String query : queries) {
            TrieNodeFlex walker = root;
            boolean flag = true;

            for (int i = 0; i < query.length(); i++) {
                int charCode = (int) query.charAt(i);
                // not matching
                if(walker.children.get(query.charAt(i)) == null && charCode >= 65 && charCode <= 90) {
                    flag = false;
                    break;
                }
                if (walker.children.get(query.charAt(i)) != null) {
                    walker = walker.children.get(query.charAt(i));
                }
                // lower case + not match will just skip
            }

            if(flag && walker.wordEndHere)
                res.add(true);
            else
                res.add(false);
        }

        return res;
    }
}
```
- Trie tree需要build pattern而不是words
- 遇到UpperCase比较，不一样就立刻停止
- 遇到lowerCase就移动到下一个
    - 不match就自动跳过
- 如果lower case不match, trie tree就不会继续往下搜
- for loop外通过 isWordEndHere检查是否Pattern检查完


### Trie Node with Words
每个Node多存了一个Words如果worEndHere, 方便在搜寻完毕以后立刻调用结果
```java
public class TrieNodeWithWord {
	public char val;
	public boolean wordEndHere;
	public String word;
	public TrieNodeWithWord[] children = new TrieNodeWithWord[26];

	public TrieNodeWithWord() { }

	public TrieNodeWithWord(char c){
		TrieNodeWithWord node = new TrieNodeWithWord();
		node.val = c;
	}
}
```

#### 解决问题1
```text
720. Longest Word in Dictionary

Given a list of strings words representing an English Dictionary, 
find the longest word in words that can be built one character at a time by other words in words. 
If there is more than one possible answer, return the longest word with the smallest lexicographical order.

If there is no answer, return the empty string.
Example 1:
Input: 
words = ["w","wo","wor","worl", "world"]
Output: "world"
Explanation: 
The word "world" can be built one character at a time by "w", "wo", "wor", and "worl".
Example 2:
Input: 
words = ["a", "banana", "app", "appl", "ap", "apply", "apple"]
Output: "apple"
Explanation: 
Both "apply" and "apple" can be built from other words in the dictionary. 
However, "apple" is lexicographically smaller than "apply".
Note:

All the strings in the input will only contain lowercase letters.
The length of words will be in the range [1, 1000].
The length of words[i] will be in the range [1, 30].
```
![trie1](/public/img/trie1.png)
比较刁钻的问题，需要寻找trie tree的最长一枝，每一个node都是wordEndHere

```java
public class LongestWordInDictionary {

    public String longestWordHashTable(String[] words) {
        Arrays.sort(words);
        Set<String> search = new HashSet<String>();
        String result = "";

        for (String word : words) {
            if (word.length() == 1 || search.contains(word.substring(0, word.length() - 1))) {
                result = word.length() > result.length()? word:result;
                search.add(word);
            }
        }

        return result;
    }

    /**
     * trie solution
     * Build trie, BFS to find the longest path
     */
    private TrieNodeWithWord root;

    public String longestWordTrie(String[] words) {
        TrieNodeWithWord trie = new TrieNodeWithWord();
        for (String word : words) {
            insert(word);
        }

        return findLongestWord();
    }

    public void insert(String word) {
        TrieNodeWithWord node = root;
        for (int i = 0; i < word.length(); i++) {
            int idx = word.charAt(i) - 'a';
            if (node.children[idx] == null) {
                node.children[idx] = new TrieNodeWithWord();
            }
            node = node.children[idx];
        }
        node.wordEndHere = true;
        node.word = word;
    }

    public String findLongestWord() {
        String result = null;
        Queue<TrieNodeWithWord> queue = new LinkedList<>();
        queue.offer(root);
        while (!queue.isEmpty()) {
            int size = queue.size();
            for (int i = 0; i < size; i++) {
                TrieNodeWithWord node = queue.poll();
                for (int j = 25; j >= 0; j--) {
                    if (node.children[j] != null && node.children[j].wordEndHere) {
                        result = node.children[j].word;
                        queue.offer(node.children[j]);
                    }
                }
            }
        }
        return result;
    }
}
```
- build每个单词的bfs
- 搜寻由DFS改为BFS，因为要得知每一层是否都有单词
    - 找到每个有wordEndHere的child node继续进queue
    - 直到queue empty
- 此时找到的结果就是最长的“组合”单词

#### 解决问题2
```
692. Top K Frequent Words

Given a non-empty list of words, return the k most frequent elements.

Your answer should be sorted by frequency from highest to lowest. If two words have the same frequency, 
then the word with the lower alphabetical order comes first.

Example 1:
Input: ["i", "love", "leetcode", "i", "love", "coding"], k = 2
Output: ["i", "love"]
Explanation: "i" and "love" are the two most frequent words.
    Note that "i" comes before "love" due to a lower alphabetical order.
Example 2:
Input: ["the", "day", "is", "sunny", "the", "the", "the", "sunny", "is", "is"], k = 4
Output: ["the", "is", "sunny", "day"]
Explanation: "the", "is", "sunny" and "day" are the four most frequent words,
    with the number of occurrence being 4, 3, 2 and 1 respectively.
Note:
You may assume k is always valid, 1 ≤ k ≤ number of unique elements.
Input words contain only lowercase letters.
```
Top K问题都是各路能玩出花的题目，这题严格来说Trie不是最好的解决途径。
Trie在这里的作用其实就是做一个sorting，以达成`alphabetical order`的目的
![trie4](/public/img/trie4.png)

```java
public class TopKFrequentWords {
    /**
     *  define a trie within each bucket to store all the words with the same frequency.
     *  With trie, it ensures that the lower alphabetical word will be met first,
     *  saving the trouble to sort the words within the bucket.
     */
    public List<String> topKFrequentBucketAndTrie(String[] words, int k) {
        if(words.length == 0) return new ArrayList<>();
        // Mapping word to frequency
        Map<String, Integer> map = new HashMap<>();
        for(String n: words){
            map.put(n, map.getOrDefault(n,0)+1);
        }

        // build the buckets and trie
        TrieNodeWithWord[] bucketRoot = new TrieNodeWithWord[words.length + 1];
        for(String word : map.keySet()) {
            int freq = map.get(word);
            if(bucketRoot[freq] == null) {
                bucketRoot[freq] = new TrieNodeWithWord();
            }
            addWord(bucketRoot[freq], word);
        }

        // get k frequent work
        List<String> result = new LinkedList<>();
        // get from the end of the list
        for(int f = bucketRoot.length - 1; f >= 1 && result.size() < k; f--) {
            if(bucketRoot[f] == null) {
                continue;
            }
            getwords(bucketRoot[f], result, k);
        }

        return result;
    }

    private void getwords(TrieNodeWithWord node, List<String> list, int k) {
        if(node == null) return;
        if(node.word != null) {
            list.add(node.word);
        }
        if(list.size() == k) return;
        for(int i = 0; i < 26; i++) {
            if(node.children[i] != null) {
                getwords(node.children[i], list, k);
            }
        }
    }

    private boolean addWord(TrieNodeWithWord root, String word) {
        TrieNodeWithWord curr = root;
        for(char c : word.toCharArray()) {
            if(curr.children[c - 'a'] == null) {
                curr.children[c - 'a'] = new TrieNodeWithWord();
            }
            curr = curr.children[c - 'a'];
        }
        curr.word = word;
        return true;
    }
}
```
- 解题核心思想还是bucket sort
- 每一个frequency的bucket都是一个Trie tree的Root
- 倒叙从Frequency最大的开始倒叙遍历直到得到K个单词
