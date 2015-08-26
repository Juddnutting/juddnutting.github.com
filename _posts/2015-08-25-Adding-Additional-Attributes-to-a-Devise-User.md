---
layout: post
title: Adding Additional Attributes to a Devise User
categories: []
tags: [Rails, Devise, Paperclip]
published: True

---

### Using Devise

Devise is an ruby Gem that allows for a plug and play user authentication system. I wanted to add some functionality to my implementation that would create a 'profile' as soon as the user signed up. This profile would have a generic name and an image field using paperclip to upload an image. In the `User` model I made a `has_one` association with `Profile` and used `accepts_nested_attributes_for` so that I could add the two fields in the sign up view. Of course a Profile `belongs_to` a User.

```
class User < ActiveRecord::Base
	devise :database_authenticatable, :registerable, :recoverable, :rememberable, :trackable, :validatable
	has_one :profile, dependent: :destroy, autosave: true
  	accepts_nested_attributes_for :profile
```

### Modifying Devise

To get access to the devise controller
`rails generate devise:controllers [scope]`
using `users` as the scope

This will generate all the controllers for you in the `app/controllers/users/` folder. Now you can modify the actions of the controller as you would like. I wanted to build a profile during the creation of a user so I modified the registration controller like so

```
class Users::RegistrationsController < Devise::RegistrationsController
  before_filter :configure_sign_up_params, only: [:create]
  before_filter :configure_account_update_params, only: [:update]

  def new
    build_resource({})
    resource.build_profile
    respond_with self.resource
  end

  ...

  def configure_sign_up_params
    devise_parameter_sanitizer.for(:sign_up) do |u|
      u.permit(:email, :password, :password_confirmation, :profile_attributes => [:name, :image])
    end
  end
```
The new method calls `resource.build_profile` which will create a profile that is associated to the resource (which is Devise's name for the user). Note that the profile attributes of `:name` and `:image` are nested inside the `:profile_attributes` hash. Make user to sanitze these params or your form will not work. 

### Change the Route

Specify the specific route you are changing. In my case it was

```
devise_for :users, :controllers => {:registrations => "users/registrations"}
```

### Change the View

Now change the registrations new.html page to add the nested fields by using `f.fields_for :profile`

```
<%= form_for(resource, as: resource_name, url: registration_path(resource_name)) do |f| %>
  <%= devise_error_messages! %>

  ...

  <%= f.fields_for :profile do |profile_fields| %>
    <div class="form-group">
      <%= profile_fields.label :name %>
      <%= profile_fields.text_field :name, class: "form-control", :autofocus => true %>
    </div>

    <div class="form-group">
      <%= profile_fields.label :image, "Avatar Image" %>
      <%= profile_fields.file_field :image, class: "form-control"%>
    </div>

      <%= profile_fields.label #INSERT ANOTHER NEW ATTRIBUTE IF DESIRED %>
  <% end %>

  <div class="form-group">
    <%= f.submit "Sign up", class: "btn btn-primary btn-lrg" %>
  </div>
<% end %>
```

You may need to move the views into a new folder. The Devise ReadMe indicates "Copy the views from devise/sessions to users/sessions. Since the controller was changed, it won't use the default views located in devise/sessions."

### Finished

You should be finished up. You can now automatically build a profile on user creation and populate it with data. I decided not to use the user's edit form to change this data. Devise requires a password when modifying content on it's edit page and I didn't want the user to need their password to change a photo. So I use the profile controller and edit the attributes through it. 

Full repo is [here](https://github.com/Juddnutting/imatar)





