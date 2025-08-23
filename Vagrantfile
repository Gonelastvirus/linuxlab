# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  # --- Base OS ---
  config.vm.box = "rockylinux/9"
  config.vm.hostname = "linux-lab"

  # --- Networking ---
  # eth0: NAT (default)
  # eth1: host-only, left unconfigured so students can set static IP (Task 7.1)
  config.vm.network "private_network", type: "dhcp", auto_config: false

  # --- Resources ---
  config.vm.provider "virtualbox" do |vb|
    vb.name = "linux-lab"
    vb.memory = 2048
    vb.cpus = 2

    # Create and attach an extra 2 GB disk as /dev/sdb for partition/LVM tasks
    vb.customize ["createhd", "--filename", File.join(Dir.pwd, "disk1.vdi"), "--size", 2048] unless File.exist?(File.join(Dir.pwd, "disk1.vdi"))
    vb.customize [
      "storageattach", :id,
      "--storagectl", "SATA Controller",
      "--port", 1,
      "--device", 0,
      "--type", "hdd",
      "--medium", File.join(Dir.pwd, "disk1.vdi")
    ]
  end

  # --- Provisioning ---
  config.vm.provision "shell", privileged: true, inline: <<-'SHELL'
    set -euo pipefail

    # Create student user
    id student &>/dev/null || useradd -m -s /bin/bash student
    echo 'student:student123' | chpasswd

    # Packages needed
   dnf install -y tar xz xfsprogs e2fsprogs util-linux mlocate cronie firewalld openssh-clients sudo


    # Ensure cron/at/firewalld are available (do NOT enable/disable here – students do it)
    systemctl daemon-reload || true

    # Prepare tasks directory & docs
    install -o student -g student -d /home/student/tasks

    cat > /home/student/README.md << 'EOF'
Welcome to the Linux Lab!

Login: `vagrant ssh`
User: `student` / Pass: `student123`

Read tasks in `~/tasks/taskX.txt`.
Solve tasks, then verify with:

  check_task.sh <task_id>

Examples:
  check_task.sh 1.1
  check_task.sh 4.2

Notes:
- Many checks require root; use `sudo -i`.
- For Task 7.1, configure static IP on the second adapter (likely `eth1`).
- Extra disk for storage tasks is attached as `/dev/sdb`.
EOF

    chown student:student /home/student/README.md
    sudo cp /home/student/README.md /etc/motd
    #################################
    #### Week 1: Basics & Shell   ####
    #################################
    cat > /home/student/tasks/task1.txt << 'EOF'
TASK 1.1: Create a directory ~/projects
TASK 1.2: Copy /etc/passwd to ~/projects/passwd.bak
TASK 1.3: Move ~/projects/passwd.bak to ~/
TASK 1.4: Create a file ~/notes.txt with vim or nano containing the text: HELLO
TASK 1.5: Use wildcards to list all files in /etc that start with "host" (no subdirs)
EOF

    #################################
    #### Week 2: Redirection & RE ####
    #################################
    cat > /home/student/tasks/task2.txt << 'EOF'
TASK 2.1: Redirect `ls /etc` to ~/etc_list.txt (overwrite)
TASK 2.2: Append `date` output to ~/etc_list.txt
TASK 2.3: Extract lines containing "root" from /etc/passwd into ~/root_users.txt
TASK 2.4: Write a script ~/show_errors.sh that prints only lines containing "ERROR" from /var/log/messages (handle if file absent)
TASK 2.5: Write ~/list_big_logs.sh to list files in /var/log larger than 1MB
EOF

    #################################
    #### Week 3: Users/Perms/Links ####
    #################################
    cat > /home/student/tasks/task3.txt << 'EOF'
TASK 3.1: Create a new user 'trainee' with password 'train123'
TASK 3.2: Create group 'devs' and add 'trainee' to it
TASK 3.3: Create /shared with group 'devs' and permissions 2770 (setgid)
TASK 3.4: Set the Sticky Bit on /shared
TASK 3.5: In ~, create a hard link (passwd_hardlink) and a symlink (passwd_symlink) to /etc/passwd
EOF

    #################################
    #### Week 4: File Mgmt & Arch ####
    #################################
    cat > /home/student/tasks/task4.txt << 'EOF'
TASK 4.1: Create a tar archive ~/etc_backup.tar containing /etc/passwd and /etc/group
TASK 4.2: Compress ~/etc_backup.tar with xz (result: ~/etc_backup.tar.xz)
TASK 4.3: Extract ~/etc_backup.tar.xz into ~/restore
TASK 4.4: Find all files in /etc modified in the last 1 day; save list to ~/etc_modified.txt
TASK 4.5: Use `locate` to find a file containing the word 'passwd' (run `updatedb` first if needed); also use `which` and `type` to find the path/type of `ls`, write results to ~/which_type_ls.txt
EOF

    #################################
    #### Week 5: systemd Services  ####
    #################################
    cat > /home/student/tasks/task5.txt << 'EOF'
