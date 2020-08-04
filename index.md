
## Old things, Bold things, Oft' untold things

I've been working with Multi-Value Database Systems (MVDB) for long enough that I have opinions about it. For example: If you are building a new application, and think "Oh, that 40 year old database looks like it would be pretty handy!" then just say "no", and google a little harder. If you need a back-end then use SQL, or No-SQL, or SQL-Lite running on a ten year old iPhone, just anything else. Alas, it is unlikely we will be rid of this relic antime soon, since it is so prohibitvely costly to move off legacy systems. And there is nothing wrong with appreciating something old, especially if it is continuing to deliver value.

Principles of good design haven't radically changed in 2 generations of developers. I would argue that thoughtful design was even more critical in older systems, as they had to work within extreme hardware and software constraints. We have more 3rd party libraries, a faster internet, and cheaper silicon to cover up flaws in our thinking. This idea extends beyond even computers, as we have had information storage and retreival systems since the first writing of non-verbal human history. The principles of the system tend to matter more than just the medium you use (scroll, tablet, magnetic tapes, Azure SQL Server, etc.).

## WRITE me a river ...

There are rough SQL equivalents in MVDB (TCL in my case), but they are for data retreival, not writing out data. To write it out, you have to use `PICK` code. For example: 

```
WRITE RECORD TO FILE,ID
```
Pretty dang simple. Each `RECORD` in a `FILE` has a unique `ID`, so this is a pretty simple setup. Plus the record is basically just a serialized array of data, so it's not even like we are talking column specific 





One of the precious few things I truly enjoy about PICK programming for MVDBs more than SQL DBs is the use of the `WRITE` command. Today we would call it `UPSERT`. It still troubles me that SQL systems can't take a page out of a very old book, and create a meaningful default `UPSERT` command. "BUT WAIT!" the SQL nerd in you is screaming. "You can do an `UPSERT` in SQL. It is very well documented. Every SQL has some version of it, and If you have SQL Server just use `MERGE`, it's basically everywhere now and soooo easy:"

```
MERGE <target_table> [AS TARGET]
USING <table_source> [AS SOURCE]
ON <search_condition>
[WHEN MATCHED 
   THEN <merge_matched> ]
[WHEN NOT MATCHED [BY TARGET]
   THEN <merge_not_matched> ]
[WHEN NOT MATCHED BY SOURCE
   THEN <merge_matched> ];
```
 
 Yeah, that is fine, and it accomplishes the same goal, but it isn't really the same thing. The `MERGE USING  ON` / `INSERT INTO ... ON CONFLICT [ conflict_target ] conflict_action` or whatever other version has to be necessarily complicated because it has to support so many different variants because of how much relational databases need a  robust `WHERE` clause. If this were an `INSERT`, you would just add a row. Is there already a duplicate row? Well unless there is a `PRIMARY KEY` constraint on a column, then who cares, right? Except, there is probably a primary key column, so you also need to check for existence and then do an `UDPATE`. Darn, we just invented `UPSERT` again. Plus, what if multiple results came back, did you really want to `UPDATE` all of them? What if you only want to write out some columns, but conditionally dependent on which columns and which data based on the results of the `WHERE` clause. So long as a relational database is trying to solve all these problems, it cannot have a simple implementation. Ambiguity is not an option, unless we are willing to except a quixotic new paradigm that randomly assigns data to a place you didn't ask for, and returns data you don't want. (If anyone wants to write this I will buy them many beers.)
 
 "Just write a stored procedure and quit whining on the internet!" your inner SQL guru says. Look, you're not all together wrong. Yes, I can, but it doesn't seem like I should have to. I want all the complicated options for when I need them, but usually I don't need them. Sometimes you get the feeling that SQL was written for SQL developers, not the "normal people" like me. Forced complexity drives choices, which are taxing on the *developer*, and that complexity can spill over into other layers of the application stack. This is accepted that the developer has to make this choice over and over again, and so a good default isn't chosen.
 
 - The HTTP does this with `POST` vs `PUT`. Do you want to make a new one, or update it? Don't know, well better do a `GET` to find out first, then tell me. Web frameworks accept the paradigmn and you often have to make different controllers/routes for different HTTP verbs
 
 - ORMs try to overcome this. For instance, SQLALchemy has a `.save()` method which *tries* to overcome the SELECT/INSERT/UPDATE overhead, but has a lot of limitations that 



