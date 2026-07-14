# Fuente de Laboratorio Analógica 0-36V DC

Proyecto KiCad de una fuente de alimentación de laboratorio regulada, de 0 a 36 voltios DC, con diseño 100% analógico.

## Especificaciones

| Parámetro | Valor |
|-----------|-------|
| Salida | 0 – 36 V DC continuo |
| Corriente máxima | 5 A |
| Topología | Lineal, regulador serie |
| Control | Analógico (BJT + MOSFET) |
| Indicador | LED de potencia |
| Tensión de entrada | A definir (transformador + rectificador) |

## Estructura del Proyecto

```
pwr_supply_labs36v_kcd/
├── power_supply_36v_analog/
│   ├── power_supply_36v_analog.kicad_pro    # Archivo principal KiCad
│   ├── power_supply_36v.kicad_sch           # Esquema raíz (jerárquico)
│   ├── control_driver.kicad_sch             # Hoja: driver de control
│   ├── pwr_driver_output.kicad_sch          # Hoja: driver de potencia
│   ├── output_pwr.kicad_sch                 # Hoja: etapa de salida
│   ├── power_supply_36v_analog.kicad_pcb    # Layout de PCB
│   ├── Library_power_labs_libraries.kicad_sym  # Librería de símbolos
│   └── sym-lib-table                        # Tabla de librerías
├── helps/                                   # Datasheets, fotos, listas
├── docs/                                    # Documentación de diseño
├── .gitignore
└── README.md
```

## Hojas Jerárquicas

El esquema está dividido en 4 hojas jerárquicas:

### 1. `power_supply_36v` — Fuente principal
Etapa de entrada y regulación de tensión base.

### 2. `control_driver` — Driver de control
Circuito de control que regula la tensión de salida mediante realimentación.

### 3. `pwr_driver_output` — Driver de potencia
Etapa de amplificación de corriente con transistores de potencia (MJL21194/MJL21193).

### 4. `output_pwr` — Etapa de salida
MOSFETs de salida (5.5N45V) y diodos de protección.

## Lista de Componentes Principales

| Ref | Componente | Descripción | Uso |
|-----|------------|-------------|-----|
| Q1, Q2 | P_Q4V5N | 5.5N 45V 5A N-MOSFET | Salida de potencia |
| Q3 | P_MJL21194 | NPN 250V 16A 200W | Driver NPN |
| Q4 | P_MJL21193 | PNP 250V 16A 200W | Driver PNP |
| Q5 | P_TIP41C | NPN 100V 6A 65W | Pre-driver NPN |
| Q6 | P_TIP42C | PNP 100V 6A 65W | Pre-driver PNP |
| D1, D2 | P_BZX84C12 | Zener 12V | Regulación |
| D3 | P_1N4742 | Zener 12V 1W | Regulación |
| D4 | P_RD91ES | LED rojo | Indicador |
| D5-D7 | P_MRA4007T3G | 700V 1A Rápido | Rectificación |
| D8, D9 | P_SR560 | Schottky 60V 5A | Salida |
| R1 | P_1806 | 68K 3W | Potencia |

## Requisitos

- [KiCad 8.0](https://www.kicad.org/) o superior
- macOS / Linux / Windows

## Abrir el Proyecto

```bash
# Clonar repositorio
git clone https://github.com/TU_USUARIO/pwr_supply_labs36v_kcd.git
cd pwr_supply_labs36v_kcd

# Abrir en KiCad
open power_supply_36v_analog/power_supply_36v_analog.kicad_pro
```

## Diseño Jerárquico

```
┌─────────────────────────────────────────────────────┐
│              power_supply_36v.kicad_sch              │
│                                                     │
│  ┌──────────────┐    ┌──────────────┐              │
│  │ control_     │    │ pwr_driver_  │              │
│  │ driver       │────│ output       │──── Salida   │
│  └──────────────┘    └──────────────┘              │
│         │                                          │
│  ┌──────────────┐                                  │
│  │ output_pwr   │                                  │
│  └──────────────┘                                  │
└─────────────────────────────────────────────────────┘
```

## Licencia

Este proyecto utiliza KiCad, que está bajo licencia **GPL v2+**. Los archivos fuente de este proyecto son compatibles con esta licencia.

## Autor

**ERTS DIGITAL**

---

*Proyecto de fuente de laboratorio analógica de alta potencia para uso educativo y de laboratorio.*
