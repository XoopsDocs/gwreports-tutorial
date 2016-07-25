# 8.0  Security Considerations 

gwreports gives the module administrator the ability to enter any SQL and give users a mechanism to execute that SQL through the CMS database connection at will. This places a significant responsibility on the module administrator to ensure that any reports created do not introduce security exposures. If you enter potentially destructive SQL, expect destructive results. If you indiscriminately expose sensitive data, expect data disclosure breaches. Below are a few suggestions that may be useful. 

Use good change control. If you are editing an existing report, make sure it is not active. A report that is not set as active is only available to the module administrator in the editing and testing functions, and is totally invisible in all other ways (i.e. menus, blocks, search, direct access.) This will keep incomplete work from inadvertently disclosing information. 

It is strongly recommended that you use limited mode (see below) for any production CMS environments. 

CMS groups are a very powerful tool if used correctly. A single user can be in multiple groups. If you have a super secret report that only a few users should be allowed to see, you can create a super secret report group, add that group to the select users, and set the authorized groups for the super secret report to only the super secret report group. 

There are no restrictions on what sort of SQL is included in a report section. The primary considered use is for SELECT statements. If you enter DELETE, DROP, INSERT, UPDATE or other SQL statements, they will be dutifully executed. The advice here is in general, don't do it, no matter how expedient it may seem at the time. There could be legitimate reasons you want to create and populate a temporary table in the process of generating a report, but there is no reliable safety net for these actions. You have been warned. 

No automatic resource limiting is applied. Queries which may return very large result set may create resource exhaustion issues on the server and/or client(s). You should consider adding a LIMIT clause to any query where the result set size could be an issue. It is highly recommended that you prototype queries in a development environment separate from your production environment to prevent service disruptions caused by excessive resource consumption of a report in development. 

An exported report definition file can be easily shared, but beware of importing reports from unknown or untrusted sources. Just as there are no restrictions on what SQL you can include in a report, there are no restrictions on what can be exported and imported. The import process will drop straight into the report editor on completion, but never just import and hit the Test button. It is highly recommended that any shared report be imported only into a suitable test environment and that you carefully review all aspects of the freshly imported report before executing it in any fashion. An imported section's SQL query can carry malicious SQL, and both queries and column formats can carry malicious HTML. 

###Limited Mode 

In the event that your CMS experiences an administrative account compromise, the power of gwreports could represent an increased attack surface due to the ability to easily run arbitrary SQL and quickly export data. As an example, with administrative credentials, the elapsed time from the login prompt to having downloaded a spreadsheet of all the data from the CMS users table can be under a minute. This same data could be obtained in other ways without using gwreports, but it could take a lot more effort and time. 

An administrative account compromise is an extremely serious situation. Using strong passwords, secure connections and other industry recommended practices are of course the front line of protection, but in the event that an intruder obtains administrator access, you don't want to make his illicit activities any easier than you have to. 

gwreports "Limited" mode is designed to mitigate this risk for production environments. In this mode, the reporting interface is available, but without the tools for SQL entry or modification. Creation of a new report can only be done through a restricted import interface, where you effectively prove you have access to the module administration and the underlying file system. At first, this might seem cumbersome, but it reinforces the basic change management philosophy of separating development and production environments and making the transition from one to the next deliberate and controlled. 

There are two ways to install gwreports in limited mode: 

First, you can choose a gwreports-limited package and install it. 

Secondly, you can convert a full installation to a limited one by copying the contents of the 

```modules/gwreports/limited ```

directory over the modules/gwreports directory, and then updating the module. 

In both cases you will need to create a directory in the 

```TRUST_PATH``` (also called ```XOOPS_LIB```) where report import files can be placed. The import function will look in 

```php XOOPS_TRUST_PATH/modules/gwreports/import/``` 

for files. You must manually create this directory. 

For maximum security, you should carefully follow your CMS instructions on naming, locating and protecting the TRUST_PATH directory. In a limited mode gwreports, the administrator can do the following: 

* Create and edit Topic definitions. 
* Edit report names, descriptions, assigned groups, topics and active flags. 
* Edit column formats and parameter definitions 
* Export existing reports. 
* Import reports from files uploaded to the special ```TRUST_PATH``` directory. 

In limited mode, the following functions are disabled: 

* Adding or editing SQL, in report sections and parameters 
* The explore function 
* Importing a report definition from a direct file upload 

You can convert a limited mode installation to a full installation by overwriting the module files with a full install package and then updating the module. 

