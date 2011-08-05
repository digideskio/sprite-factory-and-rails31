This is a VERY TEMPORARY AND EXPERIMENTAL rails 3.1 project.

I am using it to decide what changes I might need to make to the
sprite-factory gem to make it fit better with the rails 3.1
asset pipeline

So far, to use the existing sprite-factory (v1.3.0) the basic steps are

Add sprite-factory to your Gemfile
==================================

Plus its image library dependency (either rmagick or chunky_png)

    group :assets do
      # ...
      gem 'sprite-factory'
      gem 'chunky_png'
    end

Keep your individual sprite images in the new app/images directory
==================================================================

E.g

    app/assets/images/avatars/*.png
    app/assets/images/icons/*.png


Create a Rake task for regenerating your sprites
================================================

E.g. in lib/tasks/assets.rake

    namespace :assets do

      desc 'recreate sprite images and css'
      task :resprite => :environment do 
        
        require 'sprite_factory'
        
        SpriteFactory.report  = true         # output report during generation
        SpriteFactory.library = :chunkypng   # use simple chunkypng gem to handle .png sprite generation
        SpriteFactory.layout  = :packed      # pack sprite sheets into optimized rectangles
        SpriteFactory.csspath = '/assets/'   # prepend background image urls with expected rails 3.1 path

        SpriteFactory.run!( 'app/assets/images/avatars'                         )
        SpriteFactory.run!( 'app/assets/images/icons', :selector => 'img.icon_' )
      
      end

      #----------------------------------------------------------------------------

    end
  
Tell Rails 3.1 Asset Pipeline to include your CSS files
=======================================================

E.g. in app/assets/stylesheets/application.css

    *= require ../images/avatars.css
    *= require ../images/icons.css

Add a #sprite_tag helper
========================

E.g. in application_helper.rb

    def sprite_tag(klass, options = {})
      image_tag('s.gif', {:class => klass, :alt => klass}.merge(options))
    end

Use the #sprite_tag helper
==========================

Somewhere in a view, lets show a sprite

    <%= sprite_tag('icon_email') %>
    <%= sprite_tag('avatar7')    %>


GO!
==

Run your rake task to generate the unified .png and .css files

    bundle exec rake assets:resprite

View your page in a browser

 * BOOYA!



FUTURE WORK
===========

Existing version (1.3.0) of sprite factory is limited to only output .css, .sass or .scss files, but
in rails 3.1 we would like to also have the option to generate .css.erb, .sass.erb or .scss.erb files
to indicate to sprockets that they should be run through ERB as part of the asset pipeline.

This would allow our rake task to specify this:

    SpriteFactory.csspath = "<%= asset_path '$IMAGE' %>"

Instead of the hard coded prepend suggested earlier:

    SpriteFactory.csspath = "/assets/"

But I'll need to update sprite-factory to allow output filename overrides, perhaps a simple :erb => true
argument to add the .erb extension ?