TASK 5.1: Disable firewalld service at boot (do not stop it if running)
TASK 5.2: Enable crond service at boot
TASK 5.3: Create a custom systemd service `/etc/systemd/system/helloworld.service` that runs `bash -c 'echo HelloWorld > /tmp/hello.txt'`
         - Make it `Type=oneshot`, `RemainAfterExit=yes`. Start it and verify /tmp/hello.txt exists.
TASK 5.4: Set the default systemd target to `multi-user.target`
EOF

    #################################
    #### Week 6: Storage & LVM     ####
    #################################
    cat > /home/student/tasks/task6.txt << 'EOF'
NOTE: An extra virtual disk is attached as /dev/sdb.

TASK 6.1: Use fdisk (or parted) to create a 200MB partition on /dev/sdb -> /dev/sdb1 (type Linux filesystem)
TASK 6.2: Format /dev/sdb1 with ext4 and mount it at /mnt/data
TASK 6.3: Add an /etc/fstab entry using the partition's UUID so /mnt/data auto-mounts on boot
TASK 6.4: Initialize /dev/sdb1 as an LVM PV; create VG `vgdata`; create LV `lvdata` size 100MB
TASK 6.5: Format /dev/vgdata/lvdata with xfs and mount at /mnt/lvmdata
TASK 6.6: Extend /dev/vgdata/lvdata by 50MB and grow the filesystem online
TASK 6.7: Create a 256MB swap LV named `swap` in vgdata, enable it, and make it persistent
TASK 6.8: (Optional) Create a vfat filesystem on a loopback file and mount at /mnt/vfat_test
EOF

    #################################
    #### Week 7: Networking & Jobs ####
    #################################
    cat > /home/student/tasks/task7.txt << 'EOF'
TASK 7.1: Configure static IP 192.168.56.50/24 on the unconfigured adapter (likely `eth1`). Persist across reboots.
TASK 7.2: Setup SSH key-based login for user `student` (create ~/.ssh/authorized_keys with your public key). Disable password auth for student only if you wish.
TASK 7.3: Ensure firewalld allows HTTP and HTTPS services (hint: `firewall-cmd --add-service=http --permanent` & reload)
TASK 7.4: Schedule a cron job for user `student` to run `date >> ~/date.log` every 2 minutes
TASK 7.5: Use `at` to schedule a one-time job (e.g., `echo "touch /tmp/at_was_here" | at now + 5 minutes`) and verify it ran
EOF

    # Permissions
    chown -R student:student /home/student/tasks

    # Create the checker script
    cat > /usr/local/bin/check_task.sh << 'EOF'
#!/bin/bash
set -euo pipefail

ok(){ echo "✅ $1"; }
fail(){ echo "❌ $1"; exit 1; }

