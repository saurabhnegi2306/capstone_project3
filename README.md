![AWS](https://img.shields.io/badge/AWS-EC2-orange)
![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.30-blue)
![Docker](https://img.shields.io/badge/Docker-Container-blue)
![Ansible](https://img.shields.io/badge/Ansible-Automation-red)
![Jenkins](https://img.shields.io/badge/Jenkins-Pipeline-blue)
![Python](https://img.shields.io/badge/Python-3.x-green)
![License](https://img.shields.io/badge/License-Educational-success)

---

🎬 EasyMovies

This repo contains configuration to deploy End-to-end CI/CD pipeline using **Ansible**, **AWS EC2**, **Jenkins**, **Docker**, **Docker Compose**, **GitHub Webhooks**, **Docker Hub/ECR** and **Python**.
Since Project-2 is kind of a subset of `Project-3` so this repo is targetted for `Project-3` and for `Project-2` all you need to do is remove/skip the not-required part from `Jenkinsfile` and DockerHub setup in this project. 

---

# 📑 Table of Contents

- Prerequisites
- Provision Infrastructure
- Access Jenkins
- Install Jenkins Plugins
- Configure Jenkins Worker
- Run Application Locally
- Run with Docker
- Run with Docker Compose
- Configure Jenkins Pipeline
- Configure GitHub Webhook
- Verify CI/CD Pipeline

---

# 🛠 Prerequisites ( Run on Lab/local VM ) 

Install boto3:

```bash
/opt/miniconda3/bin/python -m pip install boto3 botocore
```

Configure AWS credentials:

```bash
aws configure
```

Verify credentials:

```bash
aws sts get-caller-identity
```

Install Ansible collections dependency in ```capstone3/ansible/requirements.yml``` :

```bash
ansible-galaxy collection install -r requirements.yml
```

---

Create the Target DockerHub/AWS ECR Repo where docker image will be pushed:

a) For DockerHub : 

Sign into Dockerhub -> Create Repo -> provide repo name and create public repo ( just for the testing)

<img width="1011" height="555" alt="image" src="https://github.com/user-attachments/assets/4542d4b5-0ec5-4e04-a394-70d086150b17" />
<img width="647" height="330" alt="image" src="https://github.com/user-attachments/assets/3133eccd-bd78-4e97-a192-c7cb2938efcb" />


b) For AWS ECR ( Elastic Container Registry, AWS Version of DockerHub ) : 

Navigate to the AWS ECR
<img width="850" height="212" alt="image" src="https://github.com/user-attachments/assets/e4800f93-ee2e-4a4f-8a6e-9d250350e672" />

<img width="647" height="330" alt="image" src="https://github.com/user-attachments/assets/637580af-0ac1-4c9e-8d9f-1d76bda7e652" />

For testing purpose we can create and use a Public ECR Repo ( Anyone can pull images from this but only you can upload images via your AWS credentials ). If you create Private Repository ( Standard for Production setup) all users,application will need access to your Private ECR repo for pulling the image and you can control this access by Permission Policies. 

<img width="1442" height="309" alt="image" src="https://github.com/user-attachments/assets/85212310-58c0-402a-a503-9e69b7e3e096" />


Provide the required reponame, which you should use during docker pull/tag and push commands . 

<img width="1050" height="494" alt="image" src="https://github.com/user-attachments/assets/61ae847c-5549-4639-bd68-e0a25bf7aae0" />


Note down the repo url ```ACCOUNT_NAME/REPO_NAME``` which will be needed during docker tag/push command.
<img width="1330" height="50" alt="image" src="https://github.com/user-attachments/assets/8a44f1ae-95aa-4117-88d4-cd2b10347adb" />

Click onthe ECR Repo => View Push Commands. These commands are reference commands for you to tag a local image and push to this repo.

<img width="1641" height="329" alt="image" src="https://github.com/user-attachments/assets/bc368e70-f979-415c-9614-946156449f20" />
<img width="831" height="585" alt="image" src="https://github.com/user-attachments/assets/73181b32-65eb-4442-8f44-45424ac7ed76" />


# 🚀 Step 1 - Provision Jenkins Infrastructure ( Run on Lab/local VM ) 

Run:

```bash
ansible-playbook -i inventory.ini 1.provision.yml
```

This creates:
- New VPC, public subnet in provided region
  <img width="1190" height="72" alt="image" src="https://github.com/user-attachments/assets/4dd2f867-1fb3-415a-93a2-0dc5a5fb703b" />
  <img width="1240" height="60" alt="image" src="https://github.com/user-attachments/assets/a23b0d68-729b-4944-bb94-15a96caf0cfe" />

- Jenkins Controller Node & Jenkins Worker/Agent Node
  <img width="1297" height="89" alt="image" src="https://github.com/user-attachments/assets/56752051-ccf2-4eaf-b367-28c236b92443" />

- Security Group allowing:
  - SSH (22)  & Port Range (5000-32767) covering jenkins port 8080 and all nodeports ( 30000 to 32767 ) 
  <img width="1445" height="163" alt="image" src="https://github.com/user-attachments/assets/fc9798cb-14ae-4866-9992-f43c2193e7c0" />

It also updates:

- `inventory.ini`  with details of both nodes
- `vars.yml`      with jenkins_admin_password

and generates:

- Jenkins admin password
- SSH key (`~/.ssh/jenkins-ci-generated-key`)

<img width="1107" height="185" alt="image" src="https://github.com/user-attachments/assets/136e3949-05bf-464d-8d00-54fd848c98e8" />

This step should auto update the `inventory.ini` file with new nodes IP details as well as `vars.yml` with new `admin_password`.

<img width="1001" height="104" alt="image" src="https://github.com/user-attachments/assets/681413e0-f008-4424-9200-e076739ef851" />

<img width="512" height="297" alt="image" src="https://github.com/user-attachments/assets/520c31fc-7de5-4588-ad4f-e8da809cd372" />

---

# 🔑 Access Jenkins Nodes via CLI ( Run on Lab/local VM )

SSH to Jenkins Controller Node:

```bash
ssh -i ~/.ssh/jenkins-ci-generated-key ubuntu@<CONTROLLER-NODE-PUBLIC-IP>
sudo cat /var/lib/jenkins/secrets/initialAdminPassword      # To Retrieve Jenkins UI Admin password
systemctl status jenkins                                    # To check jenkins service status
exit
```

SSH to Jenkins Worker Node ( If Needed Run on Lab/local VM ) :

```bash
ssh -i ~/.ssh/jenkins-ci-generated-key ubuntu@<WORKER-NODE-PUBLIC-IP>
```

# 🔑 Access Jenkins UI Via Browser ( From Local Laptop Web Browser )

Login URL

```
http://<CONTROLLER-NODE-PUBLIC-IP>:8080
```
Enter Jenkins admin password ( get from previous step or fetch it from `vars.yml` ( which shoul dnow have updated password).

<img width="825" height="350" alt="image" src="https://github.com/user-attachments/assets/d79de9f6-b9b7-4367-b5a4-c8c9b53e787f" />


Install Suggested Plugins.

<img width="802" height="481" alt="image" src="https://github.com/user-attachments/assets/cd1cec05-1fe8-4e49-9b23-b37204369232" />

Create a user or continue with admin.

<img width="907" height="873" alt="image" src="https://github.com/user-attachments/assets/4b3895cc-9edb-417a-8677-24689ef7a7f2" />

Jenkins Server is now up and running. 

<img width="625" height="274" alt="image" src="https://github.com/user-attachments/assets/942770e4-3bd4-446d-ac8d-fcc83e7bda37" />
---

# 🔌 Step 2 - Install Jenkins Plugins ( Run on Lab/local VM )

```bash
ansible-playbook -i inventory.ini 2.install_plugins.yml
```

This playbook will install below Plugins. Jenkins has a lot of supported plugins specific to different tools and requirements.:

- workflow-aggregator
- workflow-multibranch
- git
- github
- github-branch-source
- ssh-slaves
- docker-workflow
- credentials-binding
- ws-cleanup
- junit

---

# 👷 Step 3 - Configure Jenkins Worker ( Run on Lab/local VM )

```bash
ansible-playbook -i inventory.ini 3.configure_worker.yml
```

<img width="934" height="172" alt="image" src="https://github.com/user-attachments/assets/659b7bc2-4a47-4e81-8b0f-65e7fa677ff9" />


Verify under **Manage Jenkins → Nodes**.

<img width="1574" height="163" alt="image" src="https://github.com/user-attachments/assets/132228c5-a888-4c6a-a16f-7ba3b0246b75" />

---

# 🖥 Run Application Locally ( Run on Lab/local VM )

```bash
pip install -r requirements.txt
python app.py
```

Browse:

```
http://<LAB-VM-PUBLIC-IP>:8081
```

<img width="953" height="177" alt="image" src="https://github.com/user-attachments/assets/c0c85970-c153-4d2f-bded-eb7fb08238b4" />

<img width="903" height="425" alt="image" src="https://github.com/user-attachments/assets/adaa4a57-fa76-4f43-8519-35f4b3a97b8c" />


Stop using CTRL+C.

---

# 🐳 Docker Deployment ( Run on Lab/local VM )

Build:

```bash
docker build -t easymovies:v1 .
```
<img width="977" height="274" alt="image" src="https://github.com/user-attachments/assets/ae0636da-2340-4e7b-9640-f6e18481789f" />

Run a new container using newly built image:

```bash
docker run -d --name easymovies -p 8081:8081 easymovies:v1
```

<img width="992" height="62" alt="image" src="https://github.com/user-attachments/assets/146099b0-811c-42aa-8173-990ce335b3d2" />

Access the UI Again on 
```
http://<LAB-VM-PUBLIC-IP>:8081
```

Stop & Remove the container:

```bash
docker stop easymovies && docker rm easymovies
```

or

```bash
docker rm -f easymovies
```

---

# 🐳 Docker Compose ( Run on Lab/local VM )

Set the ENV variables required for docker compose : 
```bash
export IMAGE_TAG=v1
export BUILD_NUMBER=v1
export GIT_COMMIT=$(git rev-parse --short HEAD)
export GIT_BRANCH=$(git branch --show-current)
export BUILD_URL=local
export BUILD_USER=labuser
export BUILD_TIME="$(date)"
```

Run the docker compose up : 
```bash
docker compose up -d
```

<img width="780" height="185" alt="image" src="https://github.com/user-attachments/assets/3018959b-4ad6-471f-b647-3e4b5e58ee42" />


Application is again available @
```bash 
http://<LAB-VM-PUBLIC-IP>:8081 
```

Notice the values of ENV being displayed correctly at bottom of the page : 

<img width="966" height="469" alt="image" src="https://github.com/user-attachments/assets/bf826897-8460-4493-87a6-3150231f6e92" />


---

# ⚙ Jenkins Pipeline

Docker Hub Credentials , Go To  :  Manage Jenkins -> Credentials

 <img width="995" height="760" alt="image" src="https://github.com/user-attachments/assets/bf86c05b-a848-48d4-80d5-266b2a66c14a" />

Select Add Credentials => Username with password 
<img width="1605" height="419" alt="image" src="https://github.com/user-attachments/assets/c15501c8-e147-4708-b062-52859afa599d" />

Store Docker Username and Personal Access Token PAT. Note down the Credentials ID D, this will be needed in Jenkisfile while using this credential. 

<img width="553" height="413" alt="image" src="https://github.com/user-attachments/assets/f685311f-eb88-4ee6-9736-e02f5bafda34" />


Create a new **Pipeline**. Go to Jenkins -> New Item -> Pipeline -> Give it a Name and NEXT 

<img width="1463" height="365" alt="image" src="https://github.com/user-attachments/assets/b3178d83-bf4f-477b-9fdf-9e5fb5f32dc7" />

Enable GitHub Project and provide your public repository URI.

<img width="779" height="269" alt="image" src="https://github.com/user-attachments/assets/1f793ccc-8e3f-4bbe-a02d-01fa79c751f4" />


This is only for documentation purpose, it allows anyone to redirect to gitHub repo when they click on GitHUb link on pipeline : 

<img width="311" height="392" alt="image" src="https://github.com/user-attachments/assets/aca94699-7e20-476b-807a-8a70763a2ae5" />


MUST Enable **GitHub hook trigger for GITScm Polling** so that Jenkins pipeline is triggered as soon as you change/commit anything in your repo to create a CICD setup.

<img width="668" height="237" alt="image" src="https://github.com/user-attachments/assets/f2080411-df97-4efa-ab48-d2dcd86513b4" />

Configure SCM and ensure to choose correct branch ( main or master or dev ) and also relative path of Jenkinsfile as compare to root directory of your repo.

<img width="823" height="868" alt="image" src="https://github.com/user-attachments/assets/6c437489-0d77-43ef-b421-23ac3cc4fd87" />

---

# 🔗 GitHub Webhook Configuration For Jenkins

navigate to GitHub Repo → Settings → Webhooks → Add Webhook

<img width="1164" height="415" alt="image" src="https://github.com/user-attachments/assets/278c8731-9985-4c2b-a797-439872cea16c" />

Payload needs to provide in below format. Also choose SSL Verification as DISABLE ( not recommended for production but OK for testing) :

```
http://<JENKINS-CONTROLLER-NODE-PUBLIC-IP>:8080/github-webhook/
```

<img width="760" height="697" alt="image" src="https://github.com/user-attachments/assets/ff5454de-f79e-419b-b15b-09114ed8c190" />

---

# ✅ Verify CI/CD From Jenkins UI 

Make a dummy change in `README.md`, `Dockerfile` or application code and push to GitHub.

Jenkins should automatically trigger a build else you can always run BUILD NOW to pull code from Repo and build->deploy.


In the build log, you should see smoke test passing and image being tagged and then finally pushed to your github repo after docker login ( if credentials are correct )

<img width="688" height="409" alt="image" src="https://github.com/user-attachments/assets/32da2826-1064-45b4-bfe1-01998eea572b" />

Verify in the DockerHub repo if new images are there ( tag will be the build no of the respective pipeline run ) : 

<img width="942" height="540" alt="image" src="https://github.com/user-attachments/assets/276adc89-f04e-41f0-a8d2-7556f681a778" />


So the whole Pipeline Flow is :

```text
GitHub Push
      ↓
Webhook ( If setup Correctly ) 
      ↓
Jenkins Pipeline
      ↓
Checkout
      ↓
Docker Build
      ↓
Pytest
      ↓
Deploy to Worker
      ↓
Application ready @ port 8081
      ↓
Image pushed to DockerHub/ECR Repo
```

Application:

```
http://<JENKINS-WORKER-NODE-PUBLIC-IP>:8081
```
You can see the build details at the bottom of the website, which should change after every build.

<img width="1107" height="361" alt="image" src="https://github.com/user-attachments/assets/d64f1cdd-f647-4fda-a570-4af0da69d809" />

---

# 🎉 CI/CD Completed

Whenever code is pushed to GitHub:

- GitHub code change triggers Jenkins via webhook.
- Jenkins pulls latest code from github repo.
- Pipeline Builds Docker image on Worker/agent node.
- Pipeline Runs Pytest inside container to test the applciation.
- Pipeline Deploys the applciation image to Worker/agent Node.
- Updated applciation docker image is pushed to DockerHub if smoke test passes.

