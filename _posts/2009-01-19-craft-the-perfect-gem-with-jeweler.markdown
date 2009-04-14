--- 
title: Craft the perfect gem with Jeweler
tags: 
- bacon
- gems
- rails
- ruby
- rubygems
- shoulda
layout: post
---
I've been working on [Jeweler](http://github.com/technicalpickles/jeweler) for some time now, but never announced it officially. That changes now. This is kind of a long winded intro, so if you want to get to just jump into the action without the explanation, check out the latest [README](http://github.com/technicalpickles/jeweler).

### Intro

[Rubygems](http://www.rubygems.org/) are the awesome way of distributing your code to others. [GitHub](http://github.com/) and [git](http://git-scm.com/) are the best tools for version control of your project. GitHub can even [generate a Rubygem if you include a gemspec](http://gems.github.com/).

The trouble is when developing your Rubygems on GitHub, you generally do one of the following:

 * Manage the gemspec by hand
  * ... why bother doing something by hand when you can automate it?
 * Write your own Rake stuff to create the Gem::Specification and output it to a gemspec file, and deal with keeping the Rakefile and gemspec in sync
  * ... why keep reinventing the wheel for each project?
 * Use hoe or echoe for generating the gemspec
  * ... why use tools made for the days before GitHub existed?
  * ... why have extra stuff you aren't going to use or want?
  * ... what's with the weird configuration that's not particularly well documented except by example?

Jeweler was created with a few goals in mind:

 * Only use a Gem::Specification as configuration
 * Be one command away from version bumping and releasing
 * Store version information in one place
 * Only concern itself with git, gems, and versioning
 * Not be a requirement for using your Rakefile (you just wouldn't be able to use its tasks)
 * Use Jeweler to manage Jeweler. Oh the meta!

### Installation

Run the following if you haven't already:

    gem sources -a http://gems.github.com

Install the gem:

    sudo gem install technicalpickles-jeweler

### Configuration for an existing project

Armed with the gem, we can begin diving into an example. [the-perfect-gem](http://github.com/technicalpickles/the-perfect-gem/tree) was setup as a simple example and showcase:

{% highlight ruby %}
begin
  require 'jeweler'
  Jeweler::Tasks.new do |s|
    s.name = "jeweler"
    s.executables = "jeweler"
    s.summary = "Simple and opinionated helper for creating Rubygem projects on GitHub"
    s.email = "josh@technicalpickles.com"
    s.homepage = "http://github.com/technicalpickles/jeweler"
    s.description = "Simple and opinionated helper for creating Rubygem projects on GitHub"
    s.authors = ["Josh Nichols"]
    s.files =  FileList["[A-Z]*", "{bin,generators,lib,test}/**/*", 'lib/jeweler/templates/.gitignore']
    s.add_dependency 'schacon-git'
  end
rescue LoadError
  puts "Jeweler, or one of its dependencies, is not available. Install it with: sudo gem install technicalpickles-jeweler -s http://gems.github.com"
end
{% endhighlight %}

Here's a rundown of what's happening:

 * Wrap everything in a begin block, and rescue from LoadError
  * This lets us degrade gracefully if jeweler isn't installed
 * Create an instance of `Jeweler::Tasks`
   * It gets yielded a new `Gem::Specification`
   * This is where all the configuration happens
    * Things you definitely need to specify:
      * `name`
    * Things you probably want to specify:
      * `summary`
      * `email`
      * `homepage`
      * `description`
      * `authors`
    * Things you can specify, but have defaults
     * `files`, defaults to `FileList["[A-Z]*.*", "{bin,generators,lib,test,spec}/**/*"]`
    * Things you shouldn't specify:
     * `version`, because Jeweler takes care of this for you
    * Other things of interest
     * `executables`, if you have any scripts
     * `add_dependency`, if you have any dependencies
    * Keep in mind that this is a `Gem::Specification`, so you can do whatever you would need to do to get your gem in shape. See the [reference](http://www.rubygems.org/read/chapter/20#page85) for more details.
    
### Bootstrap a new project

Before proceeding, take a minute to setup your git environment, specifically [setup your name and email for git](http://github.com/guides/tell-git-your-user-name-and-email-address) and [your username and token for GitHub](http://github.com/guides/local-github-config):

    $ git config --global user.email johndoe@example.com
    $ git config --global user.name 'John Doe'
    $ git config --global github.user johndoe
    $ git config --global github.token 55555555555555

Jeweler provides a generator, `jeweler`. It requires only one argument, the name of a repo you want to create. It also takes a few options:

 * --[shoulda](http://github.com/thoughtbot/shoulda): generate shoulda-based tests (this is the default)
 * --[bacon](http://github.com/chneukirchen/bacon/tree/master): generate bacon-based tests
 * --summary SUMMARY: specify the summary to use
 * --create-repo: create the repo on GitHub and enable rubygems for it

Putting it all together looks like:

    $ jeweler --create-repo --summary "Oh so perfect" the-perfect-gem

The effect is this:

 * Creates the the-perfect-gem directory
 * Seeds it with some basic files:
  * `.gitignore`, with the usual suspects predefined
  * `Rakefile`, setup with tasks for jeweler, test, rdoc, and rcov
  * `README`, with your project name
  * `LICENSE`, MIT, with your name prefilled
  * `test/test_helper`, setup with shoulda, mocha, and a re-opened `Test::Unit::TestCase`
  * `test/the_perfect_gem.rb`, placeholder failing test
  * `lib/the_perfect_gem.rb`, placeholder library file
 * Initializes a git repository
 * Sets up `git@github.com:johndoe/jeweler.git` as the `origin` git remote
 * Makes an initial commit
 * Creates up a new repository on GitHub and pushes to it (omit --create-repo to skip this)
 * Enables RubyGems for the repository

### Overview of Jeweler workflow

Here's the general idea:

 * Hack, commit, hack, commit, etc, etc
 * Version bump
 * Release
 * Have a delicious scotch

The hacking and the scotch are up to you, but Jeweler provides rake tasks for the rest.

### Versioning

Versioning information is stored in `VERSION.yml`. It's a plain YAML file which contains three things:

 * major
 * minor
 * patch

Consider `1.5.3`. This maps to:

 * major == 1
 * minor == 5
 * patch == 3

##### Your first time

When you first start using Jeweler, there won't be a `VERSION.yml`, so it'll assume 0.0.0.

If you need some arbitrary version, you can do one of the following:

 * `rake version:write MAJOR=6 MINOR=0 PATCH=3`
 * Write `VERSION.yml` by hand (lame)

#### After that

You have a few rake tasks for doing the version bump:

    $ rake version:bump:patch # 1.5.1 -> 1.5.2
    $ rake version:bump:minor # 1.5.1 -> 1.6.0
    $ rake version:bump:major # 1.5.1 -> 2.0.0

If you need to do an arbitrary bump, use the same task you used to create `VERSION.yml`:

    $ rake version:write MAJOR=6 MINOR=0 PATCH=3

The process of version bumping does a commit to your repo, so make sure your repo is in a clean state (ie nothing uncommitted).

#### Release it

It's pretty straight forward:

    $ rake release

This takes care of:

 * Generating a `.gemspec` for you project, with the version you just bumped to
 * Commit and push the updated `.gemspec`
 * Create a versioned tag
 * Push the tag

#### Play the waiting game

How do you know when your gem is built? [Has My Gem Built Yet](http://hasmygembuiltyet.org/) was specifically designed to answer that question.

If it happens to be down, you can also check out the GitHub Gem repo's [list](http://gems.github.com/list.html). Just search for yourname-yourrepo.

#### Putting it all together

    # hack, hack, hack, commit, etc
    $ rake version:bump:patch release

Now browse to http://hasmygembuiltyet.org/yourname/yourproject, and wait for it to be built.

### Finale

You should have a good idea of what Jeweler is all about by now. It makes gem creation an impulse action. It makes managing versions of gems nice and automated.

Oh, and did I mention it is the [GitHub recommended way for maintaining your gem](http://img.skitch.com/20090119-h644bj6enrn2t8i5tej37uxif.jpg)?

Now go forth, and craft the perfect gem.
