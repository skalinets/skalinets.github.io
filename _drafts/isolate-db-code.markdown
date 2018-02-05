---

layout: post
title: Don't Mess Your Code with DB Calls

---

90% of .NET projects are DB related. That said they 

repository code is everywhere

You ain't need repository pattern in EF!!!!
it's hard to 

use DbContext directly in services, don't introduce repositories. DbContext is a Unit of Work already. 

problem with repositories:

service: Foreach (repo1.Where()) {repo2.check(item)}  // results in concurrency exception

LinguistActionLogs

Exception thrown: 'System.NotSupportedException' in mscorlib.dll

Additional information: A second operation started on this context before a previous asynchronous operation completed. Use 'await' to ensure that any asynchronous operations have completed before calling another method on this context. Any instance members are not guaranteed to be thread safe.