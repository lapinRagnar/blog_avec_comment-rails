rails generate scaffold post title content:text

rails g model Comment username:string body:text post:references

rails db:migrate


rails g controller comments


dans post.rb
-------------->
class Post < ApplicationRecord
  has_many :comments
end

dans comment.rb
------------------->
class Comment < ApplicationRecord
  belongs_to :post
end

dans routes.rb
---------------->
root "posts#index"
resources :posts do
	resources :comments
end

en dessour post/show.html.erb ajouter:
----------------------------------------->
<h3>Ajouter des commentaires</h3>

<%= form_for([@post, @post.comments.build]) do |f| %>
  <p>
    <%= f.label :username %> <br>
    <%= f.text_field(:username, {:class => 'form-control'} ) %>
  </p>

  <p>
    <%= f.label :body %> <br>
    <%= f.text_area(:body, {:class => 'form-control'}) %>
  </p>

  <%= f.submit({:class => 'btn btn-default'}) %>

<% end %>

dans comment controller
----------------------------->

class CommentsController < ApplicationController

  def create
    @post = Post.find(params[:post_id])
    @comment = @post.comments.create(comment_params)
    redirect_to post_path(@post)
  end


  private

  def comment_params
    params.require(:comment).permit(:username, :body)

  end

end


on affiche les commentaires dans posts/show.html.erb
----------------------------------------------------------->

<h3>Afficher les commentaires</h3>

<% @post.comments.each do |comment| %>
  <div class="well" >
    <p>
      <strong> <%= comment.username %> </strong>
      <%= comment.body %>  
    </p>
  </div>

<% end %>


refacorisation dans des partial
-------------------------------->

creer comments/_form.html.erb:
.................................

couper dedans (le formulaire de commentaire de posts/show)


<h3>Ajouter des commentaires</h3>

<%= form_for([@post, @post.comments.build]) do |f| %>
  <p>
    <%= f.label :username %> <br>
    <%= f.text_field(:username, {:class => 'form-control'} ) %>
  </p>

  <p>
    <%= f.label :body %> <br>
    <%= f.text_area(:body, {:class => 'form-control'}) %>
  </p>

  <%= f.submit({:class => 'btn btn-default'}) %>

<% end %>

et à la place on met:
.....................

<%= render 'comments/form' %>


ensuite creer comments/_comments.html.erb:
.........................................

couper dedans (l'affichage de commentaire de posts/show)


<h3>Afficher les commentaires</h3>

<% @post.comments.each do |comment| %>
  <div class="well" >
    <p>
      <strong> <%= comment.username %> </strong>
      <%= comment.body %>  
    </p>
  </div>

<% end %>

et le remplacer par:
....................

<%= render 'comments/comments' %>


---------------------------
supprimer un commentaire
--------------------------


on modifie _comments.html.erb comme:
------------------------------------->

<h3>Afficher les commentaires</h3>

<% @post.comments.each do |comment| %>
  <div class="well" >
    <p>
      <strong> <%= comment.username %> </strong>
      <%= comment.body %> 
      <%= link_to 'Supprimer commentaire',
         [comment.post, comment],
          method: :delete,
          data: { confirm: "t'es sure? " }
      %> 
    </p>
  </div>

<% end %>

et dans comments controller, on ajoute
----------------------------------------->

  def destroy
    @post = Post.find(params[:post_id])
    @comment = @post.comments.find(params[:id])
    @comment.destroy
    redirect_to post_path(@post)
  end

