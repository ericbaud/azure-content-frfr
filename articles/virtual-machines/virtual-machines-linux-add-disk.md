<properties
	pageTitle="Ajout d’un disque à une machine virtuelle Linux | Microsoft Azure"
	description="Découvrir comment ajouter un disque persistant à votre machine virtuelle Linux"
	keywords="machine virtuelle Linux, ajouter un disque de ressources"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="rickstercdn"
	manager="timlt"
	editor="tysonn"
	tags="azure-resource-manager" />

<tags
	ms.service="virtual-machines-linux"
	ms.topic="get-started-article"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.date="03/01/2016"
	ms.author="rclaus"/>

# Ajouter un disque à une machine virtuelle Linux

Cet article explique comment attacher un disque persistant à votre machine virtuelle afin de conserver vos données, et ce, même si votre machine virtuelle est remise en service en raison d’une opération de maintenance ou de redimensionnement. Pour ajouter un disque, vous aurez besoin de l’[interface de ligne de commande Azure](../xplat-cli-install.md) en mode Resource Manager (`azure config mode arm`).

## Commandes rapides

```
# In the following command examples, replace the values between &lt; and &gt; with the values from your own environment.

rick@ubuntu$ azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>
```

## Attacher un disque

La procédure d’attachement d’un nouveau disque est rapide. Il vous suffit de saisir `azure vm disk attach-new <myuniquegroupname> <myuniquevmname> <size-in-GB>` afin de créer un nouveau disque (Go) et l’associer à votre machine virtuelle. Le résultat suivant doit s’afficher :

	azure vm disk attach-new myuniquegroupname myuniquevmname 5
	info:    Executing command vm disk attach-new
	+ Looking up the VM "myuniquevmname"
	info:    New data disk location: https://cliexxx.blob.core.windows.net/vhds/myuniquevmname-20150526-0xxxxxxx43.vhd
	+ Updating VM "myuniquevmname"
	info:    vm disk attach-new command OK


## Se connecter à la machine virtuelle Linux afin de monter le nouveau disque

> [AZURE.NOTE] Dans cette rubrique, la connexion à la machine virtuelle est effectuée à l’aide de noms d’utilisateurs et de mots de passe. Pour utiliser des paires de clés publiques et privées pour interagir avec votre machine virtuelle, consultez la page [Utilisation de SSH avec Linux sur Azure](virtual-machines-linux-ssh-from-linux.md). Vous pouvez modifier la connectivité **SSH** des machines virtuelles créées avec la commande `azure vm quick-create` à l’aide de la commande `azure vm reset-access` pour complètement réinitialiser l’accès **SSH**, ajouter ou supprimer des utilisateurs ou ajouter des fichiers de clés publiques pour sécuriser l’accès.

