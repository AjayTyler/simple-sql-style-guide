# Simple SQL Style Guide

## Rules of Thumb

1. Write for clarity
2. Write for readability
3. Write for sanity

Make things easy on future you or anyone who comes after you: write simply, format sensibly, and comment generously.

Also, _**NEVER**_ use a query builder. The code those things produce violate all three rules of thumb.

## Why This Style Guide?

You have a variety of options available to you when it comes to picking or developing a style guide for your SQL code. If you take a look at a few, such as those listed below, you will notice that some agreement exists between them, but that they will often directly contradict one another (such as whether or not to line up keywords in certain ways, capitalization patterns to embrace, etc.).

**Companies**

- [GitLab](https://about.gitlab.com/handbook/business-technology/data-team/platform/sql-style-guide/)
- [Kickstarter](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
- [Mozilla](https://docs.telemetry.mozilla.org/concepts/sql_style.html)

**Individuals**

- [Ben Brumm](https://www.databasestar.com/sql-best-practices/)
- [John McCall](https://tsqlstyle.lowlydba.com/)
- [Matt Mazur](https://github.com/mattm/sql-style-guide)
- [Simon Holywell](https://www.sqlstyle.guide/)


At the end of the day, absent any formal adoption of policy, most folks on a team are going to write how they are going to write. So, this is less of a de facto guide and more of a suggestion. Code that falls within this range of formats is code that I find easiest to work with. The further afield that code goes from this range of formats makes me less inclined to want to interact with it.

The formats described here are not particularly rigid. They are simply the observations I've collected while reading other people's code, maintaining my own, and taking note of what seeme to make things easier or more difficult (and what, truly, did not matter much in the long run).

# Layout

## Summary

- Use soft tabs, not hard
- Set your tab size to 2 or 4 (your pick)
- Favor tall and narrow over short and wide
- Don't re-use table aliases (re-using column aliases is fine)
- When aliasing, be explicit rather than implicit
- Avoid subqueries; use window functions instead
- `SELECT`:
    - List each column on its own line, indented
    - When more than one table is queried, include the alias in the column selection (e.g. `table1.column1`)
- `FROM` / `JOIN`:
  - **Never** use implicit join syntax (i.e. placing join criteria in the `WHERE` clause)
  - List each source on its own line with a meaningful, succinct alias
  - List each join criteria on its own line beneath the table, indented
- `WHERE`: When multiple criteria are used, list indented beneath, each on their own line
- `GROUP BY`: Same style as with SELECT

## Editor Setup

Before you start writing your query, make sure that the text editor that you are using is set to use **soft tabs** rather than hard. Soft tabs work better because the visual indent level of a tab might differ depending on the text editor being used. Some text editors assume that your tab size is the equivalent of 8 spaces, others assume 4. If you copy-paste code from other sources, you might end up with a combination of hard and soft tabs, so be wary of that and clean up anything that you paste into your script.

Next, pick your preferred tab size. A tab size of 4 is a safe pick. Personally, I use a tab size of 2 because a lot of my queries involve nesting a few levels deep, so it helps to prevent the code from getting so long that it requires horizontally scrolling.

There are tons of options out there for text editors; most all are good. If you've not shopped around much for one, check out these three for a start. You'll figure out your preferences soon enough.

1. Visual Studio Code
2. Sublime Text
3. Notepad++

## Style Samples

> Write no less than necessary.
> 
> Type as little as required.

### General Format

Format your code so that it is tall and narrow rather than short and wide. This makes it much easier to compare queries side-by-side and generally improves readability. So, be generous with your vertical white space and stingy with the horizontal.

For simple queries, layout is a non-issue. You could put it all on one line without any readability issues.

```sql

SELECT id, fname, lname FROM Customers as cust

```

You could even tack on a join with an aggregation and let the initial SELECT-FROM live on one line.

```sql

SELECT cust.id, fname, lname, SUM(order_value) as tot_cust_val FROM Customers as cust
LEFT JOIN Orders as ord ON cust.id = ord.customer_id
GROUP BY cust.id, fname, lname

```

However, once we start adding some more meaningful column aliases, it begins to run a bit long.

```sql

SELECT cust.id as customer_id, fname as first_name, lname as last_name, SUM(order_value) as tot_cust_val FROM Customers as cust
LEFT JOIN Orders as ord ON cust.id = ord.customer_id
GROUP BY cust.id, fname, lname

```

I recommend the following format:

- `SELECT` statements with columns listed on their own lines, indented one level
- `FROM` / `JOIN` on their own line and aliased with a meaningful name that's unique within the query
- Explicitly list the join type (`INNER JOIN`, `LEFT JOIN`, `RIGHT JOIN`, `FULL OUTER JOIN`)
- Joins have a short, meaningful alias
- Indent the join criteria below the join clause
- Format the `GROUP BY` criteria in the same way as the select statement

This takes up more vertical room. For a short query like this, it may seem like overkill. However, for long queries, it makes the query much easier to read, navigate, and maintain.

```sql

SELECT
  cust.id as customer_id,
  cust.fname as first_name,
  cust.lname as last_name,
  SUM(ord.order_value) as customer_lifetime_value

FROM Customers as cust

LEFT JOIN Orders as ord
  ON cust.id = ord.customer_id

GROUP BY
  cust.id,
  cust.fname,
  cust.lname

```

On the note of maintenance, I'd recommend that you avoid the instinct to try to align things perfectly. It looks nice, but it just makes maintenance a painful chore in the event that you need to update something later, change things around, or add more complex logic. Only try for picture-perfect alignment if you're formatting a textbook or something. It's not worth it, especially if someone other than you needs to come in at a later point.

```sql

-- It looks nice, but it's a pain to do this while you're developing a query.
-- If you have to add in a new line that's longer than the rest, then you have
-- to go back and adjust everything else, which is wasted time. Just don't do it.
SELECT
    cust.id                      as customer_id,
    cust.fname                   as first_name,
    cust.lname                   as last_name,
    SUM(ord.order_value)         as customer_lifetime_value

FROM Database.Schema.Customers   as cust

LEFT JOIN Database.Schema.Orders as ord
    ON cust.id = ord.customer_id

GROUP BY
    cust.id,
    cust.fname,
    cust.lname

```

### Commas

Some people prefer to put commas at the beginning of a line. I did that for a while; I kept forgetting to put them at the end of a line, so it was nice to just put them up front where I wouldn't forget about it. Eventually, I went back to putting commas at the end of my lines just so things would left-align nicely without requiring an additional space from time to time. Also, I didn't like the way it looked with commas at the start.

However, that is a loose recommendation. If your code is clear, readable, and easy for a newcomer to understand well enough and make a change, you're already ahead of the game where it counts. Put your commas where you want them to go; just be consistent.

```sql

-- Example of comma-first lines
SELECT
   cust.id as customer_id
  ,cust.fname as first_name
  ,cust.lname as last_name
  ,SUM(ord.order_value) as customer_lifetime_value

FROM Database.Schema.Customers as cust

LEFT JOIN Database.Schema.Orders as ord
  ON cust.id = ord.customer_id

GROUP BY
   cust.id
  ,cust.fname
  ,cust.lname
  
```

### Aliases

When aliasing a column, join, or subquery, it pays to make the alias:

- Explicit (instead of implicit)
- Meaningful (rather than arbitrary)
- Succinct (rather than repetitive)

#### Explicit - Alias Keyword

Let's take our dense, three-line aliased query from before and remove the `AS` keyword (making the alias declarations implicit). I don't know about you, but certainly find it to be a bit more of a chore to take in at a glance.

```sql

SELECT cust.id customer_id, fname first_name, lname last_name, SUM(order_value) tot_cust_val FROM Customers cust
LEFT JOIN Orders ord ON cust.id = ord.customer_id
GROUP BY cust.id, fname, lname

```

Of course, breaking it out with more generous white space and explicitly prefixing each column with its parent table's alias helps quite a bit.

```sql

-- The implicit aliases are easier to recognize, now, but
-- it still isn't ideal.
SELECT
  cust.id customer_id,
  cust.fname first_name,
  cust.lname last_name,
  SUM(ord.order_value) customer_lifetime_value

FROM Customers cust

LEFT JOIN Orders ord
  ON cust.id = ord.customer_id

GROUP BY
  cust.id,
  cust.fname,
  cust.lname
  
```

It'd still be easier to visually parse this thing if we could just look for `as` and let the syntax highlighting help us out a bit more. For joins, it's especially useful to have the alias explicitly declared since syntax highlighting will make it easier to spot. In short queries like this example, it doesn't seem like much. But when you're wading through hundreds of lines of code, every little bit helps.

Additionally, when working in especially large queries, it's nice to be able to hit `ctrl + f` and have the `as` keyword as something to search for in conjunction with a table alias. It's much easier to search for a table that's been aliased `ea` if we can search for `as ea`, otherwise we'll get a lot of false positives for such a common combination of letters.

```sql

-- Now we have a better visual indicator of where aliases
-- are being declared.
SELECT
  cust.id as customer_id,
  cust.fname as first_name,
  cust.lname as last_name,
  SUM(ord.order_value) as customer_lifetime_value

FROM Customers as cust

LEFT JOIN Orders as ord
  ON cust.id = ord.customer_id

GROUP BY
  cust.id,
  cust.fname,
  cust.lname
  
```

Bottom line: when you alias something (whether columns or tables), be explicit and use the `as` keyword.

#### Explicit - Field Source

When you have more than one table referenced in a query, **explicitly list the source in the column selection**. Sometimes, this is required in order for the SQL to execute: if the column name exists in multiple tables, you'll get an error about an ambiguous column reference.

However, even if it lets you execute the query, a human trying to read the code may get frustrated when trying to figure out where each field comes from. Even short queries can be a headache to parse.

```sql

SELECT
  id, -- is this the order ID? Customer ID?
  status, -- is this a customer's status? Order status?
  value, -- is this the order value? Something else?
  billing_state -- is this tied to the customer? The order itself?

FROM Database.Schema.Customers as cust

LEFT JOIN Database.Schema.Orders as ord
  ON cust.id = ord.customer_id

LEFT JOIN Database.Schema.Interactions_Agg as ntr
  ON cust.id = ntr.customer_id

```
So, whenever more than one table is being referenced, be sure to keep the reference as part of the column selection.

```sql

-- This is explicit; the source for each field is self-evident.
SELECT
  cust.id,
  cust.status,
  ntr.value,
  ord.billing_state

FROM Database.Schema.Customers as cust

LEFT JOIN Database.Schema.Orders as ord
  ON cust.id = ord.customer_id

LEFT JOIN Database.Schema.Interactions_Agg as ntr
  ON cust.id = ntr.customer_id

```

#### Meaningful

Perhaps you've seen queries that have joins that look like this:

```sql

SELECT
  A.id as customer_id,
  C.status as rewards_program_status,
  COUNT(DISTINCT B.order_id) as lifetime_orders,
  SUM(B.order_value) as lifetime_sales

FROM Database.Schema.Customers as A

LEFT JOIN Database.Schema.Orders as B
  ON A.id = B.customer_id

LEFT JOIN Database.Schema.Rewards_Program as C
  ON A.id = C.customer_id

GROUP BY
  A.id,
  C.status

```

Using arbitrary aliases keeps things short, and you don't have to think about a name or abbreviation. However, it becomes a massive nuisance when you try to orient yourself to a script that's been written this way, or when you must re-orient yourself after not looking at it for a while.

Arbitrary aliases require you have to memorize what letter maps to which table, and you have no hints that an abbreviation would provide. You also end up scrolling up and down, reading the column that's called from the table, and then searching for the alias so that you can find out which table it's coming from.

Additionally, it's also helpful to keep your aliases unique within a query (i.e. it's best not to use the same alias inside and outside of a subquery). Otherwise, it can get a little tricky to remember what's being referenced where.

#### Succinct

When making an alias for a table, it helps to make it succinct--no more than three or four characters. This helps cut down on line length for your code, allows you the opportunity to provide a more meaningful alias for tables with somewhat cryptic names, and overall helps to reduce the density of your query.

```sql

/* Table Name Repeated */

-- This doesn't look too bad, but real-life table names are seldom
-- so human friendly, and look more like 'tbl_orc_cust' which leads
-- to much denser code which becomes a headache to read.
SELECT
  customers.id as customer_id,
  customers.customer_type as customer_type,
  program.status as rewards_program_status,
  COUNT(DISTINCT orders.order_id) as lifetime_orders,
  SUM(orders.order_value) as lifetime_sales,

  SUM(
    CASE
      WHEN
        orders.discount_code is not NULL
        AND customers.customer_type = 'Key Account'
      THEN orders.order_value
      ELSE 0
    END
  ) as lifetime_discounted_key_account_sales

FROM Database.Schema.Customers as customers

LEFT JOIN Database.Schema.Orders as orders
  ON customers.id = orders.customer_id

LEFT JOIN Database.Schema.Rewards_Program as program
  ON customers.id = program.customer_id

WHERE customers.billing_state in ('KS', 'TX')
  AND orders.category = 'Combi-Ovens'
  AND program.status in ('Active', 'Target')

GROUP BY
  customers.id,
  program.status


/* Short Table Alias */

-- This, meanwhile, is easier to read at a glance.
SELECT
  cust.id as customer_id,
  prog.status as rewards_program_status,
  COUNT(DISTINCT ord.order_id) as lifetime_orders,
  SUM(ord.order_value) as lifetime_sales,

  SUM(
    CASE
      WHEN
        ord.discount_code is not NULL
        AND cust.customer_type = 'Key Account'
      THEN ord.order_value
      ELSE 0
    END
  ) as lifetime_discounted_key_account_sales

FROM Database.Schema.Customers as cust

LEFT JOIN Database.Schema.Orders as ord
  ON cust.id = ord.customer_id

LEFT JOIN Database.Schema.Rewards_Program as prog
  ON cust.id = prog.customer_id

WHERE cust.billing_state in ('KS', 'TX')
  AND ord.category = 'Combi-Ovens'
  AND prog.status in ('Active', 'Target')

GROUP BY
  cust.id,
  prog.status

```

### Commenting

- Good comments are more important than how you lay them out
- Put comments near the thing you're talking about
- If you do something clever or difficult, comment to explain it

I recommend using the multi-line comments to create the equivalent of headings in largers queries, and single line comments for explanations.

### Capitalize Keywords According to Convenience

When it comes to capitalization, I recommend going with whatever promotes personal convenience. In my opinion:

- All lowercase: fine
- UPPERCASE functions and keywords: fine
- And also UPPERCASE table names: fine
- EVERYTHING UPPERCASE: A PAIN IN MY EYE

Operate according to readability: what will make it easier for you to find things in the future? My own preference for ultimate readability is:

- UPPERCASE clauses and functions
- Initial Caps for tables and views (with the exception of capitalizing acronyms)
- lower_snake_case for everything else

However, for long and complex queries, I often just lowercase everything simply because I can type it more easily (less use of the `shift` key). Or, if I'm in a rush trying to knock something out, I'll probably just type it however I see fit in the moment.

My only real recommendation is not to go ALLCAPS on column names. There are enough options to capitalize other things; going that route just makes it all run together.

**lowercase everything**

```sql

select
  cust.id as customer_id,
  cust.customer_group as customer_marketing_group,
  ord.cart_size as items_in_cart,
  ord.order_value as total_order_value,
  ord.trans_date as order_date

from database.schema.orders as ord

inner join database.schema.customers as cust
  on ord.customer_id = cust.id

where year(ord.trans_date) = year(current_date())

```

**UPPERCASE Clauses and Functions, Initial Caps Tables and Views**

```sql

SELECT
  cust.id as customer_id,
  cust.customer_group as customer_marketing_group,
  ord.cart_size as items_in_cart,
  ord.order_value as total_order_value,
  ord.trans_date as order_date

FROM Database.Schema.Orders as ord

INNER JOIN Database.Schema.Customers as cust
  on ord.customer_id = cust.id

WHERE YEAR(ord.trans_date) = YEAR(CURRENT_DATE())

```

**lowercase everything except for Tables / Views**

```sql

select
  cust.id as customer_id,
  cust.customer_group as customer_marketing_group,
  ord.cart_size as items_in_cart,
  ord.order_value as total_order_value,
  ord.trans_date as order_date

from DATABASE.SCHEMA.ORDERS as ord

inner join DATABASE.SCHEMA.CUSTOMERS as cust
  on ord.customer_id = cust.id

where year(ord.trans_date) = year(current_date())

```

### Joins

Do **_not_** use implicit joins. They are of the devil. Modern syntax allows for a much more sustainable, modular encapsulation of join criteria and filtering logic into separate concerns.

The implicit join syntax comes from a time before `JOIN` clauses, and there is no need to use such anachronistic chicanery. The only reason to know about them is for `UPDATE` statements in various SQL implementations, or to translate from legacy code.

```sql

-- An example of what NOT to do when joining tables.

SELECT
  cust.id as customer_id,
  program.status as rewards_program_status,
  COUNT(DISTINCT orders.order_id) as lifetime_orders,
  SUM(orders.order_value) as lifetime_sales

FROM
  Database.Schema.Customers as cust,
  Database.Schema.Orders as ord,
  Database.Schema.Rewards_Program as prog,
  Database.Schema.Campaigns as camp


-- This is gross. Links between tables are mixed in with
-- filtering logic, which makes it difficult to see what
-- connects to what and in what ways. Plus, it defaults
-- to an inner join unless you include (+) in the logic,
-- which is not an intuitive syntax.
WHERE cust.id = ord.customer_id(+)
  AND cust.id = prog.customer_id(+)
  AND ord.campaign_id = camp.id(+)
  AND camp.status = 'Active'
  AND cust.billing_state = 'TX'
  AND prog.status in ('Active', 'Targeted')

GROUP BY
  cust.id,
  program.status

```

When joining tables, list the join critera as an indented separate line. Don't include them on the same line, as it tends to leads to horizontal scrolling, which becomes a pain to manage. Plus, for non-trivial joins that require more complex logic for the join, it leads to a mess.

```sql

-- Another example of what I do not recommended.

SELECT
  cust.id as customer_id,
  prog.status as rewards_program_status,
  COUNT(DISTINCT ord.order_id) as lifetime_orders,
  SUM(ord.order_value) as lifetime_sales

FROM Database.Schema.Customers as cust

-- Don't do this. Please.
LEFT JOIN Database.Schema.Orders as ord ON cust.id = ord.customer_id

LEFT JOIN Database.Schema.Rewards_Program as prog ON cust.id = prog.customer_id

WHERE cust.billing_state in ('KS', 'TX')
  AND ord.category = 'Combi-Ovens'
  AND prog.status = 'Active'

GROUP BY
  cust.id,
  program.status

```
