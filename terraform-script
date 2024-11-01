#Creating my first VPC

resource "aws_vpc" "MyVPC" {
  cidr_block       = var.cidr
  
  tags = {
    Name = "MyVPC"
  }
}

#Adding three subnets as per detailed below

resource "aws_subnet" "sub1" {
  vpc_id = aws_vpc.MyVPC.id
  cidr_block = "10.0.0.0/24"
  availability_zone = "us-east-1a"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "sub2" {
  vpc_id = aws_vpc.MyVPC.id
  cidr_block = "10.0.1.0/24"
  availability_zone = "us-east-1b"
  map_public_ip_on_launch = true
}

resource "aws_subnet" "sub3" {
  vpc_id = aws_vpc.MyVPC.id
  cidr_block = "10.0.2.0/24"
  availability_zone = "us-east-1c"
  map_public_ip_on_launch = true
}

#Creating a internet gateway for my VPC

resource "aws_internet_gateway" "igw" {
  vpc_id = aws_vpc.MyVPC.id
}

#Indicating my VPC to public access through my internet gateway

resource "aws_route_table" "RT" {
  vpc_id = aws_vpc.MyVPC.id

  route {
    cidr_block = var.default_route
    gateway_id = aws_internet_gateway.igw.id
  }
}

#Associating each subnet to the route table

resource "aws_route_table_association" "rtal" {
  subnet_id = aws_subnet.sub1.id
  route_table_id = aws_route_table.RT.id
}

resource "aws_route_table_association" "rta2" {
  subnet_id = aws_subnet.sub2.id
  route_table_id = aws_route_table.RT.id
}

resource "aws_route_table_association" "rta3" {
  subnet_id = aws_subnet.sub3.id
  route_table_id = aws_route_table.RT.id
}

#Creating the security group to my HTTP server, allowing port 80 and port 22 SSH

resource "aws_security_group" "mysg-demo" {
  name = "mysg-demo"
  description = "Security group created on Terraform - EC2"
  vpc_id = aws_vpc.MyVPC.id

  ingress {
    description = "Allow HTTP"
    from_port = 80
    to_port = 80
    protocol = "tcp"
    cidr_blocks = [var.default_route]
  }
  
  ingress {
    description = "Allow SSH"
    from_port = 22
    to_port = 22
    protocol = "tcp"
    cidr_blocks = [var.default_route]
  }
  
  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = [var.default_route]
    
  }
  
  tags = {
    Name = "Web-sg"
  }

}

#Creating my bucket to store my file "terraformtf.state"

resource "aws_s3_bucket" "awsgabriel-bkt" {
  bucket = "awsgabriel-bkt"
  tags = {
    Name = "My Terraform bucket"
    Date = "30-Oct-24"
  }


}

resource "aws_s3_bucket_ownership_controls" "ownership" {
  bucket = aws_s3_bucket.awsgabriel-bkt.id
  rule {
    object_ownership = "BucketOwnerPreferred"
  }
}

resource "aws_s3_bucket_public_access_block" "public" {
  bucket = aws_s3_bucket.awsgabriel-bkt.id

  block_public_acls       = false
  block_public_policy     = false
  ignore_public_acls      = false
  restrict_public_buckets = false
}

resource "aws_s3_bucket_acl" "acl-demo" {
  depends_on = [
    aws_s3_bucket_ownership_controls.ownership,
    aws_s3_bucket_public_access_block.public,
  ]

  bucket = aws_s3_bucket.awsgabriel-bkt.id
  acl    = "public-read"
}

#Creating my EC2 instances to be deployed in three different AZ's


resource "aws_instance" "HTTPserver1" {
  ami = "ami-06b21ccaeff8cd686"
  instance_type = "t2.micro"
  vpc_security_group_ids = [ aws_security_group.mysg-demo.id ]
  subnet_id = aws_subnet.sub1.id
  user_data = base64encode(file("userdata.sh"))
}

resource "aws_instance" "HTTPserver2" {
  ami = "ami-06b21ccaeff8cd686"
  instance_type = "t2.micro"
  vpc_security_group_ids = [ aws_security_group.mysg-demo.id ]
  subnet_id = aws_subnet.sub2.id
  user_data = base64encode(file("userdata1.sh"))
}

resource "aws_instance" "HTTPserver3" {
  ami = "ami-06b21ccaeff8cd686"
  instance_type = "t2.micro"
  vpc_security_group_ids = [ aws_security_group.mysg-demo.id ]
  subnet_id = aws_subnet.sub3.id
  user_data = base64encode(file("userdat2.sh"))
}

#Creating my Application Load Balancer

resource "aws_lb" "MyALB" {
  name = "MyALB-demo"
  internal = false
  load_balancer_type = "application"
  security_groups = [ aws_security_group.mysg-demo.id ]
  subnets = [ aws_subnet.sub1.id, aws_subnet.sub2.id, aws_subnet.sub3.id ] 
}

#Target group for my ALB

resource "aws_lb_target_group" "Mytg" {
  name = "MyTargetGroup"
  port = 80
  protocol = "HTTP"
  vpc_id = aws_vpc.MyVPC.id
  health_check {
    path = "/"
    port = "traffic-port"
  }
}

#Attaching my EC2 instances to the target group

resource "aws_lb_target_group_attachment" "mytgAttach" {
  target_group_arn = aws_lb_target_group.Mytg.arn
  target_id = aws_instance.HTTPserver1.id
  port = 80
}

resource "aws_lb_target_group_attachment" "mytgAttach2" {
  target_group_arn = aws_lb_target_group.Mytg.arn
  target_id = aws_instance.HTTPserver2.id
  port = 80
}

resource "aws_lb_target_group_attachment" "mytgAttach3" {
  target_group_arn = aws_lb_target_group.Mytg.arn
  target_id = aws_instance.HTTPserver3.id
  port = 80
}

resource "aws_lb_listener" "listener" {
  load_balancer_arn = aws_lb.MyALB.arn
  port = 80
  protocol = "HTTP"

  default_action {
    target_group_arn = aws_lb_target_group.Mytg.arn
    type = "forward"
  }
}

output "laodbalancerdns" {
  value = aws_lb.MyALB.dns_name
  
}
