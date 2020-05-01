require 'fileutils'
require 'json'
require 'yaml'

metadata = YAML.load_file('metadata.yml')

desc 'Cleans the build artifacts'
task :clean do
  FileUtils.rm_r Dir.glob('public/**/*.{gif,json,jpg}')
end

desc 'Copies the assets present in metadata to public folder'
task :assets do
end

desc 'Generates previews for GIFs'
task :previews do
end

FileList['static/gifs/*.gif'].each do |source|
  file = source.pathmap("%n")
  metadata[file] ||= {'tags' => []}

  target = source.pathmap('public/gifs/%f')
  preview = source.pathmap('public/previews/%f').ext('.jpg')
  file target => source do
    FileUtils.copy_file(source, target)
  end

  file preview => source do
    `convert #{source}[0] -resize 400x400 -background Black -gravity center -extent 400x400 #{preview} `
  end

  task assets: target
  task previews: preview
end

desc 'Generates the JSON manifest with  our GIF library'
task :manifest do
  manifest = []

  metadata.each do |file, props|
    manifest << props.merge({
      name: file,
      path: {
        gif: file.pathmap('/gifs/%f').ext('.gif'),
        preview: file.pathmap('/previews/%f').ext('.jpg'),
      }
    })
  end

  File.open("public/manifest.json","w") do |f|
    f.write(manifest.to_json)
  end
end

namespace :metadata do
  task :format do
    metadata.each do |file, props|
      props['tags'] ||= []
      metadata[file]['tags'] = props['tags'].sort
    end

    yml = YAML.dump(Hash[metadata.sort])
    puts yml
  end
end

task :missing do
  FileList['static/gifs/*.gif'].each do |source|
    file = source.pathmap('%n')
    props = metadata[file]

    puts source.pathmap('%f') if !props || !props['tags'].any?
  end
end

task gifs: [:manifest, :assets, :previews]
task default: :gifs
