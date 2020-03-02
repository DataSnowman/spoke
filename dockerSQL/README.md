# SQL Server on Docker on Ubuntu VM

Based on [Quickstart: Run SQL Server container images with Docker](https://docs.microsoft.com/en-us/sql/linux/quickstart-install-connect-docker?view=sql-server-ver15&pivots=cs1-bash)

If you want to use PowerShell or Cmd instead the commands can be found in the Quickstart.

After you have completed the Spoke deployment of an Ubuntu VM with Docker and Azure Data Lake Storage you can setup SQL Server 2019 on Docker on the deployed Ubuntu VM.

## Installing SQL Server 2019 on Docker

1. Connect to the VM

    `ssh user@<IPaddress>`

2. In a recent update I used Docker Compose to pull the official [Microsoft SQL Server](https://hub.docker.com/_/microsoft-mssql-server) image as part of the deployment.  You should be able to skip this pull command to pull SQL 2019 container image because it should already be present on the VM.

   ```
   sudo docker pull mcr.microsoft.com/mssql/server:2019-latest
   ```

3. Run the container image

    ```
    sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>" \
        -p 1433:1433 --name sql1 \
        -d mcr.microsoft.com/mssql/server:2019-latest
    ```


4. View Docker Container

    ```
    sudo docker ps -a
    ```

5. Change SA Password 

    `Note: Remember to do this part - I spent an entire Sunday troubleshooting why I could not connect in SSMS, Azure Data Studio, and VScode.  After changing the SA password all was well.`

    ```
    sudo docker exec -it sql1 /opt/mssql-tools/bin/sqlcmd \
        -S localhost -U SA -P "<YourStrong@Passw0rd>" \
        -Q 'ALTER LOGIN SA WITH PASSWORD="<YourNewStrong@Passw0rd>"'
    ```
## Connecting to SQL Server 2019 on Docker

6. Connect to SQL Server

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

7. Create a new database

    `CREATE DATABASE TestDB`

    `SELECT Name from sys.Databases`

    `GO`

8. Insert Data

    `USE TestDB`

    `CREATE TABLE Inventory (id INT, name NVARCHAR(50), quantity INT)`

    `INSERT INTO Inventory VALUES (1, 'banana', 150); INSERT INTO Inventory VALUES (2, 'orange', 154);`

    `GO`

9. Select Data

    `SELECT * FROM Inventory WHERE quantity > 152;`

    `GO`

10. Exit sqlcmd

    1> `QUIT`

    mssql@9625004ea8ab:/$ `exit`

11. Add Inbound security rule to Network security group

    Source: `Any`
    Source port ranges: `*`
    Destination: `Any`
    Destination port ranges: `1433` 

    (or other if publishing different port like 1401 below for the -p 1401:1433 -p [port number on docker host]:[port number on container])

    sudo docker run -e "ACCEPT_EULA=Y" -e "SA_PASSWORD=<YourStrong@Passw0rd>" \
        -p `1401:1433` --name sql1 \
        -d mcr.microsoft.com/mssql/server:2019-latest

    Protocal: `TCP`
    Action: `Allow`
    Priority: `300` 

    (or 301 if you have multiple instances on the same container)

    Name: `allow-1433-sql`

    ![addInbound](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/addInboundSecurityRule.png)

12. Connect for SSMS or Azure Data Studio or VScode

    `<IPaddress>,1433`

    Connect using SQL Server Management Studio (SSMS)

    ![ssms](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/ssms.png)


    Connect using Azure Data Studio (ADS)

    ![ads](https://raw.githubusercontent.com/DataSnowman/spoke/master/images/ads.png)

    Connect using Visual Studio Code see [Use Visual Studio Code to create and run Transact-SQL scripts](https://docs.microsoft.com/en-us/sql/visual-studio-code/sql-server-develop-use-vscode?view=sql-server-ver15)


13. If shutting down the VM stop the SQL container

    ```
    sudo docker stop sql1
    ```


14. If restarting the VM start the SQL container

    Remember to login with the new IPaddress

    `ssh user@<NewIPaddress>`

    ```
    sudo docker start sql1
    ```

    ```
    sudo docker ps -a
    ```

15. Change Connection for SSMS or Azure Data Studio or VScode to use the New IPaddress

    `<NewIPaddress>,1433`