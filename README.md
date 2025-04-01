# Parcial #2 - Automatización y Control de Procesos

## Integrantes 

Tomas Candelo Montoya

Juan José Cifuente Cuellar

Ivan Alejandro Gutierrez 

Melissa Perez Patiño

**** 

# Contexto

Para este examen parcial #2 de la asignatura de Automatización y control de procesos se nos asigno el diseño de un sistem automatizado para la preparación de una taza de cafe de 6 oz. Este diseño debe de ser realizado en lenguaje ladder e implementado a través de openPLC en una maqueta funcinal que incluya sensores y actuadores reales. Como especificaciones para la implementación teniamos que tener en cuenta los siguientes:

- Para poder empezar el proceso de mezcla de debe iniciar el sistema por medio de un interruptor. 
- La maquina internamente debe vertir el café molido y el agua caliente en el recipiente de mezcla 
- El actuador de la mezcla se debe activar durante un minuto posteriomente a que hay recibido el agua y el café por separado 
- Cuando la mezcla termine, se vierte en un vaso el café listo
-Una vez el café se termino de servir, la maquina se apaga y el contador de vasos preparados aumenta en 1

En este documento se mostrara todo el proceso de ideación diseño e implementación de nuestra solución para la maquina de café. 

# Diseño del circuito lógico para el proceso automatizado 

## Diseño Lógico 
Para el diseño logico de la solución lo primero que se hizo fue la definición de las posibles variables necesarias a implementar para posteriormente implementarlas en un diagrma secuencial. En total se realizaron 2 definiciones de variables, la inicial sirvio para definir los requerimeintos minimos que tendria nuestro proyecto, para poder pasar a codesys y verificar si este primer borrador de variables cumpliese con el funcionamiento. 

Una vez se implemento en codesys a cusa de que hacian falta variables necesaarias para la implementación de la solución se realizo una actualización de esta lista de variables con todas las variables que ya se tenian y las que tuvieron que ser creadas, tanto para el funcionamiento real de la solución, como para la simulación que posteriormente seria realizada en codesys a través de un HMI. 

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

Ahora a parte de estas variables que son las necesarias para el funcionamiento como tal del la solución tambien se implementaron variables que fueron necesarias para la ejecución de la animación del HMI, las cuales fueron las siguientes: 

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


Ahora con todas estas variables  podemos pasar a los siguientes pasos de la fase de diseño de nuestra solución. 

### Diagrama secuencial 
Este diagrama es el más importante de tdos, es el que nos permitio entender como nuestras variables se relacionaran a modo de condición o de proceso dentro de nuestra solución. Este diagrama tambien fue realizado en dos fases diferentes, la primera donde, con las variables iniciales que teniamos, se planteo la secuencialidad que seguiria la cafetera, el orden en el que se realizaran diferentes verificaciones y cuando, la unión de diferentes condiciones se de, se activaran los diferentes procesos de la solución. La segunda opción fue una modificación a los inpus y outputs involucrados en el diagrama, la logica como tal no cambio, solo cambiaron en que momento se activavan ciertos inputs y como se verian afectados ciertos outpus del sistema. 

El diagrama secuancial definitivo qu se desarrollo fue el siguiente: 

![Diagrama secuencial](Images/diagrama_secuencial.png)

En este podemos ver los diferentes procesos que realizamos, estos pueden ser resumidos de esta manera:

- **S000**: Es el proceso previo a cualquier cosa, donde realiza un reinicio de todos los actuadores e indicadores de estado del sistema a través de un reinicio de los mismos 

- **S001**: Este sera el primer proceso, encargado unicamente de encender el indicador de que el sesnsor del tanque de agua detecta que hay agua en el tanque.

- **S002**: Este es similar al proceso anterior pero para el tanque de cafe molido.

- **S003**: Este tambien es similar al anterior, pero en este caso verifica que haya un vaso en el espacio designado para este mismo antes de empezar el proceso.

- **S004**: Ahora que todas las validaciones de contenido se realizó su pertinente verificación, podemos accionar el botón de encendido, el cual inciara el proceso de mezcla del café, esto por medio de los indicadores de que la maquina esta encendida, el actuador del tanque de agua que permitira el paso de esta al tanque de mezclado y acciona el timmer que indicara cuanto tiempo estara activo el actuador. 

- **S005**: Este proceso sera el encargado de vertir el contenido del tanque del café al tanque de mezclado, ese proceso sera activado cuando el rele que indica que el timmer del agua acabo se active, con este proceso se apagara el actuador del tanque de agua y se activara el actuador del tanque de café y su timmer. 

- **S006**: Este proceso activara la etapa de mezclado del contenido en el momento que el timmer del tanque de café se termine, en este proceso se hara algo similar al caso anterior donde se apaga el actuador del tanque del café y enciende el actuador y timmer del tanque de mezclado.

- **S007**: Este proceso sera el ultimo proceso involucrado en el proceso de la hecha de un vaso de café ya que este consiste en el proceso de dispensado del liquido mezclado al vaso, este se activara cuando termine el timmer de un minuto del tanque de mezclado, cuando esto ocurra se apagara el actuador de este tanque e iniciara el actuador y timmer encargados de servir el liquido.

- **S008**: Cuando temina el proceso de dispensado lo unico que se realiza en esta etapa es el reinicio del actuador del dispensador y la activación del rele que indica que el proceso esta listo.

- **S009**: Con la activación del rele IR_DONE podemos llevar a cabo el proceso final de la cafetera, el cual consiste en realizar diferentes acciones posteriores al proceso de servir la taza de café, primero apagara el rele e indicador de funcionamiento y encendido del proceso, luego iniciara el indicador de que el proceso ha terminado, finalmente con el rele del contador, envia un flanco de subida que sumara 1 a la variable del contador de vasos preparados en el día. 

Despues de este ultimo proceso el sistema se cierra y vuelve a empezar la secuencia cuando se ponga un vaso nuevo. 


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

 
