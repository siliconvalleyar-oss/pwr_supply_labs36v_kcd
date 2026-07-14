# SKILL - Guía de Trabajo con KiCad

## Comandos Esenciales de KiCad

### Abrir Proyecto
```bash
# Abrir proyecto específico
open /Applications/KiCad/KiCad.app
# Luego File > Open Project > seleccionar .kicad_pro

# Abrir desde terminal (KiCad 8+)
kicad /ruta/al/proyecto/proyecto.kicad_pro
```

### Verificar Archivos
```bash
# Contar componentes en schematic
grep -c '(symbol ' archivo.kicad_sch

# Contar pads en PCB
grep -c '(pad ' archivo.kicad_pcb

# Listar net classes
grep -c 'net_class' archivo.kicad_pcb
```

## Estructura de Archivos KiCad

```
proyecto/
├── proyecto.kicad_pro          # Archivo principal
├── proyecto.kicad_sch          # Esquemático raíz
├── proyecto.kicad_pcb          # Layout de PCB
├── proyecto.kicad_prl          # Preferencias locales (NO commitear)
├── proyecto.kicad_sym          # Librería de símbolos
├── proyecto.pretty/            # Librería de footprints
│   ├── footprint1.kicad_mod
│   └── footprint2.kicad_mod
├── 3dmodels/                   # Modelos STEP
├── sym-lib-table               # Tabla de símbolos
├── fp-lib-table                # Tabla de footprints
└── fp-info-cache               # Cache (NO commitear)
```

## Flujo de Trabajo Recomendado

### 1. Diseño de Esquemático
```
1.1 Crear estructura jerárquica
1.2 Definir symbols (librería)
1.3 Conectar componentes con wires
1.4 Agregar power symbols (+36V, GND, etc.)
1.5 Usar hierarchical labels (NO global labels)
1.6 Ejecutar ERC
```

### 2. Asignación de Footprints
```
2.1 Asignar footprints desde librería
2.2 Crear footprints custom si es necesario
2.3 Verificar 3D models
2.4 Ejecutar DRC básico
```

### 3. Layout de PCB
```
3.1 Importar netlist
3.2 placement de componentes
3.3 Routing de señales
3.4 Power planes
3.5 Via stitching
3.6 Ejecutar DRC completo
```

### 4. Verificación
```
4.1 ERC en todos los schematics
4.2 DRC en PCB
4.3 Revisar BOM
4.4 Verificar Gerber outputs
4.5 Test points accesibles
```

## Scripts Útiles de Python

### Verificar Hierarchical Labels
```python
import re

def check_hierarchical_labels(schematic_path):
    with open(schematic_path, 'r') as f:
        content = f.read()
    
    # Buscar global labels (deberían ser jerárquicos)
    global_labels = re.findall(r'\(global_label "([^"]+)"', content)
    hierarchical = re.findall(r'\(hierarchical_label "([^"]+)"', content)
    
    if global_labels:
        print(f"⚠️  Global labels encontradas: {global_labels}")
        print("   Deberían ser hierarchical labels")
    else:
        print("✅ No hay global labels")
    
    print(f"   Hierarchical labels: {hierarchical}")
```

### Generar BOM
```python
import csv
import re

def generate_bom(schematic_path, output_csv):
    with open(schematic_path, 'r') as f:
        content = f.read()
    
    # Extraer componentes
    pattern = r'\(property "Reference" "([^"]+)".*?\(property "Value" "([^"]+)".*?\(property "Footprint" "([^"]*)"'
    components = re.findall(pattern, content, re.DOTALL)
    
    with open(output_csv, 'w', newline='') as f:
        writer = csv.writer(f)
        writer.writerow(['Reference', 'Value', 'Footprint'])
        for ref, value, footprint in components:
            writer.writerow([ref, value, footprint])
    
    print(f"BOM exportado: {output_csv}")
```

## Errores Comunes y Soluciones

| Error | Causa | Solución |
|-------|-------|----------|
| ERC: Pin sin conectar | Wire no llega al pin | Extender wire hasta el pin |
| ERC: Power pin sin drive | Falta power symbol | Agregar symbol de power |
| DRC: Track too close | Clearance insuficiente | Aumentar spacing |
| DRC: Via too small | Via menor que mínimo | Usar via size estándar |
| Symbol not found | Falta librería | Agregar a sym-lib-table |
| Footprint not found | Falta librería | Agregar a fp-lib-table |

## Git para Proyectos KiCad

### .gitignore Recomendado
```gitignore
# KiCad generated
fp-info-cache
*.kicad_prl
*-backups/
*.bak

# Temporales
*.dsn
temp-*.dsn
report.txt

# macOS
.DS_Store
```

### Comandos Útiles
```bash
# Ver cambios en schematic
git diff -- '*.kicad_sch'

# Ver cambios en PCB
git diff -- '*.kicad_pcb'

# Historial de un archivo
git log --follow -- archivo.kicad_sch
```

## Referencias Rápidas

### Net Classes por Defecto
- **Default**: 6 mil clearance, 8 mil width
- **Power**: 10 mil clearance, 20 mil width
- **High Speed**: 5 mil clearance, 8 mil width

### Tamaños de Via Comunes
- **Small**: 0.3mm drill, 0.6mm pad
- **Medium**: 0.4mm drill, 0.8mm pad
- **Large**: 0.6mm drill, 1.0mm pad

### Spacing de Pines
- **Standard**: 2.54mm (100 mil)
- **Fine pitch**: 1.27mm (50 mil)
- **BGA**: 0.8mm o menos
