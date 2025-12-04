 
# Flask App with MySQL Docker Setup

This is a simple Flask app that interacts with a MySQL database. The app allows users to submit messages, which are then stored in the database and displayed on the frontend.

<img width="1232" height="389" alt="image" src="https://github.com/user-attachments/assets/c4624ad0-8e94-408b-9b26-c00aea727cdb" />

## Prerequisites

Before you begin, make sure you have the following installed:

- Docker
- Git (optional, for cloning the repository)

## Setup

1. Clone this repository (if you haven't already):

   ```bash
   git clone https://github.com/your-username/your-repo-name.git
   ```

2. Navigate to the project directory:

   ```bash
   cd your-repo-name
   ```

3. Create a `.env` file in the project directory to store your MySQL environment variables:

   ```bash
   touch .env
   ```

4. Open the `.env` file and add your MySQL configuration:

   ```
   MYSQL_HOST=mysql
   MYSQL_USER=your_username
   MYSQL_PASSWORD=your_password
   MYSQL_DB=your_database
   ```

## Begineer Guid

1. Docker Installation .

```
sudo apt install docker.io
```


2. Build Docker-image form Dockerfile
```
docker build -t flaskapp .
```
3. Check image is build
```
docker images
```

4. Create a netowk to have communication between the flaskapp and mysql container
 
 ```
 docker network create twotire

 ```
5. Now run the Containers
```
docker run -d  -p 3306:3306 --name=mysql  --network=twotire   -e MYSQL_ROOT_PASSWORD=admin   -e MYSQL_USER=admin   -e MYSQL_PASSWORD=admin   -e MYSQL_DATABASE=myDB   mysql:5.7
docker run -d -p 5000:5000 --name=flaskapp  --network=twotire   -e MYSQL_HOST=mysql   -e MYSQL_USER=admin   -e MYSQL_PASSWORD=admin   -e MYSQL_DB=myDB   flaskapp:latest


```
6. Check is the applicaiton is runing on `PbulicIP:5000`.  if security group is not configured then edit its inboud rule for port 5000.

7. To go inside mysql container take container ID from `docker ps` then

```
docker exec -it CONTAINER-ID bash
```

-  for mysql container

```
mysql -u root -p
```
-  Now  to see show databases

```
show databases;
```

-  to do inside a database 

```

use myDB; # or your database name 

```

-  then to list the tables inside that database

```
show tables;
```

8. Push the Docker image to DockerHub
```
docker login
docker tag flaskapp:latest USERNAME/flaskapp:latest 
docker push  USERNAME/flaskapp:latest 
```


9. Install Docker compose
```
sudo apt install docker-compose
```

Now part goes to docker-compose 

## Usage
1. Start the containers using Docker Compose:

```
 docker-compose up --build
 ```

2. Access the Flask app in your web browser:

   - Frontend: http://localhost
   - Backend: http://localhost:5000

3. Create the `messages` table in your MySQL database:

   - Use a MySQL client or tool (e.g., phpMyAdmin) to execute the following SQL commands:
   
     ```sql
     CREATE TABLE messages (
         id INT AUTO_INCREMENT PRIMARY KEY,
         message TEXT
     );
     ```

4. Interact with the app:

   - Visit http://localhost to see the frontend. You can submit new messages using the form.
   - Visit http://localhost:5000/insert_sql to insert a message directly into the `messages` table via an SQL query.

## Cleaning Up

To stop and remove the Docker containers, press `Ctrl+C` in the terminal where the containers are running, or use the following command:

```bash
docker-compose down
```

## To run this two-tier application using  without docker-compose

- First create a docker image from Dockerfile
```bash
docker build -t flaskapp .
```

- Now, make sure that you have created a network using following command
```bash
docker network create twotier
```

- Attach both the containers in the same network, so that they can communicate with each other

i) MySQL container 
```bash
docker run -d \
    --name mysql \
    -v mysql-data:/var/lib/mysql \
    --network=twotier \
    -e MYSQL_DATABASE=mydb \
    -e MYSQL_ROOT_PASSWORD=admin \
    -p 3306:3306 \
    mysql:5.7

```
ii) Backend container
```bash
docker run -d \
    --name flaskapp \
    --network=twotier \
    -e MYSQL_HOST=mysql \
    -e MYSQL_USER=root \
    -e MYSQL_PASSWORD=admin \
    -e MYSQL_DB=mydb \
    -p 5000:5000 \
    flaskapp:latest

```

## Notes

- Make sure to replace placeholders (e.g., `your_username`, `your_password`, `your_database`) with your actual MySQL configuration.

- This is a basic setup for demonstration purposes. In a production environment, you should follow best practices for security and performance.

- Be cautious when executing SQL queries directly. Validate and sanitize user inputs to prevent vulnerabilities like SQL injection.

- If you encounter issues, check Docker logs and error messages for troubleshooting.




# Setup K8s-Cluster on EC2

## 1- Create 2 EC2 instances. One is Master Node and other is Worker Node
## 2- Run those commands on both Nodes

STEP 1 — Disable Swap (Master + Workers)

```

sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

STEP 2 — Load Kernel Modules (Master + Workers)
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

STEP 3 — Apply sysctl Settings

```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

sudo sysctl --system

```

STEP 4 — Install Docker Engine (NOT containerd)

```
sudo apt-get update
sudo apt-get install -y ca-certificates curl gnupg

sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] \
  https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

Enable Docker:

```
sudo systemctl enable docker
sudo systemctl start docker
```


STEP 5 - Install cri-dockerd (Required for kubeadm + Docker)

```
wget https://github.com/Mirantis/cri-dockerd/releases/download/v0.3.21/cri-dockerd_0.3.21.3-0.ubuntu-jammy_amd64.deb
sudo dpkg -i cri-dockerd_0.3.21.3-0.ubuntu-jammy_amd64.deb
sudo apt-get install -f -y
sudo systemctl enable cri-docker.service
sudo systemctl enable --now cri-docker.socket
sudo systemctl status cri-docker.socket


```

STEP 6 — Install Kubernetes (kubeadm, kubelet, kubectl)


```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes.gpg] \
  https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl

```


## 3- Only on Master Node
1- Initialize the Cluster:

```
sudo kubeadm init --cri-socket unix:///var/run/cri-dockerd.sock

```

2- Set Up Local kubeconfig:

```
mkdir -p "$HOME"/.kube
sudo cp -i /etc/kubernetes/admin.conf "$HOME"/.kube/config
sudo chown "$(id -u)":"$(id -g)" "$HOME"/.kube/config
```

3- Install a Network Plugin (Calico):

```
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.26.0/manifests/calico.yaml
```

4- Generate Join Command:

```
kubeadm token create --print-join-command
```

> Copy this generated token for next command.


## Execute on ALL of your Worker Nodes

1- Perform pre-flight checks:

```
sudo kubeadm reset pre-flight checks
```

2- Paste the join command you got from the master node and append --v=5 at the end:

```
sudo kubeadm join <private-ip-of-control-plane>:6443 --token <token> --discovery-token-ca-cert-hash sha256:<hash> --cri-socket 
"unix:///run/containerd/containerd.sock" 
```

Note: When pasting the join command from the master node:

- > Add sudo at the beginning of the command
- > Add --v=5 at the end
- > specify CRI docker,Since Kubernetes removed native Docker support after v1.24, Docker now requires the cri-dockerd shim to communicate with kubelet.
Example format:

```
sudo <paste-join-command-here> --cri-socket unix:///var/run/cri-dockerd.sock 
```


