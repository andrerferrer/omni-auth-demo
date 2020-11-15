## Goal
This is an app to teach how to implement Omni Authentication login (with Facebook, Spotify etc) in Rails using the [devise gem](https://github.com/heartcombo/devise).

This demo assumes that you have the `Devise` gem set up. If you don't, [check this out](https://github.com/andrerferrer/devise-demo#goal).

[Check my other demos here.](https://github.com/andrerferrer/dedemos#devise)

You can check the Devise Omniauth overview [here](https://github.com/heartcombo/devise/wiki/OmniAuth:-Overview).

## Setup

### Add the attributes (we will use later) to the User (if we don't have them already)
```
rails g migration AddOmniauthToUsers \
    provider uid picture_url first_name last_name token token_expiry:datetime
rails db:migrate
```

### Ensure that the `User` has omniauth
```ruby
# app/models/user.rb
class User < ApplicationRecord
  devise :omniauthable
end
```

### Prepare the app to receive the callback

#### Prepare the routes
```ruby
# config/routes.rb
Rails.application.routes.draw do
  devise_for :users,
    controllers: { omniauth_callbacks: 'users/omniauth_callbacks' }
end
```

#### Create the controller
```ruby
# app/controllers/users/omniauth_callbacks_controller.rb
# Create this controller

class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
end
```

#### Add the callback method to the User
```ruby
# app/models/user.rb
class User < ApplicationRecord
  def self.find_for_oauth(auth)
    # Create the user params
    user_params = auth.slice("provider", "uid")
    user_params.merge! auth.info.slice("email", "first_name", "last_name")
    user_params[:picture_url] = auth.info.image
    user_params[:token] = auth.credentials.token
    user_params[:token_expiry] = Time.at(auth.credentials.expires_at)
    user_params = user_params.to_h
    # Finish creating the user params

    # Find the user if there was a log in
    user = User.find_by(provider: auth.provider, uid: auth.uid)

    # If the User did a regular sign up in the past, find it
    user ||= User.find_by(email: auth.info.email)

    # If we had a user, update it
    if user
      user.update(user_params)
    # Else, create a new user with the params that come from the app callback
    else
      user = User.new(user_params)
      # create a fake password for validation
      user.password = Devise.friendly_token[0,20]
      user.save
    end

    return user
  end
end
```

---
### Set everything up for Facebook
#### Add the [gem](https://github.com/simi/omniauth-facebook#omniauth-facebook--) to the Gemfile.
```ruby
# Gemfile
gem 'omniauth-facebook'
```

Remember to `bundle install`.

#### Set up the API key

We are assuming that the `dotenv` gem is set up.

[Check how to here (PT-BR)](apps_apis/facebook/facebook.md).

#### Ensure that the `User` has the provider
```ruby
# app/models/user.rb
class User < ApplicationRecord
  devise :omniauthable, omniauth_providers: [:facebook] # add this line
end
```

#### Config Devise
```ruby
# config/initializers/devise.rb
Devise.setup do |config|
  config.omniauth :facebook, ENV["FACEBOOK_ID"], ENV["FACEBOOK_PWD"],
    scope: 'email',
    info_fields: 'email, first_name, last_name',
    image_size: 'square',  # 50x50, guaranteed ratio
    secure_image_url: true
end

```

[Read the omniauth config if you want to dive deeper.](https://github.com/simi/omniauth-facebook#configuring)

#### Create the method in the Users::OmniauthCallbacksController

```ruby
# app/controllers/users/omniauth_callbacks_controller.rb
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def facebook
    user = User.find_for_oauth(request.env['omniauth.auth'])

    if user.persisted?
      sign_in_and_redirect user, event: :authentication
      set_flash_message(:notice, :success, kind: 'Facebook') if is_navigational_format?
    else
      session['devise.facebook_data'] = request.env['omniauth.auth']
      redirect_to new_user_registration_url
    end
  end
end
```

---
### Set everything up for Spotify
#### Add the [gem](https://github.com/icoretech/omniauth-spotify#spotify-omniauth-strategy) to the Gemfile
```ruby
# Gemfile
gem 'omniauth-spotify'
```

Remember to `bundle install`.

#### Set up the API key

We are assuming that the `dotenv` gem is set up.

[Check how to here](apps_apis/spotify/spotify.md).

#### Ensure that the `User` has the provider
```ruby
# app/models/user.rb
class User < ApplicationRecord
  devise :omniauthable, omniauth_providers: [:spotify] # add this line
end
```

#### Config Devise
```ruby
# config/initializers/devise.rb
Devise.setup do |config|
  config.omniauth :spotify, ENV["SPOTIFY_ID"], ENV["SPOTIFY_PWD"], scope: %w(
    playlist-read-private
    user-read-private
    user-read-email
  ).join(' ')
end
```
#### Create the method in the Users::OmniauthCallbacksController

```ruby
# app/controllers/users/omniauth_callbacks_controller.rb
class Users::OmniauthCallbacksController < Devise::OmniauthCallbacksController
  def spotify
    user = User.find_for_oauth(request.env['omniauth.auth'])

    if user.persisted?
      sign_in_and_redirect user, event: :authentication
      set_flash_message(:notice, :success, kind: 'Spotify') if is_navigational_format?
    else
      session['devise.spotify_data'] = request.env['omniauth.auth']
      redirect_to new_user_registration_url
    end
  end
end
```

[Read the omniauth config if you want to dive deeper.](https://github.com/simi/omniauth-facebook#configuring)

---
### TO BE DONE

#### Show the picture from the app with a helper
```erb
<% avatar_url = current_user.picture_url || "http://placehold.it/30x30" %>
<%= image_tag avatar_url %>

```

#### Improve the user callback into a service
