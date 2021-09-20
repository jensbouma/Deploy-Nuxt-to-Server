# Deploy-Nuxt-to-server
Deploy Nuxt App with Git on test/production server

Where using home/beta/ as our home directory on the server it can be changed when necessarily.


#SSH to your server (with certificates)
ssh root@ipnummer

#Create App folder (in users home directory)
mkdir  /home/beta/app/
mkdir /home/beta/tmp/

#Create repository folder
cd /home/beta/
mkdir app.git && cd app.git
git init --bare

#Set permissions
chgrp -R users  /home/beta/app.git/
chmod -R g+rwX .

#Sets the setgid bit on all the directories
find . -type d -exec chmod g+s '{}' +

#Make the directory a Git shared repo
git config core.sharedRepository group

#Edit githook post-receive and add the following
vi hooks/post-receive



#Make Executable
chmod +x hooks/post-receive

#Install PM2
apt install pm2 -g

#Connect dev machine
git remote add beta ssh://<your-user>@<your-ip>/home/beta/app.git/

#Commit and push
git add . 
git commit -m “Something changed“
git push beta dev
