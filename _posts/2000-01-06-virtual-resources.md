---
title: "Virtual Resources"
bg:  fnavy
color: white
fa-icon: cubes
---

A virtual resource is disposable asset similar to a deployment environment resource that can be drawn on to effectively test assumptions. 

### Docker Virtual Resource

Testify provides support for virtual resources based on Docker Containers. Before you can use virtual resources in your integration and system tests you will need to install and configure Docker on your system.

**Install Docker**

To install Docker please follow the [Docker instalation instructions for your OS platform](https://docs.docker.com/engine/installation/).

**Configuring Docker**

Testify uses Docker Remote API to pull images and manage Docker containers. By default Docker Remote API is not enabled. To enable Docker Remote API follow the bellow instructions for your OS platform.

---

#### Ubuntu 14.04 / Mint 17

---

* After you install add yourself to the `docker` group and reboot your system:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
usermod -a -G docker $USER
reboot
EOC
{% endhighlight %}
* Backup any existing docker `daemon.json` configuration file and create a new one that enables local and remote docker API hosts:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
mkdir -p /etc/docker
mv /etc/docker/daemon.json "/etc/docker/daemon.json.$(date -d "today" +"%Y%m%d%H%M")" \
 2>/dev/null
tee -a /etc/docker/daemon.json >/dev/null <<'EOF'
{
  "hosts": [
    "unix:///var/run/docker.sock",
    "tcp://127.0.0.1:2375"
  ]
}
EOF
EOC
{% endhighlight %}
* Since we are using a `daemon.json` file to configure the docker daemon we need to insure that daemon is not configured by our system docker service file:
{% highlight bash linenos=table %}
sudo sed -i 's/ExecStart=.*/ExecStart=\/usr\/bin\/dockerd/g' /lib/systemd/system/docker.service
{% endhighlight %}
* Restart the docker service:
{% highlight bash linenos=table %}
sudo service docker restart
{% endhighlight %}
* Test that the remote api is working:
{% highlight bash linenos=table %}
docker -H tcp://127.0.0.1:2375 ps
{% endhighlight %}

---

#### Ubuntut 16.04 / Mint 18 / Debian 8

---

* After you install add yourself to the `docker` group and reboot your system:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
usermod -a -G docker $USER
reboot
EOC
{% endhighlight %}
* Backup any existing docker `daemon.json` configuration file and create a new one that enables local and remote docker API hosts:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
mkdir -p /etc/docker
mv /etc/docker/daemon.json "/etc/docker/daemon.json.$(date -d "today" +"%Y%m%d%H%M")" \
 2>/dev/null
tee -a /etc/docker/daemon.json >/dev/null <<'EOF'
{
  "hosts": [
    "fd://",
    "tcp://127.0.0.1:2375"
  ]
}
EOF
EOC
{% endhighlight %}
* Since we are using a `daemon.json` file to configure the docker daemon we need to insure that daemon is not configured by the system docker service file:
{% highlight bash linenos=table %}
sudo sed -i 's/ExecStart=.*/ExecStart=\/usr\/bin\/dockerd/g' /lib/systemd/system/docker.service
{% endhighlight %}
* Restart the docker service:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
systemctl daemon-reload
systemctl restart docker
EOC
{% endhighlight %}
* Test that the remote api is working:
{% highlight bash linenos=table %}
docker -H tcp://127.0.0.1:2375 ps
{% endhighlight %}

---

#### CentOS 7 / RHEL 7 / Oracle Linux 7

---

* After you install add yourself to the `docker` group and reboot your system:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
usermod -a -G docker $USER
reboot
EOC
{% endhighlight %}
* Backup any existing docker `daemon.json` configuration file and create a new one that enables local and remote docker API hosts:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
mkdir -p /etc/docker
mv /etc/docker/daemon.json "/etc/docker/daemon.json.$(date -d "today" +"%Y%m%d%H%M")" \
 2>/dev/null
tee -a /etc/docker/daemon.json >/dev/null <<'EOF'
{
  "storage-driver": "devicemapper",
  "hosts": [
    "unix:///var/run/docker.sock",
    "tcp://127.0.0.1:2375"
  ]
}
EOF
EOC
{% endhighlight %}
* Since we are using a `daemon.json` file to configure the docker daemon we need to insure that daemon is not configured by our system docker service file:
{% highlight bash linenos=table %}
sudo sed -i 's/ExecStart=.*/ExecStart=\/usr\/bin\/dockerd/g' /lib/systemd/system/docker.service
{% endhighlight %}
* Restart the docker service:
{% highlight bash linenos=table %}
sudo -s -- <<EOC
systemctl daemon-reload
systemctl restart docker
EOC
{% endhighlight %}
* Test that the remote api is working:
{% highlight bash linenos=table %}
docker -H tcp://127.0.0.1:2375 ps
{% endhighlight %}

---

#### Windows

---

TODO

---

#### Mac OS X

---

TODO

<!--
**Mac / Windows (Docker-Machine)**

On Macs and Windows systems Docker installation and configuration process is a
bit different than on Linux systems as they require a VM (VirtualBox). Follow
the Docker [Mac Installation][docker-mac-install] and
[Windows Installation][docker-windows-install] documentation to install and
configure Docker on your system. Next you will need to configure your VM to
disable Docker Remote API SSL configuration and enable VM port forwarding.

- Capture your VM's SSH port:
{% highlight bash linenos=table %}
SSHPORT=$(docker-machine inspect default | \
grep SSHPort | \
awk '{ print $2 }' | \
sed s/\"//g | \
sed s/,//g)
{% endhighlight %}
- SSH into the Docker VM:
{% highlight bash linenos=table %}
#if you don't have sshpass installed you can enter the password 'tcuser' manually.
sshpass -p tcuser ssh -p $SSHPORT docker@localhost
{% endhighlight %}
- Update docker2boot profile configuration and insure DOCKER_TLS is set to no:
{% highlight bash linenos=table %}
tce-load -w -i nano
sudo nano /var/lib/boot2docker/profile
{% endhighlight %}
{% highlight bash linenos=table %}
CACERT=/var/lib/boot2docker/ca.pem
DOCKER_HOST='-H tcp://0.0.0.0:2376'
DOCKER_STORAGE=aufs
DOCKER_TLS=no
SERVERKEY=/var/lib/boot2docker/server-key.pem
SERVERCERT=/var/lib/boot2docker/server.pem
{% endhighlight %}
- Restart Docker and Exit
{% highlight bash linenos=table %}
sudo /etc/init.d/docker restart
exit
{% endhighlight %}
- Setup VirtualBox Port Forwarding:
{% highlight bash linenos=table %}
#assumes your VM is named "default"
VBoxManage controlvm default natpf1 report_api,tcp,127.0.0.1,2375,,2376
{% endhighlight %}
-->

---

#### Docker Remote API and Testify

---

By default Testify communicates with Docker through Docker Remote API on
non-secure `http://127.0.0.1:2375` URL endpoint. If you wish to use an endpoint
with a different IP address and port or are using a Docker-Machine, or want to
use secure-communication then you will need to explictly configure the Docker
Client used by Testify using `@ConfigHandler` (not recommended):
{% highlight java linenos=table %}
@Config
public void configure(DockerClientConfig.DockerClientConfigBuilder builder) {
    builder.withUri("http://192.168.99.100:2376");
    //add additional configuration such as registry configuration here
}
{% endhighlight %}


[docker-configuration]: https://docs.docker.com/engine/articles/configuring
[docker-mac-install]: https://docs.docker.com/engine/installation/mac/
[docker-windows-install]: https://docs.docker.com/engine/installation/windows/