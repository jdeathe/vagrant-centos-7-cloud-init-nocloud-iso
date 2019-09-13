# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'digest/sha1'
require 'fileutils'

Vagrant.require_version ">= 1.9.2"

$config_path = File.expand_path(
  "./config.rb",
  File.dirname(__FILE__)
)
$cloud_config_meta_path = File.expand_path(
  "./cidata/meta-data",
  File.dirname(__FILE__)
)
$cloud_config_user_path = File.expand_path(
  "./cidata/user-data",
  File.dirname(__FILE__)
)

# Create a hash of the user-data file so changes result in a new instance-id
$cloud_config_user_hash = Digest::SHA1.file(
  $cloud_config_user_path
).hexdigest
$cloudinit_uid = $cloud_config_user_hash[0,12]

$linked_clone = true
$vm_name = "cloud-init-iso"

if %w(up reload).include? ARGV[0]
  File.open(
    "#{$cloud_config_meta_path}",
    "w"
  ) do |meta|
    meta.write "# Auto-Generated - DO NOT CHANGE THIS\n"
    meta.write "hostname: #{$vm_name}\n"
    meta.write "instance-id: iid-#{$cloudinit_uid}"
  end

  # Generate the NoCloud ISO
  system("
    if command -v hdiutil &> /dev/null; then
      hdiutil makehybrid \
        -quiet \
        -ov \
        -hfs \
        -iso \
        -joliet \
        -default-volume-name cidata \
        -o iso/nocloud.iso \
        cidata/
    elif command -v mkisofs &> /dev/null; then
      mkisofs \
        -R \
        -J \
        -V cidata \
        -o iso/nocloud.iso \
        cidata/*
    fi
  ")
end

$cloud_config_iso = File.expand_path(
  "./iso/nocloud.iso",
  File.dirname(__FILE__)
)

if !File.exist?($cloud_config_iso)
  puts "ERROR: Missing NoCloud ISO file: 
%s" % $cloud_config_iso
  abort
end

Vagrant.configure("2") do |config|
  config.vm.box = "jdeathe/centos-7-x86_64-minimal-cloud-init-en_us"
  config.vm.post_up_message = "After the machine is booted Cloud-Init applied the changes 
defined in user-data. Check that docker (docker-latest) was installed:
  vagrant ssh -- sudo docker info"

  config.vm.define $vm_name do |config|
    config.vm.hostname = $vm_name

    config.vm.provider "virtualbox" do |vb|

      vb.check_guest_additions = false
      vb.functional_vboxsf = false
      vb.linked_clone = $linked_clone
      vb.name = $vm_name

      if File.exist?($cloud_config_iso)
        vb.customize [
          "storageattach", :id,
          "--storagectl", "SATA Controller",
          "--port", "1",
          "--type", "dvddrive",
          "--medium", "#{$cloud_config_iso}"
        ]
      end
    end

    config.vm.provision "shell", keep_color: true,
      name: "Wait for Cloud-init boot-finished",
      inline: "tail -f /var/log/cloud-init-output.log & \
        CIO_PID=${!}; \
        until [[ -e /var/lib/cloud/instance/boot-finished ]]; do \
          sleep 1; \
        done; \
        kill ${CIO_PID}"
  end
end

if %w(up reload).include? ARGV[0]
  File.open(
    "#{$cloud_config_meta_path}",
    "w"
  ) do |meta|
    meta.write "# Auto-Generated - DO NOT CHANGE THIS"
  end
end
