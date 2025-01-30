# Step 1: Launch an EC2 instance
# Go to AWS Management Console, navigate to EC2, and launch an instance with Ubuntu (Amazon Machine Image) as the OS.
# Make sure to select a security group that allows HTTP (port 80), SSH (port 22), and custom TCP for MongoDB (port 27017), and the port for your API (port 3000).

# Step 2: Generate SSH Key Pair (if you don't have one)
# In the AWS Management Console, create an SSH key pair for secure login to your EC2 instance.

# Step 3: Connect to EC2 instance via SSH
# Open terminal and use the following command to connect to the EC2 instance:
ssh -i "path-to-your-key.pem" ubuntu@<ec2-public-ip>

# Step 4: Update system packages and install MongoDB, Node.js, and npm
sudo apt update && sudo apt install -y mongodb nodejs npm

# Step 5: Start MongoDB service
sudo systemctl start mongod

# Step 6: Verify MongoDB service status
sudo systemctl status mongod

# Step 7: Allow MongoDB to accept external connections by editing the config file
sudo nano /etc/mongod.conf
# Change the bindIp value to 0.0.0.0 under the net section to allow all IP addresses:
# net:
#   bindIp: 0.0.0.0

# Step 8: Restart MongoDB to apply the changes
sudo systemctl restart mongod

# Step 9: Set up a MongoDB user with admin privileges
mongo <<EOF
use admin
db.createUser({
  user: "admin",
  pwd: "secure_password",
  roles: [{ role: "userAdminAnyDatabase", db: "admin" }]
});
EOF

# Step 10: Enable authentication by editing the MongoDB config file again
sudo nano /etc/mongod.conf
# Uncomment or add the following line under the security section to enable authorization:
# security:
#   authorization: enabled

# Step 11: Restart MongoDB to apply authentication settings
sudo systemctl restart mongod

# Step 12: Open port 27017 in AWS Security Group to allow remote connections
# (This step should be done in the AWS Management Console under Security Groups)

# Step 13: Check permissions for MongoDB data directory
sudo chown -R mongodb:mongodb /var/lib/mongodb
sudo chmod -R 700 /var/lib/mongodb

# Step 14: Restart MongoDB after fixing permissions
sudo systemctl restart mongod

# Step 15: Install necessary Node.js packages (Express, Mongoose, CORS)
sudo npm install express mongoose cors dotenv

# Step 16: Create a directory for your Node.js API project and navigate to it
mkdir node-api
cd node-api

# Step 17: Create the .env file with your MongoDB URI
echo "MONGO_URI=mongodb://admin:secure_password@<ec2-public-ip>:27017" > .env

# Step 18: Create the MongoDB database and collection
mongo <<EOF
use StockData
db.createCollection("Stock_data")
EOF

# Step 19: Create the index.js file for the Node.js API
cat <<EOF > index.js
require("dotenv").config();
const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");

const app = express();
app.use(express.json());
app.use(cors());

// MongoDB Connection
mongoose
  .connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log("MongoDB Connected"))
  .catch((err) => console.error("MongoDB Connection Error:", err));

// Define Schema for Stock Data
const StockDataSchema = new mongoose.Schema({
  symbol: String,
  name: String,
  price: Number,
  volume: Number,
  timestamp: { type: Date, default: Date.now }
});

const StockData = mongoose.model("Stock_data", StockDataSchema);

// API to Fetch Stock Data
app.get("/data", async (req, res) => {
  try {
    const data = await StockData.find();
    res.json(data);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// API to Insert Stock Data
app.post("/data", async (req, res) => {
  try {
    const { symbol, name, price, volume } = req.body;
    const newStockData = new StockData({ symbol, name, price, volume });
    await newStockData.save();
    res.status(201).json(newStockData);
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
});

// Start Server
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(\`Server running on port \${PORT}\`);
});
EOF

# Step 20: Start the Node.js server
node index.js

# Step 21: Open port 3000 in AWS Security Group to allow access to the API
# (This step should be done in the AWS Management Console under Security Groups)

# Step 22: Test API using a web browser or Postman
# Test URL: http://<ec2-public-ip>:3000/data

# Step 23: Check the server logs to ensure everything is working
sudo tail -f /var/log/syslog

# Step 24: Create an Elastic IP
# Go to AWS EC2 Dashboard -> Elastic IPs -> Allocate new Elastic IP.
# Once created, associate this Elastic IP with your EC2 instance.
# This will give you a static IP that can be used to access your API consistently.

# Step 25: Update the MongoDB URI with the new Elastic IP
echo "MONGO_URI=mongodb://admin:secure_password@<elastic-ip>:27017" > .env
# Now, replace <elastic-ip> with the new Elastic IP you just created.

# Step 26: Re-deploy Node.js API with the new MongoDB URI
# Restart your Node.js server:
pm2 restart index.js
# If you are not using PM2, you can manually stop and start the server again:
# node index.js

# Now, your MongoDB and Node.js API are accessible via the Elastic IP!
