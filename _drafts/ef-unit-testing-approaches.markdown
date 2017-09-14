post about testing approaches in EF

1. mock dbcontext.
Pros:
- quick and easy to setup
Cons:
- Does not support extension methods, like AddOrUpdate()
- Does not test Includes. e.g. you might miss Include() your test will work but prod code -- won't.

2. use Effort
Pros
- is more close to the real DB
- supports many ways of seeding DB
Cons
- more complicated to setup 