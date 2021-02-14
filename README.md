# jenkins-security-pipelines
Jenkins security pipelines for Kubernetes

* anchoreScanner - vulnerability scan using existing Anchore Engine
* buildDocker - build, test, push Docker image getting Dockerfile from specified repository
* dependencycheckUpdate - trigger update of Dependency-Check installed with MariaDB/MySQL backend. (4 cores to speed up)
* scoutsuite - ScoutSuite scan for AWS account
* zapBaseline - ZAP basic scan for specified URL
* zapActivescan - ZAP Active scan for specified URL

Some pipelines require to have Jenkins credentials and appropriate plugins. 
All run as K8S pods hence integration of such with Jenkins is required.

ZAP scan tasks use a little bit old reporting plugin, to be changed when expected new one is finally available.
