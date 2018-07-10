### Chat App

```bash
$ rails _5.0.7_ new chat_app
```



##### rails pusher

*gemfile*

```ruby
# pusher
gem 'pusher'
# authenticate
gem 'devise'
# key encrypt
gem 'figaro'

# gem 'turbolinks' // 터보링크 주석처리
```

```bash
$ bundle install
```

*app\assets\javascripts\application.js*

``` javascript
//= require turbolinks 삭제
```

*application.html.erb*

data turbolink 삭제



##### rails devise

```bash
$ rails g devise:install
$ rails g devise users
```

##### scaffold chat_room

```bash
$ rails g scaffold chat_rooms
```

##### models

```bash
$ rails g model chat
$ rails g model admission
```



##### models' columns 설정

*chat_rooms_migrate*

```ruby
t.string			:title
t.string			:master_id

t.integer			:max_count
t.integer			:admissions_count, default: 0
```

*chats_migrate*

```ruby
t.references		:user
t.references		:chat_room

t.text			    :message
```

*admissions_migrate*

```ruby
t.references		:chat_room
t.references		:user
```



```bash
$ rake db:migrate
```



##### models relations configuration

*admission.rb*

```ruby
belongs_to :user
belongs_to :chat_room, counter_cache: true
```

*chat.rb*

```ruby
belongs_to :user
belongs_to :chat_room
```

*chat_room.rb*

```ruby
has_many :admissions
has_many :users, through: :admissions

has_many :chats
```

*user.rb*

```ruby
has_many :admissions
has_many :chat_rooms, through: :admissions
has_many :chats
```

##### rails c

```console
> User.create(email: "aa@a.a", password: "123456", password_confirmation: "123456")
> ChatRoom.create(title: "chatroom", master_id: User.first.email, max_count: 10)
> Admission.create(user_id: User.first.id, chat_room_id: ChatRoom.first.id)
> ChatRoom.first.admissions.size
```

##### model coding

*ChatRoomController.rb*

```ruby
before_action :authenticate_user!, except: [:index]


...


def create
    @chat_room = ChatRoom.new(chat_room_params)
    @chat_room.master_id = current_user.email        # << 개설자 마스터 설정
    respond_to do |format|
        if @chat_room.save
            @chat_room.master_admit_room(current_user)    #  << 개설자 챗룸 참여
...

#private 위에 작성
def user_admit_room
    # 현재 유저가 있는 방에서 join 버튼을 눌렀을 때 동작하는 액션
    @chat_room.user_admit_room(current_user)
end
            
private
    # Use callbacks to share common setup or constraints between actions.
    def set_chat_room
      @chat_room = ChatRoom.find(params[:id])
    end

    # Never trust parameters from the scary internet, only allow the white list through.
    def chat_room_params
      params.fetch(:chat_room, {}).permit(:title, :max_count)          # <<
    end
end
```

*chat_room.rb*

```ruby
# 채팅방이 만들어 질 때, 현재 이 채팅방을 만든 유저가 이 채팅방의 마스터가 되고, 현재 방에 참가한 것으로 된다.

def master_admit_room(user)
	Admission.create(user_id: user.id, chat_room_id: self.id)
end

def user_admit_room(user)
    # 현재 유저가 있는 방에서 join 버튼을 눌렀을 때 동작하는 액션
    Admission.create(user_id: user.id, chat_room_id: self.id)
end
```

*views\chat_rooms\\_form.html.erb

```erb
 ...


  <div class="form-group">
    <%= f.label :title %>
    <%= f.text_field :title %>
  </div>
  
  <div class="form-group">
    <%= f.label :max_count %>
    <%= f.number_field :max_count %>
  </div>


  <div class="actions">
    <%= f.submit %>
  </div>
<% end %>
```

*views\chat_rooms\\index.html.erb*

```erb
<p id="notice"><%= notice %></p>

<h1>Chat Rooms</h1>

<table>
  <thead>
    <tr>
      <th colspan="1">방제</th>              # <<
      <th colspan="1">인원</th>			  # <<
      <th colspan="1">방장</th>              # <<
    </tr>
  </thead>

  <tbody>
    <% @chat_rooms.each do |chat_room| %>
      <tr>
        <td><%= chat_room.title %></td>     # <<
        <td><%= chat_room.admissions.size %> / <%= chat_room.max_count %></td>
        <td><%= chat_room.master_id %></td>  # <<
          
          
...
```



##### pusher.com

프로젝트 생성

​	front-end: jquery

​	back-end: rails

```bash
$ figaro install
```

*config\application.yml*

```yml
development:
    pusher_app_id: ?
    pusher_key: ?
    pusher_secret: ?
    pusher_cluster: ?
```

*config\initializers\pusher.rb* 없으면 생성

```ruby
require 'pusher'

Pusher.app_id = ENV["puser_app_id"]
Pusher.key = ENV["puser_key"]
Pusher.secret = ENV["pusher_secret"]
Pusher.cluster = ENV["pusher_cluster"]
Pusher.logger = Rails.logger
Pusher.encrypted = true
```

*models\chat_room.rb*

```ruby
after_commit :create_chat_room_notification, on: :create

def create_chat_room_notification
   Pusher.trigger('chat_room', 'create', self.as_json)
    # ('channel_name', 'event_name', data)
end
```

*views\layouts\application.html.erb*

