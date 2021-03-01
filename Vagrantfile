Vagrant.require_version ">= 1.9.7"

$provisioningScript = <<-SCRIPT
if [[ ! -f /VAGRANT_PROVISION ]] ; then
  echo "Provisioning VM, this may take a while!" | tee -a /VAGRANT_PROVISION
  echo "" | tee -a /VAGRANT_PROVISION

  echo "Updating package cache ... " | tee -a /VAGRANT_PROVISION
  sudo apt-get update >> /VAGRANT_PROVISION && echo "Done"

  echo "Installing ansible dependencies ... " | tee -a /VAGRANT_PROVISION
  sudo apt-get install -y python3-apt python3-pip python3-venv python3-urllib3 >> /VAGRANT_PROVISION && echo "Done"

  echo "Creating virtualenv ... " | tee -a /VAGRANT_PROVISION
  if [[ ! -d /opt/ve-ansible ]] ; then
    sudo mkdir -p /etc/ansible
    echo "localhost ansible_connection=local ansible_python_interpreter=/usr/bin/python3" | tee -a /etc/ansible/hosts
    sudo python3 -m venv /opt/ve-ansible >> /VAGRANT_PROVISION && echo "Done"
  else
    echo "Virtualenv already exists." | tee -a /VAGRANT_PROVISION
  fi

  echo "Activating virtualenv ... " | tee -a /VAGRANT_PROVISION
  source /opt/ve-ansible/bin/activate && echo "Done"

  echo "Updating pip ... " | tee -a /VAGRANT_PROVISION
  pip3 install pip --upgrade

  echo "Installing ansible ... " | tee -a /VAGRANT_PROVISION
  pip3 install wheel ansible>=2.9.0 >> /VAGRANT_PROVISION && echo "Done"

  echo "Deactivating virtualenv ... " | tee -a /VAGRANT_PROVISION
  deactivate && echo "Done"
else
  echo "Already Provisioned"
fi
SCRIPT

$ansibleScript = <<-SCRIPT
  echo "Activating virtualenv ... "
  source /opt/ve-ansible/bin/activate

  ansible-galaxy install geerlingguy.docker -f

  cat << EOF > /opt/ve-ansible/provision.yml
---

- name: Ensure docker is installed
  hosts: localhost
  roles:
    - geerlingguy.docker

EOF

  ansible-playbook /opt/ve-ansible/provision.yml

  echo "Deactivating virtualenv ... "
  deactivate

  echo "Adding vagrant to docker group ... "
  sudo usermod -aG docker vagrant
SCRIPT

Vagrant.configure("2") do |config|

  config.vm.hostname      = "dockerbox"
  config.vm.box           = "debian/contrib-buster64"
  config.vm.network       "private_network", ip: "192.168.12.200"
  config.vm.synced_folder ".", "/vagrant", disabled: true
  config.vm.synced_folder "#{ENV['HOME']}/projects", "/home/vagrant/projects"


  config.vm.provider "virtualbox" do |vb|
    vb.gui    = false
    vb.name   = "Dockerbox"
    vb.cpus   = "2"
    vb.memory = "4096"
  end

  config.vm.provision "file", source: "~/.gitconfig", destination: "/home/vagrant/.gitconfig"
  config.vm.provision "file", source: "~/.ssh", destination: "/home/vagrant/.ssh_host"
  config.vm.provision "shell", inline: $provisioningScript
  # config.vm.provision "shell", path: ".provision/scripts/fix_fstab_uuid.sh"
  config.vm.provision "shell", inline: $ansibleScript
  config.vm.provision "shell", inline: "cat /home/vagrant/.ssh_host/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys"
end
