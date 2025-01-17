<!--<meta>
{
    "title":"Custom Partitioning and RAID",
    "description":"Setting up CPR (Custom Partitioning & RAID).",
    "tag":["CPR", "Custom RAID", "RAID", "Partition"]
}
</meta>-->


Custom Partitioning & Raid (CPR) is powerful and yet easy to use feature that helps you configure Reserved Hardware instances during deployment.  

_Please note: this feature is not available for on-demand instances.  A reserved device is required.  This is because with reserved devices, our system knows the exact drive scheme to allow such customization.  However, all of our machine types can be converted to reserved hardware, so just reach out to [support@packet.com](mailto:support@packet.com) to arrange a reservation._

### Getting Started

First things first, you should be familiar with the [API calls available for device provisioning](https://www.packet.com/developers/api/devices/), you'll need them!

You should also be aware of our standard disk configurations for each server type.  With a few hardware-specific exceptions, generally speaking, this looks like:

*   __t1.small.x86__:   1 × 80 GB SSD (Boot)
*   __c1.small.x86__:   2 × 120 GB SSD in RAID 1 (Boot)
*   __c1.large.arm__: 1 × 340 GB SSD (Boot)
*   __c2.large.arm__: 1 × 480 GB SSD (Boot)
*   __c1.xlarge.x86__:  2 × 120 GB SSD in RAID 1 (Boot) & 1.6 TB of NVMe Flash
*   __c2.medium.x86__: 960 GB of SSD (2 x 480 GB) (1 for Boot)
*   __c3.medium.x86__: 960 GB of SSD (2 x 480 GB) (1 for Boot)
*   __m1.xlarge.x86__:   6 × 480GB SSD (1 for Boot)
*   __m2.xlarge.x86__: 2 × 120 GB SSD (1 for Boot) & 3.8 TB of NVMe Flash
*   __x1.small.x86__:   1 × 240 GB SSD (Boot)
*   __x2.xlarge.x86__: 1 × 120 GB SSD (Boot) , 2 × 240 GB SSD & 3.8 TB of NVMe Flash
*   __n2.xlarge.x86__: 2 × 240 GB SSD in Hardware RAID 1 (Boot) & 3.8 TB of NVMe Flash
*   __g2.large.x86__: 1 x 150 GB SSD (Boot), 2 x 480 GB SSD 
*   __s1.large.x86__:  1 x 120 GB SSD (Boot), 2 x 480 GB SSD & 12 X 2 TB HDD.

### Using CPR During Provisioning

Let's say you are going to deploy one of your reserved instances. An example [call to the API](https://www.packet.com/developers/api/devices/) might look like this:

```
curl -H "X-Auth-Token: token" -H "Content-Type: application/json" -d '
{
  "facility": "string",
  "plan": "string",
  "hostname": "string",
  "storage": "string",
  "billing_cycle": "string",
  "operating_system": "string",
  "userdata": "string",
  "tags": ["string"]
}'
```
Note: 'storage' and 'string' are where you would specifically state your configuration requirements.

### t1.small.x86 Partition Example

Using a simple t1.small.x86 to start, the following example shows you how to:

* State which disks you want to format.
* How you want these disks formatted.
* What filesystem should be created.
* Where to mount the partition once created.

```
{
  "disks": \[
    {
      "device": "/dev/sda",
      "wipeTable": true,
      "partitions": \[
        {
          "label": "BIOS",
          "number": 1,
          "size": 4096
        },
        {
          "label": "SWAP",
          "number": 2,
          "size": "3993600"
        },
        {
          "label": "ROOT",
          "number": 3,
          "size": 0
        }
      \]
    }
  \],
  "filesystems": \[
    {
      "mount": {
        "device": "/dev/sda3",
        "format": "ext4",
        "point": "/",
        "create": {
          "options": \[
            "-L",
            "ROOT"
          \]
        }
      }
    },
    {
      "mount": {
        "device": "/dev/sda2",
        "format": "swap",
        "point": "none",
        "create": {
          "options": \[
            "-L",
            "SWAP"
          \]
        }
      }
    }
  \]
}
```

#### m1.xlarge example

```
{  
   "disks":\[  
      {  
         "device":"/dev/sda",
         "wipeTable":true,
         "partitions":\[  
            {  
               "label":"BIOS",
               "number":1,
               "size":4096
            },
            {  
               "label":"SWAPA1",
               "number":2,
               "size":"3993600"
            },
            {  
               "label":"ROOTA1",
               "number":3,
               "size":0
            }
         \]
      },
      {  
         "device":"/dev/sdb",
         "wipeTable":true,
         "partitions":\[  
            {  
               "label":"BIOS",
               "number":1,
               "size":4096
            },
            {  
               "label":"SWAPA2",
               "number":2,
               "size":"3993600"
            },
            {  
               "label":"ROOTA2",
               "number":3,
               "size":0
            }
         \]
      }
   \],
   "raid":\[  
      {  
         "devices":\[  
            "/dev/sda2",
            "/dev/sdb2"
         \],
         "level":"1",
         "name":"/dev/md/SWAP"
      },
      {  
         "devices":\[  
            "/dev/sda3",
            "/dev/sdb3"
         \],
         "level":"1",
         "name":"/dev/md/ROOT"
      }
   \],
   "filesystems":\[  
      {  
         "mount":{  
            "device":"/dev/md/ROOT",
            "format":"ext4",
            "point":"/",
            "create":{  
               "options":\[  
                  "-L",
                  "ROOT"
               \]
            }
         }
      },
      {  
         "mount":{  
            "device":"/dev/md/SWAP",
            "format":"swap",
            "point":"none",
            "create":{  
               "options":\[  
                  "-L",
                  "SWAP"
               \]
            }
         }
      }
   \]
}
```

### c1.large.arm Partition Requirement

For the c1.large.arm and c2.medium.x86 servers, it requires a FAT32 boot partition for `/boot/efi` - an example of this particular partition would be:

`format": "vfat", "create":{"options":\[32, "-n", "BIOS"\]}, "point":"/boot/efi`
