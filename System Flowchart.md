```mermaid
flowchart TD
    %% Estilos de Nodos
    classDef terminalStyle fill:#1a365d,stroke:#2b6cb0,stroke-width:2px,color:#fff;
    classDef processStyle fill:#2d3748,stroke:#4a5568,stroke-width:2px,color:#fff;
    classDef decisionStyle fill:#744210,stroke:#d69e2e,stroke-width:2px,color:#fff;
    classDef ioStyle fill:#22543d,stroke:#38a169,stroke-width:2px,color:#fff;

    %% FASE 1: POST Y PERIFÉRICOS
    subgraph F1["1. POST Y PERIFÉRICOS"]
        START([Inicio: Botón Power]):::terminalStyle --> READ_BAT[Lectura de Voltaje Batería]:::processStyle
        READ_BAT --> DEC_BAT{¿Voltaje OK?}:::decisionStyle
        
        DEC_BAT -- NO --> ERR_BAT[Parpadear LED Azul 3 Veces]:::processStyle --> OFF1([Apagado]):::terminalStyle
        DEC_BAT -- SÍ --> BEEP_POST[/Tono POST en Buzzer/\]:::ioStyle --> SCAN_BUS[Escanear Bus Inter-FPGA]:::processStyle
        
        SCAN_BUS --> POLL_POT{¿Potenciómetro ON?}:::decisionStyle
        POLL_POT -- NO --> STANDBY[Standby Local]:::processStyle
        POLL_POT -- SÍ --> BOOT_SCR[/Pantalla de Inicio 64x64/\]:::ioStyle --> CHECK_PERIPH{¿Periférico Detectado?}:::decisionStyle
        
        CHECK_PERIPH -- NO --> WARN_P[/Aviso: Conectar Periférico/\]:::ioStyle
        WARN_P --> WAIT_P{¿Pulsó Botón?}:::decisionStyle
        WAIT_P -- SÍ --> CHECK_PERIPH
        WAIT_P -- NO --> WARN_P
    end

    CHECK_PERIPH -- SÍ --> BIOS_MENU

    RET_BIOS --> BIOS_MENU

    %% FASE 4: APAGADO (EVENTO DE HARDWARE)
    subgraph F4["4. APAGADO GENERAL"]
        EVENT_PWR[/Evento: Pulsación Botón Power en BIOS/\]:::ioStyle --> SAVE_SYS[Guardar Registros del Sistema]:::processStyle
        SAVE_SYS --> LED_OFF[LED Azul Parpadea 2 Veces]:::processStyle
        LED_OFF --> SHUTDOWN([Apagado Seguro]):::terminalStyle
    end

    BIOS_MENU -. Interrupción / Pulsación .-> EVENT_PWR

    %% FASE 2: MENÚ DE AJUSTES DEL SISTEMA (BIOS / INICIO)
    subgraph F2["2. MENÚ DE AJUSTES DEL SISTEMA"]
        BIOS_MENU[/Menú de Ajustes: Volumen - Keybindings - Batería - Jugar - Multijugador/\]:::ioStyle
        
        BIOS_MENU --> DEC_BIOS{¿Opción Seleccionada en Menú?}:::decisionStyle
        
        %% Ajustes secundarios
        DEC_BIOS -- VOLUMEN --> CONF_VOL[/Configurar Volumen Parlante Local/\]:::ioStyle --> BIOS_MENU
        DEC_BIOS -- KEYBINDINGS --> SHOW_KEYS[/Mostrar Mapeo de Teclas: Mouse - NES - Teclado/\]:::ioStyle --> BIOS_MENU
        
        %% Opción MULTIJUGADOR
        DEC_BIOS -- MULTIJUGADOR --> CHK_M_CART{¿Hay Cartucho en Puerto Maestro<br/>y es Multijugador Válido?}:::decisionStyle
        CHK_M_CART -- NO --> WARN_NO_MULTI[/Aviso: Inserte Juego Multijugador Válido en Puerto Maestro/\]:::ioStyle --> BIOS_MENU
        CHK_M_CART -- SÍ --> WAIT_SLAVES{¿Hay otra Pantalla Encendida y Lista?}:::decisionStyle
        WAIT_SLAVES -- NO --> WARN_WAIT_SCR[/Buzzer + En Espera de que otra Pantalla se Encienda/\]:::ioStyle --> WAIT_SLAVES
        WAIT_SLAVES -- SÍ --> SET_MULTI_FLAG[Activar Flag Modo Multijugador]:::processStyle --> GAME_MENU
        
        %% Opción JUGAR (Individual / Espejo)
        DEC_BIOS -- JUGAR --> CHK_ANY_CART{¿Hay Cartucho Conectado?<br/>Local o Maestro}:::decisionStyle
        CHK_ANY_CART -- NO --> WARN_NO_CART[/Aviso: Inserte un Cartucho de Juego/\]:::ioStyle --> BIOS_MENU
        CHK_ANY_CART -- SÍ --> SET_SINGLE_FLAG[Activar Flag Modo Individual]:::processStyle --> GAME_MENU
    end

    %% FASE 3: SUBMENÚ DEL JUEGO Y BUCLE DE EJECUCIÓN
    subgraph F3["3. MENÚ DEL JUEGO Y EJECUCIÓN"]
        GAME_MENU[/Menú del Juego: Iniciar Partida - Puntajes - Salir/\]:::ioStyle
        
        GAME_MENU --> DEC_GAME_OPT{¿Opción del Juego?}:::decisionStyle
        
        %% Puntajes
        DEC_GAME_OPT -- PUNTAJES --> SHOW_SCORES[/Mostrar Tabla de High Scores/\]:::ioStyle --> GAME_MENU
        
        %% Salir del Juego
        DEC_GAME_OPT -- SALIR --> SAFE_UNMOUNT[Desmontaje Seguro de Memoria / Flush EEPROM]:::processStyle --> RET_BIOS[/Volver a Menú BIOS/\]:::ioStyle
        
        %% Iniciar Partida
        DEC_GAME_OPT -- INICIAR PARTIDA --> CHK_MODE_FLAG{¿Modo de Juego?}:::decisionStyle
        
        %% Flujo Multijugador (Requiere Confirmación Doble)
        CHK_MODE_FLAG -- Multijugador --> CONFIRM_P1[/Pantalla 1: Presione Botón para Confirmar/\]:::ioStyle
        CONFIRM_P1 --> CONFIRM_P2[/Pantalla 2: Presione Botón para Confirmar/\]:::ioStyle
        CONFIRM_P2 --> SYNC_SPI[Sincronizar FPGAs vía Bus SPI]:::processStyle --> RUN_GAME[Ejecutar Lógica del Juego]:::processStyle
        
        %% Flujo Individual / Espejo
        CHK_MODE_FLAG -- Individual --> RUN_GAME
        
        %% Bucle de Juego Activo
        RUN_GAME --> GAME_STATE{¿Estado de Juego?}:::decisionStyle
        GAME_STATE -- JUGADOR MUERE --> GAME_MENU
        GAME_STATE -- PAUSA --> MENU_PAUSE[/Menú Pausa: Continuar o Salir/\]:::io
        MENU_PAUSE --> MENU_PAUSE_OPT{"¿Opción Pausa?"}:::decision
        MENU_PAUSE_OPT -- CONTINUAR --> RUN_GAME
        MENU_PAUSE_OPT -- SALIR --> GAME_MENU
    end

```
