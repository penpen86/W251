# Set up Virtual Server on the IBM Cloud - Softlayer Configuration
For reference, we are doing all the command line operations on local host using ssh tunnel.

## 1.- Create a new virtual server in the IBM Virtual Instance ($root$ = vinicloud)
```
ibmcloud sl vs create --hostname=face-server --domain=server.cloud --cpu=2 --memory=4096 --datacenter=wdc04 --san --os=UBUNTU_16_64 --disk=100 --key=serverKey_ID
```
To get the the list of keys we can use `ibmcloud sl security sshkey-list` as we did in Homework 02. If no key is created, we can create a new key list using `ibmcloud sl security sshkey-add serverKey --in-file ~/.ssh/id_rsa.pub`, after generating the new key with `ssh-keygen`

We make sure that this new server is protected against brute-force password breaking: 
* Edit /etc/ssh/sshd_config
```
PermitRootLogin prohibit-password
PasswordAuthentication no
```

## 2.- Create a Cloud Object Storage (S3 API). 
Via IBM cloud's webpage we can create a Cloud Storage Object. First we will have the credentials for the Object. Then we can create any bucket inside the server
* Bucket name: face-images. This bucket is set to public, so we can access it.

## 3.- Install Docker CE using the instructions on Lab2
I've created a .sh script to do all the updating and installing (check $DockerCE.sh$ as the files attached to the repo)
_(PS: I had to run `sed -i -e 's/\r$//' DockerCE.sh` and `chmod +x DockerCE.sh` to change the End Of Line of the file and make the file executable)_ 

We ran a `docker run hello-world` just to make sure all the installation was a success.

## 4.- Install the IBM CLoud Storage on our VI
We ran the followings commands for this installation

```
sudo apt-get update
sudo apt-get install automake autotools-dev g++ git libcurl4-openssl-dev libfuse-dev libssl-dev libxml2-dev make pkg-config
git clone https://github.com/s3fs-fuse/s3fs-fuse.git
cd s3fs-fuse
./autogen.sh
./configure
make
sudo make install
```

## 5.- Add the Cloud Storage Object credentials to the _face-server_
From _Step 2_, we can have the Access Key ID and the Secret Access Key from our Cloud Storage Object
```
echo "<AccessKeyID>:<SecretAccessKey>" > $HOME/.cos_creds
chmod 600 $HOME/.cos_creds
```

Once the credentials are stored in the HOME PATH, we can mount the object-storage, similar to mounting a SSD or SD card
```
mkdir /mnt/face-images
s3fs face-images /mnt/face-images -o passwd_file=$HOME/.cos_creds -o sigv2 -o use_path_request_style -o url=https://s3.us.cloud-object-storage.appdomain.cloud
```








