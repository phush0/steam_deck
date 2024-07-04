# steam_deck

```
cat << EOF | sudo tee /etc/systemd/system/cpu_performance.service
[Unit]
Description=CPU performance governor
[Service]
Type=oneshot
ExecStart=/usr/bin/cpupower frequency-set -g performance
[Install]
WantedBy=multi-user.target
EOF
sudo systemctl daemon-reload
sudo systemctl enable cpu_performance.service
cat << EOF | sudo tee /etc/tmpfiles.d/mglru.conf
w /sys/kernel/mm/lru_gen/enabled - - - - 7
w /sys/kernel/mm/lru_gen/min_ttl_ms - - - - 0
EOF
cat << EOF | sudo tee /etc/security/limits.d/memlock.conf
* hard memlock 2147484
* soft memlock 2147484
EOF
sudo sed -i -e '/home/s/\bdefaults\b/&,noatime/' /etc/fstab
sudo sed -i 's/\bGRUB_CMDLINE_LINUX_DEFAULT="\b/&mitigations=off nowatchdog nmi_watchdog=0 transparent_hugepage=madvise /' /etc/default/grub
sudo grub-mkconfig -o /boot/efi/EFI/steamos/grub.cfg
```
