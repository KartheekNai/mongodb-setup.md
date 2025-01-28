# MongoDB Setup and Connection Guide

This guide documents the process for setting up and connecting to MongoDB on an AWS EC2 instance, along with the commands used during the setup.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- MongoDB installed on the server.
- MongoDB Compass for GUI-based connection (optional).

## Steps

### 1. Install MongoDB
```bash
sudo apt update
sudo apt install -y mongodb
```

### 2. Start MongoDB Service
```bash
sudo systemctl start mongod
```

### 3. Verify MongoDB Service Status
```bash
sudo systemctl status mongod
```

### 4. Configure MongoDB to Allow External Connections
Edit the MongoDB configuration file:
```bash
sudo nano /etc/mongod.conf
```
Update the `bindIp` value to `0.0.0.0`:
```yaml
net:
  bindIp: 0.0.0.0
```
Save the file and restart the MongoDB service:
```bash
sudo systemctl restart mongod
```

### 5. Allow Inbound Traffic on Port 27017 in AWS Security Groups
- Open the AWS Management Console.
- Navigate to the EC2 instance's Security Group.
- Edit inbound rules to allow traffic for port `27017` from your desired IP range.

### 6. Connect MongoDB to MongoDB Compass
- Open MongoDB Compass.
- Use the connection string:
  ```
  mongodb://<ec2-public-ip>:27017
  ```
- Replace `<ec2-public-ip>` with your EC2 instance's public IP address.

### 7. Common MongoDB Commands
- **Show databases:**
  ```bash
  show dbs
  ```
- **Use a database:**
  ```bash
  use <database_name>
  ```
- **List collections:**
  ```bash
  show collections
  ```
- **Find documents in a collection:**
  ```bash
  db.<collection_name>.find()
  ```

### 8. Troubleshooting
- If MongoDB fails to start, check the logs:
  ```bash
  sudo tail -f /var/log/mongodb/mongod.log
  ```
- Ensure port `27017` is open in AWS security groups.

---

## Commands Used During Setup
```bash
# Update system packages
sudo apt update

# Install MongoDB
sudo apt install -y mongodb

# Start MongoDB service
sudo systemctl start mongod

# Check MongoDB service status
sudo systemctl status mongod

# Edit MongoDB configuration
sudo nano /etc/mongod.conf

# Restart MongoDB service
sudo systemctl restart mongod

# Check MongoDB logs for troubleshooting
sudo tail -f /var/log/mongodb/mongod.log
```

---

## Additional Notes
- Always restrict inbound rules in AWS Security Groups to trusted IP ranges for security.
- Use a strong password and enable authentication if exposing MongoDB to external networks.
