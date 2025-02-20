# Redis Master-Slave Setup on Windows

This guide will walk you through the process of setting up a Redis Master-Slave replication setup on Windows. The setup includes configuring the master and slave nodes, installing Redis as a service, and verifying the replication.

## Prerequisites

- A Windows machine with administrative privileges.
- Redis for Windows: [Download Redis](https://github.com/tporadowski/redis/releases) (Zip file).
- Two machines (Master and Slave) with different IPs.

## Installation

### 1. Download and Extract Redis

- Download the Redis Zip file from the link: [Redis Release](https://github.com/tporadowski/redis/releases).
- Extract the contents of the zip file to a folder on your C drive.

## Configuration

### 1. Configure the Master Node ([Master Server IP])

- Open `redis.windows-service.conf` and modify it to ensure the server is set as the master.
- Allow remote connections by changing the bind directive to:

    ```bash
    bind [Master server IP]
    ```

- Ensure no `replicaof` directive is present.
- Save the file and exit.

### 2. Configure the Slave Node ([Slave Server IP])

- Open `redis.windows.conf` and modify it to set it as the slave.
- Allow remote connections by changing the bind directive:

    ```bash
    bind [Slave server IP]
    ```

- Set up replication by uncommenting and setting the `replicaof` directive to the master:

    ```bash
    replicaof [Master server IP] 6379
    ```

- Save the file and exit.

### 3. Install Redis as a Windows Service

#### On the Master ([Master Server IP])

- Run the following command in Command Prompt (as Administrator):

    ```bash
    redis-server --service-install redis.windows-service.conf --loglevel notice
    ```

- Start the Redis service:

    ```bash
    redis-server --service-start
    ```

#### On the Slave ([Slave Server IP])

- Run the following command in Command Prompt (as Administrator):

    ```bash
    redis-server --service-install redis.windows.conf --loglevel notice
    ```

- Start the Redis service:

    ```bash
    redis-server --service-start
    ```

### 4. Verify Replication

#### On the Slave ([Slave Server IP])

- Run the following command to check the replication status:

    ```bash
    redis-cli info replication
    ```

- It should show `role:slave` and be connected to the master.

#### On the Master ([Master Server IP])

- Run the following command to check the replication status:

    ```bash
    redis-cli info replication
    ```

- It should show `role:master` and a connected slave.

### 5. Verify if Redis is Running

- Run the following command to check if Redis is running:

    ```bash
    tasklist | findstr redis
    ```

- If Redis is running, you should see `redis-server.exe` listed.

### 6. Start Redis Manually

- If Redis is not running, try starting it manually:

    ```bash
    redis-server redis.windows-service.conf
    ```

- If it fails to start, check for error messages.

### 7. Check Redis Configuration File (`redis.windows-service.conf`)

- Ensure the following:
  - Redis is listening on the correct IP:

      ```bash
      bind [Master server IP]
      ```

  - If `bind 127.0.0.1` is present, comment it out by adding `#` in front.
  - Protected mode is disabled:

      ```bash
      protected-mode no
      ```

  - Redis is set to run on the correct port (default `6379`):

      ```bash
      port 6379
      ```

- Save the file and restart Redis.

### 8. Restart the Redis Service

- Try restarting the Redis service:

    ```bash
    redis-server --service-stop
    redis-server --service-start
    ```

- If it still fails, uninstall and reinstall the service:

    ```bash
    redis-server --service-uninstall
    redis-server --service-install redis.windows-service.conf
    redis-server --service-start
    ```

### 9. Test Redis Again

- Run the following command to connect to the master node:

    ```bash
    redis-cli -h [Master server IP] -p 6379
    ```

- Check the replication status:

    ```bash
    info replication
    ```

### 10. Verify the Replication

- Run the following command on both the master and slave nodes:

    ```bash
    redis-cli info replication
    ```

- **Expected Output**:

  On [Master server IP] (Master):

    ```bash
    role:master
    connected_slaves:1
    slave0:ip=[Slave server IP],port=6379,state=online
    ```

  On [Slave server IP] (Slave):

    ```bash
    role:slave
    master_host:[Master server IP]
    master_port:6379
    master_link_status:up
    ```

### 11. Check Firewall Rules

- Ensure that the Windows Firewall allows Redis traffic:
  - Open Windows Defender Firewall with Advanced Security.
  - Go to **Inbound Rules → New Rule**.
  - Select **Port**, choose **TCP**, and enter **6379**.
  - Allow the connection and apply it.
- Try connecting again.

---

This setup should now have Redis Master-Slave replication working on your Windows machines.
