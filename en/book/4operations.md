# 4.0 Operating Instructions

## Concepts

#### Topic

Topics serve to group reports for menus and blocks. A topic is only visible to a user if there are one or more reports assigned to the topic that the user has the authority to view.

**Related Elements**

* **Report Topic **- Name for the topic, used through out the system, including menus 
* **Description **- A description of the topic shown in tool tips and the header of a topic's menu 

#### Report

A report is a set of SQL queries, parameter definitions, permissions and formatting instructions that together for a unit which can be invoked by a user to display data residing in MySQL tables. A user's authority to access a report is controlled by the CMS groups authorized to use the report by the module administrator.

**Related Elements**

* **Name **- Name for the report, used through out the system, including menus 
* **Authorized Groups** - CMS groups authorized to view the report Assign to Topic - Topic that a report is assigned to 
* **Description **- A description of the report shown in tool tips and when parameters are collected prior to running a report 
* **Is Active** - Allows a report to be turned on or off. A report will be available to run only when Active is Yes. When a report is first added, this is set to No, and set this way a report can be edited and tested, but it will not be available outside of the editing environment. 

#### Section

A section is a part of a report definition that specifies a single SQL query and related formating instructions. When a report is first created, the initial query is used to create a section named the same as the containing report. This section has default values, and can be edited. Multiple sections are executed in order, and their output appended to the report. The order of the sections in a report can be changed using the Reorder Section function.

**Related Elements**

* **Section Name** - Name for the section, used in editing, and optionally shown in the report. 
* **SQL Query** - A query to execute to retrieve data for the report section. This query will be executed in the context of the normal CMS database user and connection. To reference report parameters in the query use the name of the parameter in brackets, i.e. {name}. To reference the CMS database prefix use {$xpfx}. To reference the uid of the current user use {$xuid}. 
* **Show Section Name** - if Yes, the section name will be show as a header at the top of the section in the report. 
* **Section is Multirow** - if Yes, the results of the query are shown as a row of column names, followed by rows of result data. If No, the results of the query are show in rows consisting of the column name and column data, and this is repeated for each row of result data. 
* **Surpress display if empty** - if No, a message will be presented in the report if the query did not return any data. If set to Yes, the entire section will be quietly omitted from the report. 
* **Enable data tools** – If set to Yes, the section will include data tools, to change sort order, filter rows and page through the data. The data tools option is presently implemented using the jQuery dataTables plugin. This option only applies when 'Section is Multirow' is set to Yes. This option may produce undesired interference with column formats such as Sum, Break or Outline. 
* **Description** - A description of the section shown in tool tips 

#### Parameter

A parameter is a specific named input to a report that is obtained from the user before executing any SQL queries to extract data from MySQL. Insertion points for named parameters are specified in the SQL as the name surrounded with curled brackets, i.e. {name}. 

Related Elements 
* Parameter Name - the name for this parameter, used to reference the parameter value in a section's SQL, and as the root of the name used in the form for collecting parameter's from the user. Parameter names starting with a '$' and the name 'rid' are reserved and should not be used.
* Title to Display - the title displayed when this parameter is collected for or shown in summary on a report 
* Description - a description of the parameter shown in tool tips 
* Field Length - the maximum length of the form input used to collect the parameter 
* Number of Decimal Places - the number of decimal places for a decimal parameter 
* Required - if Yes, entry of the parameter is required. When a report is run, it will show a parameter entry form to the user if a required parameter has not been entered, even if the parameter has a default value defined. 
* Parameter Type - the type of parameter which controls how the parameter entry is displayed an processed. Valid types are: 
    * text - a straight text value 
    * liketext - a text value which will be surrounded by like characters before being passed to the query, i.e. %value% 
    * date - a date value which will be converted to a MySQL unix\_timestamp before being passed to the query. The input will be converted using the PHP strtotime\(\) function. 
    * integer - the input will be interpreted as an integer before being passed to the query. 
    * decimal - the input will be interpreted as a decimal number rounded and formated to tne number of decimal places specified before being passed to the query.         
    * yes\/no - the input is interpreted as a 1 \(yes\) or 0 \(no\) before being passed to the query. 
    * auto complete – the input will be treated as text, and the input will be passed through an auto-completion process as the parameter is being entered. Once two or more characters are entered and there is a short pause in typing, the input will be used as a filter against the results of the 'SQL Query for Auto Complete.' Please remember that the parameter input is text and should be used as such in queries, even if the auto complete value selected came from a numeric column. 


* Default Value - a default value for the parameter. 
* SQL Query for Auto Complete – this is a query that is executed to obtain the values used for an auto complete parameter. This query should return rows consisting of two columns, one named value and one named label. The only parameter substitution performed before execution of this query is the replacing the string ```'{$xpfx}'``` with the current database prefix. As an example, using an auto complete parameter for a system user id could use a query like this: 
```php
select uid as value, concat(uid, ' - ', uname) as label from {$xpfx}xoops_users
``` 

With this query, either a user id or a user name can be used as entry for the parameter, and selecting a user name from the list will enter the corresponding uid into the parameter field.

## Column

