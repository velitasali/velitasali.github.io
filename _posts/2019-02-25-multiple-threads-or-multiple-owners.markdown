---
layout: post
title:  "Multiple Threads or Multiple Owners?"
date:   2019-02-25 10:00:00 +0300
categories: threading design android
published: true
---
And again I am at the point where something works, but kinda. On TrebleShot's *Adding more devices to a transfer*,
everything works as long as user does not change the language, or the orientation in short recreate it, or leave
the app leaving it to the hands of LMK (Low Memory Killer). When that happens the activity that has started the thread simply loses its instance causing a new thread to spawn and the other thread to die. Other thread dies because we will not know where will it be or it will belong to us if we don't kill it.

```java
@Override
protected void onDestroy()
{
    super.onDestroy();
    getDefaultInterrupter().interrupt(false);
}
```

`getDefaultInterrupter()` returns the class level instance of the `Interrupter` object that notifies threads to seize actions. It is a good way to handle multiple threads. It supports something called *Closers*. It is a class that does action when the `interrupt()` is called. I used this method throughout TrebleShot.

# Show the right activity, but how?
What I want to achieve is that when user leaves the app and clicks the notification that they can still see what they left off. It is important for consistency. Another way is loop all the tasks as it is their turn for instance


```java
public class Task extends RemoteTask
{   
    private int mPosition = 0;

    @Override
    protected void run()
    {
        //todo: task
        mPosition++;
    }
}
```

So every task can be called one by one and all the tasks can run on the same thread. Of course, that is when a task can
detach itself in time, which means not doing recursive tasks and remembering the stage it was at. So, I did not like this and the idea that reorganizing the current methods (a lot of trouble if not done carefully).

# Remembering who was who
When a task is started, since they are mostly done by activity which has a certain `android.content.Intent` that started them then, comparing elements of them looked like a sane idea.

Before starting, the task give them the intent that started the activity so by putting that `Intent` into a `PendingIntent`
we can start the same instance of that activity. And, we can even reattach the task when it happens.

So to do that I created `intentHash()` which returns a string hash for an `Intent`. The method looks like this;

```java
public static int intentHash(@NonNull Intent intent)
{
    StringBuilder builder = new StringBuilder();

    builder.append(intent.getComponent());
    builder.append(intent.getData());
    builder.append(intent.getPackage());
    builder.append(intent.getAction());
    builder.append(intent.getFlags());
    builder.append(intent.getType());

    if (intent.getExtras() != null)
        builder.append(intent.getExtras().toString());

    return builder.toString().hashCode();
}
```

So `RunningTask` class holds a hash code and a pending intent. We have everything ready and it is time to reattach the task. Not quite so! What if I want to show a progress bar and a button and a bunch of other informative widgets that benefits from this. They should be updated by the `RunningTask` and they should definitely not crash the app when the activity dies. So to that I added something simple called *Anchors*.

Anchors come and go and they are definitely not reliable. That is they may exist at times, but mostly they do not. This means that whenever a task is attached to an activity, the activity attaches a listener (mostly itself) and by using templates, the `RunningTask` can always know what it will have if it can have it.

So that is why the definition of the RunningTask is;

```java
public abstract static class RunningTask<T extends OnAttachListener> extends InterruptAwareJob
{
 // ...
}
```

And the `OnAttachListener` is;

```java
public interface OnAttachListener
{
    void onAttachedToTask(RunningTask task);
}
```

And finally on the activity I call `WorkerService.findTaskByHash()` which returns tasks when there is any.

```java
public synchronized RunningTask findTaskByHash(int hashCode)
{
    synchronized (getTaskList()) {
       for (RunningTask runningTask : getTaskList())
           if (runningTask.hashCode() == hashCode)
               return runningTask;
    }

    return null;
}
```
