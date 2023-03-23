# really simple web application

This is a lightweight application to demo deployment to OpenShift on https://appuio.ch and accessing multiple backend services (MySQL/MariaDB, PostgreSQL and Redis) within an OpenShift cluster

One can deploy this project using
* Web-GUI using Source to image (s2i): "Add to project" -> search for "Python" -> Choose "Python" -> Next -> Version: 3.5, Name: flask-helloworld, Git Repository: https://github.com/arska/flask-helloworld.git -> Create -> Close

* OpenShift Source to Image on the CLI using the Python builder for the application only
```
oc new-app python:3.5~https://github.com/arska/python-helloworld
oc expose service python-helloworld
```
* OpenShift Docker Build (see Dockerfile, application only)
```
oc new-app --strategy=docker https://github.com/arska/python-helloworld
oc expose service python-helloworld
```
* OpenShift Template (see template.yaml) including the 3 databases:
```
oc new-app -f https://raw.githubusercontent.com/arska/python-helloworld/master/template.yaml -p NAME=python-helloworld
```

you can clean up after with:
```
oc delete all,secrets -l app=python-helloworld
```

you can also build/run this locally using docker:
```
docker build -t python-helloworld .
docker run -p 8080:8080 python-helloworld
```
the application is then accessible at http://127.0.0.1:8080/

## docker build demo
~~~~
git clone https://github.com/arska/python-helloworld
cd python-helloworld
less app.py
python app.py
less requirements.txt
virtualenv -p python3 venv
. venv/bin/activate
pip install -r requirements.txt
python app.py
open http://localhost:8080/
less Dockerfile
docker build -t python-helloworld .
docker image ls
docker run -p 8080:8080 -e MESSAGE="Hello Python Summit 2017" python-helloworld
open http://localhost:8080/
~~~~
## OpenShift
~~~~
https://console.appuio.ch/console/project/appuio-salesdemo1/overview
Add to Project
python 3.5 latest
name pysummit
https://github.com/arska/python-helloworld
OK
add webhook to github.com
copy url
click github link
add url, remember to choose JSON as type
close github
open build log
go to overview
open url http://pysummit-appuio-salesdemo1.appuioapp.ch/
show hostname in pods list
scale to 3
run in console
while [ 1 ] ; do curl http://pysummit-appuio-salesdemo1.appuioapp.ch/ ; echo; sleep 1; done
show changing hostname
quit curl in console
show pods list, show one pod detail
click on health-check warning, add default checks
show rolling deployment
applications → routes → actions → edit → show details → TLS = edge, insecure = redirect
show new url, show green lock
scale to 1
Switch to Terminal
oc status
oc new-app python~https://github.com/arska/python-helloworld --name "pysummit2" -e MESSAGE="hello python summit 2017"
oc logs -f bc/pysummit2
oc status
oc expose svc/pysummit2
open http://pysummit2-appuio-salesdemo1.appuioapp.ch
switch to https://console.appuio.ch/console/project/appuio-salesdemo1/overview to show side-by-side
oc scale dc pysummit2 --replicas=3
oc status
oc delete all -l app=pysummit2
oc status
oc export all -o json
oc export bc,is,svc,dc,route
oc export bc,is,svc,dc,route --as-template=mytemplate > template.yaml
sed -i '' 's/pysummit/${NAME}/' template.yaml
cat >> template.yaml

parameters:
- description: The name assigned to all of the frontend objects defined in this template.
  displayName: Name
  name: NAME
  required: true
  value: pysummit-template

ctrl-D
oc process -f template.yaml NAME=lol | oc create -f -
show health checks are there
oc delete all -l app=lol
oc create -f template.yaml
webgui → add to project → search: mytemplate
that was too easy, only one service
webgui → add to project → django-psql-example
group django and postgresql
show postgresql env and log
show django build log
takes about 3 minutes, so lets demo the auto-update
go to https://github.com/arska/python-helloworld/blob/master/README.md and commit dummy edit
show build being triggered
when django build is done and pod is started from pod console
python manage.py createsuperuser
log in to django admin
show log
~~~~
## clean up
~~~~
oc delete all -l app=pysummit
oc delete all -l app=django-psql-example
oc delete secret django-psql-example
~~~~

