# SQL Server on Docker on Ubuntu VM

### `Note: If you would like a simpler version of this that does not persistant SQL Server files on the VM disks (files are just in the container and will be lost if you remove the container)` Please click [HERE](https://github.com/DataSnowman/spoke/blob/master/dockerSQL/simpler.md) for a simpler faster to configure version.  

### Continue with this Readme if you are looking for a more production scenario.

Based on [Quickstart: Run SQL Server container images with Docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash)

If you want to use PowerShell or Cmd instead the commands can be found in the Quickstart.

After you have completed the ARM deployment of an Ubuntu VM with Docker and Azure Data Lake Storage you can setup SQL Server 2019 on Docker on the deployed Ubuntu VM.

## Installing SQL Server 2019 on Docker

1. Connect to the VM

    `ssh user@<IPaddress>`

2. In a recent update I used Docker Compose to pull the official [Microsoft SQL Server](https://hub.docker.com/_/microsoft-mssql-server) image as part of the deployment.  You should be able to skip this pull command to pull SQL 2019 container image because it should already be present on the VM.

   ```
   sudo docker pull mcr.microsoft.com/mssql/server:2019-latest
   ```

3. View available disk and partion, format, and mount the data disk

    ```
    sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
    ```
    The output should look something like this:

    ```
    NAME    FSTYPE  SIZE MOUNTPOINT LABEL
    sdb              32G
    └─sdb1  ext4     32G /mnt
    sr0             628K
    sdc             128G
    sda              30G
    ├─sda14           4M
    ├─sda15 vfat    106M /boot/efi  UEFI
    └─sda1  ext4   29.9G /          cloudimg-rootfs
    ```

    **Partion**

    ```
    sudo parted /dev/sdc
    ```

    **Output**

    ```
    GNU Parted 3.2
    Using /dev/sdc
    Welcome to GNU Parted! Type 'help' to view a list of commands.
    (parted)
    ```

    Within the parted, type the following to have gpt partition, gpt can allow partition larger than 2GB:

    ```
    mklabel gpt
    ```

    Set the size for partition, I here partition from 0GB to 4GB:

    ```
    mkpart primary 0GB 128GB
    ```

    Then quit the parted:

    ```
    quit
    ```

    **Format**

    Format the newly partitioned hard disk:

    ```
    sudo mkfs.ext4 /dev/sdc
    ```

    Enter `y`

    **Output**

    ```
    mke2fs 1.42.13 (17-May-2015)
    Found a gpt partition table in /dev/sdc
    Proceed anyway? (y,n) y
    Discarding device blocks: done
    Creating filesystem with 33554432 4k blocks and 8388608 inodes
    Filesystem UUID: 6c869d93-ecc9-4c00-ad48-b8396e91a79b
    Superblock backups stored on blocks:
        32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
        4096000, 7962624, 11239424, 20480000, 23887872
    
    Allocating group tables: done
    Writing inode tables: done
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done
    ```

    ***Mount** (including auto mount after reboot)

    Usually drive is mounted in /mnt/. Create a new directory in /mnt/ first


    ```
    sudo mkdir /mnt/sdc
    ```

    Then we can mount it by:

    ```
    sudo mount /dev/sdc /mnt/sdc
    ```

    View available disk again

    ```
    sudo lsblk -o NAME,FSTYPE,SIZE,MOUNTPOINT,LABEL
    ```

    ```
    NAME    FSTYPE  SIZE MOUNTPOINT LABEL
    sdb              32G
    └─sdb1  ext4     32G /mnt
    sr0             628K
    sdc     ext4    128G /mnt/sdc
    sda              30G
    ├─sda14           4M
    ├─sda15 vfat    106M /boot/efi  UEFI
    └─sda1  ext4   29.9G /          cloudimg-rootfs
    ```

    But we need to mount it for every time we reboot. To mount it automatically after each reboot, I use nano to modify the file /etc/fstab:

    ```
    sudo nano /etc/fstab
    ```

    Enter following at the end of file:

    ```
    /dev/sdc     /mnt/sdc      ext4        defaults      0       0
    ```

    Press `Ctrl and x`, Then press `Y`, Then press `Enter` when you see `File Name to Write: /etc/fstab`


    ![addfstab](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/addfstab.png)

    The first item is the path for the hard drive. The second one is the destination for the mounted drive, where we want to mount. The third one is the format type. The forth to sixth one I just kept as defaults, 0 and 0.

    At the bottom of the window, there is a list of the most basic command shortcuts to use with the nano editor.

    All commands are prefixed with either ^ or M character. The caret symbol (^) represents the Ctrl key. For example, the ^J commands mean to press the Ctrl and J keys at the same time.


4. Create `database` directory for the persistant SQL Server files

    Make a `database` directory in /mnt/sdc of data disk

    ```
    sudo mkdir /mnt/sdc/database
    ```

5. Run the container image

    ```
    sudo docker run -e 'ACCEPT_EULA=Y' -e 'MSSQL_SA_PASSWORD=<YourStrong@Passw0rd>' \
   -u 0:0 -p 1401:1433 --name sql1 \
   -v /mnt/sdc/database/data:/var/opt/mssql/data \
   -v /mnt/sdc/database/log:/var/opt/mssql/log \
   -v /mnt/sdc/database/secrets:/var/opt/mssql/secrets \
   -d mcr.microsoft.com/mssql/server:2019-latest
    ```

6. View Docker Container

    ```
    sudo docker ps -a
    ```

7. Change SA Password 

    `Note: Remember to do this part - I spent an entire Sunday troubleshooting why I could not connect in SSMS, Azure Data Studio, and VScode.  After changing the SA password all was well.`

    ```
    sudo docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd \
        -S localhost -U SA -P "<YourStrong@Passw0rd>" \
        -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong@Passw0rd>"'
    ```
## Connecting to SQL Server 2019 on Docker

8. Connect to SQL Server

    ```
    sudo docker exec -it sql1 "bash"
    ```

    ```
    /opt/mssql-tools/bin/sqlcmd -S localhost -U SA -P "<YourNewStrong@Passw0rd>"
    ```

    `Note: If you get the following error:`

    `Sqlcmd: Error: Microsoft ODBC Driver 17 for SQL Server : Login failed for user 'SA'..`

    Then try this command with single quotes around 'SA' and '<YourNewStrong@Passw0rd>'

    ```
    /opt/mssql-tools/bin/sqlcmd -S localhost -U 'SA' -P '<YourNewStrong@Passw0rd>'
    ```

9. Create a new database

    `CREATE DATABASE TestDB`

    `SELECT Name from sys.Databases`

    `GO`

10. Insert Data

    `USE TestDB`

    `CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)`

    `INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);`

    `GO`

11. Select Data

    `SELECT * FROM Inventory WHERE quantity > 152;`

    `GO`

12. Exit sqlcmd

    1> `QUIT`

    mssql@9625004ea8ab:/$ `exit`

13. Add Inbound security rule to Network security group

    Source: `Any`

    Source port ranges: `*`

    Destination: `Any`

    Destination port ranges: `1401` 

    (or other if publishing different port like 1402 below for the -p 1402:1433 -p [port number on docker host]:[port number on container])

    sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>" \
        -p `1402:1433` --name sql2 \
        -d mcr.microsoft.com/mssql/server:2019-latest

    Protocal: `TCP`

    Action: `Allow`

    Priority: `300` 

    (or 301 if you have multiple instances on the same container)

    Name: `allow-1401-sql`

    ![addInbound](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/addInboundSecurityRule.png)

14. Connect for SSMS or Azure Data Studio or VScode

    `<IPaddress>,1401`

    Connect using SQL Server Management Studio (SSMS)

    ![ssms](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/ssms.png)


    Connect using Azure Data Studio (ADS)

    ![ads](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/ads.png)

    Connect using Visual Studio Code see [Use Visual Studio Code to create and run Transact-SQL scripts](https://docs.microsoft.com/en-us/sql/visual-studio-code/sql-server-develop-use-vscode?view=sql-server-ver15)


15. If shutting down the VM stop the SQL container

    ```
    sudo docker stop sql1
    ```


16. If restarting the VM start the SQL container

    Remember to login with the new IPaddress

    `ssh user@<NewIPaddress>`

    ```
    sudo docker start sql1
    ```

    ```
    sudo docker ps -a
    ```

17. Change Connection for SSMS or Azure Data Studio or VScode to use the New IPaddress

    `<NewIPaddress>,1401`