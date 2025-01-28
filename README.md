# MongoDB Setup and Connection Guide

This guide outlines the process of installing, configuring, and connecting MongoDB on an AWS EC2 instance, including troubleshooting steps and solutions for issues encountered during the setup.

## Prerequisites
- AWS EC2 instance running Ubuntu.
- MongoDB Compass installed on your local machine (optional).
- Administrative access to the EC2 instance.

## Steps to Install and Configure MongoDB

### Step 1: Update System Packages
Ensure your system is up-to-date:
```bash
sudo apt update
```

### Step 2: Install MongoDB
Install MongoDB using the following command:
```bash
sudo apt install -y mongodb
```

### Step 3: Start MongoDB Service
Start the MongoDB service:
```bash
sudo systemctl start mongod
```

### Step 4: Verify MongoDB Service Status
Check that MongoDB is running:
```bash
sudo systemctl status mongod
```

### Step 5: Modify MongoDB Configuration
To allow external connections, edit the configuration file:
```bash
sudo nano /etc/mongod.conf
```
Update the `bindIp` parameter under the `net` section to `0.0.0.0`:
```yaml
net:
  bindIp: 0.0.0.0
```
Save the file and restart the MongoDB service:
```bash
sudo systemctl restart mongod
```

### Step 6: Adjust AWS Security Group Rules
- Log in to the AWS Management Console.
- Navigate to the Security Group associated with your EC2 instance.
- Add an inbound rule to allow traffic on port `27017` from your IP address or a trusted IP range.

### Step 7: Create a MongoDB User for Authentication
Access the MongoDB shell:
```bash
mongo
```
Switch to the `admin` database:
```bash
use admin
```
Create an administrative user:
```javascript
db.createUser({
  user: "admin",
  pwd: "securepassword",
  roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
})
```
Enable authentication by editing the MongoDB configuration file:
```bash
sudo nano /etc/mongod.conf
```
Uncomment or add the following under the `security` section:
```yaml
security:
  authorization: enabled
```
Restart MongoDB for changes to take effect:
```bash
sudo systemctl restart mongod
```

### Step 8: Connect MongoDB to Compass
Open MongoDB Compass and use the connection string:
```text
mongodb://<admin>:<securepassword>@<ec2-public-ip>:27017/?authSource=admin
```
Replace `<admin>` with your username, `<securepassword>` with your password, and `<ec2-public-ip>` with the public IP of your EC2 instance.

## Troubleshooting

### Problem: MongoDB Service Fails to Start
**Solution:** Check the MongoDB logs for errors:
```bash
sudo tail -f /var/log/mongodb/mongod.log
```
Ensure that the configuration file syntax is correct and that necessary permissions are in place.

### Problem: Unable to Connect to MongoDB Remotely
**Solution:**
1. Verify that the `bindIp` parameter is set to `0.0.0.0` in `/etc/mongod.conf`.
2. Confirm that port `27017` is open in your AWS Security Group.
3. Check network connectivity using `ping` or `telnet` from the client machine.

### Problem: Authentication Failure
**Solution:** Ensure that authentication is enabled and the credentials used match those configured in MongoDB.

## Additional Notes
- Restrict access to MongoDB by allowing only trusted IP ranges in the AWS Security Group.
- Regularly update MongoDB to the latest stable version for security and performance improvements.
- Use a strong password for MongoDB users and avoid exposing MongoDB to the public internet without proper security measures.
