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
  directory_entries = (Dir[File.join(full_path, '*')] + Dir[File.join(full_path, '.*')]).reject {|entry| %w(. ..).include?(File.basename(entry))}
  subdirectories = directory_entries.select {|entry| File.directory?(entry)}
  files = directory_entries.select {|entry| File.file?(entry)}
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

end

desc "Add graph structure to the book objects, e.g. pages, table of contents and index as appropriate, ocr vs. master, metadata"
task :structure_books do

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