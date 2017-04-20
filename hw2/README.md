# Homework 2: Real-world SQL queries and scalable algorithms

### This is an individual assignment
### Points: 25% of your final grade
### Due: Sunday, 21 May, at 11:55 pm SAST

**This assignment has been originally developed for [UC Berkeley CS186
   course](http://www.cs186berkeley.net/); we use it for COMS4037 with
   their gracious permission**


## Introduction

This assignment is meant to serve two purposes:

1. Consolidate your SQL skills.

2. Acquaint you with using SQL for expressing non-trivial algorithms. 

For this assignment, we will be using the
[SQLite](https://www.sqlite.org/) database engine and Python 2, as
well as the Jupyter notebook interface (for guidance on using Jupyter
notebook, see instructions in
[hw1](https://github.com/WITS-COMS4037/hw/tree/master/hw1)).

When completing the assignment, only add code to the scripting blocks
 indicated with the `--TODO` comment --
 do not modify any other code
 in the source files. 

The data for both parts of the assignment can be downloaded from
[here](http://www.cs.wits.ac.za/~dmitry/coms4037/static/hw2_DATA.zip).

## Part 1: Startup Company Data

We will be working with a
[data-set](https://github.com/WITS-COMS4037/crunchbase-data)
originally obtained from [CrunchBase](https://data.crunchbase.com/).
The data-set, containing information about start-up companies, is made
up of two tables, `companies` and `acquisitions`. The `companies`
table contains information about each start-up's market, status,
funding, and related times and locations. The `acquisitions` table
contains the acquisition amount, times, and locations.

You will complete this part of the assignment by writing SQL code in a
notebook.  The notebook contains the questions which you have to
answer with your SQL queries.

When running your queries, make sure that the database file resides in
your current working directory.  The schema for the database you're
going to be querying is given below. (You could also see the schema by
running `.schema <table_name>` in the `sqlite3` shell.)

```sql
CREATE TABLE companies (
    company_name varchar(200),
    market varchar(200),
    funding_total integer,
    status varchar(20),
    country varchar(10),
    state varchar(10),
    city varchar(30),
    funding_rounds integer,
    founded_at date,
    first_funding_at date,
    last_funding_at date,
    PRIMARY KEY (company_name, market,city)
);

CREATE TABLE acquisitions (
    company_name varchar(200),
    acquirer_name varchar(200),
    acquirer_market varchar(200),
    acquirer_country varchar(10),
    acquirer_state varchar(10),
    acquirer_city varchar(30),
    acquired_at date,
    price_amount integer,
    PRIMARY KEY (company_name, acquirer_name)
);
```

When inspecting the schema, make sure to notice what the primary key
for each table is.  Bear in mind that columns that are not part of the
primary key may contain `NULL` values.

You might want to play with the data in the `sqlite3` shell that comes
with an installation of `sqlite`.

## Part 2: Wikipedia

In this part of the homework, we will be working with
[Wikipedia](https://en.wikipedia.org/wiki/Main_Page) articles.

We will explore the following two questions:
- Does the
  [six degrees of separation](https://en.wikipedia.org/wiki/Six_degrees_of_separation)
  hypothesis apply to Wikipedia?  We will find out on a sample of the
  Wikipedia graph<sup>[1](#degree)</sup>.
  
- If we randomly roam the Wikipedia graph, where do we end up?  We'll
  use the [PageRank](https://en.wikipedia.org/wiki/PageRank) algorithm
  to find out.

Here is the schema of the graph database we will be using for both of
the problems mentioned above.

```sql
CREATE TABLE links (
    src varchar(200) NOT NULL,
    dst varchar(200) NOT NULL,
    PRIMARY KEY (src, dst)
);

CREATE INDEX links_dst_idx ON links(dst);
CREATE INDEX links_src_idx ON links(src);
```

### Degrees of Separation

To find the shortest paths from a given node to all other nodes, we
will use the
[Breadth-First-Search](https://en.wikipedia.org/wiki/Breadth-first_search)
(BFS) algorithm<sup>[2](#research)</sup>.  The basic idea of BFS is to
start at a node, visit all its neighbors, then all the neighbors of
the neighbors, and so on.

The [Wikipedia description of
BFS](https://en.wikipedia.org/wiki/Breadth-first_search) deals with
nodes one at a time via a queue of neighbors, but for an out-of-core,
SQL-based algorithm we will do "batch processing" on sets of paths of
each length: first all paths of length 1, then all paths of length 2,
and so on.

#### Some clarifications:
- If there is no path between nodes `x` and `y`, we ignore the
  pair `(x,y)` for calculating the average paths length.
- The Wikipedia graph is cyclic -- that is, the wiki page for
  [Prof. Hellerstein](https://en.wikipedia.org/wiki/Joseph_M._Hellerstein) could reference
  [Prof. Stonebraker](https://en.wikipedia.org/wiki/Michael_Stonebraker) and vice versa.
  This issue is important because
    1. if we explore cycles, our queries won't terminate;
    2. paths with cycles are longer than non-looping paths and thus
  are irrelevant to our shortest paths computation;  therefore, you
  have to be careful to avoid generating paths with cycles.
- For testing, we will use `part2sampled.db` rather than the full
  `part2.db`, as the computation on the full data-set takes too long. 

#### Pseudocode:

    (links) = edges in our graph
    (paths) = initialise the set of paths to the paths we've seen so
    far, which is equal to the set of links
    (pathsLastUpdated) = paths generated by the following loop:

    repeat:
        (newPaths) = join (pathsLastUpdated) with (links), finding all the new paths that extends paths by one hop
        (pathsLastUpdated) = (newPaths) // to avoiding re-computing
        (paths) += (newPaths) that are distinct and new
        if (paths) table didn't update (no new entries), break out of the loop,
            otherwise, delete (newPaths) and continue

In the pseudocode above we reuse (change the value of) the variables
`(newpaths)` and `(pathsLastUpdated)` on each iteration.  In SQL, these
"variables" are going to be tables.  Thus, you will need to come up
with a way to change the data associated with a table name in SQL.

### PageRank

The idea of PageRank is to capture the likelihood of arriving at a
certain page if we are doing a random walk along a graph. For
instance, if we have a network where all the pages point to one master
page, it is extremely likely that we will end up on the master page.

There are ways to directly solve for the probability using
eigenvectors, but in this example we will solve by simulating the
random walk. This is called the "random surfer" model: for details,
see [PageRank](https://en.wikipedia.org/wiki/PageRank).

For our specific flavour of the naive PageRank algorithm, we make the
following assumptions:
- Since the algorithm converges (i.e. stabilizes to a value), we could start with any value, and then
  update the value based on the edge directions.  In this case, we will start with 1 for all pages.
- The "damping factor" models the likelihood that the "random surfer"
  will continue surfing the web. The number often used is 0.85, which
  intuitively corresponds to saying that 85% of the time people click
  on another link.
- We are not normalizing our PageRank values to sum to 1.

#### Some clarifications:
- The updates should happen in batches; that is, for the current round
  of the update, the value used for all the nodes should come from the
  previous round. This is not a necessary requirement, but we will
  implement such batch behavior in this assignment since (a) it's
  natural in SQL, and (b) it will make testing your output easier.
- If a node has no outgoing links, then according to the "random
  surfer" model, after visiting such a node, all other nodes can be
  clicked on uniformly randomly. *However*, for simplicity, we are not
  going to account for this and will ignore the impact of random links
  from nodes with no original outgoing links.
- To further simplify things, we will not be using convergence as a
  termination condition; instead we will simply stop after a default
  number of iterations, which in our case is 10. 

#### Pseudocode:

    (links) = edges in our graph
    (nodes) = union of distinct sources and destinations in the links table, their outgoing edge
              count and the initial values
    for number of iterations:
        (nodesTemp) = update each node with new pagerank value of 0.15 + 0.85 * (sum of incoming
                      edges' pagerank value normalized by their respective source node's outgoing degree), using
                      values in (nodes)
        (nodes) gets updated by (nodesTemp)


Below is a graph to help visualize the update pattern of our pagerank
algorithm, where the directed graph is on the left, and the table
representation on the right. When updating, we want to iterate through
the node list, and update their values based on the connections and
the respective pagerank value of the sources pointing to the current
node.  ![pagerank_visual](pagerank.png)

Since PageRank is applied to the entire graph, it's a natural fit to
batch-style SQL queries---we hope that you will see this for yourself
once you have done this assignmet.


## Testing and Grading

As part of the [data
archive](http://www.cs.wits.ac.za/~dmitry/coms4037/static/hw2_DATA.zip),
we have provided test data-sets, `part1test.db` and `part2test.db`, as
well as expected output on these data-sets, for you to check your
work. Use the `test.sh` scripts to run the tests.  Each test assumes
that the `csv` files with the expected output are stored in a
particular location -- either check the scripts to see what those
locations are or change the scripts to reflect where you put those
files.  You can also create your own data-sets to check your solution.

Part1 will be worth 28% of this homework's grade, with each question
worth 7% of the final grade.  There will be no partial credits for
each question.

Part2 will be worth 72% of this homework's grade, with degrees of
separation worth 40% and PageRank worth 32% of the final grade.


Additional Notes

- All of the questions should finish within 5 minutes (we will set a time limit of 10
  minutes per question when testing).
- You should *not* use complex operators that we have not mentioned in class, such as user
  defined functions or [`WITH RECURSIVE`](https://www.sqlite.org/lang_with.html) statements.


## Submission instructions
Follow the submission instructions in [HW0](https://github.com/WITS-COMS4037/hw/tree/master/hw0).


## Resources and advice
SQL
- *Tooling*: You may find it easier to play around with the SQLite
directly by entering SQL code on the command line, such as `sqlite3
part1.db`. You may find
[Command Line Shell For SQLite](https://www.sqlite.org/cli.html)
useful.
- *SQL syntax*: You might want to look into
  [aliasing](http://www.postgresql.org/docs/9.2/static/sql-select.html#SQL-FROM)
  to eliminate ambiguity for self-joins.
- *SQL Performance*: There are often many equivalent ways to write a
  query in SQL.  In theory, a database system with a perfect query
  optimizer would produce the best possible run time for all these
  queries. In practice, though, optimizers are not ideal (especially
  in simple systems like SQLite), so the database system sometimes
  relies on you to tell it things it could and could not do to
  optimize for
  performance. [This page](https://www.sqlite.org/cvstrac/wiki?p=PerformanceTuning)
  contains some helpful tips for SQLite. You might find
  [EXPLAIN](https://www.sqlite.org/lang_explain.html) and
  [ANALYZE](https://www.sqlite.org/lang_analyze.html) useful!
- *Idiosyncratic*: SQL implementations are more or less the same but
  the details are different and hard to remember, Google/Stack
  Overflow is your friend here. For example, SQLite
  [does not support JOIN in UPDATEs](http://sqlite.org/lang_update.html)
  and this requires some maneuvering (compared to more sophisticated
  implementations like PostgreSQL).

Jupyter
- *Errors*: When you see the following error message in Jupyter,
    ERROR: An unexpected error occurred while tokenizing input The
    following traceback may be corrupted or invalid The error message
    is: ('EOF in multi-line string', (1, 4)), chances are, you have
    issues with your SQL queries, not Python code. And when you see
    the `--->` that is supposed to point to where the error is
    triggered in your SQL query, it might not be the actual place
    (might be the end of the entire script). You could either rely on
    the error reporting, like `OperationalError: near "blah"`, or
    break the script down into smaller pieces.
- Jupyter sometimes lazily flushes file
  buffers. You might want to look into
  [flush](https://docs.python.org/2/library/stdtypes.html?highlight=file%20flush#file.flush)
  should you suspect this to be happening. However note that this
  won't be too relevant for this assignment (compared to HW1) since we
  are mostly using Python to issue database commands.
- You can keep an interactive database client prompt open and poke
  around at the contents of the tables while they are being populated
  and updated.
- Sometimes when the output seems unexpected, it might be worth
  clearing the 'kernel' by going to the Kernel tab, and click
  *'Restart and Clear Output'*. You should also do this before you git
  commit/push (since the runtime might be large and make your commit
  process and diff logs harder to parse).

Misc
- Should you have accidentally changed any original data, you could
always get a "fresh" copy of the database by running the relevant
cells in your Jupyter notebook.

## Good luck!

<a name="degree">1</a> Caveat: Since we are only sampling the
wikipedia graph, we will not get an accurate answer to this question
-- it turns out that our naive sampling makes the average shortest
paths length shorter than that on the full graph.

<a name="research">2</a> Running BFS on a sample of a graph is a
subroutine in some techniques for scalably approximating shortest
paths in massive graphs as well.
