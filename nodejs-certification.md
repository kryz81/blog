After a few years from the announcement, The OpenJS Foundation officially started the Node.js Certification Program.

{% twitter 1187541033770475522 %}

The program consists of two certifications:

* [OpenJS Node.js Application Developer (JSNAD)](https://training.linuxfoundation.org/certification/jsnad)
* [OpenJS Node.js Services Developer (JSNSD)](https://training.linuxfoundation.org/certification/jsnsd)

The beta version of the exams became available in September 2019. **I had a chance to take part in it and passed the JSNAD. In this post, I would like to describe my impressions and give you some tips for the exam.**


**_Disclaimer: You won’t find here any tasks, content, questions, answers, or code exercises from the exam. Due to the OpenJS Foundation Certification and Confidentiality Agreement, I am not allowed to share such information._**

First, I’m surprised that **the exam is not a quiz or a test**, where all you need to do is to select the checkbox with the correct answer or type the function name. There are many certifications, where you need to memorize a lot of stuff, or they check if you find a typo in the code example. For such exams, there are many so-called _braindumps_ on the internet; it’s possible to memorize them and pass such exams without any previous knowledge or experience. On the contrary, **The Node.js exams are in the form of the practical lab, in which you need to solve tasks by writing real code**.

Secondly, **you don’t need to memorize the whole Node.js API.** You can use the Node.js, the npm, and even the GitHub website, but you are not allowed to use StackOverflow and other similar forums. It doesn’t mean you don’t need to prepare for the exam, and you can simply copy/paste from these pages. You still should have a good knowledge of the whole Node.js ecosystem and concepts. For instance, if you don’t know how the Node.js streams work, you would probably waste too much time if you tried to learn it during the exam. Remember, **you have only two hours to finish all the tasks,** and in my opinion, it’s not very much for this exam.

As I mentioned before, the exam has a form of a lab. You get **remote access to an environment** with Linux, Node.js, VSCode, and a web browser. You also have access to the terminal. One drawback I noticed is that it is slower than working on a local machine; I lost some time because of occasional delays when I was opening a file or switching to the browser.

The advantage is that **you can take the exam from your home or office**. I find it much less stressful than making an appointment and taking an exam in a local testing center. Don’t forget that you will be observed during the exam (remember to clean your desk before the exam :wink:)

The OpenJS Foundation states this exam has an **intermediate level**, and I agree with that. On the one hand, the coding tasks are rather simple; on the other hand, not all the tested topics are used daily (at least I don’t use them).

**Here I collected some tips for you:**

* Read the exam scope **[here](https://training.linuxfoundation.org/certification/jsnad/#domains)** and learn all the listed concepts. **Write a lot of code; try to create small real-world examples.** The exam doesn't check your ability to remember all Node.js functions, but whether you can solve a coding exercise.
* Go through the core Node.js API, **focus on streams, buffers, the event system and child processes**. As described in the exam details they are the most important topics. 
* Please go through **the Node.js CLI commands and flags**, but don't memorize all of them! When I was preparing for the exam, I came across some flags I've never used before, even in large commercial projects.
* **Learn package.json** – fields, types of dependencies. It’s a practical exam, so you must know how to install a concrete version of a package. Don't forget to learn how the symantic versioning (semver) works.
* It’s a Node.js certification, but **your Javascript knowledge can also be tested**. In the exam description, there is a point called "JavaScript Prerequisites". So, it’s a good idea to refresh basic Javascript concepts like scopes, prototypes, closures, etc.
* Have you ever unit tested your code? Not great, not terrible :wink:. Pick one of the popular frameworks like Mocha or Jest and learn basics, for example, basic assertions. **Because of Node.js asynchronous nature, you have to know how to test asynchronous code**, for example, a function that returns a promise or expects a callback.
* Don’t forget to check if your code works! I know that it sounds obvious, but taking an exam is a stressful situation; the time is counting down, and we want to get all tasks done as quickly as possible. **You have access to the terminal, and you can run your code.**

Do you have any questions? Leave a comment below. If you liked this article, please tweet it.
