# System Design

##1.Large String Process
[Work BreakII](https://leetcode.com/problems/word-break-ii/)

Q : Assuming length of string reach 1 trillion, how to design the system.

A : If I have n machines, I could divide the string into n parts and do it and merge the result.

Q : How to improve?

A : Set up a main server to cache the intermediate result, for example the word "whereiswhere", if machine 1 has processed the word "where", machine 1 can cache the result in main server 