# LEARNINGS - Habilidades y Proceso de Trabajo

> **NOTA**: Este documento contiene ejemplos genéricos. No incluye datos reales de credenciales o licencias.

## 1. Organización de Librerías KiCad

### Problema
Múltiples carpetas con archivos duplicados (.kicad_mod, .kicad_sym, .step).

### Proceso
1. **Escanear directorio**: Buscar todos los archivos por extensión
2. **Identificar duplicados**: Comparar nombres de archivo
3. **Verificar contenido**: Usar `md5` para detectar archivos idénticos
4. **Crear estructura limpia**:
   - `symbols.kicad_sym` (un solo archivo con todos los símbolos)
   - `footprints.pretty/` (carpeta con footprints individuales)
   - `3dmodels/` (carpeta con modelos STEP)
5. **Mergear símbolos**: Script Python para combinar .kicad_sym únicos

---

## 2. Extracción de Footprints desde PCB

### Proceso
```bash
# 1. Extraer nombres de footprints del PCB
grep -o '(footprint "[^"]*"' archivo.kicad_pcb | sed 's/(footprint "//;s/"//' | sort -u

# 2. Verificar cuáles faltan en la librería
while read fp; do
  name=$(echo "$fp" | sed 's/.*://')
  if [ -f "footprints.pretty/$name.kicad_mod" ]; then
    echo "✅ $fp"
  else
    echo "❌ $fp (FALTA)"
  fi
done < footprints_faltantes.txt

# 3. Buscar en librerías estándar de KiCad
find /Applications/KiCad/KiCad.app/Contents/SharedSupport/footprints -name "NombreFootprint.kicad_mod"

# 4. Copiar a la librería
cp /ruta/al/footprint.kicad_mod footprints.pretty/
```

---

## 3. Licencias en macOS

### Ubicación General de Licencias
```bash
# KiCad bundle - licencias incluidas
ls /Applications/KiCad/KiCad.app/Contents/Resources/Licenses/

# Python (incluido en KiCad)
ls /Applications/KiCad/KiCad.app/Contents/Frameworks/Python.framework/Versions/*/lib/python*/LICENSE.txt

# Otras aplicaciones
ls /Applications/*.app/Contents/Resources/Licenses/
```

### Encontrar Licencias de Cualquier App
```bash
# Buscar archivos LICENSE en una aplicación
find /Applications/NombreApp.app -name "LICENSE*" -o -name "COPYING*" 2>/dev/null

# Buscar en bundle completo
find /Applications/NombreApp.app -type f \( -name "*.txt" -o -name "*.md" \) | xargs grep -l "license\|License\|LICENSE" 2>/dev/null
```

### KiCad usa GPL v2+
- El binario de KiCad contiene la licencia GPL v2+ embebida
- Los proyectos y librerías creados con KiCad deben ser compatibles
- **Solución**: Crear archivo `LICENSE` manualmente con texto GPL v2+

### Crear LICENSE
```bash
# Descargar texto oficial
curl -o LICENSE https://www.gnu.org/licenses/old-licenses/gpl-2.0.txt

# O crear manualmente con el texto de gnu.org
```

---

## 4. Git Push con Token (HTTPS)

### Configuración
```bash
# Verificar usuario git local
git config user.name      # Ejemplo: "mi_usuario"
git config user.email     # Ejemplo: "mi@email.com"

# Verificar remote
git remote -v             # Muestra URLs configuradas
```

### Buscar Credenciales en macOS

#### Opción 1: Keychain (Recomendado)
```bash
# Buscar credenciales de GitHub en keychain
security find-generic-password -s "github.com" -w 2>/dev/null

# Buscar credenciales de GitLab
security find-generic-password -s "gitlab.com" -w 2>/dev/null

# Buscar credenciales de Bitbucket
security find-generic-password -s "bitbucket.org" -w 2>/dev/null

# Listar todas las credenciales de internet
security dump-keychain | grep -A5 "srvr"
```

