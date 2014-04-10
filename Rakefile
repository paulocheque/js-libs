def colorize(text, color)
  color_codes = {
    :black    => 30,
    :red      => 31,
    :green    => 32,
    :yellow   => 33,
    :blue     => 34,
    :magenta  => 35,
    :cyan     => 36,
    :white    => 37
  }
  code = color_codes[color]
  if code == nil
    text
  else
    "\033[#{code}m#{text}\033[0m"
  end
end

CONTENT_TYPES = {
  ".js" => "application/x-javascript",
  ".css" => "text/css",
  ".map" => "application/octet-stream",
  ".png" => "image/png",
  ".jpg" => "image/jpeg",
  ".gif" => "image/gif",
}

CONTENT_ENCODING = {
  ".js" => "gzip",
  ".css" => "gzip",
  ".map" => nil,
  ".gif" => nil,
  ".jpg" => nil,
  ".png" => nil,
}

def upload_to_s3(filename, bucket_name)
  puts colorize("Uploading to S3: #{filename}", :blue)
  require 'aws-sdk' # gem install aws-sdk
  AWS.config(access_key_id: ENV["AWS_ACCESS_KEY_ID"], secret_access_key: ENV["AWS_SECRET_ACCESS_KEY"], region: ENV["AWS_REGION"])
  s3 = AWS.s3
  bucket = s3.buckets[bucket_name]
  if not bucket.exists? then
    puts colorize("Creating bucket #{bucket_name}", :blue)
    bucket = s3.buckets.create(bucket_name, :acl => :public_read)
    bucket.configure_website
  else
    puts colorize("Bucket #{bucket_name} already exists", :blue)
  end
  key = File.basename(filename)
  obj = bucket.objects[key]
  obj.write(:file => filename, :acl => :public_read,
      :cache_control => "max-age=31536000", :expires => "Thu, 31 Dec 2015 23:59:59 GM",
      :content_type => CONTENT_TYPES[File.extname(filename)],
      :content_encoding => CONTENT_ENCODING[File.extname(filename)])
  puts colorize("Upload finished", :green)
  puts colorize(obj.public_url, :blue)
end

def minify_js(filename)
  sh "uglifyjs #{filename}.js -o #{filename}.min.js --source-map #{filename}.min.js.map -p relative -c -m"
  return "#{filename}.min.js"
end

def compress_js(filename)
  sh "gzip -9 #{filename}"
  sh "mv #{filename}.gz #{filename}"
  return "#{filename}"
end

def prepare_js(filename)
  if filename.end_with?(".js") and not filename.end_with?("min.js")
    filename = minify_js(filename.sub(".js", ""))
  end
  if filename.end_with?(".js") or filename.end_with?(".css")
    filename = compress_js(filename)
  end
  return filename
end

def upload_files_to_s3(files)
  files.each { |filename|
    if filename.end_with?(".js")
      filename = prepare_js(filename)
    end
    upload_to_s3(filename, "weblibraries")
  }
end

task :clean => [] do
  sh "rm -rf ~*"
  sh "rm -rf data/"
end

task :info => [] do
  sh "ruby --version"
  sh "nokogiri -v"
  sh "gem list aws-sdk"
  puts "AWS_ACCESS_KEY_ID = #{ENV["AWS_ACCESS_KEY_ID"]}"
  puts "AWS_SECRET_ACCESS_KEY = #{ENV["AWS_SECRET_ACCESS_KEY"]}"
  puts "AWS_REGION = #{ENV["AWS_REGION"]}"
end

task :dependencies => [:dev_env] do
  sh "sudo npm install uglify-js -g"
  sh "gem install aws-sdk"
  sh "rvm install 1.9.3 --with-gcc=clang"
  sh "rvm use ruby-1.9.3-p194" # aws-sdk is not working with Ruby 2.0.0
end

task :upload_rest_client => [] do
  files = [
    "libs/secret-rest-client/enc-base64.min.js",
    "libs/secret-rest-client/hmac-sha256.min.js",
    "libs/secret-rest-client/secret-rest-client.js",
  ]
  upload_files_to_s3(files)
end

task :upload_data_table => [] do
  files = [
    "libs/secret-data-table/secret-data-table.js",
  ]
  upload_files_to_s3(files)
end

task :upload_bootstrap => [] do
  # netdna.bootstrapcdn.com/bootstrap/3.1.0
  files = [
    "libs/bootstrap/bootstrap-theme.min.css",
    "libs/bootstrap/bootstrap.min.css",
    "libs/bootstrap/bootstrap.min.js",
    "libs/bootstrap/font-awesome.css",
  ]
  upload_files_to_s3(files)
end

task :upload_all => [:upload_rest_client, :upload_data_table, :upload_bootstrap] do
  files = [
    "libs/ba-linkify.min.js",
    "libs/bootstrap-maxlength.min.js",
    "libs/jqBootstrapValidation.min.js",
    "libs/jquery-2.0.3.min.js",
    "libs/jquery.form.min.js",
    "libs/jquery.maskedinput.min.js",
    "libs/jquery.tablesorter.min.js",
    "libs/moment.min.js",
    "libs/pnotify/jquery.pnotify.default.css",
    "libs/pnotify/jquery.pnotify.default.icons.css",
    "libs/pnotify/jquery.pnotify.min.js",
    "libs/jquery-ui/jquery-ui-1.10.4.custom.min.js",
  ]
  upload_files_to_s3(files)
end

task :upload_file => [] do
  # rake upload_file file=filepath
  filename = ENV["file"]
  upload_files_to_s3([filename])
end

task :default => [:prepare_libs_js]
