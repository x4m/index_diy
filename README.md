# Index DIY
Supplementary materials for my PGCon 2020 talk "Index DIY".
Until May 27th this file is WIP. I'm still working on contents.

If you have a question and want a detailed answer - please create Issue in this repository. Of course, I will answer all questions from IRC and live QA Zoom session too.

For large files I used [Yandex.Disk shared folder](https://yadi.sk/d/z9ZbSmp8mM1YSA).
There you can find slides in pptx and pdf formats, some referenced papers and other stuff.

## About this talk
```A database index is a data structure that improves the speed of data retrieval operations on a database table at the cost of additional writes and storage space to maintain the index data structure.```
[Wikipedia](https://en.wikipedia.org/wiki/Database_index)
![Entities](img/entities.png)

Note: The CREATE INDEX command is not a part of the ANSI SQL standard, and thus its syntax varies among vendors.

Official documentation is the source of truth. Docs for pluggable index access methods can be found here https://www.postgresql.org/docs/current/indexam.html

Thread about cache prefetches https://www.postgresql.org/message-id/flat/3B774C9E-01E8-46A7-9642-7830DC1108F1%40yandex-team.ru