#### Opción 2: Git Credential Store
```bash
# Verificar si hay credenciales guardadas
cat ~/.git-credentials 2>/dev/null

# Buscar tokens en archivos de configuración
grep -r "token\|password\|ghp_" ~/.git* 2>/dev/null | head -5
```

#### Opción 3: Cache de Git
```bash
# Verificar helper de credenciales
git config --global credential.helper

# Limpiar cache si es necesario
git credential reject <<EOF
protocol=https
host=github.com
EOF
```

### Proceso de Push
```bash
# 1. Temporalmente configurar remote con token
git remote set-url origin https://USUARIO:TOKEN@github.com/USUARIO/REPOSITORIO.git

# 2. Push
git push -u origin main

# 3. IMPORTANTE: Limpiar remote (quitar token)
git remote set-url origin https://github.com/USUARIO/REPOSITORIO.git
```

### Seguridad
- **NUNCA** commitear tokens en el repositorio
- **NUNCA** mostrar tokens en logs o pantallas
- **SIEMPRE** limpiar la URL después del push
- **USAR** keychain de macOS para almacenar tokens
- El token se almacena en la config global de git (no en el repositorio)

### Verificar Autenticación
```bash
# Probar conexión con GitHub
ssh -T git@github.com

# Verificar con curl (sin exponer token)
curl -s https://api.github.com/user | grep '"login"'

# Con token (ejemplo genérico)
# curl -H "Authorization: token ghp_EJEMPLO123..." https://api.github.com/user
```

---

## 5. Verificación de Contenido

### Contar archivos
```bash
# Footprints
ls footprints.pretty/*.kicad_mod | wc -l

# Símbolos (entradas principales)
grep -c '(symbol "' symbols.kicad_sym

# Modelos 3D
ls 3dmodels/*.step | wc -l
```

### Detectar duplicados
```bash
# Por nombre
find . -name "*.kicad_mod" -exec basename {} \; | sort | uniq -c | sort -rn

# Por contenido (hash MD5)
find . -name "*.kicad_mod" -exec md5 -r {} \; | awk '{print $1}' | sort | uniq -d
```

---

## 6. Estructura de Repositorio KiCad

```
repositorio/
├── symbols.kicad_sym      # Todos los símbolos en un archivo
├── footprints.pretty/      # Carpeta con footprints (.kicad_mod)
├── 3dmodels/              # Modelos STEP (.step)
├── LICENSE                # GPL v2+ (obligatorio)
├── README.md              # Documentación
└── .gitignore             # Excluir: *.zip, *.bak, .DS_Store, etc.
```

---

## 7. Script de Mergear Símbolos

```python
#!/usr/bin/env python3
import os
import re

def merge_symbols(source_dirs, output_file):
    """Combina múltiples .kicad_sym en uno solo"""
    unique_symbols = {}
    
    for source_dir in source_dirs:
        for root, dirs, files in os.walk(source_dir):
            for filename in files:
                if filename.endswith('.kicad_sym'):
                    filepath = os.path.join(root, filename)
                    with open(filepath, 'r') as f:
                        content = f.read()
                        symbols = re.findall(
                            r'\(symbol\s+"[^"]+".*?\n\s+\)\s*\)',
                            content, re.DOTALL
                        )
                        for symbol in symbols:
                            name = re.search(r'\(symbol\s+"([^"]+)"', symbol)
                            if name and name.group(1) not in unique_symbols:
                                unique_symbols[name.group(1)] = symbol
    
    with open(output_file, 'w') as f:
        f.write('(kicad_symbol_lib (version 20211014) (generator custom)\n')
        for name, symbol in sorted(unique_symbols.items()):
            f.write('  ' + symbol + '\n')
        f.write(')\n')
    
    return len(unique_symbols)
```

---

## 8. Comandos Útiles de KiCad

```bash
# Buscar footprints en KiCad
find /Applications/KiCad -name "*.pretty" -type d

# Buscar un footprint específico
find /Applications/KiCad -name "Nombre.kicad_mod" -type f

# Verificar estructura de archivo .kicad_sym
head -20 archivo.kicad_sym
```

---

## 9. Gitignore para KiCad

