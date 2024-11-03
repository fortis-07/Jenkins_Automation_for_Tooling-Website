# Tooling Website Deployment Automation using Jenkins


### **Introduction**

In the previous project, we implemented horizontal scalability by adding multiple web servers to the tooling website and set up a load balancer to distribute traffic between them. This setup worked well for two or three servers, but manually configuring dozens or hundreds of servers is inefficient. 

DevOps practices aim to ensure agility, fast releases, and repeatability through automation. One of the most effective tools for automating tasks is **Jenkins**, an open-source automation server. Jenkins helps in implementing **CI/CD** (Continuous Integration/Continuous Delivery), which accelerates software delivery while maintaining code quality.

### **What is Continuous Integration (CI)?**

According to CircleCI, **Continuous Integration (CI)** is a strategy in which developers frequently commit small chunks of code to a shared repository. Each commit triggers automatic builds and tests to detect issues early and streamline development.

In this project, you will use Jenkins to automatically update the Tooling Website whenever code changes are pushed to your GitHub repository. This CI implementation will ensure that changes are deployed seamlessly.

### **Objectives**

- Enhance the architecture from **Project 8** by introducing a Jenkins server.
- Automate the deployment of source code changes from GitHub to the NFS server where the tooling website is hosted.
- Configure a Jenkins job to pull changes from GitHub and deploy them to the NFS server automatically.

### **Jenkins: Overview**

Jenkins, initially developed as **Hudson** by Kohsuke Kawaguchi at Sun Microsystems, is one of the most popular CI/CD tools available today. It integrates easily with version control tools such as GitHub and can automate tasks like building, testing, and deploying applications.

### **Updated Architecture**

Upon completing this project, the  architecture will look like this:

1. **Jenkins Server**: This server will monitor changes in the GitHub repository and trigger deployment to the NFS server.
2. **NFS Server**: Jenkins will deploy the updated source code to this server, which will sync with the web servers.
3. **Web Servers**: These RHEL web servers will serve the tooling website using the code deployed on the NFS shared storage.
4. **Load Balancer**: The existing load balancer will continue distributing traffic among the web servers for better load management.


# INSTALL AND CONFIGURE JENKINS SERVER

Step 1 – Install Jenkins server

1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "JDK11" and ssh into it

