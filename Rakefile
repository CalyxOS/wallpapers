#!/usr/bin/env ruby

require 'fileutils'
require 'yaml'
require 'erb'
require 'cgi'
require 'digest/bubblebabble'

MAX_HEIGHT       = 2048
THUMB_HEIGHT     = 256
WGET             = `which wget`.chop
GM               = `which gm`.chop
CONVERT          = `which convert`.chop
HOME             = File.expand_path('..', __FILE__)
CONFIG_FILE      = "#{HOME}/sources.yml"
SRC_DIR          = "#{HOME}/original-images"
THUMBNAIL_PATH   = "thumbnails"
WALLPAPER_PATH   = "wallpapers"
API_PATH         = "photos"
THUMBNAIL_DIR    = "#{HOME}/public/#{THUMBNAIL_PATH}"
WALLPAPER_DIR    = "#{HOME}/public/#{WALLPAPER_PATH}"
GALLERY_TEMPLATE = "#{HOME}/templates/gallery.html.erb"
GALLERY_FILE     = "#{HOME}/public/index.html"
INDEX_TEMPLATE   = "#{HOME}/templates/gallery.json.erb"
INDEX_FILE       = "#{HOME}/public/#{API_PATH}/index.json"
IMAGE_TEMPLATE   = "#{HOME}/templates/image.json.erb"
IMAGE_DIR        = "#{HOME}/public/#{API_PATH}"
BASE_URL         = "https://calyxos.gitlab.io/wallpapers"

class Image
  attr_reader :thumbnail_url, :image_url, :info_url, :source_url,
              :filename, :input_file, :output_file, :thumbnail_file,
              :author, :tags, :name, :hue, :color

  def initialize(img)
    @filename       = image_filename(img)
    @url_path       = image_path(img)
    @input_file     = File.join(SRC_DIR, @filename)
    @output_file    = File.join(WALLPAPER_DIR, @url_path)
    @thumbnail_file = File.join(THUMBNAIL_DIR, @url_path)
    @thumbnail_url  = [THUMBNAIL_PATH, CGI.escape(@url_path)].join('/')
    @image_url      = [WALLPAPER_PATH, CGI.escape(@url_path)].join('/')
    @source_url     = img["source_url"]
    @info_url       = img["info_url"]
    @author         = img["author"]
    @tags           = img["tags"]
    @name           = img["name"]
  end

  def id
    @id ||= Digest::MD5.bubblebabble("#{name} #{author}")[0..22]
  end

  def process
    force = !ENV["FORCE_RENDER_IMAGE"].nil?
    if resize_image(input: input_file, output: output_file, max_height: MAX_HEIGHT, force: force)
      force = !ENV["FORCE_RENDER_THUMBNAIL"].nil?
      resize_image(input: input_file, output: thumbnail_file, max_height: THUMB_HEIGHT, force: force)
    end
  end

  def download
    if File.exist?(input_file)
      puts "SOURCE EXISTS: #{filename}"
      no_change(input_file)
    else
      puts "FETCHING: #{filename}"
      wget_download(from: source_url, to: input_file)
    end
  end

  def hue
    @hue ||= begin
      if File.exist?(@thumbnail_file)
        IO.popen([CONVERT, @thumbnail_file, '-resize', '1x1', 'txt:']) do |io|
          output = io.read
          match_data = output&.match(/srgb\((.*)\)/)
          if match_data && match_data[1]
            @r,@g,@b = match_data[1]&.split(',')&.map {|i| i&.to_i }
            if @r && @g && @b
              rgb_to_h(@r,@g,@b)
            else
              0
            end
          else
            0
          end
        end
      end
    end
  end

  def color
    @color ||= begin
      hue
      rgb_to_hex(@r||0, @g||0, @b||0)
    end
  end

  private

  def image_filename(image)
    extension = File.extname(image["source_url"]).downcase
    if image["author"] && image["author"] != ""
      "%s - %s%s" % [image["name"], image["author"], extension]
    else
      "%s%s" % [image["name"], extension]
    end
  end

  def image_path(image)
    extension = File.extname(image["source_url"]).downcase
    if image["author"] && image["author"] != ""
      path = "%s-%s" % [image["name"], image["author"]]
    else
      path = image["name"]
    end
    path = path.downcase.gsub(/[\s\.\,]+/, '-').gsub(/\-+/, '-')
    path + extension
  end

  def rgb_to_hex(r,g,b)
    str = "#"
    [r,g,b].each do |i|
      str += i.to_i.to_s(16).rjust(2, "0")
    end
    str
  end

  def rgb_to_h(r,g,b)
    max = [r,b,g].max
    min = [r,b,g].min
    d = (max - min).to_f

    if max == min
      h = 0
    else
      h = case max
      when r then (g - b) / d + (g < b ? 6 : 0)
      when g then (b - r) / d + 2
      when b then (r - g) / d + 4
      end
      h /= 6.0
    end
    h *= 360
    return h.round
  end

