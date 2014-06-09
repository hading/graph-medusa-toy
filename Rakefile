require 'neography'
require 'neography/tasks'

desc "Create our initial collections"
task :create_collections do
  db = Neography::Rest.new
  %w(Books Images).each do |collection_name|
    #Make a node with the Collection label and supplied name
    db.execute_query(%Q(
      MERGE (collection:Collection {name: '#{collection_name}'})
    ))
  end
end

desc "Ingest the sample collections at the bit level, i.e. directory structure"
task :bit_ingest_collections do
  db = Neography::Rest.new
  %w(Books Images).each do |collection_name|
    root_dir = File.join('.', 'content', collection_name.downcase)
    #make an associate a root directory node, storing the full path to the root directory
    full_path = File.absolute_path(root_dir)
    #Create the root node and return ID for later use
    result = db.execute_query(%Q(
      MATCH (c:Collection {name: '#{collection_name}'})
      CREATE UNIQUE (c)-[:HAS_BIT_ROOT_DIRECTORY]->(d:Directory {path: '#{full_path}'})
      RETURN ID(d)
                     ))
    #Find the root node. Then we pass the current path and node to a recursive function,
    #building the structure using the traversal API, which seems more natural here

    #This is a bit awkward; I bet it's possible to do better somehow
    root_node_id = result['data'][0][0]
    root_node = db.get_node(root_node_id)
    bit_ingest(full_path, root_node, db)
  end
end

def bit_ingest(full_path, directory_node, db)
  directory_entries = (Dir[File.join(full_path, '*')] + Dir[File.join(full_path, '.*')]).reject { |entry| %w(. ..).include?(File.basename(entry)) }
  subdirectories = directory_entries.select { |entry| File.directory?(entry) }
  files = directory_entries.select { |entry| File.file?(entry) }
  #Attach file nodes
  files.each do |file|
    puts "Ingesting #{file}"
    file_node = db.create_node("name" => File.basename(file))
    db.add_label(file_node, "FileAsset")
    db.create_relationship("HAS_FILE_ASSET", directory_node, file_node)
  end
  #attach directory nodes, recursively ingesting to them, depth first
  subdirectories.each do |subdirectory|
    puts "Ingesting #{subdirectory}"
    subdirectory_node = db.create_node('path' => subdirectory)
    db.add_label(subdirectory_node, 'Directory')
    db.create_relationship('HAS_SUBDIRECTORY', directory_node, subdirectory_node)
    bit_ingest(subdirectory, subdirectory_node, db)
  end
end

desc "Determine a mime type and size for each file asset"
task :characterize_file_assets do
  db = Neography::Rest.new
  #For this example we go over each file asset. If it doesn't have a size then we find its size and store in
  #the node. If it doesn't have an associated mime type then we find its mime type, create a MimeType node
  #if we need to, and then associate it to the mime type.
  #According to how we select the files we could do this collection by collection, etc. Many ways with better control.
  #But I'm just trying to do it simply here.
  nodes = db.get_nodes_labeled('FileAsset').collect { |node| Neography::Node.load(node['self'], db) }
  nodes.each do |node|
    directory = node.incoming(:HAS_FILE_ASSET).first
    file_path = File.join(directory.path, node.name)
    unless node.size
      node.size = File.size(file_path)
    end
    mime_type_node = node.outgoing(:HAS_MIME_TYPE).first
    unless mime_type_node
      mime_type = find_mime_type(node.name)
      mime_type_node = ensure_mime_type_node(mime_type, db)
      node.outgoing(:HAS_MIME_TYPE) << mime_type_node
    end
  end
end

MIME_TYPE_HASH = {'.txt' => 'text/plain',
                  '.pdf' => 'application/pdf',
                  '.jpg' => 'image/jpeg',
                  '.tiff' => 'image/tiff',
                  '.xml' => 'application.xml'}

def find_mime_type(file_name)
  MIME_TYPE_HASH[File.extname(file_name)] || 'application/octet-stream'
end

def ensure_mime_type_node(mime_type, db)
  result = db.execute_query(%Q(
      MERGE (mime_type:MimeType {name: '#{mime_type}'}) RETURN ID(mime_type)
    ))
  puts result
  return Neography::Node.load(result['data'][0][0], db)
end

