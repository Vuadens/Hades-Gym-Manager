# Propuesta TP DSW

## Grupo
### Integrantes
* 54879 - Hidalgo Alcain, Juan Manuel
* 54305 - Maza, Joaquin Emiliano
* 54465 - Blesi, Lucas Emiliano
* 53836 - Mendez, Julieta Romina

### Repositorios
* [frontend app](https://github.com/Vuadens/DSW-Frontend)
* [backend app](https://github.com/Vuadens/DSW-Backend)

Ambos están incluidos en este repositorio como submódulos, en las carpetas `DSW-Frontend` y `DSW-Backend`.


## Tema
### Descripción
Sistema web para la administración centralizada de un gimnasio. Permite gestionar socios, planes y cuotas, además de organizar
las distintas actividades, asignar profesores a clases (controlando cupos), y llevar un registro del mantenimiento del equipamiento.

### Modelo
<img width="1763" height="997" alt="MD TPI-Gimnasio" src="https://github.com/user-attachments/assets/52d98d33-294a-4609-83d3-ef545a903fce" />


## Alcance Funcional 

### Alcance Mínimo

Regularidad (implementado):

Las referencias `F` corresponden a pull requests del frontend y las `B`, del backend.

|Req|Detalle|Pull requests|
|:-|:-|:-|
|CRUD simple|1. CRUD Socio<br>2. CRUD Actividad<br>3. CRUD Profesor<br>4. CRUD Plan|1. [B#7](https://github.com/Vuadens/DSW-Backend/pull/7), [B#14](https://github.com/Vuadens/DSW-Backend/pull/14), [F#5](https://github.com/Vuadens/DSW-Frontend/pull/5)<br>2. [B#6](https://github.com/Vuadens/DSW-Backend/pull/6), [F#6](https://github.com/Vuadens/DSW-Frontend/pull/6)<br>3. [B#4](https://github.com/Vuadens/DSW-Backend/pull/4), [B#14](https://github.com/Vuadens/DSW-Backend/pull/14), [F#5](https://github.com/Vuadens/DSW-Frontend/pull/5)<br>4. [B#5](https://github.com/Vuadens/DSW-Backend/pull/5), [F#11](https://github.com/Vuadens/DSW-Frontend/pull/11)|
|CRUD dependiente|1. CRUD Clase {depende de} CRUD Profesor y CRUD Actividad<br>2. CRUD Inscripción {depende de} CRUD Clase y CRUD Socio|1. [B#8](https://github.com/Vuadens/DSW-Backend/pull/8), [B#17](https://github.com/Vuadens/DSW-Backend/pull/17), [F#7](https://github.com/Vuadens/DSW-Frontend/pull/7)<br>2. [B#8](https://github.com/Vuadens/DSW-Backend/pull/8), [B#17](https://github.com/Vuadens/DSW-Backend/pull/17), [F#7](https://github.com/Vuadens/DSW-Frontend/pull/7)|
|Listado<br>+<br>detalle| 1. Listado de clases filtrado por día de la semana, muestra horario, cupo, inscriptos y lugares disponibles => detalle muestra la actividad, el profesor a cargo, la próxima fecha y los cupos. <br> 2. Listado de socios filtrado por plan, muestra DNI, nombre y apellido => detalle muestra los datos completos de contacto, el plan y el estado de sus cuotas. El estado de las cuotas solo lo ve el administrador.|1. [B#17](https://github.com/Vuadens/DSW-Backend/pull/17), [F#8](https://github.com/Vuadens/DSW-Frontend/pull/8), [F#11](https://github.com/Vuadens/DSW-Frontend/pull/11)<br>2. [B#12](https://github.com/Vuadens/DSW-Backend/pull/12), [B#16](https://github.com/Vuadens/DSW-Backend/pull/16), [F#8](https://github.com/Vuadens/DSW-Frontend/pull/8), [F#11](https://github.com/Vuadens/DSW-Frontend/pull/11)|
|CUU/Epic|1. Inscribir a un socio al dictado de una clase para una fecha puntual, validando cupo máximo, plan y cuota del mes.<br>2. Registrar el cobro de la cuota mensual de un socio.|1. [B#17](https://github.com/Vuadens/DSW-Backend/pull/17), [F#7](https://github.com/Vuadens/DSW-Frontend/pull/7)<br>2. [B#12](https://github.com/Vuadens/DSW-Backend/pull/12), [B#19](https://github.com/Vuadens/DSW-Backend/pull/19), [F#10](https://github.com/Vuadens/DSW-Frontend/pull/10)|

Los materiales no se muestran en el detalle de la clase durante esta etapa: dependen del CRUD Material, previsto para Aprobación.


Adicionales para Aprobación (alcance previsto para el sistema completo):

|Req|Detalle|Estado|
|:-|:-|:-|
|CRUD |1. CRUD Socio<br>2. CRUD Actividad<br>3. CRUD Profesor<br>4. CRUD Plan<br>5. CRUD Clase<br>6. CRUD Inscripción<br>7. CRUD Turno<br>8. CRUD Material<br>9. CRUD Máquina<br>10. CRUD Administrador|1 a 6. Implementados<br>7. Implementado<br>8. Pendiente<br>9. En desarrollo ([B#18](https://github.com/Vuadens/DSW-Backend/pull/18), [F#9](https://github.com/Vuadens/DSW-Frontend/pull/9))<br>10. Implementado en el backend; falta la interfaz|
|CUU/Epic|1. Inscribir a un socio al dictado de una clase (validando cupo máximo).<br>2. Registrar el cobro de la cuota mensual de un socio.<br>3. Registrar una orden de mantenimiento preventivo/correctivo sobre una máquina.<br>4.  Asignar e inicializar el histórico de sueldo de un profesor|1 y 2. Implementados<br>3 y 4. Pendientes|

### Alcance Adicional Voluntario

*Nota*: El Alcance Adicional Voluntario es opcional, pero ayuda a que la funcionalidad del sistema esté completa y será considerado en la nota en función de su complejidad y esfuerzo.

|Req|Detalle|Estado|
|:-|:-|:-|
|Listados |1. Historial de máquinas en mantenimiento filtrado por fechas, muestra máquina y estado => detalle  muestra fechas de inicio/fin reales, esperadas y descripción |Pendiente|
|CUU/Epic|1. Cancelación de la inscripción de un socio a una clase. |Implementado ([B#17](https://github.com/Vuadens/DSW-Backend/pull/17), [F#7](https://github.com/Vuadens/DSW-Frontend/pull/7))|
|Otros|1. Integración de un sistema de pagos como Mercado Pago o Stripe através de su API. <br>2. Envío de comprobante automático por email al socio al registrar el pago de su cuota.<br>3. Registro público de socios, con contraseña inicial y cambio obligatorio en el primer ingreso.|1 y 2. Pendientes<br>3. Implementado ([B#19](https://github.com/Vuadens/DSW-Backend/pull/19), [F#10](https://github.com/Vuadens/DSW-Frontend/pull/10))|

