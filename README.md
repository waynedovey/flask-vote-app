# flask-vote-app
A sample web poll application written in Python (Flask).

Users will be prompted with a poll question and related options. They can vote preferred option(s) and see poll results as a chart. Poll results are then loaded into an internal DB based on sqlite. As alternative, the application can store poll results in an external MySQL database.

The repo has been modified to make it easy to build and run on OpenShift.

The application has also been primed to be easily bound to RDS using the AWS's Service Catalog command line, svcat.  Here is an [example](https://github.com/sjbylo/lab-ocp4/blob/master/workshop/content/exercises/70-add-cloud-database.md#bind-to-the-database-service) of that.

This application is intended for demo use only.

## Local deployment

This application can be deployed locally. On linux, install git and clone the reposistory:

```
sudo yum install -y git
git clone https://github.com/sjbylo/flask-vote-app
cd flask-vote-app
```

Install the dependencies:

```
pip install flask
pip install flask-sqlalchemy
pip install mysqlclient
```

and start the application:

```
python app.py
Check if a poll already exists in the db
...
* Running on http://0.0.0.0:8080/ (Press CTRL+C to quit)
```

View the app in the browser.  The test script can also be used to test the vote app:

```
./test-vote-app http://localhost:8080
```

Poll question and options are loaded from a JSON file called ``seed_data.json`` under the ``./seeds`` directory. 
This file is filled with default values, change it before starting the application.

The DB data file is called ``app.db`` and is located under the ``./data`` directory. 
To use an external MySQL database, set the environment variables by editing the ``flask.rc`` file under the application directory.

```
nano flask.rc
export PS1='[\u(flask)]\> '
export ENDPOINT_ADDRESS=db
export PORT=3306
export DB_NAME=vote
export MASTER_USERNAME=voteuser
export MASTER_PASSWORD=password
export DB_TYPE=mysql
```

Make sure an external MySQL database server is running according to the parameters above.

Source the file and restart the application:

```
source flask.rc
python app.py
```

Cleanup:

```
rm -f data/app.db    # optionaly remove the database 
```

## Docker deployment

A Dockerfile is provided in the reposistory to build a docker image and run the application as linux container.

On Linux, install and start Docker:

```
sudo yum install -y docker
systemctl start docker
```

Install git and clone the reposistory:

```
sudo yum install -y git
git clone https://github.com/sjbylo/flask-vote-app
cd flask-vote-app
```

Build a Docker image:

```
docker build -t vote-app:latest .
docker images
REPOSITORY            TAG                 IMAGE ID            CREATED             SIZE
vote-app              latest              e6e0578f5f2d        2 minutes ago       695.4 MB
```

Start the container:

```
docker run -d -p 8080:8080 --name=vote-app vote-app:latest
```

Seed data directory, containing the seed data file ``seed_data.json``, can be mounted as an external volume under the host ``/mnt`` directory:

```
cp flask-vote-app/seeds/seed_data.json /mnt
docker run -d -p 8080:8080 -v /mnt:/app/seeds --name=vote-app vote-app:latest
```

An external MySQL database can be used instead of the internal sqlite by setting the desired env variables:

```
docker run -e ENDPOINT_ADDRESS=db \
           -e PORT=3306 \
           -e DB_NAME=vote \
           -e MASTER_USERNAME=voteuser \
           -e MASTER_PASSWORD=password \
           -e DB_TYPE=mysql \
           -d -p 8080:8080  --name=vote-app vote-app:latest
```

Cleanup:

```
docker stop vote-app && docker rm vote-app
```

## Install the app onto OpenShift

Build and launch the app:

```
oc new-app python~https://github.com/sjbylo/flask-vote-app.git --name vote-app
```

Expose the app to the external network:

```
oc expose svc vote-app
```

Start a database (optional, if scale-out if needed):

```
oc new-app --name db mysql:5.7 -e MYSQL_USER=user -e MYSQL_PASSWORD=password -e MYSQL_DATABASE=vote
```

Connect the app to the DB:

```
oc set env dc vote-app \
   ENDPOINT_ADDRESS=db \
   PORT=3306 \
   DB_NAME=vote \
   MASTER_USERNAME=user \
   MASTER_PASSWORD=password \
   DB_TYPE=mysql
```

## Develop the app and launch it on OpenShift from the local directory

Create a 'binary' build.  Binary is refering to the below tar file which is uploaded to he build pod:

```
oc new-build python --name vote-app --binary
```

Start the build.  This will upload the app code from the current working dir:

```
oc start-build vote-app --from-dir=. --follow
```

Wait for the build to complete. Launch the app:

```
oc new-app vote-app
```

Expose the app to the external network:

```
oc expose svc vote-app
```

Test the app, e.g. on mac:

```
VOTE_APP=`oc get route vote-app --template='{{.spec.host}}'`
./test-vote-app $VOTE_APP 
open http://$VOTE_APP/
```

## CodeReady Workspace deployment

You can instantiate a workspace on demand by opening the devfile.yaml file in CodeReady Workspaces.

Example of a Che Factory URL:

```
https://codeready-workspaces.apps.example.com/f?url=https://github.com/sjbylo/flask-vote-app
```