A column is a named data as returned by an SQL query. A query returns a result set that may contain rows. Each row contains one of more named columns. Column formating instructions specific to a section are associated with the result data by the column name. 

Related Elements 
* Column Name - the name of the column in the MySQL result set when the section query is executed. It may be convenient to use an alias when the select\_expr for the column is a function, i.e. ```SELECT CONCAT(last_name,', ',first_name) AS full_name``` 

* Display Title - the title of the column shown in the report section header. If blank, the column name will be used. Hide this column? - if Yes, the column will not be shown Sum this column?- if Yes, the column values will be summed, and shown in summary at the end of the report section and at any column change breaks. Break on column change? - if Yes, a break line will be inserted at any change in the column value from one row to the next. This break column will include a sub-total of any sum columns in the section. Outline column? - if Yes, the display of any column value which is equal to the column value of the preceding row will be suppressed and replaced by a blank value. Convert newlines? - if Yes, any newline characters in the column value will be converted to HTML break tags. Column is Unix Time? - if Yes, the value of the column will be treated as a MySQL unix\_timestamp. By default this will be converted to a display value using the CMS formatTimestamp\(\) function, but an alternative format can be specified in the format string. sprintf\(\) or date\(\) format string - a format string to be used for this column, which will be interpreted in one of two ways: If the column is unix time is yes, the format string and column value will be passed to the PHP date\(\) function, and the return of that function will be shown in the report Otherwise, the format string and column value will be passed to the PHP sprintf\(\) function, and the return of that function will be shown in the report HTML\/CSS style for column - any value supplied here will be inserted as attributes in the HTML TD tag surrounding the column data in the report display. Extended format - The extended format can be used to build any arbitrary string from the column data for a result row. Any column data can be referenced by the column name in {} brackets, i.e. {name}. The value of the the current column will be as otherwise specified in the definition, while other columns will be the value as returned in the result set. In addition to named column values, the term {$xurl} in the extended format will be replaced by the base CMS URL \(i.e. XOOPS\_URL.\)

## Accessing Reports

Menu System gwreports has a menu system built on Topics. When accessed, the main page will show a list of all topics that have reports that the current CMS user can access. If the user can access only a single topic, that topic is automatically selected. When a topic is selected, a list of all reports the user can access for that single topic is shown. When a report is selected, the report viewer will be shown. A form showing the parameters of the report will be shown. If the report has no required parameters, the report will run automatically and the results are displayed. The user can enter or change the parameters and run the report. If enabled, the user also has the options to print the report, or export the data to a spreadsheet. The menus and report viewer have an optional breadcrumb menu for quick navigation. Blocks gwreports offers several CMS blocks. Topic Menu Displays a list of reports for a single topic that the current user can access. Selecting a report from this list invokes the report viewer for the selected report. Quick Report Displays a parameter entry form to run a single report in a block. Submitting the form invokes the report viewer. If no parameters are called for, a link to the report is shown. Report in a Block Displays a preselected report in a block. No opportunity for parameter entry is presented, so the report should have either no parameters or suitable default values for all parameters. It is recommended that any report used in this context have a predictably small result set. To minimize resource use, the block group permissions should only include groups which have permission to view the report. Also, specifying block caching should be considered. Search Access Reports can be located through the CMS search function by name or description. Only reports the user is authorized to view will be returned in the search results. Direct Report Access A report can be invoked directly with a URL of the following form: XOOPS\_URL\/modules\/gwreports\/report\_view.php?rid=NN&name1=value1&name2=value2 Where: NN is the report ID \(shown in the Report list in the Administration area\) nameX is the parameter name valueX is the value to be used for the parameter For a printable version of the report, you can substitute report\_print.php. For a spreadsheet version of the report, you can substitute report\_xls.php.

#### Explore

The Explore tab in the Administration area provides a quick view of the tables you can access when creating a report. Select a database, and a list of tables will be displayed. Select one of those tables, and a query including all of the columns of the table will be displayed. The generated query will also include a simple where clause referencing the tables primary key if one is defined. You can optionally pass the generated query to the report editor as a new report. Export \/ Import A report definition, including all parameter, section and column format definitions, can be exported to a text file using the Export option on the administration area Reports list. A corresponding Import function, to import an exported report definition file, can be accessed from the Import link near the bottom of the administration area Reports page. These functions allow you to more easily move a report from a development to a production environment. Using options such as the predefined {$xpfx} parameter in your query SQL can make the transfer process simpler, however there are a few things that do not transfer or that may need additional attention. Imported reports are ALWAYS not active The Authorized Groups list is ALWAYS EMPTY on imported reports Column Extended formats are not modified in any way, so any reference to environment specifics, such as URL strings, may need to be adjusted.

#### jQuery

The auto-complete parameter and data tools section options are implemented using jQuery UI. These features are optional, and jQuery code will only be loaded if these features are used. Techniques are used to try to limit the impact on any other jQuery use in the system, but these are not foolproof. Gwreports loads jquery and initializes the required functions when needed using the include\/gwreports\_ac\_dt.js script file. If the current system supports jQuery UI autocomplete and the dataTables plugin, the script can be modified to eliminate the use and loading of a conflicting version.

