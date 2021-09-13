# Sentry challange Readme file

This task was writen in terraform and developed on AWS. All steps are done one by one as remote-exec, to keep view of stages that terraform is performing. Ofcourse the same result could be done by injecting bash script on remote EC2 instance, by using user-data parameter. But in this case I prefer to use clear terraform. 

# Usage
First install AWS CLI and configure `aws configure`

Next initiate project

`terraform init`

If everything is ok, execute

`terraform plan`

if the main.tf file is valid type

`terraform apply`

After succefull deployment, service should be available under https://sentry.n24.cloud

# DNS Configuration
This script is adding DNS record to my private DNS server which is controlled by Plesk panel. I'm the owner of the n24.cloud domain as well. So basically this part

```
provisioner "remote-exec" {
  inline = [
     "sudo plesk bin dns -a n24.cloud -a sentry -ip ${self.public_ip}",
  ]
  connection {
    type = "ssh"
    user = "kacper"
    private_key = file("keypair/plesk-temp.pem")
    host = "plesk.nasper.pl"
  }
}
```

Is creating DNS A record. To give this block more features, command could check does record already exist, if so then it should be updated with new IP. (this would be usefull in case of many test executions)

Plese note, keypair is not valid anymore.

# Nginx Reverse proxy
Sentry docker is binding on port 9000, therefour I've use nginx as Reverse Proxy service and Let'sEncrypt for certificate request to expose application on https://sentry.n24.cloud

```
provisioner "remote-exec" {
   inline = [
      "sudo unlink /etc/nginx/sites-enabled/default",
      "sudo wget https://raw.githubusercontent.com/kasproso/nginx-proxy-sentry/main/reverse-proxy.conf -O /etc/nginx/sites-enabled/sentry.conf",
      "sudo certbot --nginx --non-interactive --agree-tos --domains sentry.n24.cloud --email webmaster@n24.cloud",
      "sudo nginx -t",
      "sudo service nginx restart",
    ]
   connection {
      type = "ssh"
      user = "ubuntu"
      private_key = file("keypair/Kacper.pem")
      host = self.public_ip
    }
 }
 ```
 
 # Termination
 
 This pipeline doesnt't contain test and termination funtionality. This needs to be done manually. This cound be done in two ways in this case:
 
 Edit main.tf file and change count to 0
 
 ```
 resource "aws_instance" "SentryTest" {
  count = 0
  ```
  then
  ```
  terraform apply
  ```

  or simple execute 
 
  ```
  terraform destroy
  ```
