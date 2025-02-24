# Device Passthrough

Device passthrough for TDX guests requires updates to the host kernel
and QEMU. The patch sets below track the bleeding edge state of
pre-release Kernel and QEMU development branches. They are provided here
for preview and test purposes only, and may be update frequently. Any
feedback or issues should be reported as a reply to the latest upstream
posting of the given patch. See the linux-coco, linux-kvm, and qemu-devel
archives on lore.kernel.org for the most recent public posting of these patches. Note that
there some temporary workarounds and shortcuts included while formal
replacements are in development.

Wait for these patches to be accepted by their respective upstream
projects, Next, wait for new releases of those upstream project versions to be picked up
by your chosen Linux distribution provider before using them for any production
use case.

## Repo Layout

* `tdx-kvm` - Contains an mbox file with host side TDX patches for KVM. This can be applied using:
  ```
  git am --empty=drop tdx-kvm/tdx_kvm_baseline_<sha>.mbox
  ```
  The baseline (**as noted in the filename above indicated by sha**) for these patches is the
`kvm-coco-queue` branch in the [KVM
repo](https://git.kernel.org/pub/scm/virt/kvm/kvm.git/). Since this is a
rebasing branch, the commit is not guaranteed to be present in kvm.git. A
snapshot of the older version of kvm-coco-queue can be found, for
example,
[here](https://github.com/intel/tdx/releases/tag/kvm-coco-queue-snapshot%2Fkvm-coco-queue-snapshot-20250220).
It is possible that you migh encounter git am failure on the kselftests patches, just skip
those patches because the error is caused by empty patches(in case --empty=drop does
not work)

* `tdx-qemu` - Contains an mbox file with TDX patches for QEMU. This can be applied using:
  ```
  git am --empty=drop tdx-qemu/tdx_qemu_baseline_<sha>.mbox
  ```
  The baseline for these should be found in [qemu.git](https://git.qemu.org/git/qemu.git).

* `tdx-edk2` - The EDK2 enabling we need so far is already in an upstream tag. This simply contains a file with the stable tag name. from the [EDK2 repo](https://github.com/tianocore/edk2.git)

## Building the components

* `tdx-kvm`:
  * Config options:
    ```
    Enables:
    CONFIG_INTEL_TDX_HOST
    CONFIG_KVM
    CONFIG_KVM_INTEL
    CONFIG_TDX_GUEST_DRIVER
    CONFIG_HYPERV
    CONFIG_IOMMUFD_DRIVER
    CONFIG_VFIO_DEVICE_CDEV
    CONFIG_IOMMUFD

    Disables:
    CONFIG_KEXEC
    CONFIG_CRASH_DUMP
    ```
    
  * Build and install the kernel on the host machine. Add module options:
    ```
    # echo "options kvm_intel tdx=on" > /etc/modprobe.d/tdx.conf
    # grubby --update-kernel=ALL --args="console=ttyS0,115200 kvm_intel.tdx=on nohibernate"
    ```

* `tdx-qemu`: 
  * Config options:
    ```
    ./configure --enable-kvm --target-list=x86_64-softmmu
    ```
  * Build and install qemu as usual

* `tdx-edk2`:
  * Clone the [EDK2 repo](https://github.com/tianocore/edk2) and checkout the tag as noted in this repo.
  * Build the OVMF image:
    ```
    rm -rf Build
    make -C BaseTools
    . edksetup.sh
    cat <<-EOF > Conf/target.txt
    	ACTIVE_PLATFORM = OvmfPkg/OvmfPkgX64.dsc
    	TARGET = DEBUG
    	TARGET_ARCH = X64
    	TOOL_CHAIN_CONF = Conf/tools_def.txt
    	TOOL_CHAIN_TAG = GCC6
    	BUILD_RULE_CONF = Conf/build_rule.txt
    	MAX_CONCURRENT_THREAD_NUMBER = $(nproc)
    EOF
    build clean
    build

    if [ ! -f Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd ]; then
    	echo "Build failed, OVMF.fd not found"
    	exit 1
    fi

    cp Build/OvmfX64/DEBUG_GCC5/FV/OVMF.fd ./OVMF.fd
    ```

* Additional notes on Host and Guest setup and booting can be found in the [wiki](https://github.com/intel/tdx-linux/wiki/Instruction-to-set-up-TDX-host-and-guest). One thing not mentioned on the wiki: make sure TDX guest has **"clearcpuid=mtrr"** in its kernel command line.


## Specific notes for device passthrough to TD
* `Host and Guest Kernel`
  * Prepare host, guest kernel and qemu according to the above info.
  * For guest kernel to support SPDM session establishment make sure the following configuration options are enabled.
    ```
    CONFIG_CRYPTO_ECC=y
    CONFIG_CRYPTO_ECDH=y
    CONFIG_CRYPTO_ECDSA=y
    CONFIG_CRYPTO_ECRDSA=y
    ```
* `Boot TD with GPU passthrough`
  * To passthrough the GPU card to TD, say 4b:00.0, in addition to the normal cmdline to boot a TD, add the following additional cmdline to qemu (ensure to bind the device to a VFIO driver before initiating the virtual machine)
    ```
    -object iommufd,id=iommufd0 \
    -device pcie-root-port,id=pci.1,bus=pcie.0 \
    -device vfio-pci,host=4b:00.0,bus=pci.1,iommufd=iommufd0 -fw_cfg name=opt/ovmf/X-PciMmio64,string=262144
    ```
