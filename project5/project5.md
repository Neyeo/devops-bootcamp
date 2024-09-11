# Setup Service Discovery Using Nginx & Consul


## Project 5

|S/N | Project Tasks                                      |
|----|----------------------------------------------------|
| 1  |Deploy 4 Ubuntu Server                              |
| 2  |Allow required ports in the security group          |
| 3  |Set up architecture                                 |
| 4  |Setup Consul Server                                 |
| 5  |Setup Backend Servers                               |
| 6  |Setup Load-Balancer                                 |
| 7  |Validate Service Discovery Setup                    |

## Documentation

## Deploying 4 ubuntu server.

- Rename your EC2 instances to prevent any confusion during your project.

- Click on the **edit icon**.

![FOur ubuntu servers](./img/PROJECT54UBUNTUSERVER.jpg)


### Allow Required Ports In The Security Group

The Consul service requires specific ports to function correctly. Opening the following ports in your security group.

### Consul Servers

|S/N |Port Name  |Protocol      |Default Port   |
|----|-----------|--------------|---------------|
| 1  |DNS        |TCP and UDP   |8600           |
| 2  |HTTP API   |TCP           |8500           |
| 3  |HTTPS API  |TCP           |8501           |
| 4  |gRPC       |TCP           |8502           |
| 5  |gRPC TLS   |TCP           |8503           |
| 6  |Server RPC |TCP           |8300           |
| 7  |LAN Serf   |TCP and UDP   |8301           |
| 8  |WAN Serf   |TCP and UDP   |8302           |

- Select the **checkbox①** next to your instance, click on **Security②**, and then click on the **security group ID③**.

![editing inbounds](./img/p5afrerssh.jpg)


> [!NOTE]
Currently, the ports are being opened manually one at a time. However, You will need to do it for the remaining severs carfully also chossing the appropriate CIDR block.

---

### Setup Consul Server

- SSH into the consul server and run **`sudo apt update`** to refresh the package cache.

![ssh into consul server](./img/sshintoconsul.jpg)

