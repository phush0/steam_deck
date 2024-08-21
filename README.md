# steam_deck

```
cat << EOF | sudo tee /home/gen_zram.sh
#!/bin/bash
var=$(< /sys/block/zram0/disksize)
swapoff /dev/zram0
echo 1 > /sys/block/zram0/reset
losetup -f /home/test.img
echo /dev/loop0 > /sys/block/zram0/backing_dev
echo $var > /sys/block/zram0/disksize
mkswap /dev/zram0
swapon /dev/zram0 -p 100
echo all > /sys/block/zram0/idle
echo huge_idle > /sys/block/zram0/writeback
EOF
cat << EOF | sudo tee /etc/systemd/system/cpu_performance.service
[Unit]
Description=CPU performance governor
[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower frequency-set -g performance
[Install]
WantedBy=multi-user.target
EOF
cat << EOF | sudo tee /etc/systemd/system/zram_gen.service
[Unit]
Description=ZRAM gen service with loop backend
[Service]
Type=oneshot
ExecStart=/home/gen_zram.sh
[Install]
WantedBy=multi-user.target
EOF
sudo dd if=/dev/zero of=/home/test.img bs=4M count=1024
sudo chmod +x /home/gen_zram.sh
sudo systemctl daemon-reload
sudo systemctl enable cpu_performance.service
sudo systemctl enable zram_gen.service
cat << EOF | sudo tee /etc/tmpfiles.d/mglru.conf
w /sys/kernel/mm/lru_gen/enabled - - - - 7
w /sys/kernel/mm/lru_gen/min_ttl_ms - - - - 0
w /sys/kernel/mm/transparent_hugepage/shmem_enabled - - - - advise
w /sys/kernel/mm/transparent_hugepage/defrag - - - - defer
w /sys/kernel/mm/transparent_hugepage/khugepaged/defrag - - - - 0
w /proc/sys/vm/compaction_proactiveness - - - - 0
w /proc/sys/vm/page_lock_unfairness - - - - 0
EOF
cat << EOF | sudo tee cat /etc/systemd/zram-generator.conf
# -*- mode: sh; indent-tabs-mode: nil; sh-basic-offset: 2; -*-
# vim: et sts=2 sw=2

#  SPDX-License-Identifier: LGPL-2.1+
#
#  Copyright © 2023 Valve Corporation.
#
#  This file is part of holo.

[zram0]
# size at 50% of physical RAM (after discounting memory used by the kernel)
zram-size = ram/2
compression-algorithm = zstd
swap-priority = 100
fs-type = swap
EOF
cat << EOF | sudo tee /etc/security/limits.d/memlock.conf
* hard memlock 2147484
* soft memlock 2147484
EOF
cat << EOF | sudo tee /etc/sysctl.d/99-swappiness.conf
vm.page-cluster=0
vm.watermark_boost_factor=0
vm.watermark_scale_factor=125
vm.swappiness=180
EOF
sudo sed -i -e '/home/s/\bdefaults\b/&,noatime/' /etc/fstab
sudo sed -i 's/\bGRUB_CMDLINE_LINUX_DEFAULT="\b/&mitigations=off nowatchdog nmi_watchdog=0 /' /etc/default/grub
sudo grub-mkconfig -o /boot/efi/EFI/steamos/grub.cfg
```
