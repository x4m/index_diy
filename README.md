# Index DIY
Supplementary materials for my PGCon 2020 talk "Index DIY".

If you have a question and want a detailed answer - please create Issue in this repository. Of course, I will answer all questions from IRC and live QA Zoom session too.

For large files I used [Yandex.Disk shared folder](https://yadi.sk/d/z9ZbSmp8mM1YSA).
There you can find slides in pptx and pdf formats, some referenced papers and other stuff.

## About this talk
This talk is about forking Index Access Methods from PostgreSQL core.

```A database index is a data structure that improves the speed of data retrieval operations on a database table at the cost of additional writes and storage space to maintain the index data structure.```
[Wikipedia](https://en.wikipedia.org/wiki/Database_index)

![Entities](img/entities.png)

Index Access Method is implementation of idea how to search.
PostgreSQL has a lot of core index access methods:
Method | Idea | Docs
--- | --- | ---
B-tree | Search among sorted objects | [docs](https://www.postgresql.org/docs/current/btree.html)
GiST \ SP-GiST | Descending from group with generic features to specific | [docs](https://www.postgresql.org/docs/current/gist.html)
GIN | Searching object by its part | [docs](https://www.postgresql.org/docs/current/gin-intro.html)
Hash | Reducing search region to objects with same hash | [docs](https://www.postgresql.org/docs/current/indexes-types.html)
BRIN | Digesting groups of data to skip them during sequential search | [docs](https://www.postgresql.org/docs/current/brin-intro.html)
Bloom | Digesting all data to skip search for non-existent values | [docs](https://www.postgresql.org/docs/current/bloom.html)

High modularity of many Index Access Methods like GiST allows to fork them into extension.

Official documentation is the source of truth. Docs for pluggable index access methods can be found here https://www.postgresql.org/docs/current/indexam.html

I'm not discussing [operator classes](https://www.postgresql.org/docs/current/sql-createopclass.html) in this talk. If ideas of core indexes suits your search well, probably, you should start with development of operator class.

Thread about cache prefetches https://www.postgresql.org/message-id/flat/3B774C9E-01E8-46A7-9642-7830DC1108F1%40yandex-team.ru

Interface of what index can and must do is well described in [amapi.h](https://github.com/postgres/postgres/blob/master/src/include/access/amapi.h)

```
/*
 * API struct for an index AM.
 */
typedef struct IndexAmRoutine
{
	....

	/*
	 * Total number of strategies (operators) by which we can traverse/search
	 * this AM.  Zero if AM does not have a fixed set of strategy assignments.
	 */
	uint16		amstrategies;
	/* total number of support functions that this AM uses */
	uint16		amsupport;
	/* opclass options support function number or 0 */
	uint16		amoptsprocnum;
	/* does AM support ORDER BY indexed column's value? Only B-tree */
	bool		amcanorder;
	/* does AM support ORDER BY result of an operator on indexed column? Only GiST and SP-GiST */
	bool		amcanorderbyop;
	/* does AM support backward scanning? Only B-tree and Hash */
	bool		amcanbackward;
	/* does AM support UNIQUE indexes? Only B-tree */
	bool		amcanunique;
	/* does AM support multi-column indexes? */
	bool		amcanmulticol;
	/* does AM require scans to have a constraint on the first index column? */
	bool		amoptionalkey;
	/* does AM handle ScalarArrayOpExpr quals? */
	bool		amsearcharray;
	/* does AM handle IS NULL/IS NOT NULL quals? */
	bool		amsearchnulls;
	/* can index storage data type differ from column data type? */
	bool		amstorage;
	/* can an index of this type be clustered on? Only GiST and B-tree */
	bool		amclusterable;
	/* does AM handle predicate locks? */
	bool		ampredlocks;
	/* does AM support parallel scan? */
	bool		amcanparallel;
	/* does AM support columns included with clause INCLUDE? */
	bool		amcaninclude;
	/* does AM use maintenance_work_mem? */
	bool		amusemaintenanceworkmem;
	/* OR of parallel vacuum flags.  See vacuum.h for flags. */
	uint8		amparallelvacuumoptions;
	/* type of data stored in index, or InvalidOid if variable */
	Oid			amkeytype;

	...
	/* interface functions */
	ambuild_function ambuild;
	ambuildempty_function ambuildempty;
	aminsert_function aminsert;
	ambulkdelete_function ambulkdelete;
	amvacuumcleanup_function amvacuumcleanup;
	amcanreturn_function amcanreturn;	/* can be NULL */
	amcostestimate_function amcostestimate;
	amoptions_function amoptions;
	amproperty_function amproperty; /* can be NULL */
	ambuildphasename_function ambuildphasename; /* can be NULL */
	amvalidate_function amvalidate;
	ambeginscan_function ambeginscan;
	amrescan_function amrescan;
	amgettuple_function amgettuple; /* can be NULL */
	amgetbitmap_function amgetbitmap;	/* can be NULL */
	amendscan_function amendscan;
	ammarkpos_function ammarkpos;	/* can be NULL */
	amrestrpos_function amrestrpos; /* can be NULL */

	/* interface functions to support parallel index scans */
	amestimateparallelscan_function amestimateparallelscan; /* can be NULL */
	aminitparallelscan_function aminitparallelscan; /* can be NULL */
	amparallelrescan_function amparallelrescan; /* can be NULL */
} IndexAmRoutine;
```

### Generic WAL limitations
I did not say it clearly in talk. The only place where index-as-extension can be slower than core index is WAL usage.
Core indexes use hand-crafted WAL write function. In generic WAL, instead, developer only points system to a buffer that was changed. And it's responsibility of a system to determine which bytes of a block changed. That's the main source of inefficiency.

### Learned Indexes
Learned Indexes is an interesting idea to generate searching data structure with machine learning. Unfortunately, proof of concept lacks many things to be an index in OLTP database. OLTP database is about changing data, with high level of concurrency and strict isolation guarantees.
