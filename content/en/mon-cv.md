---
title: "Christophe Delacourt | Principal Architect"
sitemap:
  disable: true
date: 2026-03-23
draft: false
---

{{< contact_print >}}

{{< print_button >}}

I am an architect who has never stopped developing. This is the core of my approach: defining a strategic vision and establishing its credibility by prototyping the most complex components myself. My motivation lies in guiding talented teams toward ambitious, production-ready solutions by providing them with the confidence and autonomy to innovate.

# Professional Experience

## Principal Architect
**Obvios** | *January 2022 - Present*

* **Leadership & Strategic Interface:** Leading the architecture department (4 experts). As the guardian of the cross-functional technical vision, I define the strategic direction for the **development teams (~30 people)** and serve as the lead technical point of contact for clients and partners on innovation projects (ANR, PIIEC).

* **Technical Strategy & Quality:** Driving cross-functional initiatives to improve processes and code quality: contributing to testing strategies and component update policies, implementing secure coding practices, leading workshops on development best practices, and managing various technical initiatives such as solution-level feature flagging and technical debt management.

* **Latest Architecture Design (NDR System):** End-to-end design of a cloud-native Network Detection and Response (NDR) system: direct metrics/logs ingestion, asynchronous correlation engine (`Go`/`Lua`), and Kubernetes `CRD` integration. To validate the approach, I developed the V0.1 of its most critical component (*SigStreak*), a `C++`/`LLVM` compiler generating filtering rules at 10 Gb/s injected directly into the kernel via `eBPF`/`XDP`. This prototype, validated and presented to prospects, enabled the team to move forward with the pre-production of the entire system.

* **Strategic Cross-functional Design:** Leading the drafting of architectures and specifications for complex, structural topics (High Availability systems, stateless architecture on Kubernetes, DevSecOps pipeline overhaul).

## Tech Lead 
**b<>com** | *December 2017 - January 2022*

* **4G/IoT RAN Scheduler:** Designed a 4G scheduler optimized for IoT (`NB-IoT`). Produced a prototype in collaboration with a 4G expert and developed an advanced test simulator for validation.

* **Cloud-Native LoRa IoT Gateway:** Optimized a research prototype (migrating from `Python` to `C/C++`). Validated the solution by **designing and developing a real-time radio simulator** capable of generating LoRa signals with controlled impairments. Designed a microservices architecture on `Kubernetes` and contributed to a new synchronization algorithm (see [publications](#publications)).

* **5G RAN Foundations:** Established CI/CD, TDD, and Clean Code practices, and developed a high-performance multithreaded communication library at the core of the 5G RAN.

## Co-founder & CTO
**Insane Unity** | *December 2013 - July 2017*

* **Management & Launch:** **Directed the development team (9 engineers)** and coordinated technically with the artists (total team of 14) until the commercial launch of the [game on Steam](https://store.steampowered.com/app/599040/Win_That_War/).

* **Innovation & Funding:** Obtained the **Jeune Entreprise Innovante (JEI)** status, securing **€1 million** in funding (private investors & Bpifrance) based on the proprietary technology I developed.

* **Technical & Product Direction:** End-to-end design of a **proprietary game engine** and a scalable server architecture on **`AWS`**.

## R&D Engineer & Project Manager 
**Caps Entreprise** | *October 2007 - December 2013*

* **Project Management:** Led customer service and optimization missions (OpenMP, MPI) with project teams of **up to 11 people**.

* **Design & Development:** Designed the Intermediate Representation (IR) for an HPC compiler and developed a low-level backend for ATI Radeon GPUs.

## Embedded Software Developer
**Yaccom** | *January 2005 - October 2007*

Integrated a `Bluetooth` stack and a Rich Media engine onto `NXP` mobile telephony platforms.

## Corporal, Signal Corps
**French Army** | *September 1998 - January 2005*

Administration and maintenance of military communication systems (tactical radios, satellites). This experience served as a springboard for my transition into IT.

# Key Skills

**Architecture & Cloud Native**

* Distributed systems design, stateless, high availability
* Microservices architecture, `Kubernetes`, `AWS`
* Observability platforms (`Prometheus`, `Grafana`, `Loki`)
* Application Security & DevSecOps (`SonarQube`, `Trivy`, `CI/CD`)

**Engineering & High Performance**

* **Tech Stack & Mastery:** `C/C++` (real-time, optimization), `Rust`, `Go`, `Python`, `C#`
* **Systems & Low-Level:** Compilers (`LLVM`), 3D Rendering & GPGPU (`Vulkan`, `OpenGL`, `HLSL/GLSL`, `CUDA`/`OpenCL`)
* **HPC & Performance Optimization:** `OpenMP`, `MPI`

**Networking & Telecoms**

* **Low-Level Networking & Kernel:** `DPDK`, `eBPF`, `CNI`
* Core Network (`5G`), `RAN` (`5G`, `4G`), `IoT` (`NB-IoT`, `LoRa`)
* Signal Processing (see [publications](#publications))

**Leadership & Strategy**

* Technical Direction (CTO), roadmap management
* Technical workshop facilitation (Agile Archi, TDD, Clean Code)
* Communication, diplomacy, and conflict resolution
* Multi-disciplinary team management, mentoring
* Entrepreneurship, fundraising, technical strategy
* Agile Methodologies (SCRUM)

# Publications

* [A Cloud RAN Architecture for LoRa](https://www.researchgate.net/publication/339835183_A_Cloud_RAN_Architecture_for_LoRa) - March 2020
* [On Time-Frequency Synchronization in LoRa System...](https://www.researchgate.net/publication/348549568_On_Time-Frequency_Synchronization_in_LoRa_System_From_Analysis_to_Near-Optimal_Algorithm) - January 2021
* [Considering Sync Word and Header Error Rates for...](https://www.researchgate.net/publication/349949777_Considering_Sync_Word_and_Header_Error_Rates_for_Performance_Assessment_in_LoRa_System) - April 2021
* [Quantization Methods Based on Norms for Complex...](https://www.researchgate.net/publication/363576337_Quantization_Methods_Based_on_Norms_for_Complex_Gaussian-Like_Signals_in_SDR_Applications) - August 2022

# Education & Continuous Learning

*Self-taught by nature, I have forged my expertise through intensive practice, technical experimentation, and the end-to-end design of distributed systems.*

* **2024**: English Language Improvement – Yes 'n You
* **2020**: MOOC in Signal Processing – EPFL via Coursera
* **2010**: Agile Project Management Certification (SCRUM) – Aubry Conseil
* **2004–2007**: DEST modules in Information Systems Architecture – CNAM
* **2003**: Associate Degree (DUT) in Computer Science – IUT de Vannes

# Languages & Interests

* **Languages**: French (Native), English (Professional).
* **Interests**: Game Development (Rust, Vulkan, Engine Architecture), Video Games, Tabletop RPGs, Wargaming, Cooking, Reading.
