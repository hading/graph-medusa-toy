This is a toy project to envision very roughly how a graph database based medusa might look, at least in part.

As such, the content files are basically trivial, metadata files don't necessarily conform to the schemas that
you would expect them to, etc. The point is to show how we might deal with the real thing in a graph database
context. The content is fairly regular, as this is a toy, but has some roughness to show how that can work.

The /content directory contains this information.

There will then be a series of tasks/scripts/whatever to show how we might progressively deal with this content.
These should be interpreted as examples, of course, and not comprehensive. E.g. I doubt that I'm going to get
to the point of adding events and so forth, but it is certainly possible.

Run ```rake -T``` to see the neo4j tasks, including installation, starting/stopping the server, and clearing
the database.


