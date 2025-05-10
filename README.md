# Building a Secure and Communicative VPC with AWS CloudFormation.

![](https://miro.medium.com/v2/resize:fit:640/1*cZetZPRM8P3BjFAgpyf5XQ.png)

In this project, I will describe the process of deploying a **Bastion Host using AWS CloudFormation**. CloudFormation is a powerful tool that allows you to automate the management of AWS infrastructure using template files. By leveraging CloudFormation, I was able to quickly and effectively create a Bastion Host, significantly simplifying the process of configuring and managing infrastructure.

![](https://miro.medium.com/v2/resize:fit:700/1*bCyxDE4Cx-bRvWwYT9xZwg.png)

# **Writing the CloudFormation Template**

To write the CloudFormation template, I used the Visual Studio Code (VSCode) editor. VSCode is a convenient tool that offers many extensions to facilitate working with CloudFormation templates, such as syntax highlighting and auto-completion. This made the process of creating and editing template files much faster and more efficient.

## **Creating a VPC**

The first step in creating the template was writing a ***YAML* file** named `cloudformation-vpc2.yaml`. At the beginning of the template, I specified the **template version** and a brief **description**:

```
Resources:
  BastionHostVPC:
    Type: 'AWS::EC2::VPC'
```

![](https://miro.medium.com/v2/resize:fit:700/1*agg4hWbBDglhJZQJsSYYrA.png)

Next, I began defining the **resources**, starting with the **VPC** where all other resources will reside. A VPC is the primary organizational unit in AWS that allows for the creation of isolated networks. In my template, I defined a VPC named `BastionHostVPC` with the **CIDR block** of `183.14.0.0/16`, **DNS support enabled**, and **DNS hostnames enabled**. Here is a snippet of the template code:

```
Resources:
  BastionHostVPC:
    Type: 'AWS::EC2::VPC'
    Properties:
      CidrBlock: '183.14.0.0/16'
      EnableDnsSupport: true
      EnableDnsHostnames: true
      Tags:
        - Key: 'Name'
          Value: 'BastionHostVPC'
```

![](https://miro.medium.com/v2/resize:fit:700/1*1iVtV6eDltaedZVu1lHQYw.png)

## **Creating Public Subnet 1A**

The next step was to create the `PublicSubnet1A` in (**AZ-1a**). This subnet was defined with the **CIDR block** `183.14.1.0/24` and assigned to the **first Available Zone in the region**. In the template, I also specified that instances launched in this subnet would **automatically receive public IP addresses**.

```
# Public Subnet 1A
PublicSubnet1A:
  Type: 'AWS::EC2::Subnet'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    CidrBlock: '183.14.1.0/24'
    AvailabilityZone: !Select [0, !GetAZs '']
    MapPublicIpOnLaunch: true
    Tags:
      - Key: 'Name'
        Value: 'PublicSubnet1A
```

![](https://miro.medium.com/v2/resize:fit:700/1*456KVGHp_ge3iIZ3tJeyEg.png)

## **Creating Private Subnet 1A**

The next step was to deploy the **Private Subnet** `AppPrivateSubnet1A` (**AZ-1a**). This subnet was defined with the **CIDR block** of `183.14.2.0/24`.

```
# App Private Subnet 1A
AppPrivateSubnet1A:
  Type: 'AWS::EC2::Subnet'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    CidrBlock: '183.14.2.0/24'
    AvailabilityZone: !Select [0, !GetAZs '']
    Tags:
      - Key: 'Name'
        Value: 'AppPrivateSubnet1A'
```

![](https://miro.medium.com/v2/resize:fit:700/1*fxOp_nwFYG73AQnPv-4alw.png)

## **Creating Data Private Subnet 1A**

The next step was to create the **Private Subnet** `DataPrivateSubnet1A`

(**AZ-1a**). This subnet was defined with **the CIDR block** `183.14.3.0/24`.

![](https://miro.medium.com/v2/resize:fit:700/1*mgbeaiEYGS8QevtJ_Koh7Q.png)

## **Creating Subnets in the Second Availability Zone**

Next, I repeated these steps for the **second Availability Zone** (**AZ-1b**). In AWS, the first Availability Zone is indexed as `0` and the second as `1`. This is because AWS indexing starts from zero, so when selecting Availability Zones using the `!Select` function, we use `0` for the first zone and `1` for the second. I created the **Public Subnet, App Private Subnet,** and **Data Private Subnet** in (**AZ-1b**).

```
# Public Subnet 1B
PublicSubnet1B:
  Type: 'AWS::EC2::Subnet'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    CidrBlock: '183.14.4.0/24'
    AvailabilityZone: !Select [1, !GetAZs '']
    MapPublicIpOnLaunch: true
    Tags:
      - Key: 'Name'
        Value: 'PublicSubnet1B'

# App Private Subnet 1B
AppPrivateSubnet1B:
  Type: 'AWS::EC2::Subnet'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    CidrBlock: '183.14.5.0/24'
    AvailabilityZone: !Select [1, !GetAZs '']
    Tags:
      - Key: 'Name'
        Value: 'AppPrivateSubnet1B'

# Data Private Subnet 1B
DataPrivateSubnet1B:
  Type: 'AWS::EC2::Subnet'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    CidrBlock: '183.14.6.0/24'
    AvailabilityZone: !Select [1, !GetAZs '']
    Tags:
      - Key: 'Name'
        Value: 'DataPrivateSubnet1B'
```

![](https://miro.medium.com/v2/resize:fit:700/1*AxY1iryaQXO8ZLIlv1q1yA.png)

## **Creating the Internet Gateway**

The next step was to define and deploy the **Internet Gateway**. The Internet Gateway **allows instances** in the Public Subnet **to communicate with the internet**. In my template, I defined the it and added appropriate tags for easier identification.

```
# Internet Gateway
InternetGateway:
  Type: 'AWS::EC2::InternetGateway'
  Properties:
    Tags:
      - Key: 'Name'
        Value: 'InternetGateway'
```

![](https://miro.medium.com/v2/resize:fit:700/1*Z73oa5p-dbykuSagxVMcMQ.png)

## **Creating the Gateway Attachment**

The next step was to create the **Gateway Attachment**, which **assigns the Internet Gateway to our VPC**. This connection **is necessary to allow instances in the Public Subnets to access the internet**. In my template, I specified that the VPC to which the Internet Gateway should be attached is `BastionHostVPC`.

```
# Gateway Attachment
AttachGateway:
  Type: 'AWS::EC2::VPCGatewayAttachment'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    InternetGatewayId: !Ref 'InternetGateway'
```

## **Creating the Route Table**

The next step was to create the **Route Table**. The Route Table **specifies routing rules for network traffic within the VPC**. In my template, I defined the Route Table and assigned it to `BastionHostVPC`, using appropriate tags to facilitate its identification.

```
# Route Table
RouteTable:
  Type: 'AWS::EC2::RouteTable'
  Properties:
    VpcId: !Ref 'BastionHostVPC'
    Tags:
      - Key: 'Name'
        Value: 'RouteTable'
```

![](https://miro.medium.com/v2/resize:fit:700/1*jRhgCAwYHPtiNNzl1IgF5A.png)

## **Defining the Public Route**

The next step was to define the route (**Public Route**) in our **Route Table**. In this entry, **I specified which Route Table it should refer to** and **where it should direct the traffic**, which is the Internet Gateway. I also **opened the network traffic to the internet** by defining the **CIDR block** `0.0.0.0/0`, **allowing incoming and outgoing traffic from the internet**.

```
# Public Route
PublicRoute:
  Type: 'AWS::EC2::Route'
  DependsOn: 'AttachGateway'
  Properties:
    RouteTableId: !Ref 'RouteTable'
    GatewayId: !Ref 'InternetGateway'
    DestinationCidrBlock: '0.0.0.0/0'
```

![](https://miro.medium.com/v2/resize:fit:700/1*38jS4VvJKc6uXqH-q6AstA.png)

## **Route Table Association**

The next step was to **associate the Route Table with the respective subnets**. This **ensures that my subnets do not use the default routing tables**. I associated the Route Table with the Public Subnets `PublicSubnet1A` and `PublicSubnet1B` to enable them to use the custom routing table defined earlier.

```
# Route Table Association for Public Subnet 1A
PublicSubnet1ARouteTableAssociation:
  Type: 'AWS::EC2::SubnetRouteTableAssociation'
  Properties:
    SubnetId: !Ref 'PublicSubnet1A'
    RouteTableId: !Ref 'RouteTable'

# Route Table Association for Public Subnet 1B
PublicSubnet1BRouteTableAssociation:
  Type: 'AWS::EC2::SubnetRouteTableAssociation'
  Properties:
    SubnetId: !Ref 'PublicSubnet1B'
    RouteTableId: !Ref 'RouteTable'
```

![](https://miro.medium.com/v2/resize:fit:700/1*H_izHsy9BQ4pGfurkapXSw.png)

# **Placing the Bastion Host in Public Subnet 1A**

The next part of the project was **to place the Bastion Host** in `PublicSubnet1A`, similar to what I described in my previous blog post using the AWS console.

I created an EC2 instance and assigned it to Availability Zone 1A. Then, I **specified the instance type** as `t2.micro` and **the image** (**AMI**) from which the system would be built. The instance type defines resources like **CPU and memory available to the instance**.

Next, I assigned a **Security Group** to the instance, which I will define in the next section of the code, and **specified the SSH key that will be used for connections.**

```
# Bastion Host Instance
BastionHostInstance:
  Type: 'AWS::EC2::Instance'
  Properties:
    AvailabilityZone: !Select [0, !GetAZs '']
    SubnetId: !Ref 'PublicSubnet1A'
    InstanceType: 't2.micro'
    ImageId: 'ami-0910ce22fbfba68e1'
    SecurityGroupIds:
      - !Ref 'BastionHostSG'
    KeyName: 'BastionHostKeyPair'
    Tags:
      - Key: 'Name'
        Value: 'BastionHostInstance'
```

![](https://miro.medium.com/v2/resize:fit:700/1*6-Wk7_OMWycftfursYucVA.png)

I used an AMI image obtained from the **AMI Catalog in the AWS console**. An AMI (**Amazon Machine Image**) is a **pre-configured environment that includes an operating system and other software**, which can be used to launch EC2 instances.

![](https://miro.medium.com/v2/resize:fit:700/1*l02-OqOvaO2uYj6K4rj1Cw.png)

## **Defining the Security Group for Bastion Host**

This part of the code creates the **Security Group** I mentioned earlier. I named it `EnableSSH` because it **allows SSH connections from my computer**. In the template, I defined the **group name**, the **VPC it belongs to**, and the **inbound traffic rules**. The **protocol** used for the connection is **TCP**, the **IP address of my computer is specified**, and **port 22** is opened for **SSH connections**.

```
# Bastion Security Group
BastionHostSG:
  Type: 'AWS::EC2::SecurityGroup'
  Properties:
    GroupDescription: 'EnableSSH'
    GroupName: 'BastionHostSecurityGroup'
    VpcId: !Ref 'BastionHostVPC'
    SecurityGroupIngress:
      - IpProtocol: 'tcp'
        CidrIp: '83.24.86.53/32'
        FromPort: 22
        ToPort: 22
```

![](https://miro.medium.com/v2/resize:fit:700/1*k3QhXRcZHlQtcrOJarny7w.png)

## **Coding App Instance 1A**

Coding the `AppInstance1A`, which will be located in the `AppPrivateSubnet1A` subnet, **is similar to coding the Bastion Host**. In the CloudFormation template, we define similar properties: Availability Zone, Subnet, Instance type, AMI Image, Security Group, and SSH key.

```
# App Instance 1A
AppInstance1A:
  Type: 'AWS::EC2::Instance'
  Properties:
    AvailabilityZone: !Select [0, !GetAZs '']
    SubnetId: !Ref 'AppPrivateSubnet1A'
    InstanceType: 't2.micro'
    ImageId: 'ami-0910ce22fbfba68e1'
    SecurityGroupIds:
      - !Ref 'AppInstance1ASG'
    KeyName: 'BastionHostKeyPair'
    Tags:
      - Key: 'Name'
        Value: 'AppInstance1A'
```

![](https://miro.medium.com/v2/resize:fit:700/1*rTRuMWDDlxmBy1KplIoVFA.png)

## **Defining the Security Group for App Instance 1A**

Next, we define the **Security Group for this instance**. It differs from the previous Security Group in that this time **we do not specify a specific IP** for SSH connections but use `SourceSecurityGroupId`. This means that instead of allowing traffic from a specific IP address, **we allow traffic from instances assigned to a specified SG**, in this case, from the Bastion Host.

```
# App Instance 1A Security Group
AppInstance1ASG:
  Type: 'AWS::EC2::SecurityGroup'
  Properties:
    GroupDescription: 'Allow SSH from Bastion'
    VpcId: !Ref 'BastionHostVPC'
    SecurityGroupIngress:
      - IpProtocol: 'tcp'
        SourceSecurityGroupId: !Ref 'BastionHostSG'
        FromPort: 22
        ToPort: 22
    Tags:
      - Key: 'Name'
        Value: 'AppInstance1ASG'
```

![](https://miro.medium.com/v2/resize:fit:700/1*8LoiqcrMSSPn2Ry8dszyLQ.png)

## **Coding App Instance 1B**

Similarly to the previous instance, we now set up `AppInstance1B`, which will be in the `AppPrivateSubnet1B` subnet, **in another Availability Zone**

(**AZ-1b**). In the CloudFormation template, we define the following properties: Availability Zone, Subnet, Instance Type, AMI Image, Security Group, and SSH key.

```
# App Instance 1B
AppInstance1B:
  Type: 'AWS::EC2::Instance'
  Properties:
    AvailabilityZone: !Select [1, !GetAZs '']
    SubnetId: !Ref 'AppPrivateSubnet1B'
    InstanceType: 't2.micro'
    ImageId: 'ami-0910ce22fbfba68e1'
    SecurityGroupIds:
      - !Ref 'AppInstance1BSG'
    KeyName: 'BastionHostKeyPair'
    Tags:
      - Key: 'Name'
        Value: 'AppInstance1B'
```

![](https://miro.medium.com/v2/resize:fit:700/1*uih5US6SSxI4UbSUbKXEzA.png)

## **Defining the Security Group for App Instance 1B**

For the `AppInstance1B` instance, we define a Security Group that allows **ICMP traffic** from the `AppInstance1A` instance **located in another Availability Zone** (**AZ-1a**). ICMP (**Internet Control Message Protocol**) is often used for **network diagnostics**, such as the **ping command**, which checks if an instance is reachable. By using `SourceSecurityGroupId`, **we allow ICMP traffic only from instances assigned to a specific SG**, in this case, `AppInstance1ASG`.

```
# App Instance 1B Security Group
AppInstance1BSG:
  Type: 'AWS::EC2::SecurityGroup'
  Properties:
    GroupDescription: 'Allow ICMP from AppInstance1A'
    VpcId: !Ref 'BastionHostVPC'
    SecurityGroupIngress:
      - IpProtocol: 'icmp'
        SourceSecurityGroupId: !Ref 'AppInstance1ASG'
        FromPort: -1
        ToPort: -1
    Tags:
      - Key: 'Name'
        Value: 'AppInstance1BSG'
```

![](https://miro.medium.com/v2/resize:fit:700/1*K99SNHxpvtPy7uroNdn0Yw.png)

# **Deploying VPC Using VS Code**

To deploy a VPC using CloudFormation, I use **VS Code terminal**.

**Creating the stack**:

- In the VS Code terminal, I used the command to create the stack:

```
aws cloudformation create-stack --stack-name BastionHostVPC --template-body file://cloudformation-vpc2.yaml
```

![](https://miro.medium.com/v2/resize:fit:700/1*EzvNRgfTbd-oEq5rrOsS6w.png)

**Command output**:

- After executing the above command, I received c**onfirmation of the stack creation along with the stack ID**:

![](https://miro.medium.com/v2/resize:fit:700/1*a1lDKMGQuNSnKguK3rP4mQ.png)

**Checking the status**:

- To **check the status** of the created stack, I used the command:

```
aws cloudformation describe-stacks --stack-name BastionHostVPC
```

![](https://miro.medium.com/v2/resize:fit:700/1*z1gWvQSPJ1v97_E0e4bZ1Q.png)

- I received information that the **stack is currently being created**:

![](https://miro.medium.com/v2/resize:fit:700/1*WHm7ixcnpRW7aIKiMkJS8w.png)

- After a few minutes, I **rechecked** the stack status using the same command and received information that the **stack was successfully created**:

![](https://miro.medium.com/v2/resize:fit:700/1*STYe58o22oGKJu_12k0v2w.png)

# **Confirmation of Stack Creation in AWS CloudFormation**

After creating the stack using CloudFormation in VS Code, **I logged into the AWS console to confirm that the stack was successfully created**.

In the “Stacks” section, I could see the created stack named `BastionHostVPC` with the status `CREATE_COMPLETE`.

![](https://miro.medium.com/v2/resize:fit:700/1*NYMM9rcqf16VX5iuaR-e8g.png)

I clicked on the `BastionHostVPC` stack to view its details and navigated to the "**Events**" tab.

![](https://miro.medium.com/v2/resize:fit:700/1*wAz75jcoKA4kTVP9JSAr6A.png)

I could see there that **all resources were successfully created**.

# **Validating SSH Connection**

To validate the SSH connection to our Bastion Host, **we navigate to the EC2 panel in the AWS console**.

In the **instances table**, I found our `BastionHostInstance` and checked its **public IP address**.

![](https://miro.medium.com/v2/resize:fit:700/1*of6q5VAmRPtz8M0xNcFzWg.png)

Next, using the terminal, I attempted to connect to the Bastion Host using the following command:

```
ssh -i BastionHostKeyPair.pem ec2-user@3.71.114.139
```

However, I encountered an `Operation timed out` error. This is likely due to a change in my public IP address, **as I had restarted my router in the meantime, causing the IP address to change**.

![](https://miro.medium.com/v2/resize:fit:700/1*6s10qHBA5Hi-EKcoOXS0Vw.png)

## **Debugging Process**

After encountering the SSH connection error, I decided to debug the issue.

I went to [**whatismyipaddress.com**](https://whatismyipaddress.com/) and **checked my current IP address**.

Then, I changed the IP address in the code responsible for the Bastion Host Security Group to reflect the new IP address.

![](https://miro.medium.com/v2/resize:fit:700/1*x0WmGWicdiXzkW046q_0ig.png)

After **saving the modified template**, I used the terminal in VSCode to update the stack with the following command:

```
aws cloudformation update-stack --stack-name BastionHostVPC --template-body file://cloudformation-vpc2.yaml
```

![](https://miro.medium.com/v2/resize:fit:700/1*94jXOx2eXIvtdsMDOPW9xQ.png)

![](https://miro.medium.com/v2/resize:fit:700/1*FVWbTqQtjpW4Vb8Tud41fQ.png)

After a few moments, I checked the stack status using the `describe-stacks` command and saw that the **update was completed successfully**:

![](https://miro.medium.com/v2/resize:fit:700/1*35TyAcwC6HS968ESYt_DOQ.png)

## **Rechecking the SSH Connection**

When I attempted to reconnect to the BastionHost via SSH, **I encountered a permissions error with the key file**.

![](https://miro.medium.com/v2/resize:fit:700/1*F9tuHbCWDyMby-GHc5zczw.png)

This error occurred because the **file permissions were too open**, making the **private key file accessible to other users**.

To resolve this issue, I changed the file permissions using the command:

```
chmod 400 BastionHostKeyPair.pem
```

![](https://miro.medium.com/v2/resize:fit:700/1*-71q_FvzVJeLWV3y_ld4dw.png)

## **Copying Over Key**

Next, I **transferred the private key to the BastionHost to connect to the next instance from there**. I used the following SCP command to copy the key from my local machine to the BastionHost:

```
scp -i BastionHostKeyPair.pem BastionHostKeyPair.pem ec2-user@3.71.114.139:~/
```

`~/` denotes the home directory of the user on the remote server. In this case, `scp` (**secure copy**) transfers the SSH key file from the local machine to the **home directory** of the `ec2-user` on the BastionHost.

![](https://miro.medium.com/v2/resize:fit:700/1*frsoqTeGlmc8UDT8PWKDqA.png)

## **Connecting to Instance in AppPrivateSubnet1A**

After connecting to BastionHost, I could now connect to the instance in AppPrivateSubnet1A. I used the **private IP address of this instance**, which I found in the AWS console.

![](https://miro.medium.com/v2/resize:fit:700/1*_2HpQ9hddd5rAhtl1pmQSQ.png)

![](https://miro.medium.com/v2/resize:fit:700/1*4WhcsJRsk_e5ySnAChJG_w.png)

# **Checking ICMP Connection to Instance in AppPrivateSubnet1B**

Next, I **checked the ICMP connection** (**ping**) to the instance located in **AppPrivateSubnet1B**.

Already being connected to the instance in AppPrivateSubnet1A, I used the private IP address of the AppPrivateSubnet1B instance, which I obtained from the AWS console.

![](https://miro.medium.com/v2/resize:fit:700/1*bP9ZUQonrGGAuFODqdvstQ.png)

I used the following command to perform the ping:

```
ping 183.14.5.19 -c 4
```

![](https://miro.medium.com/v2/resize:fit:700/1*ZPhyYcqeONfN3mtikKZDLw.png)

# **Summary**

EC2 instances **located in different availability zones** (AZs) within my VPC network **can communicate with each other**. I successfully confirmed the ICMP connection between the instances in AppPrivateSubnet1A and AppPrivateSubnet1B, **proving that the VPC configuration, including subnets, route tables, and security groups, was done correctly**.
