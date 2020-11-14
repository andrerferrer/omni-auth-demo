## Goal
This is an app to teach how to implement basic Authentication in Rails using the [devise gem](https://github.com/heartcombo/devise).

## Setup

### Add devise gem
```ruby
# Gemfile
gem 'devise'
```

You need to `bundle install` after that.

### rails install devise
```ruby
rails generate devise:install
```

It will show you some manual configs that we must do.
```
Some setup you must do manually if you haven't yet:

  1. Ensure you have defined default url options in your environments files. Here
     is an example of default_url_options appropriate for a development environment
     in config/environments/development.rb:

       config.action_mailer.default_url_options = { host: 'localhost', port: 3000 }

     In production, :host should be set to the actual host of your application.

  2. Ensure you have defined root_url to *something* in your config/routes.rb.
     For example:

       root to: "home#index"

  3. Ensure you have flash messages in app/views/layouts/application.html.erb.
     For example:

       <p class="notice"><%= notice %></p>
       <p class="alert"><%= alert %></p>

  4. You can copy Devise views (for customization) to your app by running:

       rails g devise:views

```

### Generate the user (or whatever) model that will use Devise
```ruby
rails generate devise User
```

### Add some navigation for the user
```
<% if user_signed_in? %>
  <%= link_to "Edit my account", edit_user_registration_path  %>
  <%= link_to "Sign out", destroy_user_session_path, method: :delete %>
<% else %>
  <%= link_to "Log In", new_user_session_path %>
  <%= link_to "Sign up", new_user_registration_path %>
<% end %>
```

### Implement the White-list approach

The white-list approach states that we'll block everything and ONLY allow what we specifically state as allowed.

To do so, all we have to do is to add the `skip` on the right controller.

e.g.
```ruby
class PagesController < ApplicationController
  skip_before_action :authenticate_user!, only: [:home]

  def home
  end
end
```

## Adding attributes to the user

If we want to add a nickname to the user, what should we do?

1. Generate and run the migration
```
rails g migration AddNicknameToUsers nickname:string
rails db:migrate
```

2. Make devise handle it in the ApplicationController
Add this to you `app/controllers/application_controller.rb`.

```ruby
# Config devise params if we have a devise controller running
before_action :config_devise_params, if: :devise_controller?

protected

def config_devise_params
  # This allows nickname to be added to the sign up
  devise_parameter_sanitizer.permit(:sign_up, keys: [:nickname])
  # This allows nickname to be added to the editting of the account
  devise_parameter_sanitizer.permit(:account_update, keys: [:nickname])
end
```

## Tips and Tricks

* You can check if a user is logged in with `user_signed_in?`
* You can retrieve the user that is logged in with `current_user`