end

def resize_image(input:, output:, max_height:, force: false)
  if File.exist?(output)
    if force || File.mtime(output) <= File.mtime(input) || File.size(output) == 0
      FileUtils.rm(output)
    else
      no_change output
      return true
    end
  end

  if GM != ""
    FileUtils.cp(input, output)
    ok = system("gm", "mogrify", "-geometry", "x#{max_height}", output)
  else
    ok = system("convert", input, "-adaptive-resize", "x#{max_height}", output)
  end
  if ok
    created output
    return true
  else
    return false
  end
end

def wget_download(from:, to:)
  if system(WGET, from, "-O", to)
    created to
  end
end

def load_sources
  sources_list = YAML.load_file(CONFIG_FILE)
  return_list = []
  sources_list.each do |image_hash|
    next if image_hash.nil? || image_hash["source_url"].nil? || image_hash["source_url"] == ""
    return_list << Image.new(image_hash)
  end
  return return_list
rescue Psych::SyntaxError => exc
  puts "ERROR: could not parse the file #{CONFIG_FILE}:"
  puts exc.to_s
  exit
end

def created(file)
  puts "CREATED: #{file}"
  $files_created[file] = true
end

def no_change(file)
  puts "NO CHANGE: #{file}"
  $files_created[file] = true
end

def render(template:, output_file:, images:nil, image:nil)
  renderer = ERB.new(File.read(template), nil, "<>")
  if images
    @images = images
  else
    @image = image
  end
  File.open(output_file, 'w') do |f|
    f.write renderer.result(binding)
  end
  created output_file
end

def clean_up_files
  [SRC_DIR, THUMBNAIL_DIR, WALLPAPER_DIR].each do |dir|
    Dir.glob(dir + '/*').each do |file|
      if $files_created[file]
        next
      else
        puts "REMOVING: #{file}"
        FileUtils.rm file
      end
    end
  end
  [IMAGE_DIR].each do |dir|
    Dir.glob(dir + '/*/index.json').each do |file|
      if $files_created[file]
        next
      else
        puts "REMOVING: #{file}"
        FileUtils.rm file
        FileUtils.rmdir File.dirname(file)
      end
    end
  end
end

def check_requirements
  if GM == ""
    if CONVERT == ""
      puts "MISSING REQUIREMENT: apt install graphicsmagick"
    end
  end
  if WGET == ""
    puts "MISSING REQUIREMENT: apt install wget"
  end
end

task :default => :process

desc "download and process all the images"
task :process do
  check_requirements
  source_images = load_sources
  $files_created = {}
  source_images.each do |image|
    image.download
    image.process
  end
  source_images = source_images.sort {|a,b| a.hue <=> b.hue}
  render(
    images: source_images,
    template: GALLERY_TEMPLATE,
    output_file: GALLERY_FILE
  )
  render(
    images: source_images,
    template: INDEX_TEMPLATE,
    output_file: INDEX_FILE
  )
  source_images.each do |image|
    FileUtils.mkdir_p(File.join(IMAGE_DIR, image.id))
    render(
      image: image,
      template: IMAGE_TEMPLATE,
      output_file: File.join(IMAGE_DIR, image.id, "index.json")
    )
  end
  clean_up_files
end
