# Tools

## Install snapd with root privilege
```bash
yum install epel-release -y
yum install snapd -y
systemctl enable --now snapd.socket
ln -s /var/lib/snapd/snap /snap
```
# Install k9s in centos 7
```bash
snap install k9s
echo "export K9S_HOME=/var/lib/snapd/snap/k9s/151" >> ~/.bashrc
echo "export PATH='$PATH:$K9S_HOME'" >> ~/.bashrc
source ~/.bashrc
k9s
```
![](./media/k9s_screenshot.png)

## Install lens in ubuntu 18.04
```bash
snap install kontena-lens --classic
```
![](./media/lens_screenshot.png)