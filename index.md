
## Old things, Bold things, Oft' untold things

I've been working with [Multi-Value Database Systems](https://en.wikipedia.org/wiki/Rocket_U2) (MVDB) for just long enough that I feel entitled to opinions. For example: If you are building a new application, and think "Oh, that 40+ year old database looks like it would be pretty handy!" then just say "no", and google a little harder. If you need a back-end then use SQL, or No-SQL, or SQL-Lite running on a ten year old iPhone, just anything else. Alas, it is unlikely we will be rid of this relic antime soon, since it is so prohibitvely costly to move off legacy systems. And there is nothing wrong with appreciating something old, especially if it is continuing to deliver value.

Many principles of good design haven't radically changed in 2 generations of developers. I would argue that thoughtful design was even more critical in older systems, as they had to work within extreme hardware and software constraints. We have more 3rd party libraries, a faster internet, and cheaper silicon to cover up our flaws. This idea extends beyond even computers, as we have had information storage and retreival systems since the first writing of non-verbal human history. The principles of the system tend to matter more than just the medium you use (scroll, clay tablet, magnetic tapes, Azure SQL Server, etc.).

## Ask me no questions, I'll udpate no files

Let's do a thought experiment. (FUN!) You have a form to drop off at an office. You approach the desk, hand the form to the smiling office manager and ask them to process it for you. They open up a drawer with a folder, find the spot with your name, and see that something is already there, the old form that had your OLD address and needs to be updated. 

```
"Sorry", you are told "there is already a form on file." The office manager the proceeds to light the form on fire and drop it in the trash. 

"Why did you do that?" you ask, outraged. 

"There was alreadya a form on file" goes the reply. 

"Why didn't you just replace the old file with the new one?" 

"Because you filled out the new address form, not the updated address form. Otherwise you could have filled out the removal of original address form." 

"But how was I supposed to know there was an old form to remove?" 

"That would be the access your own data form." 
```

By the time you understand the shenanigans and the form is finally filed, you are hoping you can just as quickly unlearn the ways of the beaurocratic craziness. This, dear reader, is the learning curve for modern databse development. The database assumes you know what it contains, even if you put it there. It does exactly what you tell it to do, not always what you want it to do. CRUD can sometimes be pretty cruddy. 

But, it doesn't have to be this way. What if it went like this:
```
"Here is my form."

"OK, great, it is now on file."

"You don't need me to tell you how to file it?"

"Nope, that's my job, don't worry about it."

"Really?"

"Yep. There is just one form, and it is pretty short, so that's it. Are you OK?"

You back away slowly and leave before more questions arise.

```

This unexpected miracle, is what it is like to use a MVDB. You `READ`, you `WRITE`, maybe you `DELETE`, and you are DONE.

## WRITE me a river ...

