.. meta::
   :description: How ROCm uses PCIe atomics
   :keywords: PCIe, PCIe atomics, atomics, AMD, ROCm

*****************************************************************************
How ROCm Uses PCIe Atomics
*****************************************************************************
AMD ROCm is an extension of the Heterogeneous System Architecture (HSA). To meet the requirement of an HSA-compliant system, ROCm must support queuing model, memory model, signaling, and synchronization protocols. To learn more about the requirement of HSA-compliant system, see 
`HSA Platform System Architecture Specification <http://hsafoundation.com/wp-content/uploads/2021/02/HSA-SysArch-1.2.pdf>`_.

When memory operations like queuing, signaling, and synchronization needs to be performed across multiple CPUs and GPUs agents, platform atomics are required. Platform atomics ensures that atomic operations run synchronously without any interruption and conflict across multiple shared resources. Learn how platform atomics are supported in ROCm by Peripheral Component Interconnect Express 3.0 (PCIe™ 3.0).

Usage of Platform Atomics in ROCm 
=====================================

Platform atomics are the set of atomic operations that performs Read Modify Write (RMW) actions across multiple processor, devices, and memory location synchronously without any interruption. Platform atomics used in ROCm are:

* Update HSA queue's ``read_dispatch_id``: 64 bit atomic add used by the command processor on the
  GPU agent. It updates the packet ID it processed.
* Update HSA queue's ``write_dispatch_id``: 64 bit atomic add used by the CPU and GPU agent. It supports multi-writer queue insertions.
* Update HSA Signals: 64 bit atomic operation used for CPU & GPU synchronization.

Atomic Operations
-------------------
An atomic operation is a sequence of computing instructions that are executed as a single, indivisible unit. These instructions needs to be completed at its entirety without any interruption or should not execute at all. It is a non-posted transaction supporting 32-bit and 64-bit address formats. A successful completion of these operations are identified by response of completion containing the operation result. However, any errors associated with the operation like issue in accessing the target location or executing the atomic operation are signaled to the requester by updating the Completion Status field present in the Completion Descriptor. Depending upon the error, the Completion Status field is updated to Completer Abort (CA) or Unsupported Request (UR).

PCIe in ROCm
======================
AMD ROCm uses atomic RMW transactions that extends inter-processor synchronization mechanisms to Input/Output (I/O) devices starting from  PCIe 3.0. It supports the defined set of HSA capabilities needed for queuing and signaling memory operations. 

PCIe operates as a completer for atomic operations like:  
* ``CAS`` (Compare and Swap)
* ``FetchADD``
* ``SWAP``

The atomic operations are initiated by the I/O devices that support 32-bit, 64-bit, and
128-bit operand. The target address has to be naturally aligned to the operation size.

To learn more about the industry standards and specifications of PCIe, see `PCI-SIG Specification <https://pcisig.com/specifications>`_

To learn more about PCIe and its capabilities, see the listed white papers:

* `Atomic Read Modify Write Primitives by Intel <https://www.intel.es/content/dam/doc/white-paper/atomic-read-modify-write-primitives-i-o-devices-paper.pdf>`_
* `PCI express 3 Accelerator White paper by Intel <https://www.intel.sg/content/dam/doc/white-paper/pci-express3-accelerator-white-paper.pdf>`_
* `PCIe Generation 4 Base Specification includes atomic operations <https://astralvx.com/storage/2020/11/PCI_Express_Base_4.0_Rev0.3_February19-2014.pdf>`_
* `Xilinx PCIe Ultrascale White paper <https://docs.xilinx.com/v/u/8OZSA2V1b1LLU2rRCDVGQw>`_

Working of PCIe 3.0 Atomic Operations in ROCm
-------------------------------------------------

The PCIe 3.0 atomic operations feature allows atomic transactions to be requested by, routed through, 
and completed by PCIe components. Routing and completion does not require software support.
Component support for each can be identified by the Device Capabilities 2 (DevCap2) register. Upstream
bridges needs to have atomic operations routing enabled. If not enabled, the atomic operations will fail even if the 
PCIe endpoint and the PCIe I/O devices have the capability to perform atomic operations.

To enable atomic operations routing between multiple Root Ports, each Root Port must show this capability through the atomic operations routing supported bit in the DevCap2 register.

If your system has a PCIe Express Switch it needs to support atomic operations routing. Atomic
operations requests are permitted only if a component's ``DEVCTL2.ATOMICOP_REQUESTER_ENABLE``
field is set. These requests can only be serviced if the upstream components support atomic operation
completion and/or can route it to a component that does. 

ROCm also takes the advantage of PCIe ID based ordering technology for peer-to-peer (P2P) data transmission when the GPU
initiates two write operations to two different targets. As an example scenario, there are two write operations:

1. Write to another GPU memory
2. Write to system memory to indicate transfer complete

In the above scenario, the write operations are routed off to different ends of the computer. However, it should be ensured that the order of the operation should be, write to GPU completion followed by write to system memory to indicate transfer complete.

I/O Devices and CPUs with PCIe Atomics Support
------------------------------------------------

For optimum use of PCIe 3.0 atomic operations features, PCIe supported I/O devices are need. Some of the I/O devices with PCIe atomic support are: 

* Mellanox ConnectX-5 InfiniBand Card
* Cray Aries Interconnect
* Xilinx 7 Series Devices

For optimum memory access to data and resources Gen-Z interconnect standard can be used. Gen-Z is one of the bus technology with advanced I/O atomics operation support.

ROCm requires CPUs that support PCIe atomics. Modern CPUs after the release of first generation AMD Zen CPU and Intel™ Haswell support PCIe atomics. Some of the PCIe Endpoints with support beyond AMD Ryzen, AMD EPYC, Intel™ Haswell, or newer CPUs with PCIe 3.0 support are:

* Mellanox Bluefield SOC
* Cavium Thunder X2




