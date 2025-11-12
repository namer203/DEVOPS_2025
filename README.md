# DEVOPS_2025
Homework: Vagrant and Cloud-init
Automate creating and provisioning a VM with a project (application stack) of your choice. Ideally, it should consist of at least 4 components, like HTTP server, application, (sql) database, cache (redis?).

The automation for the 1. part of the homework (vagrant) was made by Vagrant and Ansible. The app consists of multiple php and js files, one db file and one css file.

Solution:
1. Download Vagrant, virtual box and ansible.
2. Create a box -> in this case it's hashicorp-education/ubuntu-24-04 version 0.1.0
3. Connect Vagrant with Ansible through modifying the Vagrantfile.

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
         This part got replaced by syncing folders, which makes it faster and easier to work with. (We just commented it). Instead of the copy           part, there is a checker for permissions on the folder.
     e. Creating and importing DB
     f. Enabling SSL module
     g. Restarting services if needed

5. Explainations of certain parts

We added init.php to the app, where it connects to Redis and uses Redis for sessions. Every file that had a "start session()" was replaced with 
require_once "init.php", so all the sessions go through Redis.
In "provision.yml", the SQL imports only if the DB is empty - this avoids the errors that occur when you provision an already provisioned vagrant box (otherwise it'd say "table already exists"). This could be also done by deleting the table (if it exists) and importing it again, but I believe this way is safer and better.

6. Problems we've encountered

Copying the files - By running multiple provisionings to test if certain parts work, it kept copying files into var/www/html/myapp and we ended up with a folder that contained many myapp folders -> /var/www/html/myapp/myapp/myapp/myapp/myapp/myapp - each myapp contained all the app files - fixed that by removing the copy part and syncing the folders. 
