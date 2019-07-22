# Stage 1

## 0.0 Hello World
## 3.0 Swarm stack introduction
Using Node1 and Node2

#### Init your swarm
`docker swarm init --advertise-addr $(hostname -i)` On the master node (Node 1). 
`hostname -i` gives the IP Address

This will be the output on Node1
```
Swarm initialized: current node (b3nvddvuwjr7r9o1wb527m4zv) is now a manager.

To add a worker to this swarm, run the followingcommand:

    docker swarm join --token SWMTKN-1-3l60aqx95kyyzu1335c3wno4pw4dlpqqw18cn081ejd4ywmrqg-43yxqps0a1tpgj83m6xbmoao8 10.0.57.3:2377
```
Then you must the join command on Node2

#### Show members of swarm
On Node 1 `docker node ls`

#### Clone the voting-app
On Node 1
```
git clone https://github.com/docker/example-voting-app
cd example-voting-app
```

#### Deploy stack
A stack is a group of services that are deployed together. The docker-stack.yml in the current folder will be used to deploy the voting app as a stack.

On Node 1 `docker stack deploy --compose-file=docker-stack.yml voting_stack`

`docker stack ls` Shows stacks deployed

`docker stack services voting_stack` Shows the service in the stack

`docker service ps voting_stack_vote` Shows the tasks of the vote service

#### Questions
What is a stack? *a multi-service app running on a Swarm*

A stack can: *be deployed from the commandline, can use the compose file format to deploy, can be used to manage services over multiple nodes*