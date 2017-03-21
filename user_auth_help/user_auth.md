***USER AUTH CONTROLLER***
get "/users" do
  erb :'/users/index'
end

get "/users/new" do

  erb :"/users/new"
end

post '/users' do
  @user = User.new(params[:user])
  if @user.save
    login(@user)
    redirect "/users/#{@user.id}"
  else
    erb :"/users/new"
  end
end

get "/users/login"do
  erb :"users/login"
end

post "/users/login" do
  @user = User.find_by(username: params[:user][:username])
  @password_attempt = params[:user][:password]
  if @user.authenticate(@password_attempt)
    login(@user)
    redirect "/users/#{@user.id}"
  else
    erb :"users/login"
  end
end

get "/users/logout" do
  logout
  redirect "/decks"
end

get "/users/:id" do
  if current_user
    @decks = Deck.all
    erb :"users/show"
  else
    redirect "/decks"
  end
end

^^^HELPER METHODS^^^


 def login(user)
    session[:user_id] = user.id
  end

  def logout
    session.destroy
  end

  def current_user
    @user = User.find_by(id: session[:user_id])
  end

!!!MODEL!!!
class User < ActiveRecord::Base
  ***Associations***

include BCrypt

  def password
    @password ||= Password.new(password_hash)
  end

  def password=(new_password)
    @password = Password.create(new_password)
    self.password_hash = @password
  end

  def authenticate(pw_input)
    self.password == pw_input
  end

end

###LOGIN PAGE###
<center>
<h1>log in!</h1>
<form action="/users/login" method="POST">
<input type="text" name="user[username]" placeholder="username">
<input type="password" name="user[password]" placeholder="Password">
<br><br>
<input type="submit" value="log in!">
</form>
</center>


###REGISTRATION PAGE###
<center>
<h1>register account</h1>
<form action="/users" method="POST">
<input type="text" name="user[username]" placeholder="username">
<input type="password" name="user[password]" placeholder="Password">
<br><br>
<input type="submit" value="register an account">
</form>
</center>


###SHOW PAGE###

<h1> hello <%=@user.username%></h1>

<h2>your scores</h2>
<p>
<%@user.games.each do |game|%>
  <%=game.deck.name%>
  <br>
  points: <%=game.points%>
  guesses: <%=game.guesses%>
  <br>
  <br>
<%end%>
</p>
<h3>Flashcard Decks</h3>
  <%@decks.each do |deck|%>
  <ul>
    <li>
      <a href="/decks/<%=deck.id%>/cards"> <%=deck.name%> </a>
    </li>
  </ul>
  <%end%>
