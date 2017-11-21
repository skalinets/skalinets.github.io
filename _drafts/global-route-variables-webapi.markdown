

We have a Web API application that should be run in multitenancy mode. It means that different tenants
should have unique branding and not be able to see data of another tenant. From the very start of the
project the decision was made to use rather naive multi-tenancy model -- every tenant specific table
just has `TenantId` column. Built on top of that foundation we released few versions of application with
only one tenant. Since it was only one, we did not bother with complete support of multiple tenants. In
particular our URLs did not have anything related to tenants. Tenant details were hardcoded across the
system and everyone was happy with that. 

Then time has come for adding another tenants. And we faced the problem: how to     


Ninject does not honor OWIN request scope (https://github.com/ninject/Ninject.Web.WebApi/issues/17)