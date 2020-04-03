![rainbow_csv](https://i.imgur.com/EhV2niB.png)

## Installation
Use your favorite package manager.  

Vundle: `Plugin 'mechatroner/rainbow_csv'`  
VimPlug: `Plug 'mechatroner/rainbow_csv'`  
dein: `call dein#add('mechatroner/rainbow_csv')`  

No additional steps required - Rainbow CSV will work out of the box.  

## Overview
Main features:  
* Highlight CSV columns in different rainbow colors. 
* Provide info about column under the cursor
* Provide _SELECT_ and _UPDATE_ queries in RBQL: SQL-like transprogramming query language.
* Consistency check for csv files (CSVLint)
* Align and Shrink CSV fields (add/remove trailing spaces in fields)

There are 4 ways to enable csv columns highlighting:
1. CSV autodetection based on file content and/or extension  
2. Manual CSV delimiter selection with _:RainbowDelim_ command with cursor over the delimiter  
3. Manual CSV delimiter selection with _:RainbowMultiDelim_ for multi-character delimiters selecting them in "VISUAL" mode  
4. Explicitly activate one of the built-in filetypes, e.g. `:set ft=csv`  

To run an RBQL query either press _F5_ or enter the query in vim command line e.g. _:Select a1, a2_  
As soon as you finish entering ":select" (or ":update") and press whitespace, the plugin will show column names in the status line.  

The core functionality of the plugin is written in pure vimscript, no additional libraries required.  

### Demonstration of rainbow_csv highlighting and RBQL queries 


![demo_screencast](https://i.imgur.com/4PIVvjc.gif)


In this demo python expressions were used, but JavaScript is also available.


# Plugin description

### Built-in and autogenerated filetypes
Rainbow CSV has 7 built-in CSV filetypes and infinite number of autogenerated filetypes.  
Each Rainbow CSV filetype is mapped to a separator and "policy" which describes additional properties e.g. if separator can be escaped inside double quoted field.  
If user uses _:RainbowDelim_ or _:RainbowMultiDelim_ to select a separator that doesn't map to one of the built-in filetypes, Rainbow CSV will dynamically generate the filetype syntax file and put it into the "syntax" folder.  
List of built-in filetypes:  

|Filetype       | Separator     | Extension | Properties                                          |
|---------------|---------------|-----------|-----------------------------------------------------|
|csv            | , (comma)     | .csv      | Ignored inside double-quoted fields                 |
|tsv            | \t (TAB)      | .tsv .tab |                                                     |
|csv_semicolon  | ; (semicolon) |           | Ignored inside double-quoted fields                 |
|csv_whitespace | whitespace    |           | Consecutive whitespaces are merged                  |
|csv_pipe       | &#124; (pipe) |           |                                                     |
|rfc_csv        | , (comma)     |           | Same as "csv" but allows multiline fields           |
|rfc_semicolon  | ; (semicolon) |           | Same as "csv_semicolon" but allows multiline fields |


### Associating file extensions with CSV dialects
In most cases the built-in autodetection algorithm should correctly detect correct CSV dialect for all CSV tables that you open in Vim, but if you have disabled the autodetection algorithm or don't want to rely on it for some reason, you can manually associate file extensions with available csv dialects.  
Example: to associate ".dat" extension with "csv_pipe" dialect and ".csv" extension with "csv_semicolon" add the folowing lines to your .vimrc:  
```
autocmd BufNewFile,BufRead *.csv   set filetype=csv_semicolon
autocmd BufNewFile,BufRead *.dat   set filetype=csv_pipe
```


### Rainbow highlighting for non-table files
You can use rainbow highlighting and RBQL even for non-csv/tsv files.  
E.g. you can highlight records in log files, one-line xmls and other delimited records.  
You can even highlight function arguments in your programming language using comma or comma+whitespaces as a delimiter for _:RainbowDelim_ or _:RainbowMultiDelim_ commands.  
And you can always turn off the rainbow highlighting using _:NoRainbowDelim_ command.  

Here is an example of how to extract some fields from a bunch of uniform single-line xmls:  

![demo_xml_screencast](https://i.imgur.com/HlzBWOV.gif)


### Working with multiline CSV fields
In rare cases some CSV files can contain double-quoted fields spanning multiple lines.  
To work with such files you can set filetype to either "rfc_csv" or "rfc_semicolon".  
Syntax highlighting for rfc_csv and rfc_semicolon dialects can go out of sync with the file content under specific conditions, use `:syntax sync fromstart` command in that case  
rfc_csv and rfc_semicolon are fully supported by RBQL which among other things allows you to easily convert them to line-by-line CSV by replacing newlines in fields with sequences of 4 spaces or something like that.  
rfc_csv and rfc_semicolon take their name from [RFC 4180](https://tools.ietf.org/html/rfc4180) memo with which they are fully compatible.  


### Key Mappings
Key mappings are only assigned for files with active rainbow highlighting

|Key                       | Action                                             |
|--------------------------|----------------------------------------------------|
|**F5**                    | Start query editing for the current csv file       |
|**F5**                    | Execute currently edited query                     |
|**F7**                    | Copy query result set to the parent buffer         |

You can disable these mappings by setting `let g:disable_rainbow_key_mappings = 1`


### Commands

#### :RainbowDelim

Mark current file as a table and highlight it's columns in rainbow colors. Character under the cursor will be used as a delimiter. The delimiter will be saved in a config file for future vim sessions.

You can also use this command for non-csv files, e.g. to highlight function arguments  
in source code in different colors. To return back to original syntax highlighting run _:NoRainbowDelim_

#### :RainbowMultiDelim

Same as _:RainbowDelim_, but works with multicharacter separators.
Visually select the multicharacter separator (e.g. `~#~`) and run _:RainbowMultiDelim_ command.

#### :NoRainbowDelim

Disable rainbow columns highlighting for the current file.

#### :CSVLint

The linter checks the following:  
* consistency of double quotes usage in CSV rows  
* consistency of number of fields per CSV row  

#### :RainbowAlign

Align CSV columns with whitespaces.  
Don't run this command if you treat leading and trailing whitespaces in fields as part of the data.  
You can edit aligned CSV file in Vim column-edit mode (Ctrl+v).  

#### :RainbowShrink

Remove leading and trailing whitespaces from all fields. Opposite to RainbowAlign

#### :Select ...

Allows to enter RBQL select query as vim command.
e.g. _:Select a1, a2 order by a1_

#### :Update ...

Allows to enter RBQL update query as vim command.
e.g. _:Update a1 = a1 + " " + a2_

#### :RainbowName \<name\>

Assign any name to the current table. You can use this name in join operation instead of the table path. E.g.
```
JOIN customers ON a1 == b1
``` 
intead of:
```
JOIN /path/to/my/customers/table ON a1 == b1
```

### Configuration

#### g:disable_rainbow_hover
Set to `1` to stop showing info about the column under the cursor in Vim command line  
Example:  
```
let g:disable_rainbow_hover = 1
```

#### g:disable_rainbow_csv_autodetect
Set to `1` to disable CSV autodetection mechanism  
Example:  
```
let g:disable_rainbow_csv_autodetect = 1
```
Manual delimiter selection would still be possible. 
You can also manually associate specific file extensions with 'csv' or 'tsv' filetypes  

#### g:rcsv_max_columns
Default: _30_

Autodetection will fail if buffer has more than _g:rcsv\_max\_columns_ columns.  
You can increase or decrease this limit.

#### g:rcsv_colorpairs
List of color name pairs to customize rainbow highlighting.  
Each entry in the list is a pair of two colors: the first color is for terminal mode, the second one is for GUI mode.  
Example:
```
let g:rcsv_colorpairs = [['red', 'red'], ['blue', 'blue'], ['green', 'green'], ['magenta', 'magenta'], ['NONE', 'NONE'], ['darkred', 'darkred'], ['darkblue', 'darkblue'], ['darkgreen', 'darkgreen'], ['darkmagenta', 'darkmagenta'], ['darkcyan', 'darkcyan']]
```

#### g:multiline_search_range
Default: _10_  
This settings is only relevant for rfc_csv and rfc_semicolon dialects.  
If some multiline records contain more lines that this value, hover info will not work correctly. It is not recommended to significantly increase this value because it will have negative impact on hover info performance 

#### g:disable_rainbow_key_mappings
Disable default key mappings introduced by the extension  


#### g:rbql_backend_language
Default: _python_
Supported values: 'python', 'js'  

Scripting language to use in RBQL expressions.


#### g:rbql_encoding
Default: _utf-8_
Supported values: 'utf-8', 'latin-1'  

CSV files encoding for RBQL. 


#### g:rbql_output_format
Default: _input_  
Supported values: _tsv_, _csv_, _input_

Format of RBQL result set tables.

* input: same format as the input table
* tsv: doesn't allow quoted tabs inside fields. 
* csv: is Excel-compatible and allows quoted commas.

Essentially format is a pair: delimiter + quoting policy.  
This setting for example can be used to convert files between tsv and csv format:
* To convert _csv_ to _tsv_: **1.** open csv file. **2.** `:let g:rbql_output_format='tsv'` **3.** `:Select *`
* To convert _tsv_ to _csv_: **1.** open tsv file. **2.** `:let g:rbql_output_format='csv'` **3.** `:Select *`


### Optional "Header" file feature

Rainbow csv allows you to create a special "header" file for any of your spreadsheet table files. It must have the same name as the table file but with ".header" suffix (e.g. for "table.tsv" table the header file is "table.tsv.header"). The only purpose of header file is to provide csv column names.


# RBQL (Rainbow Query Language) Description

RBQL is a technology for (not only) CSV files processing. It provides SQL-like language that supports SELECT queries with Python or JavaScript expressions.  
RBQL is distributed with CLI apps, text editor plugins, Python and JS libraries and can work in web browsers.  
RBQL core module is very generic and can process all kind of objects and record formats, but most popular RBQL implementation works with CSV files.  

[Official Site](https://rbql.org/)

### Main Features

* Use Python or JavaScript expressions inside _SELECT_, _UPDATE_, _WHERE_ and _ORDER BY_ statements
* Result set of any query immediately becomes a first-class table on it's own
* Supports input tables with inconsistent number of fields per record
* Output records appear in the same order as in input unless _ORDER BY_ is provided
* Each record has a unique NR (record number) identifier
* Supports all main SQL keywords
* Supports aggregate functions and GROUP BY queries
* Provides some new useful query modes which traditional SQL engines do not have
* Supports both _TOP_ and _LIMIT_ keywords
* Supports user-defined functions (UDF)
* Works out of the box, no external dependencies

#### Limitations:

* RBQL doesn't support nested queries, but they can be emulated with consecutive queries
* Number of tables in all JOIN queries is always 2 (input table and join table), use consecutive queries to join 3 or more tables

### Supported SQL Keywords (Keywords are case insensitive)

* SELECT
* UPDATE
* WHERE
* ORDER BY ... [ DESC | ASC ]
* [ LEFT | INNER ] JOIN
* DISTINCT
* GROUP BY
* TOP _N_
* LIMIT _N_

All keywords have the same meaning as in SQL queries. You can check them [online](https://www.w3schools.com/sql/default.asp)  


### RBQL variables
RBQL for CSV files provides the following variables which you can use in your queries:

* _a1_, _a2_,..., _a{N}_  
   Variable type: **string**  
   Description: value of i-th field in the current record in input table  
* _b1_, _b2_,..., _b{N}_  
   Variable type: **string**  
   Description: value of i-th field in the current record in join table B  
* _NR_  
   Variable type: **integer**  
   Description: Record number (1-based)  
* _NF_  
   Variable type: **integer**  
   Description: Number of fields in the current record  
* _a.name_, _b.Person_age_, ... _a.{Good_alphanumeric_column_name}_  
   Variable type: **string**  
   Description: Value of the field referenced by it's "name". You can use this notation if the field in the first (header) CSV line has a "good" alphanumeric name  
* _a["object id"]_, _a['9.12341234']_, _b["%$ !! 10 20"]_ ... _a["Arbitrary column name!"]_  
   Variable type: **string**  
   Description: Value of the field referenced by it's "name". You can use this notation to reference fields by arbitrary values in the first (header) CSV line, even when there is no header at all  


#### Notes:
* You can mix all variable types in a single query, example:
  ```select a1, b2 JOIN /path/to/b.csv ON a['Item Id'] == b.Identifier WHERE NR > 1 and int(a.Weight) * 100 > int(b["weight of the item"])```
* Referencing fields by header names does not automatically skip the header line (you can use `where NR > 1` trick to skip it)
* If you want to use RBQL as a library for your own app you can define your own custom variables and do not have to support the above mentioned CSV-related variables.


### UPDATE statement

_UPDATE_ query produces a new table where original values are replaced according to the UPDATE expression, so it can also be considered a special type of SELECT query. This prevents accidental data loss from poorly written queries.  
_UPDATE SET_ is synonym to _UPDATE_, because in RBQL there is no need to specify the source table.  


### Aggregate functions and queries

RBQL supports the following aggregate functions, which can also be used with _GROUP BY_ keyword:  
_COUNT_, _ARRAY_AGG_, _MIN_, _MAX_, _SUM_, _AVG_, _VARIANCE_, _MEDIAN_  

Limitation: aggregate functions inside Python (or JS) expressions are not supported. Although you can use expressions inside aggregate functions.  
E.g. `MAX(float(a1) / 1000)` - valid; `MAX(a1) / 1000` - invalid.  
There is a workaround for the limitation above for _ARRAY_AGG_ function which supports an optional parameter - a callback function that can do something with the aggregated array. Example:  
`select a2, ARRAY_AGG(a1, lambda v: sorted(v)[:5]) group by a2` - Python; `select a2, ARRAY_AGG(a1, v => v.sort().slice(0, 5)) group by a2` - JS


### JOIN statements

Join table B can be referenced either by it's file path or by it's name - an arbitary string which user should provide before executing the JOIN query.  
RBQL supports _STRICT LEFT JOIN_ which is like _LEFT JOIN_, but generates an error if any key in left table "A" doesn't have exactly one matching key in the right table "B".  
Limitation: _JOIN_ statements can't contain Python/JS expressions and must have the following form: _<JOIN\_KEYWORD> (/path/to/table.tsv | table_name ) ON a... == b... [AND a... == b... [AND ... ]]_


### SELECT EXCEPT statement

SELECT EXCEPT can be used to select everything except specific columns. E.g. to select everything but columns 2 and 4, run: `SELECT * EXCEPT a2, a4`  
Traditional SQL engines do not support this query mode.


### UNNEST() operator
UNNEST(list) takes a list/array as an argument and repeats the output record multiple times - one time for each value from the list argument.  
Example: `SELECT a1, UNNEST(a2.split(';'))`  


### LIKE() function
RBQL does not support LIKE operator, instead it provides "like()" function which can be used like this:
`SELECT * where like(a1, 'foo%bar')`


### User Defined Functions (UDF)

RBQL supports User Defined Functions  
You can define custom functions and/or import libraries in two special files:  
* `~/.rbql_init_source.py` - for Python
* `~/.rbql_init_source.js` - for JavaScript


## Examples of RBQL queries

#### With Python expressions

* `select top 100 a1, int(a2) * 10, len(a4) where a1 == "Buy" order by int(a2) desc`
* `select * order by random.random() where NR > 1` - skip header record and random sort
* `select len(a.vehicle_price) / 10, a2 where NR > 1 and a['Vehicle type'] in ["car", "plane", "boat"] limit 20` - referencing columns by names from header record, skipping the header and using Python's "in" to emulate SQL's "in"
* `update set a3 = 'NPC' where a3.find('Non-playable character') != -1`
* `select NR, *` - enumerate records, NR is 1-based
* `select * where re.match(".*ab.*", a1) is not None` - select entries where first column has "ab" pattern
* `select a1, b1, b2 inner join ./countries.txt on a2 == b1 order by a1, a3` - example of join query
* `select MAX(a1), MIN(a1) where a.Name != 'John' group by a2, a3` - example of aggregate query
* `select *a1.split(':')` - Using Python3 unpack operator to split one column into many. Do not try this with other SQL engines!

#### With JavaScript expressions

* `select top 100 a1, a2 * 10, a4.length where a1 == "Buy" order by parseInt(a2) desc`
* `select * order by Math.random() where NR > 1` - skip header record and random sort
* `select top 20 a.vehicle_price.length / 10, a2 where NR > 1 and ["car", "plane", "boat"].indexOf(a['Vehicle type']) > -1 limit 20` - referencing columns by names from header record and skipping the header
* `update set a3 = 'NPC' where a3.indexOf('Non-playable character') != -1`
* `select NR, *` - enumerate records, NR is 1-based
* `select a1, b1, b2 inner join ./countries.txt on a2 == b1 order by a1, a3` - example of join query
* `select MAX(a1), MIN(a1) where a.Name != 'John' group by a2, a3` - example of aggregate query
* `select ...a1.split(':')` - Using JS "destructuring assignment" syntax to split one column into many. Do not try this with other SQL engines!


### FAQ

#### How do I skip header record in CSV files?

You can use the following trick: add `... where NR > 1 ...` to your query  

And if you are doing math operation you can modify your query like this, example:  
`select int(a3) * 1000, a2` -> `select int(a3) * 1000 if NR > 1 else a3, a2`  


#### How does RBQL work?

RBQL parses SQL-like user query, generates new Python or JavaScript code and executes it.  

Explanation of simplified Python version of RBQL algorithm by example.
1. User enters the following query, which is stored as a string _Q_:
```
    SELECT a3, int(a4) + 100, len(a2) WHERE a1 != 'SELL'
```
2. RBQL replaces all `a{i}` substrings in the query string _Q_ with `a[{i - 1}]` substrings. The result is the following string:
```
    Q = "SELECT a[2], int(a[3]) + 100, len(a[1]) WHERE a[0] != 'SELL'"
```

3. RBQL searches for "SELECT" and "WHERE" keywords in the query string _Q_, throws the keywords away, and puts everything after these keywords into two variables _S_ - select part and _W_ - where part, so we will get:
```
    S = "a[2], int(a[3]) + 100, len(a[1])"
    W = "a[0] != 'SELL'"
```

4. RBQL has static template script which looks like this:
```
    for line in sys.stdin:
        a = line.rstrip('\n').split(',')
        if %%%W_Expression%%%:
            out_fields = [%%%S_Expression%%%]
            print ','.join([str(v) for v in out_fields])
```

5. RBQL replaces `%%%W_Expression%%%` with _W_ and `%%%S_Expression%%%` with _S_ so we get the following script:
```
    for line in sys.stdin:
        a = line.rstrip('\n').split(',')
        if a[0] != 'SELL':
            out_fields = [a[2], int(a[3]) + 100, len(a[1])]
            print ','.join([str(v) for v in out_fields])
```

6. RBQL runs the patched script against user's data file (real RBQL implementation calls "exec" in Python or "eval" in JS): 
```
    ./tmp_script.py < data.tsv > result.tsv
```
Result set of the original query (`SELECT a3, int(a4) + 100, len(a2) WHERE a1 != 'SELL'`) is in the "result.tsv" file.  
Adding support of TOP/LIMIT keywords is trivial and to support "ORDER BY" we can introduce an intermediate array.  




# General info

### Comparison of Rainbow CSV technology with traditional graphical column alignment

#### Advantages

* WYSIWYG  
* Familiar editing environment of your favorite text editor  
* Zero-cost abstraction: Syntax highlighting is essentially free, while graphical column alignment can be computationally expensive  
* High information density: Rainbow CSV shows more data per screen because it doesn't insert column-aligning whitespaces.
* Works with non-table and semi-tabular files (text files that contain both table(s) and non-table data like text)
* Ability to visually associate two same-colored columns from two different windows. This is not possible with graphical column alignment  

#### Disadvantages

* Rainbow CSV technology may be less effective for CSV files with many (> 10) columns
* Current Rainbow CSV implementations do not support newlines inside double-quoted csv fields. Adding multiline fields support is technically possible under certain conditions but would impair other Rainbow CSV features and advantages.

### References


#### Rainbow CSV and similar plugins in other editors:

* Rainbow CSV extension in [Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=mechatroner.rainbow-csv)
* rainbow-csv package in [Atom](https://atom.io/packages/rainbow-csv)
* rainbow_csv plugin in [Sublime Text](https://packagecontrol.io/packages/rainbow_csv)
* rainbow_csv plugin in [gedit](https://github.com/mechatroner/gtk_gedit_rainbow_csv) - doesn't support quoted commas in csv
* rainbow_csv_4_nedit in [NEdit](https://github.com/DmitTrix/rainbow_csv_4_nedit)
* CSV highlighting in [Nano](https://github.com/scopatz/nanorc)
* Rainbow CSV in [IntelliJ IDEA](https://plugins.jetbrains.com/plugin/12896-rainbow-csv/)

#### RBQL

* RBQL website [RBQL](https://rbql.org/)
* Library and CLI App for JavaScript [RBQL](https://www.npmjs.com/package/rbql)  
* Library and CLI App for Python [RBQL](https://pypi.org/project/rbql/)  

#### Related vim plugins:
Rainbow CSV name and original implementation was significantly influenced by [rainbow_parentheses](https://github.com/kien/rainbow_parentheses.vim) Vim plugin.  

There also exists an old vim syntax file [csv_color](https://vim.sourceforge.io/scripts/script.php?script_id=518) which, despite it's name, can highlight only *.tsv files.  
And, of course, there is [csv.vim](https://github.com/chrisbra/csv.vim)
