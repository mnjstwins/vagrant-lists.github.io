require "rubygems"
require 'rake'
require 'yaml'
require 'time'
require 'erb'

SOURCE = "."
CONFIG = {
  'layouts' => File.join(SOURCE, "_layouts"),
  'posts' => File.join(SOURCE, "_posts"),
  'post_ext' => "md",
}

# Usage: rake box name="A name" [date="2014-02-15"] [tags="tag1,tag2"]
desc "Add a new box to List"
task :box do
  atts = {}
  atts[:category] = 'boxes'
  atts.merge!(common_part('boxes'))
  atts[:link] = get_stdin("link? (eg. https://example.com/)")
  atts[:size] = get_stdin("Size?(eg. 100MB)")
  atts[:arch] = ask_arch
  atts[:provider] = ask_single_provider
  put_post(atts[:filename], atts)
end # task :box

# Usage: rake plugin name="A name" [date="2014-02-15"] [tags="tag1,tag2"]
desc "Add a new plugin to List"
task :plugin do
  atts = {}
  atts[:category] = 'plugins'
  atts.merge!(common_part('plugins'))
  atts[:type] = ask_plugin_type
  atts[:link] = get_stdin("link? (eg. https://example.com/)")
  put_post(atts[:filename], atts)
end # task :plugin

# Usage: rake recipe name="A name" [date="2014-02-15"] [tags="tag1,tag2"]
desc "Add a new recipe to List"
task :recipe do
  atts = {}
  atts[:category] = 'configs'
  atts.merge!(common_part('configs'))
  atts[:link] = get_stdin("link? (eg. https://example.com/)")
  atts[:supported] = get_stdin("supported providers(eg. virtualbox, vmware)?")
  put_post(atts[:filename], atts)
end # task :recipe

desc "Launch preview environment"
task :preview do
  system "jekyll serve -w"
end # task :preview


# Usage: rake plugin_version
desc "Check gems versions and store to _data"
task :plugin_version do
  version_data = {}
  plugin_filelist = FileList.new('_posts/plugins/*.md')
  plugin_filelist.each do |file|
    open(file,'r').each do |line|
        if m = /name:\s\"(?<plugin>.+)\"/.match(line)
          plugin_name = m[:plugin]
          puts "processing #{plugin_name}"
          ver = gems_version(plugin_name)
          if ver
            version_data[plugin_name] = ver
          end
        end
    end
  end
  outfile = File.join('_data/plugin_versions.yml')
  open(outfile,'w') do |outf|
    outf.puts YAML.dump(version_data)
  end
end


def gems_version(plugin)
  ans = `gem query -n #{plugin} --remote`
  if m = /\((?<version>[0-9\.]+)\)/.match(ans)
    return m[:version]
  end
  nil
end

def common_part(category)
  abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])
  if ENV["name"].to_s == ''
    name = get_stdin("Name? ")
  else
    name = ENV["name"]
  end
  desc = get_stdin("Description(eg. VirtualBox provider)?: ")
  if ENV["tags"]
    tags = ENV["tags"].downcase.strip.split(',')
  else
    taglist = get_stdin("Tags?(eg. tag1,tag2)")
    tags = taglist.downcase.strip.split(',')
  end

  resp = get_stdin("Any note?:")
  if resp.empty?
    note=nil
  else
    note=resp
  end

  slug = name.downcase.strip.gsub(' ', '-').gsub(/[^\w-]/, '')
  begin
    date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%Y-%m-%d')
  rescue => e
    puts "Error - date format must be YYYY-MM-DD, please check you typed it correctly!"
    exit -1
  end
  filename = File.join(CONFIG['posts'], "#{category}", "#{date}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  {:name=>name,:note=>note,:desc=>desc,:tags=>tags,:filename=>filename,:date=>date}
end

def put_post(filename, attribute)
  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    attribute.each do |key, val|
      case key
      when :name then
        post.puts "name: \"#{val}\""
      when :link then
        post.puts "link: #{val}"
      when :desc then
        post.puts "description: \"#{val}\""
      when :category then
        post.puts "category: #{val}"
      when :size then
        post.puts "size: #{val}"
      when :tags then
        post.puts "tags:"
        val.each { |t|
          post.puts" - #{t}"
        }
      when :doc then
        post.puts "doc: #{val}"
      when :type then
        post.puts "type: #{val}"
      when :box then
        post.puts "box: #{box}"
      when :provider then
        post.puts "provider: #{val}"
      when :supported then
        post.puts "providers: #{val}"
      when :filename then
        # skip
      else
        post.puts "#{key}: #{val}"
      end
    end
    post.puts "---"
  end
end

def ask(message, valid_options)
  if valid_options
    answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
  else
    answer = get_stdin(message)
  end
  answer
end

def ask_arch
  case ask("architecure?(x86_64[6], arm[a], i386[3], arm64[r])",['a','3','6','r'])
  when '6'
    'x86_64'
  when '3'
    'i686'
  when 'a'
    'arm'
  when 'r'
    'arm64'
  else
    'x86_64'
  end
end

def ask_plugin_type
  case ask("plugin type?(_Provider, p_Rovisioner, _Command, _Sync)", ['p','r','c','s'])
  when 'p'
    type = "provider"
  when 'r'
    type = "provisioner"
  when 'c'
    type = "command"
  when 's'
    type = "sync_folder"
  else
    type = "unknown"
  end
  type
end

def ask_single_provider
  case ask("Provider?(_Virtualbox, v_Mware, _Lxc, _Kvm, libv_Irt)", ['v','m','l','k','i'])
  when 'v'
    provider = "VirtualBox"
  when 'm'
    provider = "VMware"
  when 'l'
    provider = "Vagrant-LXC"
  when 'k'
    provider = "Vagrant-KVM"
  when 'i'
    provider = "Vagrant-libvirt"
  else
    provider = "unknown"
  end
  provider
end

def get_stdin(message)
  print message
  STDIN.gets.chomp
end
