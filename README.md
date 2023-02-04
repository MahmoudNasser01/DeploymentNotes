# DeploymentNotes
this repo is for my deployment notes



Step 1:
create directory called `infrastructure/`
 - in it put the following files (common.conf/gunicorn.conf/nginx.conf)
 
 
 
 Step 2:
 - add the fabfile.py in the main directory of your project
 
 
 
 
 
 
 # please note that $1 is the project name and $2 is the ip address of the server
# create new project and copy files from the original project
cd ~/work
django-admin startproject $1
cp -r menu/main $1/
cp menu/menu/settings.py $1/$1/
cp menu/fabfile.py $1/
cp menu/menu/urls.py $1/$1/
cp menu/requirements.txt $1/
cp menu/.gitignore $1/
cp menu/.env.example $1/.env.example
cp menu/.env $1/.env
cp -r menu/frontend $1/
cp -r menu/infrastructure $1/
cp -r menu/.github $1/

# replace proj with the project name and domain with the domain name in infrastructure files
sed -i 's/proj/'$1'/g' $1/infrastructure/common.conf
sed -i 's/proj/'$1'/g' $1/infrastructure/gunicorn.conf
sed -i 's/domain/'$2'/g' $1/infrastructure/gunicorn.conf

# add missing lines to files that need the project name or domain name
echo "env.hosts = ['ubuntu@$2']" >> $1/fabfile.py
echo 'ROOT_URLCONF = "'$1'.urls"' >> $1/$1/settings.py
echo 'WSGI_APPLICATION = "'$1'.wsgi.application"' >> $1/$1/settings.py

# setup local development environment
cd $1
virtualenv venv
. venv/bin/activate 
pip install -r requirements.txt
python manage.py migrate
python manage.py createdefaultadmin

# setup github repo
git init
git add .
git commit -m "Initial commit"
gh repo create $1 --private
git remote add origin https://github.com/ahmedyasserays/$1.git
git push -u origin main

gh api --method PUT -H "Accept: application/vnd.github+json"  /repos/ahmedyasserays/$1/collaborators/mahmoud-abu-attiya -f permission='push' 
gh api --method PUT -H "Accept: application/vnd.github+json"  /repos/ahmedyasserays/$1/collaborators/m7mdGNo -f permission='push' 

cd ~/work/scripts
sh deploy.sh $1 $2

code ~/work/$1






cd ~/work/$1
# add the server ssh key to the github repo deploy keys
cat ~/.ssh/id_rsa.pub | ssh -i ~/work/aws-ssh-private-key.pem ubuntu@$2 "cat >> .ssh/authorized_keys"
fab keygen
scp ubuntu@$2:~/.ssh/id_rsa.pub ~/work/scripts/temp/$1.pub
gh repo deploy-key add ~/work/scripts/temp/$1.pub --repo github.com/ahmedyasserays/$1 -t "Deploy key for aws"

# add private ssh key to the github repo secrets for the deploy action
ssh-keygen -b 2048 -t rsa -f ~/work/scripts/temp/$1_rsa -q -N ""
cat ~/work/scripts/temp/$1_rsa.pub | ssh ubuntu@$2 "cat >> .ssh/authorized_keys"
gh secret set SERVER_IP --repo github.com/ahmedyasserays/$1 --body "$2"
gh secret set SSH_KEY --repo github.com/ahmedyasserays/$1 --body "$(cat ~/work/scripts/temp/$1_rsa.pub)"
fab deploy
fab createdefaultsuperuser