![Screenshot (474)](https://github.com/user-attachments/assets/75ec8fae-797c-4d51-b792-e16cb8592a37)

![Screenshot (446)](https://github.com/user-attachments/assets/ac8d2163-860c-4ecf-adff-14c5f275c162)

2. Install JDK (since Jenkins is a Java-based application)

```
sudo apt update
sudo apt install default-jdk-headless
```

![image](https://github.com/user-attachments/assets/b234634b-f562-41e6-aeb2-949419f45cbc)

![Screenshot (448)](https://github.com/user-attachments/assets/58a91232-4c55-46d2-a92c-a86d5e7add0c)



3. Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > \
    /etc/apt/sources.list.d/jenkins.list'
sudo apt update
sudo apt-get install jenkins
```

![image](https://github.com/user-attachments/assets/e51f304d-ece9-409e-b13c-bef00223a739)

Make sure Jenkins is up and running

```
sudo systemctl status jenkins
```

![image](https://github.com/user-attachments/assets/5f263b77-6591-4c34-9637-5df98a528ff7)


4. Perform initial Jenkins setup.
   From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080

You will be prompted to provide a default admin password



Retrieve it from your server:

```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```


![image](https://github.com/user-attachments/assets/bb8908f9-1635-4f53-9a3f-6ff90afa9cb9)

By default, Jenkins uses TCP port 8080, so the server will be accessible via this port.

Open Port 8080 in Security Group
Go to your EC2 instance's Security Group in the AWS Management Console.

Create a new Inbound Rule to allow traffic on port 8080

Type: Custom TCP
Port range: 8080
Source: Anywhere (0.0.0.0/0) or your specific IP address range for better security.


![image](https://github.com/user-attachments/assets/2e356ee8-4477-4c17-9979-38596a2f6d8a)

Then you will be asked which plugings to install – choose suggested plugins.

![image](https://github.com/user-attachments/assets/9ac7f953-d2c6-4a20-95a7-34edd1b54eb4)


Once plugins installation is done – create an admin user and you will get your Jenkins server address.

The installation is completed!

![image](https://github.com/user-attachments/assets/4ec89b06-9672-4fe0-b8ff-6d07eec36959)


## Step 2: Configure Jenkins to Retrieve Source Code from GitHub Using Webhooks

In this step, we will configure a Jenkins job that automatically retrieves the source code from our GitHub repository via webhooks. This is an essential part of automating the build and deployment process, where Jenkins will trigger a job every time a change is pushed to GitHub.

1. Enable Webhooks in Your GitHub Repository

    1. Navigate to your GitHub repository.
    2. Go to Settings > Webhooks.
    3. Click on Add webhook.
    4. In the Payload URL field, enter the URL of your Jenkins server followed by `/github-webhook/`
    ``` 
    http://<Jenkins-Server-Public-IP-Address>:8080/github-webhook/
    ``` 
    5. Set Content type to application/json.
    6. Leave Secret blank (for now), and select Just the push event under the "Which events would you like to trigger this webhook?" section.
    7. Click Add webhook.

    ![image](https://github.com/user-attachments/assets/c78b012a-0edb-4287-8667-c931b61d58d5)


   ![image](https://github.com/user-attachments/assets/76f467b8-f851-4425-8f06-2a17d95501b4)



2. Create a Jenkins Job (Freestyle Project)
    1. Log in to your Jenkins dashboard.
    2. Click New Item to create a new job.
    3. Name your job (e.g., `tooling_github`), and select Freestyle project.
    4. Click OK to proceed.
  
  ![image](https://github.com/user-attachments/assets/4076d3a3-67a9-4a13-a0ea-523512656a75)
  

![image](https://github.com/user-attachments/assets/35e15e2e-0be2-4fae-8744-b4ac27e02377)



3. Configure Git Repository in Jenkins
    1. In the job configuration page, scroll down to the Source Code Management section.
    2. Choose Git.
    3. In the Repository URL field, paste the URL of your GitHub repository (e.g., https://github.com/<your-username>/tooling).

   ![image](https://github.com/user-attachments/assets/05b0c698-af59-4d4c-a0cd-f02f8c6b64af)
 


    4. Under Credentials, either
        - Select Add to input your GitHub credentials (username and password or personal access token).
        - If you already have credentials saved, select them from the dropdown.

![image](https://github.com/user-attachments/assets/5624ca98-fa76-4cf5-8f62-152f7b88a1e7)

    5. Ensure that the Branch to build is set to */master or the branch you want Jenkins to track.



4. Manual Build Test

    - Scroll down and click Save.
    - To test the configuration manually, go to the project page and click Build Now.
    - The build process will start, and you can monitor the output by clicking the build number (#1) and selecting Console Output.
    - If the build succeeds, congratulations! You've successfully configured your first Jenkins build.

  ![image](https://github.com/user-attachments/assets/dc5d72da-24c5-4ed7-8869-3c5477dfe870)



5. Automate Builds with GitHub Webhook
    - Go back to the Configuration page of your Jenkins job by clicking Configure.
    - Scroll down to the Build Triggers section.
    - Check the box for GitHub hook trigger for GITScm polling. This enables Jenkins to trigger a build automatically when changes are pushed to the GitHub repository.
    - Save the configuration.

![image](https://github.com/user-attachments/assets/9de675c3-86aa-49d7-ab1c-d6b18c957de0)





6. Add Post-Build Actions

To ensure the build artifacts (resulting files) are saved, configure Post-build Actions:

- In the Jenkins job configuration, scroll to the Post-build Actions section.

- Choose Archive the artifacts.

- In the Files to archive field, enter `**/*` to archive all files generated during the build.

- Click Save.

![image](https://github.com/user-attachments/assets/6c988b56-ba8f-461a-9ca0-f3d39f8b1da8)



7. Test the Webhook Configuration
    - Make a change to any file in your GitHub repository (for example, edit the README.md file) and push the changes to the master branch.
    - Check your Jenkins dashboard — a new build should automatically trigger as a result of the webhook.
    - Once the build is complete, you can find the archived artifacts under:
    ```javascript
    /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
    ```
    
![image](https://github.com/user-attachments/assets/c3961a54-37df-4ae7-8daa-f2fcaeefa796)


!![image](https://github.com/user-attachments/assets/a1e7ae30-16ee-48d5-ac45-9200d819d6b1)


![image](https://github.com/user-attachments/assets/c74de087-9c88-46c4-b764-c751ebcd9adc)



We have now successfully set up an automated Jenkins job that receives files from GitHub via webhook triggers. This method is considered a "push" because GitHub initiates the file transfer when changes are pushed.

By default, the artifacts are stored on Jenkins server locally
```bash
ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/
```

![image](https://github.com/user-attachments/assets/c74de087-9c88-46c4-b764-c751ebcd9adc)


## Step 3: Configure Jenkins to Copy Files to NFS Server via SSH

In this step, we'll configure Jenkins to copy the build artifacts from the Jenkins server to your NFS server using the "Publish Over SSH" plugin. This allows the web servers to access the updated files stored on the NFS server.

1. Install the "Publish Over SSH" Plugin
    1. Log in to your Jenkins dashboard.
    2. Click on Manage Jenkins > Manage Plugins.
    3. Go to the Available tab.
    4. Search for "Publish Over SSH" and select it.
    5. Click Install without restart to install the plugin.

   ![image](https://github.com/user-attachments/assets/3b905f54-1754-40a6-ac8d-ec463711bdca)

2. Configure SSH Connection to NFS Server
    1. After installing the plugin, go to the Jenkins dashboard.

    2. Click on Manage Jenkins > Configure System.

    3. Scroll down to the Publish over SSH section.

    4. Click Add and configure the SSH connection to your NFS server:
        - Private Key: Paste the contents of your `.pem` file that you use to connect to the NFS server via SSH.
        - Name: Choose an arbitrary name for this configuration (e.g., `NFS-server-DT`).
        - Hostname: Enter the private IP address of your NFS server (you can find this in your AWS EC2 dashboard).
        - Username: Set it to `ec2-user` (assuming you're using RHEL 8).
        - Remote Directory: Enter `/mnt/apps`, which is the mount point for your web servers.
        - 

       ![image](https://github.com/user-attachments/assets/4069d066-b690-46f7-99e2-d7628ed60d85)

        

    5. Click Test Configuration to ensure the SSH connection works. You should see Success if everything is set up correctly. Make sure that port 22 is open on your NFS server to allow SSH connections.

    ![](img/success.png)


3. Configure Jenkins Job to Copy Artifacts to NFS Server
    1. Go back to your Jenkins job configuration page by selecting Configure from the project dashboard.
    2. Scroll to the Post-build Actions section.
    3. Click on Add post-build action and select Send build artifacts over SSH.
    4. In the SSH Server dropdown, select the NFS server configuration you created earlier (`NFS-server-DT`).
    5. Set the Source files field to `**` (this will send all files and directories produced by the build).
   
    6. Click Save.
   

  ![image](https://github.com/user-attachments/assets/6dc2e9fa-7758-40e2-837f-f68b4435f73b)



4. Test the Configuration
    1. Go to your GitHub repository and make a change (e.g., modify the README.md file).
    2. Push the change to the master branch. The GitHub webhook will trigger a Jenkins job automatically.
    3. Monitor the job in Jenkins and check the Console Output for the following line:

    ```makefile
    SSH: Transferred <number_of_files> file(s)
    Finished: SUCCESS
    ```
    
![image](https://github.com/user-attachments/assets/4fbdc17c-7784-48ff-9ef3-b68b6350d0cb)


 ![image](https://github.com/user-attachments/assets/bc688d49-7432-4a73-8fcd-40a6ab96f9b0)


5. Verify Files on NFS Server
    1. SSH into your NFS server:
    ```bash
    ssh -i <path-to-your-pem-file> ec2-user@<NFS-Server-IP>
    ```

    2. Navigate to the `/mnt/apps` directory:
    ```bash
    cd /mnt/apps
    ```

    3. Check the updated `README.md` file:

    ```bash
    cat README.md
    ```

    4. If the changes you made in the GitHub repository are reflected in the file, your configuration is working as expected.


![image](https://github.com/user-attachments/assets/7e7af654-8caa-45b5-8d57-38c8aedc006e)




### Lessons Learned

1. **Importance of Automation in DevOps:**
   - This project highlighted the immense value of automation in scaling DevOps operations. Instead of manually updating each web server, Jenkins allowed seamless CI/CD workflows, automatically triggering deployments upon code changes. Automation ensures consistent and reliable updates, reducing human error and saving time.

2. **Permission Management is Critical:**
   - Proper permission management on both the NFS server and Jenkins server was crucial for ensuring smooth file transfers and secure access. The right combination of user and group ownership, along with appropriate read/write permissions, played a significant role in resolving access issues while maintaining security.

3. **Efficient Use of Jenkins Plugins:**
   - Leveraging the `Publish Over SSH` plugin in Jenkins greatly simplified the process of transferring artifacts. It was a practical solution that avoided the need for custom scripts, showing how Jenkins plugins can enhance functionality and integrate different environments.

4. **Webhooks for CI/CD:**
   - Configuring GitHub webhooks to trigger Jenkins builds every time code is pushed to the repository illustrated the power of Continuous Integration (CI). It ensured the tooling website was always up-to-date with the latest changes, allowing for agile and responsive deployment workflows.

5. **Learning from Setup Errors:**
   - During this project, the setup errors, such as those with permissions, served as a reminder of the complexity involved in cloud-based deployments. However, these obstacles also offered valuable learning opportunities and reinforced the need for detailed planning when working with multi-server environments.

---

### Conclusion

By automating the tooling website deployment with Jenkins, we transformed what would have been a time-consuming manual process into a highly efficient CI/CD pipeline. This approach not only ensured quick and reliable updates but also enhanced the overall scalability of the system. Through the challenges faced, valuable lessons in server permissions, SSH access, and Jenkins configuration were learned, which will be applied in future projects to create more robust and automated systems.

This setup serves as a solid foundation for future enhancements, such as incorporating automated testing and monitoring tools, to further optimize the DevOps workflow.
