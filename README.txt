- Compile Game.java.
- Compile and run AppServer.java.
- Compile and run Client.java and follow instructions.
- Run 2 seperate Client programs to play the game.

I wrote and tested this using eclipse, and partially tested in CLI.

User Instructions:
- User input can be "q" (for query) to send GET request, 1-9 to make move(POST(9)), or a String between 2-10 characters long to enter username(POST(username)).


Functionality:

- When server starts, it waits for two players to join (player is considered 'joined' when they send POST request with username i.e. not just from GET)
- Two people can join. When two people are connected any other client request is met with a "Game is full." response body.
- If one person leaves mid-game then the game (not server) automatically restarts and makes room for another client to join
- When game finishes, another game automatically begins.


Overview:

- Client program handles input and has a GET method to be called if input = "q", a POST method if input in 1-9, and another POST method if input is greater than 1 character.
  The first POST method handles attempts to make a move and the second handles attempts to enter a username.

- A HTTPRequest is generated from these and sent to the HTTPServer using HTTPExchange. 

- The application server is in charge of all the logic of sending an appropriate response to the client, based on the game state and the client's identifier (IP address).

- Game() initialised in application server

Thought Process:

I wasn't sure whether to use something like jetty or nginx but I initially felt that the problem description was simple 
enough to warrant using the JDK's built in HTTPClient, HTTPExchange, and HTTPServer. I also toyed with the idea of using TCP
sockets and just having hard coded HTTPResponses and Requests but that felt like cheating and not strictly adhering to HTTP.
The upside of this socket approach (I think) would have been getting more control over the sockets which would have made it 
easier to handle things like disconnects. If I was given a choice on which protocol to use I think websocket would be more 
suitable to handle bidirectional communication than http.

The main issues I identified in the planning phase were having the server push to the clients unprompted when appropriate, identifying users,
and detecting disconnects. For server push I implemented it as the client sending GET requests every 40ms (arbitrarily chosen)
to the server and if there is any change since the last printed state then print that. This is not a very elegant approach.
A better way to do it would have been longpolling where by the client sends an asynchronous request and the server responds
when a change has occured. I felt implementing this would have taken too long and that the constant pinging was servicable.
The issue with the constant requests is that the server is sending back a response (usually the string representation
of the board) and the checking for change is done client-side. This is quite inefficient sending over a reponse every 40ms. 
With longpolling the change is detected server-side so there is no unneccessary requests/responses sent.

I decided to use this constant request sending to handle disconnects as well. Every three seconds ,the application server checks
if the total number of requests from each player has changed since the last check. If it hasn't, the user is considered disconnected
and the game ends as the number should have increased due to the automatic requests. Another player is free to join and the game will start again.

For player identification I decided to use spoofed IP addresses. The client generates a random IP and sends it in the x-forwarded-for header.
I originally used the HttpRequest.getRemoteAddress() method but the IP returned would change based on the method the client input invoked due to
(I think) built in proxies. I realise now that there is better ways for client identification, namely cookies and that IP would not be suitable for a 
proper web deployment due to spoofing and dynamic addresses.

While there is those things I would change if doing it again, I think its a good implementation that smoothly handles things like disconnects and the game ending,
allowing for persistent play.

Bugs/Issues:

Some issues that I'm sure could be fixed relatively easily but I don't have enough time.

- Application Server cannot handle pregame disconnects (i.e. first player joins and disconnects before second player joins)
- When game is won only the winner sees the "usernameX has won." response. This is an issue caused by having the game restart
  immediately after it is won. To revert to the game ending (and requiring server to restart and clients rejoin) when someone wins - remove lines 258 - 262 in the code.


Testing:

I didn't have enough time to do unit testing. I have, however, done extensive manual integration testing (i.e. me trying to break it).


