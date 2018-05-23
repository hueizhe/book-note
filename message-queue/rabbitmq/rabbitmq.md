## rabbitmq




## Management Plugin Introduction
>The rabbitmq-management plugin provides an HTTP-based API for management and monitoring of your RabbitMQ server, along with a browser-based UI and a command line tool, rabbitmqadmin. Features include:

1. Declare, list and delete exchanges, queues bindings, users, virtual hosts and permissions.
Monitor queue length, message rates globally and per channel, data rates per connection, etc.
2. Monitor resource usage, such as file descriptors, memory use, available disk space.
3. Manage users (provided administrative permissions of the current user).
4. Export and import object definitions (vhosts, users, permissions, queues, exchanges, bindings, parameters, policies) to JSON.
5. Force close connections, purge queues.
Send and receive messages (useful in development environments and for troubleshooting).
### Getting started
The management plugin is included in the RabbitMQ distribution. To enable it, use rabbitmq-plugins:
~~~
 rabbitmq-plugins enable rabbitmq_management
~~~
* The Web UI is located at: http://server-name:15672/
* The Web UI uses an HTTP API provided by the same plugin. 
* Said API's documentation can be accessed at http://server-host:15672/api/ or our latest HTTP API documentation here.
* Download rabbitmqadmin at: http://server-name:15672/cli/
* NB: The port for RabbitMQ versions prior to 3.0 is 55672.

> To use the web UI you will need to authenticate as a RabbitMQ user (on a fresh installation the user "guest" is created with password "guest"). From here you can manage exchanges, queues, bindings, virtual hosts, users and permissions. Hopefully the UI is fairly self-explanatory.

> The management UI is implemented as a single static HTML page which makes background queries to the HTTP API. As such it makes heavy use of Javascript. It has been tested with recent versions of Firefox, Chromium and Safari, and with versions of Microsoft Internet Explorer back to 6.0.


## rabbitmqctl

 You can access the user-management with rabbitmqctl and use the command:

~~~
add_user {username} {password}
~~~
or more preferably maybe edit an existing user, or set the permissions for the new user with:
~~~
set_permissions [-p vhostpath] {user} {conf} {write} {read}
~~~
For example use the following commands: (it is important to perform these three steps even when creating a new user, if you want to be able to login to the UI console and for your programs to work without facing any permission issues)
~~~
rabbitmqctl add_user newadmin s0m3p4ssw0rd
rabbitmqctl set_user_tags newadmin administrator
rabbitmqctl set_permissions -p / newadmin ".*" ".*" ".*"
~~~
...to create a new administrator user with full access to the default / vhost.