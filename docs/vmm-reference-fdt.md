# [Flatenned Device Tree](../src/arch/src/lib.rs)

## Description
VMM keeps track of the virtio devices  in a data structure called Flatenned Device Tree. FDT as the name suggests is a Tree where each node is a device or collection of devices. There can be different types of nodes in the FDT which can be identified using the attributes of the node. Each node contains a key-value pair mapping which determine the type of the node and configuration of that node.
This arrangement provides an abstracts out the hardware specifications from the code.

## Implementation
The implementation of the FDT in vmm-reference is done using rust crate vm_fdt and vm_memory so are the following import statements.
```rs
pub use vm_fdt::{Error as FdtError, FdtWriter};
use vm_memory::{guest_memory::Error as GuestMemoryError, Bytes, GuestAddress, GuestMemory};
```
Then some constants are set up which include the max size of the FDT, start of DRAM in physical address space, base for MMIO and AXI ports and set up the virtual Generic Interrupt Controller.\
Then the Error handler is set up with default error handler of both vm_memory and vm_fdt crates in addition to a error handler for cases of missing required configuration.
```rs
#[derive(Debug)]
pub enum Error {
    Fdt(FdtError),
    Memory(GuestMemoryError),
    MissingRequiredConfig(String),
}
```
The structure of the FdtBuilder is defined as 
```rs
#[derive(Default)]
pub struct FdtBuilder {
    cmdline: Option<String>,
    mem_size: Option<u64>,
    num_vcpus: Option<u32>,
    serial_console: Option<(u64, u64)>,
    rtc: Option<(u64, u64)>,
    virtio_devices: Vec<DeviceInfo>,
}
```
where each device in virtio_devices vector is in the form of DeviceInfo struct defined as follows
```rs
struct DeviceInfo {
    addr: u64,
    size: u64,
    irq: u32,
}
```

FdtBuilder implementation provides with methods to create a new node in te FDT and then configure that node. It also provides methods to query number of virtio devices in that node and add or a device to them.