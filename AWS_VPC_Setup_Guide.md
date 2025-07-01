
# AWS VPC Setup with Public & Private Subnet, IGW, NAT Gateway, and EC2 Instances

This guide explains how to set up a VPC with:
- One Public Subnet
- One Private Subnet
- Internet Gateway (IGW)
- NAT Gateway
- EC2 instances in both subnets
- SSH access to private EC2 via bastion (public EC2)

---

![image](https://github.com/user-attachments/assets/4bda45c4-e7db-45ac-8d40-73130c086643)

---

## Step 1: Create a VPC
1. Open **VPC** in AWS Console.
2. Click **Create VPC** → Choose **VPC and more**.
3. Name: `my-vpc`, CIDR block: `10.0.0.0/16`
4. Leave rest as default → Click **Create**.

---

## Step 2: Create Subnets

### Public Subnet
- Name: `PublicSubnet`
- Availability Zone: `ap-south-1a`
- CIDR Block: `10.0.1.0/24`

### Private Subnet
- Name: `PrivateSubnet`
- Availability Zone: `ap-south-1a`
- CIDR Block: `10.0.2.0/24`

> ⚠️ Ensure no CIDR overlap. If `10.0.2.0/24` is taken, use `10.0.3.0/24`.

---

## Step 3: Create and Attach Internet Gateway
1. Go to **Internet Gateways** → Create IGW → Name: `MyIGW`
2. Attach to `my-vpc`.

---

## Step 4: Create Route Table for Public Subnet
1. Go to **Route Tables** → Create → Name: `PublicRouteTable`
2. Add Route: `0.0.0.0/0` → Target: `MyIGW`
3. Associate with `PublicSubnet`.

---

## Step 5: Create NAT Gateway
1. Allocate Elastic IP from VPC Dashboard.
2. Go to **NAT Gateways** → Create NAT → Subnet: `PublicSubnet`.
3. Use allocated Elastic IP.

---

## Step 6: Private Route Table
1. Create Route Table: `PrivateRouteTable`
2. Add Route: `0.0.0.0/0` → Target: NAT Gateway
3. Associate with `PrivateSubnet`.

---

## Step 7: Launch EC2 Instances

### Public EC2
- Subnet: `PublicSubnet`
- Auto-assign Public IP: **Enabled**
- Security Group: Allow **SSH (22)** from your IP, **ICMP** from `0.0.0.0/0`

### Private EC2
- Subnet: `PrivateSubnet`
- Auto-assign Public IP: **Disabled**
- Security Group: Allow **SSH (22)** from **Public EC2’s Private IP**

---

## Step 8: SSH into EC2s

### SSH to Public EC2 (from your PC)
```bash
ssh -i "your-key.pem" ec2-user@<Public-EC2-IP>
```

### SSH to Private EC2 (via Public EC2)
1. From Public EC2 terminal:
```bash
chmod 400 hema.pem
ssh -i "your-key.pem" ec2-user@<Private-EC2-IP>
```

---

## Step 9: Ping Test
On both EC2s, run:
```bash
ping google.com
```

✅ If both succeed:
- IGW and NAT Gateway are correctly set up.
- Subnets and routing are correct.
- You can access and test internet from both EC2s.

---

## Troubleshooting Tips
- Security Groups must allow SSH and ICMP.
- Route tables must have correct routes.
- Use `chmod 400` on your `.pem` key to avoid SSH errors.
