# AutoMesh: The Self-Organizing, Low-Power Mesh Protocol for IoT

**⚠️STATUS: Design Stage - Concept & Architecture Definition⚠️**

## Overview

**AutoMesh** is an innovative, custom-built low-power communication protocol (LCP) designed for robust, self-healing, and highly efficient mesh networking in constrained Internet of Things (IoT) environments. Built from the ground up to minimize complexity, AutoMesh offers a streamlined alternative to more feature-heavy standards, focusing on reliable local device communication without requiring full IP stack overhead.

## The Problem AutoMesh Solves

Modern IoT mesh protocols, while powerful, often come with significant overhead (e.g., full IPv6 networking, complex service discovery mechanisms, extensive memory footprints) that can be overkill for many simple, localized IoT applications. This complexity can lead to increased development time, higher power consumption, and greater resource requirements for constrained, battery-powered devices.

AutoMesh aims to provide a lean, efficient, and "just works" solution for these scenarios.

## How AutoMesh Works (Core Principles)

AutoMesh is founded on a **Tree-Based Addressing (TBA)** scheme, where devices form a hierarchical mesh network managed by a single elected **Leader**.

* **Tree-Based Addressing (TBA):** Devices are assigned a unique 16-bit address (8-bit Router ID + 8-bit Child ID) reflecting their position in the network's tree structure. This enables efficient, explicit routing within the local mesh.
* **Self-Healing & Auto-Configuration:** The network dynamically forms, automatically assigns IDs to new devices, and seamlessly re-elects a Leader or reconfigures paths in case of device failures or topology changes. Devices automatically discover and attach to optimal parent routers.
* **Simplified Service Discovery:** A lightweight mechanism allows devices to register "roles" (e.g., "gateway," "temperature_sensor") with the Leader. Other devices can query the Leader to find services by role or ID, facilitating application-layer communication.
* **Ultra Low-Power Design:** Optimized for battery-powered End Devices (SEDs) through scheduled communication with their parent routers, allowing for deep sleep cycles and extended battery life.
* **Robust Security:** All communication within the mesh is secured using AES128 encryption with a single, network-wide pre-shared key, ensuring data privacy and integrity.
* **OS-Independent LCP:** The core protocol logic is designed to be highly portable, minimizing dependencies on specific operating system features, making it adaptable to various embedded platforms.

## Key Features

* **Hierarchical Addressing:** Efficient and intuitive routing.
* **Automatic Network Formation & ID Assignment:** Plug-and-play setup.
* **Dynamic Mesh Self-Healing:** Resilient against node failures.
* **Leader-Managed Network Map & Roles:** Centralized intelligence for simplified operation.
* **Optimized for Constrained Devices:** Low memory footprint and power consumption.
* **Reliable & Secure Communication:** AES128 encryption for all traffic.
* **Planned Hardware/Software:** Envisioned for nRF52840, utilizing Zephyr RTOS and C programming for performance and control.

## Ideal Use Cases

AutoMesh is perfectly suited for applications requiring:

* **Local, Private IoT Networks:** Where devices communicate primarily within a confined space.
* **Battery-Powered Sensor Networks:** Smart home sensors, environmental monitoring, asset tracking.
* **Simple Control Systems:** Lighting control, basic appliance automation.
* **Applications Minimizing Cloud Dependency:** Where core functionality should remain local even without internet access.
* **Cost-Sensitive Deployments:** Reducing overhead can lead to simpler, cheaper hardware.

## Next Steps

*(Here, you would describe the immediate next steps in the design process, e.g., "Detailed specification writing," "Protocol state machine definition," "Module breakdown," etc. You might also add a placeholder for when development is expected to begin.)*

## Contributing

*(You might add a section here about how others can contribute to the design phase, if applicable.)*

## License

*(Specify your chosen open-source license, e.g., MIT, Apache 2.0.)*

## Contact

*(Your contact information or preferred method for inquiries.)*
