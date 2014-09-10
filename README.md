# Garden Linux Release

A [BOSH](http://docs.cloudfoundry.org/bosh/) release for deploying Garden
Linux. Garden Linux may be deployed to its own virtual machine using Vagrant BOSH.
Alternatively, Garden Linux may be deployed to a (currently Garden) container in
a virtual machine using BOSH lite.

# Vagrant BOSH

To get started with [Vagrant BOSH](https://github.com/cppforlife/vagrant-bosh):

```sh
cd garden-release/

# Obtain submodules
git submodule update --init --recursive

# install the Vagrant BOSH provisioner
vagrant plugin install vagrant-bosh

# install BOSH
gem install bosh_cli

# provision
vagrant up
```


## Development

See the [usage of directories in a BOSH
release](https://www.pivotaltracker.com/story/show/78508966).


## Debugging

```sh
cd garden-linux-release/

vagrant ssh

# escalate to root
sudo su -

# check logs:
tail -f /var/vcap/sys/log/**/*.log

# check monit:
monit status

# restart garden
monit restart garden

# poke around the deployed jobs
less /var/vcap/jobs/...
```


## Kick the tyres

```sh
# list containers (should be empty)
curl http://127.0.0.1:7777/containers

# create a container
curl -H "Content-Type: application/json" \
  -XPOST http://127.0.0.1:7777/containers \
  -d '{"rootfs":"docker:///busybox"}'

# list containers (should list the handle returned above)
curl http://127.0.0.1:7777/containers

# spawn a process
#
# curl will choke here as the protocol is hijacked, but...it probably worked.
curl -H "Content-Type: application/json" \
  -XPOST http://127.0.0.1:7777/containers/${handle}/processes \
  -d '{"path":"sleep","args":["10"]}'

# from inside the vagrant vm, see 'sleep 10' running:
ps auxf

# hop in the container:
cd /var/vcap/data/garden/depot/${handle}
./bin/wsh
```

## Update the release

Once the VM is up, modify it by issuing:
```
vagrant provision
```

## Destroy the release
```
vagrant destroy
```

# BOSH Lite

To get started with [BOSH lite](https://github.com/cloudfoundry/bosh-lite), follow the
instructions to [Prepare the Environment](https://github.com/cloudfoundry/bosh-lite#install-and-boot-a-virtual-machine)
and [Install and Boot a Virtual Machine](https://github.com/cloudfoundry/bosh-lite#install-and-boot-a-virtual-machine), then:

```sh
cd garden-linux-release/

# Obtain submodules
git submodule update --init --recursive

# create and upload a BOSH release
# (if there are changes in the git repository, specify --force on create)
bosh -n create release
bosh -n upload release
```

Then follow the instructions for downloading a stemcell in [Manually Deploying Cloud Foundry](https://github.com/cloudfoundry/bosh-lite/blob/master/docs/deploy-cf.md#manual-deploy), choosing a stemcell with `warden-boshlite-ubuntu-trusty` in the name:
```
bosh public stemcells
bosh download public stemcell <stemcell_name>
```

Upload the downloaded stemcell to the BOSH lite instance:
```
bosh upload stemcell <stemcell_file_name>
```

Then ...
