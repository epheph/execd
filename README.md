你好！
很冒昧用这样的方式来和你沟通，如有打扰请忽略我的提交哈。我是光年实验室（gnlab.com）的HR，在招Golang开发工程师，我们是一个技术型团队，技术氛围非常好。全职和兼职都可以，不过最好是全职，工作地点杭州。
我们公司是做流量增长的，Golang负责开发SAAS平台的应用，我们做的很多应用是全新的，工作非常有挑战也很有意思，是国内很多大厂的顾问。
如果有兴趣的话加我微信：13515810775  ，也可以访问 https://gnlab.com/，联系客服转发给HR。
# execd

A very lightweight SSH server frontend written in Go. The backend auth and execution logic is handled by commands you specify, letting you customize its behavior via your own scripts/executables.

## Using execd
```
Usage: ./execd [options] <auth-handler> <exec-handler>

  -d=false: debug mode displays handler output
  -e=false: pass environment to handlers
  -k="": pem file of private keys (read from SSH_PRIVATE_KEYS by default)
  -h="": host ip to listen on
  -p="22": port to listen on
  -s=false: run exec handler via SHELL
```


#### auth-handler $user $key

 * `$user` argument is the name of the user being used to attempt the connection
 * `$key` argument is the public key data being provided for authentication

auth-handler is the path to an executable that's used for authenticating incoming SSH connections. If it returns with exit status 0, the connection will be allowed, otherwise it will be denied. The output of auth-handler must be empty, or key-value pairs in the form `KEY=value` separated by newlines, which will be added to the environment of exec-handler.

Although auth-handler is required, you can still achieve no-auth open access by providing `/usr/bin/true` as auth-handler.


#### exec-handler $command...

 * `$command...` arguments is the command line that was specified to run by the SSH client

exec-handler is the path to an executable that's used to execute the command provided by the client. The meaning of that is quite flexible. All of the stdout and stderr is returned to the client, including the exit status. If the client provides stdin, that's passed to the exec-handler. Any environment variables provided by the auth-handler output will be available to exec-handler, as well as `$USER` and `$SSH_ORIGINAL_COMMAND` environment variables.


## Examples

**These examples bypass all authentication and allow remote execution, *do not* run this in production.**

Echo server (with accept-all auth):

```
server$ execd $(which true) $(which echo)
client$ ssh $SERVER "hello world"
hello world
```

Echo host's environment to clients (with accept-all auth):

```
server$ execd -e $(which true) $(env)
client$ ssh $SERVER
USER=root
HOME=/root
LANG=en_US.UTF-8
...
```

Bash server (with accept-all auth):

```
server$ execd $(which true) $(which bash)
client$ ssh $SERVER
bash-4.3$ echo "this is a bash instance running on the server"
this is a bash instance running on the server
```


## Credit / History

It started with [gitreceive](https://github.com/progrium/gitreceive), which was then used in [Dokku](https://github.com/progrium/dokku). Then I made a more generalized version of gitreceive, more similar to execd, called [sshcommand](https://github.com/progrium/sshcommand), which eventually replaced gitreceive in Dokku. When I started work on Flynn, the first projects included [gitreceived](https://github.com/flynn/gitreceived) (a standalone daemon version of gitreceive). This was refined by the Flynn community, namely Jonathan Rudenberg. 

Eventually I came to realize gitreceived could be generalized / simplified further in a way that could be used *with* the original gitreceive, *and* replace sshcommand, *and* be used in Dokku, *and* potentially replace gitreceived in Flynn. This project takes learnings from all those projects, though mostly gitreceived.

## Sponsors

This project was made possible thanks to [DigitalOcean](http://digitalocean.com).

## License

BSD