There are rough equivalents to SQL for MVDBs ([TCL](https://www3.rocketsoftware.com/rocketd3/support/documentation/d3nt/91/refman/tcl/tcl_commands.htm) in my case), but they are for data retreival, not writing out data. To write it out, you have to use [`PICK`](https://en.wikipedia.org/wiki/Pick_operating_system) code. For example: 

```
WRITE RECORD TO FILE,ID
```
Pretty dang simple. Each `RECORD` in a `FILE` has a unique `ID`, and that is all there is to it. Plus, the record is basically just a serialized array, so it's not even like we are talking columns or data types. If you need it back you just:

```
READ RECORD FROM FILE,ID ELSE RECORD=''
```

One of the precious few things I truly enjoy about PICK programming for MVDBs more than SQL DBs is beautiful simplicity in the use of the `WRITE` command. Today we call it `UPSERT`. It still troubles me that SQL systems can't take a page out of a very old book, and create a meaningfully simple syntax for an `UPSERT` command. 

"BUT WAIT!" the SQL nerd in you is screaming. "You CAN do an `UPSERT` in SQL! It is [very well documented](https://www.mssqltips.com/sqlservertip/1704/using-merge-in-sql-server-to-insert-update-and-delete-at-the-same-time/). [Every SQL implementation has some version of it](https://blog.usejournal.com/update-insert-upsert-multiple-records-in-different-db-types-63aa44191884), and since you have MS SQL Server just use [`MERGE`](https://docs.microsoft.com/en-us/sql/t-sql/statements/merge-transact-sql?view=sql-server-ver15), it's basically everywhere now and soooo easy:"

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
 
 Yeah, that is fine, and it accomplishes the same goal, but it isn't exactly the same thing. The `MERGE USING  ON` / `INSERT INTO ... ON CONFLICT DO UPDATE` or whatever other version has to be necessarily complicated. By its nature it has to support a  robust `WHERE` clause, and havesome variant of a `SWITCH` statement to handle the possible outputs. Trying to keep that in mind just leads to dead ends when thinking of alternatives. If this were an `INSERT`, you would just add a row. Is there already a duplicate row? Well, unless there is a `UNIQUE KEY` constraint on a column, then who cares, right? Except, there is probably a primary key with `UNIQUE` on the column, so you also need to check for existence and then do an `UDPATE`. Darn, we just invented `UPSERT` again. Plus, what if multiple results came back, did you really want to `UPDATE` all of them? What if you only want to write out some columns, or only some data, and each is dependent on a different outcome of the `WHERE` clause? So long as a relational database is trying to solve all these problems, it cannot have a simple implementation. You have to have some kind of `IF INSERT ELSE UPDATE`. Ambiguity is not an option, unless we are willing to accept a quixotic new paradigm that randomly assigns data to a place you didn't ask for, and returns data you don't want. (If someone built this please contact me.)
 
The other thing that can be frustrating is that the syntax of these `UPSERT`s are written to `SELECT` from one table into another. You can have as many nested `SELECT` statements as you want! But what if 80% of the time you just want to `UPSERT` one record into a table? Well, just `SELECT` the data you want to `UPSERT` in the query you build. I suppose, that is the SQL way to do it, but seems so unlike how I think about data in the rest of my coding. I have an object, and I want to write it to the DB, it just *feels* weird to have to turn it into a table, select it, and `MERGE` it into another one. This is especially true for Object Oriented paradigms, where there is often a 1:1 relationship between a table and an object. Even foreign key relationships turn into references to related objects. Heck, the schemas are even built from OOP code paradigmns in many ORMs ([Ruby on Rails Migrations](https://guides.rubyonrails.org/active_record_migrations.html), [Alembic](https://alembic.sqlalchemy.org/en/latest/tutorial.html), [Django Migrations](https://docs.djangoproject.com/en/3.0/topics/migrations/ ), etc.) and the table is basically a reflection of the class!

```"Just write a stored procedure for the syntax you want and quit whining on the internet!" your inner SQL guru says.```

Look, you're not all together wrong. Yes, I can, but it doesn't seem like I should have to. I want all the complicated options for when I need them, but usually I don't need them. Sometimes you get the feeling that SQL was written for SQL developers, not the "normal people" like me. Forced complexity drives choices, which are taxing on the *developer*, and that complexity can spill over into other layers of the application stack. This is accepted that the developer has to make this choice over and over again. 
 
 - The [HTTP](https://tools.ietf.org/html/rfc2616#section-9.5) does this with `POST` vs `PUT`. Do you want to make a new one, or update it? Don't know, well better do a `GET` to find out first, then tell me. Web frameworks accept the paradigmn and you often have to make different controllers/routes for different HTTP verbs that do ALMOST the same thing, and probably share most of the same code.
 
 - ORMs try to overcome this in variety of ways through even more complex code interactions, but ultimately require the developer to decide what to do in each case of an [`INSERT` or `UPDATE`.](https://stackoverflow.com/questions/41724658/how-to-do-a-proper-upsert-using-sqlalchemy-on-postgresql)
 
 ## NO SQL for you!
 
```"Just ditch SQL and do No-SQL if you've got so many problems."```

But, that would make me sad. :( I like SQL. I like foreign keys, multiple primary keys, and complex data relationships. I like triggers and stored procedures, constraints and row level security. I suppose that grass is looking a little greener. MongoDB has an [`.upsert()`](https://docs.mongodb.com/manual/reference/method/Bulk.find.upsert/) method built in, and Redis has [`.set()`](https://redis.io/commands/set). But, I've rarely had an app where No-SQL didn't have it's own problems when trying to create increasingly intricate data models with lots of interactions. I've usually seen them as more of a supplement to my architecture to run micro-services with a specific function.

Ultimately this comes down to issues of abstractions. Every time we create structures for thinking about our data we are losing nuance. Sometimes that is good, sometimes that is bad. It is possible to create very robust systems that can perform many different and complex operations which also have good defaults set. It is OK to add a thin abstraction on top of the complexity. It's there when you need it, but not unless you want it. This is very akin to [Nudge Theory](https://en.wikipedia.org/wiki/Nudge_theory). If you have good pre-sets for a system, and it is not costly to the user to override them, then you are making it the right kind of easy. Google used this as a [design principle for Android](https://stackoverflow.com/questions/27707609/is-it-common-practice-to-ask-users-whether-they-wish-to-receive-push-notificatio). Some pople even thought [this could work when filing taxes](https://www.propublica.org/article/congress-is-about-to-ban-the-government-from-offering-free-online-tax-filing-thank-turbotax). 

## Dare to Dream

OK, put on your imagination hat with me. What if SQL stopped trying to be so agnostic, and created a more explicity `UPSERT` syntax that didn't require so many decision points? If you have a single table, which has at least one primary key, and you just want to `UPSERT` one (or more) rows, then you don't need to create an `IF/ELSE` structure. You just do like the cave dwelling engineers who built our mainframes did, you just do a `WRITE` and walk away.

This could look look identical to an insert:

```
UPSERT INTO table(col1,col2,col3)
VALUES(val1, val2, val3)
```

The database knows what the primary keys are. It knows if there is not a primary key on the table, or if it is missing from the row being `UPSERT`ed. It can throw all the errors it needs to throw just like if any other normal constraints were unmet. If there are columns that already have data and aren't in the new row, then they should get left in place. All of this is possible because, like in the antiquated Record/File system, there is necessarily a `UNIQUE` constraint on the table. If it doesn't have one, then it should throw an error. Is this really better? Meh... Maybe not. I enjoy this as an idea, but the cost is an increasingly asburd number of assumptions. 

I am especially picking on MSSQL here, but I think [Postgres is similar](https://wiki.postgresql.org/wiki/UPSERT). Some good news is that MySQL has an implementation that almost hits this mark using [`REPLACE INTO`](https://dev.mysql.com/doc/refman/8.0/en/replace.html), [but internally does a DELETE which can be very costly, plus you lose data in non-primary key columns you didn't `UPDATE`](http://code.openark.org/blog/mysql/replace-into-think-twice). SQLite has [`INSERT OR REPLACE`](https://stackoverflow.com/questions/418898/sqlite-upsert-not-insert-or-replace?rq=1), which is maybe syntactically better but still has struggles with ambiguity, so now has `INSERT.. ON CONFLICT`. It all comes back to how much ambiguity is tolerable, and if you need to write an `ELSE` statement to resolve it, just write the `ELSE` statement. 

Nobody is asking me to design SQL standards, and that shows great wisdom on their part. I am a developer who uses SQL as a means to an end, not a DB engineer who thinks SQL is equal to any other Turing complete language. I am asking something new to be like something old. But, like I said, older here isn't better. If all you have is `READ`/`WRITE`/`DELETE` then your world is very small. You can only build so high if you only have 3 tools, and this is why it is on it's way out. SQL is what it needs to be, and I am the one who will conform to it. Giving up simplicity means I, like every other developer, have to be of two minds when it comes to data modeling, and accept the complexity that comes with it.


