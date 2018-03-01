# Chat
## HOW this app was created

`mix phx.new chat --database mysql`

`mix phx.gen.channel chat_room`

Add the channel to your `lib/chat_web/channels/user_socket.ex` handler, for example:


    `channel "chat_room:lobby", ChatWeb.ChatRoomChannel`


lib/chat_web/templates/layout/app.html.eex

``` 
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/2.2.4/jquery.min.js"></script>

<script src="<%= static_path(@conn, "/js/app.js") %>"></script> 
```
	  

update assets/js/app.js
uncomment inclusion of socket.js

update assets/js/socket.js
verify call to `chat_room:lobby`

update assets/js/app.js

```
let channel = socket.channel('chat_room:lobby', {});
	let list = $('#message-list');
	let message = $('#msg');
	let name = $('#name');
message.on('keypress', event => {
	    if (event.keyCode == 13) {
	        channel.push('shout', {
	            name: name.val(),
	            message: message.val()
	        });
	        message.val('');
	    }
	});
	

	channel.on('shout', payload => {
	    list.append(`<b>${payload.name || 'new_user'}:</b> ${payload.message}<br>`);
	    list.prop({
	        scrollTop: list.prop('scrollHeight')
	    });
	});
	

	channel
	    .join()
	    .receive('ok', resp => {
	        console.log('Joined successfully', resp);
	    })
	    .receive('error', resp => {
	        console.log('Unable to join', resp);
	    });
```


update lib/chat_web/templates/page/index.html.eex

```
<div id='message-list' class='row'>
	</div>
	<div class='row form-group'>
	  <div class='col-md-3'>
	    <input type='text' id='name' class='form-control' placeholder='Name' />
	  </div>
	  <div class='col-md-9'>
	    <input type='text' id='msg' class='form-control' placeholder='Message' />
	  </div>
	</div>
```

update assets/css/app.css

```
#message-list {
	    border: 1px solid #000;
	    height: 600px;
	    padding: 5px;
	    overflow: scroll;
	    margin-bottom: 30px;
	}
```

### run server

` mix phx.server `

### add persistance 

` mix phx.gen.schema Message messages name:string message:string ` 
` mix ecto.migrate `

update lib/chat_web/channels/chat_room_channel.ex

 ```
 def handle_in("shout", payload, socket) do
	    spawn(fn -> save_msg(payload) end)
	    broadcast socket, "shout", payload
	    {:noreply, socket}
	  end

  defp save_msg(msg) do
	    Chat.Message.changeset(%Chat.Message{}, msg) |> Chat.Repo.insert  
	  end
```
   
under join in authorised add

``` 
send(self(), :after_join) 
```

update lib/chat_web/channels/chat_room_channel.ex

```  
def handle_info(:after_join, socket) do
	    Chat.Message.get_msgs() 
	    |> Enum.each(fn msg -> push(socket, "shout",
	      %{
	        name: msg.name,
	        message: msg.message,
	      }) end)
	    {:noreply, socket}
	  end
```

update lib/chat/message.ex

```
  def get_msgs(limit \\ 20) do
	    Chat.Repo.all(Message, limit: limit)
	  end
```


### run server
` mix phx.server `





## To start your Phoenix server:

  * Install dependencies with `mix deps.get`
  * Create and migrate your database with `mix ecto.create && mix ecto.migrate`
  * Install Node.js dependencies with `cd assets && npm install`
  * Start Phoenix endpoint with `mix phx.server`

Now you can visit [`localhost:4000`](http://localhost:4000) from your browser.

Ready to run in production? Please [check our deployment guides](http://www.phoenixframework.org/docs/deployment).

## Learn more

  * Official website: http://www.phoenixframework.org/
  * Guides: http://phoenixframework.org/docs/overview
  * Docs: https://hexdocs.pm/phoenix
  * Mailing list: http://groups.google.com/group/phoenix-talk
  * Source: https://github.com/phoenixframework/phoenix