desc "Add graph structure to the book objects, e.g. pages, table of contents and index as appropriate, ocr vs. master, metadata"
task :structure_books do
  #Here we would do some analysis and might determine the following:
  #Under the books collection, each root directory represents a book with the title given by the directory name
  #In each book directory there is a metadata directory that may have dublin core and/or mods
  #In each book directory the contents are directly in that directory and may be in either or both .txt and .pdf form
  #For our example, .pdf are masters and .txt are ocr
  #In each book directory there may be a table of contents and/or index represented by toc.xyz and index.xyz
  #In each book directory there are pages which have the convention page_num.xyz
  #The following is one way we might encode the above in the database
  #The basic strategy is to create a book object under the collection. This will then have pages, metadata, etc.
  #as described. I'm not trying to get the best model here, just showing how we can relate things back to some model.
  #There are a variety of other ways one might model this.
  #The nice thing to notice here is that we are able to create our model directly by creating relationships in the
  #database. So it coexists with our bit-level model.
  db = Neography::Rest.new
  result = db.execute_query(%Q(
      MATCH (books:Collection {name: 'Books'}) RETURN ID(books)
    ))
  collection_node = Neography::Node.load(result['data'][0][0], db)
  db.add_label(collection_node, "BookCollection")
  book_directory_nodes = collection_node.outgoing(:HAS_BIT_ROOT_DIRECTORY).first.outgoing(:HAS_SUBDIRECTORY).to_a
  book_directory_nodes.each do |book_directory_node|
    book_node = Neography::Node.create('name' => File.basename(book_directory_node.path).capitalize)
    collection_node.outgoing(:HAS_BOOK) << book_node
    db.add_label(book_node, "Book")
    metadata_directory_node = book_directory_node.outgoing(:HAS_SUBDIRECTORY).detect { |node| File.basename(node.path) == 'metadata' }
    if dc_node = metadata_directory_node.outgoing(:HAS_FILE).detect { |node| node.name == 'dc.xml' }
      book_node.outgoing(:HAS_DC_METADATA_FILE) << dc_node
    end
    if mods_node = metadata_directory_node.outgoing(:HAS_FILE).detect { |node| node.name == 'mods.xml' }
      book_node.outgoing(:HAS_MODS_METADATA_FILE) << mods_node
    end
    book_directory_node.outgoing(:HAS_FILE_ASSET).select { |node| node.name.match(/^toc/) }.each do |table_of_contents_node|
      book_node.outgoing(:HAS_TABLE_OF_CONTENTS_FILE) << table_of_contents_node
      book_node.outgoing(:HAS_MASTER_FILE) if table_of_contents_node.name.match(/pdf$/)
      book_node.outgoing(:HAS_OCR_FILE) if table_of_contents_node.name.match(/txt$/)
    end
    book_directory_node.outgoing(:HAS_FILE_ASSET).select { |node| node.name.match(/^index/) }.each do |table_of_contents_node|
      book_node.outgoing(:HAS_INDEX_FILE) << table_of_contents_node
      book_node.outgoing(:HAS_MASTER_FILE) << table_of_contents_node if table_of_contents_node.name.match(/pdf$/)
      book_node.outgoing(:HAS_OCR_FILE) << table_of_contents_node if table_of_contents_node.name.match(/txt$/)
    end
    book_directory_node.outgoing(:HAS_FILE_ASSET).select { |node| node.name.match(/^page/) }.each do |page_node|
      page_node.name.match(/^page_(\d+)\.(.*)/)
      page_number = $1.to_i
      extension = $2
      relationship = Neography::Relationship.create(:HAS_PAGE, book_node, page_node)
      relationship[:page] = page_number
      book_node.outgoing(:HAS_MASTER_FILE) << page_node if extension == 'pdf'
      book_node.outgoing(:HAS_OCR_FILE) << page_node if extension == 'txt'
    end
  end
end

desc "Add graph structure to the image objects, e.g. access vs. master, metadata, etc."
task :structure_images do

end

desc "Migrate the image assets, changing master format from tiff to jpg"
task :migrate_image_assets do

end

desc "Assuming an empty database run all of the tasks to get to the end state"
task :create_whole_database => [:create_collections, :bit_ingest_collections, :characterize_file_assets,
                                :structure_books, :structure_images, :migrate_image_assets] do
  #don't need to do anything here, the other tasks take care of it
end