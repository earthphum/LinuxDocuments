### How to install Docker from pi 5
i mostly install via pacman so i just write a tutorial for myself to install on pi
```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```
then
```bash
sudo usermod -aG docker $USER
```
Check Is Docker actuclly worked?
```bash
docker --version
sudo systemctl status docker
```
test
```bash
sudo docker run -d -p 80:80 nginx
```
