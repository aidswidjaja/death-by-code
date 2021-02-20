<title>Combination sum | death_by_code()</title>

# Combination sum

So for my first deep dive into computer science I took a look at this question on the front of [dailycodingproblem.com](http://web.archive.org/web/20210116012845/https://www.dailycodingproblem.com/)

#### Prompt
<span>

There's a staircase with N steps, and you can climb 1 or 2 steps at a time. Given N, write a function that returns the number of unique ways you can climb the staircase. The order of the steps matters. 

For example, if N is 4, then there are 5 unique ways:

```
1, 1, 1, 1
2, 1, 1
1, 2, 1
1, 1, 2
2, 2
```

What if, instead of being able to climb 1 or 2 steps at a time, you could climb any number from a set of positive integers X? For example, if X = {1, 3, 5}, you could climb 1, 3, or 5 steps at a time. 

Generalize your function to take in X. 

</span>

#### Exploration
So after some quick research, this is a simple combination sum. Now mathematically it's pretty easy to calculate how **many** permutations there are, but of course, a computer wants to calculate **every single permutation**. 

Now, I am too dumb to figure out the maths to write this. I don't really even know where to start. Should I do this recursively or use iteration? How would I implement an algorithm to cycle all the possibilities. Would I kill the hypothetical memory of the machine? In fact, there are many different ways to solve the combination sum - [this site](http://rosettacode.org/wiki/Count_the_coins) contains this problem solved in like a ton of different languages. Gosh imagine doing this in asm...

So let's steal someone else's code instead. I'll use the example provided at [geeksforgeeks.org](https://www.geeksforgeeks.org/combinational-sum/)

They cite this algorithmic process:

> 1. Sort the array(non-decreasing).
> 2. First remove all the duplicates from array.
> 3. Then use recursion and backtracking to solve the problem.
>   - If at any time sub-problem sum == 0 then add that array to the result (vector of vectors).
>   - Else if sum is negative then ignore that sub-problem.
>   - Else insert the present array in that index to the current vector and call the function with sum = sum-ar[index] and index = index, then pop that element from  current index (backtrack) and call the function with sum = sum and index = index+1

Ok this will be fun to untangle. Let's work through this step by step.

```
1. Sort the array(non-decreasing).
```
Alright, simple, that's just sorting the input array (which in our combination sum prompt == the permitted steps) from smallest to biggest. In C++ this is represented through this method:

```
vector<vector<int>> combinationSum(vector<int> &ar,
                                int sum)
{
    // sort input array
    sort(ar.begin(), ar.end());

    // remove duplicates
    ar.erase(unique(ar.begin(), ar.end()), ar.end());

    vector<int> r;
    vector<vector<int>> res;
    findNumbers(ar, sum, res, r, 0);

    return res;
}
```