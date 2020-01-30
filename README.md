![MIKES DATA WORK GIT REPO](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_01.png "Mikes Data Work")        

# Solution To SQL Revoke Permissions Error
**Post Date: February 26, 2015**        



## Contents    
- [About Process](##About-Process)  
- [SQL Logic](#SQL-Logic)  
- [Build Info](#Build-Info)  
- [Author](#Author)  
- [License](#License)       

## About-Process

<p>Error: Server principal 'INNChris' has granted one or more permission(s). Revoke the permission(s) before dropping the server principal.
You've run across this error before many times. Yes; it's annoying, and then even more so when you have to run through the Securable Permissions under the Login Properties to resolve the problem. Why go through the GUI when you can simply run some quick SQL logic.</p>    

![Find Securables]( https://mikesdatawork.files.wordpress.com/2015/02/image001.jpg "Find SQL Securables")
 
What it looks like when you query it directly.



## SQL-Logic
```SQL
use master;
set nocount on
 
select
       'account name'             = servprin.name
,      'type'                     = servprin.type_desc
,      'permission'               = servperm.permission_name
,      'state'                    = servperm.state_desc
from
       sys.server_principals servprin
       join sys.server_permissions servperm
       on  servprin.principal_id = servperm.grantee_principal_id
where
       servprin.type in ('s', 'u', 'g')
       and servprin.name like '%chris%'
order by
       'account name' asc
```
![See Account Info]( https://mikesdatawork.files.wordpress.com/2015/02/image006.jpg "See Account Info")
 
Once you can see the explicit permissions that were granted, all you need to do is run the following statement to remove (aka REVOKE) the permissions from the account so you can drop the login.
Lets drop those CONNECT permissions from the account.
</p>      
## SQL-Logic
```SQL
1	revoke connect sql to [innchris] as [sa];
```
Next; we'll need to determine what kind of GRANTS that Chris did under his account, then simply remove those granted permissions from those objects. You might want to notate what objects had the GRANT permissions applied to them and simply apply the GRANTS back to them using a more universal account.



## SQL-Logic
```SQL
1
2
3	declare @grants     varchar(50)
set     @grants     = ( select principal_id from sys.server_principals where name = 'mydomain\myusername' )
select * from sys.server_permissions where grantor_principal_id = @grants
```
![See Major_ID]( https://mikesdatawork.files.wordpress.com/2015/02/image008.png "See Major_ID")
 
So now we know that Chris granted Connect Permissions to an Endpoint. This is where you need to be careful because even though we can simply revoke the permission; you kinda don't want to break the connectivity to the Endpoint. The best thing to do is have another account take ownership of the Endpoint, and by doing so all corresponding permissions will simply be transferred to the new account.
First; lets see which endpoint that we are dealing with. So far all we have to go on is the 'Major_ID', but not the Endpoint name which we'll need in order to run the 'take ownership' grant statement.

![See Endpoint_ID]( https://mikesdatawork.files.wordpress.com/2015/02/image009.png "See Endpoint_ID")
 

## SQL-Logic
```SQL


1
2
3
4	use master;
grant take ownership on endpoint::hadr_endpoint to sa
    with grant option;
go
```
Notice the 'sa'. Unfortunately; you can't do this. You may feel compelled to use a basic pre-existing account to get through this step, but before you think about using 'sa', consider this… It will error out. That's just the way it's designed.

![See Grant Statement]( https://mikesdatawork.files.wordpress.com/2015/02/image007.png "See Grant Statement")
 
All you need do in this case is create another SQL Login. In this case; we'll be using an SQL Login called EndpointOwner. Nothing special here, just a new account with sa rights. You can add something more complex with more granular rights later. The main goal here is to remove the accounts that need to be removed, and making sure you are giving ownership of the existing endpoint to another account so you don't cause any problems by revoking the old one.




[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

[![Gist](https://img.shields.io/badge/Gist-MikesDataWork-<COLOR>.svg)](https://gist.github.com/mikesdatawork)
[![Twitter](https://img.shields.io/badge/Twitter-MikesDataWork-<COLOR>.svg)](https://twitter.com/mikesdatawork)
[![Wordpress](https://img.shields.io/badge/Wordpress-MikesDataWork-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

     
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Mikes Data Work](https://raw.githubusercontent.com/mikesdatawork/images/master/git_mikes_data_work_banner_02.png "Mikes Data Work")

