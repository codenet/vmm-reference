# Interrupts (x86_64 architecture)

## Local Advanced Programmable Interrupt Controller (LAPIC)
LAPIC is a memory-mapped device that is used to send interrupts to the VCPUs. It is used to send interrupts to the VCPUs and to receive interrupts from the VCPUs. It is also used to send inter-processor interrupts (IPIs) to the VCPUs.

There are five delivery modes for interrupts:
* Fixed: The interrupt is sent to a specific VCPU.
* SMI: System Management Interrupt. It is used to notify the hypervisor that a VCPU has entered SMM.
* NMI: Non-Maskable Interrupt. It is used to notify the hypervisor that a VCPU has entered NMI.
* INIT: It is used to initialize the VCPU.
* ExtINT: External Interrupt. Causes the processor to respond to the interrupt as if the interrupt originated in an 
externally connected 

Refer to the [LAPIC documentation ](https://www.intel.com/content/dam/support/us/en/documents/processors/pentium4/sb/25366821.pdf) in intel's manual for more details. Figure 8.8 on Vol. 3A 8-17 shows the local vector table structure.

The function `set_klapic_delivery_mode` sets the delivery mode from available delivery modes: `Fixed`, `SMI`, `NMI`, `INIT`, `ExtINT`. 

There are two helper functions defined to read and write to the LAPIC registers. The `read_le_i32` function reads from the LAPIC register at the given offset and returns the value read. 
```rust
pub fn get_klapic_reg(klapic: &kvm_lapic_state, reg_offset: usize) -> Result<i32> {
    let range = reg_offset..reg_offset + 4;
    let reg = klapic.regs.get(range).ok_or(Error::InvalidRegisterOffset)?;
    Ok(read_le_i32(reg))
}
```

The `write_le_i32` function writes the given value to the LAPIC register at the given offset.
```rust
pub fn set_klapic_reg(klapic: &mut kvm_lapic_state, reg_offset: usize, value: i32) -> Result<()> {
    // The value that we are setting is a u32, which needs 4 bytes of space.
    // We're here creating a range that can fit the whole value.
    let range = reg_offset..reg_offset + 4;
    let reg = klapic
        .regs
        .get_mut(range)
        .ok_or(Error::InvalidRegisterOffset)?;
    write_le_i32(reg, value);
    Ok(())
}
```
The helper functions `get_klapic_reg` uses the `read_le_i32` function to read from the LAPIC register at the given offset and returns the value read. The values are stored as `i32` which requires 4 `i8` values. Hence, the range is defined as `reg_offset..reg_offset + 4`. 

```rust
pub fn get_klapic_reg(klapic: &kvm_lapic_state, reg_offset: usize) -> Result<i32> {
    let range = reg_offset..reg_offset + 4;
    let reg = klapic.regs.get(range).ok_or(Error::InvalidRegisterOffset)?;
    Ok(read_le_i32(reg))
}
```

Similary `set_klapic_reg` uses the `write_le_i32` function to write the given value to the LAPIC register at the range offset, offset+4. 
```rust
pub fn set_klapic_reg(klapic: &mut kvm_lapic_state, reg_offset: usize, value: i32) -> Result<()> {
    // The value that we are setting is a u32, which needs 4 bytes of space.
    // We're here creating a range that can fit the whole value.
    let range = reg_offset..reg_offset + 4;
    let reg = klapic
        .regs
        .get_mut(range)
        .ok_or(Error::InvalidRegisterOffset)?;
    write_le_i32(reg, value);
    Ok(())
}
```
## Usage in code

While creating a new ```kvmVcpu```, LAPICs for the vcpu is created and initialized with ```configure_lapic``` function. The function ```configure_lapic``` initializes the LAPICs with the following values: 
1. `LAPIC0`: external interrupts 
2. `LAPIC1`: NMI.

```rust
    fn configure_lapic(&self) -> Result<()> {
        let mut klapic: kvm_lapic_state = self.vcpu_fd.get_lapic().map_err(Error::KvmIoctl)?;
        set_klapic_delivery_mode(&mut klapic, APIC_LVT0_REG_OFFSET, DeliveryMode::ExtINT).unwrap();
        set_klapic_delivery_mode(&mut klapic, APIC_LVT1_REG_OFFSET, DeliveryMode::NMI).unwrap();
        self.vcpu_fd.set_lapic(&klapic).map_err(Error::KvmIoctl)
    }
```
Here `APIC_LVT0_REG_OFFSET` is the register offset corresponding to the APIC Local Vector Table for LINT0 and set to `0x350` and `APIC_LVT1_REG_OFFSET` is the register offset corresponding to the APIC Local Vector Table for LINT1 and set to `0x360`.
The function first gets the `kvm_lapic_state` from the vcpu_fd and then sets the delivery mode for the two LAPICs and then sets the `kvm_lapic_state` back to the vcpu_fd, where `kvm_lapic_state` is a array of 1024 `i8` values.

Note: Configuring LAPICs should be done after creating `irqchip`. `irqchip` creates the virtual IOAPIC and virtual PIC and sets up future vCPUs for local APIC. 


## Unit Testing
1. Range testing: 
   The `kvm_lapic_state` is array of 1024 `i8` values. During `set_klapic_reg`, as the `value` is of `u32` type, `value` is stored across the range `reg_offset`...`reg_offset`+4. Hence, if the offset is greater than 1020, the `set_klapic_reg` function will fail with `InvalidRegisterOffset` error.

2. Delivery Mode enumeration:
   enum type `DeliveryMode` is tested for corresponding values.


* Note: This document only covers the x86_64 architecture. For aaarch64, the interrupt controller is called GIC (Generic Interrupt Controller). 
