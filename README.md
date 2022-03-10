# nomad
Notes from Install Nomad

Pre-reqs:

* Commands that use `code` assume you have Visual Studio Code installed
* At this point I had already installed [Nomad, Vagrant, and Virtualbox following the steps here](
https://learn.hashicorp.com/tutorials/nomad/get-started-install?in=nomad/get-started)
* Signed up for a Trial and install

Note: In restrospect I don't believe I need Vagrant + Virtualbox for the purpose of this test

### First make the Server!!

1. Make a new directory
`mkdir nomad-test`

2. `nomad --version` to verify a successful install.
3. `code server.hcl` to make a new file and launch it in Visual Studio Code

```
# server.hcl
# Increase log verbosity
log_level = "DEBUG"

# Setup data dir
data_dir = "/tmp/server1"

# Give the agent a unique name. Defaults to hostname
name = "server1"

# Enable the server
server {
  enabled = true

  # Self-elect, should be 3 or 5 for production
  bootstrap_expect = 1
}
```
4. `nomad agent -config server.hcl` This will start a development server that includes a Web UI available at http://localhost:4646.

```
==> WARNING: Bootstrap mode enabled! Potentially unsafe operation.
==> Loaded configuration from server.hcl
==> Starting Nomad agent...
==> Nomad agent configuration:

       Advertise Addrs: HTTP: 192.168.1.207:4646; RPC: 192.168.1.207:4647; Serf: 192.168.1.207:4648
            Bind Addrs: HTTP: [0.0.0.0:4646]; RPC: 0.0.0.0:4647; Serf: 0.0.0.0:4648
                Client: false
             Log Level: DEBUG
                Region: global (DC: dc1)
                Server: true
               Version: 1.2.6

==> Nomad agent started! Log data will stream in below:

    2022-03-09T17:42:02.862-0800 [WARN]  agent.plugin_loader: skipping external plugins since plugin_dir doesn't exist: plugin_dir=/tmp/server1/plugins
    2022-03-09T17:42:02.863-0800 [DEBUG] agent.plugin_loader.docker: using client connection initialized from environment: plugin_dir=/tmp/server1/plugins
    2022-03-09T17:42:02.863-0800 [DEBUG] agent.plugin_loader.docker: using client connection initialized from environment: plugin_dir=/tmp/server1/plugins
    2022-03-09T17:42:02.863-0800 [INFO]  agent: detected plugin: name=java type=driver plugin_version=0.1.0
    2022-03-09T17:42:02.863-0800 [INFO]  agent: detected plugin: name=docker type=driver plugin_version=0.1.0
    2022-03-09T17:42:02.863-0800 [INFO]  agent: detected plugin: name=raw_exec type=driver plugin_version=0.1.0
    2022-03-09T17:42:02.863-0800 [INFO]  agent: detected plugin: name=exec type=driver plugin_version=0.1.0
    2022-03-09T17:42:02.863-0800 [INFO]  agent: detected plugin: name=qemu type=driver plugin_version=0.1.0
    2022-03-09T17:42:03.063-0800 [INFO]  nomad.raft: initial configuration: index=1 servers="[{Suffrage:Voter
```
<img width="1341" alt="image" src="https://user-images.githubusercontent.com/27694443/157572053-6212e258-7ab1-49fe-8a76-55af3ecd66c8.png">


### Now make a client! Open a new terminal window!

1. `code client1.hcl`
```
# client1.hcl
# Increase log verbosity
log_level = "DEBUG"

# Setup data dir
data_dir = "/tmp/client1"

# Give the agent a unique name. Defaults to hostname
name = "client1"

# Enable the client
client {
  enabled = true

  # For demo assume we are talking to server1. For production,
  # this should be like "nomad.service.consul:4647" and a system
  # like Consul used for service discovery.
  servers = ["127.0.0.1:4647"]
}

# Modify our port to avoid a collision with server1
ports {
  http = 5656
}
```
Run `nomad agent -config client1.hcl`
You should see the new client listed in the console!
```
==> Loaded configuration from client1.hcl
==> Starting Nomad agent...
==> Nomad agent configuration:

       Advertise Addrs: HTTP: 192.168.1.207:5656
            Bind Addrs: HTTP: [0.0.0.0:5656]
                Client: true
             Log Level: DEBUG
                Region: global (DC: dc1)
                Server: false
               Version: 1.2.6

==> Nomad agent started! Log data will stream in below:

    2022-03-09T17:46:55.493-0800 [WARN]  agent.plugin_loader: skipping external plugins since plugin_dir doesn't exist: plugin_dir=/tmp/client1/plugins
    2022-03-09T17:46:55.494-0800 [DEBUG] agent.plugin_loader.docker: using client connection initialized from environment: plugin_dir=/tmp/client1/plugins
```

<img width="1341" alt="image" src="https://user-images.githubusercontent.com/27694443/157571649-a10ad3e9-7ad9-4b7d-9a9e-08b17e1f7e2c.png">

### Now make a Nomad Job!!

1. Open a third terminal window to the same directory as before
2. Generate a new Nomad job with `nomad job init` This will create a new file called `example.nomad` in your current directory.
`Example job file written to example.nomad`
3. Optional if you want to view/edit the file: `code example.nomad`
4. Submit the job by running `nomad job run example.nomad`
5. To view status and details of the job `nomad job status example`
6. Since we are using the Docker driver, we can see the running container Nomad is with `docker ps`
7. Each job has an allocation number `nomad job status example`
8. See the details with `nomad alloc status ALLOC_ID`
9. See help options `nomad alloc`


### Resources

* [What are Nomad Job Specifications](https://www.nomadproject.io/docs/job-specification)
* [Tutorial for Starting Nomad Server and Client - very helpful](https://sysadvent.blogspot.com/2019/12/day-8-going-nomad-in-kubernetes-world.html)
* [HashiCorp Configuration Language support for Visual Studio Code - Make HCL and Nomad files pretty](https://marketplace.visualstudio.com/items?itemName=wholroyd.HCL)
