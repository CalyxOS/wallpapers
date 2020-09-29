#!/usr/bin/env ruby

require 'fileutils'
require 'yaml'
require 'erb'
require 'uri'

MAX_HEIGHT       = 2048
THUMB_HEIGHT     = 200
WGET             = `which wget`.chop
GM               = `which gm`.chop
HOME             = File.expand_path('..', __FILE__)
CONFIG_FILE      = "#{HOME}/sources.yml"
SRC_DIR          = "#{HOME}/original-images"
THUMBNAIL_DIR    = "#{HOME}/public/thumbnails"
WALLPAPER_DIR    = "#{HOME}/public/wallpapers"
GALLERY_TEMPLATE = "#{HOME}/gallery.html.erb"
GALLERY_FILE     = "#{HOME}/public/index.html"

class Image
  attr_reader :thumbnail_url, :image_url, :filename,
              :input_file, :output_file, :thumbnail_file,
              :source_url
  def initialize(img)
    @filename       = image_filename(img)
    @input_file     = File.join(SRC_DIR, filename)
    @output_file    = File.join(WALLPAPER_DIR, filename)
    @thumbnail_file = File.join(THUMBNAIL_DIR, filename)
    @thumbnail_url  = URI.escape File.join("thumbnails", @filename)
    @image_url      = URI.escape File.join("wallpapers", @filename)
    @source_url     = img["source_url"]
  end

  def process
    if resize_image(input: input_file, output: output_file, max_height: MAX_HEIGHT)
      resize_image(input: input_file, output: thumbnail_file, max_height: THUMB_HEIGHT)
    end
  end

  def download
    if File.exist?(input_file)
      puts "SOURCE EXISTS: #{filename}"
    else
      puts "FETCHING: #{filename}"
      wget_download(from: source_url, to: input_file)
    end
  end

  private

  def image_filename(image)
    extension = File.extname(image["source_url"])
    "%s - %s%s" % [image["description"], image["author"], extension]
  end
end

def resize_image(input:, output:, max_height:)
  if File.exist?(output)
    if File.mtime(output) <= File.mtime(input)
      FileUtils.rm(output)
    else
      puts "NO CHANGE: %s" % output
      return true
    end
  end

  FileUtils.cp(input, output)

  ok = system("gm", "mogrify", "-geometry", "x#{max_height}", output)
  if ok
    puts "CREATED: %s" % output
    return true
  else
    return false
  end
end

def wget_download(from:, to:)
  system(WGET, from, "-O", to)
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

def image_filename(image)
  extension = File.extname(image["source_url"])
  "%s - %s%s" % [image["description"], image["author"], extension]
end

def render_gallery(images)
  renderer = ERB.new(File.read(GALLERY_TEMPLATE))
  @images = images
  File.open(GALLERY_FILE, 'w') do |f|
    f.write renderer.result(binding)
  end
  puts "CREATED: #{GALLERY_FILE}"
end

def check_requirements
  if GM == ""
    puts "MISSING REQUIREMENT: apt install graphicsmagick"
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
  source_images.each do |image|
    image.download
    image.process
  end
  render_gallery(source_images)
end
