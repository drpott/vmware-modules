# vmware-modules
compiling modules on new kernels

$ sudo dnf install kernel-devel kernel-headers gcc gcc-c++ make git

Installation
This repository tracks patches needed to build VMware (Player and Workstation) host modules against recent kernels.

For Example, I would like to Patch Workstation:

wget https://github.com/mkubecek/vmware-host-modules/archive/workstation-x.y.z.tar.gz
tar -xzf workstation-x.y.z.tar.gz
cd vmware-host-modules-workstation-x.y.z
make
sudo make install

Fixing SSL lib issues:

$ cd /etc/ld.so.conf.d/
$ sudo touch vmware-authdlauncher.conf
$ ls
libiscsi-x86_64.conf                 tix-x86_64.conf
pipewire-jack-x86_64.conf      vmware-authdlauncher.conf
$ cat /etc/ld.so.conf.d/vmware-authdlauncher.conf
#linking libssl and libcrypto
/usr/lib/vmware/lib/libssl.so.1.0.2
/usr/lib/vmware/lib/libcryto.so.1.0.2
$ sudo ldconfig

To correct the issue with secure boot enabled:
Generate a key pair using the openssl to sign vmmon and vmnet modules:

$openssl req -new -x509 -newkey rsa:2048 -keyout MOK.priv -outform DER -out MOK.der -nodes -days 36500 -subj "/CN=VMware/"

Replace MOK with the name of the file you want for the key.
 
Sign the modules using the generated key by running these commands:

$sudo /usr/src/linux-headers-`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmmon)

$sudo /usr/src/linux-headers-`uname -r`/scripts/sign-file sha256 ./MOK.priv ./MOK.der $(modinfo -n vmnet)
Import the public key to the system's MOK list by running this command:

$mokutil --import MOK.der
 
Confirm a password for this MOK enrollment request.
Reboot your machine. Follow the instructions to complete the enrollment from the UEFI consol.
