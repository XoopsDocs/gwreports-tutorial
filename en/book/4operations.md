# 4.0 Operating Instructions

### A Hands On Quick Start Guide
For the impatient, we will offer some quick examples before going through a comprehensive review of gwreports features. For this guide, we will assume you have downloaded and installed the full gwreports module on a suitable test system. It installs just like any CMS module.

####A 2 minute introduction
Go to the CMS administration area and enter the gwreports module Reports tab. Choose the Add Report link near the bottom of the page. On the Add New Report page enter the
following:
```
Name: Engines
SQL Query: select * from information_schema.ENGINES
```
Then click the Add button. You should now be on the Edit Report page. Near the bottom of the page, there is a Test button. Try it. You should now see a report of the storage engines available on your MySQL server.

Now, let's add it to a menu. Below the report is a Topics link, follow it. You will be back in the administration area. Follow the Add Topic link near the bottom. Fill in a Report Topic such as "Testing" (and add Description if you like.) Click Add.

Now from the bottom of the Edit Topic page, follow the Reports link. You should see the "Engines" report we entered a moment ago. Click on it. You should see the definition for our report on the Edit Report page.

Here we want to set three things that will make our report visible:
```
Authorized Groups - pick one or more groups (i.e. Webmasters)
Assign to Topic - pick our "Testing" topic
Is Active - set to Yes
```
Then save your changes. Then go to the gwreports module through the system main menu.
You should see a menu for the "Testing" topic listing the "Engines" report. Click on the link to view the report.

#### A more in depth example

In the previous example, we saw the mechanics of adding a simple query as a report
selectable from a menu. This example will demonstrate report parameters and column
formatting.

Add a new report named "Logins" with the following as the SQL Query:
```
select uid, uname, email, last_login
from {$xpfx}users
where last_login between {begin} and {end}
```
While you are at it, also set the Authorized Groups and Topic. From the report editor, follow the "Add Parameter" link in the Report Tools area near the bottom (just below the Test button.)

On the Add Report Parameter page, enter the following:
```
Parameter Name: begin
Title to Display: Begin Date
Parameter Type: date
Default Value: monday this week
```
Now, click Add. Now, we need to add another parameter, so follow the "Add Parameter" link in the Report Parameters area (with no parameters, it was called the Report Tools area.)

For the second parameter, enter the following:

```
Parameter Name: end
Title to Display: End Date
Parameter Type: date
Default Value: tomorrow
```
Click Add. Now click Test in the Report Parameters area. You should see a listing of user(s) who have logged on to the system this week. It is rather ugly. What is up with that last_login column?

Click on the last_login column header. Welcome to the Column Format editor. 

For right now, enter the following:
```
Display Title: Last Login
Column is Unix Time: Yes
```
Then, click Save. Test the report again.

To help see how this all fits together, click on the Logins link in the Report Sections area to see our query again. "{begin}" and "{end}" refer to the parameter names we entered. 

The brackets serve to make them identifiable in the SQL. By declaring them as date types, gwreports showed them as a date entry form input. The default values we entered got passed through the PHP strtotime() function before being displayed, so that wonderfully versatile function made sense of "monday this week" and "tomorrow" turning them into a actual dates. When we clicked Test, the query was executed with the unix time versions of our begin and end parameters substituted for the ```"{begin}"``` and ```"{end}"``` respectively. The ```{$xpfx}``` is a predefined parameter, that substitutes as the database prefix for your CMS. That just serves
to makes it easier to move a query between systems.

Lets go one step further. Test the report again. Click on the uname column header. Enter the following:
```
Display Title: Username
Extended format: <a href="{$xurl}/userinfo.php?uid={uid}">{uname}</a>
```
Then save, and test again. Notice that the Username column in the report is now a link to the system user info page. In the Extended format, we can reference any column by using the column name in brackets, uid as {uid}, uname as {uname}, etc. There is also another handy predefined substitution, ```XOOPS_URL``` as ```{$xurl}```.

One more fun trick, enter the column editor with the uid column. Set Hide this column to Yes and save. Test the report again. The uid column is hidden from view, but still used in the uname link.

So far we have learned that we can declare parameters and reference them within our SQL, and we can make formatting changes to columns with the column editor. To complete this example, go back and edit the report. We need to set the following:
```
Is Active: 
```
Yes Now go check out how the report functions from the user side. All of the editor functions are gone, and you have a report that looks, feels and functions like a part of your CMS.