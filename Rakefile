require 'neography/tasks'

desc "Create our initial collections"
task :create_collections do

end

desc "Ingest the sample collections at the bit level, i.e. directory structure"
task :bit_ingest_collections do

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