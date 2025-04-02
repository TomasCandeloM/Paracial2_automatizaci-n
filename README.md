# Parcial #2 - Automatización y Control de Procesos

## Integrantes 

Tomas Candelo Montoya

Juan José Cifuentes Cuellar

Ivan Gutierrez Espinosa

Melissa Pérez Patiño

**** 

# Contexto

Para este Examen Parcial #2 de la asignatura Automatización y Control de Procesos, se nos asignó el diseño de un sistema automatizado para la preparación de una taza de café de 6 oz.

Este diseño debe ser realizado en lenguaje Ladder e implementado a través de OpenPLC en una maqueta funcional que incluya sensores y actuadores reales.

Especificaciones del Sistema

Para la implementación, se consideraron los siguientes puntos:
	•	Inicio del proceso: El sistema debe activarse mediante un interruptor.
	•	Preparación de la mezcla: La máquina debe verter automáticamente el café molido y el agua caliente en el recipiente de mezcla.
	•	Activación del actuador de mezcla: Se debe activar durante un minuto después de haber recibido el agua y el café por separado.
	•	Dispensado del café: Una vez completada la mezcla, el café listo se vierte en un vaso.
	•	Finalización y conteo: Al terminar de servir el café, la máquina se apaga y el contador de vasos preparados aumenta en uno.

En este documento, se detallará el proceso de ideación, diseño e implementación de nuestra solución para la máquina de café.

# Diseño del circuito lógico para el proceso automatizado 

## Diseño Lógico 
Para el diseño lógico de la solución lo primero que se hizo fue la definición de las posibles variables necesarias a implementar, para posteriormente implementarlas en un diagrma secuencial. En total, se realizaron dos definiciones de variables, la inicial sirvió para definir los requerimeintos mínimos que tendrían nuestro proyecto, para poder pasar a Codesys y verificar si este primer borrador de variables cumple con el funcionamiento. 

Una vez se implementó en Codesys, a causa de que hacían falta variables necesarias para la implementación de la solución se realizó una actualización de esta lista de variables con todas las variables que ya se tenían y las que tuvieron que ser creadas, tanto para el funcionamiento real de la solución, como para la simulación que posteriormente sería realizada en Codesys a través de un HMI (Human Machine Interaction). 

La lista de variables final que se utilizaron en nuestra solución fueron las siguientes:

