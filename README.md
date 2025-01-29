# Parallel Writeback
Performance data with Linux kernel parallel writeback implementation

# Experiment
We run a fio workload, spawning 24 jobs each doing 1GiB of I/O. The total I/O issued from fio is 24 GiB.
The idea is to measure the amount of time taken (and bandwidth) to writeback this data from cache to device. We measure that using iostat.


# Evaluation script
```
mkfs.xfs -f /dev/nvme0n1
mount /dev/nvme0n1 /mnt
sync

echo 3 > /proc/sys/vm/drop_caches
 
fio --directory=/mnt --name=test --bs=4k --iodepth=1024 --rw=randwrite --ioengine=io_uring  --numjobs=24 --size=1G  --direct=0 --eta-interval=1 --eta-newline=1 --group_reporting

umount /mnt
echo 3 > /proc/sys/vm/drop_caches
sync
```

# Iostat command
```
iostat -dxz /dev/nvme0n1 1
```
# Results
We compared parallel writeback with the base case (single-thread writeback). The parallel writeback finishes in 14-15 seconds, while the single-thread writeback takes 28 seconds. Parallel writeback offers higher bandwidth. In fact, parallel writeback completes the same data writeback to the device in roughly half the time of the single-thread approach.

![Image](https://github.com/user-attachments/assets/86a3bcaf-30be-4295-bd2e-7eab4b95cf56)
