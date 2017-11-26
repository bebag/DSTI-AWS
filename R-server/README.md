# Installing R-Studio Server on AWS 

1.  **Using user-data**

    - EC2 -\> launch instance

    - Select Amazon Linux AMI 2017.09.01, SSD Volume Type (64-bit)

    - Select the appropriate instance type according to your R workloads. For this demo, we will use t2.micro (free)

    - In Configure Instance details -\> Advanced Details -\> User data (as text), type the following bash script:

      ```
      #!/bin/bash
      # upgrade all system software to latest version
      yum -y update
      # install (latest) R
      yum -y install R
      # get the latest 64-bit version of R server
      wget https://download2.rstudio.org/rstudio-server-rhel-1.1.383-x86_64.rpm
      # install R server
      yum -y install --nogpgcheck rstudio-server-rhel-1.1.383-x86_64.rpm
      # add a user to sign-in to the R server
      useradd r-user
      echo r-user:changeme | chpasswd

      ```

      ​

    - Add a Tag: key=name and value=R-server

    - Configure security group: Change the security group name to "R-Server" and Add a rule "Custom TCP; TCP port 8787 (the default one for R server) ; source: 0.0.0.0/0"

    - Launch the instance

    - Once the status is OK, you can sign-in to the R server with the user: r-user and password: changeme on http://\<instance\_dns\_name\>:8787

2.  **Using AMI (created from Option 1)**

    - End your R session (if you have an opened one) on the instances created in option 1

    - In your "Instances" view, select the instance created in option 1, and click on 

      Actions-\>Image-\>Create Image and name it (AMI R-server) (Keep "No Reboot" unselected).

    - Click on Launch Instance

    - Select "My AMIs" on the left tab and select "AMI R-server"

    - Select the appropriate type of server (For the demo, t2.micro (free))

    - Add Tag: key=Name value="AMI-R-Server"

    - Security Group -\> Select an existing security group and select the one created in step 1 (Name: R-server)

    - Launch the instance

    - Once the status is OK, you can sign-in to the R server with the user: r-user and password: changeme on http://\<instance\_dns\_name\>:8787

