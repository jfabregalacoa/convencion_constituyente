# CONVENCIÓN CONSTITUYENTE - CHILE 2021

## INTRODUCCIÓN
Entre el 15 y 16 de mayo del 2021, Chile eligió a 155 personas para que redactaran una nueva constitución. Muchas de ellas no tenían militancia o adherían a partidos o conglomerados políticos por lo que sus posturas ideológicas no eran fácilmente identificables.
En el diario La Tercera hicieron una encuesta a todos los candidatos a nivel nacional. De poco más de 1200 candidatos, más de 950 respondieron.
Este repositorio contiene las estimaciones de las posturas ideológicas de los que finalmente fueron elegidos a partir de esas respuestas de La Tercera. 
Para una revisión de las posturas ideológicas del conjunto de postulantes visite: [aquí](https://www.linkedin.com/pulse/estimaci%25C3%25B3n-de-las-posiciones-ideol%25C3%25B3gicas-en-cada-uno-los-fabrega). El método usado para la estimación está descrito en "The Statistical Analysis of Roll Call Data" (Jackman, Clinton y Rivers, APSR, 2004, vol 98(2): 355-370. 


## ESTIMACION DE IDEOLOGIA EN LA CONVENCIÓN CONSTITUYENTE

El 15 y 16 de mayo del 2021, Chile fue a las urnas para elegir a las 155 personas que deberán redactar un nuevo texto constitucional. Al finalizar la jornada, muchos se llevaron una gran sorpresa al constatar que un número significativo de las personas electas no estaban adscritas a ningún partido político ¿Qué piensan y cómo las ubicamos en los tradicionales ejes ideológicos? 

A medida que las semanas pasen, esa incertidumbre se irá despejando. :smiley:

No obstante, como un esfuerzo para contribuir a reducir la incertidumbre, en este sitio presento una estimación ideológica de los Convencionales Constituyentes, incluidos los independientes. Se trata de un proyecto que se irá actualizando en la medida que se disponga de información nueva y relevante. 

En su versión inicial, se cuenta con una estimación ideológica de 135 de los 155 miembros. En la siguiente sección explico cómo se hizo la estimación y los supuestos que se realizaron. Posteriormente presento algunos gráficos ilustrativos de las posiciones ideológicas de miembros de la convención. Finalmente, presento las estimaciones para que puedan ser usadas por investigadores y público en general que desee construir nuevos análisis a partir de ellos. 

Las bases de datos con las estimaciones están depositadas en el siguiente repositorio de Dataverse.

# EL MÉTODO
El Diario La Tercera hizo una encuesta a los candidatos a la convención constitucional (932 postulantes respondieron su encuesta) y entre los finalmente electos, 120. Los datos fueron utilizados para elaborar un sitio interactivo denominado [Tu March Constituyente](https://interactivo.latercera.com/tu-match-constituyente/candidatos-constituyentes/). La encuesta contenía set de preguntas tales como:  
> ¿Qué se debe introducir en la nueva Constitución respecto a este tema?  
> ¿Qué cambios se le deben realizar al TC?  
> ¿El Congreso Nacional debe tener una o dos cámaras?  
  
Las alternativas de respuesta contenían afirmaciones que tienen la virtud de discriminar posturas políticamente relevantes. Por ejemplo, en relación a la composición del congreso, las alternativas eran:

> a. Debe haber un Congreso bicameral como actualmente existe: Senado y Cámara de Diputadas y Diputados.  
> b. Se debe mantener la estructura de las dos Cámaras pero dejando una de ellas para la representación territorial y otra para la representación política.  
> c. Congreso unicameral: una sola Cámara.  

El ejercicio de estimación convirtió las posibles respuestas en dummies. Ello puede interpretarse del siguiente modo: "¿selecciona el/la candidato/a la alternativa $z$?: Sí o No". Con esta transformación, se contruye una matriz de respuestas que pueden ser interpretadas como votaciones en un cuerpo colegiado. Y, debido a que se trata de preguntas políticamente relevantes, se puede asumir que las diferencias en los patrones observados de respuesta deben responder a concepciones ideológicas que distinguen a unos de otros.  

Bajo esa premisa, se puede proceder a estimar cuál es el mejor vector o conjunto de vectores que mejor refleja dichas diferencias de visión entre los miembros de la convención. Para hacer la estimación se obtuvieron los datos de las respuestas de todos los constituyentes a todas las alternativas y se aplicaron métodos frecuentistas (W-NOMINATE) y bayesianos de estimación de ideología. 

El siguiente gráfico muestra que ambos métodos de estimación generan resultados similares con algunas variaciones en las estimaciones hacia los extremos de la distribución.

![Figura 1](/plots/fig1.png)

Dado el resultado anterior, el resto del análisis se hizo considerando la estimación bayesiana puesto que permite generar intervalos de confiabilidad sobre los resultados obtenidos. 

![Figura 2](/plots/fig2.png)

  
El siguiente gráfico resume los resultados de la estimación. Cada punto representa un miembro de la convención. La línea horizontal que cruza cada punto, es el intervalo de confiabilidad estimado. Dado que son muchas personas, sólo se incluyen los nombres de algunas de ellas en el gráfico o de lo contrario, no se podrían distinguir. 

Ahora bien, lo más relevante de la figura es lo que está representado en los colores. En rojo se destaca a los votantes pivotales bajo una regla de 2/3. Mirando de arriba hacia abajo, el primer punto rojo es el votante pivotal para incluir en la constitución algún texto que es preferido desde miembros ubicados a la izquierda de la distribución. El punto rojo de más abajo es el votante pivotal cuando dichos textos son promovidos desde la derecha de la distribución. Los puntos en amarillo contienen un rango de veinte personas en torno al punto rojo. Ese rango refleja el hecho que hay aún veinte personas para las que no tenemos una estimación ideológica. Por ende, ese es el rango en que puede variar la ubicación del votante pivotal en cada caso.

Por último, es importante aclarar que ésta es una escala normalizada (esto es, centrada en cero). Por ende, no debe interpretarse el gráfico como si los números positivos significa que las personas allí representadas son de derecha y los números negativos, de izquierda. Al respecto no debe olvidarse que las candidaturas políticamente identificadas como de derecha (Lista Vamos por Chile) no alcanzaron un tercio del total de escaños.   

Ahora bien, una de las preguntas que más interesa sabes es dónde están ubicados ideológicamente las personas elegidas que no provienen de los partidos y grupos políticos tradicionales. Particularmente, ¿dónde se ubican ideológicamente las personas de la lista del pueblo? En el siguiente gráfico se presenta la misma información del gráfico precedente pero agrupados por lista. Como puede verse, la lista del pueblo se ubica en un punto intermedio de Apruebo Dignidad y la Lista del Apruebo. 

![Figura 3](/plots/fig3.png)

Finalmente, volviendo sobre la identificación de actores pivotales. En el gráfico siguiente se destacan los actores que serían claves en la discusión en la convención constituyente. Actores que por su ubicación en la distribución ideológica están son los que con mayor probabilidad dan los mínimos necesarios para cumplir con los quórums que la convención establezca en su reglamento.

En azul están los votantes medianos. Es decir, los que tienen al enumerar hacia su derecha o izquierda se contabiliza la misma cantidad de personas. Dado eso, ellos son los actores que dan las mayorías simples para proyectos. Y en rojo se presentan los actores pivotales de 2/3 explicado anteriormente. Ahora bien, a estas alturas que el proceso constituyente está empezando, más importante que saber los nombres específicos, es entender las visiones que están en una posicioón de ser pivotales en una discusión. Ello por dos motivos, primero porque la propia estimación tiene un margen de variación que ya expliqué en un gráfico anterior y, por ende, para efectos estadísticos cuando dos miembros de la convención tienen estimaciones traslapadas, no podemos afirmar cuál de ellos está más a la derecha o más a la izquierda. Y segundo, porque no hay que olvidar que hay aún 20 personas para las cuales este ejercicio no ha hecho una estimación de su posición ideológica. 

![Figura 4](/plots/fig4.png)

Los datos que sustentan los gráficos precedentes se encuentran disponibles [aquí](https://dataverse.harvard.edu/file.xhtml?fileId=4656268&version=1.0)

Si usted tiene comentarios o consultas, por favor dirigirlas al email jfabrega@udd.cl.

