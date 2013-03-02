# High Voltage [![Build Status](https://travis-ci.org/thoughtbot/high_voltage.png)](http://travis-ci.org/thoughtbot/high_voltage)

Rails engine for static pages.

... but be careful. [Danger!](http://www.youtube.com/watch?v=HD5tnb2RBYg)

Static pages?
-------------

Yeah, like "About us", "Directions", marketing pages, etc.

Installation
------------

    $ gem install high_voltage

Include in your Gemfile:

    gem "high_voltage"

For Rails versions prior to 3.0, use the 0.9.2 tag of high_voltage:

    https://github.com/thoughtbot/high_voltage/tree/v0.9.2

Usage
-----

Write your static pages and put them in the RAILS_ROOT/app/views/pages directory.

    $ mkdir app/views/pages
    $ touch app/views/pages/about.html.erb

After putting something interesting there, you can link to it from anywhere in your app with:

    link_to "About", page_path("about")

This will also work, if you like the more explicit style:

    link_to "About", page_path(:id => "about")

You can nest pages in a directory structure, if that makes sense from a URL perspective for you:

    link_to "Q4 Reports", page_path("about/corporate/policies/HR/en_US/biz/sales/Quarter-Four")

Bam.

Routes
------

By default, the static page routes will be like /pages/:id (where :id is the view filename).

If you want to route to a static page in another location (for example, a homepage), do this:

    get 'pages/home' => 'high_voltage/pages#show', :id => 'home'

In that case, you'd need an app/views/pages/home.html.erb file.

Generally speaking, you need to route to the 'show' action with an :id param of the view filename.

High Voltage will generate a named route method of `page_path` which you can use, as well.  If you
want to generate a named route (with the :as routing option) for some route which will be handled
by High Voltage, make sure not to use :page as the name, because that will conflict with the named
route generated by High Voltage itself.

For example, this will work for top-level routes (we will
get a named route called `static_path` which does not conflict with the generated `page_path` method):

    get '/:id' => 'high_voltage/pages#show', :as => :static, :via => :get

#### Specifying a root path

You can route the root url to a high voltage page like this:

    root :to => 'high_voltage/pages#show', :id => 'home'

Which will render a homepage from app/views/pages/home.html.erb

#### Enabling root domain routes

You can remove the directory `pages` from the URL path and serve up routes from 
the root of the domain path:

    http://www.example.com/about
    http://www.example.com/company

Would look for corresponding files:

    app/views/pages/about.html.erb
    app/views/pages/company.html.erb

This is accomplished by changing the `HighVoltage.route_drawer` to `HighVoltage::RouteDrawers::Root`

    # config/initializers/high_voltage.rb
    HighVoltage.route_drawer = HighVoltage::RouteDrawers::Root

Note: This is not a catchall route. It will check the `HighVoltage.content_path` 
(by default this is `app/views/pages`) and validate the view exists.

#### Disabling routes

The default routes can be completely removed by changing the
`HighVoltage.routes` to `false`

    # config/initializers/high_voltage.rb
    HighVoltage.routes = false

Customize
--------

High Voltage uses a default path and folder of 'pages', i.e. 'url.com/pages/contact' , 'app/views/pages'

You can change this in an initializer:

    HighVoltage.content_path = "site/"

Override
--------

Most common reasons to override?

  * You need authentication around the pages to make sure a user is signed in.
  * You need to render different layouts for different pages.
  * You need to render a partial from the `app/views/pages` directory.

Create a PagesController of your own:

    $ rails generate controller pages

Override the default route:

    # in config/routes.rb
    get "/pages/*id" => 'pages#show', :as => :page, :format => false

    # if routing the root path, update for your controller
    root :to => 'pages#show', :id => 'home'

Then modify it to subclass from High Voltage, adding whatever you need:

    class PagesController < HighVoltage::PagesController
      before_filter :authenticate
      layout :layout_for_page

      protected
        def layout_for_page
          case params[:id]
          when 'home'
            'home'
          else
            'application'
          end
        end
    end

Custom finding
--------------

You can further control the algorithm used to find pages by overriding
the `page_finder_factory` method:

    class PagesController < HighVoltage::PagesController
      private

      def page_finder_factory
        Rot13PageFinder
      end
    end

The easiest thing is to subclass `HighVoltage::PageFinder`, which
provides you with `page_id`:

    class Rot13PageFinder < HighVoltage::PageFinder
      def find
        paths = super.split('/')
        directory = paths[0..-2]
        filename = paths[-1].tr('a-z','n-za-m')

        File.join(*directory, filename)
      end
    end

Use this to create a custom file mapping, clean filenames for your file
system, A/B test, and so on.

Testing
-------

You can test your static pages using [RSpec](https://github.com/rspec/rspec-rails) 
and [shoulda-matchers](https://github.com/thoughtbot/shoulda-matchers):

    # spec/controllers/pages_controller_spec.rb
    describe PagesController, '#show' do
      %w(earn_money screencast about contact).each do |page|
        context 'on GET to /pages/#{page}' do
          before do
            get :show, :id => page
          end

          it { should respond_with(:success) }
          it { should render_template(page) }
        end
      end
    end

If you're not using a custom PagesController be sure to test
`HighVoltage::PagesController` instead.

Enjoy!

Contributing
------------

Please see CONTRIBUTING.md for details.

Credits
-------

![thoughtbot](http://thoughtbot.com/images/tm/logo.png)

High Voltage is maintained and funded by [thoughtbot, inc](http://thoughtbot.com/community)

Thank you to all [the contributors](https://github.com/thoughtbot/high_voltage/contributors)!

The names and logos for thoughtbot are trademarks of thoughtbot, inc.

License
-------

High Voltage is Copyright © 2009-2013 thoughtbot. It is free software, and may
be redistributed under the terms specified in the MIT-LICENSE file.
