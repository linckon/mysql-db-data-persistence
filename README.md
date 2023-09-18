
# MySQL data persistence hosted on two seperate EC2 intances using Elastic Block Storage (EBS)

This guide will help you set up a MySQL container using Docker Compose on an EC2 instance with an EBS volume and db data persistence.

## Prerequisites

1. Sign in to the AWS Management Console and open the Amazon EC2 console at https://console.aws.amazon.com/ec2/.
2. Launch two EC2 instances named Instance-1 and Instance-2.
3. In the navigation pane, choose Volumes under Elastic Block Store.
4. Click Create Volume.
   - Choose the type of volume you want to create. General purpose SSD (gp2 or gp3) is a good default choice for many workloads.
   - Set the size of the volume.
   - For Availability Zone, choose the same zone as your EC2 instance. This is important because volumes can only be attached to instances in the same zone.
   - Click Create.
5. After the volume is created, select it, click on the Actions button, and then choose Attach Volume.
6. In the Attach Volume dialog box:
   - For Instance, start typing the name or ID of the instance, and then select Instance-1 from the list.
   - For Device, enter a device name. The name will depend on your instance type and the OS. For example, /dev/sdf (for instances launched using a classic block device mapping) or /dev/nvme1n1 (for Nitro-based instances).
   - Click Attach.

## Instructions

### Setting Up on the First EC2 Instance

- SSH into the first EC2 instance and run below command:
   ```
   sudo apt update
   sudo apt install docker.io docker-compose -y
   ```
- Clone docker compose file and create a directory `data`
   ```
   git clone https://github.com/linckon/mysql-db-data-persistence.git
   cd mysql-db-data-persistence
   mkdir data
   ```
- Check if the disk is visible using:
   ```
    sudo lsblk
   ```
- Identify the new disk. It might be listed as nvme1n1, xvdf, or something similar, depending on your instance type and how many disks are attached:
  ```
   sudo file -s /dev/xvdf
  ```
- Format the disk (assuming it's a new volume and you're not trying to preserve any data on it). For example, to format it with the ext4 filesystem:
   ```
   sudo mkfs -t xfs /dev/xvdf
   ```
- Mount the EBS volume to a directory, for example `/data`:
   ```
   sudo mount /dev/xvdf /home/ubuntu/mysql-db-data-persistence/data
   ```
- Navigate to the directory containing the `docker-compose.yml` file.
- Run the Docker Compose command:
   ```
   docker-compose up -d
   ```
- Login to mysql inside docker container and create a table called `person` inside database `db` and insert some sample data
   ```
   CREATE TABLE person (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    age INT
   );


   INSERT INTO person (name, age) VALUES
   ('John Doe', 30),
   ('Jane Smith', 25),
   ('Bob Johnson', 40),
   ('Alice Brown', 35),
   ('Eva Davis', 28);



   mysql> select * from person;
   +----+-------------+------+
   | id | name        | age  |
   +----+-------------+------+
   |  1 | John Doe    |   30 |
   |  2 | Jane Smith  |   25 |
   |  3 | Bob Johnson |   40 |
   |  4 | Alice Brown |   35 |
   |  5 | Eva Davis   |   28 |
   +----+-------------+------+
   5 rows in set (0.00 sec)
   ```

### Now mysql data persist into additional disk ,if the instance is corrupted we can simply detach the disk and attach it will another instance.

- Detach the EBS volume from this instance and attach it to the second EC2 instance .

### Setting Up on the Second EC2 Instance

- SSH into the second EC2 instance and run below command:
   ```
   sudo apt update
   sudo apt install docker.io docker-compose -y
   ```
- Clone docker compose file and create a directory `data`
   ```
   git clone https://github.com/linckon/mysql-db-data-persistence.git
   cd mysql-db-data-persistence
   mkdir data
   ```
- Mount the EBS volume:
   ```
   sudo mount /dev/xvdf /home/ubuntu/mysql-db-data-persistence/data
   ```

- Navigate to the directory containing the `docker-compose.yml` file.
- Run the Docker Compose command:
   ```
   docker-compose up -d
   ```
- login to mysql inside docker container and run:
  ```
  select * from person;

   mysql> select * from person;
   +----+-------------+------+
   | id | name        | age  |
   +----+-------------+------+
   |  1 | John Doe    |   30 |
   |  2 | Jane Smith  |   25 |
   |  3 | Bob Johnson |   40 |
   |  4 | Alice Brown |   35 |
   |  5 | Eva Davis   |   28 |
   +----+-------------+------+
   5 rows in set (0.00 sec)
  ```
### See all data should remain same as Instance-1. For detail demostration visit below link:
https://iamlinckon.hashnode.dev/google-cloud-vpc-network-peering


