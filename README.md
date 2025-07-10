# axvma

Virtual Memory Area (VMA) management for file-backed memory mappings.

`axvma` provides safe abstractions for managing memory-mapped regions with file backing. It handles on-demand page loading, region splitting/merging, and overlap resolution for virtual memory areas.

## Core Types

- `VmFile` - Trait for file operations required by VMA management
- `MmapRegion<F>` - Memory-mapped region with file backing
- `VmaManager<F>` - Manager for multiple memory-mapped regions
- `PageSize` - Page alignment configuration

## TODO

the crate will extend to support more about vma management.

## Example

```rust
use axvma::{VmFile, MmapRegion, VmaManager};
use memory_addr::{VirtAddr, VirtAddrRange};
use page_table_multiarch::PageSize;

// Implement VmFile for your file type
#[derive(Clone)]
struct MyFile {
    // Your file implementation
}

impl VmFile for MyFile {
    fn read_at(&self, buf: &mut [u8], offset: u64) -> LinuxResult<usize> {
        // Read file data at offset
    }
    
    fn len(&self) -> LinuxResult<u64> {
        // Return file length
    }
}

// Create VMA manager
let mut vma_manager: VmaManager<MyFile> = VmaManager::new();

// Map a file to virtual memory
let file = MyFile::new();
let vaddr_range = VirtAddrRange::from_start_size(0x1000_0000.into(), 0x10000);
let region = MmapRegion::new(
    vaddr_range,
    file,
    0,  // offset in file
    PageSize::Size4K,
);

// Add region to manager
vma_manager.add_region(region)?;

// Find region by address
let vaddr = VirtAddr::from(0x1000_1000);
if let Some(region) = vma_manager.find_region(vaddr) {
    // Load page data on demand
    let page_data = region.get_buf(vaddr)?;
}

// Remove overlapping regions
let remove_range = VirtAddrRange::from_start_size(0x1000_2000.into(), 0x2000);
let removed_regions = vma_manager.remove_overlapped(remove_range);
```
