# SOFTWARE-ARCH


graph TB
    %% ===== USER INTERFACE LAYER =====
    subgraph UI[
        U I LAYER]
        direction LR
        HMI[HMI Touch Interface<br/>User Control Panel]
        WEB[Mainsail Web Interface<br/>Debug & Monitoring]
    end
    
    %% ===== NETWORK & API LAYER =====
    subgraph NET_LAYER[NETWORK LAYER]
        direction TB
        API[Moonraker API<br/>Central Control Hub]
        VBRIDGE[VBridge<br/>ROS ↔ Moonraker<br/>Protocol Translator]
        
    end
    
    %% ===== PROCESSING LAYER =====
    subgraph PROC[PROCESSING LAYER]
        direction TB
        subgraph RPI[ Raspberry Pi - Central Controller]
            KLIPPY[ Klippy Host<br/>Motion Coordinator]
            QUEUE[Queue Manager<br/>Task Scheduler]
            KLIP_L[ Klipper FW 1<br/>Left Frame]
            KLIP_R[ Klipper FW 2<br/>Right Frame]
        end
        
        ESP[ ESP32 Controller<br/>Servo Bus Manager<br/>5 Servos]
    end
    
    %% ===== HARDWARE CONTROL LAYER =====
    subgraph HW_CTRL[HARDWARE CONTROL LAYER]
        direction LR
        OCT_L[Octopus Pro 1<br/>Left Frame Controller<br/>6 Steppers]
        OCT_R[Octopus Pro 2<br/>Right Frame Controller<br/>6 Steppers]
    end
    
    %% ===== PHYSICAL HARDWARE LAYER =====
    subgraph MOTORS[ HARDWARE LAYER]
        direction TB
        
        subgraph LEFT_FRAME[LEFT FRAME]
            direction LR
            L1[S1] --- L2[S2] --- L3[S3]
            L4[S4] --- L5[S5] --- L6[S6]
        end
        
        subgraph RIGHT_FRAME[RIGHT FRAME]
            direction LR
            R1[S1] --- R2[S2] --- R3[S3]
            R4[S4] --- R5[S5] --- R6[S6]
        end
        
        subgraph SERVO_CLUSTER[ESP SERVO BUS - 5 SERVOS]
            direction LR
            S1[SV1] --- S2[SV2] --- S3[SV3] --- S4[SV4] --- S5[SV5]
        end

    end
    
    %% ===== DIGITAL TWIN SYSTEM =====
    subgraph DT_SYS[DIGITAL TWIN SYSTEM - SEPARATE SERVER]
        direction TB
        ROS2[ROS2 Framework<br/>Distributed Nodes]
        MUJOCO[MuJoCo Physics<br/>Real-time Simulation]
        VISION[Vision Node<br/>Camera Processing]
        CAM[Camera System<br/>Optional]
    end
    
    %% ===== SYSTEM SERVICES =====
    subgraph SERVICES[SYSTEM SERVICES]
        direction LR
        LOGS[Custom Logging<br/>System Monitor]
        OTA[OTA Updates<br/>Firmware Manager]
    end
    
    %% =================== CONNECTIONS ===================
    
    %% User Interface Connections
    UI -.->|User Commands| NET_LAYER
    
    %% API Layer Connections
    API <--> QUEUE
    API <--> VBRIDGE
    API <--> KLIPPY
    API <--> ESP
    
    %% VBridge to Digital Twin
    VBRIDGE <-.->|Real-time Data<br/>Bidirectional| ROS2
    
    %% Processing Layer Internal
    KLIPPY --> KLIP_L
    KLIPPY --> KLIP_R

    %% Processing to Hardware Control
    KLIP_L -->|UART 1| OCT_L
    KLIP_R -->|UART 2| OCT_R
    ESP -->|Servo Bus Protocol| SERVO_CLUSTER
    
    %% Hardware Control to Motors
    OCT_L --> LEFT_FRAME
    OCT_R --> RIGHT_FRAME
    
    %% Digital Twin Internal
    ROS2 <--> MUJOCO
    CAM -.->|If Available| VISION
    VISION --> ROS2
    
    %% System Services
    SERVICES <--> RPI
    SERVICES <--> ESP
    
    %% Data Flow Indicators
    API -.->|System State| LOGS
    MUJOCO -.->|Simulation Data| LOGS
    
    %% =================== STYLING ===================
    classDef userLayer fill:#e3f2fd,stroke:#1976d2,stroke-width:3px,color:#000
    classDef networkLayer fill:#f3e5f5,stroke:#7b1fa2,stroke-width:3px,color:#000
    classDef processLayer fill:#e8f5e8,stroke:#388e3c,stroke-width:3px,color:#000
    classDef hardwareLayer fill:#fff3e0,stroke:#f57c00,stroke-width:3px,color:#000
    classDef motorLayer fill:#fce4ec,stroke:#c2185b,stroke-width:3px,color:#000
    classDef digitalLayer fill:#f1f8e9,stroke:#689f38,stroke-width:3px,color:#000
    classDef serviceLayer fill:#ede7f6,stroke:#512da8,stroke-width:3px,color:#000
    
    class UI userLayer
    class NET_LAYER,API,VBRIDGE,QUEUE networkLayer
    class PROC,RPI,KLIPPY,KLIP_L,KLIP_R,ESP processLayer
    class HW_CTRL,OCT_L,OCT_R,MANUAL hardwareLayer
    class MOTORS,LEFT_FRAME,RIGHT_FRAME,SERVO_CLUSTER,MANUAL_AXES,L1,L2,L3,L4,L5,L6,R1,R2,R3,R4,R5,R6,S1,S2,S3,S4,S5,MX,MY,MZ,MB motorLayer
    class DT_SYS,ROS2,MUJOCO,VISION,CAM digitalLayer
    class SERVICES,LOGS,OTA serviceLayer