```gitignore
# Archivos KiCad
*.kicad_pcb-bak
*.kicad_sch-bak
fp-info-cache

# ZIP (no subir)
*.zip

# Temporales
*.tmp
*.bak
*~

# macOS
.DS_Store

# Python
__pycache__/
*.pyc
```

---

## 10. Guía Completa: Pines Jerárquicos en Sheet Symbols

### Conceptos Clave

Un **sheet symbol** en KiCad es el rectángulo que aparece en el schematic padre (root) y representa un sub-schematic (child). Los **pins** son las conexiones que permiten pasar señales entre el padre y el hijo.

**Anatomía de un pin (KiCad 8+):**
```
(pin "NOMBRE" electrical_type
    (at X Y ROT)        ← Punto de conexión (donde se conectan los wires)
    (uuid "...")
    (effects ...)
)
```

**Tipos eléctricos válidos:**
- `input` - Entrada
- `output` - Salida
- `bidirectional` - Bidireccional
- `tri_state` - Tri-estado
- `passive` - Pasivo
- `power_in` - Power input
- `power_out` - Power output

### Regla #1: SIEMPRE espaciado de 2.54mm

El estándar de KiCad es 2.54mm (100 mil) entre pines.

```
RECTÁNGULO              PIN_IZQ              PIN_DER
┌──────────┐            •───────────────• PIN_1 (y=0)
│          │            •───────────────• PIN_2 (y=2.54)
│  CHILD   │            •───────────────• PIN_3 (y=5.08)
│          │
└──────────┘
         X = sheet_x                   X = sheet_x + width
```

### Fórmulas

```python
SPACING = 2.54  # mm entre cada pin (estándar KiCad)
MARGIN = 2.54   # mm desde borde superior del sheet

# Pines IZQUIERDOS (rotación 180°)
at_x = sheet_x
at_y = sheet_y + MARGIN + (index * SPACING)

# Pines DERECHOS (rotación 0°)
at_x = sheet_x + sheet_width
at_y = sheet_y + MARGIN + (index * SPACING)
```

### Ejemplo Completo: Sheet con 6 pines

```python
import uuid as uuid_mod

# Datos del sheet
sheet_x = 100.0
sheet_y = 50.0
sheet_w = 55.0
sheet_h = 40.0
spacing = 2.54
margin = 2.54

# Pines del sub-schematic (ordenados por puerto)
left_labels = ['AN8', 'AN9', 'RA0']   # 3 pines izquierdos
right_labels = ['RD2', 'RD3', 'RD4']  # 3 pines derechos

# Generar pines izquierdos
for i, name in enumerate(left_labels):
    y = sheet_y + margin + (i * spacing)
    pin = f'''    (pin "{name}" input
        (at {sheet_x} {y:.2f} 180)
        (uuid "{str(uuid_mod.uuid4())}")
        (effects
            (font (size 1.27 1.27))
        )
    )'''

# Generar pines derechos
for i, name in enumerate(right_labels):
    y = sheet_y + margin + (i * spacing)
    pin = f'''    (pin "{name}" input
        (at {sheet_x + sheet_w} {y:.2f} 0)
        (uuid "{str(uuid_mod.uuid4())}")
        (effects
            (font (size 1.27 1.27))
        )
    )'''
```

### Errores Comunes

| Error | Causa | Solución |
|-------|-------|----------|
| Zig-zag verde | `at` en el borde del rectángulo | Mover `at` fuera del rectángulo |
| Pines superpuestos | Espaciado < 2.54mm | Usar spacing = 2.54 |
| Pins fuera de vista | Sheet muy pequeño | Calcular `sheet_h` según num_pins |
| Pins mezclados | Búsqueda regex incorrecta | Usar `instances` como límite |

### Script de Verificación

