# frozen_string_literal: true

user = ENV['USER']

# virtual machine ram
alpha_ram = 6
beta_ram = 6

setup_user = <<~SCRIPT
  useradd -m #{user} -s /bin/bash -G sudo && echo #{user}:#{user} | chpasswd
  echo "#{user} ALL=(ALL) NOPASSWD: ALL" > /etc/sudoers.d/#{user}
SCRIPT

move_configs = <<~SCRIPT
  mv /tmp/.ssh /home/#{user}/
  (
    # manipulate .ssh/authorized_keys so we can log in over ssh as our own user
    cd /home/#{user}/.ssh
    ls | grep '\.pub$' | xargs cat >> authorized_keys
  )
  chown #{user}:#{user} -R /home/#{user}/.ssh
  mv /tmp/.gitconfig /home/#{user}/.gitconfig
  chown #{user}:#{user} /home/#{user}/.gitconfig
SCRIPT

Vagrant.configure('2') do |config|
  config.vm.provision 'shell', inline: setup_user
  config.vm.provision 'file', source: '~/.gitconfig', destination: '/tmp/.gitconfig'
  config.vm.provision 'file', source: '~/.ssh', destination: '/tmp/.ssh'
  config.vm.provision 'shell', inline: move_configs

  config.vm.define :alpha do |alpha|
    alpha.vm.network 'private_network', ip: '172.100.10.2'
    alpha.vm.box = 'generic/ubuntu2010'
    alpha.vm.network :forwarded_port, guest: 22, host: 2022, host_ip: '127.0.0.1', id: 'ssh'
    alpha.vm.provider 'virtualbox' do |v|
      v.cpus = 4
      v.memory = 1024 * alpha_ram
    end
  end

  config.vm.define :beta do |beta|
    beta.vm.network 'private_network', ip: '172.100.10.3'
    beta.vm.box = 'generic/ubuntu2010'
    beta.vm.network :forwarded_port, guest: 22, host: 2023, host_ip: '127.0.0.1', id: 'ssh'
    beta.vm.provider 'virtualbox' do |v|
      v.cpus = 4
      v.memory = 1024 * beta_ram
    end
  end
end
