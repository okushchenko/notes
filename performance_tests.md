Tests
=====

##### CPU test 1
```sh
sysbench --test=cpu --cpu-max-prime=50000 --num-threads=2 run
```

1. Azure A2 instance (Intel Xeon E2660 2.2GHz) 121.1436
2. Local Openstack (Intel Core i7 9xx 2.7GHz ) 61.9038
3. Local laptop (Intel Core i5 3230M 2.60GHz) 51.6740
4. Azure A2 instance (Intel Xeon E5-2673 v3 2.40GHz) 123.8244

##### CPU test 2
```sh
sysbench --test=cpu --cpu-max-prime=40000 --num-threads=1 run
```

1. Azure A2 instance (Intel Xeon E2660 2.2GHz) 173.4591
2. Azure DS1 instance (Intel Xeon E2660 2.2GHz) 103.1595
3. Local Openstack m1.medium (Intel Core i7 9xx 2.7GHz ) 62.8479
4. Local laptop (Intel Core i5 3230M 2.60GHz) 70.2707s
5. AWS t2.micro () 356.0369
6. Azure A2 instance (Intel Xeon E5-2673 v3 2.40GHz) 185.7479
7. Azure A1 instance (Intel Xeon E5-2673 v3 2.40GHz) 183.2495
8. Azure A0 instance (Intel Xeon E5-2673 v3 2.40GHz) 354.7480

##### Disk io test
```
sysbench --test=fileio --file-total-size=8GB --file-test-mode=rndrw --file-block-size=4K --max-time=300 --max-requests=1000000 --num-threads=1 --init-rng=on --file-num=16 --file-extra-flags=direct --file-fsync-freq=0 --file-io-mode=async run
```

1. Azure A0 (Intel Xeon E5-2673 v3 2.40GHz) ephemeral 32: 213.05 16: 196.44 volume 477.08
system 16: 448.31 ephemeral 16: 359.19 
3. Azure A1 system 502.81 ephemeral 406.89 2*raid0 989.49
4. Azure DS1 ephemeral 4279.90
laptop local hdd avg: 133.74 max: 466.35
5. Openstack local hdd avg: 134.33, max: 547.17
6. Opestack ceph hdd avg: 145.41, max: 151.23

##### Installation process of sysbench for rhel based OS
```
sudo yum -y install epel-release && sudo yum -y install sysbench
```

##### The old disk io test
```
sysbench --test=fileio --file-total-size=8GB --file-test-mode=rndrw --file-block-size=4K --max-time=300 --max-requests=10000000 --num-threads=16 --init-rng=on --file-num=16 --file-extra-flags=direct --file-fsync-freq=0 prepare
sysbench --test=fileio --file-total-size=8GB --file-test-mode=rndrw --file-block-size=4K --max-time=300 --max-requests=10000000 --num-threads=16 --init-rng=on --file-num=16 --file-extra-flags=direct --file-fsync-freq=0 run
sysbench --test=fileio --file-total-size=8GB --file-test-mode=rndrw --file-block-size=4K --max-time=300 --max-requests=10000000 --num-threads=16 --init-rng=on --file-num=16 --file-extra-flags=direct --file-fsync-freq=0 cleanup
```

##### TODO: test disks using fio
```sh
# Random read/write
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=1G --readwrite=randrw --rwmixread=75
# Random read
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=1G --readwrite=randread
# Random write
fio --randrepeat=1 --ioengine=libaio --direct=1 --gtod_reduce=1 --name=test --filename=test --bs=4k --iodepth=64 --size=1G --readwrite=randwrite
```
