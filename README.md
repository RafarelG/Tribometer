# Tribometer Control in LabVIEW

This repository contains the documentation and block structure for the control of Arcus stepper motors and data acquisition of sensors using NI-DAQmx.

## System Flowchart

```mermaid
graph TD
    %% Node Styling Technical/GitHub style
    classDef init fill:#edf2f7,stroke:#4a5568,stroke-width:2px,color:#2d3748;
    classDef loop fill:#ebf8ff,stroke:#3182ce,stroke-width:2px,color:#2b6cb0;
    classDef state fill:#f0fff4,stroke:#38a169,stroke-width:2px,color:#276749;
    classDef hardware fill:#fffaf0,stroke:#dd6b20,stroke-width:2px,color:#7b341e;
    classDef endState fill:#fff5f5,stroke:#e53e3e,stroke-width:2px,color:#9b2c2c;

    %% 1. INITIALIZATION
    subgraph System_Initialization [1. Startup & Initial Configuration]
        A[Load Calibration File] --> B[Define Units: N and N-m]
        B --> C[Initialize Reference Queues: X, Y, Z Axis]
        C --> D[Configure NI-DAQmx Virtual Channels]
    end
    class System_Initialization init;

    %% 2. MAIN LOOP / INTERFACE
    D --> E[Enter Main WHILE LOOP]
    
    subgraph Execution_Loop [2. Core Program: Event Structure & States]
        E --> F{Which Action Occurs?}
        
        %% User Event (Keyboard)
        F -- User Interaction --> G[Event Structure: Key Down / Up]
        G --> H[State: Manual Control]
        H --> H1[Send ARCUS CMD via Keyboard - Arrows and Numbers: Move X, Y, Z]
        
        %% Automatic State Event
        F -- Automatic Test Flow --> I[Case Selector: State Machine]
        
        I --> J[State: TARE]
        J --> J1[Read Sensors in Vacuum and Set to Zero ATARE]
        
        I --> K[State: MEASURE]
        K --> K1[Continuous DAQmx Acquisition - Forces Fx, Fy, Fz and Torques]
        K1 --> K2[Calculate Estimated Friction Coefficient]
        K2 --> K3[PID Control Loop: Adjust Actual Load vs Testing Load]
        K3 --> K4[Update Counters: Wear Cycles, Velocity and Distance]
        
        I --> L[State: STOP]
        L --> L1[Safely Stop All Motors]
    end
    class Execution_Loop loop;
    class H1,J1,K1,K2,K3,K4,L1 state;

    %% 3. HARDWARE LAYER
    subgraph Hardware_Layer [3. Real Physical Connection]
        H1 -. USB .- M[ARCUS Motor Controllers]
        K3 -. USB .- M
        K1 -. NI-DAQmx .- N[Sensors / Tribometer Load Cell]
    end
    class Hardware_Layer hardware;

    %% 4. CLOSURE AND EXIT
    K4 -->|End of Cycles or ABORT Button?| O[State: TERMINATE]
    L1 --> O
    
    subgraph Safe_Exit [4. System Shutdown]
        O --> P[Stop DAQmx Tasks from Root]
        P --> Q[Close Communication Channels and Queues]
        Q --> R((End of Program))
    end
    class Safe_Exit endState;
