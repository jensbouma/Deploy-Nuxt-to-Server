# Deploy-Nuxt-to-Server
This is a guide how to deploy a Nuxt App with Git Push to a test or production server.

The script handle the instalation of NPM packages and starts build the nuxt server before running it on port 3000.
Ngnix is proxying my requests, to the webserver, to 127.0.0.1:3000.

In this example /home/beta/ is used as the home directory.
The repository I want to push is dev, in the post-receive script this one gets selected to checkout.


## SSH to your server (with certificates)
ssh root@ipnummer

## Create App folder (in users home directory)
mkdir  /home/beta/app/

mkdir /home/beta/tmp/

## Create repository folder
cd /home/beta/

mkdir app.git && cd app.git

git init --bare

## Edit githook post-receive and add the following
vi hooks/post-receive

```
#!/bin/sh
BRANCH="beta"
TARGET="/home/beta/app"
TEMP="/home/beta/tmp"
REPO="/home/beta/app.git"

while read oldrev newrev ref
do
        # only checking out the master (or whatever branch you would like to deploy)
        if [ "$ref" = "refs/heads/$BRANCH" ];
        then
                echo "Ref $ref received. Deploying ${BRANCH} branch to production..."
                mkdir -p $TEMP
                git --work-tree=$TEMP --git-dir=$REPO checkout -f $BRANCE

                cd $TEMP

                npm install
                npm run build

                rm -rf $TARGET
                mv $TEMP $TARGET && cd /
                wait 2
                cd $TARGET
                wait 1
                pm2 start
        else
                echo "Ref $ref received. Doing nothing: only the ${BRANCH} branch may be deployed on this server."
        fi
done

```

## Make Executable
chmod +x hooks/post-receive

## Install PM2
apt install pm2 -g

## Add ecosystem.config.js to your nuxt project
```
module.exports = {
    apps: [
      {
        name: 'Beta',
        exec_mode: 'cluster', // Optional: If you want it run multiple instances.
        instances: 'max', // Or a number of instances.
        // 'max' auto detects how many CPU cores there are.
        // The previous option must exist to use the above.
        script: './node_modules/nuxt/bin/nuxt.js',
        args: 'start',
      },
    ],
  }
```

## Add the server to the development machine
git remote add beta ssh://[your username]@[your-ip]/home/beta/app.git/

## Commit and push
git add . 

git commit -m “Something changed“

git push beta dev
