---
layout: post
title: "Parenthesis Confusion in C++"
date: 2011-01-15T00:00:00-05:00
---

I ran into this problem at least once, and I never actually thought about why it worked with or without the parenthesis, and thought that I was just doing something else wrong. But it was only so easy for me because I was doing a relatively small project, and the changes usually didn't affect any other files. I never really thought why or how it works. But if you're developing large scale applications or just portions of it, you need to make sure that you use it properly, or it might be a pain to debug it later. For example, as I will explain later, forgetting to add a parenthesis will keep POD class values from being properly initialized in some cases. If you don't know the properties of POD class initialization, you could be scratching your head when a comparison fails or the compiler throws you a run-time error! If this doesn't make sense yet, I recommend that you continue reading.

First thing I want to clear up is this post deals with the dilemma when you create objects using the <a href="http://en.wikipedia.org/wiki/New_%28C%2B%2B%29">new operator</a>. If you don't, then there really is no debate. Consider the following example:

<pre lang="cpp-qt" lineno="1">
struct Stan
{
     int man;
};

//now to test it out

Stan s1;
Stan s2();

</pre>

Here, the second initialization is obviously wrong, because the C++ compiler will interpret it as a function definition with no parameters that returns a Stan object. Glad that's over with aren't you? Now let's get down to the exciting stuff.

So does it matter if you add a parenthesis at the end of a <strong>dynamically generated class</strong>? It should definitely matter right? Let's give a test run shall we?

<pre lang="cpp-qt" lineno="1">
//using the Stan class from the previous example

Stan *s1 = new Stan;
Stan *s2 = new Stan();

</pre>

So what's going on here! s1 and s2 can't both be correct right? But once you test it out, it might appear that it doesn't matter at all. In Visual Studio 2008, the compiler doesn't throw an error or warning at either of these two declarations. But there are some important differences between these two initializations. I will review the differences in accordance with the c++03 revision. For the run down with the c++98 revision, see Philip Taylor's comment <a href="http://blogs.msdn.com/oldnewthing/archive/2006/12/14/1285437.aspx#1296360">here on a msdn blog</a>. 

First let me introduce some terminology. POD means 'Plain Old Data'. In C++, objects are either POD or non-POD. A POD class can be described as a simple class with little logic. More specifically, to be a POD class, it has to have data members of one of the following data-types: a built-in type(ie int, double), pointer, union, struct, array, or a class with a trivial constructor. The constructor part basically means that if you did not create a default constructor for your class, the C++ compiler will generate the generic constructor for you. A trivial constructor is a constructor that the compiler generates for you. In addition, a POD class cannot have other data types that are non-POD(for example, a string data type is not a POD class because it has a defined constructor). There are many things that would make a class non-POD even if it contains any of the members mentioned above:

<ol>
	<li>User defined constructor</li>
	<li>User defined copy constructor</li>
	<li>User defined assignment operator</li>
	<li>User defined destructor</li>
	<li>Non-POD data types such as a string or a non-POD class</li>
</ol>

So now let's example the differences between these two initializations. 

<pre lang="cpp-qt" lineno="1">
//using the Stan class from the previous example

Stan *s1 = new Stan;

</pre>

In this case, s1 is default initialized, which means that all POD data-types will be left uninitialized. In this case, any int will be uninitialized.  

<pre lang="cpp-qt" lineno="1">
//using the Stan class from the previous example

Stan *s2 = new Stan();

</pre>

Here s2 will be value initialized, which means that all data members will be initialized. Here, int will be zero-initialized to 0. Looking at the differences between these two initializations(s1 and s2), it's clear that if you initialize a POD class without parenthesis, you risk having uninitialized data members. Now if you try to access them, you might get a segfault error. This can be a real headache. If you don't know that leaving out parenthesis can cause an int to be uninitialized, then it will be hard to find the problem. 

Now let's consider another class. 

<pre lang="cpp-qt" lineno="1">
struct Stan
{
     int the;
public:
     Stan():the() {};
};
</pre>

According to C++03, since I have create a non-trivial default constructor, this is a non-POD class now. So it doesn't matter if I use the parenthesis, or I don't. 

So to sum it up, if you don't have a POD class, or if you're not intentionally planning to leave member objects uninitialized, then I would recommend that you always use parenthesis at the end of your class initializations to avoid possible headaches down the road. In reality, you will rarely create POD classes because they are so simple, it would make little sense to even need such a class. 

Note that there is a bug/feature in Visual Studio users <a href="https://connect.microsoft.com/VisualStudio/feedback/ViewFeedback.aspx?FeedbackID=100744">where you might have unitialized data members even if you use parenthesis</a>. I find this to be more of a bug then a feature, but then again I don't work for Microsoft. 