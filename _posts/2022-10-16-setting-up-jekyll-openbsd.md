---
layout: post
title: "Setting up a local jekyll server on OpenBSD"
---
First, we need ruby. OpenBSD's ports collection has 3 versions, in my case
`vim` decided on `ruby-3.0.4` (aka `ruby30`).

```
pkg_add ruby30
gem30 install jekyll
```

The `minima` theme has to be installed... in a predictable way?

I don't like having things get out of control is `/usr`, so let's figure out
how to get packages to install in a way like npm does with `node_modules`
or python with `venv`s. I'm getting `CPAN` flashbacks.

After a few web searches...

```csh
touch .bundle/config
bundle30 config --local path vendor/bundle
echo 'vendor/' >> .gitignore
cat <<EOT > Gemfile
source "https://rubygems.org"

gem "minima"
EOT
bundle30
```

*(Note: the `Gemfile` wants a *global source*, otherwise it complains about
something or other being deprecated.)*

Oups, that started to installed jekyll again (but this time in the vendor
dir), `^C`! `^C`! `rm -rf vendor`.

Let's check out that `.bundle/config` file... Well, the docs tell me I *can*
use system installed gems. I just need to...

```
---
BUNDLE_PATH: "vendor/bundle"
BUNDLE_DISABLE_SHARED_GEMS: false
```

...and we get an error saying this is not supported. Pffsh... Let's mess
around with GEM:

```csh
setenv GEM_HOME `pwd`/vendor/bundle
setenv GEM_PATH /usr/local/lib/ruby/gems/3.0/:$GEM_HOME
```

Let's re-run `bundle30`:

```
blog> bundle30 
Fetching gem metadata from https://rubygems.org/...........
Resolving dependencies..........
Using bundler 2.2.33
Using public_suffix 5.0.0
Using colorator 1.1.0
Using concurrent-ruby 1.1.10
...
Installing github-markdown 0.6.9 with native extensions
Bundle complete! 2 Gemfile dependencies, 32 gems now installed.
Use `bundle info [gemname]` to see where a bundled gem is installed.
```

This time it went by much faster. And it didn't ask me for my password, so
no `sudo`, so it did the right thing. Ironically, I don't have `sudo` set
up, I didn't even realize it's installed! It turns out some random window
manager package had pulled it in.

In any case, I'll need to remember to set those env vars.

Let's try jekyll!

```csh
blog> jekyll
jekyll: Command not found.
```

Since OpenBSD's method of dealing with multiple versions of the same port
existing on the same system, all installed binaries have the `30` suffix
attached to them. So:

```csh
blog> jekyll30 --serve drafts
Configuration file: /home/jakkal/Projects/blog/_config.yml
            Source: /home/jakkal/Projects/blog
       Destination: /home/jakkal/Projects/blog/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
Markdown processor: "GFM" is not a valid Markdown processor.
                    Available processors are: kramdown
...
```

Okay, let's add that to our Gemfile.

```
echo 'gem "gfm"' >> Gemfile
bundle30
```

Many compiles later...

```
> jekyll30 serve --drafts
...
                    ------------------------------------------------
      Jekyll 4.2.2   Please append `--trace` to the `serve` command 
                     for any additional information or backtrace. 
                    ------------------------------------------------
/usr/local/lib/ruby/gems/3.0/gems/jekyll-4.2.2/lib/jekyll/commands/serve/servlet.rb:3:in `require': cannot load such file -- webrick (LoadError)
```

Argh. `echo 'gem "webrick"' >> Gemfile ; bundle30`.

```
> jekyll30 serve --drafts                                                   Configuration file: /home/jakkal/Projects/blog/_config.yml
            Source: /home/jakkal/Projects/blog
       Destination: /home/jakkal/Projects/blog/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
       Jekyll Feed: Generating feed for posts
                    done in 5.456 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    require 'rbconfig'
    if RbConfig::CONFIG['target_os'] =~ /(?i-mx:bsd|dragonfly)/
      gem 'rb-kqueue', '>= 0.2'
    end
 Auto-regeneration: enabled for '/home/jakkal/Projects/blog'
    Server address: http://127.0.0.1:4000
  Server running... press ctrl-c to stop.
```

Finally!

![screenshot of browser displaying this page]({{ "/assets/images/2022-10-16-jekyll-in-links.png" }})

It turns out:
- this local jekyll really wants that frontmatter thing
  + index.md needs `home`
  + posts need `post`
  + pages need `page`
- assets have to be in `project_root/assets`
- jekyll should probably be run with `bundle30 exec jekyll30 server --drafts`
- I now have gems randomly installed in `/usr` and in the project dir; this is a problem for future me, because building some of the native dependencies takes forever on this ancient laptop
