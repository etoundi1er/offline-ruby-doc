<section id="content" class="clearfix">

<header>

## Offline Ruby/Rails Documentation

</header>

<article class="post">

18 May 2014

<div>

Undoubtedly, we software developers spend a good portion of our time reading documentation, often online. Recently, I have been traveling a lot and although I have my mobile phone set up for tethering, the internet connection is very fragile at times. Waiting for pages to load forever is not really fun, so I needed a way to access the most important Ruby and Rails documentation offline. Since there are quite a few steps involved in setting this up, I thought I’d share it and hope it will be useful for others as well.

Please be aware that this guide is written for OS X users; if you use a different operating system, you will need to adjust the steps accordingly.

### Pow

Pow lets you start a local server that serves apps under a local url like `http://app-name.dev`. This makes it convenient to access the documentation we will set up below and frees you from need to keep servers running on localhost with meaningless port numbers that noone can remember. You can install it via

<figure class="highlight">

```
curl get.pow.cx | sh
```

</figure>

For more information, visit [pow.cx](http://pow.cx). I recommend using Pow in conjunction with the [`powder` gem](https://github.com/rodreegez/powder), which provides a convenience wrapper around the bare shell commands that are used to manage up Pow apps. The most important command we will use is `powder link [APP_NAME]`, which sets up your app to be served locally by Pow. You can install `powder` from Rubygems via

<figure class="highlight">

```
gem install powder
```

</figure>

### Ruby Core Documentation

Pre-rendered Ruby core documentation is [available from ruby-doc.org](http://www.ruby-doc.org/downloads/). To serve it locally via Pow, you can do:

<figure class="highlight">

```
mkdir ruby-doc-dev
cd ruby-doc-dev
curl -O http://www.ruby-doc.org/downloads/ruby_2_1_1_core_rdocs.tgz
tar xvf ruby_2_1_1_core_rdocs.tgz
mv ruby_2_1_1_core public
powder link ruby-doc
```

</figure>

Your Ruby documentation is now available at [http://ruby-doc.dev](http://ruby-doc.dev). Of course, adjust the version number to your needs and/or download multiple versions.

### Gem Documentation

<figure class="highlight">

```
gem install yard
```

</figure>

Save the following file as `~/Library/LaunchAgents/YardDocs.plist` and adjust the path to the `yard` executable as needed.

<figure class="highlight">

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>KeepAlive</key>
  <true/>
  <key>Label</key>
  <string>YardDocs</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/opt/rbenv/shims/yard</string>
      <string>server</string>
      <string>--gems</string>
  </array>
  <key>RunAtLoad</key>
  <true/>
</dict>
</plist>
```

</figure>

Next, start the server and set up Pow to do the port forwarding.

<figure class="highlight">

```
launchctl load -w ~/Library/LaunchAgents/YardDocs.plist
echo '8808' > ~/.pow/gem-doc
```

</figure>

Your Ruby documentation is now available at [http://gem-doc.dev](http://gem-doc.dev). If you are missing some gems in the list or have disabled documentation generation to speed up Bundler as I did, you can regenerate all Gem documentations with the following command:

<figure class="highlight">

```
gem rdoc --all
```

</figure>

### Rails API Documentation

Rails documentation can be generated directly from a Rails app, so the first step is to make a minimal Rails app that will serve the only purpose of displaying documentation.

<figure class="highlight">

```
mkdir rails-api-dev
```

</figure>

First, add the `rails` gem in the desired version to your `Gemfile`. You will also need the `sdoc`, `redcarpet` and `nokogiri` gems to generate the documentation, so go ahead and add these as well.

<figure class="highlight">

```
# Gemfile

source 'https://rubygems.org'

gem 'rails', '4.1.1'

gem 'sdoc'
gem 'redcarpet'
gem 'nokogiri'
```

</figure>

Then, install all the Gems with

<figure class="highlight">

```
bundle install
```

</figure>

Rails expects an application to be present before it lets you run the Rake task to generate the documentation. You can fake an entire Rails application directly in the `Rakefile` like this:

<figure class="highlight">

```
# Rakefile

require 'rails/all'
Bundler.require

module FakeApp
  class Application < Rails::Application; end
end

Rails.application.load_tasks
```

</figure>

Now, you have Rails’ Rake tasks at your disposal and are able to generate the documentation:

<figure class="highlight">

```
rake doc:rails
```

</figure>

To serve the documentation with Pow, you need to add a Rackup file at `config.ru`

<figure class="highlight">

```
# config.ru

require 'rack'

use Rack::Static, urls: ['/'], root: 'doc/api', index: 'index.html'
run ->{}
```

</figure>

and finally you can serve it via Pow:

<figure class="highlight">

```
powder link rails-api
```

</figure>

### Rails Guides

To read the Rails Guides locally, you can simply clone the folder that you have created for the Rails API Documentation.

<figure class="highlight">

```
cd ..
cp -R rails-api-dev rails-guides-dev
cd rails-guides-dev
```

</figure>

This time, you need to run a different Rake task to generate the guides:

<figure class="highlight">

```
rake doc:guides
```

</figure>

Since the guides end up in a different directory than the API documentation, you will need to adjust the location specified in the Rackup file as well:

<figure class="highlight">

```
# config.ru

require 'rack'

use Rack::Static, urls: ['/'], root: 'doc/guides', index: 'index.html'
run ->{}
```

</figure>

Finally, you can set up Pow to serve the Rails Guides:

<figure class="highlight">

```
rm .powder
powder link rails-guides
```

</figure>

</div>

</article>

</section>
