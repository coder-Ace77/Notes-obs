
---

Jenkins is a **Continuous Integration (CI)** and **Continuous Delivery (CD)** tool.

### **Jenkins Master (Controller)**

- The **central server** that manages the entire system.
    
- Responsible for:
    - Scheduling build jobs.
    - Dispatching builds to agents (slaves).
    - Monitoring agents and builds.
    - Managing the UI, user requests, and job configurations.
- Runs the **web interface** and API.

### 2. **Jenkins Agents (Slaves)**

- Machines or nodes that perform the actual build work.
- Agents connect to the master and execute jobs dispatched to them.
- Can run on different platforms (Linux, Windows, macOS).
- Help in **scaling** Jenkins horizontally by distributing workload.