```erb
<!DOCTYPE html>
<html>
  <head>
    <title>ChatApp</title>
    <%= csrf_meta_tags %>

    <%= stylesheet_link_tag    'application', media: 'all' %>
    <%= javascript_include_tag 'application' %>
    <script src="https://js.pusher.com/4.1/pusher.min.js"></script>  # <<
      
...
```

*views\chat_rooms\index.html.erb*

```erb
<table>
  <thead>
    <tr>
      <th colspan="1">방제</th>
      <th colspan="1">인원</th>
      <th colspan="1">방장</th>
    </tr>
  </thead>

  <tbody class="chat_room_list">
    <% @chat_rooms.reverse_each do |chat_room| %>
      <tr>
        <td><%= chat_room.title %></td>
        <td><span class="current<%= chat_room.id %>"><%= chat_room.admissions.size %></span> / <%= chat_room.max_count %></td>
        <td><%= chat_room.master_id %></td>
        <td><%= link_to 'Show', chat_room %></td>
      </tr>
    <% end %>
  </tbody>
</table>

...

<script>
$(document).on('ready', function() {
   // 방이 만들어졌을 때, 방에 대한 데이터를 받아서
   // 방 목록에 추가해주는 js function
    function room_created(data) {
      $('.chat_room_list').prepend(`
      <tr>
        <td>${data.title}</td>
        <td><span class="current${data.id}"></span> / ${data.max_count}</td>
        <td>${data.master_id}</td>
        <td><a href="/chat_rooms/${data.id}">Show</a></td>
      </tr>`);
      alert("방이 추가됨~");
    }
    var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
                            cluster: "<%= ENV["pusher_cluster"] %>",
                            encrypted: true});

    var channel = pusher.subscribe('chat_room');
    channel.bind('create', function(data) {
      console.log(data);
      room_created(data);  
    });  

});
</script>
```



*models\admissions.rb*

```ruby
after_commit :user_join_chat_room_notification, on: :create

def user_join_chat_room_notification
    Pusher.trigger('chat_room', 'join', {chat_room_id: self.chat_room_id}.as_json) 
end
```

*views\chat_rooms\index.html.erb*

```erb
<script>
$(document).on('ready', function() {
   // 방이 만들어졌을 때, 방에 대한 데이터를 받아서
   // 방 목록에 추가해주는 js function
    function room_created(data) {
      $('.chat_room_list').prepend(`
      <tr>
        <td>${data.title}</td>
        <td><span class="current${data.id}">0</span> / ${data.max_count}</td>
        <td>${data.master_id}</td>
        <td><a href="/chat_rooms/${data.id}">Show</a></td>
      </tr>`);
    }
    
    function user_joined(data) {
      var current = $(`.current${data.chat_room_id}`);
      current.text(parseInt(current.text()) + 1); 
    }
    var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
                            cluster: "<%= ENV["pusher_cluster"] %>",
                            encrypted: true});

    var channel = pusher.subscribe('chat_room');
    channel.bind('create', function(data) {
      console.log(data);
      room_created(data);  
    });  
    channel.bind('join', function(data) {
      console.log(data);
      user_joined(data);
    });

});
</script>
```



##### log in log out

*views\chat_rooms\index.html.erb*

```erb
// 상단에 노티스 지우고
<% if user_signed_in? %>
<%= current_user.email %> / <%= link_to 'log out', destroy_user_session_path, method: :delete %>
<% else %>
<%= link_to 'log in', new_user_session_path %>
<% end %>

<hr>

...
```

##### join

*views\chat_rooms\show.html.erb*

```erb
// 상단에 노티스 지우고
<%= current_user.email %>
<h3>현재 로그인한 사람</h3>
<% @chat_room.users.each do |user| %>
    <p><%= user.email %></p>
<% end %>
<hr>
<%= link_to 'Join', '', %>
<%= link_to 'Edit', edit_chat_room_path(@chat_room) %> |
<%= link_to 'Back', chat_rooms_path %>
```



*routes.rb*

```ruby
resources :chat_rooms do
    member do
        post '/join', to: 'chat_rooms#user_admit_room', as: 'join'
    end
end
```



*views\chat_rooms\show.html.erb*

```erb
<%= current_user.email %>
<h3>채팅참여자</h3>
<% @chat_room.users.each do |user| %>
    <p><%= user.email %></p>
<% end %>
<hr>
<%= link_to 'Join', join_chat_room_path(@chat_room), method: 'POST', remote: true, class: 'join_room' %>
<%= link_to 'Edit', edit_chat_room_path(@chat_room) %> |
<%= link_to 'Back', chat_rooms_path %>
```

> remote: true
>
> ajax로 동작 시킴

*ChatRoomsController.rb*

```ruby
before_action :set_chat_room, only: [:show, :edit, :update, :destroy, :user_admit_room]
```

*views\chat_rooms\show.html.erb*

```erb
<script>
function user_joined(data) {
    $('.joined_user_list').append(`<p>${data.email}</p>`);
};
var pusher = new Pusher('<%= ENV["pusher_key"]%>', {
                        cluster: "<%= ENV["pusher_cluster"] %>",
                        encrypted: true});

var channel = pusher.subscribe('chat_room');
channel.bind('join', function(data) {
  console.log(data);
  user_joined(data);
});
</script>
```

*models\admission.rb*

```ruby
def user_join_chat_room_notification
    Pusher.trigger('chat_room', 'join', {chat_room_id: self.chat_room_id, email: self.user.email}.as_json) 
end
```



- 과제
  - 현재 이 방에 들어와있는 사람은 join 버튼이 안 보임
  - 한 유저는 방 하나에 한 번만 들어갈 수 있음.