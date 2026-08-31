# Propuesta TP DSW

## Grupo
### Integrantes
* 54879 - Hidalgo Alcain, Juan Manuel
* 54305 - Maza, Joaquin Emiliano
* 54465 - Blesi, Lucas Emiliano
* 53836 - Mendez, Julieta Romina

### Repositorios
* [frontend app](https://github.com/Vuadens/DSW-frontend)
* [backend app](https://github.com/Vuadens/DSW-Backend)


## Tema
### Descripción
Sistema web para la administración centralizada de un gimnasio. Permite gestionar socios, planes y cuotas, además de organizar
las distintas actividades, asignar profesores a clases (controlando cupos), y llevar un registro del mantenimiento del equipamiento.

### Modelo
<img width="1763" height="997" alt="MD TPI-Gimnasio" src="https://github.com/user-attachments/assets/52d98d33-294a-4609-83d3-ef545a903fce" />


## Alcance Funcional 

### Alcance Mínimo

Regularidad:
|Req|Detalle|
|:-|:-|
|CRUD simple|1. CRUD Socio<br>2. CRUD Actividad<br>3. CRUD Profesor<br>4. CRUD Plan|
|CRUD dependiente|1. CRUD Clase {depende de} CRUD Profesor<br>3. CRUD Inscripción {depende de} CRUD Clase y CRUD Socio|
|Listado<br>+<br>detalle| 1. Listado de clases filtrado por día de la semana, muestra horario y cupos disponibles => detalle muestra la actividad, el profesor a cargo y los materiales requeridos. <br> 2. Listado de socios filtrado por plan, muestra DNI, nombre y apellido => detalle muestra los datoscompletos de contacto y el estado de sus cuotas.|
|CUU/Epic|1. Inscribir a un socio al dictado de una clase (validando cupo máximo).<br>2. Registrar el cobro de la cuota mensual de un socio.|


Adicionales para Aprobación
|Req|Detalle|
|:-|:-|
|CRUD |1. CRUD Socio<br>2. CRUD Actividad<br>3. CRUD Profesor<br>4. CRUD Plan<br>5. CRUD Clase<br>6. CRUD Inscripción<br>7. CRUD Turno<br>8. CRUD Material<br>9. CRUD Máquina<br>10. CRUD Administrador|
|CUU/Epic|1. Inscribir a un socio al dictado de una clase (validando cupo máximo).<br>2. Registrar el cobro de la cuota mensual de un socio.<br>3. Registrar una orden de mantenimiento preventivo/correctivo sobre una máquina.<br>4.  Asignar e inicializar el histórico de sueldo de un profesor|

### Alcance Adicional Voluntario

*Nota*: El Alcance Adicional Voluntario es opcional, pero ayuda a que la funcionalidad del sistema esté completa y será considerado en la nota en función de su complejidad y esfuerzo.

|Req|Detalle|
|:-|:-|
|Listados |1. Historial de máquinas en mantenimiento filtrado por fechas, muestra máquina y estado => detalle  muestra fechas de inicio/fin reales, esperadas y descripción |
|CUU/Epic|1. Cancelación de la inscripción de un socio a una clase. |
|Otros|1. Integración de un sistema de pagos como Mercado Pago o Stripe através de su API. <br>2. Envío de comprobante automático por email al socio al registrar el pago de su cuota.|

