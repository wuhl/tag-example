# README

Based on https://www.youtube.com/watch?v=03enr4NNgLI

Ruby version
```
$ ruby -v
ruby 3.3.0 (2023-12-25 revision 5124f9ac75)[x86_64-linux]
```
Rails version
```
$ rails -v
Rails 7.1.3.2
```
Create application with bootstrap
```
$ rails new -j esbuild -c bootstrap tag_example
```
Switch into application
```
$ cd tag_example
```
Scaffold 2 tables + 1 tag-table
```
$ rails g scaffold post title body:text
$ rails g scaffold tag name
$ rails g model taggable post:belongs_to tag:belongs_to
```
Create database
```
$ rails db:migrate
```
Edit config/routes.rb
```
Rails.application.routes.draw do
  root 'posts#index'
  resources :tags
  resources :posts
  # Define your application routes per the DSL in https://guides.rubyonrails.org/routing.html

  # Reveal health status on /up that returns 200 if the app boots with no exceptions, otherwise 500.
  # Can be used by load balancers and uptime monitors to verify that the app is live.
  get "up" => "rails/health#show", as: :rails_health_check

  # Defines the root path route ("/")
  # root "posts#index"
end
```
Edit app/models/post.rb
```
class Post < ApplicationRecord
    has_many :taggables, dependent: :destroy
    has_many :tags, through: :taggables
end
```
Edit app/models/tag.rb
```
class Tag < ApplicationRecord
    has_many :taggables, dependent: :destroy
    has_many :posts, through: :taggables
end
```
Edit app/views/posts/form.html.erb
```
<%= form_with(model: post) do |form| %>
  <% if post.errors.any? %>
    <div style="color: red">
      <h2><%= pluralize(post.errors.count, "error") %> prohibited this post from being saved:</h2>
      <ul>
        <% post.errors.each do |error| %>
          <li><%= error.full_message %></li>
        <% end %>
      </ul>
    </div>
  <% end %>
  <div>
    <%= form.label :title, style: "display: block" %>
    <%= form.text_field :title %>
  </div>
  <div>
    <%= form.label :body, style: "display: block" %>
    <%= form.text_area :body %>
  </div>
  <div>
    <%= form.label :tag, style: "display: block" %>
    <%= form.text_field :tags, value: post.tags.map(&:name).join(",") %>
  </div>
  <div>
    <%= form.submit %>
  </div>
<% end %>
```
Edit app/controllers/posts_controller.rb
```
class PostsController < ApplicationController
  before_action :set_post, only: %i[ show edit update destroy ]

  # GET /posts or /posts.json
  def index
    @posts = Post.all
  end

  # GET /posts/1 or /posts/1.json
  def show
  end

  # GET /posts/new
  def new
    @post = Post.new
  end

  # GET /posts/1/edit
  def edit
  end

  # POST /posts or /posts.json
  def create
    @post = Post.new(post_params.except(:tags))
    create_or_delete_post_tags(@post, params[:post][:tags])

    respond_to do |format|
      if @post.save
        format.html { redirect_to post_url(@post), notice: "Post was successfully created." }
        format.json { render :show, status: :created, location: @post }
      else
        format.html { render :new, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # PATCH/PUT /posts/1 or /posts/1.json
  def update
    create_or_delete_post_tags(@post, params[:post][:tags])

    respond_to do |format|
      if @post.update(post_params.except(:tags))
        format.html { redirect_to post_url(@post), notice: "Post was successfully updated." }
        format.json { render :show, status: :ok, location: @post }
      else
        format.html { render :edit, status: :unprocessable_entity }
        format.json { render json: @post.errors, status: :unprocessable_entity }
      end
    end
  end

  # DELETE /posts/1 or /posts/1.json
  def destroy
    @post.destroy!

    respond_to do |format|
      format.html { redirect_to posts_url, notice: "Post was successfully destroyed." }
      format.json { head :no_content }
    end
  end

  private

    def create_or_delete_post_tags(post, tags)
      post.taggables.destroy_all
      tags = tags.strip.split(",")
      tags.each do |tag|
        post.tags << Tag.find_or_create_by(name: tag)
      end
    end

    # Use callbacks to share common setup or constraints between actions.
    def set_post
      @post = Post.find(params[:id])
    end

    # Only allow a list of trusted parameters through.
    def post_params
      params.require(:post).permit(:title, :body, :tags)
    end
end
```
Edit app/views/posts/_posts.rb
```
<div id="<%= dom_id post %>">
  <p>
    <strong>Title:</strong>
    <%= post.title %>
  </p>
  <p>
    <strong>Body:</strong>
    <%= post.body %>
  </p>
  <strong>Tags:</strong>
  <% post.tags.each do |tag| %>
    <%= link_to tag.name.capitalize, tag, class: "btn btn-primary" %>
  <% end %>
</div>
```
Edit app/views/tagsgit statu/_tag.rb 
```
<div id="<%= dom_id tag %>">
  <p>
    <strong>Name:</strong>
    <%= tag.name %>
  </p>
  <% tag.taggables.each do |taggable| %>
    <h2>Post:</h2>
    <%= render 'posts/post', post: taggable.post %>
    <%= link_to "Read more", taggable.post %>
    <hr />
  <% end %>
</div>
```
Start application on http://localhost:3000
```
$ bin/dev
```
