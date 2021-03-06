![Playlyfe Ruby SDK](https://dev.playlyfe.com/images/assets/pl-ruby-sdk.png "Playlyfe Ruby SDK")

Playlyfe Ruby SDK [![Gem Version](https://badge.fury.io/rb/playlyfe.svg)](http://badge.fury.io/rb/playlyfe)
=================
This is the official OAuth 2.0 Ruby client SDK for the Playlyfe API.
It supports the `client_credentials` and `authorization code` OAuth 2.0 flows.
For a complete API Reference checkout [Playlyfe Developers](https://dev.playlyfe.com/docs/api) for more information.

> Note: Breaking Changes the API has changed a lot from the previous version. It doesn't use named parameters anymore. You have to pass in all the params accordingly. Like method, route, query, body.

ex:
```ruby
    playlyfe = Playlyfe.new(
      version: 'v1'
      client_id: 'Your Playlyfe game client id',
      client_secret: 'Your Playlyfe game client secret',
      type: 'client'
    )
```

Requires
--------
Ruby >= 1.9.3

Install
----------
```ruby
gem install playlyfe
```
or if you are using rails
Just add it to your Gemfile
```ruby
gem 'playlyfe'
```
and do a bundle install

# Examples
The Playlyfe class allows you to make rest api calls like GET, POST, .. etc
**For v1 routes**
```ruby
playlyfe = Playlyfe.new(
  version: 'v1'
  client_id: 'Your Playlyfe game client id',
  client_secret: 'Your Playlyfe game client secret',
  type: 'client'
)
# To get infomation of the player johny
player = playlyfe.get('/player', { player_id: 'johny' })
puts player['id']
puts player['scores']

# To get all available processes with query
processes = playlyfe.get('/processes', { player_id: 'johny' })
puts processes
#To play a process
playlyfe.post("/processes/#{@process_id}/play", { player_id: 'johny' }, { trigger: "#{@trigger}" })
```
**For v2 routes**
```ruby
pl = Playlyfe.new(
  client_id: 'Your Playlyfe game client id',
  client_secret: 'Your Playlyfe game client secret',
  type: 'client'
)
# To get infomation of the player johny
player = pl.get('/runtime/player', { player_id: 'johny' })
puts player['id']
puts player['scores']
```

Using
-----
### Create a client
  If you haven't created a client for your game yet just head over to [Playlyfe](http://playlyfe.com) and login into your account, and go to the game settings and click on client.

## 1. Client Credentials Flow
In the client page select Yes for both the first and second questions
![client](https://cloud.githubusercontent.com/assets/1687946/7930229/2c2f14fe-0924-11e5-8c3b-5ba0c10f066f.png)

In your Application class add this so the Playlyfe SDK will be initialized at the start of your app. A typical rails app using client credentials code flow would look something like this
**config/application.rb**
This is where you will initialize the sdk with your client_id and client_secret
```ruby
class Application < Rails::Application
    playlyfe = Playlyfe.new(
      client_id: 'Your Playlyfe game client id',
      client_secret: 'Your Playlyfe game client secret',
      type: 'client'
    )
end
```
**controllers/welcome_controller.rb**
This is where we make an api request to the Playlyfe Platform the fetch all the players and display them
```ruby
class WelcomeController < ApplicationController
  def index
    @players = pl.get('/runtime/players', { player_id: 'johny' })
  end
end
```
**views/welcome/index.html.erb**
This will display the index page and list all the players in the game
```ruby
<div class="container">
<div class="panel panel-default">
<% @players["data"].each do |player| %>
    <li class="list-group-item">
      <p>
        <%= player["id"] %>
        <%= player["alias"] %>
      </p>
    </li>
<% end %>
</div>
</div>
```
## 2. Authorization Code Flow
In the client page select yes for the first question and no for the second
![auth](https://cloud.githubusercontent.com/assets/1687946/7930231/2c31c1fe-0924-11e5-8cb5-73ca0a002bcb.png)
```ruby
Playlyfe.new(
  client_id: 'Your Playlyfe game client id',
  client_secret: 'Your Playlyfe game client secret'
  type: 'code'
  redirect_uri: 'https://example.com/oauth/callback'
)
```
In this flow then you need a view which will allow your user to redirect to the login using the playlyfe platform. In that route you will get the authorization code so that the sdk can get the access token
```ruby
exchange_code(code)
```

Now you should be able to access the Playlyfe api across all your
controllers.
A typical rails app using authorization code flow would look something like this
**config/application.rb**
This is where you will initialize the sdk with your client_id and client_secret
```ruby
class Application < Rails::Application
    playlyfe = Playlyfe.new(
      client_id: "",
      client_secret: "",
      type: 'code',
      redirect_uri: 'http://localhost:3000/welcome/index'
    )
end
```
**controllers/welcome_controller.rb**
This is where we check if the user successfully logged in and set the authorization code using Playlyfe.exchange_code
```ruby
class WelcomeController < ApplicationController

  def index
    if params['code'].nil?
      puts 'login again'
      # here you need to add some logic if the user is logged in and
      # you need to redirect to the playlyfe login page if needed
    else
      pl.exchange_code(params['code'])
      redirect_to :action => 'home'
    end
  end

  def home
    @players = pl.get('/runtime/players', { player_id: 'johny' })
  end
end
```
**views/welcome/index.html.erb**
This is the main index page. It will redirect the user to login into playlyfe
```ruby
<div class="container">
<div class="form-signin">
Please sign in using the Playlyfe Platform
<a href=<%= pl.get_auth_url() %>> Login </a>
</div>
</div>
```
**views/welcome/home.html.erb**
This will display the home page after a user logs in. Here it displays all the players in the game
```ruby
<div class="container">
<div class="panel panel-default">
<% @players["data"].each do |player| %>
    <li class="list-group-item">
      <p>
        <%= player["id"] %>
        <%= player["alias"] %>
      </p>
    </li>
<% end %>
</div>
</div>
```
**config/routes.rb**
There are 2 routes here
The index route is for the login and the home route is for the user after logging in
```ruby
Rails.application.routes.draw do
  root 'welcome#index'
  get 'welcome/index'
  get 'welcome/home'
end
```
**Images**
For Images create a proxy route which can be used to get the images
and you can directly refer to the urls of the image
```ruby
def image
    response = pl.get_raw("/runtime/assets/players/#{session[:player_id]}", { player_id: session[:player_id] }
    )
    send_data response, :type =>'image/png', :disposition => 'iniline'
end

# In routes.rb add
get 'image/:filename' => 'welcome#image'

# Use it in your views like
<%= image_tag "/image/user", :align => "left" %//>
```
## 3. Custom Login Flow using JWT(JSON Web Token)
In the client page select no for the first question and yes for the second
![jwt](https://cloud.githubusercontent.com/assets/1687946/7930230/2c2f2caa-0924-11e5-8dcf-aed914a9dd58.png)
```ruby
token = Playlyfe.createJWT(
    client_id: 'your client_id',
    client_secret: 'your client_secret',
    player_id: 'johny', # The player id associated with your user
    scopes: ['player.runtime.read', 'player.runtime.write'], # The scopes the player has access to
    expires: 3600 # 1 hour
)
```
This is used to create jwt token which can be created when your user is authenticated. This token can then be sent to the frontend and or stored in your session. With this token the user can directly send requests to the Playlyfe API as the player.


# Documentation
You can initiate a client by giving the client_id and client_secret params
```ruby
Playlyfe.new(
    version: 'v2' # by default it uses Playlyfe v2 api
    client_id: 'Your client id'
    client_secret: 'Your client secret'
    type: 'client' or 'code'
    redirect_uri: 'The url to redirect to' #only for auth code flow
    store: lambda { |token| } # The lambda which will persist the access token to a database. You have to persist the token to a database if you want the access token to remain the same in every request
    load: lambda { return token } # The lambda which will load the access token. This is called internally by the sdk on every request so the
    #the access token can be persisted between requests
)
```
In development the sdk caches the access token in memory so you don't need to provide the store and load lambdas. But in production it is highly recommended to persist the token to a database. It is very simple and easy to do it with redis. You can see the test cases for more examples.
```ruby
    require 'redis'
    require 'playlyfe'
    require 'json'

    redis = Redis.new
    Playlyfe.new(
      client_id: "Your client id",
      client_secret: "Your client secret",
      type: 'client',
      store: lambda { |token| redis.set('token', JSON.generate(token)) },
      load: lambda { return JSON.parse(redis.get('token')) }
    )
```
**API**
```ruby
api(
    'GET' # The request method can be GET/POST/PUT/PATCH/DELETE
    '' # The api route to get data from
    {} # The query params that you want to send to the route
    {} # The data you want to post to the api this will be automagically converted to json
    false # Whether you want the response to be in raw string form or json
)
```
**Get**
```ruby
get(
    '' # The api route to get data from
    {} # The query params that you want to send to the route
)
```
**Get Raw**
This returns the raw response from the request. Useful for images.
```ruby
get_raw(
    '' # The api route to get data from
    {} # The query params that you want to send to the route
)
```
**Post**
```ruby
post(
    '' # The api route to post data to
    {} # The query params that you want to send to the route
    {} # The data you want to post to the api this will be automagically converted to json
)
```
**Patch**
```ruby
patch(
    '' # The api route to patch data
    {} # The query params that you want to send to the route
    {} # The data you want to update in the api this will be automagically converted to json
)
```
**Put**
```ruby
put(
    '' # The api route to put data
    {} # The query params that you want to send to the route
    {} # The data you want to update in the api this will be automagically converted to json
)
```
**Delete**
```ruby
delete(
    '' # The api route to delete the component
    {} # The query params that you want to send to the route
)
```
**Get Login Url**
```ruby
get_login_url()
#This will return the url to which the user needs to be redirected for the user to login. You can use this directly in your views.
```

**Exchange Code**
```ruby
exchange_code(code)
#This is used in the auth code flow so that the sdk can get the access token.
#Before any request to the playlyfe api is made this has to be called atleast once.
#This should be called in the the route/controller which you specified in your redirect_uri
```
**Errors**

A ```PlaylyfeError``` is thrown whenever an error occurs in each call.The Error contains a name and message field which can be used to determine the type of error that occurred.

License
=======
Playlyfe Ruby SDK

http://dev.playlyfe.com/

Copyright(c) 2013-2014, Playlyfe IT Solutions Pvt. Ltd, support@playlyfe.com

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

Contributing
============
```ruby
gem build playlyfe.gemspec
gem push playlyfe.gem
```