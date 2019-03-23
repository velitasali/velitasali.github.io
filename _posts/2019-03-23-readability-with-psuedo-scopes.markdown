---
layout: post
title: Readability with Scopes
date: 2019-03-23 10:00:00 +0300
categories: coding-style
published: true
---
Most programmers do think that their style of coding is superior when they develop small projects and they sometimes find themselves staring at the code that they wrote earlier, admiring the way it looks and works until they find out that it is not simply the best nor the most readable one. Readability is an important part of software development and it becomes more important as the number of people involving grows. Even at times the developer who wrote the code may not recognize or understand. This is either because he/she already has changed the way architects his/her software or because he/she is a messy programmer. Well, there are ways to improve this kind of situations by simply adopting some traits. Commenting, documenting aside, there are the ones that changes the way the code is read like wrapping, spacing etc. I will not talk about them. I will talk about what I call `scoping`. I can't really say this is the best way referring to it, but when you are done reading this post, hopefully you will understand. I discovered this one last year and I consider it one of the most helpful way of stating the way your software works. I have never seen or heard someone mentioning them in software development, and probably you haven't either. So let's go.

You know when you have a `if` condition with multiple lines of code and working with Java, C++ etc., you use braces `{` `}` to tell the computer where it should do tricky task and where it should end it. What if I tell you, braces can always be used even though they do not have any keyword before them like `do`, `while`, `try` etc. And using them certainly have an effect with a powerful IDE, and on amostly a language like C++.

Look here where I connect to server. This is a client that should connect two separate servers to do its tasks.
```cpp
try {
  {
    auto * activeConnection = client->communicate(device, connection);
    //todo: other tasks
    delete activeConnection;
  }

  {
    auto * activeConnection = client->communicate(device, connection2);
    //todo: other tasks
    delete activeConnection;
  }
} catch(...) {
  //todo: catch errors
}
```

As you can see, there two different pointers with the same name, but neither compiler nor you powerful IDE will confuse them because they are in different scope. This code throws an exception when any of the two connections fails, so nothing breaks the code.

You may say "why would I need this?" Simple! This will save you from using a lot of variables mistakenly and when you use temporary objects, they will be cleaned when they are created in these headless braces.

Like this;

```cpp
#include <iostream>

class TmpObject {
public:
  ~TmpObject() {
    std::cout << "TmpObject deleted" << std::endl;
  }
}

int main(int argc, char *argv[]) {
  {
    TmpObject();  
  }

  std::cout << "Main scope" << std::endl;
  return 0;
}
```

The output is;
```
TmpObject deleted
Main scope
```

If we didn't call `TmpObject();` in those headless braces the first output for this program would be `Main scope` and then `TmpObject deleted` which means `TmpObject` is deleted after the function exits. However, in the example above, we get rid of not used objects faster and clean mess earlier.

For me, using this was a time saver, since in a server-client structure, there are too many variables to handle and it is hard to remember what they are used for.

Check [SeamlessClient](https://github.com/genonbeta/TrebleShot-Desktop/blob/84902b3d889b85c9d45439e58b6c78d108f9a603/src/broadcast/SeamlessClient.cpp#L125) and
[SeamlessServer](https://github.com/genonbeta/TrebleShot/blob/55175f432fd5df6eee7d9b1c3ced4efc84927aed/app/src/main/java/com/genonbeta/TrebleShot/service/CommunicationService.java#L988) out. One is C++ and the other is Java and they work together.
