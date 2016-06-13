<!-- .slide: data-background="#6C1D5F" -->
<center><div style="width: 75%; height: auto;"><img src="img/xebia.svg"/></div></center>
<br />
<center>
<br />
<br />
**OS Hardening with Packer**

!SLIDE
<!-- .slide: data-background="#6C1D5F" -->
# OS Hardening
Reducing available **vectors of attack** by applying best practices in security mitigations on the operating system level.

!SUB
# OS Hardening

* Not the only tool in your warchest (application/network hardening)
* Implementation is never finished
* Maintenance is essential.
* Do not just implement: audit ( **openscap** )
  * OVAL (Open Vulnerability and Assessment Language )
  * SCAP (Security Content Automation Protocol)

!SUB
# STIG
(Security Technical Implementation Guide)

  * Created by DISA (Defense Informations Agency) as guide for Department of Defense
  * Machine-readable profiles
    * RHEL 7 STIG - SCAP profile ([github.com/OpenSCAP/scap-security-guide](https://github.com/OpenSCAP/scap-security-guide))

!SUB
# CIS Benchmarks
(Center of Internet Security)
* Frequently used to meet FISMA, PCI, HIPAA requirements
* Many different benchmarks:
  * Docker Host
  * RHEL 5/6/7
* Machine readable profiles available through paid membership

!SUB
# Combine rather than choose

* A lot of similarities between CIS and STIG

!SLIDE
<!-- .slide: data-background="#6C1D5F" -->
# CIS RHEL 7

!SUB
# Example Requirements

* GPG must be used for all yum repositories
* Removal of packages: rsh-server, telnet-server, xorg-x11-server-common
* Disable anything you do not specifically use:
  * **xinetd** (services), **avahi** (zeroconf), **dhcp**, **nfs**
* Ensure NTP is active and properly synchronized

!SLIDE
# You should not be able to...
<!-- .slide: data-background="#6C1D5F" -->

!SUB
# ..  have execute rights on files in /tmp

!SUB
# .. by default have world readable files

!SLIDE
<!-- .slide: data-background="#6C1D5F" -->
# Must have multiple mounts

* /var
* /var/log
* /var/log/audit
* /home
* /tmp

!SUB
# Rationale

Prevents resource exhaustion.

Allow (audit) logging to continue while under attack.

!SUB
# cat /dev/zero > zerofile

Check and see how fast you can fill up your disk..

!SUB
# Mitigations

* Create multiple virtual filesystems (files) and mount to apply disk quota (RHEL 6)
* If using XFS, use `xfs_quota` to set quota's per project (folder) (XFS is RHEL 7 default root)
* **Add LVM Logical Volume Manager** (RHEL 6)
  * Span a filesystem over multiple block devices
  * Easy resizing
  * Portability

!SLIDE
<!-- .slide: data-background="#6C1D5F" -->
# Add LVM to image with Packer

!SUB
# Use the `amazon-chroot` provider

Workflow:

1. Creates volume from AMI snapshot
  * No start of machine required
2. Mounts the Volume to host on AWS
  * Allows direct manipulation of the block device
3. Mounts several `chroot_mounts` (/dev) to
4. Provisioners are started
  * Each provisioner is wrapped by a `chroot` command
5. Unmounts all `chroot_mounts` and detaches volume

!SUB
# Challenges

1. Packer wants to always unmount the everything it mounted at the beginning
2. No way to access {{ .Device }} in provisioner (/dev/xvda1)

!SUB
# Solution

* Use `shell-local` provisioner to execute a script on host: `lvm.sh`
  1. Unmount
  2. Backup the device with `tar`
  3. Partition (Boot + LVM)
  4. Create LVM Volume group
    * Create LVM Logical volumes (var,varlog,varlogaudit)
    * Create filesystems
  5. Mount new filesystems
  6. Restore

!SUB
# Tricking Packer..
<center><div style="width: 75%; height: auto;"><img src="img/flow.svg"/></div></center>

!SLIDE
<!-- .slide: data-background="#6C1D5F" -->
# Conclusion

Able to apply filesystems/paritions automatically, no reboots.

**Caveats**:

* Create an image hierarchy
* Use amazon-ebs for the child images
  * Has EBS encryption
  * Better support
  * Faster debugging
  * LVM only needs to be added __once__ per source image

!SLIDE
<!-- .slide: data-background="#6C1D5F" -->
<center>![HashiConf](img/hashiconf.png)</center>
Win a GoPro by participating in the HashiContest at the Xebia booth
