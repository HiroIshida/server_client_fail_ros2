In my private project, I found that client's request doesn't work always when the request call is made in a callback of a server. More specifically, `rclcpp::spin_until_future_complete` causes exception saying `Node has already been added to an executor`. I would like to ask how to avoid it. I suspect that this [TODO-comment](https://github.com/ros2/rclcpp/blob/fa3a6fa597c5d40b2fce46e5fb95f541e62076e7/rclcpp/include/rclcpp/executor.hpp#L336) is relevant to this issue, but I'm not sure.

To reproduce this phenomenon, I made a minimal repository in https://github.com/HiroIshida/server_client_fail_ros2.git

To reproduce the phenomenon, after installing this package, please run
```
ros2 run sample server
ros2 run sample serverclinet
ros2 run sample client
```
in order, on different terminals.

As the following diagram shows, `client` request `/trigger1`. `serverclient` then recieve `trigger1` and in its callback for `trigger1` the `serverclient` then send a request of `trigger2` to `server`. Here `client`, `serverclient` and `server` are the node names.
```
client --> (/trigger1) --> serverclient --> (/trigger2) --> server
```
As I mentioned, serverclient makes a request inside the server's callback function, thus the following error occurs:
```
h-ishida@ishidax1:~$ ros2 run sample serverclient 
[INFO] [1632708805.596528109] [serverclient]: serverclient init
[INFO] [1632708805.597011208] [serverclient]: found server
[INFO] [1632708812.997027211] [serverclient]: trigger 1 called
[INFO] [1632708812.997086511] [serverclient]: trigger 2 requested
terminate called after throwing an instance of 'std::runtime_error'
  what():  Node has already been added to an executor.
```
