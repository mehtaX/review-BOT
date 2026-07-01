# AWS Cloud Deployment Guide (Manual Setup)

This guide explains how to deploy the Flipkart Product Review Chatbot to AWS EC2 using Docker and Amazon Elastic Container Registry (ECR).

---

## 1. AWS IAM User Setup

To interact with AWS services, create an IAM user with programmatic access:
1. Search for **IAM** in the AWS Console.
2. Go to **Users** and click **Create user**.
3. Enter a user name (e.g., `flipkart-chatbot-deployer`).
4. Attach the following policies directly:
   - `AdministratorAccess` (or narrower permissions like `AmazonEC2ContainerRegistryFullAccess`, `AmazonEC2FullAccess`)
5. Click **Next** and then **Create user**.
6. Select the created user, go to the **Security credentials** tab, scroll down to **Access keys**, and click **Create access key**.
7. Select **Local code** or **Command Line Interface (CLI)** as the use case, click Next, and download the `.csv` file containing the **Access Key ID** and **Secret Access Key**.
8. Configure the AWS CLI locally by running:
   ```bash
   aws configure
   ```
   Provide your Access Key ID, Secret Access Key, region (e.g., `us-east-1`), and default output format.

---

## 2. Amazon ECR Repository Setup

Create a repository to store the application's Docker image:
1. Search for **Elastic Container Registry (ECR)** in the AWS Console.
2. Click **Create repository**.
3. Choose **Private** and enter a repository name (e.g., `flipkart-chatbot`).
4. Click **Create repository**.
5. Note the ECR Repository URI (looks like `<aws_account_id>.dkr.ecr.<region>.amazonaws.com/flipkart-chatbot`).

---

## 3. AWS EC2 Instance Setup

Launch a virtual machine to host the application:
1. Search for **EC2** in the AWS Console and click **Launch instance**.
2. Name the instance (e.g., `flipkart-chatbot-server`).
3. Select **Ubuntu** (LTS) as the OS Image.
4. Select an Instance Type (e.g., `t2.medium` or `t3.medium` is recommended depending on the requirements).
5. Choose or create a **Key pair** for SSH access.
6. Under **Network settings**, check the boxes to:
   - Allow SSH traffic from anywhere (or your IP)
   - Allow HTTPS traffic from the internet
   - Allow HTTP traffic from the internet
7. Launch the instance.

### Configure Security Group Port Access:
1. Go to the **Security** tab of your running EC2 instance.
2. Click on the Security Group ID.
3. Click **Edit inbound rules**.
4. Click **Add rule** and select **Custom TCP** as the type.
5. Set **Port range** to `5000` (the Flask server port) and **Source** to `0.0.0.0/0` (or restrict it to specific IPs).
6. Click **Save rules**.

---

## 4. Install Docker on the EC2 Instance

Connect to your EC2 instance via SSH and install Docker:
```bash
# Update package list
sudo apt-get update -y
sudo apt-get upgrade -y

# Download and run the Docker installation script
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add the ubuntu user to the docker group to run docker commands without sudo
sudo usermod -aG docker ubuntu
newgrp docker
```

---

## 5. Build, Tag, and Push the Docker Image

From your **local developer machine** (where the project is located), build and push the Docker image to ECR:

1. **Log in to ECR**:
   ```bash
   aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com
   ```
   *(Replace `<your-region>` and `<aws_account_id>` with your actual AWS values).*

2. **Build the Docker Image**:
   ```bash
   docker build -t flipkart-chatbot .
   ```

3. **Tag the Image for ECR**:
   ```bash
   docker tag flipkart-chatbot:latest <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/flipkart-chatbot:latest
   ```

4. **Push the Image to ECR**:
   ```bash
   docker push <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/flipkart-chatbot:latest
   ```

---

## 6. Run the Application on the EC2 Instance

Connect to the EC2 instance via SSH and execute the following:

1. **Log in to ECR on EC2**:
   ```bash
   aws ecr get-login-password --region <your-region> | docker login --username AWS --password-stdin <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com
   ```

2. **Run the Container**:
   Start the application and inject your environment variables from a `.env` file or directly via the command line:
   ```bash
   docker run -d -p 5000:5000 \
     --name flipkart-chatbot \
     -e GROQ_API_KEY="your-groq-api-key" \
     -e ASTRA_DB_API_ENDPOINT="your-astra-db-endpoint" \
     -e ASTRA_DB_APPLICATION_TOKEN="your-astra-token" \
     -e ASTRA_DB_KEYSPACE="default_keyspace" \
     -e HF_TOKEN="your-huggingface-token" \
     <aws_account_id>.dkr.ecr.<your-region>.amazonaws.com/flipkart-chatbot:latest
   ```

3. **Clean Up Old Docker Images/Containers** (Optional):
   ```bash
   docker system prune -f
   ```

4. **Verify Running Application**:
   Open a web browser and navigate to:
   `http://<your-ec2-public-ip>:5000`