---
layout: post
title: "An Uninformed Design Choice"
date: 2019-03-17 10:00:00 +0300
categories: idea
published: false
---
The most discouraging part of the software development is that not knowing that there is already the wheel that you are inventing. However if you can justify your wheel with some features that no other wheel has then maybe there is hope. Yet sometimes the very feature you are building your wheel upon can easily be done with some tools that already existed before you and does the task well while going with your implementation requires some informed choices like sustainability or usability or dependability.

My stance on this was simply to write an entire TCP server and its client both on Java and C++ with Qt5 framework. It is, of course, easy to use and for the source belongs to me, I do any change that eases the development process. Multiple tasks can be carried out simultaneously and so on. And it is always easy to write this than use somebody else's base. Still though the server did never do the byte transferring task and the child tasks depend on the parent task to maintain the connection and their preparation. To start a receive process, the receiver has to inform the sender than the safe-checks gets done and finally the transfer begins. Yet there was always a better technology that could the tasks with more sophisticated manner and more error-proof way. That is torrenting, of course. At the time of writing and I have a bit of knowledge that helps me understand why it would be a better choice. For instance, it is UDP not TCP meaning that it does not active and packets are subject to get lost, still the task is always safe since once the packet arrives there is nothing important. Packet loss would be bad for voice communication, but this is not it. File transfer is something else. The result is the important phase not the progress. It could also help ease transfers among different devices.

With that said, I am now into writing a torrent based file sharer. It would not even need per platform compatibility.  It is always good to do some research before starting a project. 
