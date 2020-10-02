#!/usr/bin/env ruby

require 'fileutils'
require 'yaml'
require 'erb'
require 'uri'

MAX_HEIGHT       = 2048
THUMB_HEIGHT     = 600
WGET             = `which wget`.chop
GM               = `which gm`.chop
CONVERT          = `which convert`.chop
HOME             = File.expand_path('..', __FILE__)
CONFIG_FILE      = "#{HOME}/sources.yml"
SRC_DIR          = "#{HOME}/original-images"
THUMBNAIL_DIR    = "#{HOME}/public/thumbnails"
WALLPAPER_DIR    = "#{HOME}/public/wallpapers"
GALLERY_TEMPLATE = "#{HOME}/templates/gallery.html.erb"
GALLERY_FILE     = "#{HOME}/public/index.html"
JSON_TEMPLATE    = "#{HOME}/templates/gallery.json.erb"
JSON_FILE        = "#{HOME}/public/index.json"
BASE_URL         = "https://calyxos.gitlab.io/wallpapers"

class Image
  attr_reader :thumbnail_url, :image_url, :filename,
              :input_file, :output_file, :thumbnail_file,
              :source_url, :author, :tags, :name

  def initialize(img)
    @filename       = image_filename(img)
    @input_file     = File.join(SRC_DIR, filename)
    @output_file    = File.join(WALLPAPER_DIR, filename)
    @thumbnail_file = File.join(THUMBNAIL_DIR, filename)
    @thumbnail_url  = URI.escape File.join("thumbnails", @filename)
    @image_url      = URI.escape File.join("wallpapers", @filename)
    @source_url     = img["source_url"]
    @author         = img["author"]
    @tags           = img["tags"]
    @name           = img["name"]
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
          puts output
          match_data = output&.match(/srgb\((.*)\)/)
          if match_data && match_data[1]
            r,g,b = match_data[1]&.split(',')&.map {|i| i&.to_i }
            if r && g && b
              rgb_to_h(r,g,b)
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

  private

  def image_filename(image)
    extension = File.extname(image["source_url"])
    "%s - %s%s" % [image["name"], image["author"], extension]
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
  end

end

def resize_image(input:, output:, max_height:, force: false)
  if File.exist?(output)
    if force || File.mtime(output) <= File.mtime(input)
      FileUtils.rm(output)
    else
      puts "NO CHANGE: %s" % output
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

def render(images:, template:, output_file:)
  renderer = ERB.new(File.read(template), nil, "<>")
  @images = images
  File.open(output_file, 'w') do |f|
    f.write renderer.result(binding)
  end
  puts "CREATED: #{output_file}"
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
  source_images.each do |image|
    image.download
    image.process
  end
  source_images = source_images.sort {|a,b| a.hue <=> b.hue}
  render(images: source_images, template: GALLERY_TEMPLATE, output_file: GALLERY_FILE)
  render(images: source_images, template: JSON_TEMPLATE, output_file: JSON_FILE)
end