| Nombre               | Atributo      | Tipo   | Comentario                                                    |
|----------------------|--------------|--------|----------------------------------------------------------------|
| I_SNS_Agua          | [Input]       | BOOL   | Sensor que mide que el tanque de agua no está vacío          |
| I_SNS_Cafe          | [Input]       | BOOL   | Sensor que mide que el tanque de café no está vacío          |
| I_SNS_Vaso          | [Input]       | BOOL   | Sensor que mide que hay un vaso puesto en la máquina         |
| I_BTN_ON           | [Input]       | BOOL   | Botón que permite iniciar el proceso de dispensar café      |
| I_BTN_STOP         | [Input]       | BOOL   | Botón que detiene el proceso de dispensar café              |
| O_ACT_AGUA         | [Output]      | BOOL   | Válvula que permite el paso del agua al tanque mezclador    |
| O_ACT_Cafe         | [Output]      | BOOL   | Válvula que permite el paso del café al tanque mezclador    |
| O_ACT_Mezcla       | [Output]      | BOOL   | Válvula que mezcla los contenidos del tanque mezclador      |
| O_ACT_Dispensador  | [Output]      | BOOL   | Válvula que permite el paso de la mezcla al vaso            |
| O_DISP_Contador    | [Output]      | INT    | Contador que indica el número de vasos dispensados al día   |
| O_LED_Agua        | [Output]      | BOOL   | Led que indica la presencia de agua en el tanque mezclador  |
| O_LED_Cafe        | [Output]      | BOOL   | Led que indica la presencia de café en el tanque mezclador  |
| O_LED_Vaso        | [Output]      | BOOL   | Led que indica que el vaso está en su lugar                |
| O_LED_ON          | [Output]      | BOOL   | Led que indica que la máquina está encendida               |
| O_LED_DONE        | [Output]      | BOOL   | Led que indica que el proceso terminó                      |
| TON_Mezcla        | [T_On]        | TON    | Timer que mide el minuto especificado de mezcla            |
| TON_Agua          | [T_On]        | TON    | Timer que mide el tiempo de salida del agua del tanque     |
| TON_Cafe          | [T_On]        | TON    | Timer que mide el tiempo de salida del café del tanque     |
| TON_Dispensador   | [T_On]        | TON    | Timer que mide el tiempo de dispensado del producto final  |
| IR_Timer_Agua     | [Relay]       | BOOL   | Relay interno que enciende el timer de agua                |
| IR_Timer_Cafe     | [Relay]       | BOOL   | Relay interno que enciende el timer café                   |
| IR_Timer_Mezcla   | [Relay]       | BOOL   | Relay interno que enciende el timer mezcla                 |
| IR_Timer_Dispensador | [Relay]   | BOOL   | Relay interno que enciende el timer dispensador            |
| IR_ON             | [Relay]       | BOOL   | Relay interno que mantiene el sistema en funcionamiento    |
| IR_Agua          | [Relay]       | BOOL   | Relay interno que indica la presencia de agua en el proceso|
| IR_Cafe          | [Relay]       | BOOL   | Relay interno que indica la presencia de café en el proceso|
| IR_Vaso          | [Relay]       | BOOL   | Relay interno que indica la presencia del vaso en su lugar |
| IR_DONE_Agua     | [Relay]       | BOOL   | Relay interno que indica que finalizó el proceso de agua   |
| IR_DONE_Cafe     | [Relay]       | BOOL   | Relay interno que indica que finalizó el proceso de café   |
| IR_DONE_Mezcla   | [Relay]       | BOOL   | Relay interno que indica que finalizó el proceso de mezcla |
| IR_DONE_Dispensador | [Relay]   | BOOL   | Relay interno que indica que finalizó el proceso de dispensado |
| IR_DONE             | [Relay]   | BOOL   | Relay interno que indica que el proceso esta listo y se le ha dado start al proceso
| IR_READY             | [Relay]   | BOOL   | Relay interno que indica que el proceso cuenta con las condiciones iniciales necesarias para empezar
| IR_Contador          | [Relay]   | BOOL   | Relay interno que envia el flanco de subida que suma 1 a la variable contador

Ahora, además de estas variables que son las necesarias para el funcionamiento de la solución, también se implementaron variables que fueron necesarias para la ejecución de la animación del HMI, las cuales fueron las siguientes: 

| Nombre                 | Atributo  | Tipo  | Comentario                                                               |
|------------------------|----------|------|---------------------------------------------------------------------------|
| HMI_Vaso              | [HMI]    | BOOL | Botón del HMI que indica la presencia o ausencia del vaso en su posición |
| HMI_Agua              | [HMI]    | BOOL | Botón del HMI que indica la presencia o ausencia de agua                 |
| HMI_Cafe              | [HMI]    | BOOL | Botón del HMI que indica la presencia o ausencia de café                 |
| HMI_Agua_Mezclador    | [HMI]    | BOOL | Variable que indica que el agua ha sido depositada en el mezclador mediante un indicador visual |
| HMI_Cafe_Mezclador    | [HMI]    | BOOL | Variable que indica que el café ha sido depositado en el mezclador mediante un indicador visual |
| HMI_Mezcla            | [HMI]    | BOOL | Indicador visual (mediante el color) de que el agua y el café han sido mezclados |
| HMI_Servido           | [HMI]    | BOOL | Indicador visual de que la mezcla está siendo servida en el vaso         |
| HMI_Reset             | [HMI]    | BOOL | Representa el proceso de retirar el vaso del lugar, el cual se ve representado en el HMI como retirar el líquido del vaso |


