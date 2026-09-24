#Protocolo SPI (SERIAL PERIPHERAL INTERFACE)

Éste es el protocolo de memoria designado para el funcionamiento de la memoria RAM. Es necesario conocer cómo funciona antes de implementarlo al sistema y para esa finalidad se mencionan aquí las características más importantes. 

El nombre del protocolo, por sus siglas en inglés, significa interfaz de periféricos en serie. Es un bus de interfaz usado comunmente para la comunicación entre microcontroladores y periféricos que trabaja con línea de reloj, datos y selección separadas.
Aunque el protocolo es serial, no consiste en el serial convensional con TX y RX asíncrono sin control sobre las líneas de datos; no se sabe cuándo se envían los datos ni se asegura que estén sincronizados con un único reloj (obligatorio en equipos de computación). Para corregir los problemas producto de la asincronía en el protocolo convencional, se crearon algunos métodos para permitir la lectura correcta de los datos como: 

*Establecer una velocidad de transmisión previa al envío de un byte.
*La aparición de bits de inicio y parada en cada que permitían al receptor corregir las pequeñas diferencias en la velocidad de transmisión.\

Aunque el protocolo serial convencional funcionaba, se generaba mucha carga por la presencia de los bits adicionales y podían producirse errores.



##La solución seríal síncrona
Para corregir los problemas en la diferencia del reloj de la protocolo serial convencional se creó el protocolo SPI. 