```python
import re

def verify_sheet_pins(filepath, sheet_uuid):
    with open(filepath) as f:
        content = f.read()
    
    idx = content.find(f'uuid "{sheet_uuid}"')
    sheet_start = content.rfind('(sheet', 0, idx)
    instances_idx = content.find('(instances', idx)
    sheet_section = content[sheet_start:instances_idx]
    
    # Encontrar posición del sheet
    at_match = re.search(r'\(at ([\d.]+) ([\d.]+)\)', content[sheet_start:idx])
    sheet_x = float(at_match.group(1))
    
    size_match = re.search(r'\(size ([\d.]+) ([\d.]+)\)', content[sheet_start:idx])
    sheet_w = float(size_match.group(1))
    
    # Verificar spacing entre pins
    pins_y = []
    for m in re.finditer(r'\(pin "[^"]+" \w+\s*\n\s*\(at ([\d.]+) ([\d.]+) (\d+)\)', sheet_section):
        x, y, rot = float(m.group(1)), float(m.group(2)), int(m.group(3))
        pins_y.append((y, rot))
    
    pins_y.sort()
    for i in range(1, len(pins_y)):
        diff = pins_y[i][0] - pins_y[i-1][0]
        if abs(diff - 2.54) > 0.1:
            print(f'WARNING: Spacing {diff:.2f}mm (expected 2.54mm)')
    
    print('Verification complete')
```

### Resumen de Reglas

1. **Siempre** espaciado de 2.54mm entre pins
2. **Siempre** usar electrical_type válido (input, output, bidirectional, etc.)
3. **Calcular** altura del sheet según número de pins
4. **Ordenar** pins por puerto (RA, RB, RC, RD, etc.)
5. **Verificar** que todos los pins están en la posición correcta

---

## 11. Flujo de Trabajo Completo

1. **Explorar**: Encontrar todos los archivos en el directorio
2. **Deduplicar**: Identificar y eliminar duplicados
3. **Organizar**: Crear estructura limpia (symbols, footprints, 3d)
4. **Mergear**: Combinar símbolos en un solo archivo
5. **Verificar**: Contar archivos, detectar faltantes
6. **Documentar**: Actualizar README con contenido real
7. **Licenciar**: Agregar GPL v2+ (requerido por KiCad)
8. **Git init**: Inicializar repositorio
9. **Commit**: Guardar cambios
10. **Push**: Subir a GitHub (con token temporal)
11. **Limpiar**: Remover token del remote

---

## 12. Búsqueda de Credenciales en macOS (Referencia Genérica)

> **IMPORTANTE**: Esta sección es solo para referencia. Nunca incluir datos reales en documentación.

### Ubicaciones Comunes de Credenciales

```bash
# 1. Keychain de macOS (método preferido)
security find-generic-password -s "SERVICIO" -a "USUARIO" -w 2>/dev/null

# 2. Archivos de configuración de Git
cat ~/.gitconfig
cat ~/.git-credentials

# 3. Variables de entorno
env | grep -i "token\|key\|secret\|password"

# 4. Archivos .env en proyectos
find . -name ".env*" -exec cat {} \; 2>/dev/null
```

### Ejemplos de Búsqueda por Servicio

```bash
# GitHub
security find-generic-password -s "github.com" -a "USUARIO" -w 2>/dev/null

# GitLab
security find-generic-password -s "gitlab.com" -a "USUARIO" -w 2>/dev/null

# Bitbucket
security find-generic-password -s "bitbucket.org" -a "USUARIO" -w 2>/dev/null

# AWS
security find-generic-password -s "aws" -a "USUARIO" -w 2>/dev/null
```

### Verificar Conexión (Sin Exponer Datos)

```bash
# GitHub - verificar autenticación
ssh -T git@github.com 2>&1 | head -1

# GitLab - verificar con API
curl -s https://gitlab.com/api/v4/user 2>/dev/null | grep '"username"'

# Verificar si hay token configurado
git config --global --list | grep -i "credential\|token"
```

### Notas para Otra IA

Al trabajar con este proyecto:
1. **NUNCA** exponer tokens o credenciales en logs
2. **USAR** variables de entorno o keychain para almacenar secrets
3. **LIMPIAR** URLs después de usar tokens
4. **VERIFICAR** permisos antes de acceder a credenciales
5. **DOCUMENTAR** ubicaciones de credenciales (sin valores) para referencia
