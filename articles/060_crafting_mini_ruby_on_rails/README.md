# Crafting mini RubyOnRails

<!-- Photo by  [Jason D](https://unsplash.com/@jasondeblooisphotography?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText)  on  [Unsplash](https://unsplash.com/?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText) -->

![Photo by Jason D on Unsplash](image01.jpg)



## Introduction
Once I watched Maple Ong's talk  [Building a Ruby web app using the Ruby Standard Library](https://www.youtube.com/watch?v=lxczDssLYKA)  on Euruko 2021.  The talk inspired me and I've got an idea to write a little web-server from scratch. Then I started to improve the [original script](https://github.com/kopylovvlad/mini_rails/blob/master/script.rb) and decided to write an MVC-framework as RubyOnRails on plain Ruby. Such a good challenge.

I called it `MiniRails`, the source code is available in [mono repo on GitHub](https://github.com/kopylovvlad/mini_rails). In the article I describe the process of creating the library. The goal of the article is to describe which concepts and ideas I used: how MVC layers work, how router matches a client request and how I implemented a test library. Some code examples are short to show the main concept without excess amount of code. If you want to look deeper, there is always links to the course code on Github. I hope my experience will be useful for readers.

## Build a rack middleware
Begin with writing a rack-middleware. [Rack](https://github.com/rack/rack) is a standard library for writing a web server. The main structure is simple. Here is [an example](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_dispatcher/hello_handler.rb):
```ruby
module MiniActionDispatch
  # Rack-middleware to render hello world page
  class HelloHandler
    # Receive rack-env and build response in rack format
    def call(env)
      [200, { "Content-Type" => 'text/html' }, ["<h1>Hello to Ruby on MiniRails</h1>"]]
    end
  end
end
```

Middleware for the application looks like this. At the first sight it's more difficult [example](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_dispatcher/request_handler.rb), but the algorithm is clear.
```ruby
# Rack-middleware to handler rack-request
class MiniActionDispatch::RequestHandler
  def call(env)
    # Wrap data to convenient interface
    req = Rack::Request.new(env)

    # Fetch params from a request
    method_token = req.request_method
    path = req.path || req.path_info

    # Wrap params to data-object to handle request params and http-headers
    action_params = ::MiniActionParams.parse(req)
    params = action_params.params
    headers = action_params.headers

    # DELETE, PUT, PATCH support
    if method_token == 'POST' && ['DELETE', 'PUT', 'PATCH'].include?(params[:_method]&.upcase)
      method_token = params[:_method].upcase
    end

    # Match path with route map and find the controller to handle request
    selected_route = MiniActiveRouter::Base.instance.find(method_token, path)
    controller_name, controler_method_name = selected_route.controller_data
    # Route's placeholder support such as :id
    placeholders = selected_route.parse_placeholders(path)
    params = params.merge(placeholders)

    # Find a controller
    controller_class = Object.const_get "#{controller_name.camelize}Controller"
    controller = controller_class.new(params, headers)

    # Run controller's action
    # Construct the HTTP response and return it
    controller.build_response(controler_method_name)
  end
end
```

The algorithm has 3 steps:
	* Handle user request data
	* Match route and find a controller
	* Pass data to controller and execute a method

## Dive deeper and look how the router works
For routing I've written `MiniActiveRouter` module. My goal was to implement basic algorithm from Rails router, such as in the [example](https://github.com/kopylovvlad/mini_rails/blob/master/todo_list/config/router.rb) below:
```ruby
MiniActiveRouter::Base.instance.draw do
  get '/', to: 'home#index'

  # Group scope - JSON
  get '/api/groups', to: 'api/groups#index'
  get '/api/groups/:id', to: 'api/groups#show'
  post '/api/groups', to: 'api/groups#create'
  patch '/api/groups/:id', to: 'api/groups#update'
  delete '/api/groups/:id', to: 'api/groups#destroy'

  not_found to: 'not_found#index'
end
```

There are `get`, `post`, `patch`, `put`, `delete` functions to define routes and `not_found` method for 404 page handler. Routes also support placeholders such as "id" in path. `MiniActiveRouter::Base` is a singleton class, current [code is here](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_router/base.rb). The singleton stores array of routes in `@map` variable and holds a route for 404 page in `@fallback_route` variable.
```ruby
class MiniActiveRouter::Base
  include ::Singleton

  def initialize
    @map = []
    @fallback_route = nil
  end

  # NOTE: Method for drawing routes map
  # Use it in config/router.rb file
  def draw(&block)
    instance_eval &block
  end

  # @param path [String, Regexp]
  # @param arg [Hash]
  # NOTE: post, delete, put, patch method are similar
  def get(path, arg)
    write_to_map('GET', path, **arg)
  end

  # NOTE: Method for set a route for 404 page
  def not_found(to: )
    @fallback_route = Route.new(nil, nil, to: to)
  end

  private
  def write_to_map(method, path, to:)
    transformed_path = transform_path(path)
    @map << Route.new(method, transformed_path, to: to)
  end
end
```

Drawing routes algorithm isn't difficult. For `draw` function it uses `instance_eval` , it make the code inside the do-end block plain and easy to read. `get`, `post`, `patch`, etc methods are just wrappers for `write_to_map` function.

Adding placeholder in path feature wasn't a trivial task. I wanted to define a route in the following manner:
```ruby
patch '/api/groups/:id', to: 'api/groups#update'
patch '/groups/:group_id/items/:id', to: 'items#update'
```

The easiest way I figured out is using regular expressions. Regexp has groups feature. For route `/api/groups/:id` we can write a regexp `/\/api\/groups\/(?<id>[0-9]*)/` and it will match group "id".
```ruby
"/api/groups/123" =~ /\/api\/groups\/(?<id>[0-9]*)/
# => 0
match_data = Regexp.last_match
# => #<MatchData "/api/groups/123" id:"123">
```

For placeholders feature I've written the method `transform_path`. It match placeholders, if  they are here it converts string to a regexp.
```ruby
class MiniActiveRouter::Base
  # If a string has placeholders (for example "items/:id")
  # It converts string to regexp with groups (for example /items\/(:id[0-9a-zA-Z]*)/)
  def transform_path(path)
    # 1: Find all placeholders
    placeholders = path.scan(/:[0-9a-zA-Z\-_]*/)
    return path if placeholders.size == 0

    # 2: Replace each placeholder to (?<placeholder_name>[0-9a-zA-Z]*)
    placeholders.each do |placeholder|
      path = path.gsub(placeholder, "(?<#{placeholder}>[0-9a-zA-Z\\-_]*)")
    end

    # 3: Return the route as an regexp
    return Regexp.new("^#{path}$")
    end
  end
end
```

How does the route matching work? There is a line of code in the the previous rack-middleware. Method `.find` receives a method token, a path and matches data with all available routes.
```ruby
selected_route = MiniActiveRouter::Base.instance.find(method_token, path)
# Example: MiniActiveRouter::Base.instance.find('GET', '/items/1234')
```

Under the hood, the algorithm is simple: match all routes, if there is not a match return the default route if it exists.
```ruby
class MiniActiveRouter::Base
  # NOTE: it iterates each route from @map
  # Returns matched router or @fallback_route
  # @param method [String]
  # @param path [String]
  # @return [Route]
  def find(method, path)
    matched_route = @map.find{ |route| route.match?(method, path) }
    if matched_route.present?
      matched_route
    elsif @fallback_route.present?
      @fallback_route
    else
      raise "ERROR: Can't find route for #{method}##{path}"
    end
  end
end
```

## Controllers layer
After the matching, we know which controller could handle the client request. Below you can see the code from rack-middleware.
```ruby
controller_name, controler_method_name = selected_route.controller_data
# Find controller
controller_class = Object.const_get "#{controller_name.camelize}Controller"
controller = controller_class.new(params, headers)

# Run controller's action
# Construct the HTTP response and return it
controller.build_response(controler_method_name)
```

Inside the application a [controller looks familiar](https://github.com/kopylovvlad/mini_rails/blob/master/todo_list/app/controllers/items_controller.rb) as an average controller in Ruby On Rails application.
```ruby
class ItemsController < ApplicationController
  before_action :find_group

  def index
    @items = @group.items.sort_by(&:active?).map{ |i| ItemDecorator.new(i) }
    render :index
  end

  def create
    @item = Item.new(permited_params)
    if @item.save
      redirect_to "/groups/#{@group.id}/items"
    else
      @alert = @item.errors.full_messages.join(', ')
      render :new, status: '422 Unprocessable Entity'
    end
  end

  private
  def find_group
    @group = Group.find(params[:group_id])
  end
end
```
Also [it supports](https://github.com/kopylovvlad/mini_rails/blob/master/todo_list/app/controllers/application_controller.rb) `before_action` callback and `rescue_from` handler.
```ruby
class ApplicationController < MiniActionController::Base
  rescue_from ::MiniActiveRecord::RecordNotFound, with: :not_found

  private
  def not_found
    render('not_found/index', status: '404 Not Found')
  end
end
```

Let's look how `#build_response` [method works](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_controller/base.rb).
```ruby
class MiniActionController::Base
  # @param controler_method_name [String, Symbol]
  def build_response(controler_method_name)
    begin
      # 1: Run all callbacks
      run_callbacks_for(controler_method_name.to_sym)
      # 2: Run the controller action
      response = public_send(controler_method_name)
    rescue StandardError => e
      # 3: If there is an exception, try to find :rescue_from handler
      response = try_to_rescue(e)
    end
    build_rack_response(response)
  end
end
```
There are few steps:
	* Run all callbacks if they exist
	* Run the controller action
	* Build a standard rack response
	* If it catches an error, it tries to rescue the exception

### Callbacks
How to define and run callbacks? You can see the code in `MiniActionController::Callbacks` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_controller/callbacks.rb). There is the method `before_action`  in MiniRails. Using it we can define callbacks with conditions such as:
```ruby
before_action :method_name, only: [:index, :show], unless: -> { @foo.nil? }
before_action :method_name, except: [:index, :show], if: -> { @foo }
```

`before_action` method stores data in `callbacks`. In order to share data between class and class instance I use the `class_attribute` feature from `ActiveSupport` . [Here is](https://apidock.com/rails/Class/class_attribute) the link to docs. It's very useful feature, I like it and often use it.
```ruby
module MiniActionController::Callbacks
  def self.included(base)
    base.class_attribute :callbacks
    base.callbacks = []
  end
end
```

Return to `MiniActionController::Base#build_response` method, `run_callbacks_for` method iterates each `before_action` defined callback, matches conditions such as `only:`, `except:`, `if:`, `unless:`  and then it executes a method.

### Rescuing
How to rescue an exceptions? You can see the code in  `MiniActionController::Rescuable` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_controller/rescuable.rb).
```ruby
module MiniActionController::Rescuable
  def self.included(base)
    base.class_attribute :rescue_attempts
    base.rescue_attempts = []
    base.extend ClassMethods
  end

  private
  # NOTE: Find first handle for the exception and run
  # @param exception [StandardError]
  def try_to_rescue(exception)
    rescue_attempt = self.class.rescue_attempts.find do |meta|
      exception.is_a?(meta[:exception])
    end
    raise exception if rescue_attempt.nil?

     send(rescue_attempt[:with])
  end

  module ClassMethods
    # Example of usage:
    # rescue_from User::NotAuthorized, with: :deny_access
    # rescue_from ActiveRecord::RecordInvalid, with: :show_errors
    # @param exception [StandardError]
    # @param with [String, Symbol]
    def rescue_from(exception, with: nil)
      rescue_attempt = { exception: exception, with: with }
      self.rescue_attempts = self.rescue_attempts + [rescue_attempt]
    end
  end
end
```
Algorithm is similar. It uses the class attribute `rescue_attempts` to store rescue handlers. Function `rescue_from` adds data to `rescue_attempts`, method `try_to_rescue` receives an exception and tries to find a handle.

## Views layer
### Rendering
We saw how controllers layer works, let's see how it renders views. For views layer I've implemented  `MiniActionView` namespace. For view files I use ERB-files, because it's simple and familiar to use. Example of a simple view:
```ruby
# Controler method
def new
  @group = Group.new
  render :new, status: "200 OK"
end

# View new.html.erb file
<h2>Create new group</h2>

<%= render_partial 'shared/_new_group_form', locals: { item: @group } %>

<a href="/">Go to back</a>
```
It supports partial file render, and received instance variables from a controller method.

How are controllers layer and views layer connected and how it passes instance variables from controller method to a view? The easiest way is to include `MiniActionView` module to `MiniActionController`, but I don't consider it as a good idea, I prefer a different way.

Let's see how method `render` works. [The code](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_controller/render.rb) inside `MiniActionController::Render` module collects all instance variables inside `collect_variables` method and passes data as a  `Hash` to a `MiniActionView::Base` instance.
```ruby
module MiniActionController::Render
  # @param view_name [String, Symbol]
  # @param status [String]
  # @return [MiniActionController::Response]
  def render(view_name, status: MiniActionController::DEFAULT_STATUS)
    # collect and forward instance variables to MiniActionView::Base
    variables_to_pass = collect_variables
    MiniActionView::Base.new(variables_to_pass, entity).render(view_name, status: status)
  end

  private
  def collect_variables
    instance_variables.reduce({}) do |memo, var_symbol|
      memo[var_symbol] = instance_variable_get(var_symbol)
      memo
    end
  end
end
```

The `render`  method of `MiniActionView::Base`  [class](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_view/base.rb) executes `render_view` function in order to render ERB-file and it returns a value-object as a result
```ruby
module MiniActionView::Base
  # @param view_name [String, Symbol]
  # @param status [String]
  # @param content_type [String] html by default
  # @return [MiniActionController::Response]
  def render(view_name, status: MiniActionController::DEFAULT_STATUS, content_type: 'html')
    response_message = render_view("#{view_name}.html.erb")
    MiniActionController::Response.new(
      status: status, response_message: response_message, content_type: content_type, headers: {},
    )
  end
end
```

You can see the rendering algorithm in `MiniActionView::Render` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_view/render.rb), here is the main logic in the snippet below:
```ruby
module MiniActionView::Render
  # NOTE: If view_name has `/` symbol it searches for a file in app/views folder
  # If view_name hasn't `/` symbol it searches for a file in entity folder
  def render_view(view_name, locals: {})
    root_path = MiniRails.root.join('app', 'views')
    root_path = root_path.join(entity) unless view_name.include?('/')
    view_path = root_path.join(view_name).to_s

    # assign data from locals: as local variables
    local_binding = binding
    locals.each do |key, value|
      local_binding.local_variable_set(key, value)
    end
    ERB.new(read_or_open(view_path)).result(local_binding)
  end
end
```

The  `locals` argument is used to pass value to a view. In Rails you used to write something like `render form, locals: {zone: @zone, item: @item}`.  In order to pass data to ERB-file I create a copy of current `binding`, assign data and pass the variable `local_binding` to `result` method.

In the same module you can see the code how to render partial views inside a current view. `render_partial` is a function to render a partial view. Under the hood it's just a wrapper for private `render_view` function.
```ruby
module MiniActionView::Render
  # @param view_name [String, Symbol]
  # @param collection [Array<Object>] each item will be passed as 'item' variable
  # @param locals [Hash<Symbol,Object>] params to passing data as local_variables
  def render_partial(view_name, collection: [], locals: {})
    if collection.size > 0
      collection.map { |i| render_view(view_name, locals: {item: i}) }.join('')
    else
      render_view(view_name, locals: locals)
    end
  end
end
```


### Layout
When view is already rendered it's time to render a layout. You can see [the code](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_view/layout.rb) in `MiniActionView::Layout` class. The `render_response` method renders layout with the result of controller's action and builds a rack response.
```ruby
# Note: Class to render layout for views.
# ERB-file that contains layout-template must be in app/views/layouts/ folder
# For example 'app/views/layouts/application.html.erb'
class MiniActionView::Layout < ::MiniActionView::Base
  # @param layout [String, Symbol]
  # @param response [MiniActionController::Response]
  def render_response(layout, response)
    status_code, _status_text = response.status.split(' ')
    additional_headers = response.headers.map{ |k,v| "#{k}: #{v}" }.join("\n\r")
    headers = {"Content-Type" => "text/html"}.merge(response.headers)
    response_body = render_layout(layout) { response.response_message }
    # Construct the Rack response
    [status_code, headers, [response_body]]
  end

  private
  def render_layout(layout_name)
    view_path = MiniRails.root.join('app', 'views', self.entity, "#{layout_name}.html.erb").to_s
    ERB.new(read_or_open(view_path)).result(binding)
  end
end
```

### JSON response
MiniRails also supports JSON-responses and serialiser-objects for ruby objects. In a controller you able to write the code as:
```ruby
class Api::GroupsController < ::Api::ApplicationController
  before_action :groups, only: [:index]
  before_action :group, only: [:show, :update, :destroy]

  def index
    render_json(@groups, each_serializer: GroupSerializer)
  end

  def show
    render_json(@group, serializer: GroupSerializer)
  end
end
```

The source code also locates in `MiniActionController::Render` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_controller/render.rb), it's able to receive different data types:
```ruby
module MiniActionController::Render
  # Examples of usage:
  # String: render_json({a: 123}.to_json)
  # Array: render_json([1,2,3])
  # Array with root: render_json([1,2,3], root: 'data')
  # Object. render_json(Item.all)
  # With serializer: render_json(Item.first, serializer: ItemSerializer)
  # With each_serializer: render_json(Item.all, each_serializer: ItemSerializer)
  #
  # @param object [String, Hash, Object]
  # Object should respond to .as_json and return Hash
  # @param opts [Hash]
  # @option opts [String] :status Http status
  # @option opts [Object] :serializer child of MiniActiveRecord::Serializer
  # @option opts [Object] :each_serializer Param object should be Array
  # @option opts [String] :root
  # @return [MiniActionController::Response]
  def render_json(object, opts = {})
    status = opts[:status] || MiniActionController::DEFAULT_STATUS
    MiniActionView::Json.new(object).render(opts.merge(status: status))
  end
end
```

The function is just a wrapper for `MiniActionView::Json#render` [method](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_view/json.rb). Layout for JSON response works similar as HTML layout algorithm.

### Assets rendering
The library supports JS and CSS assets rendering. There are `stylesheet_link_tag` and `javascript_include_tag` [methods](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_view/render.rb) in a layout template (ERB file) to render HTML-tags.
```html
# todo_list/app/views/layouts/application.html.erb
<!DOCTYPE html>
<html lang="ru">
<head>
  <title>My TODO list</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <%= stylesheet_link_tag "application" %>
  <%= javascript_include_tag "application" %>
# …
```

A file with stylesheets can look like:
```html
# todo_list/app/assets/stylesheets/application.css.erb
<%= import 'bootstrap' %>
<%= import 'bootstrap_pricing' %>

/* custom styles */
p.disabled {
  color: #6c757d
}
```

It renders partial files `bootstrap` , `bootstrap_pricing`  via `import` method and has its own css-styles. The algorithm of partial files rendering are written in `MiniActionView::Asset` [class](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_view/asset.rb).
```ruby
class MiniActionView::Asset
  # It renders text file and ERB files
  # @param file_path [String]
  # @return [String]
  def render(file_path = nil)
    file_path ||= @original_file_path
    file_context = File.open(file_path).read
    if file_path.to_s =~ /\.erb$/
      ERB.new(file_context).result(binding)
    else
      file_context
    end
  end

  private
  def import(file_name)
    @original_file_path = @current_folder.join("#{file_name}#{@file_extention}")
    # Try to find original file or file with .erb
    if File.exist?(@original_file_path)
      render(@original_file_path)
    elsif File.exist?("#{@original_file_path}.erb")
      render("#{@original_file_path}.erb")
    else
      raise "ERROR: Can not open file '#{@original_file_path}'"
    end
  end
end
```
In assets file we can use `import` method which is just a wrapper for `render` function.

How to handle client request, compile assets and return response? For it there is `MiniActionDispatch::AssetHandler` [rack-middleware](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_action_dispatcher/asset_handler.rb).
```ruby
# Rack-middleware to handler request and render assets
class MiniActionDispatch::AssetHandler
  def initialize(app)
    @app = app
  end

  # NOTE: Attempt to find asset by path.
  # If doesn't find a file, pass the request to another middleware
  def call(env)
    attempt(env) || @app.call(env)
  end

  private
  def attempt(env)
    request = Rack::Request.new(env)
    return nil unless valid_request?(request)

    path_info = request.path_info
    file_path = find_original_file(path_info)
    if file_path.present?
      file_context = ::MiniActionView::Asset.new(file_path).render
      # Build rack answer
      return [200, build_headers(path_info), [file_context]]
    end
    # Return nil in order to pass request to another middleware
    nil
  end
end
```

JS-asset files are rendered absolutely the same way.

## Models layer
In order to implement a models layer I've written the `MiniActiveRecord` module. A standard model looks like:
```ruby
class Item < MiniActiveRecord::Base
  attribute :title, type: String
  attribute :group_id, type: String
  attribute :done, type: [TrueClass, FalseClass], default: false

  validates :title, presence: true, length: { max: 100, min: 3 }
  validates :group_id, presence: true

  belongs_to :group

  scope :active, -> { where(done: false) }
  scope :not_active, -> { where(done: true) }
end
```
There are attributes, validations, relations and scopes. Let's look at the code deeper.

## Attributes
Firstly, I had to define attributes. I wanted to have an interface which is similar to [mongoid](https://github.com/mongodb/mongoid). The code is incapsulated in `MiniActiveRecord::Attribute` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/attribute.rb).
```ruby
module MiniActiveRecord::Attribute
  def self.included(base)
    base.extend ClassMethods
    # Define storage to collect info about all defined attributes.
    base.class_attribute :fields
    base.fields = []

    # Define base attributes.
    base.attribute :id, type: String
    base.attribute :created_at, type: DateTime
  end

  module ClassMethods
    # @param field_name [String, Symbol]
    # @option options [Class, Array<Class>] :type
    # @option options [Object] :default The field's default
    def attribute(field_name, type: String, default: nil)
      new_field_params = { name: field_name.to_sym, type: type, default: default }
      self.fields = fields | [new_field_params]

      instance_eval do
        # Define a getter
        define_method(field_name) do
          field_params = fields.find{ |i| i[:name] == field_name.to_sym }
          instance_variable_get("@#{field_name}") || field_params[:default]
        end

        # Define a setter
        define_method("#{field_name}=") do |value|
          # CODE
          instance_variable_set("@#{field_name}", value)
        end
      end
    end
  end
end
```
I use `class_attribute` feature to collect info about all implemented attributes to `fields` variable. `attribute` method saves data to `fields` variable, defined getter and setter with meta programming.

## Relations
Relations feature is one of the easiest and smallers part. You can see the code inside `MiniActiveRecord::Association` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/association.rb).
```ruby
# NOTE: Module with assiciation logic, such as: has_many, belongs_to, etc.
module MiniActiveRecord::Association
  # Example of usage: has_many :items
  # It will create method .items
  # The method returns Array
  # @param assosiation_name [String, Symbol]
  # @param class_name [String] Model name
  def has_many(association_name, class_name:)
    instance_eval do
      define_method(association_name) do
        another_model = Object.const_get(class_name)
        attribute = "#{self.class.name.downcase}_id"
        another_model.where(attribute.to_sym => id)
      end
    end
  end

  # Example of usage: has_many :user
  # It will create method .user
  # The method returns Object
  # @param assosiation_name [String, Symbol]
  def belongs_to(association_name)
    instance_eval do
      define_method(association_name) do
        class_name = association_name.to_s.camelize
        another_model = Object.const_get(class_name)
        refer_id = public_send("#{association_name}_id")
        another_model.find(refer_id)
      end
    end
  end
end
```
I've written `has_many` and `belongs_to` relations. Both methods create other methods with meta programming. The logic is clear and it doesn't work like a magic.

## Validations
So, we have attributes and relations. It's time for validations. The code is written inside `MiniActiveRecord::Validation` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/validation.rb).
```ruby
module MiniActiveRecord::Validation
  def self.included(base)
    base.class_attribute :validations
    base.validations = []
    base.extend ClassMethods
  end

  module ClassMethods
    # Example of usage:
    #   validates :title, presence: true
    #   validates :title, length: { max: 100, min: 3 }
    # @param field_name [String, Symbol]
    # @param presence [Boolean]
    # @param length [Hash]
    # @option length [Number] :max
    # @option length [Number] :min
    def validates(field_name, presence: nil, length: {})
      if presence.present?
        validates_presence_of(field_name)
      end
      if length.present?
        validates_length_of(field_name, max: length[:max], min: length[:min])
      end
    end

    # @param field_name [String, Symbol]
    def validates_presence_of(field_name)
      new_validation = { field_name: field_name.to_sym, type: :presence_of }
      self.validations = validations | [new_validation]
    end

    def validates_length_of(field_name, max: nil, min: nil)
      new_validation = { field_name: field_name.to_sym, type: :length_of, max: max, min: min }
      self.validations = validations | [new_validation]
    end
  end
end
```

I again use `class_attribute`  feature in order to store meta data about all validations in the `validations` variable. Then I defined two methods for validations `validates_presence_of` , `validates_length_of` and wrote a wrapper `validates` to use it easily.

Each validation class is an implementation of visitor OOP-pattern. It makes it easy to read the code and add one more validation.
```ruby
# validation/presence_of_validation
# NOTE: Validation class that checks existence of a value
class MiniActiveRecord::Validation::PresenceOfValidation < BaseValidation
  def call
    value = object.public_send(field_name)
    return true if value.present?

    object.errors.add(field_name, 'must be present')
  end
end

# validation/length_of_validation
# NOTE: Validation class that checks string length
class MiniActiveRecord::Validation::LengthOfValidation < BaseValidation
  def call
    value = object.public_send(field_name)
    return if value.nil?

    max = meta_data[:max]
    min = meta_data[:min]
    if max.present? && value.size > max
      object.errors.add(field_name, "must be less then #{max}")
    end
    if min.present? && value.size < min
      object.errors.add(field_name, "must be greater then #{min}")
    end
  end
end
```

How to run all validations? The algorithm is the same as in RubyOnRails, method `valid?` runs all validations.
```ruby
module MiniActiveRecord::Validation
  def valid?
    validate!
    errors.size == 0
  end

  # Runs all validations
  def validate!
    # 1: Init new error-object
    @errors_object = ::MiniActiveRecord::Validation::Errors.new
    validation_namespace = ::MiniActiveRecord::Validation
    # 2: Run each validation
    self.validations.each do |validation|
      class_object = "#{validation[:type].to_s.camelize}Validation"

      if validation_namespace.const_defined?(class_object)
        klass = validation_namespace.const_get(class_object)
        klass.new(validation, self).call
      else
        raise "ERROR: Can not find class #{validation_namespace}::class_object"
      end
    end
    self
  end

  # @return [MiniActiveRecord::Validation::Errors]
  def errors
    @errors_object
  end
end
```
User can explicitly run `valid?` method, or the library can do it implicitly using `save`, or `save!` methods. Example:
```ruby
# Code in controller
def create
  @item = Item.new(permited_params)
  if @item.save
    redirect_to "/groups/#{@group.id}/items"
  else
    render :new, status: '422 Unprocessable Entity'
  end
end

module MiniActiveRecord::Operate
  # @return [Boolean]
  def save
    return false unless valid?
    # CODE is here
    true
  end

  # @return [Boolean]
  # @raise MiniActiveRecord::RecordInvalid
  def save!
    if save
      true
    else
      raise MiniActiveRecord::RecordInvalid
    end
  end
end
```

## Scopes and querying
Querying is a very important part of the models layer. We are used to write code like this:
```ruby
Group.where(id: '123')
Item.where(group_id: '123')
Item.find('123)
```
Scoping is also very important. We are used to write code in a model like that, instead of defining new method.
```ruby
class Item < MiniActiveRecord::Base
  attribute :done, type: [TrueClass, FalseClass], default: false

  scope :active, -> { where(done: false) }
  scope :not_active, -> { where(done: true) }
end
```

Under the hood, the scope-feature just a stroring meta data about each ruby `Proc` in a global variable for future use and define new singleton method in the model class. The source code is incapsulated in `MiniActiveRecord::Scope` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/scope.rb).
```ruby
module MiniActiveRecord::Scope
  def self.included(base)
    base.extend ClassMethods
    base.class_attribute :scopes
    base.scopes = []
  end

  module ClassMethods
    # @param name [String, Symbol]
    # @param procc [Proc]
    def scope(name, procc)
      new_scope_params = { name: name.to_sym, proc: procc }
      self.scopes = scopes | [new_scope_params]
      self.class_eval do
        define_singleton_method(name, &procc)
      end
    end
  end
end
```
I again use `class_attribute` feature and store data in `scopes` variable for future use. There is no magic. The magic you will see below, querying should support chain methods like:
```ruby
Group.first.items.active.count
Group.first.items.where(done: false).count
Record.where(a: 'foo').where(b: 'bar')
```
In order to do it I wrote  `MiniActiveRecord::Relation` [module](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/relation.rb) and defined methods as `all`, `where`, `find_by` etc.
```ruby
module MiniActiveRecord::Relation
  # @param conditions [Hash<Symbol, Object>] Object could be String, Integer, Array
  # @return [MiniActiveRecord::Proxy]
  def where(conditions = {})
    init_proxy.where(conditions)
  end

  # @param conditions [Hash<Symbol, Object>] Object could be String, Integer, Array
  # @return [MiniActiveRecord::Base]
  def find_by(conditions)
    where(conditions).first
  end

  # @param conditions [Hash<Symbol, Object>] Object could be String, Integer, Array
  # @return [Object]
  # @raise [MiniActiveRecord::RecordNotFound]
  # @return [MiniActiveRecord::Base]
  def find_by!(conditions)
    item = where(conditions).first
    raise ::MiniActiveRecord::RecordNotFound if item.nil?
    item
  end

  # @return [MiniActiveRecord::Proxy]
  def all
    where({})
  end

  # @return [MiniActiveRecord::Base]
  # @raise [MiniActiveRecord::RecordNotFound]
  def find(selected_id)
    find_by!({id: selected_id})
  end

  private
  def init_proxy
    # proxy_class is a child of MiniActiveRecord::Proxy
    self.proxy_class.new({})
  end
end
```
The main method is `where`, other functions are just wrappers for the function. The `where` method passes all arguments to  `MiniActiveRecord::Proxy` [class](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/proxy.rb). The proxy class is similar to `ActiveRecord::Associations::CollectionProxy`. It's an implementation of the proxy patter and it's useful for chain methods.
```ruby
class MiniActiveRecord::Proxy
  # @param where_condition [Hash]
  def initialize(where_condition = {})
    @where_condition = where_condition.transform_keys(&:to_sym)
    @limit = nil
  end

  # @return [MiniActiveRecord::Proxy]
  def all
    where({})
    self
  end

  # @return [MiniActiveRecord::Proxy]
  def where(conditions)
    @where_condition.merge!(conditions.transform_keys(&:to_sym))
    self
  end

  private
  def method_missing(message, *args, &block)
    # The magic is here
  end
end
```
Each public method returns  `self`, it supports the chain methods. The main magic locates inside `method_missing`. Firstly, the methods chain should support scopes I stored in `scopes` variable. Secondly, a proxy should extract data from a database and return an array of `MiniActiveRecord::Base` objects for us. It's urgent for code like that:
```ruby
class Item < MiniActiveRecord::Base
  scope :active, -> { where(done: false) }
  scope :not_active, -> { where(done: true) }
end

# .each and .map are methods of Array, not MiniActiveRecord::Proxy
Items.all.active.each { |i| puts i.id }
Items.where(a: 'foo').map { |i| i.do_something }
```

I wrote the `method_missing` method which tries to find a scope (actual for `active`, `not_active` scopes). If there is no scope, it passes the data to DB driver and wrap raw data into `MiniActiveRecord::Base` (it's our models `Item`, `Group`, etc).
```ruby
class MiniActiveRecord::Proxy
  # NOTE: Use it in order to exec driver even before data manitulation
  # It pass methods to Array<ActiveRecord::Base>
  # For example:
  # .where().each {}
  # .where().where().map {}
  def method_missing(message, *args, &block)
    # 1: Try to find scope in the model class
    scope_meta = model_class.scopes.find{ |i| i[:name] == message }
    if !scope_meta.nil?
      instance_exec(&scope_meta[:proc])
    else
      # 2: Execute and find method
      execute.public_send(message, *args, &block)
    end
  end

  # Run driver and wrap raw data to a model-class
  # @return [Array<ActiveRecord::Base>]
  def execute
    raw_data = driver.where(@where_condition, table_name, @limit)
    raw_data.map { |data| model_class.new(data) }
  end
end
```

## Data storages
Inspired by Maple Ong's talk, I keep using a yaml-file storage. With the template method OOP-pattern, it's possible to write a driver for different databases in future such as: MongoDB, SQL or Redis. But now I'm good with just a yaml file.

Firstly I implemented an abstract class `MiniActiveRecord::Driver` with [interface](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/driver.rb).
```ruby
# NOTE: Abstract class.
# Inherite from the class in order to implement a driver for different DBs.
class MiniActiveRecord::Driver
  class << self
    # @param table_name [String]
    def all(table_name)
      _all(table_name)
    end

    # @param conditions [Hash<Symbol, Object>]
    # @param table_name [String]
    # @param limit [Integer]
    def where(conditions, table_name, limit)
      _where(conditions, table_name, limit)
    end

    # @param selected_id [String, Integer]
    # @param table_name [String]
    def find(selected_id, table_name)
      _find(selected_id, table_name)
    end
    # CODE
  end
end
```

Then, I inherited `MiniActiveRecord::YamlDriver` [class](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_active_record/yaml_driver.rb) with low-level code described how to store and manipulate the data from a yaml file.
```ruby
# NOTE: Driver for YAML-file local data storage
class MiniActiveRecord::YamlDriver < MiniActiveRecord::Driver
  class << self
    private
    def _all(table_name)
      _where({}, nil, table_name)
    end

    def _find(selected_id, table_name)
      _where({id: selected_id}, 1, table_name)
    end

    def _where(conditions, table_name, limit = nil)
      store = init_store(table_name)
      store.transaction do
        memo = store[table_name.to_sym]
        memo = conditions.reduce(memo) do |memo, (cond_key, cond_value)|
          if cond_value.is_a?(Array)
            memo.select{ |i| cond_value.include?(i[cond_key])  }
          else
            memo.select{ |i| i[cond_key] == cond_value }
          end
        end
      memo
    end

    def init_store(table_name)
      full_file_path = MiniRails.root.join("db/db_#{table_name}.yml")
      YAML::Store.new(full_file_path)
    end
  end
end
```

## Testing
These three MVC layers and a rack-middleware are enough to write an application. In order to prove that it works, I had to write a library for testing. I had an idea to write something like RSpec from scratch.

### Fabrics
Firstly I had to implement a library for fabrics such as `FactoryBot`. My goal was to implement a factory definition with `sequence` and `trait`. I called it `MiniFactory` and I wanted be able to write the code as:
```ruby
# todo_list/spec/factories/item_factory.rb
MiniFactory.define do
  factory :item, class: 'Item' do
    sequence(:title) { |i| "title_#{i}" }
    done { false }

    trait :done do
      done { true }
    end
  end
end
```
Start with the factory definition. `MiniFactory::define` is just a syntax sugar for `MiniFactory::Base` .
```ruby
module MiniFactory
  class << self
    def define(&block)
      Base.instance.instance_exec(&block)
    end
  end
end
```
`MiniFactory::Base` [class](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_factory/base.rb) is a singleton object which stores the information about defined factories in the  `@factories` variable.
```ruby
class MiniFactory::Base
  include Singleton
  def initialize
    @factories = {} # Hash to store data about all factories
  end

  # @param factory_name [String, Symbol]
  # @param opts [Hash<Symbol, Object>]
  # @option opts [Object, String] :class
  def factory(factory_name, opts, &block)
    buidler = Builder.new(opts[:class])
    buidler.instance_exec(&block)
    @factories[factory_name.to_sym] = { count: 0, buidler: buidler }
  end
end
```

In order to define various model attributes I wrote `MiniFactory::Builder` [class](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_factory/builder.rb) which is similar to the factory OOP-pattern. Look at example below, each model has different attributes such as `title` and `done` for item factory. How to support it?
```ruby
MiniFactory.define do
  factory :item, class: 'Item' do
    title { 'Hello' }
    done { false }
  end
end
```
There is the  `method_missing` method to cope with it. It stores a model attribute name in the `@attributes` variable and also stores the Proc.
```ruby
class MiniFactory::Builder
  def initialize(klass)
    @klass = klass
    @attributes = {}
  end

  def method_missing(message, *args, &block)
    @attributes[message] = { type: :attribute, block: block }
  end
end
```
So, I have all information to create a factory, let's do it. In order to build a factory, I've written the code:
```ruby
MiniFactory.build(:item)

module MiniFactory
  # @param factory_name [Symbol, String]
  # @param traits_and_opts [Array]
  def self.build(factory_name, *traits_and_opts)
    opts = traits_and_opts.extract_options!
    traits = traits_and_opts.map(&:to_sym)
    Base.instance.build_factory(factory_name, traits, opts)
  end
end

class MiniFactory::Base
  # @param factory_name [String, Symbol]
  # @param traits [Array<Symbol>]
  # @param opts [Hash<Symbol, Object>]
  def build_factory(factory_name, traits, opts)
    # Find MiniFactory::Builder instance
    buidler = @factories[factory_name.to_sym][:buidler]
    # Build new object
    buidler.build_object(number, traits, opts)
  end
end

class MiniFactory::Builder
  # @param number [Integer] Params for sequences
  # @param selected_traits [Array<Symbol>]
  # @param opts [Hash<Symbol, Object>]
  def build_object(number = 1, selected_traits = [], opts = {})
    @klass = Object.const_get(@klass)
    # 1: Collect all attributes
    all_attributes = (opts.keys + @attributes.keys).uniq

    # 3: Assign data from opts params or fetch it from stored proc
    attrs = all_attributes.reduce({}) do |memo, key|
      memo[key] = (opts.key?(key) ? opts[key] : @attributes[key][:block].call)
      memo
    end
    # 4: Assign data to a model instance
    @klass.new(**attrs)
  end
end
```

Let's add `sequence` feature. The `Sequence` is a method for a factory class. It stores data as I wrote in `method_missing` but with different type. And let's improve `fetch_params` method to pass a number to sequence's Proc.
```ruby
class MiniFactory::Builder
  # Example of usage:
  # senquence(:title) { |i| "My title ##{i}" }
  # @param attr_name [String, Symbol]
  def sequence(attr_name, &block)
    @attributes[attr_name.to_sym] = { type: :sequence, block: block }
  end

  def fetch_params(params, number)
    if params[:type] == :attribute
      params[:block].call
    else
      # If the attribute is sequence, pass a number
      params[:block].call(number)
    end
  end
end
```

What about `trait` feature? Trait is a factory inside a factory :-)
```ruby
class MiniFactory::Builder
  def initialize(klass)
    @traits = {}
  end

  # @param trait_name [String, Symbol]
  def trait(trait_name, &block)
    trait_builder = self.class.new(@klass)
    trait_builder.instance_exec(&block)
    @traits[trait_name.to_sym] = trait_builder.attributes
  end
end
```

### Testing and specs
I decided to implement the core functional of RSpec and [called it](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_r_spec.rb) `MiniRSpec`. Firstly, I had to implement a test tree feature. It's a bunch of methods like `context`, `describe`, and `it`. With these methods I'm able to write a test tree.
```ruby
MiniRSpec.describe 'Item' do
  describe 'context 1' do
    it 'works' {}
    describe 'context 1.1' do
      it 'works' {}
    end
  end
  describe 'context 2' do
    it 'works' {}
  end
end
```
It was a real challenge. The structure of the code is an AST (abstract syntax tree), therefore there are 3 types of AST leaves:
* Main leaf  `MiniRSpec.describe`
* Context leaf  `context` and `describe`
* Test leaf  `it`

Let's begin with a data storage and the main leaf.
```ruby
# mini_rails/mini_r_spec/base.rb
# Singleton class to store test data as AST in @ast variable
class MiniRSpec::Base
  attr_accessor :ast
  include ::Singleton

  def initialize
    @ast = []
  end
end

# mini_rails/mini_r_spec.rb
# The main leaf
module MiniRSpec
  # Initialize describe leaf and store data in the base object
  def self.describe(title, &block)
    unit = Base.instance
    leaf = DescribeLeaf.new(title)
    leaf.instance_exec(&block)
    unit.ast << leaf
    unit
  end
end
```

The algorithms of `context` and `it` leaves are similar, therefore I implement `MiniRSpec::Context` module to include it.
```ruby
# mini_rails/mini_r_spec/context.rb
# NOTE: There is the main logic for describe and it leaves
module MiniRSpec::Context
  # @param described_object [String, Object] Object must respond to method .to_s
  # @return [DescribeLeaf]
  def describe(described_object, &block)
    leaf = DescribeLeaf.new(described_object)
    leaf.instance_exec(&block) if block_given?
    leaf
  end
  alias_method :context, :describe

  # @param described_object [String, Object] Object must respond to method .to_s
  # @return [ItLeaf::Base]
  def it(described_object, &block)
    leaf = ItLeaf::Base.new(described_object)
    leaf.proc = block
    leaf
  end
end

# mini_rails/mini_r_spec/describe_leaf.rb
# NOTE: `describe` leaf
class MiniRSpec::DescribeLeaf
  include Context
  def initialize(title)
    @title = title
    @children = []
  end

  # @described_object [String, Object] Object must respond to method .to_s
  # @return [DescribeLeaf]
  def describe(described_object, &block)
    leaf = super
    @children << leaf
    nil
  end
  # NOTE: Rewrite aliase
  alias_method :context, :describe

  # @described_object [String, Object] Object must respond to method .to_s
  # @return [ItLeaf::Base]
  def it(described_object = '', &block)
    leaf = super
    @children << leaf
    nil
  end
end

# NOTE: `it` leaf
# mini_rails/mini_r_spec/it_leaf/base.rb
class MiniRSpec::ItLeaf::Base
  include Context
  def initialize(title)
    @title = title
    @proc = nil
  end

  def describe(described_object)
   raise 'ERROR: Can not use describe inside "it" block'
  end

  # NOTE: Rewrite aliase
  alias_method :context, :describe
end
```

The AST is completed. How to run test cases? I had to traverse AST and run each ruby `Proc` in `it` leaf.
```ruby
# mini_rails/mini_r_spec/base.rb
# Singleton class to store test data as AST in @ast variable
class MiniRSpec::Base
  def run_tests
    @ast.each do |node|
    if node.is_a?(ItLeaf::Base)
     node.run_tests
    elsif node.is_a?(DescribeLeaf)
      node.run_tests
    end
  end
end

# mini_rails/mini_r_spec/describe_leaf.rb
# NOTE: `describe` leaf
class MiniRSpec::DescribeLeaf
  # @param context [String]
  def run_tests(context = nil)
    context = [context, title].compact.join(' ')
    children.each do |node|
      if node.is_a?(ItLeaf::Base)
        node.run_tests(context)
      elsif node.is_a?(DescribeLeaf)
        node.run_tests(context)
      end
    end
  end
end

# mini_rails/mini_r_spec/it_leaf/base.rb
# NOTE: `it` leaf
class MiniRSpec::ItLeaf::Base
  # @param context [String]
  def run_tests(context = nil)
    context = [context, title].compact.join(' ')
    return nil if @proc.nil?

    # Clear DB before running the test case
    ::MiniActiveRecord::Base.driver.destroy_database!

    # Run the test case
    instance_exec(&@proc)
    TestManager.instance.add_success(context)
  rescue ::StandardError => e
    TestManager.instance.add_failure(context, e)
  end
end
```
It works, but there is only core features.

### Variables and callbacks
Now I can write and run tests cases but there will be a lot amount of the same code in the test cases. All tests should be DRY, helpers as `let`, `let!` and callbacks as `before_each` help with it. I decided to implement `let!` and `before_each` features for the `describe` leaf to save data in instance variables.
```ruby
# mini_rails/mini_r_spec/describe_leaf.rb
# NOTE: `describe` leaf
class MiniRSpec::DescribeLeaf
  def initialize(title)
    # CODE
    @callbacks = [] # Array for before_each callbacks
    @variables = {} # Hash for let! data
  end

  # @param variable_name [String, Symbol]
  def let!(variable_name, &block)
    @variables[variable_name.to_sym] = block
  end

  def before_each(&block)
    @callbacks.push(block)
  end
end
```

Then I extended  `run_tests` method to collect and pass variables and callbacks to the `it` block.
```ruby
# mini_rails/mini_r_spec/describe_leaf.rb
# NOTE: `describe` leaf
class MiniRSpec::DescribeLeaf
  # @param context [String]
  # @param before_callbacks [Array<Proc>]
  # @param variables [Hash<Symbol, Proc>]
  def run_tests(context = nil, before_callbacks = [], variables = {})
    # CODE
    merged_callbacks = before_callbacks + @callbacks
    merged_variables = variables.merge(@variables)

    children.each do |node|
      if node.is_a?(ItLeaf::Base)
        node.run_tests(context, merged_callbacks, merged_variables)
      elsif node.is_a?(DescribeLeaf)
        node.run_tests(context, merged_callbacks, merged_variables)
      end
    end
  end
end
```

Further I extended the `it` leaf to run callbacks and receive data from `let!` handlers.
```ruby
# mini_rails/mini_r_spec/it_leaf/base.rb
# NOTE: `it` leaf
class MiniRSpec::ItLeaf::Base
  def initialize(title)
    # CODE
    @variables = {} # Hash for let! data
  end

  # @param context [String]
  # @param before_callbacks [Array<Proc>]
  # @param variables [Hash<Symbol, Proc>]
  def run_tests(context = nil, before_callbacks = [], variables = {})
    # CODE
    # Run all let! blocks
    variables.each do |var_name, proc|
      @variables[var_name] = instance_exec(&proc)
    end

    # Run all before-callbacks
    before_callbacks.each do |callback|
      instance_exec(&callback)
    end

    # Run the test case
    instance_exec(&@proc)
    # CODE
  rescue ::StandardError => e
    # CODE
  end

  def method_missing(message, *args, &block)
    if @variables.key?(message)
      # let! variables support
      @variables[message]
    else
      super
    end
  end
end
```

### Test matching
Final point is to implement a test matching such as:
```ruby
expect(1).to eq(1)
expect(1).not_to eq(2)
expect(1).to be_present
expect(true).to be_truthy
expect([1,2,3]).to include(2)
```

In order to implement it I [wrote the](https://github.com/kopylovvlad/mini_rails/blob/master/mini_rails/mini_r_spec/matchers.rb) `MiniRSpec::Matcher` as the template method OOP-pattern. Under the hood the code above is just syntax sugar for the code below:
```ruby
EqMatcher.new(1) == Matcher.new(1)
EqMatcher.new(2) != Matcher.new(1)
BePresentMatcher.new == Matcher.new(1)
EqMatcher.new(true) == Matcher.new(true)
IncludeMatcher.new(2) == Matcher.new([1,2,3])
```

Source code looks like this:
```ruby
# mini_rails/mini_r_spec/matchers.rb
class MiniRSpec::Matcher
  def initialize(value = nil)
    @value = value
  end

  def to(matcher)
    matcher == @value
  end

  def not_to(matcher)
    matcher != @value
  end
end

class MiniRSpec::EqMatcher < Matcher
  def ==(new_value)
    a = @value == new_value
    raise MatchError, "'#{new_value}' does not equal '#{@value}'" if a == false
    a
  end

  def !=(new_value)
    a = @value != new_value
    raise MatchError, "'#{new_value}' equals '#{@value}'" if a == false
    a
  end
end
```

## Conclusions
Write something from scratch is fun. The challenge to write a framework was absolutely new experience to me. After working on it I noticed some useful things and whould like to share my thoughts with you.

###  `class_attribute`
It's very [cool feature](https://apidock.com/rails/Class/class_attribute) to share data between class and class instance. In the article and in the source code you see that I often use it to implement some features.

### A function wrapper
A usable interface is very important when you write a library. A function wrapper can reduce amount of code and make it easy to write and read. Example for validation module:
```ruby
module MiniActiveRecord::Validation::ClassMethods
  # A function wrapper
  def validates(field_name, presence: nil, length: {})
    if presence.present?
      validates_presence_of(field_name)
    end
    if length.present?
      validates_length_of(field_name, max: length[:max], min: length[:min])
    end
  end

  def validates_presence_of(field_name)
    # Core function
  end

  def validates_length_of(field_name, max: nil, min: nil)
    # Core function
  end
end
```

Also ActiveSupport's `extract_options!` method is also useful to make the interface flexible. You can see below an example for factories. `build` functions receive factory name, traits and options.
```ruby
module MiniFactory
  class << self
	  # Example of usage: build(:factory_name, :trait1, attr1: ''1)
    #   build(: factory_name, :trait1, :trait2, attr1: '1')
    #   build(: factory_name, attr1: '1')
    def build(factory_name, *traits_and_opts)
      opts = traits_and_opts.extract_options!
      traits = traits_and_opts.map(&:to_sym)
      Base.instance.build_factory(factory_name, traits, opts)
    end
  end
end
```

### Use recursion to write a flexible inferface
Data in function's argument can be flexible. There is an example for render module below.  `view_name` params can be a string, a symbol and can be without `_`. In order to implement it easily I used recursion.
```ruby
module MiniActionView::Render
  # @param view_name [String, Symbol]
  #   Can be: '_header', :header, 'shared/header'
  #   For symbol it adds _ in the beggining of name.
  # @param collection [Array<Object>] each item will be passed as 'item' variable
  # @param locals [Hash<Symbol,Object>] params to passing data as local_variables
  def render_partial(view_name, collection: [], locals: {})
    if view_name.is_a?(Symbol)
      render_partial("_#{view_name}", collection: collection, locals: locals)
    elsif view_name.exclude?('.html.erb')
      render_partial("#{view_name}.html.erb", collection: collection, locals: locals)
    elsif collection.size > 0
      collection.map { |i| render_view(view_name, locals: {item: i}) }.join('')
    else
      render_view(view_name, locals: locals)
    end
  end
end
```

[dev.to](https://dev.to/kopylov_vlad/crafting-mini-rubyonrails-118l)
