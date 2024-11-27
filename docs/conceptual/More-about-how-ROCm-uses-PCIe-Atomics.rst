.. meta::
   :description: How ROCm uses PCIe atomics
   :keywords: PCIe, PCIe atomics, atomics, AMD, ROCm

*****************************************************************************
How ROCm uses PCIe atomics
*****************************************************************************
AMD ROCm is an extension of the Heterogeneous System Architecture (HSA). To meet the requirements of an HSA-compliant system, ROCm must support queuing models, memory models, and signaling and synchronization protocols. To learn more about the requirements of an HSA-compliant system, see the 
`HSA Platform System Architecture Specification <http://hsafoundation.com/wp-content/uploads/2021/02/HSA-SysArch-1.2.pdf>`_.

ROCm needs platform atomics to perform memory operations like queuing, signaling, and synchronization across multiple CPU and GPU agents. Platform atomics ensures that atomic operations run synchronously without any interruptions or conflicts across multiple shared resources. Similarly, memory alignment and synchronization of processing data with shared memory locations and Input/Output (I/O) devices is needed. ROCm uses atomic Read-Modify-Write (RMW) transactions that extend inter-processor synchronization mechanisms to I/O devices starting from Peripheral Component Interconnect Express 3.0 (PCIe™ 3.0). It supports the defined set of HSA capabilities needed for queuing and signaling memory operations. Learn how platform atomics are processed in ROCm and supported by PCIe™ 3.0.

Usage of platform atomics by ROCm 
=====================================

Platform atomics are the set of atomic operations that perform RMW actions across multiple processors, devices, and memory locations synchronously without any interruption. An atomic operation is a sequence of computing instructions executed as a single, indivisible unit. These instructions are completed in their entirety without any interruptions or would not be executed at all. These operations support 32-bit and 64-bit address formats.

Platform atomics are used by ROCm to:

* Update the HSA queue's ``read_dispatch_id``. A 64-bit atomic add operation is used by the command processor on the
  GPU agent. It updates the packet ID it processed.
* Update the HSA queue's ``write_dispatch_id``. A 64-bit atomic add operation is used by the CPU and GPU agent. It supports multi-writer queue insertions.
* Update HSA Signals. A 64-bit atomic operation is used for CPU & GPU synchronization.


PCIe in ROCm
======================

PCIe operates as a completer for ``CAS`` (Compare and Swap), ``FetchADD``, and ``SWAP`` atomic operations. These atomic operations are initiated by the I/O devices that support 32-bit, 64-bit, and 128-bit operands. Likewise, the target memory address where these atomic operations are performed should also be aligned to the size of the operand. This alignment ensures that the operations are performed efficiently and correctly without failure. 

When an atomic operation is successful the requester receives a response of completion along with the operation result. However, any errors associated with the operation, are signaled to the requester by updating the Completion Status field. Some common errors can be issues in accessing the target location or executing the atomic operation. Depending upon the error, the Completion Status field is updated to Completer Abort (CA) or Unsupported Request (UR). The field is present in the Completion Descriptor.

To learn more about the industry standards and specifications of PCIe, see `PCI-SIG Specification <https://pcisig.com/specifications>`_.

To learn more about PCIe and its capabilities, consult the following white papers:

* `Atomic Read Modify Write Primitives by Intel <https://www.intel.es/content/dam/doc/white-paper/atomic-read-modify-write-primitives-i-o-devices-paper.pdf>`_
* `PCI Express 3 Accelerator White paper by Intel <https://www.intel.sg/content/dam/doc/white-paper/pci-express3-accelerator-white-paper.pdf>`_
* `PCIe Generation 4 Base Specification includes atomic operations <https://astralvx.com/storage/2020/11/PCI_Express_Base_4.0_Rev0.3_February19-2014.pdf>`_
* `Xilinx PCIe Ultrascale White paper <https://docs.xilinx.com/v/u/8OZSA2V1b1LLU2rRCDVGQw>`_

Working of PCIe 3.0 atomic operations in ROCm
-------------------------------------------------
PCIe 3.0 allows atomic operations to be requested by, routed through, and completed by PCIe components. Routing and completion do not require software support. Component support for each can be identified by the Device Capabilities 2 (DevCap2) register. Upstream
bridges need to have atomic operations routing enabled. If not enabled, the atomic operations will fail even if the 
PCIe endpoint and the PCIe I/O devices can perform atomic operations.

To enable atomic operations routing between multiple Root Ports, each Root Port must support atomic operations routing. This capability can be identified from the atomic operations routing supported bit in the DevCap2 register. If the bit has value 1, routing is supported and if the value is 0, routing is not supported.

If your system has a PCIe Express Switch it needs to support atomic operations routing. Atomic
operations requests are permitted only if a component's ``DEVCTL2.ATOMICOP_REQUESTER_ENABLE``
field is set. These requests can only be serviced if the upstream components also support atomic operation
completion and/or can route it to a component that supports it. 

ROCm also takes advantage of the PCIe-ID-based ordering technology for peer-to-peer (P2P) data transmission when the GPU
initiates two write operations to two different targets. As an example scenario, there are two write operations:

1. Write to another GPU memory.
2. Write to system memory, to indicate the transfer is complete.

In the above scenario, the two write operations are routed off to different locations. However, it should be ensured that the order of the operation is, write to GPU completion followed by write to system memory, to indicate transfer is complete.

I/O devices and CPUs with PCIe atomics support
------------------------------------------------

For optimum use of PCIe 3.0 atomic operations features, PCIe-supported I/O devices are needed. Some of the I/O devices with PCIe atomic support are: 

* Mellanox ConnectX-5 InfiniBand Card
* Cray Aries Interconnect
* Xilinx 7 Series Devices

Likewise, the Gen-Z interconnect standard can be used for optimum memory access to data and resources. It is one of the bus technologies with advanced I/O atomics operation support.

ROCm requires CPUs that support PCIe atomics. Modern CPUs after the release of first generation AMD Zen CPU and Intel™ Haswell support PCIe atomics. Some of the PCIe Endpoints with support beyond AMD Ryzen, AMD EPYC, Intel™ Haswell, or newer CPUs with PCIe 3.0 support are:

* Mellanox Bluefield SOC
* Cavium Thunder X2