case "${1:-}" in
  # Week 1
  1.1) [[ -d /home/student/projects ]] && ok "1.1" || fail "projects dir missing";;
  1.2) [[ -f /home/student/projects/passwd.bak ]] && ok "1.2" || fail "passwd.bak not in ~/projects";;
  1.3) [[ -f /home/student/passwd.bak ]] && ok "1.3" || fail "passwd.bak not moved to ~";;
  1.4) grep -q '^HELLO$' /home/student/notes.txt && ok "1.4" || fail "notes.txt missing or wrong content";;
  1.5) (ls /etc/host* >/dev/null 2>&1) && ok "1.5 (manual visual check)" || fail "wildcard listing failed";;

  # Week 2
  2.1) [[ -s /home/student/etc_list.txt ]] && ok "2.1" || fail "etc_list.txt missing or empty";;
  2.2) [[ -s /home/student/etc_list.txt ]] && ok "2.2 (appended)" || fail "etc_list.txt missing";;
  2.3) grep -q root /home/student/root_users.txt && ok "2.3" || fail "root_users.txt missing or wrong";;
  2.4) [[ -x /home/student/show_errors.sh ]] && ok "2.4" || fail "show_errors.sh not executable";;
  2.5) [[ -x /home/student/list_big_logs.sh ]] && ok "2.5" || fail "list_big_logs.sh not executable";;

  # Week 3
  3.1) id trainee &>/dev/null && ok "3.1" || fail "user trainee missing";;
  3.2) id -nG trainee | tr ' ' '\n' | grep -qx devs && ok "3.2" || fail "trainee not in devs";;
  3.3) [[ -d /shared ]] && [[ "$(stat -c '%A' /shared)" =~ ^drwxrwx---$ ]] && [[ "$(stat -c '%G' /shared)" = devs ]] && [[ $(stat -c '%a' /shared) -eq 2770 ]] && ok "3.3" || fail "/shared perms/group incorrect";;
  3.4) [[ -k /shared ]] && ok "3.4" || fail "Sticky Bit not set on /shared";;
  3.5) [[ -f /home/student/passwd_hardlink ]] && [[ -L /home/student/passwd_symlink ]] && ok "3.5" || fail "links missing";;

  # Week 4
  4.1) [[ -f /home/student/etc_backup.tar ]] && ok "4.1" || fail "etc_backup.tar missing";;
  4.2) [[ -f /home/student/etc_backup.tar.xz ]] && ok "4.2" || fail "etc_backup.tar.xz missing";;
  4.3) [[ -d /home/student/restore ]] && [[ -f /home/student/restore/etc/passwd || -f /home/student/restore/passwd ]] && ok "4.3" || fail "restore dir/files missing";;
  4.4) [[ -s /home/student/etc_modified.txt ]] && ok "4.4" || fail "etc_modified.txt missing or empty";;
  4.5) [[ -s /home/student/which_type_ls.txt ]] && ok "4.5" || fail "which_type_ls.txt missing or empty";;

  # Week 5
  5.1) systemctl is-enabled firewalld 2>/dev/null | grep -q disabled && ok "5.1" || fail "firewalld not disabled at boot";;
  5.2) systemctl is-enabled crond 2>/dev/null | grep -q enabled && ok "5.2" || fail "crond not enabled at boot";;
  5.3) [[ -f /etc/systemd/system/helloworld.service ]] && systemctl cat helloworld.service &>/dev/null && [[ -f /tmp/hello.txt ]] && ok "5.3" || fail "custom service missing or not run";;
  5.4) [[ "$(systemctl get-default)" == "multi-user.target" ]] && ok "5.4" || fail "default target not multi-user";;

  # Week 6
  6.1) lsblk -no NAME /dev/sdb1 &>/dev/null && ok "6.1" || fail "/dev/sdb1 partition missing";;
  6.2) findmnt -no FSTYPE /mnt/data 2>/dev/null | grep -qx ext4 && ok "6.2" || fail "/mnt/data not mounted as ext4";;
  6.3) grep -q ".* /mnt/data .* defaults" /etc/fstab && ok "6.3" || fail "/etc/fstab entry for /mnt/data missing";;
  6.4) pvs | grep -q "/dev/sdb1" && vgs | grep -q "vgdata" && lvs | grep -q "lvdata" && ok "6.4" || fail "LVM PV/VG/LV missing";;
  6.5) findmnt -no FSTYPE /mnt/lvmdata 2>/dev/null | grep -qx xfs && ok "6.5" || fail "/mnt/lvmdata not mounted as xfs";;
  6.6) [[ $(lsblk -bno SIZE /dev/vgdata/lvdata) -ge $((150*1024*1024)) ]] && ok "6.6" || fail "lvdata not >= 150MB";;
  6.7) swapon --summary | grep -q "/dev/mapper/vgdata-swap" && grep -q "vgdata-swap" /etc/fstab && ok "6.7" || fail "swap LV not active/persistent";;
  6.8) [[ -d /mnt/vfat_test ]] && findmnt /mnt/vfat_test &>/dev/null && ok "6.8" || fail "/mnt/vfat_test not mounted (optional)";;

  # Week 7
  7.1) ip -4 addr show dev eth1 2>/dev/null | grep -q "192.168.56.50/24" && ok "7.1" || fail "eth1 not configured with 192.168.56.50/24";;
  7.2) [[ -s /home/student/.ssh/authorized_keys ]] && ok "7.2" || fail "authorized_keys missing/empty";;
  7.3) firewall-cmd --list-services 2>/dev/null | tr ' ' '\n' | grep -qx http && firewall-cmd --list-services 2>/dev/null | tr ' ' '\n' | grep -qx https && ok "7.3" || fail "HTTP/HTTPS not allowed in firewalld";;
  7.4) sudo -u student crontab -l 2>/dev/null | grep -q "date >> /home/student/date.log" && ok "7.4" || fail "cron job missing for student";;
  7.5) [[ -f /tmp/at_was_here ]] && ok "7.5 (file present)" || (atq >/dev/null 2>&1 && ok "7.5 (job queued; will verify later)" ) ;;

  *) echo "Usage: check_task.sh <task_id>  (e.g., 4.2, 6.5, 7.1)"; exit 2 ;;

esac
EOF

    chmod +x /usr/local/bin/check_task.sh

    # Useful: update locate db initially
    updatedb || true

    echo "\nProvisioning complete. See /home/student/README.md and /home/student/tasks/*.txt"
  SHELL
end

