require 'nanoc3/tasks'
require 'fileutils'

begin
  require "vlad"
  Vlad.load :scm => "git"
rescue LoadError
  # DO NOTHING
end

# rake create:article title='Post title'
namespace :create do

  desc "Creates a new article"
  task :article do
    $KCODE = 'UTF8'
    require 'active_support/core_ext'
    require 'active_support/multibyte'
    @now = Time.now.to_s(:db)
    @ymd = @now.split(' ')[0]
    if !ENV['title']
      $stderr.puts "\t[error] Missing title argument.\n\tusage: rake create:article title='article title'"
      exit 1
    end

    title = ENV['title']
    path, filename, full_path = calc_path(title)

    if File.exists?(full_path)
      $stderr.puts "\t[error] Exists #{full_path}"
      exit 1
    end

    template = <<TEMPLATE
---
created_at: #{@now}
timestamp: #{Time.now.to_i}
excerpt: ""
kind: article
publish: true
tags: [misc]
disqus: true
title: "#{title}"
---

TODO: Add content to `#{full_path}.`
TEMPLATE

    FileUtils.mkdir_p(path) if !File.exists?(path)
    File.open(full_path, 'w') { |f| f.write(template) }
    $stdout.puts "\t[ok] Edit #{full_path}"
  end

  def calc_path(title)
    year, month_day = @ymd.split('-', 2)
    path = "content/" + year + "/"
    filename = title.parameterize('-') + ".md"
    [path, filename, path + filename]
  end
end
