# Deploying application from github to AWS EC2 with AutoScaling and Load Balancing.

## First Two Role is required for CodeDeployment:
1. One for the IAM Instance Profile.
```
- Create a Role from IAM console.
- Select "AWS service" for Trusted entity type and Select "EC2" for UseCase.
- click "Next"
- Search for "codedeploy" on search text of permissions and select "AmazonEC2RoleforAWSCodeDeploy".
- click "Next"
- Give role_name and click "create role".
```
2. Another for the CodeDeployment Group.
```
- Create a another role.
- Select "AWS service" for Trusted entity type and Select "CodeDeploy" for UseCase.
- Next
- Next
- Give role_name and click "create role".
```

## Step-1: Create a launch Template: 
- Give name: Server_launchTemplate_Name
- Select New_Security_Group:
- give name : SG_Name
- Add SG Rule:
- Type: shh and SourceType: Anywhere
- Add Http rule also.

##### In Advanced Details: 
- Select "EC2_role_for_CodeDeploy_InstanceProfile" in text box of IAM instance profile.
- In UserData section:
```
  #!/bin/bash
  #This part is for installing ngnix server.
   sudo su
   apt update
   apt upgrade -y
   apt install nginx -y
   #This part is for installing the code deployment agent.
   sudo apt update 
   sudo apt install ruby-full -y
   sudo apt install wget
   cd /home/ubuntu
   wget https://aws-codedeploy-ap-southeast-1.s3.ap-southeast-1.amazonaws.com/latest/install
   chmod +x ./install
   sudo ./install auto
```

## Step-2: Create AutoScaling Group:
- Create ASG:
- give name: Server_ASG-urname
- Select the launch template(above created launch template)
- Next
- Select AZ and Subnets:
- Next
- Select "Attach to a new load Balancer":
  ``
  - Select Network LB
  - give name: server_NLB
  - select 'Internet Facing'
  - In listener and routing section:
  - select "Create a target group" in default routing and give name: server_LB-targetgroup
  ``
- Next
- Put Group Size Capacity (acc. to ur needs)
- Select "Target tracking scaling policy"
- leave default for others
- Next
- Next
- Create ASG


## Step-3: Goto Load-Balancing.
- While LB state is in Provisioning, move to next step
## Step-4: Create Git Repository:  
- Create a new repo:
- give repo name:
- select Public/Private
- create repo
- create a newfile: appspec.yml
  ```
  version: 0.0
  os: linux
  files:
    - source: /
      destination: /var/www/html/
  file_exists_behavior: OVERWRITE
  BeforeInstall:
    - location: /before_script.sh
      timeout: 300
      runas: root
  ApplicationStop:
    - location: /after_script.sh
      timeout: 300
      runas: root
  ```

  - Create a Script Folder: before_script.sh file and after_script.sh file
  - before_script.sh:
    ```
    #!/bin/bash
    rm /var/www/html/index.html
    ```

   - after_script.sh is a blank file.
   - commit changes and push code to the git.
 
##### upload or push your application to the git.
## Step-5: Creation of Pipeline:
- Goto CodeDeploy
- Under Deploy Section:
- Select Applications
- create applications
- give name:
- select "EC2/on-premises" on compute Platform.
- Create application.
- Create Deployment Group.
- give deploymentgroup_name:
- In service_role section: select the above create deploymentgroup role or paste the ARN of the above created DeploymentGroupRole.
- Choose Deployment Type: Acc. to the user needs.
- Select the above created ASG.
- In deployment settings: In deployment configuration box: choose CodeDeployDefaultOneAtATime
- Select Load Balancer:
- Select above created TargetGroup.
- Finally select Create DeploymentGroup

##### Goto CodePipeline:
- Select Pipeline
- create a new Pipeline
- give pipeline name:
- others leave default
- Next
- In Source Provider:
- Select GitHub (version2)
- Next
- Select connect to Github
- give connection_name:
- select connect to github.

- In GitHub Apps:
- Select Install a newapp
- In Repository access:
- select "Only Select Repository"
- select your repository
- save
- connect
- GitHub connection is ready now.
- Select repo in Repository Name.
- Specify Branch Name (main mostly)
- Next
- In build stage:
- skip build stage now.
- skip
- In Deploy Stage:
- Select "CodeDeploy"
- select region
- select application name
- Select DeploymentGroup
- Next
- Review
- Create Pipeline



# Now take the EC2 ip's or LB DNS and paste on the browser.
