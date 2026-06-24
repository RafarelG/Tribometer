# Control de Tribómetro en LabVIEW

Este repositorio contiene la documentación y estructura de bloques para el control de motores paso a paso Arcus y la adquisición de datos de sensores mediante NI-DAQmx.

## Diagrama de Flujo del Sistema

```mermaid
graph TD
    %% Estilo de nodos estilo GitHub/Tech
    classDef init fill:#edf2f7,stroke:#4a5568,stroke-width:2px,color:#2d3748;
    classDef loop fill:#ebf8ff,stroke:#3182ce,stroke-width:2px,color:#2b6cb0;
    classDef state fill:#f0fff4,stroke:#38a169,stroke-width:2px,color:#276749;
    classDef hardware fill:#fffaf0,stroke:#dd6b20,stroke-width:2px,color:#7b341e;
    classDef endState fill:#fff5f5,stroke:#e53e3e,stroke-width:2px,color:#9b2c2c;

    %% 1. INICIALIZACIÓN
    subgraph Inicializacion_del_Sistema [1. Inicio y Configuración Inicial]
        A[Cargar Archivo de Calibración] --> B[Definir Unidades: N y N-m]
        B --> C[Inicializar colas de referencia: X, Y, Z Axis]
        C --> D[Configurar Canales Virtuales NI-DAQmx]
    end
    class Inicializacion_del_Sistema init;

    %% 2. BUCLE PRINCIPAL / INTERFAZ
    D --> E[Entrada al WHILE LOOP Principal]
    
    subgraph Bucle_Ejecucion [2. Núcleo del Programa: Estructura de Eventos y Estados]
        E --> F{¿Qué acción ocurre?}
        
        %% Evento de Usuario (Teclado)
        F -- Interacción de Usuario --> G[Estructura de Eventos: Key Down / Up]
        G --> H[Estado: Manual Control]
        H --> H1[Enviar Comandos ARCUS CMD por Teclado - Flechas y Números: Mover X, Y, Z]
        
        %% Evento de Estado Automático
        F -- Flujo Automático del Ensayo --> I[Selector de Casos: Máquina de Estados]
        
        I --> J[Estado: TARE]
        J --> J1[Leer Sensores en Vacío y poner en Cero ATARE]
        
        I --> K[Estado: MEASURE]
        K --> K1[Adquisición DAQmx continua - Fuerzas Fx, Fy, Fz y Torques]
        K1 --> K2[Calcular Coeficiente de Fricción Estimado]
        K2 --> K3[Lazo de Control PID: Ajustar Carga Real vs Testing Load]
        K3 --> K4[Actualizar contadores: Ciclos de Desgaste, Velocidad y Distancia]
        
        I --> L[Estado: STOP]
        L --> L1[Frenar Motores de Forma Segura]
    end
    class Bucle_Ejecucion loop;
    class H1,J1,K1,K2,K3,K4,L1 state;

    %% 3. CAPA DE HARDWARE
    subgraph Capa_Hardware [3. Conexión Física Real]
        H1 -. USB .- M[Controladores Motores ARCUS]
        K3 -. USB .- M
        K1 -. NI-DAQmx .- N[Sensores / Celda de Carga del Tribómetro]
    end
    class Capa_Hardware hardware;

    %% 4. CIERRE Y SALIDA
    K4 -->|¿Fin de ciclos o botón ABORT?| O[Estado: TERMINATE]
    L1 --> O
    
    subgraph Salida_Segura [4. Cierre del Sistema]
        O --> P[Detener Tareas DAQmx de Raíz]
        P --> Q[Cerrar Canales de Comunicación y Colas]
        Q --> R((Fin del Programa))
    end
    class Salida_Segura endState;
