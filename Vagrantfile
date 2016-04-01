# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure(2) do |config|
  config.vm.box = "ubuntu/xenial64"
  config.vm.provision "shell", inline: <<-SHELL
  sudo apt-get update
  sudo apt-get install -y borgbackup
  touch /home/ubuntu/borgtest
  chmod +x /home/ubuntu/borgtest
  tee /home/ubuntu/borgtest <<'EOF'
#!/bin/bash
set -e

# Make the source
function create_source {
  echo "Creating source..."
  mkdir -p "/tmp/source"
  touch "/tmp/source/testfile"
}

# Make the backup destination
function create_destination {
  echo "Creating destination..."
  mkdir -p "/tmp/backups"
}

# Make sure there's a borg repo
function init_repo {
  echo "Initializing borg repo..."
  $BACKUP_TOOL init --encryption=none "/tmp/backups" || true
}

# Run a backup function create
function create {
  echo "Creating new backup..."
  $BACKUP_TOOL create -v --stats \
    "/tmp/backups"::`date | md5sum | cut -c -12` \
    "/tmp/source"
}

# Prune old backups
function prune {
  echo "Pruning old backups..."
  $BACKUP_TOOL prune \
    --keep-within=10d \
    --keep-weekly=4 \
    --keep-monthly=-1 \
    "/tmp/backups"
}

# Check the backup
function check {
  echo "Checking backups..."
  $BACKUP_TOOL check "/tmp/backups"
}

# Unmount the source and the backup disk
function clean_up {
  echo "Unmounting /tmp/backups"
  #umount $BACKUP_MOUNT
  echo "Unmounting /tmp/source"
  #umount $SOURCE_DIR
}

function run_backup {
  create_source &&
  create_destination &&
  init_repo &&
  create &&
  prune &&
  check
}

BACKUP_TOOL=/usr/bin/borg

trap clean_up EXIT
run_backup
clean_up
trap - EXIT
EOF
  touch /home/ubuntu/traptest
  chmod +x /home/ubuntu/traptest
  tee /home/ubuntu/traptest <<'EOF'
#!/bin/bash
set -e

function work {
  echo "Doing some work..."
  sleep 2
}

function processing {
  work && work && work && work
}

function cleanup {
  echo "Cleaning up"
}

trap cleanup EXIT
processing
cleanup
trap - EXIT
EOF
  SHELL
end
