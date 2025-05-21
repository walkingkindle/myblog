+++
date = '2025-05-21T11:44:09+02:00'
title = 'Concurrency In C# Cookbook'
+++


How would you solve a problem where you are inheriting from an async method, but you want to implement it synchronously (in C#)?

What about Reporting Progress as a double value as an operation is executing? Or returing any of the tasks that were completed, instead of all?


This is what Concurrency in C# is about. Stephen Cleary is one of the authors I found through his blog post that was very famous among the C# community. This inspired him to write a book only about Concurrency, and async implementations in C#, and share his ideas.

Chapter 1 goes through some of the basic ideas about concurrency, parrarelism, multithreading, as well as some of the misconceptions that people have regarding these.

Chapter 2 is then a more practical chapter where he poses a problem that is set by an async method, and how would you solve it based on the result.

I will list some of the approaches here that I found useful:

>2.3 Reporting Progress

>Problem: You need to respond to progress while an operation is executing.

For this, you can use the Progress<T> argument, passing in an immutable, or at least a value reference, as suggested by Stephen Cleary. that you can return like so:

```
static async Task MyMethodAsync(IProgress<double> progress = null)
{
 double percentComplete = 0;
 while (!done)
 {
 ...
 if (progress != null)
 progress.Report(percentComplete);
 }
}
2.3. Reporting Progress | 21
Calling code can use it as such:
static async Task CallMyMethodAsync()
{
 var progress = new Progress<double>();
 progress.ProgressChanged += (sender, args) =>
 {
 ...
 };
 await MyMethodAsync(progress);

```

>2.4 Waiting for a set of Tasks to Complete.

This one is already familiar, but worth mentioning.

>Problem :You have several tasks and need to wait for them all to complete.

The framework provides a Task.WhenAll method for this purpose. This method takes
several tasks and returns a task that completes when all of those tasks have completed:
```
Task task1 = Task.Delay(TimeSpan.FromSeconds(1));
Task task2 = Task.Delay(TimeSpan.FromSeconds(2));
Task task3 = Task.Delay(TimeSpan.FromSeconds(1));
await Task.WhenAll(task1, task2, task3);

```
>If any of the tasks throws an exception, then Task.WhenAll will fault its returned task
with that exception. If multiple tasks throw an exception, then all of those exceptions
are placed on the Task returned by Task.WhenAll. However, when that task is awaited,
only one of them will be thrown. If you need each specific exception, you can examine
the Exception property on the Task returned by Task.WhenAll:


>2.5 Waiting for any of the tasks to complete

>You have several tasks and need to respond to just one of them completing. The most
common situation for this is when you have multiple independent attempts at an op‐
eration, with a first-one-takes-all kind of structure. For example, you could request stock
quotes from multiple web services simultaneously, but you only care about the first one
that responds.

>Use the Task.WhenAny method. This method takes a sequence of tasks and returns a
task that completes when any of the tasks complete. The result of the returned task is
the task that completed. Don’t worry if that sounds confusing; it’s one of those things
that’s difficult to explain but easy to demonstrate:
```
// Returns the length of data at the first URL to respond.
private static async Task<int> FirstRespondingUrlAsync(string urlA, string urlB)
{
 var httpClient = new HttpClient();
 // Start both downloads concurrently.
 Task<byte[]> downloadTaskA = httpClient.GetByteArrayAsync(urlA);
 Task<byte[]> downloadTaskB = httpClient.GetByteArrayAsync(urlB);
 // Wait for either of the tasks to complete.
 Task<byte[]> completedTask =
 await Task.WhenAny(downloadTaskA, downloadTaskB);
 // Return the length of the data retrieved from that URL.
 byte[] data = await completedTask;
 return data.Length;
}

```

>The task returned by Task.WhenAny never completes in a faulted or canceled state. It
always results in the first Task to complete; if that task completed with an exception,
then the exception is not propogated to the task returned by Task.WhenAny. For this
reason, you should usually await the task after it has completed.

>When the first task completes, consider whether to cancel the remaining tasks. If the
other tasks are not canceled but are also never awaited, then they are abandoned. Aban‐
doned tasks will run to completion, and their results will be ignored. Any exceptions
from those abandoned tasks will also be ignored.


>When an async method resumes after an await, by default it will resume executing
within the same context. This can cause performance problems if that context was a UI
context and a large number of async methods are resuming on the UI context.
Solution
To avoid resuming on a context, await the result of ConfigureAwait and pass false
for its continueOnCapturedContext parameter:
```
async Task ResumeOnContextAsync()
{
 await Task.Delay(TimeSpan.FromSeconds(1));
 // This method resumes within the same context.
}
async Task ResumeWithoutContextAsync()
{
 await Task.Delay(TimeSpan.FromSeconds(1)).ConfigureAwait(false);
}

 ```


>2.8. Handling Exceptions from async Task Methods

>Exceptions raised from async Task methods are placed on the returned Task. They are
only raised when the returned Task is awaited.