D4A bundle intro
-----

To create and deploy a bundle on D4A you need the following:

 * docker-compose built of `master`: https://dl.bintray.com/docker-compose/master/
 * D4A deployment, with SSH tunnel configured: `$ ssh -NL localhost:2375:/var/run/docker.sock root@<SSH-ELB>`
 * Experimental CLI build:
    - https://experimental.docker.com/builds/Windows/x86_64/docker-latest.zip

Set `DOCKER_HOST` to point at the tunnel endpoint:

    $Env:DOCKER_HOST="127.0.0.1:2375"

Create the bundle and push containers. The `docker-compose.yml` file in this project store images referenced in the bundle on Docker Hub:

    docker-compose build
    docker-compose push
    docker-compose pull db redis
    docker-compose bundle --fetch-digests
    ...
    Wrote bundle to examplevotingapp.dsb

At this point, you can `scp` the bundle to the manager. If you have multiple managers, be sure that it got onto the manager you're using.

    scp -i ~/Downloads/test-key-friis.pem examplevotingapp.dsb root@friism-ELB-SSH-1510648233.us-west-1.elb.amazonaws.com:~/.

Log onto the manager and deploy:

    $ ssh -i ~/Downloads/test-key-friis.pem root@friism-ELB-SSH-1510648233.us-west-1.elb.amazonaws.com
    Welcome to Docker!
    ~ # docker deploy examplevotingapp
    Loading bundle from examplevotingapp.dsb
    Creating network examplevotingapp_back-tier
    Creating network examplevotingapp_front-tier
    Creating service examplevotingapp_result-app
    Creating service examplevotingapp_voting-app
    Creating service examplevotingapp_worker
    Creating service examplevotingapp_db
    Creating service examplevotingapp_redis

Add labels to the services that you want to expose to the public internet through the ELB:

    docker service update --label docker.swarm.lb=http://:80 votingapp_voting-app
    docker service update --label docker.swarm.lb=http://:81 votingapp_result-app

The sites are now accesible on the ELB DNS target.

Example Voting App
==================

This is an example Docker app with multiple services. It is run with Docker Compose and uses Docker Networking to connect containers together. You will need Docker Compose 1.6 or later.

More info at https://blog.docker.com/2015/11/docker-toolbox-compose/

Architecture
-----

* A Python webapp which lets you vote between two options
* A Redis queue which collects new votes
* A Java worker which consumes votes and stores them in…
* A Postgres database backed by a Docker volume
* A Node.js webapp which shows the results of the voting in real time

Running
-------

Run in this directory:

    $ docker-compose up

The app will be running on port 5000 on your Docker host, and the results will be on port 5001.

Docker Hub images
-----------------

Docker Hub images for services in this app are built automatically from master:

 - [docker/example-voting-app-voting-app](https://hub.docker.com/r/docker/example-voting-app-voting-app/)
 - [docker/example-voting-app-result-app](https://hub.docker.com/r/docker/example-voting-app-result-app/)
 - [docker/example-voting-app-worker](https://hub.docker.com/r/docker/example-voting-app-worker/)