- Visit the consul [**downloads**](https://developer.hashicorp.com/consul/install) page to **copy** the installation command.

Or execute the following commands to install Consul.

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install consul
```

![installing consul](./img/downloadingconsul.jpg)

- Confirm Consul installation by checking its version with the **`consul --version`** command.

![consul version](./img/consoleversionp5.jpg)

- All the Consul server configurations are located in the **`/etc/consul.d`** folder. To configure the Consul server, start by backing up the default configuration file **`consul.hcl`** by renaming it to **`consul.hcl.back`**, using the following command: **`sudo mv /etc/consul.d/consul.hcl /etc/consul.d/consul.hcl.back`**

- Generate an **encrypted key** using the **`consul keygen`** command.

![generated keygen](./img/2purposekey.jpg)

- Create a new file named **`consul.hcl`** in the **`/etc/consul.d`** directory, using the following command: **`sudo vi /etc/consul.d/consul.hcl`**

- Add the following content to the **`consul.hcl`** file, replacing **<PKY97Vcc8oEymtWCOgvntByqk7ZzmyJWFg+fl8BxMTA=>** with the encrypted key you generated:


![configuration](./img/p5firstconfig.jpg)

Save this file after adding the content.

---

- Run the following command to start the Consul server in the background: **`sudo nohup consul agent -dev -config-dir /etc/consul.d/ &`**.

![starting consul](./img/Startconsul.jpg)

> [!NOTE]
I use the **`-dev`** flag to indicate that we are running a single Consul server in development mode.

- Check the status of the Consul server with the following command: **`consul members`**.

![consul members](./img/consolemembers.jpg)

- If you visit **`<13.60.248.154>:8500`**, you should be able to access the Consul dashboard.

![cinsul dashboard](./img/firstserverec2p5up.jpg)

---

### Setup Backend Servers

We have the Consul server up and running, let's manage our Nginx backend servers more easily using service discovery. To do this, we'll install Nginx and the Consul agent on all the backend servers. The Consul agent acts like a messenger, automatically registering both the server and the Nginx service running on it with the Consul server, which acts like a central directory.

**Applying the configurations below on both backend servers:**

- SSH into the backend servers and run **`sudo apt-get update -y`** to update package information.

![ssh into first server](./img/Sshintoserver1p5.jpg)


![ssh into second server](./img/Sshintosever2project5.jpg)


![UPDATING PACKAGE](./img/Sever1p5update.jpg)


![updating package](./img/Sever2p5update.jpg)

- Install Nginx on both instances by running the following command: **`sudo apt install nginx -y`**.

![first server installing nginx](./img/Server1installnginxp5.jpg)

![Second server installing nginx](./img/Sever2installingnginxp5.jpg)

> [!NOTE]
After installing Nginx, navigate to the default HTML directory and modify the index.html file on both servers to differentiate them.

- Navigate to the HTML directory by executing the following command: **`cd /var/www/html`**.

- Open the HTML file with your preferred text editor to make edits: **`sudo vi index.html`**.

- Copy the HTML content below into the index.html file. On the second server, replace **SERVER-01** with **SERVER-02** in the HTML file to differentiate between the two backend servers.


![HTML SERVER 1](./img/SERVER1HTMLP5.jpg)

![HTML SERVER 2](./img/SERVER2htmlp5.jpg)

- Install Consul as an agent on the servers. Run the following commands to install Consul:

```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list

sudo apt update && sudo apt install consul
```

![first server installing consul](./img/firstserverinstallconsulp5.jpg)

![second server installing consul](./img/Server2p5installconsul.jpg)

- Verify that Consul is installed properly by running the following command: **`consul --version`**.

- Replace the default Consul configuration file **`config.hcl`** located in **`/etc/consul.d`** with your custom **`consul.hcl`** file.

- Rename the default file and create a new one by running the following commands:

```
sudo mv /etc/consul.d/consul.hcl /etc/consul.d/consul.hcl.back
sudo vi /etc/consul.d/consul.hcl
```

- Add the following contents to the file. Replace **`<PKY97Vcc8oEymtWCOgvntByqk7ZzmyJWFg+fl8BxMTA=>`①** with your encryption key. Also, replace **`13.60.248.154`②** with your Consul server's IP address.

```
"server" = false
"datacenter" = "dc1"
"data_dir" = "/var/consul"
"encrypt" = "PKY97Vcc8oEymtWCOgvntByqk7ZzmyJWFg+fl8BxMTA="
"log_level" = "INFO"
"enable_script_checks" = true
"enable_syslog" = true
"leave_on_terminate" = true
"start_join" = ["13.60.248.154"]
```

![backend1 configuration](./img/backend1editp5.jpg)

![backend2 configuration](./img/backend2edit.jpg)

---

- Next, we need to create a **`backend.hcl`** configuration file in the **`/etc/consul.d`** directory to register the Nginx service and its health check URLs with the Consul server. This will enable the Consul server to continuously monitor the health of the Nginx service. Use the following command to create and edit the file: **`sudo vi /etc/consul.d/backend.hcl`**.

- Add the following contents to the **`backend.hcl`** file and save it.

```
"service" = {
  "Name" = "backend"
  "Port" = 80
  "check" = {
    "args" = ["curl", "localhost"]
    "interval" = "3s"
  }
}
```

![health monitor backend server 1](./img/backendserver1healthchechp5.jpg)

![health monitor backend server 2](./img/healthcheck2jpg.jpg)

This configuration registers your backend servers with the Consul server and sets up a health check that uses curl to test the service every 3 seconds.

- Verify the configurations by executing the following command: **`consul validate /etc/consul.d`**.

![validation backend1](./img/backend1validatep5.jpg)

![validation backend2](./img/backend2validate.jpg)

- Once all configurations are complete, start the Consul agent with the following command: **`sudo nohup consul agent -config-dir /etc/consul.d/ &`**.

![starting consul on backend1](./img/backend1startconsulp5.jpg)

![starting consul on backend2](./img/backend2serverstartconsul.jpg)


- To verify if everything is working correctly, visit your Consul UI. If you see the backend listed in the UI as depicted below, it indicates that the backend has successfully registered itself with Consul.

![services](./img/projec5clickingservices.jpg)


![services](./img/backednlisted.jpg)

---

### Setup Load-Balancer

Next, setting up the load balancer to automatically update its backend server information based on the service registry maintained by Consul.
To retrieve the backend server details, we will use the **`consul-template`** binary. This tool interacts with the Consul server via API calls to fetch the backend server information. It then uses a template to substitute values and generate the **`loadbalancer.conf`** file, which is utilized by Nginx.

- Log in to the load-balancer server. Update the package information and install unzip with the following commands:

```
sudo apt-get update -y
sudo apt-get install unzip -y
```

![ssh into LB](./img/LBinstallingnginxp5.jpg)

- Install Nginx using the following command: **`sudo apt install nginx -y`**.

![installing nginx](./img/originalinstallnginxp5.jpg)

- Download the consul-template binary using the following command:

```
sudo curl -L  https://releases.hashicorp.com/consul-template/0.30.0/consul-template_0.30.0_linux_amd64.zip -o /opt/consul-template.zip

sudo unzip /opt/consul-template.zip -d  /usr/local/bin/
```

![template](./img/p5template.jpg)

- To verify the installation of consul-template, check its version with the following command: **`consul-template --version`**.

![template version](./img/p5templateversion.jpg)

- Create and edit a file named **`load-balancer.conf.ctmpl`** in the **`/etc/nginx/conf.d`** directory, using the following command: **`sudo vi /etc/nginx/conf.d/load-balancer.conf.ctmpl`**.

- Paste the following content into the file:

```
upstream backend {
 {{- range service "backend" }} 
  server {{ .Address }}:{{ .Port }}; 
 {{- end }} 
}

server {
   listen 80;

   location / {
      proxy_pass http://backend;
   }
}
```

![configuration](./img/upstreamp5.jpg)

---

> [!NOTE]
This setup keeps Nginx's backend server list in sync with Consul's. It ensures that Nginx always routes traffic to the currently available backend servers.

---

- Create a file named **`consul-template.hcl`** in the **`/etc/nginx/conf.d/`** directory. This configuration file is used by **consul-template** to specify details about the Consul server IP and the destination path where the processed load-balancer.conf file will be saved.

Use the following command to create and edit the file: **`sudo vi /etc/nginx/conf.d/consul-template.hcl`**.

- Add the following content to the file, replacing **`13.60.248.154`** with your Consul server's IP address. This configuration specifies the Consul server details, the path to the template file, the destination for the rendered Nginx configuration, and the command to reload Nginx after updating the configuration.

```
consul {
 address = "13.60.248.154:8500"

 retry {
   enabled  = true
   attempts = 12
   backoff  = "250ms"
 }
}
template {
 source      = "/etc/nginx/conf.d/load-balancer.conf.ctmpl"
 destination = "/etc/nginx/conf.d/load-balancer.conf"
 perms       = 0600
 command = "service nginx reload"
}
```

![consul template](./img/updatepp5.jpg)

- Delete the default server configuration to disable it by running the following command: **`sudo rm /etc/nginx/sites-enabled/default`**.

* The default server configuration file should be deleted to avoid inconsistencies with the server's settings.*

- Restart Nginx to apply the changes by running the following command: **`sudo systemctl restart nginx`**.

![restarting](./img/restartandtempp5.jpg)

- Once configurations are complete, start the Consul Template agent using the following command. It continuously monitors Consul for changes.

```
sudo nohup consul-template -config=/etc/nginx/conf.d/consul-template.hcl &
```

![restarting](./img/restartandtempp5.jpg)

- Upon completion, a load-balancer.conf file will be created with backend server information populated from the Consul service registry.

Now, if you access the load balancer IP in your web browser, it will display the custom HTML content from one of the backend servers. When you refresh the page, the load balancer will route your request to the other backend server, displaying its custom HTML content.

![html page](./img/loadbalancerwebsiteloading1.jpg)

![html page](./img/loadbalancingwebsite2.jpg)

This behavior occurs because the load balancer uses a round-robin algorithm by default, distributing incoming requests evenly across all available backend servers.

---

### Service Discovery Test

Now that everything is set up and running, you can test the configuration by observing what happens when you stop one of your backend servers by using the command **'sudo sytemctl stop nginx on one backend server'**.

![stop one backend server](./img/stopingoneloadbalancer.jpg)

Stop one of the backend servers. The Consul server will monitor the health of each registered service. Once a backend server is stopped, Consul will detect the server's unavailability and mark it as unhealthy. The health check for that server will fail, and it will be removed from the load balancer's active pool of servers.

![stopped one backend health](./img/loadbalacerstopped.jpg)

As a result, the load balancer will only direct traffic to the remaining healthy backend servers. This ensures that your application continues to run smoothly without any disruption to users, demonstrating the effectiveness of your service discovery and health check configuration with Consul and Nginx.


**Service Check:** These checks are specific to the services running on the nodes (in this case, Nginx). When you stopped Nginx, the service check that monitors the health of Nginx on that particular node would fail, leading to the "all service checks failed" error.

**Node checks:** These checks monitor the overall health of the node itself, which includes the underlying operating system and possibly other metrics (like CPU, memory, and disk usage). Since stopping Nginx does not necessarily mean the node is unhealthy (the node could still be up and running, responding to pings, etc.), the node checks would still pass.

---
---

#### The End Of Project 5