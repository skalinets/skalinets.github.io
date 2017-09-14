---

layout: post
title: "Spatial Pain with Entity Framework"
date: 2017-09-05 12:00 +02:00
categories: ELK devops windows .NET

---

In one of our project there is a requirement to sort addresses by the distance to one of them. We use Azure SQL Database
as a storage and Entity Framework 6.1.3 as an ORM. 

In 2017 all modern databases [seem](https://www.scribd.com/presentation/2569355/Geo-Distance-Search-with-MySQL) to support 
spatial calculations so does Azure SQL Database. Moreover, Entity Framework also allows to use geo operations in its
queries. After all it's just a bunch of [sines and cosines](https://www.scribd.com/presentation/2569355/Geo-Distance-Search-with-MySQL) -- nothing complicated, right?      


post about spatial and entity framework

calculations should be pretty simple: https://www.scribd.com/presentation/2569355/Geo-Distance-Search-with-MySQL

First you get [crazy error](https://stackoverflow.com/questions/19551142/entitytype-dbgeography-has-no-key-defined), that 
is caused by a wrong namespace in [Microsoft's tutorial](https://msdn.microsoft.com/en-us/data/hh859721). OK, thanks to 
Stackoverflow, it's fixed.

Next -- a new one. 
{%highlight json%}
error 2: {"Spatial types and functions are not available for this provider because the assembly 'Microsoft.SqlServer.Types' version 10 or higher could not be found. "}
{%endhighlight%}

error 1: 
System.Data.Entity.ModelConfiguration.ModelValidationException was unhandled
  HResult=-2146233088
  Message=One or more validation errors were detected during model generation:

TestApp.DbGeography: : EntityType 'DbGeography' has no key defined. Define the key for this EntityType.
DbGeographies: EntityType: EntitySet 'DbGeographies' is based on type 'DbGeography' that has no keys defined.

  Source=EntityFramework
  StackTrace:
       at System.Data.Entity.Core.Metadata.Edm.EdmModel.Validate()
       at System.Data.Entity.DbModelBuilder.Build(DbProviderManifest providerManifest, DbProviderInfo providerInfo)
       at System.Data.Entity.DbModelBuilder.Build(DbConnection providerConnection)
       at System.Data.Entity.Internal.LazyInternalContext.CreateModel(LazyInternalContext internalContext)
       at System.Data.Entity.Internal.RetryLazy`2.GetValue(TInput input)
       at System.Data.Entity.Internal.LazyInternalContext.InitializeContext()
       at System.Data.Entity.Internal.InternalContext.GetEntitySetAndBaseTypeForType(Type entityType)
       at System.Data.Entity.Internal.Linq.InternalSet`1.Initialize()
       at System.Data.Entity.Internal.Linq.InternalSet`1.get_InternalContext()
       at System.Data.Entity.Internal.Linq.InternalSet`1.ActOnSet(Action action, EntityState newState, Object entity, String methodName)
       at System.Data.Entity.Internal.Linq.InternalSet`1.Add(Object entity)
       at System.Data.Entity.DbSet`1.Add(TEntity entity)
       at TestApp.Program.Main(String[] args) in c:\users\admin\work\SpatialDemo\TestApp\Program.cs:line 25
       at System.AppDomain._nExecuteAssembly(RuntimeAssembly assembly, String[] args)
       at System.AppDomain.ExecuteAssembly(String assemblyFile, Evidence assemblySecurity, String[] args)
       at Microsoft.VisualStudio.HostingProcess.HostProc.RunUsersAssembly()
       at System.Threading.ThreadHelper.ThreadStart_Context(Object state)
       at System.Threading.ExecutionContext.RunInternal(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
       at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state, Boolean preserveSyncCtx)
       at System.Threading.ExecutionContext.Run(ExecutionContext executionContext, ContextCallback callback, Object state)
       at System.Threading.ThreadHelper.ThreadStart()
  InnerException: 


error 2: {"Spatial types and functions are not available for this provider because the assembly 'Microsoft.SqlServer.Types' version 10 or higher could not be found. "}

I tried to install it via chocolatey: https://chocolatey.org/packages/sql2016-clrtypes. Maybe I need to reboot? No luck..
https://stackoverflow.com/questions/43221467/assembly-microsoft-sqlserver-types-version-10-or-higher-could-not-be-found here guy posted link to CLR types for SQL Server 2012 (!). After that install it started to work in console application, but apparently not in ASP.NET. 
I found this [SO comment] saying that some x86 magic should be done with IIS. In my case it was IIS Express and I don't know how to enable 32bit applications there. But instead I just installed x86 version of  SQL CLR types and it started to work.