Vous devrez exécuter SSH dans votre machine virtuelle Azure afin de partitionner, de formater et de monter votre nouveau disque pour que la machine virtuelle Linux puisse l’utiliser. Si vous ne maîtrisez pas la connexion via **ssh**, la commande prend la forme `ssh <username>@<FQDNofAzureVM> -p <the ssh port>` et ressemble à ceci :

	ssh ops@myuni-westu-1432328437727-pip.westus.cloudapp.azure.com -p 22
	The authenticity of host 'myuni-westu-1432328437727-pip.westus.cloudapp.azure.com (191.239.51.1)' can't be established.
	ECDSA key fingerprint is bx:xx:xx:xx:xx:xx:xx:xx:xx:x:x:x:x:x:x:xx.
	Are you sure you want to continue connecting (yes/no)? yes
	Warning: Permanently added 'myuni-westu-1432328437727-pip.westus.cloudapp.azure.com,191.239.51.1' (ECDSA) to the list of known hosts.
	ops@myuni-westu-1432328437727-pip.westus.cloudapp.azure.com's password:
	Welcome to Ubuntu 14.04.2 LTS (GNU/Linux 3.16.0-37-generic x86_64)

	 * Documentation:  https://help.ubuntu.com/

	  System information as of Fri May 22 21:02:32 UTC 2015

	  System load: 0.37              Memory usage: 2%   Processes:       207
	  Usage of /:  41.4% of 1.94GB   Swap usage:   0%   Users logged in: 0

	  Graph this data and manage this system at:
	    https://landscape.canonical.com/

	  Get cloud support with Ubuntu Advantage Cloud Guest:
	    http://www.ubuntu.com/business/services/cloud

	0 packages can be updated.
	0 updates are security updates.

	The programs included with the Ubuntu system are free software;
	the exact distribution terms for each program are described in the
	individual files in /usr/share/doc/*/copyright.

	Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
	applicable law.

	ops@myuniquevmname:~$

Maintenant que vous êtes connecté à votre machine virtuelle, vous êtes prêt à attacher un disque. Recherchez tout d’abord le disque à l’aide de `dmesg | grep SCSI`. (La méthode que vous utilisez pour découvrir les capacités du nouveau disque peut varier). Le résultat est alors le suivant :

	dmesg | grep SCSI
	[    0.294784] SCSI subsystem initialized
	[    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
	[    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
	[    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
	[ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk

et dans le scénario évoqué dans cette rubrique, le disque `sdc` est celui que nous voulons. Partitionnons maintenant le disque avec `sudo fdisk /dev/sdc`, en supposant que dans votre cas, il s’agissait d’un disque `sdc` et utilisons-le en tant que disque principal sur la partition 1, en acceptant les autres valeurs par défaut.

	sudo fdisk /dev/sdc
	Device contains neither a valid DOS partition table, nor Sun, SGI or OSF disklabel
	Building a new DOS disklabel with disk identifier 0x2a59b123.
	Changes will remain in memory only, until you decide to write them.
	After that, of course, the previous content won't be recoverable.

	Warning: invalid flag 0x0000 of partition table 4 will be corrected by w(rite)

	Command (m for help): n
	Partition type:
	   p   primary (0 primary, 0 extended, 4 free)
	   e   extended
	Select (default p): p
	Partition number (1-4, default 1): 1
	First sector (2048-10485759, default 2048):
	Using default value 2048
	Last sector, +sectors or +size{K,M,G} (2048-10485759, default 10485759):
	Using default value 10485759

Créez la partition en saisissant `p` lors de l’invite :

	Command (m for help): p

	Disk /dev/sdc: 5368 MB, 5368709120 bytes
	255 heads, 63 sectors/track, 652 cylinders, total 10485760 sectors
	Units = sectors of 1 * 512 = 512 bytes
	Sector size (logical/physical): 512 bytes / 512 bytes
	I/O size (minimum/optimal): 512 bytes / 512 bytes
	Disk identifier: 0x2a59b123

	   Device Boot      Start         End      Blocks   Id  System
	/dev/sdc1            2048    10485759     5241856   83  Linux

	Command (m for help): w
	The partition table has been altered!

	Calling ioctl() to re-read partition table.
	Syncing disks.

Écrivez un système de fichiers sur la partition à l’aide de la commande **mkfs**, en spécifiant le type de votre système de fichier et le nom de l’appareil. Dans cette rubrique, nous utilisons `ext4` et `/dev/sdc1` (voir ci-dessus) :

	sudo mkfs -t ext4 /dev/sdc1
	mke2fs 1.42.9 (4-Feb-2014)
	Discarding device blocks: done
	Filesystem label=
	OS type: Linux
	Block size=4096 (log=2)
	Fragment size=4096 (log=2)
	Stride=0 blocks, Stripe width=0 blocks
	327680 inodes, 1310464 blocks
	65523 blocks (5.00%) reserved for the super user
	First data block=0
	Maximum filesystem blocks=1342177280
	40 block groups
	32768 blocks per group, 32768 fragments per group
	8192 inodes per group
	Superblock backups stored on blocks:
		32768, 98304, 163840, 229376, 294912, 819200, 884736
	Allocating group tables: done
	Writing inode tables: done
	Creating journal (32768 blocks): done
	Writing superblocks and filesystem accounting information: done

Nous créons désormais un répertoire afin de monter le système de fichiers à l’aide de `mkdir` :

	sudo mkdir /datadrive

Vous montez le répertoire à l’aide de `mount` :

	sudo mount /dev/sdc1 /datadrive

Le disque de données est désormais utilisable en tant que `/datadrive`.

	ls
	bin   datadrive  etc   initrd.img  lib64       media  opt   root  sbin  sys  usr  vmlinuz
	boot  dev        home  lib         lost+found  mnt    proc  run   srv   tmp  var

> [AZURE.NOTE] Vous pouvez également vous connecter à votre machine virtuelle Linux à l'aide d'une clé SSH pour l'identification. Pour en savoir plus, consultez la page [Utilisation de SSH avec Linux sur Microsoft Azure](virtual-machines-linux-ssh-from-linux.md).

## Étapes suivantes

- N’oubliez pas que votre nouveau disque n’est généralement pas disponible sur votre machine virtuelle en cas de redémarrage, sauf si vous écrivez les informations sur votre fichier [fstab](http://en.wikipedia.org/wiki/Fstab).
- Passez en revue les recommandations visant à [optimiser les performances de votre machine virtuelle Linux](virtual-machines-linux-optimization.md) pour garantir que votre machine virtuelle Linux est correctement configurée.
- Développez votre capacité de stockage en ajoutant des disques supplémentaires et [configurez RAID](virtual-machines-linux-configure-raid.md) pour augmenter les performances.

<!---HONumber=AcomDC_0406_2016-->