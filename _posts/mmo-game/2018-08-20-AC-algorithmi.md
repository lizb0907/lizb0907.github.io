---
layout: post
title: AC算法过滤游戏里敏感词汇实战
categories: mmo-game
description: AC算法过滤游戏里敏感词汇
keywords: AC算法，过滤，敏感词汇
---

AC算法过滤游戏里敏感词汇实战


**目录**

* TOC
{:toc}

## 屏蔽字校验--AC算法

算法理解参考文章：

http://www.hyuuhit.com/2018/06/12/Aho%E2%80%93Corasick-Algorithm/

http://www.hankcs.com/program/algorithm/aho-corasick-double-array-trie.html

### 1.处理敏感词汇表数据

![](/images/posts/mmo_game/2.png)

```java
@Override
public void init(List<IDictBase> l)
{
    super.init(l);

    HashSet<String> chatSet = new HashSet<>();
    HashSet<String> nameSet = new HashSet<>();

    TIntObjectIterator<DictSensitiveWord> iterator = map.iterator();
    while(iterator.hasNext())
    {
        iterator.advance();
        
        DictSensitiveWordData config = (DictSensitiveWordData) iterator.value();

        String word = config.getWord();
        
        if(word.contains("&"))
        {
            continue;
        }
        
        if(config.isChatCheck)
        {
            chatSet.add(word);
        }
        
        if(config.isNameCheck)
        {
            nameSet.add(word);
        }
    }

    String[] chatArr = new String[chatSet.size()];
    chatSet.toArray(chatArr);
    String[] nameArr = new String[nameSet.size()];
    nameSet.toArray(nameArr);

    ChatUtils.load(chatArr, nameArr);
}
```
分析：分别将聊天屏蔽字和命名屏蔽字字符串转为字符串数组，调用load（）方法初始化AC算法敏感词数据结构

### 2.AC算法过滤敏感词汇
```java
/**
 * @author lizhibiao
 * @date 2019/8/1 15:57
 */
public class DFATest
{
    public static final char SUBSTITUTE_CHAR = '*';
    private static final char[] IGNORE_CHARS = {	'　', ' ', '*', '-', '_', '+', '/', '.', '(', ')', '&', '%', '$', '#',
            '@', '!' };

    private static KeyWordsACFilter util = null;

    public static void main(String[] args)
    {
        init();
        boolean result = util.contain("shis");
        System.out.println(result);

    }

    private static void init()
    {
        String[] keys = {"he", "she", "his", "hers"};
        util = new KeyWordsACFilter(IGNORE_CHARS, SUBSTITUTE_CHAR);
        util.initialize(keys);
    }

}
```
分析:初始化构造"he", "she", "his", "hers"敏感词汇goto表和failure表

```java
/**
    * 构造goto表
    * @param keyWords
    * @return
    */
public boolean initialize(String[] keyWords) {
    clear();
    for (int s = 0; s < keyWords.length; s++) {
        String _keyword = keyWords[s];
        if (_keyword == null || (_keyword = _keyword.trim()).length() == 0) {
            continue;
        }
        char[] patternTextArray = _keyword.toCharArray();
        DFANode currentDFANode = dfaEntrance;
        for (int i = 0; i < patternTextArray.length; i++) {
            final char _c = patternTextArray[i];
            // 逐点加入DFA
            final Character _lc = toLowerCaseWithoutConfict(_c);
            DFANode _next = currentDFANode.dfaTransition.get(_lc);
            if (_next == null) {
                //如果为空就往当前节点的map里put操作
                _next = new DFANode();
                currentDFANode.dfaTransition.put(_lc, _next);
            }
            currentDFANode = _next;
        }

        //尾节点设置为终止状态
        if (currentDFANode != dfaEntrance) {
            currentDFANode.isTerminal = true;
        }
    }

    //构造失效节点
    buildFailNode();
    return true;
}
```
分析:构造goto表,并且借鉴了DFA算法的终止状态

```java
/**
* 构造失效节点： 一个节点的失效节点所代表的字符串是该节点所表示它的字符串的最大 部分前缀
*/
private final void buildFailNode() {
    // 以下构造失效节点
    List<DFANode> queues = new ArrayList<DFANode>();
    dfaEntrance.failNode = dfaEntrance;//
    //状态0是根节点，failure跳转为其自身。深度为1的状态节点failure跳转均为根节点。
    for (Iterator<DFANode> it = dfaEntrance.dfaTransition.values().iterator(); it.hasNext(); ) {
        DFANode node = it.next();
        node.level = 1;
        queues.add(node);
        node.failNode = dfaEntrance;// 失效节点指向状态机初始状态
    }
    DFANode curNode = null;
    DFANode failNode = null;
    while (!queues.isEmpty()) {
        curNode = queues.remove(0);// 该节点的失效节点已计算
        failNode = curNode.failNode;
        for (Iterator<Map.Entry<Character, DFANode>> it = curNode.dfaTransition.entrySet().iterator(); it
                .hasNext(); ) {
            Map.Entry<Character, DFANode> nextTrans = it.next();
            Character nextKey = nextTrans.getKey();
            DFANode nextNode = nextTrans.getValue();
            // 如果父节点的失效节点中有条相同的出边，那么失效节点就是父节点的失效节点
            while (failNode != dfaEntrance && !failNode.dfaTransition.containsKey(nextKey)) {
                failNode = failNode.failNode;
            }
            nextNode.failNode = failNode.dfaTransition.get(nextKey);
            if (nextNode.failNode == null) {
                nextNode.failNode = dfaEntrance;
            }
            nextNode.level = curNode.level + 1;
            queues.add(nextNode);// 计算下一层

        }

    }
}
```
分析:goto表结合failure表

```java
public boolean contain(final String inputMsg) 
{
    char[] input = inputMsg.toCharArray();
    DFANode currentDFANode = dfaEntrance;
    DFANode _next = null;
    for (int i = 0; i < input.length; i++) {
        final Character _lc = this.toLowerCaseWithoutConfict(input[i]);
        if (!isIgnore(_lc)) {
            _next = currentDFANode.dfaTransition.get(_lc);
            while (_next == null && currentDFANode != dfaEntrance) {
                currentDFANode = currentDFANode.failNode;
                _next = currentDFANode.dfaTransition.get(_lc);
            }
        }
        if (_next != null) {
            // 找到状态转移，可继续
            currentDFANode = _next;
        }
        // 看看当前状态可退出否
        if (currentDFANode.isTerminal) {
            // 可退出，记录，可以替换到这里
            return true;
        }
    }

    return false;
}
```
分析:检验是否包含敏感词,如果goto表中当前状态对于字符a没有合法跳转，则根据failure表转移状态。

