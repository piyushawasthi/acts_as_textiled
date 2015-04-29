## Upgrade Acts as Textiled for Rails 4

## Getting Started

1. Take Clone the Repository:

        git clone git@github.com:piyushawasthi/acts_as_textiled.git

2. Create a Directory lib/plugins in your Rails 4 app:

        mkdir lib/plugins

3. Move "acts_as_textiled" in lib/plugins directory:

        lib/plugins/acts_as_textiled

4. Create initializer to load all plugins in Rails 4.
	
		config/initializers/plugins.rb

5. Paste Code in config/initializers/plugins.rb file.
	# Method for Load all plugins

	Dir[Rails.root.join('lib', 'plugins', '*')].each do |plugin|
	  next if File.basename(plugin) == 'initializers'

	  lib = File.join(plugin, 'lib')
	  $LOAD_PATH.unshift lib

	  begin
	    require File.join(plugin, 'init.rb')
	  rescue LoadError
	    begin
	      require File.join(lib, File.basename(plugin) + '.rb')
	    rescue LoadError
	      require File.join(lib, File.basename(plugin).underscore + '.rb')
	    end
	  end

	  initializer = File.join(File.dirname(plugin), 'initializers', File.basename(plugin) + '.rb')
	  require initializer if File.exists?(initializer)
	end

= Acts as Textiled

This simple plugin allows you to forget about constantly rendering Textile in 
your application.  Instead, you can rest easy knowing the Textile fields you 
want to display as HTML will always be displayed as HTML (unless you tell your
code otherwise).

No database modifications are needed.

You need RedCloth, of course.  And Rails.

== Usage

  class Story < ActiveRecord::Base
    acts_as_textiled :body_text, :description
  end

  >> story = Story.find(3)
  => #<Story:0x245fed8 ... >

  >> story.description
  => "<p>This is <strong>cool</strong>.</p>"

  >> story.description(:source)
  => "This is *cool*."

  >> story.description(:plain)
  => "This is cool."

  >> story.description = "I _know_!"
  => "I _know_!"

  >> story.save
  => true

  >> story.description
  => "<p>I <em>know</em>!</p>"

  >> story.textiled = false
  => false

  >> story.description
  => "I _know_!"

  >> story.textiled = true
  => true

  >> story.description
  => "<p>I <em>know</em>!</p>"

== Different Modes

RedCloth supports different modes, such as :lite_mode.  To use a mode on 
a specific attribute simply pass it in as an options hash after any
attributes you don't want to mode-ify.  Like so:

  class Story < ActiveRecord::Base
    acts_as_textiled :body_text, :description => :lite_mode
  end

Or:

  class Story < ActiveRecord::Base
    acts_as_textiled :body_text => :lite_mode, :description => :lite_mode
  end

You can also pass in multiple modes per attribute:

  class Story < ActiveRecord::Base
    acts_as_textiled :body_text, :description => [ :lite_mode, :no_span_caps ]
  end

Get it?  Now let's say you have an admin tool and you want the text to be displayed
in the text boxes / fields as plaintext.  Do you have to change all your views?  

Hell no.

== form_for

Are you using form_for?  If you are, you don't have to change any code at all.

  <% form_for :story, @story do |f| %>
    Description: <br/> <%= f.text_field :description %>
  <% end %>

You'll see the Textile plaintext in the text field.  It Just Works.

== form tags

If you're being a bit unconvential, no worries.  You can still get at your 
raw Textile like so:

  Description: <br/> <%= text_field_tag :description, @story.description(:source) %>

And there's always object.textiled = false, as demo'd above.

== Pre-fetching

acts_as_textiled locally caches rendered HTML once the attribute in question has 
been requested.  Obviously this doesn't bode well for marshalling or caching.

If you need to force your object to build and cache HTML for all textiled attributes,
call the +textilize+ method on your object.

If you're real crazy you can even do something like this:

  class Story < ActiveRecord::Base
    acts_as_textiled :body_text, :description

    def after_find
      textilize
    end
  end

All your Textile will now be ready to go in spiffy HTML format.  But you probably
won't need to do this.

Enjoy.

* By Piyush Awasthi
