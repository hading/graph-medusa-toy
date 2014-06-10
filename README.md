This is a toy project to envision very roughly how a graph database based medusa might look, at least in part.

As such, the content files are basically trivial, metadata files don't necessarily conform to the schemas that
you would expect them to, etc. The point is to show how we might deal with the real thing in a graph database
context. The content is fairly regular, as this is a toy, but has some roughness to show how that can work.

The /content directory contains this information.

There will then be a series of tasks/scripts/whatever to show how we might progressively deal with this content.
These should be interpreted as examples, of course, and not comprehensive. E.g. I doubt that I'm going to get
to the point of adding events and so forth, but it is certainly possible. Moreover a lot of it may be obvious
fakery, e.g. characterizing files by their extension and so on. Again, doing this part of the job for real
is not the point here.

Run ```rake -T``` to see the neo4j tasks, including installation, starting/stopping the server, and clearing
the database.

Run rake tasks in order to build up the structure:

* create_collections
* bit_ingest_collections
* characterize_file_assets
* structure_books
* structure_images
* migrate_image_assets

See the individual rake tasks for more information about what each is supposed to illustrate.

A rake task create_whole_database will run all of the above in order (or at least the ones that are currently
implemented).

As you go along you can use the neo4j-shell or the web console at http://localhost:7474 to play with things,
run Cypher queries, etc.

I am not yet an expert (or beyond rank beginner) in neo4j, so don't assume that I'm doing anything well.

Here's a cypher query that will get at the bit level structure. Try it in the browser:

    match (c:Collection)-[rd:HAS_BIT_ROOT_DIRECTORY]->(r:Directory)-[:HAS_SUBDIRECTORY*]->(d:Directory)-[:HAS_FILE_ASSET]->(f:FileAsset) return rd, r, d, f

Here's one to see groups by mime type after you've characterized the files:

    match (r:MimeType)<-[:HAS_MIME_TYPE]-(f) return r, f

Once the books are structured you might try the following to see the structure of the whole book collection:

    match (c:Collection {name: 'Books'})-[r*]->(n) return c,r, n

That's a bit complex. To see the two page three objects for the Dogs book:

    match (c:BookCollection)-[:HAS_BOOK]->(b:Book{name: 'Dogs'})-[:HAS_PAGE {page: 3}]->(p) return c,b,p

Those last two don't show up great in the web browser because there are nodes with multiple relationships
(both :HAS_PAGE and :HAS_MASTER_FILE, for example)

Let's say we want to find all of the pages in book that don't have an ocr file. We could do these two queries
and take the difference (I don't know how to do this in one step currently):

    Match (b:Book {name:'Dogs'})-[r:HAS_MASTER_FILE]->(p)<-[q:HAS_PAGE]-(b) return b.name,collect(q.page)
    Match (b:Book {name:'Dogs'})-[r:HAS_OCR_FILE]->(p)<-[q:HAS_PAGE]-(b) return b.name,collect(q.page)

Okay, here's how to do it in one query:

    match (p)<-[r:HAS_PAGE]-(b:Book {name: 'Dogs'})-[q:HAS_MASTER_FILE]->(p)
    with r,  b
    optional match (b)-[q:HAS_OCR_FILE]->(s:FileAsset)<-[t:HAS_PAGE {page: r.page}]->(b)
    with r,s where s is null
    return collect(r.page)

The key here is that putting a where clause with a match doesn't post filter the match, it puts a constraint on the match.
To filter after the match is done you need to use where in the with clause.