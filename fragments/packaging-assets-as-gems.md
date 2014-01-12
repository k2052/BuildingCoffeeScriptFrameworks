---
title: Packaging Assets as Gems
---

Even in the era of bower there are still a benefit to packaging your assets as a gem. Namely, it allows you to include relevant ruby code along with your assets. You might have helpers that only work with specific JS or JS that only works with specific backend code; it might make little since to package them separately.

First you need to have a gem. If you don't have only you can generate one using bundler e.g `$ bundle gem lorem`.
Place your assets into a folder, lib/assets is a good place.

Then you'll need to make the magic happen. m

    module Ryggrad
      if defined? ::Rails::Engine
        class Rails < ::Rails::Engine
         initializer :assets do |config|
            Rails.application.config.assets.paths << root.join("lib", "assets", "javascripts")
          end
        end

      elsif defined? ::Sprockets
        root = File.expand_path(File.dirname(File.dirname(__FILE__)))
        ::Sprockets.append_path File.join(root, "lib", "assets", "javascripts")
      end
    end
{: .language-ruby}

Simply requiring this will be enough to append to Sprockets. Usually though you'll want to place it this in a register event for your framework:

For Sinatra/Padrino it looks like:

And for Rails: 

That is it! Enjoy!

If you want to checkout an example you can take a look at my ryggrad-ruby gem.

