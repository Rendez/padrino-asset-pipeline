# Padrino Asset Pipeline [![Build Status](https://travis-ci.org/Rendez/padrino-asset-pipeline.svg?branch=master)](https://travis-ci.org/Rendez/padrino-asset-pipeline)
======================

An asset pipeline implementation for Padrino based on [Sprockets](https://github.com/sstephenson/sprockets), **working for Padrino multi-apps**, with support for CoffeeScript, SASS, SCSS, LESS, ERB as well as CSS (SASS, YUI) and JavaScript (uglifier, YUI, Closure) minification.

padrino-asset-pipeline is a fork of [sinatra-asset-pipeline](https://github.com/kalasjocke/sinatra-asset-pipeline) and supports both compiling assets on the fly for development as well as precompiling assets for production.

# Installation

Include padrino-asset-pipeline in your project's Gemfile:

```ruby
gem 'padrino-asset-pipeline'
```

Make sure to add the padrino-asset-pipeline Rake task in your applications `Rakefile`:

```ruby
require 'sinatra/asset_pipeline/task'

Sinatra::AssetPipeline::Task.define!(SampleBlog::App)
Sinatra::AssetPipeline::Task.define!(SampleBlog::Admin)
```

If your application runs Sinatra in classic style you can define your Rake task as follows:

```ruby
Sinatra::AssetPipeline::Task.define! Sinatra::Application
```

Now, when everything is in place you can precompile assets located in `assets/<asset-type>` with:

```bash
$ RACK_ENV=production rake assets:precompile
```

And remove old compiled assets with:

```bash
$ RACK_ENV=production rake assets:clean
```

# Sample Gemfile

```ruby
source 'https://rubygems.org'
source 'https://rails-assets.org'

gem 'rake'

gem 'padrino', '0.12.4'
gem 'padrino-asset-pipeline', :require => 'sinatra/asset_pipeline', :git => 'https://github.com/proudsugar/padrino-asset-pipeline.git'

gem 'compass'

# Bower components
# Add https://rails-assets.org as the new gem source, then reference any Bower components that you need as gems in the following convention:
# gem 'rails-assets-BOWER_PACKAGE_NAME'
gem 'rails-assets-bootstrap-sass-official', '~> 3.3' # contains jquery dependency

group :production do
  gem 'uglifier'
end
```

# Example of Padrino Sub-Apps

In its most simple form, you just register the `Sinatra::AssetPipeline`
Padrino extension within your applications:

```ruby
module SampleBlog
  class App < Padrino::Application
    # [...]
    configure do
      # If your application doesn't follow the defaults you can customize it as follows:
      # Include these files when precompiling assets
      set :assets_precompile, %w(application.js application.css jquery.js *.png *.jpg *.svg *.eot *.ttf *.woff)
      # Logical paths to your assets
      set :assets_prefix, %w(app/assets)
      # Use another host for serving assets
      set :assets_host, '<id>.cloudfront.net'
      # Serve assets using this protocol (http, :https, :relative)
      set :assets_protocol, :http
      # CSS minification
      set :assets_css_compressor, :sass
      # JavaScript minification
      set :assets_js_compressor, :uglifier
      register Sinatra::AssetPipeline
      # Register the AssetPipeline extention, make sure this goes after all customization
      if defined?(RailsAssets)
        # Actual Rails Assets integration, everything else is Sprockets
        RailsAssets.load_paths.each do |path|
          settings.sprockets.append_path(path)
        end
      end
    end

    get '/' do
      haml :index
    end
  end
end
```

```ruby
module SampleBlog
  class Admin < Padrino::Application
    # [...]
    configure do
      # If your application doesn't follow the defaults you can customize it as follows:
      # Include these files when precompiling assets
      set :assets_precompile, %w(application.js application.css jquery.js *.png *.jpg *.svg *.eot *.ttf *.woff)
      # Logical paths to your assets
      set :assets_prefix, %w(admin/assets)
      # Public path: affects compilation destination of targets and path prefixes
      set :assets_path, File.join(public_dir, "admin/assets")
      # CSS minification
      set :assets_css_compressor, :sass
      # JavaScript minification
      set :assets_js_compressor, :uglifier
      register Sinatra::AssetPipeline
      # Register the AssetPipeline extention, make sure this goes after all customization
      if defined?(RailsAssets)
        # Actual Rails Assets integration, everything else is Sprockets
        RailsAssets.load_paths.each do |path|
          settings.sprockets.append_path(path)
        end
      end
    end
  end
end
```

Now when everything is in place you can use all helpers provided by [sprockets-helpers](https://github.com/petebrowne/sprockets-helpers), an example:

```scss
body {
  background-image: image-url('cat.png');
}
```

Note that you don't need to require [sprockets-helpers](https://github.com/petebrowne/sprockets-helpers) inside your code to leverage the functionallity given to you by the integration, padrino-asset-pipeline handles that for you.

### CSS and JavaScript minification

If you would like to use CSS and/or JavaScript minification make sure to require the needed gems in your `Gemfile`:

<table>
  <tr>
    <th>Minifier</th>
    <th>Gem</th>
  </tr>
  <tr>
    <td>:sass</td>
    <td>sass</td>
  </tr>
  <tr>
    <td>:closure</td>
    <td>closure-compiler</td>
  </tr>
  <tr>
    <td>:uglifier</td>
    <td>uglifier</td>
  </tr>
  <tr>
    <td>:yui</td>
    <td>yui-compressor</td>
  </tr>
</table>

### Compass integration

Given that we're using [sprockets-sass](https://github.com/petebrowne/sprockets-sass) under the hood we have out of the box support for [compass](https://github.com/chriseppstein/compass). Just include the compass gem in your `Gemfile` and include the compass mixins in your `app.css.scss` file.

