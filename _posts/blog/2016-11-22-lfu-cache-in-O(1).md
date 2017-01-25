---
layout: post
title: LFU cache in O(1)
category: blog
comments: true
published: true 
excerpt: Step-by-step implementation of least frequently used cache in O(1) time
tags: 
  - development
  - java
---

I recently came across [this question in leetcode](https://leetcode.com/problems/lfu-cache/). 
It looks naive at first glance but is tricky to do in time-complexity O(1). 

Let us try to implement an acceptable solution which performs insert/delete in O(1) and access in O(n). 
Further we will implement [this paper](http://dhruvbird.com/lfu.pdf) which performs even access in O(1) using a beautiful yet simple data-structure.

## Problem statement

Design and implement a data structure for Least Frequently Used (LFU) cache. It should support the following operations: get and set.

get(key) - Get the value (will always be positive) of the key if the key exists in the cache, otherwise return -1.
set(key, value) - Set or insert the value if the key is not already present. When the cache reaches its capacity, it should invalidate the least frequently used item before inserting a new item. For the purpose of this problem, when there is a tie (i.e., two or more keys that have the same frequency), the least recently used key would be evicted.

Could you do both operations in O(1) time complexity?


## Solution

### Step 1: Store values and access frequencies 

We need 2 things to start with

1. Map to store key-value pairs
2. Map to store counts/frequency of access

Now insert and access operations are O(1) i.e. they perform in constant time.

<figure style="max-width: 600px; margin-left: auto; margin-right: auto">
    <a href="{{ site.url }}/images/blog/lfu/lfu-cache-1.png"><img src="{{ site.url }}/images/blog/lfu/lfu-cache-1.png"></a>
</figure>

{% highlight java %}

public class LFUCache {

    private Map<Integer, Integer> values = new HashMap<>();
    private Map<Integer, Integer> frequencies = new HashMap<>();
    private final int MAX;

    public LFUCache(int capacity) {
        MAX = capacity;
    }

    public int get(int key) {
        if (!values.containsKey(key)) {
            return -1;
        }
        frequencies.put(key, frequencies.get(key) + 1); 
        return values.get(key);
    }

    public void set(int key, int value) {
        if (!values.containsKey(key)) { 
            // TODO: eviction code
            values.put(key, value);
            frequencies.put(key, 1); // set initial access count as 1
        }
    }
}

{% endhighlight %}

### Step 2: Implement Evict method

How do we implement evict method? 
When size of map reaches max capacity, we need to find item with lowest frequency count. 
There are 2 problems:

1. We have to iterate through all values of frequencies map, find lowest count and remove corresponding key from both maps. 
This will take O(n) time.
2. Also, what if there are multiple keys with same frequency count? How do we find least recently used? 
That's not possible because HashMap does not store the order of insertion. 
 
To solve both of above problems we need to add one more data structure: 
Sorted map with frequency as map-keys and 'list of item-keys with same frequency' as map-values. 

<figure style="max-width: 600px; margin-left: auto; margin-right: auto">
    <a href="{{ site.url }}/images/blog/lfu/lfu-cache-3.png"><img src="{{ site.url }}/images/blog/lfu/lfu-cache-3.png"></a>
</figure>


**Awesome! Problem solved:**

1. We can add new item can to the end of the list with frequency 1.  
2. We can find the list with lowest frequency in O(1), since map is sorted by frequencies.
3. We can delete the first item of the list (of lowest frequency) since that will be least recently used. Also O(1).

Thus, both insert and delete are now O(1) i.e. constant time operations. 

{% highlight java %}

public class LFUCache {

    private Map<Integer, Integer> values = new HashMap<>();
    private Map<Integer, Integer> counts = new HashMap<>();
    private TreeMap<Integer, List<Integer>> frequencies = new TreeMap<>();
    private final int MAX_CAPACITY;

    public LFUCache(int capacity) {
        MAX_CAPACITY = capacity;
    }

    public int get(int key) {
        if (!values.containsKey(key)) {
            return -1;
        }

        // Move item from one frequency list to next. Not O(1) due to list iteration.
        int frequency = counts.get(key);
        frequencies.get(frequency).remove(new Integer(key));
        if (frequencies.get(frequency).size() == 0) {
            frequencies.remove(frequency);  // remove from map if list is empty
        }
        frequencies.computeIfAbsent(frequency + 1, k -> new LinkedList<>()).add(key);

        counts.put(key, frequency + 1);
        return values.get(key);
    }

    public void set(int key, int value) {
        if (!values.containsKey(key)) {

            if (values.size() == MAX_CAPACITY) {
                // first item from 'list of smallest frequency'
                int lowestCount = frequencies.firstKey();
                int keyToDelete = frequencies.get(lowestCount).remove(0);
                if (frequencies.get(lowestCount).size() == 0) {
                    frequencies.remove(lowestCount); // remove from map if list is empty
                }
                values.remove(keyToDelete);
                counts.remove(keyToDelete);
            }

            values.put(key, value);
            counts.put(key, 1);
            frequencies.computeIfAbsent(1, k -> new LinkedList<>()).add(key); // starting frequency = 1
        }
    }
}
{% endhighlight %}

### Step 3: Store Item's position


**Problem** 

While solving our delete problem, we accidentally increased our access operation time to O(n). How?
Note that all of item-keys sharing same frequency are in a list. Now if one of these items is accessed, 
how do we move it to list of next frequency? We will have to iterate through the list first to find the item,
which in worst-case will take O(n) operations. 

To solve the problem, we somehow need to jump directly to that item in the list without iteration. 
If we can do that, it will be easier to delete the item and add it to end of next frequency list. 
 
Unfortunately, this is not possible using our in-built data structures. 
We need to create a new one (mentioned in the [paper](http://dhruvbird.com/lfu.pdf)). 

<figure>
    <a href="{{ site.url }}/images/blog/lfu/lfu-cache-4.png"><img src="{{ site.url }}/images/blog/lfu/lfu-cache-4.png"></a>
</figure>


**Solution**

We need to store each item's position. 

1. We will create a simple class which stores item's key, value and its position in the list.
2. We will convert the linked list to 

<figure>
    <a href="{{ site.url }}/images/blog/lfu/lfu-cache-5.png"><img src="{{ site.url }}/images/blog/lfu/lfu-cache-5.png"></a>
</figure>

{% highlight java %}

public class LFUCache {

    private Map<Integer, Node> values = new HashMap<>();
    private Map<Integer, Integer> counts = new HashMap<>();
    private TreeMap<Integer, DoubleLinkedList> frequencies = new TreeMap<>();
    private final int MAX_CAPACITY;

    public LFUCache(int capacity) {
        MAX_CAPACITY = capacity;
    }

    public int get(int key) {
        if (!values.containsKey(key)) {
            return -1;
        }

        Node node = values.get(key);

        // Move item from one frequency list to next. O(1) this time.
        int frequency = counts.get(key);
        frequencies.get(frequency).remove(node);
        removeIfListEmpty(frequency);
        frequencies.computeIfAbsent(frequency + 1, k -> new DoubleLinkedList()).add(node);

        counts.put(key, frequency + 1);
        return values.get(key).value;
    }

    public void set(int key, int value) {
        if (!values.containsKey(key)) {

            Node node = new Node(key, value);

            if (values.size() == MAX_CAPACITY) {

                int lowestCount = frequencies.firstKey();   // smallest frequency
                Node nodeTodelete = frequencies.get(lowestCount).head(); // first item (LRU)
                frequencies.get(lowestCount).remove(nodeTodelete);

                int keyToDelete = nodeTodelete.key();
                removeIfListEmpty(lowestCount);
                values.remove(keyToDelete);
                counts.remove(keyToDelete);
            }

            values.put(key, node);
            counts.put(key, 1);
            frequencies.computeIfAbsent(1, k -> new DoubleLinkedList()).add(node); // starting frequency = 1
        }
    }

    private void removeIfListEmpty(int frequency) {
        if (frequencies.get(frequency).size() == 0) {
            frequencies.remove(frequency);  // remove from map if list is empty
        }
    }

    private class Node {
        private int key;
        private int value;
        private Node next;
        private Node prev;

        public Node(int key, int value) {
            this.key = key;
            this.value = value;
        }

        public int key() {
            return key;
        }

        public int value() {
            return value;
        }
    }

    private class DoubleLinkedList {
        private int n;
        private Node head;
        private Node tail;

        public void add(Node node) {
            if (head == null) {
                head = node;
            } else {
                tail.next = node;
                node.prev = tail;
            }
            tail = node;
            n++;
        }

        public void remove(Node node) {

            if (node.next == null) tail = node.prev;
            else node.next.prev = node.prev;

            if (head.key == node.key) head = node.next;
            else node.prev.next = node.next;

            n--;
        }

        public Node head() {
            return head;
        }

        public int size() {
            return n;
        }
    }
}

{% endhighlight %}


### Step 4: Remove the counts HashMap

Note that we need the intermediate map called counts to jump to the appropriate list. 
We can go one step further (code not written) to remove this extra data structure.

1. Convert frequencies HashMap keys into a doubly linked list
2. Add variable reference to each item, which points to corresponding frequency
3. So instead of counts hashmap, we can go get frequency node directly from the item itself.

This is precisely the algorithm implemented in [this paper](http://dhruvbird.com/lfu.pdf)

<figure>
    <a href="{{ site.url }}/images/blog/lfu/lfu-cache-7.png"><img src="{{ site.url }}/images/blog/lfu/lfu-cache-7.png"></a>
</figure>
*[Image Source](http://www.laurentluce.com/posts/least-frequently-used-cache-eviction-scheme-with-complexity-o1-in-python/)*


## Conclusion

Its fascinating how simple tweaks and combinations of data-structures can lead to more efficient solutions.

Hit me up in the comments if you have any doubts or critiques. Thanks.

