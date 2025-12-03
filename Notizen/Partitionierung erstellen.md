# Partitionierung für LFS

1. **Swap-Partition erstellen**:
    
    ```bash
    sudo mkswap -L swap-partition /dev/sda4
    swapon /dev/sda4
    ```
    
2. **LFS-Root-Partition**:
    
    ```bash
    sudo mkfs.ext4 -L LFS-root /dev/sda5
    sudo mkdir -pv /mnt/lfs
    chown root:root $LFS
    chmod 755 $LFS
    sudo mount -v -t ext4 /dev/sda5 /mnt/lfs
    ```
    
3. **Automatisches Mounten** (in `/etc/fstab`):
    
    ```
    /dev/sda5  /mnt/lfs  ext4  defaults  0  1
    ```
    