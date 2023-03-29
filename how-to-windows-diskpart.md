
# Windows Disk Utility CLI

| Command | Description |
| ----------- | ----------- |
| diskpart | runs disk partition program in CLI |
| help | lists all the commands for diskpart |
| list disk | lists all the connected disks |
| select disk [#] | lists all the connected disks |
| list volume | lists all the connected disks |
| select volume [#] | lists all the connected disks |
| create partition primary size=[#in_MB] | creates a partition and sets a partition type |
| clean | erases all contents on that disk/volume |
| format fs=[file_system_type] quick | formats a volume to a file system |

### Quick Start Guide

`list disk`

`select disk [#]`

`clean`

`convert [gpt/mbr]`

`create partition primary size=[full_disk_size]`

`format fs=exfat label="[disk_name]" quick`

`assign letter=[drive_letter]`

### Windows Disk Utility How-to

- Delete a volume:
```sh
list volume
select volume [#]
delete volume
```
> Note: you cannot delete system volumes

- Convert MBR to GPT:

```sh
list disk
select disk [#]
```

> **Important Note**: this next command will *completely wipe the drive, erasing all files*. When working with new drives, it is helpful to first run the `clean` command. Also, Windows will not allow you to add a volume to a drive unless you have first converted the drive `convert <mbr or gpt>`.

```sh
clean
convert [gpt/mbr]
```

- Create a partition: 
```sh 
list disk
select disk [#]
create partition primary size=[#_in_MB]
```

> Note: if you want to create a different types of partition, simply replace `primary` with the desired partition type such as:  `efi`, `extended`, `logical`, or `msr`

> Note on HD Sizing: If you are unsure how many MB to type in as a number when you partition the drive, you can use view it in the drive properties > volumes tab. Just copy and paste the number you see when partitioning a drive. <br>![alt_text](./images/volume-properties.jpg)

> **Troubleshooting MB Size Issue**: You may get a message saying: "No usable free extent could be found. It may be that there is insufficient free space to create a partition at the specified size and offset." In my experience, I had to subtract exactly 18MB from the total volume size listed in the drive properties to get it to work.
>> If that doesn't work for you, try this method:
>>
>> * Open Disk Management (Windows button + search "disk" + click on "create and format hard disk partitions"
>> * Right-click the drive you want to partition (it helps to `clean` and `convert` the drive you want to partition in diskpart first).
>> * Click "New Simple Volume" > Hit next on the wizard GUI > take note of the value in "Simple volume size in MB"
>> <br> ![alt_text](./images/simple-vol-wiz.jpg)
>> * This value is the *max amount* you can partition. Enter this value in the the `diskpart` CLI `create primary partition =size <that_value>`


- Format partition: `list volume`, if the file system (Fs) reads as "RAW" then you must first format the volume to a useable file system first. To do so simply type:
```sh
select volume [#]
format fs=exfat quick
```

> Note: you can format to other fs by changing `exfat` to: `ntfs`, `fat`, `fat32` <br>
> Note: diskpart cannot partition `fat32` volumes greater than 32GB


- Extend a Partition (if you want to expand the size of a partition and have extra space still available on the same drive):
```sh
list volume
select volume [#]
extend size=[#_in_MB]
```

- Shrink a Partition (if you want to shrink the size of a partition):
```sh
list volume
select volume [#]
shrink desired=[#_in_MB]
```

- Assign drive letter:
```sh
select disk [#]
assign letter=[drive_letter]
```

- Mark Partition as Active:
```sh
list volume
select volume [#]
active
```

### Increase Available HD Storage

- Once you are in, you will notice two separate storage units inside of the node created by default (local-lvm and local). It turns out you don't need the local-lvm. So we can delete it to increase your storage capacity. Navigate to `Storage` and then select the "local-lvm" storage unit, and then click `Remove` to delete, and it will be gone.

> Note: We still have some work to do in the shell to fully remove "local-lvm" and increase the storage for "local".

- Click on the node (underneath the `Datacenter` dropdown under the server view_.
- Next, click on `>_ Shell` to enter the command line needed now.

-This will remove the local-lvm:

```sh
lvremove /dev/pve/data
ENTER
y
```

-Now to resize the storage to the maximum size available on the target HD:

```sh
lvresize -l +100%FREE /dev/pve/root
ENTER
```

-And, one last command to finish everything off:

```sh
resize2fs /dev/mapper/pve-root
ENTER
```

-Now if you check the summary, you will see the "local" storage unit of your ProxMox node has been increased fully.
