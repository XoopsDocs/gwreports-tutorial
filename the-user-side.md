# The User Side

To see how it works on the User Side, please go and explore the [**Demo**](http://geekwright.com/modules/imblogging/post.php?post_id=11)

To see how it works on the User Side, please go and explore the [**Demo**](http://geekwright.com/modules/imblogging/post.php?post_id=11)


After there was a working **[gwreports](https://sourceforge.net/projects/gwreports/)** module and it was nearing release to the world, the question of how to **[demonstrate it](http://geekwright.com/modules/gwreports/)** came up. It needed a substantial set of data that wasn't proprietary, confidential or otherwise encumbered. After striking out with a google search for "big gobs of data" \(hey. sometimes you get lucky,\) the idea surfaced to look for weather data. That lead rather quickly to the **[Global Historical Climatology Network data base](http://www.ncdc.noaa.gov/ghcnm/v2.php)**. This was a perfect find.The data has very familiar qualities for anyone that has spent time around legacy business data locked in older technologies. The data is in fixed width text files with the instructions for interpreting it expressed in Fortran code. But, too, this is not long forgotten data. Quite the opposite, this is a current and growing collection, invaluable to ongoing climate research. The only thing more one might want is maybe a "big gobs of data" meta tag?This made for a nearly perfect fit for a system that is supposed to be a tool to make data more accessible. Hopefully, this document describing the transformation from a set of text files into the interactive reports of the **[gwreports demo](http://geekwright.com/modules/gwreports/)** will be of value as a case study for those facing similar problems in real world business situations.

**A Plan**

As this was not an open-ended project plan, some base decisions had to be made to limit the scope. After a quick review of the available data, here was the plan outline:

* Focus on precipitation and related meta-data
* Define tables based on existing data model
* Convert data into loadable form
* Load data into MySQL
* Build some reports

OK, it was a bit loose, but it was a plan.

In this plan, these were the files of interest:

* v2.country.codes - countries and numerical country code

* v2.prcp.inv - meta data describing observation stations

* v2.prcp - raw precipitation data

* v2.prcp\_adj - adjusted precipitation data


The v2.prcp.readme indicates the adjusted data file both eliminates some data points and adds some others. This lead to another design decision, merge these two sets for the demo.

For reference, here are brief snippets from the three major files. \(v2.prcp and v2.prcp\_adj are identical in structure.\)

**v2.country.codes**

```text
153 UGANDA                                  
154 ZAIRE                                   
155 ZAMBIA                                  
156 ZIMBABWE                                
157 AMSTERDAM ISLAND (FRANCE)               
158 ASCENSION ISLAND (U.K.)                 
159 CANARY ISLANDS (SPAIN)  
```

v2.prcp.inv

```text
10266390004 QUIHITA             ANGOLA     -15.40   14.00 1310
10266390005 QUILENGUES          ANGOLA     -14.00   14.10  860
10266390006 VILA ARRIAGA        ANGOLA     -14.80   13.20  920
10266410000 SERPA PINTO/MENONGUEANGOLA     -14.70   17.70 1343
10266422000 MOCAMEDES           ANGOLA     -15.20   12.20   45
10266422001 CARACUL                        -14.60   12.40 -999
10266422002 TOMBUA (PORTO ALEXANDRE)       -15.50   11.50   10
```

v2.prcp

```text
1016035500011996  690 1858 2195  285   20   70   10   80  380  386 1187 2617
1016035500011997 1020  130  180  350   80  270    0   30-9999 1900 1740 1480
1016035500011998  820 1040  420  570 1320  110    0  250  810  260 1980  860
1016035500011999 1170 1070  730  320  150   40   10   80  300  250-9999 2380
1016035500012000  610  200  210-9999-9999  430    0   50  180  520  370-9999
1016035500012001 1230  870  110-9999  280    0    0   50-9999   20 1200-9999
1016035500012002  570 1220  210  590  100-8888-9999  310  360  850 2430-9999
```

**Defining tables**

The tables below mimic the file layout of the existing data. The full station code is actually three fields, one of which is the country code. The country code will be important to accessing the data, while the full station code is a key to the precipitation data. They both were needed separately.

```sql
CREATE TABLE IF NOT EXISTS COUNTRY (
  country_code char(3) NOT NULL,
  country_name varchar(255) NOT NULL DEFAULT '',
  PRIMARY KEY (country_code)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

CREATE TABLE IF NOT EXISTS STATION (
  station_code char(11) NOT NULL,
  country_code char(3) NOT NULL,
  station_name varchar(40) NOT NULL DEFAULT '',
  latitude decimal(7,2) NOT NULL DEFAULT 0.0,
  longitude decimal(8,2) NOT NULL DEFAULT 0.0,
  elevation integer(5) NOT NULL DEFAULT 0,
  PRIMARY KEY (station_code),
  UNIQUE KEY (country_code, station_code)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;

CREATE TABLE IF NOT EXISTS PRECIP (
  station_code char(11) NOT NULL,
  record_year char(4) NOT NULL,
  country_code char(3) NOT NULL,
  precip01 integer(5) NOT NULL DEFAULT 0,
  precip02 integer(5) NOT NULL DEFAULT 0,
  precip03 integer(5) NOT NULL DEFAULT 0,
  precip04 integer(5) NOT NULL DEFAULT 0,
  precip05 integer(5) NOT NULL DEFAULT 0,
  precip06 integer(5) NOT NULL DEFAULT 0,
  precip07 integer(5) NOT NULL DEFAULT 0,
  precip08 integer(5) NOT NULL DEFAULT 0,
  precip09 integer(5) NOT NULL DEFAULT 0,
  precip10 integer(5) NOT NULL DEFAULT 0,
  precip11 integer(5) NOT NULL DEFAULT 0,
  precip12 integer(5) NOT NULL DEFAULT 0,
  PRIMARY KEY (station_code, record_year),
  KEY (country_code, record_year)
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

Not beautiful, but it represents the existing data.

**From Files to Database Tables**

There are lots of options for pulling data out of text files. And there are several options for getting data into some MySQL tables. The "quick" part of the initial planning was still a major consideration. This led to a choice of a few lines of [**Awk**](http://www.gnu.org/software/gawk/manual/) code to transform the data as needed, followed by a MySQL LOAD DATA to pull things into the database.

Awk is a terse but powerful tool. The economy of effort for tasks like this more than offsets the initial investment in learning the language. In example, here is the complete program to convert the v2.country.codes file into a file ready to use with LOAD DATA:

```sql
# split v2.country.codes into tab separated fields for load data
BEGIN { OFS = "\t"; ORS = "\n" }
{
    print substr($0,1,3), substr($0,5);
}
```

That's it. When it was saved as countryload.awk the command line to perform the conversion looked like this:

```sql
awk -f countryload.awk < v2.country.codes > country.load
```

The resulting file was then loaded into the database with the following MySQL command:

```sql
LOAD DATA INFILE '/path/to/country.load' REPLACE INTO TABLE COUNTRY;
```

One down, two \(or three\) to go.

Just a note on the REPLACE option. Sometimes, the data files as supplied end with repeated lines. I don't know why, and I'm not too concerned with the reason that is probably buried in some Fortran code. I've used Fortran before, but in this case it doesn't really matter, since someone else pulls the data, we just want to use it. By using the REPLACE option we sidestep any errors on the repeating lines, make it so we automatically merge the two v2.prcp files, and make is so we can set up an automatic refresh with nothing more than rerunning the process. Hmmmm, love that "quick" constraint.

Loading the stations was just a tiny bit more complicated:

```sql
# split v2.prcp.inv into tab separated fields for load data
BEGIN { FIELDWIDTHS = "11 1 30 7 8 5"; OFS = "\t"; ORS = "\n" }
{
    print $1, substr($1,1,3), $3, $4, $5, $6;
}
```

And the precipitation data was similar, just with more columns:

```sql
# split v2.prcp_adj into tab separated fields for load data
BEGIN { FIELDWIDTHS = "11 1 4 5 5 5 5 5 5 5 5 5 5 5 5"; OFS = "\t"; ORS = "\n" }
{
    print $1, $3, substr($1,1,3), $4, $5, $6, $7, $8, $9, $10, $11, $12, $13, $14, $15, $16;
}
```

We need to use this last bit twice, once for v2.prcp, and again for v2.prcp\_adj, and LOAD DATA in that order to capture the adjusted records.

**Got data, now what?**

Populating the tables is just the first step, as the whole point of this exercise was to create a demo, not of precipitation data, but of gwreports. There were three main reports that were obvious:

* A list of Countries
* A list of Stations in a Country
* Precipitation records for a Station

Starting with the Country list, the report SQL looks like this:

```sql
SELECT s.country_code
, c.country_name
, count(*) as station_count
FROM STATION s, COUNTRY c
where c.country_code=s.country_code
and country_name like '{cname}'
group by c.country_name, s.country_code
order by c.country_name
```

Add a parameter definition for _cname_ as Like Text and we have a report ready to run. We can move on to the Stations by Country list:

```sql
select station_code
, country_code
, station_name
, latitude
, longitude
, CASE 
WHEN elevation = -999 THEN 'n/a' 
ELSE elevation
END as elevation
from STATION
where country_code = '{ccode}'
order by station_name, station_code
```

Note here we use the MySQL CASE construct to convert the coded elevation value of -999 \(meaning the elevation is not available\) into something more meaningful for humans.

We can make this a bit fancier by adding another section with country information like this:

```sql
SELECT country_name
, c.country_code
, count(*) as station_count
FROM STATION s, COUNTRY c
where s.country_code = '{ccode}'
and c.country_code = s.country_code
group by country_name, country_code
```

Make that section Multirow=No, and reorder the sections to put it on top. Define a ccode parameter (Text, length of) and we've got another one down.

The Precipitation listing has more CASE magic, and lots of repetition, but is fairly simple. We also convert the tenths of millimeters into millimeters, again for human readability:

```sql
        select record_year
        , CASE 
        WHEN precip01 = -9999 THEN 'n/a' 
        WHEN precip01 = -8888 THEN 'trace' 
        ELSE round((precip01/10),1)
        END as Jan
        , CASE 
        WHEN precip02 = -9999 THEN 'n/a' 
        WHEN precip02 = -8888 THEN 'trace' 
        ELSE round((precip02/10),1)
        END as Feb
        , CASE 
        WHEN precip03 = -9999 THEN 'n/a' 
        WHEN precip03 = -8888 THEN 'trace' 
        ELSE round((precip03/10),1)
        END as Mar
        , CASE 
        WHEN precip04 = -9999 THEN 'n/a' 
        WHEN precip04 = -8888 THEN 'trace' 
        ELSE round((precip04/10),1)
        END as Apr
        , CASE 
        WHEN precip05 = -9999 THEN 'n/a' 
        WHEN precip05 = -8888 THEN 'trace' 
        ELSE round((precip05/10),1)
        END as May
        , CASE 
        WHEN precip06 = -9999 THEN 'n/a' 
        WHEN precip06 = -8888 THEN 'trace' 
        ELSE round((precip06/10),1)
        END as Jun
        , CASE 
        WHEN precip07 = -9999 THEN 'n/a' 
        WHEN precip07 = -8888 THEN 'trace' 
        ELSE round((precip07/10),1)
        END as Jul
        , CASE 
        WHEN precip08 = -9999 THEN 'n/a' 
        WHEN precip08 = -8888 THEN 'trace' 
        ELSE round((precip08/10),1)
        END as Aug
        , CASE 
        WHEN precip09 = -9999 THEN 'n/a' 
        WHEN precip09 = -8888 THEN 'trace' 
        ELSE round((precip09/10),1)
        END as Sep
        , CASE 
        WHEN precip10 = -9999 THEN 'n/a' 
        WHEN precip10 = -8888 THEN 'trace' 
        ELSE round((precip10/10),1)
        END as Oct
        , CASE 
        WHEN precip11 = -9999 THEN 'n/a' 
        WHEN precip11 = -8888 THEN 'trace' 
        ELSE round((precip11/10),1)
        END as Nov
        , CASE 
        WHEN precip12 = -9999 THEN 'n/a' 
        WHEN precip12 = -8888 THEN 'trace' 
        ELSE round((precip12/10),1)
        END as `Dec` 
        , p.country_code
        from PRECIP p, STATION s
        where s.station_code = '{scode}'
        and s.station_code = p.station_code
        order by record_year
```

Of course we need a scode parameter, text, length of 11. And, just for fun, add a Station information section:

```sql
select station_name
, station_code
, s.country_code
, country_name
, latitude, longitude
, CASE 
WHEN elevation = -999 THEN 'n/a' 
ELSE elevation
END as elevation
from STATION s, COUNTRY c
where c.country_code = s.country_code
and station_code = '{scode}'
```

**From Dumb Reports to a System**

The reports "work" at this point. You can look up a country code, plug that into the stations report to look up a station, plug that into the precipitation listing. Uggh! How about we go to the country list, click test, click on the country\_name column header, and enter the following as the Extended format:

```markup
<a href="report_view.php?rid=2&ccode={country_code}">{country_name}</a>
```

OK, we really needed to look up the report id to put after the rid= part, but that really is all there is to linking the country list to the station list. Repeat that concept on the stations listing, adding this to the station\_code column:

```markup
<a href="report_view.php?rid=3&scode={station_code}" title="View Station Records">{station_code}</a>
```

Voila! No more memorizing or cutting and pasting. Just click through from one report to another. Now, ready for the kicker?

**Elapsed Time**

The time from setting out to find "big gobs of data" to having a system of three interlinked reports was one afternoon for one person.Yes, really. "Quick" was a requirement, remember? OK, there were several more tweaks to be made, and after a little bit of playing, a few more reports suggested themselves. There was more effort expended to polish the demo, test it under different circumstances, and find and fix a tiny bug in the process that gave everyone very dry Decembers. But building the proof of concept with live data fit between lunch and dinner. Time for pizza and beer!

**Wrapping Up**

Hopefully, when you put this write-up together with the **[manual](http://geekwright.com/modules/gwreportsmanual/)** and a little bit of quality experimentation time, you will be able to envision more practical uses for this power. The purpose behind **[gwreports](https://sourceforge.net/projects/gwreports/)** was to make data accessible, and we hope this **[demonstration](http://geekwright.com/modules/gwreports/)** proves success in that goal.

