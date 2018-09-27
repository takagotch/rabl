### rabl
---
https://github.com/nesquena/rabl

```sh
gem install rabl
gem 'rabl'
gem 'oj'
bundle install
Rabl.register!
```

```ruby
app/app.rb
get "/posts", :provides => [] do
  @user = current_user
  @posts = Post.order("id DESC")
  render "posts/index"
end

app/views/posts/index.rabl
collection @posts
attributes :id, :subject
child(:user) { attributes :full_name }
node(:read) { |post| post.read_by?(@user) }


```

```
http://localhost:3000/posts.json
[{  "post" :
  {
    "id" : 5, "title": "...", "subject": "...",
    "user" : { "full_name" : "..." },
    "read" : true
  }
}]
```

```ruby
config/initializers/rabl_init.rb
require 'rabl'
Rabl.configure do |config|
  # config.cache_all_output = false
  # config.cache_sources = Rails.env != 'development'
  # config.cache_engine = Rabl::CacheEngine.new
  # config.perform_caching = false
  # config.escape_all_output = false
  # config.json_engine = nil
  # config.msgpack_engine = nil
  # config.bson_engine = nil
  # config.plist_engine = nil
  # config.include_json_root = true
  # config.include_msgpack_root = true
  # config.include_plist_root = true
  # config.include_xml_root = false
  # config.include_child_root = true
  # config.enable_json_callbacks = false
  # config.xml_options = { :dasherize => true, :skip_types => false }
  # config.view_paths = []
  # config.raise_on_missing_attribute = true
  # config.replace_nil_values_with_empty_strings = true
  # config.replace_empty_string_values_with_nil_values = true
  # config.exclude_nil_values = true
  # config.exclude_empty_values_in_collections = true
  # config.camelize_keys = :upper
end



gem 'oj'
config.json_engine = ActiveSupport::JSON

# app/views/users/show.json.rabl
object @user
object @user => :person
# => { "person" : {...} }
collection @users
# => [ { "user" : {...} } ]
collection @users => :people
# => { "people" : [ { "person" : {...} } ] }
collection @users, :root => "people", :object_root => "user"
# => { "people" : [ { "user" : {...} } ] }
collection @users, :root => "people", :object_root => false
# => { "people" : [ {...}, {...} ] }

object false
node(:some_count) { |m| @user.posts.count }
child(@user){ attribute :name }

# app/views/users/show.json.rabl
attributes :id, :foo, :bar
attribute :foo => :bar
# => { bar : 5 }

attributes :bar => :baz, :dog => :animal
# => # { baz : <bar value>, animal : <dog value> }
attributes :foo, :bar, :if => lambda { |m| m.condition? }
attributes :foo, :bar => :baz

attributes :foo
attributes :bar => :baz

child :address do
  attributes :street, :city, :zip, :state
end

child @posts => :foobar do
  attributes :id, :title
end

child :posts => :foobar do
  attributes :id, :title
end

object @user
child :posts do |user|
  attribute :title unless user.suspended?
end

glue @post do
  attributes :id => :post_id, :name => :post_name
end

object @user
glue(@post) {|user| attribute :title if user.active? }

# app/views/users/show.json.rabl
node :full_name do |u|
  u.first_name + " " + u.last_name
end

node do |u|
  { :full_name => u.first_name + " " + u.last_name }
  # => { full_name : "Tky Takagotch" }
end

node :location do
  { :city => @city, :address => partial("users/address", :object => @address) }
end

node :location do |m|
  { :city => m.city, :address => partial("users/address", :object => m.address) }
end

# app/views/users/advanced.json.rabl
extends "users/base"
node :can_drink do |m|
  m.age > 21
end

# app/views/users/show.json.rabl
child @address do
  extends "address/item"
end

# app/views/posts/index.json.rabl
collection @posts
extends('posts/show', :locals => { :hide_comments => true } )
# node(false) { |post| partial('posts/show', :object => :post, :locals => { :hide_comments => true })}

#app/views/posts/show.json.rabl
object @post
attributes :id, :title, :body, :created_at
node(:comments){ |post| post.comments } unless locals[:hide_comments]

collection @posts
node(:coolness, :if => lambda { |m| m.coolness > 5 }) do |m|
  m.coolness
end

collection @posts
attribute(:coolness, :if => lambda { |m| m.coolness > 5 })

class Post
  def cool?
    coolness > 5
  end
end

collection @posts
attribute :coolness, if: :cool?

# app/some/template.rabl
object @post
child(@user => :user){...}
node(Lformatted_body){ |post| simple_format(post.body) }

# app/views/quizzes/show.json.rabl
object @quiz
attribute :title
child :questions do
  attribute :caption
  child :answers do
    extends "answers/item"
  end
end


# app/views/users/show.json.rabl
object @quiz
cache @quiz # key = rabl/quiz/[cache_key]
attribute :title

Rabl.render(object, template, :view_path => 'app/views', :format => :json)
# => "{...json...}"

Rabl::Renderer.new('post/show', @post, :view_path => 'app/views', :format => 'hash').render

Rabl::Renderer.new('post/show', @post, :locals => { :custom_title => "Hello world" })

attribute :content
node(:title) { locals[:custom_title] }




```






















