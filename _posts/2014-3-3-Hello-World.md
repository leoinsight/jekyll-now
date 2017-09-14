---
layout: post
title: Github Page Hosting (by Jekyll)
---

## Jekyll is
- A static site generator that's perfect for GitHub hosted blogs
- Static, Public, Markdown

## How to start
1. Fork [Jekyll Now](https://github.com/barryclark/jekyll-now) to my repository
2. Rename repository name (Settings menu) (ex. yourusername.github.io)
3. Modify `_config.yml` (name, description of my site)
4. Modify `yyyy-m-d-Hello-World.md` at *_posts* directory
5. Add a new `yyyy-m-d-File-Name.md` for new contents

## Local Development (for Windows)
1. Install Ruby from [RubyInstaller for Windows](https://rubyinstaller.org/downloads/)
2. Install Jekyll and plug-ins. `gem install github-pages`
3. Clone down your fork `git clone https://github.com/yourusername/yourusername.github.io.git`
4. Start local server 'jekyll serve -w' and view your website at <http://127.0.0.1/4000/>
5. Commit any changes and push everything to the master branch of your GitHub user repository.

## Thanks to
- [barryclark/jekyll-now](https://github.com/barryclark/jekyll-now) (English)
- <http://fkkmemi.com/github/github-hosting/> (Korean)