# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'digest/sha1'
require 'fileutils'

Vagrant.require_version ">= 1.9.2"

$cloud_config_user_path = File.expand_path(
  "./iso/data/user-data",
  File.dirname(__FILE__)
)
$cloud_config_meta_path = File.expand_path(
  "./iso/data/meta-data",
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
  File.open("#{$cloud_config_meta_path}", "w") do |meta|
    meta.write "# Auto-Generated - DO NOT CHANGE THIS\n"
    meta.write "hostname: #{$vm_name}\n"
    meta.write "instance-id: #{"iid-%s" % $cloudinit_uid}"
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
        iso/data/
    elif command -v mkisofs &> /dev/null; then
      mkisofs \
        -R \
        -J \
        -V cidata \
        -o iso/nocloud.iso \
        iso/data/*
    fi
  ")
end

$cloud_config_iso = File.expand_path(
  "./iso/nocloud.iso",
  File.dirname(__FILE__)
)

Vagrant.configure("2") do |config|
  config.vm.box = "jdeathe/centos-7-x86_64-minimal-cloud-init-en_us"
  config.vm.post_up_message = "After the machine is booted Cloud-Init will apply the changes 
defined in user-data. Check progress with:
  vagrant ssh -- tail -f /var/log/cloud-init-output.log"

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
  end
end

if %w(up reload).include? ARGV[0]
  File.open("#{$cloud_config_meta_path}", "w") do |meta|
    meta.write "# Auto-Generated - DO NOT CHANGE THIS"
  end
end
