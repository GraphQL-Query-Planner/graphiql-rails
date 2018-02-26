require 'bundler/setup'
Bundler::GemHelper.install_tasks

require 'rake/testtask'

Rake::TestTask.new do |t|
  t.libs << "test" << "lib"
  t.pattern = "test/**/*_test.rb"
end

task(default: :test)

desc "Vendor the latest GraphiQL version"
task :update_graphiql do
  require "fileutils"
  require "json"

  new_js_versions = {}
  new_css_versions = {}

  def replace_versions(path, new_versions)
    old_contents = File.read(path)
    new_contents = new_versions.reduce(old_contents) do |contents, (package, new_version)|
      contents.gsub(/#{package}-\d+\.\d+\.\d+/, "#{package}-#{new_version}")
    end
    File.write(path, new_contents)
  end

  def npm_version(package_json_path)
    npm_config = JSON.parse(File.read(package_json_path))
    npm_config["version"]
  end

  update_path = "./packages/graphiql/"

  old_assets = Dir["app/assets/**/{react,graphiql}-*.{js,css}"]
  puts "Removing #{old_assets.join(", ")}"
  FileUtils.rm(old_assets)

  FileUtils.mkdir_p(update_path)
  FileUtils.cd(update_path) do
    sh("yarn install")
    sh("yarn run build")

    new_version = npm_version("./package.json")
    new_js_versions["graphiql"] = new_version
    new_css_versions["graphiql"] = new_version

    puts "Copying GraphiQL #{new_version}"
    FileUtils.cp("./graphiql.js", "../../app/assets/javascripts/graphiql/rails/graphiql-#{new_version}.js")
    FileUtils.cp("./graphiql.css", "../../app/assets/stylesheets/graphiql/rails/graphiql-#{new_version}.css")
    # end

    FileUtils.cd("./node_modules/react") do
      new_version = npm_version("./package.json")
      new_js_versions["react"] = new_version

      puts "Copying React #{new_version}"
      FileUtils.cp("./dist/react.js", "../../../../app/assets/javascripts/graphiql/rails/react-#{new_version}.js")
    end

    FileUtils.cd("./node_modules/react-dom") do
      new_version = npm_version("./package.json")
      new_js_versions["react-dom"] = new_version

      puts "Copying ReactDOM #{new_version}"
      FileUtils.cp("./dist/react-dom.js", "../../../../app/assets/javascripts/graphiql/rails/react-dom-#{new_version}.js")
    end
  end

  js_manifest_path = "app/assets/javascripts/graphiql/rails/application.js"
  css_manifest_path = "app/assets/stylesheets/graphiql/rails/application.css"
  puts "Updating manifests"
  replace_versions(js_manifest_path, new_js_versions)
  replace_versions(css_manifest_path, new_css_versions)
end


task :update_graphiql_plugins do
  require "fileutils"
  require "json"

  package = "./packages/graphiql-plugins"
  assets_dir = "./app/assets/javascripts/graphiql-plugins"
  rails_assets = "#{assets_dir}/main.js"

  FileUtils.mkdir_p(assets_dir)
  puts "Created directory: #{assets_dir}"

  if File.exist?(rails_assets)
    puts "Removing #{rails_assets}"
    FileUtils.rm(rails_assets)
  end

  puts "Copying graphiql-plugins"
  FileUtils.cp("#{package}/dist/main.js", rails_assets)
end
