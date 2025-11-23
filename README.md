# DEVOPS_2025
Homework: Vagrant and Cloud-init
Automate creating and provisioning a VM with a project (application stack) of your choice. Ideally, it should consist of at least 4 components, like HTTP server, application, (sql) database, cache (redis?).

The automation for the 1. part of the homework (vagrant) was made by Vagrant and Ansible. The app consists of multiple php and js files, one db file and one css file.
The 2nd part - cloud-init - was done with cloud_init ofcourse and multipass. It's made for the same app.

# --------------- VAGRANT AND ANSIBLE ---------------

### Solution:
1. Download and install Vagrant, virtual box and ansible.
2. Create a box -> in this case it's hashicorp-education/ubuntu-24-04 version 0.1.0
3. Set up sync folders, port forwarding and ansible in Vagrantfile.

  config.vm.box = "hashicorp-education/ubuntu-24-04"
  config.vm.box_version = "0.1.0"

  IMENUJ HOST KOT "phpapp", DODAJ FORWARD PORTE ZA HTTP IN HTTPS
  config.vm.hostname = "phpapp"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 443, host: 8443

  SYNCED FOLDERS, NAMESTO KOPIRANJA (app file mora bit v istem folderju kot vagrantfile - razn ce spremenis
  path za app folder)
  config.vm.synced_folder "./app", "/var/www/html", owner: "www-data", group: "www-data"

  AKTIVIRAJ PROVISION.YML ZA ANSIBLE PROVISION-ANJE
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provision.yml"
  end
  
4. In the provisioning file provision.yml, specify the neccesary tasks such as
     a. Update apt cache
     b. Install web server, PHP, MySQL, Redis
          Apache (web server) and PHP for website loading with php files
          MySQL for DB
          Redis for loading sessions in cache
     c. Certain services to make sure each part is up and running
     d. Copying app files to /var/www/html/myapp
         This part got replaced by syncing folders, which makes it faster and easier to work with. (We just commented it). Instead of the copy part, there is a checker for permissions on the folder.
     e. Creating and importing DB
     f. Setting up a firewall which allows SSH, HTTP and HTTPS to go through

To provision the vm, just type "vagrant provision" - before that you write "vagrant up" to start the vm.
Then after provisioning just type "vagrant ssh" and you're in.

5. Explainations of certain parts

We added init.php to the app, where it connects to Redis and uses Redis for sessions. Every file that had a "start session()" was replaced with require_once "init.php", so all the sessions go through Redis.
In "provision.yml", the SQL imports only if the DB is empty - this avoids the errors that occur when you provision an already provisioned vagrant box (otherwise it'd say "table already exists"). This could be also done by deleting the table (if it exists) and importing it again, but I believe this way is safer and better.

6. Problems we've encountered

Copying the files - By running multiple provisionings to test if certain parts work, it kept copying files into var/www/html/myapp and we ended up with a folder that contained many myapp folders -> /var/www/html/myapp/myapp/myapp/myapp/myapp/myapp - each myapp contained all the app files - fixed that by removing the copy part and syncing the folders.


# --------------- CLOUD INIT AND MULTIPASS ---------------

This part was made similar to the vagrant+ansible part. We used multipass as it's very beginner friendly. The difference between this set up and vagrant set up is that there's no need to download PHP packages as some Ubuntu images already have them - vagrant has a more minimalistic set up - depends on the boxes/images you use.

### Solution:
1. Download and install snapd - with this package manager you download multipass. Cloud-init comes with multipass.
2. Create a file name user-data.yml and add the provisioning text like
      a. Update and upgrade packages
      b. List the packages
      c. Set up the main user
      d. Enable and start services
            SSH, Apache, MySQL, Redis
      f. Download and unzip the app file from github
      g. Clean up the folders
      h. Create DB and User if they don't exist already
      i. Import the apps database
3. Run this command: sudo multipass launch --name phpapp-test --cloud-init user-data.yml
4. It starts provisioning, after it finishes enter the vm: multipass shell phpapp-test
