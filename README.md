# Simple chaincode demo
Chaincode implementing a blacklist on Hyperledger 1.0.5 .

Idea: Participants can add entries to blacklist, and count the number of entries by different peers
for the same name. E.g. A user might consider jon@doe.name as blacklisted if 2 peers have added him.

## Installation and trial

1. Install docker
2. Follow [installation of Hyperledger samples](http://hyperledger-fabric.readthedocs.io/en/release/chaincode4ade.html#install-hyperledger-fabric-samples)
and start up in three terminals as described:
Network, a chaincode environment, and the connection to the CLI container.
3. In the chaincode terminal: `git clone https://github.com/georgkoester/blacklist_chaincode.git`
4. Run tests. Should PASS:

 ```
 cd blacklist_chaincode
 go test
 ```

 Yes, you need to do this in the chaincode environment. See below for how to get an almost complete
 dev environment setup on your machine.

5. Start chaincode:

 ```
 go build
 CORE_PEER_ADDRESS=peer:7051 CORE_CHAINCODE_ID_NAME=blacklist:0 ./blacklist_chaincode
 ```

6. Switch to CLI terminal, run `docker exec -it cli bash`
7. Install chaincode:

 ```
 peer chaincode install -p chaincodedev/chaincode/blacklist_chaincode -n blacklist2 -v 0
 ```

8. Instantiate one blacklist in channel 'myc' (calls the Init method!):

 ```
 peer chaincode instantiate -n blacklist -v 0 -c '{"Args":["init", "testlist"]}' -C myc
 ```

9. Invoke a method to add an entry:

 ```
 peer chaincode invoke -n blacklist -c '{"Args":["add", "email", "jon@doe.name"]}' -C myc
 ```

10. Invoke a method to count entries in blacklist:

 ```
 peer chaincode invoke -n blacklist -c '{"Args":["count", "email", "jon@doe.name"]}' -C myc
 ```

11. To test a higher count add more peers and invoke the add method from them for jon@doe.name ...

## FAQ and trouble shooting

- __Where do I find documentation on shim commands?__ Check out the `interfaces_stable.go` file, e.g. in the
fabric github: [interface_stable.go on master branch](https://github.com/hyperledger/fabric/blob/master/core/chaincode/shim/interfaces_stable.go).
Be aware that hyperledger is under heavy development, so you might want to switch to the tagged version, e.g.
[interfaces.go version 1.0.5, selected from interfaces_stable and interfaces_experimental](https://github.com/hyperledger/fabric/blob/v1.0.5/core/chaincode/shim/interfaces.go).

- __I cannot connect to the CLI docker container:__ I CTRL+C the docker-compose process and `docker rm` `cli`,
`chaincode`, `orderer`, and `peer`. Then docker-compose up again. Now you might have to reinstall and reinstantiate,
depending on if you also cleaned the directory of files. I suggest you just use a new chaincode name and
start over. You don't need to run chaincode containers for old chaincodes.

- __I cannot reinstall the chaincode:__ Chaincodes need to be installed either with a new name (`-n`) or
new version (`-v`).

- __I cannot reinstantiate the chaincode:__ Chaincode instances are limited to one per channel. Use a new name.

- __Aren't the chaincode limits of one instance and one container-per-instance weird?__ Depends on how you
create your application. Let's meet and discuss!

- __Isn't the requirement to run a peer to be able to interact with the chaincode inefficient?__ Depends again,
but be aware that it is possible to setup your own user management inside of the chaincode. This would
allow you to setup a full user- and auth/auth-management following your design.


## Dev environment setup for your IDE

You need to install a couple of additional dependencies that I didn't care to install to get a
build environment to work in your IDE. I use the docker environment instead, pasting code into
the chaincode files in docker right now. file sharing is also quite easy when you adapt
the docker compose configuration.

1. Install go
2. Setup GOPATH, e.g. in `/Users/<your user>/go` . Also your IDE might require restarting or setting
a config variable. Idea required me to File->Invalidate Caches/Restart )
3. Clone Hyperledger fabric from https://github.com/hyperledger/fabric to

 ```
 /Users/<your user>/go/src/src/github.com/hyperledger
 ```
 so that the fabric code is in
 ```
 /Users/<your user>/go/src/src/github.com/hyperledger/fabric
 ```

4. Now your IDE should provide you with code completion and src navigation/documentation.
