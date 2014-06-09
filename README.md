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

A rake task create_whole_database will run all of the above in order.

As you go along you can use the neo4j-shell or the web console at http://localhost:7474 to play with things,
run Cyber queries, etc.