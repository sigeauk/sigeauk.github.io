---
layout: default
title: "GEKO: Build Part 2"
date: 2025-08-11
description: "Building GEKO - the G"
---

# GEKO: Build Part 2

### <span class="highlight">Intro: Why GEKO?</span>

Maybe I should answer what is GEKO first... G = Gitlab, E = Elasticsearch, K =Kibana O = OpenCTI
What if your Gitlab repositories could do more than just host your detection rules.
What if it could validate, manage, update, revert, query and even more . Welcome to Detection as Code (DaC) - where Gitlab, Elasticsearch, and Kibana team up to not only host your detections, but change the way your rules are managed.
If you missed GEKO Part 1  follow the link, if not hello again.

### Where we left off

We used the below docker-compose.yml to set up and run gitlab, by running  `docker-compose -d up gitlab` in the same directory as the file. We retrieved the password from `/etc/gitlab/initial_root_password` and signed in. 

```yaml
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.17.0
    container_name: es
    environment:
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - xpack.security.enabled=true
      - ELASTIC_PASSWORD=Changeme
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - esdata:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:8.17.0
    container_name: kib
    volumes:
      - /home/lab/kibana.yml:/usr/share/kibana/config/kibana.yml
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
      - ELASTICSEARCH_SERVICEACCOUNTTOKEN=<run the following in elastic to get key> # sudo docker exec -it es ./bin/elasticsearch-service-tokens create elastic/kibana kibana (Run this in ES to generate a token)
      - xpack.encryptedSavedObjects.encryptionKey=abcdeabcdeabcdeabcdeabcdeabcdeab
    ports:
      - "5601:5601"
    depends_on:
      - elasticsearch
    networks:
      - elastic

  gitlab:
    image: gitlab/gitlab-ce:latest
    container_name: gitlab
    hostname: gitlab.local
    environment:
      GITLAB_OMNIBUS_CONFIG: |
        external_url 'http://gitlab.local'
        gitlab_rails['gitlab_shell_ssh_port'] = 2224
    ports:
      - "80:80"
      - "443:443"
      - "2224:22"
    volumes:
      - gitlab_config:/etc/gitlab
      - gitlab_logs:/var/log/gitlab
      - gitlab_data:/var/opt/gitlab
    networks:
      - elastic

  gitlab-runner:
    image: gitlab/gitlab-runner:latest
    container_name: gitlab-runner
    volumes:
      - './runner_config:/etc/gitlab-runner'
      - '/var/run/docker.sock:/var/run/docker.sock'
    networks:
      - elastic
    restart: always

volumes:
  esdata:
    driver: local
  gitlab_config:
    driver: local
  gitlab_logs:
    driver: local
  gitlab_data:
    driver: local

networks:
  elastic:
    driver: bridge
```

### Next Steps: Add a runner.

Using the docker-compose.yml above, all we need to do to start the runner is run docker-compose -d up gitlab-runner. This will align a runner to run our commands. But we need to let Gitlab know about the runner, and tell the runner what we want it to do. 

Go to settings > CI/CD > Runners, as shown below. 

Clicking New project runner will take us to the screen below, ensure Run untagged jobs is checked. this will allow us to run jobs that believe it or not are untagged. We may want to tag jobs in the future to only run a python job if tagged with python for example.

Once we click Create runner Gitlab will instruct us what we need to do, although we'll make a few adjustments as our runner is running in a docker instance.

### Side quest: Docker 

I said in Part 1 that I'd share some docker commands, so before we move on and run some commands here's a few helpful things;
We've currently only run docker from a docker-compose.yml file, but we could just as easily run it as 1 command line; which would be cumbersome for persistence. If we wanted to start the runner from above instance the command would be;

`docker run -d --name gitlab-runner --restart always --network elastic -v $(pwd)/runner_config:/etc/gitlab-runner -v /var/run/docker.sock:/var/run/docker.sock gitlab/gitlab-runner:latest`  

You can see how this can grow, however we will need to run some of these going forward. So below should clarify some thing. 

| Command | Description |
|---|---|
|docker --help | Shows the options available, --help can be used through to give current options available. |
|-v or --volume | Map volumes or folders from your host to the docker container <local>:<container>. |
| exec | Execute commands in the running container. |
|-d or --detach | Runs the container in the background (detached).|
|-i or --interactive | Keeps STDIN open, even if not attached, so we can interact with the container.|
| -t or --tty | Allocates a pseudo-TTY (i.e., creates a terminal session).|
|--network| Attach container to specific network, to allow them to communicate.|
|-p or --ports | Maps host ports to container ports.|

### Connect the runner!

Now that we've familiarized ourselves with some commands, we can add the GitLab-runner. After clicking Create runner we get the register runner page. 
> If this didn't load, change the domain to the host IP of Gitlab, alternatively, add the IP to your host file.

We need to run the command gitlab-runner register from our gitlab-runner container we can run the command from our host with `docker exec -it gitlab-runner gitlab-runner register` or docker exec -it gitlab-runner bash (giving us a shell to the Gitlab-runner) followed by gitlab-runner register. Either way we'll be presented with a few questions.

I've provided my answers 'http://gitlab.local', my token that was generated by Gitlab, then a name, I've chosen 'Linux' then an executor. This will depend on your needs, but I've chosen 'shell'. This means that any commands I give to the runner, the runner will run as shell commands. Once completed, you should see the You've registered a new runner. We can now use the runner.

### Conclusion

With GitLab and Gitlab-runner all talking nicely in Docker, and our GitLab Runner now registered and ready, we’ve officially laid the groundwork for Detection as Code.

### NEXT STEPS

We’ll get the runner running, along with Elasticsearch and Kibana. 
Remember: GEKO isn’t just a stack; it’s a mindset. By combining version control, pipeline automation, and search-driven visibility, you’re building a system that’s powerful, traceable, and scalable. Stay tuned, this is where GEKO really starts to fly.