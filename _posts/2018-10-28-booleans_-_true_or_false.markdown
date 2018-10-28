---
layout: post
title:      "Booleans - True or False"
date:       2018-10-28 18:03:43 +0000
permalink:  booleans_-_true_or_false
---

It's Halloween time!  I told my husband that I would dress up as a boolean.  I would be a ghost (boo) and have in one hand the word true and the other hand the word false.  He is a programmer and this went directly over his head.  

Anyways, I'm going to be Salt Bae now.  

**WHAT IS A BOOLEAN?**

Booleans, however are not scary.  It's a data type for a conditional statement value that is either true or false.  Boolean expressions allow us to conditionally excute a branch of code.  By default the value is set to `false`.  It is named after George Boole, an English mathematician who helped establish modern symbolic logic - now called Boolean algebra.

![](https://www.tutorialspoint.com/scala/images/scala_decision_making.jpg)

**COMPARISON/RELATIONAL OPERATORS**

| Operator | Function | 
| -------- | -------- | 
| `==`   | Equals to    |
| `!=`     | Not equal to   | 
| `<`     | Less than   | 
| `>`     | Greater than   | 
| `<=`     | Less than or equal to   | 
| `>=`     | Greater than or equal to   | 


**BOOLEAN/LOGICAL OPERATORS**

Boolean operators are used to define the relationship between your search terms.  They can be used to narrow or broaden your search.  There are three main operators are AND ( `&&` ), OR ( `||`) and NOT ( `!`).  

The order of operation is read from left to right.  Both `&&` and `||` use [short circuit evaluation](https://en.wikipedia.org/wiki/Short-circuit_evaluation).  For `&&`, as soon as an expression evaulates as false, arguments following are not evaluated and the entire expression evaluates as false.  For `||`, as soon as an expression evaluates as true, arguments following are not evaluated and the entire expression evaulates as true.

**AND ( `&&` )**

Values on both sides of the symbol must be true.  Otherwise the expression is false.  

Examples:

```
9 == 9 && 8 == 8 TRUE
true && false FALSE
```

Short Circuit Example:
```
1 == 2 && reboot_computer
```

**OR ( `||`)** 
One of the values on either side of the symbol must be true.  Otherwise the expression is false. 

Examples:

```
true || false TRUE 
8 == 9 || 1 == 0  FALSE 
```

Short Circuit Example:
```
country = user_input || 'USA'
```

**NOT ( `!` )** 
Reverses the logic.  If the condition is true, it will now be false.   

Examples:

```
!!true =  TRUE 
!true = FALSE 
```
