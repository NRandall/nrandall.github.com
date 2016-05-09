---
layout: post
title: "Key Takeaways for a Basic Socket.io App"
date: 2016-05-09  6:51:39
---

# Socket.io Basics

Start a new project with `npm init`.

Use `npm install --save jquery socket.io` to get your node modules.

Create a `server.js` file to run your node server.
{% highlight javascript %}
var http = require('http');
var url = require('url');
var fs = require('fs');
var io = require('socket.io');


var server = http.createServer(function(request, response){
  var path = url.parse(request.url).pathname;

  switch(path) {
    case '/':
      response.writeHead(200, {'Content-Type': 'text/html'});
      response.write('hello you');
      response.end();
      break;
    case '/test-socket.html':
      fs.readFile(__dirname + path, function(error, data){
        if(error) {
           response.writeHead(404);
           response.write('Oops, this doesn\'t exist');
           response.end();
        } else {
          response.writeHead(200, {'Content-Type': 'text/html'});
          response.write(data,'utf8');
          response.end();
        }
      });
      break;
    default:
      response.writeHead(404);
      response.write('Oops, this doesn\'t exist');
      response.end();
      break;
    }
});

server.listen(8001);

var listener = io.listen(server);
listener.sockets.on('connection', function(socket){
    //send data to client
    setInterval(function(){
        socket.emit('date', {'date': new Date()});
    }, 1000);
    //receive client data
    socket.on('client_data', function(data){
    process.stdout.write(data.letter);
  });
});
{% endhighlight %}

Create an `test-socket.html` to use for your client.
{% highlight html %}
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Socket Test</title>
  <script src="https://code.jquery.com/jquery-2.2.3.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/1.4.6/socket.io.min.js"></script>
</head>
<body>
  <h1>This is socket yo</h1>
  <textarea id="text"></textarea>
  <div id="date"></div>
  <script>
    var socket= io.connect();

    socket.on('date', function(data){
      $('#date').html('<h2>'+data.date+'</h2>');
    });

    $('#text').keypress(function(e){
      socket.emit('client_data', {'letter': String.fromCharCode(e.charCode)});
    });
  </script>
</body>
</html>
{% endhighlight %}
Key takeaways from this study session are that you are creating an open channel with an emitter and listener at each point. On the server it looks like this:
{% highlight javascript %}
var listener = io.listen(server);

listener.sockets.on('connection', function(socket){
    
    //send data to client
    setInterval(function(){
        socket.emit('date', {'date': new Date()});
    }, 1000);

    //receive client data
    socket.on('client_data', function(data){
    process.stdout.write(data.letter);

  });
});
{% endhighlight %}

On the client the corresponding handlers look like this:
{% highlight javascript %}
socket.on('date', function(data){
  // Access to data.date info being sent from the server every second here.
});

// On keypress `emit` the charCode back to the server.
$('#text').keypress(function(e){
  socket.emit('client_data', {'letter': String.fromCharCode(e.charCode)});
});
{% endhighlight %}

