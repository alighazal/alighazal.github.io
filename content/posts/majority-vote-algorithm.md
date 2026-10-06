---
title: "an elegant algorithm that made me smile"
draft: false
build:
  render: always
  list: always
  publishResources: true
---


In an attempt to refresh my problem solving skills, I found a very simple problem that led me to discover a beautiful, elegant algorithm that put a smile on my face. 

The problem is as follows. Given a sequence of N elements, we would like to find the majority item which appeared in this sequence more than floor(N/2). And, it is guaranteed that a majority item exists in this sequence. 

The problem is very straight forward, and in coding sites, this is considered an 'easy' problem. On top of my mind, I thought of different solutions. 

First, we can have a collection (a map), and we could count the number of items there. Then, we can iterate over all those items and return the one that appeared in more than floor(N/2). This would have a linear time complexity O(N) and a linear space complexity O(N). 

Or, another solution would be sorting the entire array, and then returning the item right in the middle. And the intuition behind this is that the majority item is contained in a sequence strictly larger than floor(N/2) and thus the middle element must belong there. Just for fun, this could be proven by contradiction. Assuming that the item at floor(N/2) is not the majority item. This means that the majority item would exist either to the right of to the left of it, and either way this means that it appeared less then floor(N/2). And this is a contradiction to our original assumption that the majority items appears more than floor(N/2). The time complexity of this solution would be the complexity of the sorting algorithm used, and the space complexity would be constant O(1).  

Another silly, but viable solution would be asking the AI to find the majority item in a sequence. Although, I cannot comment on the space and time complexity of this solution, there is a gives you the possibility to claim that you have an AI products. :D


Joking aside, there was a hint after I answered the question if this problem could be solved in O(N) time complexity and O(1) space complexity. After some digging, I found one of the most elegant algorithms that I have seen in a while.

The Boyer–Moore majority vote algorithm ([wikipedia link](https://en.wikipedia.org/wiki/Boyer%E2%80%93Moore_majority_vote_algorithm)).

So, first of all, why do we need such an algorithm in the first place? Well, it is efficient. And, it is heavily used in as a streaming algorithm. for the two (or three) previous solutions, each time we needed to calculate the majority item, we had to go over the entire array. With the Boyer–Moore majority vote algorithm, we don't

Ok, I will spare you the drama and my enthusiasm, Here is the code in c++:

```cpp

int find_majority(vector<int>& arr){
    int majority_item = arr[0], count = 1;
    for(int i = 1; i < arr.size(); i++){
        if (count == 0){
            majority_item = arr[i];
            count = 1;
        }else if (majority_item == arr[i]){
            count++;
        }else{
            count--;
        }
    }
    return majority_item;
}

```

The intuition of this algorithm, is that we are going over the sequence, and it is a cancellation game. And, since it is guaranteed that the majority item appears more than floor(N/2), it will survive the cancellation game. A formal proof for this algorithm exists in the mentioned wikipedia page and [here](https://math.stackexchange.com/questions/4997602/the-boyer-moore-majority-vote-algorithm-proof-of-correctness) as well. 

As the saying goes, if you don't already see how elegant this algorithm is, I can't make you see it. I hope it did put a smile on your face like it did with me.


