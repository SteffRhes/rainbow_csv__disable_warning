## Overview
Rainbow CSV has 2 main features:
* Highlights csv columns in different rainbow colors. 
* Allows to run "SELECT" and "UPDATE" queries in RBQL: SQL-like transprogramming query language.

There are 2 ways to enable csv columns highlighting:
1. CSV autodetection based on file content. File extension doesn't have to be .csv or .tsv
2. Manual CSV delimiter selection with `:RainbowDelim` command (So you can use it even for non-csv files, e.g. to highlight function arguments in different colors)

To run an RBQL query either press 'F5' or enter the query in vim command line e.g. `:Select a1, a2`

### Demonstration of rainbow_csv highlighting and RBQL queries 
1-st query with Python expressions and 2-nd query with JavaScript:


![demo_screencast](https://raw.githubusercontent.com/mechatroner/rainbow_csv/master/demo/rbql_demo_2.gif)


The demo table is demo/movies.tsv. There are also some other test datasets in python/test_datasets.


# RBQL (RainBow Query Language) Description
RBQL is a technology which provides SQL-like language that supports "SELECT" and "UPDATE" queries with Python or JavaScript expressions.

### Main Features
* Use Python or Java Script expressions inside "select", "update", "where" and "order by" statements
* Output entries appear in the same order as in input unless "ORDER BY" is provided.
* Input csv/tsv table may contain varying number of entries (but select query must be written in a way that prevents output of missing values)
* Unicode support

### Supported SQL Keywords (Keywords are case insensitive)

* SELECT \[ TOP _N_ \] \[ DISTINCT [ COUNT ] \]
* UPDATE \[ SET \]
* WHERE
* ORDER BY ... [ DESC | ASC ]
* [ [ STRICT ] LEFT | INNER ] JOIN

#### Keywords rules
All keywords have the same meaning as in SQL queries. You can check them online e.g. here: https://www.w3schools.com/sql/default.asp
But there are also two new keywords: `DISTINCT COUNT` and `STRICT LEFT JOIN`:
* `DISTINCT COUNT` is like `DISTINCT`, but adds a new column to the "distinct" result set: number of occurences of the entry, similar to `uniq -c` unix command.
* `STRICT LEFT JOIN` is like `LEFT JOIN`, but generates an error if any key in left table "A" doesn't have exactly one matching key in the right table "B".

Some other rules:
* `UPDATE SET` is synonym to `UPDATE`, because in RBQL there is no need to specify the source table.
* `UPDATE` has the same semantic as in SQL, but it is actually a special type of `SELECT` query.
* `JOIN` statements must have the following form: `<join_keyword> /path/to/table.tsv on ai == bj`

### Special variables
* `*` - Current record
* `a1`, `a2`, ... , `aN` _(string)_ - Value of i-th column
* `b1`, `b2`, ... , `bN` _(string)_ - Value of i-th column in right hand side table B in JOIN operations
* `NR` _(integer)_ - Number of Records being processed (or line number). NR is 1-based
* `NF` _(integer)_ - Number of Fields in the current line/record

### Examples of RBQL queries

#### With Python expressions

* `select top 100 a1, int(a2) * 10, len(a4) where a1 == "Buy" order by int(a2)`
* `select * order by random.random()` - random sort, this is an equivalent of bash command "sort -R"

#### With JavaScript expressions

* `select top 100 a1, a2 * 10, a4.length where a1 == "Buy" order by parseInt(a2)`
* `select * order by Math.random()` - random sort, this is an equivalent of bash command "sort -R"

# Plugin description

### Mappings

|Key           | Action                                                      |
|--------------|-------------------------------------------------------------|
|`<leader>d`   | Print info about current column (under the cursor)          |
|`F5`          | Start query editing for the current csv file                |
|`F5`          | Execute currently edited query                              |


### Commands

#### :Select ...

Allows to enter RBQL select query in vim command line.
The query must start with `:Select` command e.g. `:Select a1, a2 order by a1`

#### :Update ...

Allows to enter RBQL update query in vim command line.
The query must start with `:Update` command e.g. `:Update a1 = a1 + " " + a2`

#### :RainbowDelim

Mark current file as csv and highlight columns in rainbow colors. Character
under the cursor will be used as a delimiter. The delimiter will be saved in a
config file for future vim sessions.

You can also use this command for non-csv files, e.g. to highlight function arguments
in source code in different colors. To return back to original syntax highlighting run `:NoRainbowDelim`

#### :NoRainbowDelim

This command will disable rainbow columns highlighting for the current file.
Useful when autodection mechanism has failed and marked non-csv file as csv.


### Configuration

#### g:rbql_meta_language
*Default: 'python'*
Scripting language to use in RBQL expression. Either 'js' or 'python'
To use JavaScript add `let g:rbql_meta_language = 'js'` to .vimrc

#### g:rcsv_delimiters
*Default: [	,]*
By default plugin checks only TAB and comma characters during autodetection stage.
You can override this variable to autodetect tables with other separators. e.g. `let g:rcsv_delimiters = [	;:,]`

#### g:disable_rainbow_csv_autodetect
csv autodetection mechanism can be disabled by setting this variable value to 1.
Manual delimiter selection would still be possible.

#### g:rcsv_max_columns
*Default: 30*
Autodetection will fail if buffer has more than `g:rcsv_max_columns` columns.
You can increase or decrease this limit.


### Optional "Header" file feature

Rainbow csv allows you to create a special "header" file for your table files. It should have the same name as the table file but with ".header" suffix (e.g. for "input.tsv" the header file is "input.tsv.header"). The only purpose of header file is to provide csv column names for `:RbGetColumn` command.


### Installation

Install with your favorite plugin manager.

If you want to use RBQL with JavaScript expressions, make sure you have Node.js installed


# Other

### How does it work?
Python module rbql.py parses RBQL query, creates a new python worker module, then imports and executes it.

### Some more examples of RBQL queries:

#### With Python expressions

* `select a1, a2 where a2 in ["car", "plane", "boat"]` - use Python's "in" to emulate SQL's "in"
* `update set a3 = 'United States' where a3.find('of America') != -1`
* `select * where NR <= 10` - this is an equivalent of bash command "head -n 10", NR is 1-based')
* `select a1, a4` - this is an equivalent of bash command "cut -f 1,4"
* `select * order by int(a2) desc` - this is an equivalent of bash command "sort -k2,2 -r -n"
* `select NR, *` - enumerate lines, NR is 1-based
* `select * where re.match(".*ab.*", a1) is not None` - select entries where first column has "ab" pattern
* `select a1, b1, b2 inner join ./countries.txt on a2 == b1 order by a1` - an example of join query

#### With JavaScript expressions

* `select a1, a2 where ["car", "plane", "boat"].indexOf(a2) > -1`
* `update set a3 = 'United States' where a3.indexOf('of America') != -1`
* `select * where NR <= 10` - this is an equivalent of bash command "head -n 10", NR is 1-based')
* `select a1, a4` - this is an equivalent of bash command "cut -f 1,4"
* `select * order by parseInt(a2) desc` - this is an equivalent of bash command "sort -k2,2 -r -n"
* `select * order by Math.random()` - random sort, this is an equivalent of bash command "sort -R"
* `select NR, *` - enumerate lines, NR is 1-based
* `select a1, b1, b2 inner join ./countries.txt on a2 == b1 order by a1` - an example of join query


### cli_rbql.py script

rainbow_csv comes with cli_rbql.py script which is located in ~/.vim extension folder.  
You can use it in standalone mode to execute RBQL queries from command line. Example:
```
./cli_rbql.py --query "select a1, a2 order by a1" < input.tsv
```
To find out more about cli_rbql.py and available options, execute:
```
./cli_rbql.py -h
```



