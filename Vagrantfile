numnodes=2
baseip="192.168.10"

#global script
$global = <<SCRIPT

#check for private key for vm-vm comm
[ -f /vagrant/id_rsa ] || {
  ssh-keygen -t rsa -f /vagrant/id_rsa -q -N ''
}

#deploy key
[ -f /home/vagrant/.ssh/id_rsa ] || {
    cp /vagrant/id_rsa /home/vagrant/.ssh/id_rsa
    chmod 0600 /home/vagrant/.ssh/id_rsa
}

#allow ssh passwordless
grep 'vagrant@node' ~/.ssh/authorized_keys &>/dev/null || {
  cat /vagrant/id_rsa.pub >> ~/.ssh/authorized_keys
  chmod 0600 ~/.ssh/authorized_keys
}

#exclude node* from host checking
cat > ~/.ssh/config <<EOF
Host node*
   StrictHostKeyChecking no
   UserKnownHostsFile=/dev/null
EOF

#populate /etc/hosts
for x in {11..#{10+numnodes}}; do
  grep #{baseip}.${x} /etc/hosts &>/dev/null || {
      echo #{baseip}.${x} node${x##?} | sudo tee -a /etc/hosts &>/dev/null
  }
done

SCRIPT

Vagrant.configure("2") do |config|
  config.vm.provision "shell", privileged: false, inline: $global
  prefix="node"
  #node box
  (1..numnodes).each do |i|
    vm_name = "#{prefix}#{i}"
    config.vm.define vm_name do |node|
      node.vm.box = "hashicorp/precise64"
      node.vm.hostname = vm_name
      ip="#{baseip}.#{10+i}"
      node.vm.network "private_network", ip: ip
    end
  end
end
