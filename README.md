# molecule-ansible-vagrant-macosx
Example project showing how to use Ansible &amp; Molecule with MacOS as infrastructure

[![asciicast](https://asciinema.org/a/239997.svg)](https://asciinema.org/a/239997)

As introduced in [molecule-ansible-docker-vagrant](https://github.com/jonashackt/molecule-ansible-docker-vagrant), we're using a system level use case (installing Docker) to test a Ansible role with [Molecule](https://molecule.readthedocs.io/).

Running MacOSX in VirtualBox with Vagrant isn't a common task, but you maybe need that to test your development enviroment Ansible setup. The problem is already solved here: [vagrant-macosx](https://github.com/jonashackt/vagrant-macosx)

Therefore we use the [molecule.yml](docker/molecule/vagrant-macosx/molecule.yml) to implement these findings:

```yaml
scenario:
  name: vagrant-macosx

driver:
  name: vagrant
  provider:
    name: virtualbox
platforms:
  - name: vagrant-macosx
    box: yzgyyang/macOS-10.14
    memory: 4096
    cpus: 2
    provider_raw_config_args:
      # Some needed OSX configs
      - "customize ['modifyvm', :id, '--cpuid-set', '00000001', '000106e5', '00100800', '0098e3fd', 'bfebfbff']"
      - "customize ['setextradata', :id, 'VBoxInternal/Devices/efi/0/Config/DmiSystemProduct', 'MacBookPro11,3']"
      - "customize ['setextradata', :id, 'VBoxInternal/Devices/efi/0/Config/DmiSystemVersion', '1.0']"
      - "customize ['setextradata', :id, 'VBoxInternal/Devices/efi/0/Config/DmiBoardProduct', 'Iloveapple']"
      - "customize ['setextradata', :id, 'VBoxInternal/Devices/smc/0/Config/DeviceKey', 'ourhardworkbythesewordsguardedpleasedontsteal(c)AppleComputerInc']"
      - "customize ['setextradata', :id, 'VBoxInternal/Devices/smc/0/Config/GetKeyFromRealSMC', '1']"
      - "customize ['setextradata', :id, 'VBoxInternal2/EfiGopMode', '4']"

provisioner:
  name: ansible
  lint:
    name: ansible-lint
    enabled: false
  playbooks:
    converge: ../playbook.yml

lint:
  name: yamllint
  enabled: false
verifier:
  name: testinfra
  directory: ../tests/
  env:
    # get rid of the DeprecationWarning messages of third-party libs,
    # see https://docs.pytest.org/en/latest/warnings.html#deprecationwarning-and-pendingdeprecationwarning
    PYTHONWARNINGS: "ignore:.*U.*mode is deprecated:DeprecationWarning"
  lint:
    name: flake8
  options:
    # show which tests where executed in test output
    v: 1
```

Now let's try to run the MacOSX VagrantBox with Molecule:

```
cd docker
molecule --debug create --scenario-name vagrant-macosx
```