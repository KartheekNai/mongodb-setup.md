## Prerequisites
- AWS EC2 instance running Ubuntu.
- SSH access to the EC2 instance.
- MongoDB Compass installed locally (optional for GUI-based access).

---

## Steps

### Step 1: Install MongoDB
bash
sudo apt update
sudo apt install -y mongodb


### Step 2: Start MongoDB Service
bash
sudo systemctl start mongod


### Step 3: Verify MongoDB Service Status
bash
sudo systemctl status mongod

Ensure the service is active. If it is not, check the logs for errors:
bash
sudo tail -f /var/log/mongodb/mongod.log


### Step 4: Configure MongoDB to Allow External Connections
Edit the MongoDB configuration file:
bash
sudo nano /etc/mongod.conf

Update the bindIp value to 0.0.0.0 to allow connections from any IP:
yaml
net:
  bindIp: 0.0.0.0


Save the changes and restart MongoDB:
bash
sudo systemctl restart mongod


### Step 5: Set Up a MongoDB User with Authentication
1. Connect to MongoDB shell:
   
bash
   mongo

2. Create an admin user:
   
javascript
   use admin
   db.createUser({
     user: "admin",
     pwd: "secure_password",
     roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
   });

3. Enable authentication by editing the configuration file:
   
bash
   sudo nano /etc/mongod.conf

   Uncomment or add the following line under security:
   
yaml
   security:
     authorization: enabled

4. Restart the MongoDB service:
   
bash
   sudo systemctl restart mongod


### Step 6: Allow Inbound Traffic on Port 27017 in AWS Security Groups
- Navigate to your EC2 instance's Security Group in the AWS Management Console.
- Edit the inbound rules to allow traffic on port 27017 for your IP or range of trusted IPs.

### Step 7: Verify Permissions
Ensure the data directory has the correct ownership and permissions:
bash
sudo chown -R mongodb:mongodb /var/lib/mongodb
sudo chmod -R 700 /var/lib/mongodb

If permissions are incorrect, restart MongoDB:
bash
sudo systemctl restart mongod


### Step 8: Connect Using MongoDB Compass
- Open MongoDB Compass.
- Use the connection string:
  
mongodb://admin:secure_password@<ec2-public-ip>:27017

- Replace <ec2-public-ip> with your EC2 instance's public IP.

### Step 9: Remove MongoDB (if required)
To remove MongoDB completely, execute:
bash
sudo apt purge -y mongodb
sudo rm -rf /var/lib/mongodb
sudo rm -rf /etc/mongod.conf


---

## Troubleshooting

### Problem 1: MongoDB Service Fails to Start
- Check logs:
  
bash
  sudo tail -f /var/log/mongodb/mongod.log

- Verify permissions of the data directory.

### Problem 2: Authentication Fails
- Ensure the correct username and password are used.
- Verify the authorization setting in mongod.conf is enabled.

### Problem 3: Connection Refused in Compass
- Verify port 27017 is open in AWS Security Groups.
- Confirm the bindIp is set to 0.0.0.0 in mongod.conf.

---

## Commands Used
bash
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

# Set permissions for the data directory
sudo chown -R mongodb:mongodb /var/lib/mongodb
sudo chmod -R 700 /var/lib/mongodb

# Remove MongoDB completely
sudo apt purge -y mongodb
sudo rm -rf /var/lib/mongodb
sudo rm -rf /etc/mongod.conf


---

## Notes
- Always restrict access to trusted IPs in AWS Security Groups.
- Use strong passwords for authentication.
- Regularly monitor logs for potential issues.

### Problem: Unable to Connect to MongoDB Remotely
**Solution:**
1. Verify that the bindIp parameter is set to 0.0.0.0 in /etc/mongod.conf.
2. Confirm that port 27017 is open in your AWS Security Group.
3. Check network connectivity using ping or telnet from the client machine.

### Problem: Authentication Failure
**Solution:** Ensure that authentication is enabled and the credentials used match those configured in MongoDB.

## Additional Notes
- Restrict access to MongoDB by allowing only trusted IP ranges in the AWS Security Group.
- Regularly update MongoDB to the latest stable version for security and performance improvements.
- Use a strong password for MongoDB users and avoid exposing MongoDB to the public internet without proper security measures.
