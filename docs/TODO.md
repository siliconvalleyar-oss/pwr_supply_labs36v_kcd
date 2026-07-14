# TODO - Fuente de Laboratorio 0-36V DC

## Estado del Proyecto

| Fase | Estado | Notas |
|------|--------|-------|
| Esquemáticos | ✅ Completado | 4 hojas jerárquicas |
| Pines jerárquicos | ✅ Completado | Convertidos de global labels |
| PCB Layout | 🔄 En progreso | Verificar layout |
| BOM | ✅ Completado | 14 componentes |
| Datasheets | ✅ Completado | En carpeta helps/ |

## Tareas Pendientes

### Alta Prioridad
- [ ] Verificar ERC (Electrical Rules Check) en todos los esquemáticos
- [ ] Revisar DRC en PCB
- [ ] Verificar que todos los footprints están en la librería
- [ ] Validar que los 3D models (.step) coinciden con footprints

### Media Prioridad
- [ ] Crear Bill of Materials (BOM) completo
- [ ] Agregar test points en PCB
- [ ] Documentar valores de componentes críticos
- [ ] Revisar thermal relief en power components

### Baja Prioridad
- [ ] Agregar serigrafía con logo
- [ ] Crear enclosures 3D
- [ ] Documentar procedimiento de calibración
- [ ] Agregar fotos del prototipo terminado

## Notas de Diseño

### Topología
- **Tipo**: Lineal, regulador serie
- **Entrada**: Transformador AC → Rectificador → Filtro
- **Control**: BJT (MJL21194/MJL21193) + MOSFET (5.5N45V)
- **Salida**: 0-36V DC, 5A máximo

### Hojas Jerárquicas
1. `power_supply_36v.kicad_sch` - Fuente principal
2. `control_driver.kicad_sch` - Driver de control
3. `pwr_driver_output.kicad_sch` - Driver de potencia
4. `output_pwr.kicad_sch` - Etapa de salida

## Referencias

- [ ] Datasheet MJL21194/MJL21193
- [ ] Datasheet 5.5N45V MOSFET
- [ ] Datasheet TIP41C/TIP42C
- [ ] App notes de reguladores lineales