Ahora, con todas estas variables  podemos pasar a los siguientes pasos de la fase de diseño de nuestra solución. 

### Diagrama secuencial 
Este diagrama es el más importante de todos, es el que nos permitió entender cómo nuestras variables se relacionarán a modo de condición o de proceso dentro de nuestra solución. Este diagrama tambien fue realizado en dos fases diferentes, la primera donde, con las variables iniciales que teníamos, se planteó la secuencialidad que seguiria la cafetera, el orden en el que se realizarán diferentes verificaciones y cuando, la unión de diferentes condiciones se de, se activarán los diferentes procesos de la solución. La segunda opción fue una modificación a los inputs y outputs involucrados en el diagrama, la lógica como tal no cambió, solo cambiaron en que momento se activarán ciertos inputs y como se verían afectados ciertos outpus del sistema. 

El diagrama secuencial definitivo que se desarrolló fue el siguiente: 

![Diagrama secuencial](Images/diagrama_secuencial.png)

En este esquema, podemos ver los diferentes procesos involucrados en la preparación de un vaso de café. Estos pueden resumirse de la siguiente manera:

- **S000**: Es el proceso previo a cualquier operación. Su función es reiniciar todos los actuadores e indicadores de estado del sistema, garantizando que todo esté en condiciones iniciales antes de comenzar.

- **S001**: Es el primer proceso y se encarga únicamente de encender el indicador que confirma la detección de agua en el tanque por parte del sensor.

- **S002**: Similar al proceso anterior, pero en este caso verifica la presencia de café molido en su respectivo tanque.

- **S003**: También es similar al anterior, pero aquí se asegura que haya un vaso colocado en el espacio designado antes de iniciar el proceso.

- **S004**: Una vez completadas todas las verificaciones de contenido, se puede accionar el botón de encendido, lo que inicia el proceso de mezcla del café. En esta etapa, se activan los indicadores de encendido de la máquina, el actuador del tanque de agua para permitir el flujo hacia el tanque de mezclado y el temporizador que controlará el tiempo de activación de este actuador.

- **S005**: Este proceso se encarga de verter el contenido del tanque de café en el tanque de mezclado. Se activa cuando finaliza el temporizador del agua, lo que apaga el actuador del tanque de agua y enciende el actuador del tanque de café junto con su temporizador.

- **S006**: Este proceso activará la etapa de mezclado del contenido en el momento que el timmer del tanque de café se termine, en este proceso se hara algo similar al caso anterior donde se apaga el actuador del tanque del café y enciende el actuador y timer del tanque de mezclado.

- **S007**: Este será el último proceso en la preparación de un vaso de café, ya que consiste en el dispensado del líquido mezclado en el vaso. Se activará cuando finalice el temporizador de un minuto del tanque de mezclado. En ese momento, el actuador del tanque se apagará, y se activarán el actuador y el temporizador encargados de servir el líquido.

- **S008**: Cuando termina el proceso de dispensado lo unico que se realiza en esta etapa es el reinicio del actuador del dispensador y la activación del rele que indica que el proceso está listo.

- **S009**: Con la activación del relé IR_DONE, se lleva a cabo el proceso final de la cafetera. En esta etapa, se realizan diferentes acciones posteriores a la dispensación del café. Primero, se apagan el relé y el indicador de funcionamiento. Luego, se activa el indicador de finalización del proceso. Finalmente, mediante el relé del contador, se genera un pulso que incrementa en 1 la variable del contador de vasos preparados en el día.

Después de este último proceso, el sistema se reinicia y la secuencia comienza nuevamente cuando se coloca un nuevo vaso.


### Diagrama electrico


### Diseño Ladder


## Implementación en CodeSys


### Esquemático Ladder del diseño


### Simulación con un HMI 



## Implementación del sistema en OpenPLC



### Diagrama Ladder 



# Validación con Equipo Real y OpenPLC


### Configuración OpenPLC



### Montaje Físico



## Montaje Final

 
