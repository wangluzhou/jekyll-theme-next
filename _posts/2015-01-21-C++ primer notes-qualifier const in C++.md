---
title: C++ primer notes-qualifier const in C++
date: 2015-01-21 20:25:37
categories:
- 计算机
tags:
- C++
---
<img src="http://7bv8rz.com1.z0.glb.clouddn.com/20150121-qualifier-const-in-cpp/const.png" class="img-topic" />
I think the usage of const is complex. I spent a lot of time in it(Someone may think it is unnecessary) but I am still not sure of everything. Something in the notes may be wrong. If you find something wrong, please point it out.
# General usage of const

```C++
const int bufSize = 512;
```

bufSize is a constant so that any attemp to assign to bufSize ia an error.
To avoid multiple definitions of the same variable, const variables are defined as local to the file.
We must initialize the const value in its declaration.
<!--more -->
# Const Pointer and pointer to Const
Of course, they are different!
("Reference to Const" and "Const Reference" are the same!)
Different declaration

```C++
	int *const p;//We can not change the value the pointer itself.
	const int *const p;//We can not change the value of the underlying object through p
```
Const Pointer is a pointer whose value(i.e. the address it holds) may not be changed, and it must be initialized!
Pointer to Const is a pointer which can not be used to change the value the underlying object. However it can point to const or non-const object.
Pay attention that we can only store the address of a const object in a pointer to const.
By the way, we can change the value of underlying object by const pointer.


Pay attention that
Until now, only reference to const can be initilized form any expression that can be converted to the type of the reference.
For example:

```C++
	const int &ri1 = 42;
	const int &ri2 = i * 2;
	//int &ri3 =42; ERROR!
```
Same as pointer to const, reference to const can refer to both const or non-const object.
You may understand it like this:
**Mr. Pointer.To.Const and Mr. Reference.To.Const are two guys who think they point or refer to a const object, so they reject to change the value of underlying object. However, in fact, the object they point to or refer to may not be a const one.**


# A guy with two consts
Let's see a freaking guy:
```C++
const double *const pip;
```
A guy with two consts! What the hell is it? As Stanley said, we can read the declaration from right to left. First we met const so that Mr. pip himself is a const object. Secondly, we meet *, so Mr. pip is a pointer. Hello pointer! Then we meet double which means that Mr. pip point to a double. Finally, we meet const again! What does it mean? Does it say that the underlying object is a const? Nononononononono! Remember what I told you in the 4th point? It only tells us that poor Mr. pip himself thinks that he will always point to a const object, so he rejects to change the value through himself. As to the truth, who knows and who cares?

# What will happen if we assgin a different type object to the reference to const?
For example:
```C++
	double dval = 3.14;
	const int &ri = dval;
```
The compiler will transfer the code into something like:
```C++
	const int temp = dval;//a temp object is an unnamed object created by the compiler
	const int &ri = temp;
```
You see!? The reference never refer to dval anymore! Think about the situation of ri. he can not refer to dval and he can not change the value he refers. It seems that the object he refers is a mysterious thing! He is the only one who knows the address of the temp object, but he can not touch it and he can not refer to anything else either. In conclusion, once he is defined like this, he becomes a literal. Congratulations!

#low-level and top-level const#
Only reference and pointer have low-level const.
Pointers and other objects have top-level const.
Top level can be ingored in the assignment of pointers and objects, however it is not suitable for reference.
As to reference, just remember two rules:
(1) two types must match for plain reference
(2) two types can be different for const reference
Unlike C++ primer, I have my own method to comprehensive it,first I just name the rule "top-->ignored, low-->same" as **top-low rule**

(1)  "top-low rule" is only suitable for the assignment between the objects in the same type, i.e. pointer and pointer or variables and varibales.
(2) For the assginment between different types, such as build-in type and pointer, build-in type and reference, we just use their own rules which is introduced before.
Actually I think only using "top-low" to generalize the discipline is confusing!
I think everything can be implied by rules introduced before. The only thing well need to remember is that the assigment to a non-low-const pointer from a low-const pointer is illegal. As I have said before, low-const pointer think he refers to a const object(The truth may be not). At same time, the address of const object can only be stored by low-const pointer. So that it will be risky to assign low-const poniter to non-low-const pointer! To make it more easier, the compiler just forbid this assigment. Then the world recovers to be harmonious!

# About the initializer
Pointers can be initialized from reference, so can reference.
