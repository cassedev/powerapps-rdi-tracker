# Power Apps Operations Tracker (R&D-I)

![Power Apps](https://img.shields.io/badge/Power%20Apps-Canvas%20App-742774?style=for-the-badge&logo=powerapps&logoColor=white)
![Power Fx](https://img.shields.io/badge/Power%20Fx-Declarative%20Logic-blueviolet?style=for-the-badge)
![Microsoft Lists](https://img.shields.io/badge/Microsoft%20Lists-Central%20Data%20Source-6A1B9A?style=for-the-badge&logo=sharepoint&logoColor=white)
![Enterprise Solutions](https://img.shields.io/badge/CoE%20CX-Telecomunicaciones-0078D4?style=for-the-badge)
![Data Governance](https://img.shields.io/badge/Data%20Governance-WIP%20Integrity-success?style=for-the-badge)

[![Demo en Vivo](https://img.shields.io/badge/Demo%20Interactiva-GitHub%20Pages-purple?style=for-the-badge)](https://cassedev.github.io/powerapps-rdi-tracker/)

> *Categoría: Casos de Uso Empresariales | Centro de Excelencia CX (Telecomunicaciones)*

> **Aplicación Canvas de autoservicio y seguimiento de requerimientos para solicitantes internos (B2B/B2C). Permite la consulta en tiempo real del ciclo de vida de tickets, el estado coordinado por disciplina técnica (Research, Diseño UX e Implementación), fechas de compromiso y captura de notas operativas directamente sobre Microsoft Lists mediante Power Fx.**

---

## 📌 Contexto Operativo y Desafío en el CoE CX

Tras la ingesta automatizada de solicitudes (vía Forms/Automate) y su posterior visualización en el tablero ejecutivo de Power BI, los clientes internos demandaban un canal directo de autogestión para:
1. **Evitar consultas manuales y correos sueltos:** Conocer al especialista asignado, tablero activo de Planner y fecha límite estimada sin interrumpir la operación del equipo técnico.
2. **Visibilidad multidisciplinaria:** Un requerimiento involucra con frecuencia tareas encadenadas entre Research, Diseño UX e Implementación; la app desglosa el avance individual de cada área técnica.
3. **Control y Auditoría de Notas:** Posibilitar que el usuario anexe comentarios o consultas de seguimiento directamente sobre el ticket registrado sin alterar las columnas de gestión del equipo.

---

## 🛡️ Gobernanza de Datos: Cierre por Bloqueo y Protección del WIP

En la gestión ágil de operaciones, si una solicitud se traba de manera definitiva por falta de viabilidad o dependencias externas (por ejemplo, en la fase de Diseño UX), mantenerla marcada como *En Curso* distorsiona la capacidad real del equipo y falsea el indicador de tiempo medio de resolución (MTTR).

* **Regla de Negocio:** La app implementa el estado resolutivo formal `Cerrada - Bloqueada`.
* **Impacto en el Modelo:** Al seleccionar este estado, el requerimiento se remueve de la cola activa de trabajo (*Work In Progress*), señaliza el motivo raíz en el banner de resolución y concluye la línea de vida con el cierre en alerta, preservando la exactitud analítica del tablero de Power BI.

---

## ⚙️ Arquitectura Técnica y Fórmulas Power Fx

La solución consume de forma nativa la lista maestra `Fact_Solicitudes_Lists` alojada en Microsoft Lists / SharePoint Online:

### 1. Búsqueda Declarativa del Ticket (LookUp)
Recupera el registro unificado del requerimiento según el ID ingresado en el buscador:

Set(
    varTicketActual,
    LookUp(
        'Fact_Solicitudes_Lists',
        Ticket_ID = TextInputSearch.Text
    )
);

### 2. Gestión Visual de Estados y Alertas (Switch)
Evalúa las condiciones de cierre, estados activos o bloqueos operativos para aplicar formato condicional dinámico:

Switch(
    varTicketActual.Estado,
    "Completada", Color.Green,
    "Cerrada - Bloqueada", Color.Red,
    "En Curso", Color.Blue,
    "Backlog", Color.Orange,
    Color.Gray
);

### 3. Sincronización Bidireccional de Notas (Patch)
Permite ingresar notas de auditoría conservando la firma del solicitante y la marca temporal:

Patch(
    'Fact_Solicitudes_Lists',
    varTicketActual,
    {
        Ultima_Nota: TextInputNota.Text,
        Fecha_Ultimo_Estado: Now(),
        Modificado_Por: User().FullName
    }
);

---

## 🔄 Integración de Extremo a Extremo (Ecosistema CoE CX)

Esta aplicación consolida el circuito integral desarrollado para el Centro de Excelencia:

1. **Ingesta (Forms & Automate):** El solicitante remite la necesidad y recibe por correo su identificador oficial (`RDI-2026-XXX`).
2. **Autoservicio (Power Apps):** Con ese identificador consulta el estado, verifica qué disciplina está trabajando y añade notas de seguimiento.
3. **Repositorio Único (Microsoft Lists):** Todas las lecturas y actualizaciones operativas impactan sobre la misma lista centralizada.
4. **Operación de Tareas (Microsoft Planner):** Las aprobaciones y avances técnicos se reflejan en las tarjetas de los tableros de ejecución.
5. **Capa Analítica (Power BI):** Las métricas de SLA, tasas de cierre por bloqueo y tiempos de ciclo se procesan mediante DAX en memoria para consumo directivo.