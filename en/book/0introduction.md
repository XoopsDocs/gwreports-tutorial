# Introduction

**gwReports **is a MySQL reporting tool for XOOPS Content Management System (CMS).

**gwReports **allows the CMS module administrator to enter SQL queries in such a way that they can be run by users in a controlled manner. The module administrator defines reports, which can consist of one or more SQL queries, known as sections. Access to a report is strictly controlled by the group permissions the module administrator assigns to it. A report can take parameters, such as a date range, if needed. The column output of a report section can be customized with a variety of formatting options. Reports can be organized into a menu system by topic, or accessed through system blocks. All of this is done without any programming required; only the creation of the SQL query is needed.

## Some possible uses 

#### Adding visibility to data collected within the CMS 

To a great extent, CMS data visibility is controlled by the individual module developer. Any use which differs from the original design may benefit from different visibility -- the same data, but presented in a different way. It may be possible to 'hack' the module in question to add the new features, or to persuade the developer to add options to the module to handle the different uses. But both of these options may present longer term issues in maintaining the changes against future updates to the module. Further, consider the possibility of views to data sourced from multiple modules. If it exists in the CMS database, gwreports will allow you give it visibility to your CMS users. 

#### Making external data visible in the CMS 

The CMS is a natural fit for a portal system. But the portal for an organization is likely just one of many systems. Users may benefit from dashboard style summaries from other systems to make the portal more valuable. Since you can't expect other systems to provide a XOOPS block for you, you need to forget it or find another way. If there isn't some neat web API you can use, but you can connect to the underlying data with MySQL, gwreports will allow you give it visibility to your CMS users. 

#### Accessing archived data 

Data is a valuable asset to an organization. The useful life of data can extend long beyond the life of the systems that originally captured it. gwreports and a CMS can make a cost effective and simple to implement front-end tool for accessing a MySQL back-end archive. 